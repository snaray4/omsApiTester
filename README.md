<sub>[Download](https://github.com/jaydeeparikh/SterlingOMSTester/blob/master/tester.tar)</sub>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<sub>[Installation](#installation)</sub>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<sub>[Run from Commad Line](#running-sterling-test-tool-from-command-line)</sub>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<sub>[Run as Server](#running-sterling-test-tool-in-server-mode)</sub>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<sub>[Command Line Arguments](#command-line-arguments)</sub>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<sub>[Notes](#notes)</sub>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

# Sterling OMS Test Tool



It is a stand-alone, lightweight tool for Unit Testing Sterling OMS APIs and Services in isolation.

The tool speeds up unit testing of custom Java APIs, Synchronous Services, User Exit Java code, or any other Java code that exposes a public method with the signature (YFSEnvironment, Document). 

The tool requires:
-	Access to Sterling OMS runtime folder contents
-	Sterling OMS database


---
# Features
-	Does not require Application Server instance to be running Sterling OMS Application.
-	Does not require JMS/MQ Server or an Agent/Integration Server to be running.
-	A custom API/user exit Java class can be tested without including that class in custom jar (eg: yfsextn.jar). (detail on this can be found in notes section at the end of this document)
-	Test runs in isolation. It does not impact other developers using the same Sterling environment.
-	Does not require additional code to be written to unit test the customer API code.

It can be run as:
  - Command Line utility  
  - Lightweight HTTP Server 
---
### Tech

The utility uses NoAppLoader framework provided by Sterling OMS to launch a new JVM to unit test a Sterling API or Service. 

When run in command line mode (from unix/linux/windows command prompt),  it reads input xml from a file and writes out the reply xml in another file or displays it on the screen.

---

### Installation

##### Pre-requisites: 
- Sterling runtime folder must be available on the system where test tool is to be installed.
- Sterling database must be accessible from the system where test tool is being installed (to run the tests after install).

##### Steps:
- Download [tester.tar](https://github.com/jaydeeparikh/SterlingOMSTester/blob/master/tester.tar)
- Linux/Unix - Open a ssh session (putty terminal etc) 

  Windows – Open Command Prompt window.
- Extract the contents of the tar file in your HOME folder (Linux/Unix - $HOME, Windows - %HOMEPATH%)

```sh
Linux/Unix-
$ cd ~
$ tar -xf tester.tar
```
    Windows-
    Use WinZip/7-Zip/WinRAR etc. to extract the tar file contents in %HOMEPATH% folder.
	(Following command on windows command shows HOMEPATH value – 
	echo %HOMEPATH%  )

##### Running the test tool:
The tool can be run in two modes-
-	Command Line mode
-	Server mode


Refer to the steps below for detail on running the tool.

---

### Running Sterling Test tool from Command Line:
-	Linux/Unix - Open a ssh session (putty terminal etc.) 

  	Windows – Open Command Prompt window.

-	Go to “tester” folder under the home folder.
```sh
$ cd  ~/tester		        	- Linux/Unix
C:\> cd %HOMEPATH%\tester		- Windows
```
-	Inside the “tester” folder, run the following script to run a test.
```sh'
$ SterlingTester.sh			- Linux/Unix
C:\> SterlingTester.bat		    - Windows
```
-	Example commands-
```sh
Custom Java API:
$ ./SterlingTester.sh -u=qauser   -p=thepassword    -c=com.xyz.order.api.CustomProcessOrder -m=invoke -i=process_order.xml

OOB API:
$ ./SterlingTester.sh -c=getOrderDetails -k=oob -i=/tmp/go.xml -t=/tmp/t.xml -console -nolog
```

![OOB API](https://github.com/jaydeeparikh/SterlingOMSTester/blob/master/image/example1.jpg)

```sh
Synchronous Service:
./SterlingTester.sh -u=qauser -p=thepassword -c=ConfirmShipmentService -k=service -i=order.xml

Cusom API with arguments (parameters):
./SterlingTester.sh -u=qauser -p=thepassword -c=com.xyz.order.api.CustomProcessShipment -m=invoke -i=process_order.xml -a=LINE_TYPE=GC -a=ORDER_TYPE=WEB
```
Command line arguments supported by the tool are described at the end of this document.

---

### Running Sterling Test tool in Server Mode:

Sterling Test tool can also be run in server mode to run multiple unit tests sequentially or in parallel.
Once the server is running, use an HTTP client such as – browser, curl utility etc. to interact with the server.
Running in Server Mode:

- 	Go to “tester” folder under the home folder.
```sh
	cd  ~/tester			- Linux/Unix

	cd %HOMEPATH%\tester		- Windows
```
- 	Inside the “tester” folder, use the following script to launch the test tool in server mode.

			SterlingTestServer.sh		- Linux/Unix
			
			SterlingTestServer.bat		- Windows
			
```sh
Example commands-
$ ./SterlingTestServer.sh   -nodebug
$ ./SterlingTestServer.sh -port=9000  -timeout=1800000  -nodebug
```
-	By default the server runs on port 9876 (when -port= parameter is not specified while launching server).
- Log is displayed on the screen when -nodebug option is not used while launching the server.

  -timeout= parameter specifies number of milliseconds the user session will stay valid in Sterling OMS. If not specified, the default session timeout is 5 minutes (300000 millis).
  
  -nodebug is an optional parameter, without this flag the API/Service runs in verbose log mode. Log output is displayed on the screen in server mode.

-	Stopping Server-

    Use Ctrl-C to stop the server process.

---

### Launching HTTP Client to interact with test tool running in server mode-

-	Open SterlingTester.html from tester folder, then fill-in and submit the form.

	(“Server DNS/IP & Port” in the screen shot below is the host and port where the test tool is running in server mode.)
	![SterlingTester.html](https://github.com/jaydeeparikh/SterlingOMSTester/blob/master/image/form.jpg)

-	Use curl client-
```sh
$ curl     \
 --data-urlencode "-k=oob" \
 --data-urlencode "-c=getOrderList" \
 --data-urlencode "-u=qauser" \
 --data-urlencode "-p=thePassword" \
 --data-urlencode '-ix=<Order OrderHeaderKey="201906042102331198407"/>' \
 http://localhost:9876
```

Parameters used with curl utility are same as command line parameters (listed at the end of the document), except for the input xml file (-i=) and template xml file (-t=) parameters.

Use following parameters to supply input xml and template xml from curl command.

	-ix=		to supply input xml  (instead of -i=)

	-tx=		to supply template xml  (instead of -t=)

---
### NOTES:

-	Test tool takes approximately 30 seconds to warm up-
		
		-For every test executed in command line mode.
		
		-For the first test executed in server mode.

-	Server mode supports only HTTP protocol (HTTPS is not supported).

-	Server mode is lightweight web server implementation using sockets. It does use any commercial application server or embedded web server library.

-	If the test tool server is not accessible from the browser, adjust the firewall rules on the VM or Antivirus service on Windows to open the server port 9876 (or the port specified with -port= parameter).

    In CentOS environment, use fw.sh script provided in ~/tester folder.
```sh
                   ~/tester/fw.sh 9876
```

-	To test a class by itself without including it in custom jar (eg: yfsextn.jar), copy the class file in “tester” folder in your home directory (~/tester  - Linux/Unix,  %HOMEPATH%\tester – Windows).

    Example:
    If the class name is com.xyz.order.api.CustomProcessOrder
    
	The above example class file is to be copied to-
	
            ~/tester/com/xyz/order/api folder 		    	– Linux/Unix
	    
            %HOMEPATH%\tester\com\xyz\order\api folder 		– Windows

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Command Line mode - after copying the class file, class should recognized by test tool.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Server mode - When using server mode, restart the server after copying the class in tester folder.

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**IMPORTANT:** Remove the custom api java class from tester folder after testing is complete and the class has been included in custom jar (eg: yfsextn.jar).

---

### Command Line arguments:

Parameter |	Description	| Example
----------|-------------|--------
-u=       | A valid Sterling login id. If parameter not used, the SterlingTester.sh script prompts for user id.| -u=qauser
-p=	| Sterling user's password. If parameter not used, the SterlingTester.sh script prompts for password. | -p=thePassword
-c=	| Name of API or Service being tested. |	-c=com.custom.xxx.api.shipment.CustomProcessASNMessage -c=createOrder -c=CustomProcessOrderService
-m=	| Method to run in the class.Only needed when testing a custom api class. | -m=invoke
-i=	| Input xml file name | -i=order.xml
-t=	| Template xml file	| -t=order_template.xml
-k=	| Kind of component being tested. Default is oob (when parameter not specified)	| -k=custom -k=oob -k=service
-nodebug | Optional parameter, without this flag the API/Service runs in verbose log mode | -nodebug
-console | Display output xml on the screen (instead of saving as a file in home folder under tester/output  folder) | -console
-ix= | to supply input xml as string  (instead of -i=) |-ix='<Order OrderHeaderKey=\"201906042102331198407\"/>'
-tx= | to supply template xml as string  (instead of -t=) |-tx='&lt;Order OrderNo=\"\" OrderDate=\"\" DocumentType=\"\"&gt;&lt;OrderLines&gt;&lt;OrderLine PrimeLineNo=\"\" SubLineNo=\"\" OrderedQty=\"\"&gt;&lt;Item ItemID=\"\" UnitOfMeasure=\"\"/&gt;&lt;/OrderLine&gt;&lt;/OrderLines&gt;&lt;/Order&gt;'





