# DME - DIRECT MESSAGING ENGINE
			       
# OVERVIEW
	
DME2 is the second version of the Direct Messaging Engine available under
the AT&T Frameworks and Tools (AFT) umbrella. DME2 will efficiently transport 
JMS (Java Message Service) data using HTTP protocols and without intermediate 
brokers. Additionally, DME2 can be used instead of vendor HTTP stacks or in 
proxy mode. A key benefit of DME2 is that it allows the dynamic routing of
 messages based upon data, business partner or geographic affinity. It also has
capabilities supporting dynamic registration, load balancing and failover for
service endpoints. DME2 can be implemented and utilized by clients, servers,
or both, and it can process messages asynchronously over HTTP and 
websockets.


# REQUIREMENTS

* Java Development Kit (JDK) 1.6 and above
* Java Message Service (JMS) 1.1 (or higher when backwards compatible with 1.1)
* Servlet API 2.5 (or higher when backwards compatible with 2.5)


# BUILD
Checkout code using command:

https://github.com/att/DME.git

* Build scld-config-api project using maven command “mvn clean install”.
* Build dme2-schemas project using maven command “mvn clean install”.
* Build dme3-base project using maven command “mvn clean install”.
* Build dme3-pkg project using maven command “mvn clean install”.
  This project generates dme2.jar.

# RUN
Below is the dependency for DME:
```
<dependency>
    <groupId>com.att.aft</groupId>
    <artifactId>dme2</artifactId>
    <version>3.1.200</version>
</dependency>
```
To implement a Client or Service, you will need to determine which API is most appropriate.
Below are the available API:

1. DME2 - Servlet API
2. DME2 - JMS API
3. DME2 - Websocket API
4. DME2 - JAX-WS Client API


## 1. Servlet API Service Implementation
This section provides steps to implement a standalone DME2 HTTP servlet implementation.
Implement a Custom HTTP Servlet
First, you will need to define a custom HttpServlet extension that adheres to the JEE specification and implements the service method with the required functionality to process the request.  The payload input and output format is not important as long as it is valid HTTP and adheres to the Servlet specification.
In our example, we will override the service(ServletRequest req, ServletResponse resp) method and provide a simple implementation that returns a snippet of HTML.
package examples;
 
import java.io.IOException;
 
import javax.servlet.HttpServlet;
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
``` 
public class ExampleServlet extends HttpServlet {
 
    private String service;
 
    public ExampleServlet(String service) {
        this.service = service;
    }
 
    @Override
    public void service(ServletRequest request, ServletResponse response)
            throws ServletException, IOException {
        // Implement the service method to process request and return response
        response.getWriter().println(
                "<html><title>" +
                                service +
                                "</title><body><h3>Service: " +
                                service +
                                "</h3><br/><h3>Timestamp: " +
                                System.currentTimeMillis() +
                                "</h3></body></html>");
        response.flushBuffer();
        return;
    }
}
```
### Create a Startup Class
Next, we need to implement a startup routine for our service process.  There are many methods this could be done, but in our example we are implementing a simple main() method that invokes the DME2Manager API to register our Servlet dynamically.
package examples;
 ```
import javax.servlet.Servlet;
 
import com.att.aft.dme2.api.DME2Manager;
 
public class RunServer {
    /**
     * @param args
     */
    public static void main(String[] args) throws Exception {
             String serviceName = arg[0];
        Servlet s = new ExampleServlet(serviceName);
        DME2Manager manager = DME2Manager.getInstance();
             manager.bindServiceListener(serviceName, s);
        // keep the JVM running
        while (true) {
            Thread.sleep(5000);
        }
    }
}
```
### Build the Server
To build, you will need to first install DME2. See the main User Guide - DME2 page for how to install the DME2 jar files.
We will not cover how to build Java code here, but your classpath for the build will need to include:
dme2-<version>.jar
servlet-api-2.5.jar
Run the Server
Once you have built your code, you can execute your examples.RunServer.main(String[]) using the following command line (note that this backslash is used to line wrap and should not be included in actual command):
```
java -DAFT_LATITUDE=(run-location-latitude) 
     -DAFT_LONGITUDE=(run-location-longitude) 
     -DAFT_ENVIRONMENT=AFTUAT 
     -classpath <dme2jars>:<compiled-example-classes> examples.RunServer 
service=com.att.IT.MyServiceV1/version=1.0.0/envContext=DEV/routeOffer=BAU_SE
 ```
### DME2 Required Configuration for runtime
In this example, you must set the location and environment used for DME2 to bootstrap. DME2 Server and Client environment both needs to be provided with the below config as either JVM args or via configuration property file:

AFT_LATITUDE
The Earth-latitude of the location of the running OS process being started.

AFT_LONGITUDE
The Earth-longitude of the location of the running OS process being started.

AFT_ENVIRONMENT
The bootstrap environment to use. Normally AFTUAT for nonprod, AFTPRD for production.

### Validating the Server
If the server starts successfully (you will see logs going to standard out indicating whether it started or not) you can immediately use a browser to connect directly to the HTTP URL published in standard output. When DME2 receives the request from the browser it will invoke your servlet and return your coded example response.
 The default contextPath for a DME2 enabled service would be the same as serviceURI. Using the examples, the context path for application would be /service=com.att.IT.MyServiceV1/version=1.0.0/envContext=DEV/routeOffer=BAU_SE. You can override the default contextPath by using DME2ServiceHolder API, that's found in this same page.
You can also use the DME2 "Quick Client" to validate the interface through the registry. This is a simple client that allows you to pass an input message and displays the output to standard out:
```
java -DAFT_LATITUDE=(run-location-latitude) \
     -DAFT_LONGITUDE=(run-location-longitude) \
     -DAFT_ENVIRONMENT=AFTUAT \
     -classpath <dme2jars> com.att.aft.dme2.api.quick.QuickClient \
     -s service=com.att.IT.MyServiceV1/version=1.0.0/envContext=DEV/routeOffer=BAU_SE \
     -m "My Test Message" \
     -t 10000
```
### Creating a Client Application
This section covers creating a simple blocking, simple non-blocking, and async listener client implementations.
DME2Client should be provided URI's with DME2RESOLVE/DME2SEARCH or direct absolute URL's as explained here
Applications intending to use DME2Client for sending request, should ensure to construct a new DME2Client object for each request send.
Simple Blocking Client
The following shows an example client implementation that explicitly blocks for a provided timeout period to receive a response from a server.
```
package examples;
 
public RunClient {
 
    public static void main(String[] args) throws Exception {
         // get inputs
         String service = args[0];
         long timeoutMs = Long.parseLong(args[1]);
 
         // try to call the service just registered
    DME2Client sender = new DME2Client(new URI(service), 30000);
    sender.setPayload("this is a test");
         String response = sender.sendAndWait(timeoutMs);
         System.out.println(response);
         System.exit(0);
    }
}
```
### Simple Blocking - DME2Client StreamPayload
The below example shows how to set a input stream as request payload. This feature is available only from DME2 2.3.15 version onwards .
Assuming that application has to stream the request and expects to receive a text response, creating a DME2StreamPayload and setting that as payload on DME2Client would be required. For a request with stream payload, DME2 will ignore any failover attempts on the same request and return the error, as stream cannot be read twice.
```
public class GWServlet extends HttpServlet {
 public void service(HttpServletRequest req, HttpServletResponse rsp) {
  InputStream ins = null;
  try {
   ins = req.getInputStream();
  } catch (IOException e1) {
   // TODO Auto-generated catch block
   e1.printStackTrace();
  }
  try {
   Properties props = new Properties();
   DME2Manager mgr = new DME2Manager("JettyClient", props);
   String clientUri = "http://DME2RESOLVE/service=com.att.aft.dme2.SpeechRestfulServlet/version=1/envContext=LAB/routeOffer=DEFAULT";
   DME2Client client = new DME2Client(mgr, new URI(clientUri), 10000);
   // Create DME2StreamPayload object with Inputstream
   DME2StreamPayload inPayload = new DME2StreamPayload(ins);
   // set the  inputstream payload on DME2Client
   client.setDME2Payload(inPayload);
   // Add any headers if known based on incoming stream
   client.addHeader("Content-Type", "audio/wav");
   client.addHeader("Transfer-encoding", "chunked");
   // set endpoint subcontext if anything additional to be required
   client.setSubContext("rsservlet");
   String response = client.sendAndWait(30000);
   if(response != null)
    rsp.getWriter().print(response);
  } catch (DME2Exception e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  } catch (URISyntaxException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  } catch (Exception e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
}
```
### Simple Non-Blocking Client
The following shows an example client implementation that does not block waiting for a reply until it is ready. This allows other code to execute during the callout to the server.
```
package examples;
 
public RunClient {
 
    public static void main(String[] args) throws Exception {
         // get inputs
         String service = args[0];
         long timeoutMs = Long.parseLong(args[1]);
 
         // try to call the service just registered
    DME2Client sender = new DME2Client(new URI(service), 30000);
    DME2SimpleReplyHandler replyHandler = new DME2SimpleReplyHandler();
    sender.setReplyHandler(replyHandler);
    sender.setPayload("this is a test");
         sender.send();
 
         // do some other stuff here....
 
 
         // now go back and get response.  still need a timeout
         // in case response hasn't returned yet
         // note this method will throw an exception if there was an issue with communication
         String response = replyHandler.getResponse(timeoutMs);
         System.out.println(response);
         System.exit(0);
    }
}
```
### Asynchronous Listener Client
The following shows an example client implementation that registers a callback reply handler and handles the reply seperately from the originating thread. 
```
package examples;
 
import com.att.aft.dme2.api.DME2ReplyHandler;
 
public ExampleReplyHandler implements DME2ReplyHandler {
    final int MSG_PARSING_BUFFER = 8096;
 
    @Override
    public void handleException(Map<String, String> requestHeaders, Throwable e) {
             System.err.println("Oh no! I got an exception back: " + e.toString());
             e.printStackTrace();
    }
 
    @Override
    public void handleReply(int responseCode, String responseMessage,
            InputStream in, Map<String, String> requestHeaders,
            Map<String, String> responseHeaders) {
             String response = null;
        if (responseCode == 200) {
            StringBuilder output = new StringBuilder(MSG_PARSING_BUFFER);
            try {
                InputStreamReader input = new InputStreamReader(in, "UTF-8");
                final char[] buffer = new char[MSG_PARSING_BUFFER];
                for (int read = input.read(buffer, 0, buffer.length);
                    read != -1;
                    read = input.read(buffer, 0, buffer.length)) {
                    output.append(buffer, 0, read);
                }
                System.out.println("Response: " + output.toString());
            } catch (IOException io) {
                handleException(requestHeaders, io);
            }
        } else {
            System.err.println("Failure returned: ReturnCode=" + responseCode +
                        ", ResponseMessage=" + responseMessage);
        }
    }
 
    public String getResponse(long timeoutMs) throws Exception {
             // we implemented our handling in handleReply, just throw a not implemented exception
             throw new Exception("getResponse not implemented by this ReplyHandler");
    }
}
```
Then, implement a main() to instantiate the reply handler and registering it with the client. Notice this is not exaclty real world since we have to block the main thread while the reply handler waits for a response but it gives a good starting point.
package examples;
 ```
public RunClient {
 
    public static void main(String[] args) throws Exception {
         // get inputs
         String service = args[0];
         long timeoutMs = Long.parseLong(args[1]);
 
         // try to call the service just registered
    DME2Client sender = new DME2Client(new URI(service), 30000);
    DME2ReplyHandler replyHandler = new ExampleReplyHandler();;
    sender.setReplyHandler(replyHandler);
    sender.setPayload("this is a test");
         sender.send();
         // sleep while our background reply handler waits to get its answer
         try {
             Thread.sleep(timeoutMs + 5000);
         } catch (Exception e) {
             //
         }
         System.exit(0);
    }
}
```
### Programatically Configuring DME2
You can override the default DME2 Manager configuration by providing an alternate DME2Manager instance:
```
// Initialize DME2Server and set attributes as required by application
Properties props = new Properties();
props.load("some file");
props.setProperty("AFT_DME2_CONN_IDLE_TIMEOUTMS", "5000");
DME2Manager manager = new DME2Manager("MyCustomManager", props);
 
// if we want to we can also configure some things programatically
DME2Server server = manager.getServer();
server.setPort(45312);
server.setSocketAcceptorThreads(20);
server.setMaxPoolSize(40);
 
// now we can also use this to create our new client
DME2Client client = manager.newClient(uri, connTimeoutMs);
 
// or to bind listner
manager.bindServiceListener(service, servlet);
Setting Custom HTTP/HTTPs Headers
In some scenarios you may need to provide custom HTTP header data. You can do this by using the DME2Client API before calling the send() or sendAndWait() method:
DME2Client client = new DME2Client(service, 30000);
 
// Set the HTTP Method - defaults to POST
sender.setMethod("GET");
 
// Set the HttpRequest headers
Map requestHeaders = new HashMap();
requestHeaders.put("Content-Type","text/plain");
sender.setHeaders(requestHeaders);
 
sender.setPayload("this is a test");
DME2SimpleReplyHandler replyHandler = new DME2SimpleReplyHandler();
sender.setReplyHandler(replyHandler);
sender.send();
 
String reply = replyHandler.getResponse(30000);
 
System.out.println(reply);
Adding QueryParams on client request
For application provider expecting query parameter inputs, DME2Client provides setQueryParams API as shown below
queryParameters as a string
// try to call a service we just registered
String uriStr = "http://DME2SEARCH/service=MyService/version=1.0.0/envContext=PROD/dataContext=205977/partner=TEST";
String queryParams = "queryParam1=1&queryParam2=2";
DME2Client sender = new DME2Client(new URI(uriStr), 30000);
sender.setPayload("this is a test");
com.att.aft.dme2.test.util.EchoReplyHandler replyHandler = new com.att.aft.dme2.test.util.EchoReplyHandler();
sender.setReplyHandler(replyHandler);
// Set the queryParam string on DME2Client
sender.setQueryParams(queryParams);
sender.send();
String reply = replyHandler.getResponse(60000);


 queryParameters as Hashmap input
// try to call a service we just registered
String uriStr = "http://DME2SEARCH/service=MyService/version=1.0.0/envContext=PROD/dataContext=205977/partner=TEST";
HashMap<String,String> queryParams = new HashMap<String,String>();
queryParams.put("queryParam1","1");
queryParams.put("queryParam2","2");
DME2Client sender = new DME2Client(new URI(uriStr), 30000);
sender.setPayload("this is a test");
com.att.aft.dme2.test.util.EchoReplyHandler replyHandler = new com.att.aft.dme2.test.util.EchoReplyHandler();
sender.setReplyHandler(replyHandler);
// Set the queryParam HashMap, the boolean arg is for "encoding" the query string with UTF-8 charset or not
sender.setQueryParams(queryParams,true);
sender.send();
String reply = replyHandler.getResponse(60000);
Enabling ServletFilters, ServletContextListeners for Servlets
If your application has the need to add ServletFilter and ServletContextListeners, you can use DME2ServiceHolder, DME2ServletHolder, DME2FilterHolder API's to enable them as shown in below examples*:*
Adding Servlet Filter using DME2FilterHolder
// Get the default DME2Manager instance if no override is required
 // for default values of Jetty server instance.
 DME2Manager mgr = DME2Manager.getDefaultInstance();
 // Create service holder for each service registration
 DME2ServiceHolder svcHolder = new DME2ServiceHolder();
 svcHolder
   .setServiceURI("service=com.att.aft.ServiceHolderTest/version=1.0.0/envContext=DEV/routeOffer=DEFAULT");
 svcHolder.setManager(mgr);
 // DME2 will register the service with the below root context, also use this as context path published on SOA Cloud registry.
 svcHolder.setContext("/ServiceHolder");
 // Registering multiple servlet's.
 EchoResponseServlet echoServlet = new EchoResponseServlet(
   "com.att.aft.ServletHolderTest", "1");
 // Servlet url mappings/context that can be used to resolve servlets in runtime.
 String urlMappingPattern[] = { "/test", "/servletholder" };
 DME2ServletHolder srvHolder = new DME2ServletHolder(echoServlet,
   urlMappingPattern);
 srvHolder.setContextPath("/servletholdertest");
 EchoResponseServlet echoServlet1 = new EchoResponseServlet(
   "com.att.aft.ServletHolderTest1", "1");
 String urlMappingPattern1[] = { "/test1", "/servletholder1" };
 DME2ServletHolder srvHolder1 = new DME2ServletHolder(echoServlet1,
   urlMappingPattern1);
 srvHolder1.setContextPath("/servletholdertest1");
 // Create a DME2ServletHolder list to add all servlet associated with service grouping.
 List<DME2ServletHolder> shList = new ArrayList<DME2ServletHolder>();
 shList.add(srvHolder);
 shList.add(srvHolder1);
 
 // Adding a Log filter to print incoming msg for ServletHolderTest.
 TestDME2LogFilter filter = new TestDME2LogFilter();
 // Add the dispatcher types on when the request should be processed by filter.
 ArrayList<RequestDispatcherType> dlist = new ArrayList<RequestDispatcherType>();
 dlist.add(DME2FilterHolder.RequestDispatcherType.REQUEST);
 dlist.add(DME2FilterHolder.RequestDispatcherType.FORWARD);
 dlist.add(DME2FilterHolder.RequestDispatcherType.ASYNC); 
 
 // Creating a DME2FilterHolder to filter requests resolved with url pattern /servletholder
 DME2FilterHolder filterHolder = new DME2FilterHolder(filter,
     "/servletholder", EnumSet.copyOf(dlist));
 
 // Create a list to hold the filters for the service grouping
 List<DME2FilterHolder> flist = new ArrayList<DME2FilterHolder>();
 flist.add(filterHolder);
 
 // Set the filter list, servletHolder to DME2ServiceHolder.
 svcHolder.setFilters(flist);
 svcHolder.setServletHolders(shList);
 
 // Start the default server using DME2Manager and bind the serviceHolder
 mgr.getServer().start();
 mgr.bindService(svcHolder);
 
 Thread.sleep(4000);
 // Invokes the above registered service by resolving
 // endpoints via SOA registry
 DME2Client client = new DME2Client(
   new URI(
     "http://DME2RESOLVE/service=com.att.aft.ServiceHolderTest/version=1.0.0/envContext=DEV/routeOffer=DEFAULT"),
   10000);
 client.setPayload("<data>testmessagewithtrademark</data>");
 // The sub context path for the servlet to which request should be sent
 client.setSubContext("/servletholder");
 // Send request and wait for 10 secs.
 String reply = client.sendAndWait(10000);
Adding FilterInitParams using DME2FilterHolder
// Code snippet.
 
 DME2Manager mgr = DME2Manager.getDefaultInstance();
 // Create service holder for each service registration
 DME2ServiceHolder svcHolder = new DME2ServiceHolder();
 svcHolder
   .setServiceURI("service=com.att.aft.ServiceHolderTest/version=1.0.0/envContext=DEV/routeOffer=DEFAULT");
 svcHolder.setManager(mgr);
 
  List<DME2FilterHolder> flist = new ArrayList<DME2FilterHolder>();
  ArrayList<RequestDispatcherType> dlist = new ArrayList<RequestDispatcherType>();
  dlist.add(RequestDispatcherType.REQUEST);
  dlist.add(RequestDispatcherType.FORWARD);
  dlist.add(RequestDispatcherType.ASYNC);
  DME2FilterHolder fholder = new DME2FilterHolder(     new TestInitParamFilter("test"), "/*",     EnumSet.copyOf(dlist));
  // Create a properties object to set filter init params to be supplied.
  Properties initParams = new Properties();
  initParams.setProperty("testFilterParam", "TEST_FILTER_PARAM");
  // Set the filter init params property on DME2FilterHolder as below.
  fholder.setInitParams(initParams);
  flist.add(fholder);
  List<DME2ServletHolder> shList = new ArrayList<DME2ServletHolder>();
  shList.add(srvHolder);
  svcHolder.setFilters(flist);
Adding ServletContextListener
// Create service holder for each service registration
     DME2ServiceHolder svcHolder = new DME2ServiceHolder();
svcHolder
        .setServiceURI("service=com.att.aft.FilterTest/version=1.0.0/envContext=LAB/routeOffer=DEFAULT");
svcHolder.setManager(mgr);
svcHolder
        .setServlet(new EchoResponseServlet(
        "service=com.att.aft.FilterTest/version=1.0.0/envContext=LAB/routeOffer=DEFAULT/",
        "1"));
// If context is set, DME2 will use this for publishing as context with
// endpoint registration, else serviceURI above will be used
svcHolder.setContext("/FilterTest");
// Add Servlet ContextListener list to the service.
 
DME2TestContextListener ctxLsnr = new DME2TestContextListener();
ArrayList<ServletContextListener> clist = new ArrayList<ServletContextListener>();
clist.add(ctxLsnr);
svcHolder.setContextListeners(clist);
Adding Servlet Init Parameters
// Create service holder for each service registration
 DME2ServiceHolder svcHolder = new DME2ServiceHolder();
 svcHolder
   .setServiceURI("service=com.att.aft.ServletInitParamTest/version=1.0.0/envContext=LAB/routeOffer=DEFAULT");
 svcHolder.setManager(mgr);
 svcHolder.setContext("/ServletInitParamTest");
 
 
 EchoServlet echoServlet =new EchoServlet(
   "service=com.att.aft.ServletInitParamTest/version=1.0.0/envContext=LAB/routeOffer=DEFAULT/",  "1");
 String pattern[] = {"/test"};
 DME2ServletHolder srvHolder = new DME2ServletHolder(echoServlet,pattern);
 srvHolder.setContextPath("/ServletInitParamTest");
 Properties params = new Properties();
 params.setProperty("testParam", "TEST_INIT_PARAM");
 srvHolder.setInitParams(params);
 
 List<DME2ServletHolder> shList = new ArrayList<DME2ServletHolder>();
 shList.add(srvHolder);
 
 svcHolder.setServletHolders(shList);
 
       mgr.addService(svcHolder);
       mgr.getServer().start();
Addiing Servlet Context parameter
// Create service holder for each service registration
DME2ServiceHolder svcHolder = new DME2ServiceHolder();
svcHolder
        .setServiceURI("service=com.att.aft.ServletContextParamTest/version=1.0.0/envContext=LAB/routeOffer=DEFAULT");
svcHolder.setManager(mgr);
svcHolder.setContext("/ServletContextParamTest");
 
 
EchoServlet echoServlet =new EchoServlet(
            "service=com.att.aft.ServletContextParamTest/version=1.0.0/envContext=LAB/routeOffer=DEFAULT/",  "1");
String pattern[] = {"/test"};
DME2ServletHolder srvHolder = new DME2ServletHolder(echoServlet,pattern);
srvHolder.setContextPath("/ServletContextParamTest");
Properties params = new Properties();
params.setProperty("testContextParam", "TEST_CONTEXT_PARAM");
srvHolder.setContextParams(params);
 
List<DME2ServletHolder> shList = new ArrayList<DME2ServletHolder>();
shList.add(srvHolder);
 
svcHolder.setServletHolders(shList);
 
mgr.addService(svcHolder);
mgr.getServer().start();
```
## 2. JMS API Service Implementation
### Key Points - DME2JMS
1. DME2 JMS Implementation supports only point to point (PTP) delivery at this time.
2. All messages are in-memory and messages do not live after a JVM dies
3. Messages do not get persisted and by default messages are not queued. By default if there are no available listeners for a server type provider queue, client will be routed to another available endpoint where service has listeners.
4. JMS transactions are not supported in real time, but DME2 mimics transactional support.
5. JMS Client does failover by default if server does not respond in given timeout or does not have enough listeners
6. DME2 only supports text messages as payload
7. Application owners should take care of closing the resources as specified by JMS best practices. Delete TemporaryQueue , Close Session, Close Producer/Consumer, Close Connection objects when finished.
DME2 implementation is based on JMS 1.1 spec. Check the below links for knowing about JMS API requirements and usage.
http://download.oracle.com/otn-pub/jcp/7195-jms-1.1-fr-spec-oth-JSpec/jms-1_1-fr-spec.pdf
 http://docs.oracle.com/javaee/6/api/index.html?javax/jms/package-summary.html
DME2 JMS Service provider process:
Service provider implements a standard JMS Listener interface, configures to use DME2 JMS implementation
On JVM startup, the DME2 API starts a Jetty server on a dynamic port, binds DME2 JMS servlet with a context and publishes the endpoint on the cloud data store
The service provider implementation is registered as listener for the logical DME2 queue and starts listening for requests
The DME2 JMS servlet on request arrival dispatches the request to logical queue and JMS listener gets notified on message arrival.
DME2 Client process:
Application client utilizing standard JMS API, invokes send message to DME2 logical request queue and moves on to receive with a timeout on DME2 logical reply queue
DME2 HTTP Client discovers the published endpoint, sends an async request to DME2 JMS Servlet
DME2 HTTP Client receives the async reply, dispatches the reply message to client reply queue
 
### Setting up DME2 as a standalone JMS Message Listener
Define a standard JMS MessageListener class which implements
javax.jms.MessageListener as below. The onMessage() method implementation is left to service provider's intended use, but the sample below illustrates how a request is received and reply is being returned.
```
import java.lang.management.ManagementFactory;
import java.util.Random;
import java.util.logging.Level;
import java.util.logging.Logger;
 
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageProducer;
import javax.jms.Queue;
import javax.jms.QueueConnection;
import javax.jms.QueueReceiver;
import javax.jms.QueueSession;
import javax.jms.TextMessage;
 
import com.att.aft.dme2.api.util.DME2Constants;
 
public class TestServerListener implements javax.jms.MessageListener {
    private Logger logger = DME2Constants.getLogger(TestServerListener.class.getName());
    private int id = 0;
    private QueueSession session = null;
    private static int counter = 0;
    private Queue serverDest = null;
    private QueueReceiver receiver = null;
    private QueueConnection connection;
    private String PID_DATA;
 
    public TestServerListener(QueueConnection connection, Queue serverDest) {
        this.connection = connection;
        this.serverDest = serverDest;
        id = counter++;
        PID_DATA = ManagementFactory.getRuntimeMXBean().getName();
    }
 
    public void start() throws JMSException {
        session = connection.createQueueSession(true, QueueSession.AUTO_ACKNOWLEDGE);
        receiver = session.createReceiver(serverDest);
        receiver.setMessageListener(this);
    }
 
    public void stop() throws JMSException {
        receiver.setMessageListener(null);
        receiver.close();
        session.close();
    }
 
    private Random random = new Random();
 
    public void onMessage(Message message) {
        try {
            Queue replyTo = (Queue)message.getJMSReplyTo();
            if (replyTo == null) {
                logger.info("AFTJMSSVR.RECOW " + " - [" + id + "] Processed OneWay MessageID="
                        + message.getJMSMessageID() + ", CorrelationID="
                        + message.getJMSCorrelationID());
                return;
            } else {
                Thread.sleep(random.nextInt(2000) + 250);
                TextMessage replyMessage = session
                        .createTextMessage(((TextMessage)message).getText() + "; Receiver: PID@HOST: " + PID_DATA);
                replyMessage.setJMSCorrelationID(message.getJMSMessageID());
                MessageProducer producer = session.createProducer(replyTo);
                producer.send(replyMessage);
                producer.close();
                logger.info("AFTJMSSVR.RECRR " + " - [" + id + "] Processed RequestReply MessageID="
                        + replyMessage.getJMSMessageID() + ", CorrelationID="
                    + replyMessage.getJMSCorrelationID() + ", ReplyTo=" + replyTo.getQueueName());
            }
        } catch (JMSException e) {
            logger.log(Level.WARNING,"[" + id + "] " + e.toString(), e);
        } catch (Exception e) {
            logger.log(Level.WARNING,"[" + id + "] " + e.toString(), e);
        }
    }
 
}
```
Define a startup class that can start required number of listener threads and register the listener as DME2 JMS provider.
Build and start the service provider
```
import java.util.Hashtable;
 
import javax.jms.JMSException;
import javax.jms.Queue;
import javax.jms.QueueConnection;
import javax.jms.QueueConnectionFactory;
import javax.naming.InitialContext;
 
public class TestServer {
 
    private String jndiClass = null;
    private String jndiUrl = null;
    private String serverConn = null;
    private String serverDest = null;
    private int threads = 0;
 
    private QueueConnection connection = null;
 
    public TestServer(String jndiClass, String jndiUrl, String serverConn, String serverDest, int threads) throws Exception {
        this.jndiClass = jndiClass;
        this.jndiUrl = jndiUrl;
        this.serverConn = serverConn;
        this.serverDest = serverDest;
        this.threads = threads;
    }
 
    public void start() throws JMSException, javax.naming.NamingException {
        Hashtable<String,Object> table = new Hashtable<String,Object>();
        table.put("java.naming.factory.initial",
               jndiClass);
        table.put("java.naming.provider.url", jndiUrl);
 
        System.out.println("Getting InitialContext");
        InitialContext context = new InitialContext(table);
 
        System.out.println("Looking up QueueConnectionFactory");
        QueueConnectionFactory qcf = (QueueConnectionFactory) context
                .lookup(serverConn);
 
        System.out.println("Looking up request Queue");
        Queue requestQueue = (Queue) context
                .lookup(serverDest);
 
        System.out.println("Creating QueueConnection");
        connection = qcf.createQueueConnection();
 
        listeners = new TestServerListener[threads];
        for (int i = 0; i < threads; i++) {
            listeners[i] = new TestServerListener(connection, requestQueue);
            listeners[i].start();
        }
    }
 
    private TestServerListener[] listeners = null;
 
    public void stop() throws JMSException {
        if (listeners != null) {
            for (int i = 0; i < threads; i++) {
                listeners[i].stop();
            }
        }
        listeners = null;
        connection.close();
        connection = null;
    }
 
    public static void main(String[] args) throws Exception {
        String jndiClass = null;
        String jndiUrl = null;
        String serverConn = null;
        String serverDest = null;
        String serverThreadsStr = "0";
 
        System.out.println("Starting HttpJMS TestServer");
 
        String usage = "TestServer -jndiClass <jndiClass> -jndiUrl <jndiUrl> -serverConn <url> -serverDest <url> -serverThreads <n>";
 
        for (int i = 0; i < args.length; i++) {
            if ("-jndiClass".equals(args[i])) {
                jndiClass = args[i+1];
            } else if ("-jndiUrl".equals(args[i])) {
                jndiUrl = args[i+1];
            } else if ("-serverConn".equals(args[i])) {
                serverConn = args[i+1];
            } else if ("-serverDest".equals(args[i])) {
                serverDest = args[i+1];
            } else if ("-serverThreads".equals(args[i])) {
                serverThreadsStr = args[i+1];
            } else if ("-?".equals(args[i])) {
                System.out.println(usage);
                System.exit(0);
            }
        }
 
        int serverThreads = Integer.parseInt(serverThreadsStr);
 
        System.out.println("Running with following arguments:");
        System.out.println("    JNDI Provider Class: " + jndiClass);
        System.out.println("    JNDI Provider URL: " + jndiUrl);
        System.out.println("    Server Connection: " + serverConn);
        System.out.println("    Server Destination: " + serverDest);
        System.out.println("    Server Threads: " + serverThreads);
 
        TestServer server = null;
        if (serverThreads > 0) {
            System.out.println("Starting listeners...");
            server = new TestServer(jndiClass, jndiUrl, serverConn, serverDest, serverThreads);
            server.start();
        } else {
            System.out.flush();
            System.err.println("No thread count specified, cannot start server");
            System.exit(1);
        }
 
        while (true) {
            Thread.sleep(5000);
        }
    }
}
```
CLASSPATH requirements:
```
/opt/app/aft/dme2/lib/dme2-<version>.jar
```
### Runtime JVM Requirements:

DME2 needs the below args with appropriate values suited for the environment

-DAFT_LATITUDE=33.6

-DAFT_LONGITUDE=-86.6

-DAFT_ENVIRONMENT=AFTUAT 

### Standard JMS ( or )Application args for the above provided sample
InitialContextFactory

-jndiClass com.att.aft.dme2.jms.DME2JMSInitialContextFactory


ProviderURL


-jndiUrl dme2://


QueueConnectionFactory


-serverConn qcf://dme2

Queue Name

-serverDest http://DME2LOCAL/service=com.att.IT.MyServiceV1/version=1.0.0/envContext=LAB/routeOffer=APPLE_SE

The server destination (-serverDest) as shown above needs to be provided with
fully qualified service name - required,
service version - required,
envContext – required, where the service provider instance is going to be running under and
routeOffer – optional attribute that determines which partner the service might be serving.
Validating the service provider startup
On starting the service provider with above set of configurations, DME2 would publish a dynamic port enabled address entry under the Cassandra directory.

### DME2 JMS Client
```
try{
Hashtable<String, Object> table = new Hashtable<String, Object>();
table.put("java.naming.factory.initial", jndiClass);
table.put("java.naming.provider.url", jndiUrl);
 
System.out.println("Getting InitialContext");
InitialContext context = new InitialContext(table);
 
System.out.println("Looking up QueueConnectionFactory");
QueueConnectionFactory qcf = (QueueConnectionFactory) context
        .lookup(clientConn);
 
System.out.println("Looking up requeust Queue");
Queue requestQueue = (Queue) context.lookup(clientDest);
 
 
System.out.println("Creating QueueConnection");
conn = qcf.createQueueConnection();
 
System.out.println("Creating Session");
session = conn.createQueueSession(true, 0);
 
System.out.println("Creating MessageProducer");
sender = session.createSender(requestQueue);
 
// Creating a tempQueue based on input
if(useTempQueue) {
     replyToQueue = session.createTemporaryQueue();
}
else {
    System.out.println("Looking up reply Queue");
    replyToQueue = (Queue) context.lookup(clientReplyTo);
 
}
TextMessage message = session.createTextMessage();
String dataContext = System.getProperty("dataContext");
if(dataContext!=null) {
    message.setObjectProperty("com.att.aft.dme2.jms.dataContext",
    dataContext);
}
else {
message.setObjectProperty("com.att.aft.dme2.jms.dataContext",
        "205977");
}
String partner = System.getProperty("partner");
if(partner!= null) {
message.setStringProperty("com.att.aft.dme2.jms.partner",
        partner);
}
else {
    message.setStringProperty("com.att.aft.dme2.jms.partner",
    "1C");
}
String selKey = System.getProperty("selKey");
if(selKey != null) {
    message.setStringProperty("com.att.aft.dme2.jms.stickySelectorKey",selKey);
}
String sentID = ID + "::" + System.currentTimeMillis();
 
String msgText = "this is my message";
 
 
message.setText(sentID + msgText);
long start = System.currentTimeMillis();
sentCounter++;
message.setJMSReplyTo(replyToQueue);
// send message
  sender.send(message);
// Create receiver for receiving the reply.
  QueueReceiver consumer = session.createReceiver(replyToQueue,
    "JMSCorrelationID = '" + message.getJMSMessageID() + "'");
// Invoke receive with a timeout for how long to wait on reply
  TextMessage response = (TextMessage) consumer.receive(60000);
 
 long elapsed = System.currentTimeMillis() - start;
if (response == null) {
    logger.info("AFTJMSCLT.TIMEOUT - [" + ID
            + "] TIMEOUT after waiting " + elapsed
            + " for RequestReply MessageID="
            + message.getJMSMessageID() + ", ReplyTo="
            + this.clientReplyTo);
    timeoutCounter++;
} else {
    logger.log(Level.FINE,"AFTJMSCLT.DATA - Expecting back [" + sentID
            + "], got [" + response.getText() + "]");
    if (response.getText().startsWith(sentID)) {
        logger.info("AFTJMSCLT.SUCCESS - [" + ID
                + "] SUCCESS, RESPONSE [" + response.getText()
                + "], ELAPSED [" + elapsed
                + "], Request MessageID ["
                + response.getJMSMessageID()
                + "], CorrelationID " + "["
                + response.getJMSCorrelationID()
                + "], ReplyTo [" + this.clientReplyTo + "]");
        successCounter++;
    } else {
        logger.info("AFTJMSCLT.MISMATCH - [" + ID
                + "] MISMATCH response after " + elapsed
                + " for RequestReply MessageID="
                + response.getJMSMessageID()
                + ", CorrelationID="
                + response.getJMSCorrelationID() + ", ReplyTo="
                + this.clientReplyTo);
        mismatchCounter++;
    }
}
// Catch Exceptions and deal according to your application requirements
catch (Exception e) {
    logger.log(Level.SEVERE,"AFTJMSCLT.FATAL - [" + ID + "] FATAL response "
      + e.toString(), e);
    failCounter++;
    return;
}
finally {
    // Close all open JMS resources like sender, session, tempqueue, connections once finished
    if (conn != null) {
     try {
      sender.close();
     } catch(Exception e) {
      
     }
     // Delete the temporary queue
     if (useTempQueue) {
      try {
       ((TemporaryQueue) replyToQueue).delete();
      } catch (Exception e) {
      }
     }
     // Close the session that's not in use.
     try {
       session.close();
     } catch (Exception e1) {
     }
     
     try {
      conn.close();
     } catch (Exception e2) {
     }
     sender = null;
     session = null;
     conn = null;
    }  
   }
```
Application args for the above provided client sample
-jndiClass= com.att.aft.dme2.jms.DME2JMSInitialContextFactory

-jndiUrl =dme2://

-conn =qcf://dme2

-dest= http://DME2SEARCH/service=com.att.IT.MyServiceV1/version=1.0.0/envContext=LAB/partner=APPLE

-replyTo= http://DME2LOCAL/clientResponseQueue

## 3. Websocket API Service Implementation
```
package com.att.aft.dme2.api;
 
import java.io.IOException;
import org.eclipse.jetty.io.Buffer;
import org.eclipse.jetty.io.ByteArrayBuffer;
import com.att.aft.dme2.api.websocket.DME2ServerWSConnection;
import com.att.aft.dme2.api.websocket.DME2ServerWebSocketHandler;
 
public class TestDME2ServerWebSocketHandler  extends DME2ServerWebSocketHandler{
    @Override
    public void onOpen(DME2ServerWSConnection dme2wsConnection) {
     
        System.out.println("Connection opened : {}" + dme2wsConnection);
                 
    }
     
    @Override
    public void onMessage( String data) {
        // TODO Auto-generated method stub
        System.out.println("I am in onMessage() in handler");
        try
        {
            // echo back this TEXT message
            this.getDme2wsConnection().sendMessage("I am fine. What can I do for you?");
 
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } finally {
             
        }      
    }
     
    @Override
    public void onClose(int closeCode,
            String message) {
           System.out.println("Closed {} : {}" + closeCode + message);        
    }
 
    @Override
    public void onMessage(byte[] data, int offset, int length) {
        // Wrap in Jetty Buffer (only for logging reasons)
        Buffer buf = new ByteArrayBuffer(data,offset,length);
        System.out.println("on Text : {}" + buf.toDetailString());
        try {
            String msg = "I am fine. What can I do for you?";
            this.getDme2wsConnection().sendMessage(msg.getBytes(),0, msg.length());
         } catch (IOException e) {
           e.printStackTrace();
        }      
    }
     
}
```
### Create a websocket Server
//Example1:
```
package examples;
import java.util.Properties;
import com.att.aft.dme2.test.util.RegistryGrmSetup;
public class RunServer {
    public static void main(String args[]) throws Exception {
        DME2Manager mgr = null;
        String service = "/service=com.att.aft.dme2.test.TestDME2ServerDME2WebSocket/version=1.0.0/envContext=LAB/routeOffer=DME2_PRIMARY?DME2WebSocket=true";
        try {
            System.setProperty("SCLD_PLATFORM", "SANDBOX-LAB");
            Properties props = RegistryGrmSetup.init();
            props.setProperty("AFT_DME2_PORT", "30316");
            props.setProperty("AFT_DME2_EP_READ_TIMEOUT_MS", "20000");
            mgr = new DME2Manager("TestDME2WebSocketServer", props);
            mgr.bindServiceListener(service,
                    new TestDME2ServerWebSocketHandler());
            while (true) {
                Thread.sleep(100 * 1000);// 100 sec
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            mgr.unbindServiceListener(service);
        }
    }
}
```  
### // Example2:
```
package examples;
import java.util.Properties;
import com.att.aft.dme2.test.util.RegistryGrmSetup;
public class RunServer {
     
    public static void main(String args[]) throws Exception{               
        DME2Manager mgr = null;
        String service = "/service=com.att.aft.dme2.test.TestDME2ServerDME2ServiceHolderWay/version=1.0.0/envContext=LAB/routeOffer=DME2_PRIMARY?DME2WebSocket=true";
        System.setProperty("SCLD_PLATFORM", "SANDBOX-LAB");
        try {
            Properties props = RegistryGrmSetup.init();
            props.setProperty("AFT_DME2_PORT", "30320");
            props.setProperty("AFT_DME2_EP_READ_TIMEOUT_MS", "20000");
            mgr = new DME2Manager("TestDME2WebSocketServer", props);
             
            DME2ServiceHolder svcHolder = new DME2ServiceHolder();
            svcHolder.setManager(mgr);
            svcHolder.setContext("/DME2ServerWebSocket");
            svcHolder.setDme2WebSocketHandler(new TestDME2ServerWebSocketHandler());
            svcHolder.setServiceURI(service);
 
            mgr.bindService(svcHolder);
             
            while(true){
                Thread.sleep(100 * 1000);// 100 sec
            }          
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
             mgr.unbindServiceListener(service);
        }
    }
}
```
### Build the Server
To build, you will need to first install DME2. See the main User Guide - DME2 page for how to install the DME2 jar files.
We will not cover how to build Java code here, but your classpath for the build will need to include:
dme2-<version>.jar
servlet-api-2.5.jar

### Run the Server
java -DAFT_LATITUDE=(run-location-latitude) 
-DAFT_LONGITUDE=(run-location-longitude) 
-DAFT_ENVIRONMENT=AFTUAT 
-classpath <dme2jars>:<compiled-example-classes> examples.RunServer

### DME2 Required Configuration for runtime
In this example, you must set the location and environment used for DME2 to bootstrap. DME2 Server and Client environment both needs to be provided with the below config as either JVM args or via configuration property file:
Property
Description
AFT_ENVIRONMENT
The bootstrap environment to use. Normally AFTUAT for nonprod, AFTPRD for production.
AFT_LATITUDE
The Earth-latitude of the location of the running OS process being started.
AFT_LONGITUDE
The Earth-longitude of the location of the running OS process being started.
Validating the Server
Validating the Server
If the server starts successfully (you will see logs going to standard out indicating whether it started or not) you can immediately use a Simple websocket like chrome browser extension and start sending messages to the websocket, and receive messages as well if the handler attached to the server returns messages.
The default contextPath for a DME2 enabled service would be the same as serviceURI. Using the examples, the context path for application would be /service=com.att.aft.dme2.test.TestDME2ServerDME2ServiceHolderWay/version=1.0.0/envContext=LAB/routeOffer=DME2_PRIMARY. 
So your websocket url would look like ws://localhost:30320/service=com.att.aft.dme2.test.TestDME2ServerDME2ServiceHolderWay/version=1.0.0/envContext=LAB/routeOffer=DME2_PRIMARY/
You can override the default contextPath by using DME2ServiceHolder API.

### Creating a Client Application
This section covers creating a simple wesocket client implementation.
DME2Client should be provided URI's with DME2RESOLVE/DME2SEARCH or direct absolute URL's as explained here
Applications intending to use DME2Client for sending request, should ensure to construct a new DME2Client object for each request send.
Simple Client
The following shows an example client implementation that creates a client handler to handle the websocket communication.The client registers a callback message handler and handles the conversation seperately from the originating thread. 
Example 1:
``` 
package examples;
 
import com.att.aft.dme2.api.websocket.DME2WSCliConnection;
import com.att.aft.dme2.api.websocket.DME2WSCliTextMessageHandler;
 
public class EchoTextMessageHandler extends DME2WSCliTextMessageHandler {
    public String receivedMsg;
    public String exceptionMessage = "";
    @Override
    public void processTextMessage(String message) {
        System.out.println("Received message " + message);
        receivedMsg = message;
    }
    @Override
    public void onConnClose(int closeCode, String message) {
        System.out.println("Connection closed with closeCode: " + closeCode);
    }
    @Override
    public void onOpen(DME2WSCliConnection conn) {
        System.out.println("Connection Open,");      
    }
    @Override
    public void onException(Exception e) {
        this.exceptionMessage = e.getMessage();
    }
}
 ```
Example2:
```
public class EchoBinaryMessageHandler extends DME2WSCliBinaryMessageHandler {  
    public byte[] receivedMsg;
    public final byte[] waiter = new byte[0];
    public int closeCode = 0;
    @Override
    public void onConnClose(int closeCode, String message) {
        this.closeCode = closeCode;
    }
    @Override
    public void processBinaryMessage(byte[] data, int offset, int length) {
//      System.out.println("Received message " + data);
        receivedMsg = data;    
        synchronized (this.waiter) {
            waiter.notify();
        }
    }
    @Override
    public void onOpen(DME2WSCliConnection conn) {
        // TODO Auto-generated method stub
         
    }
    @Override
    public void onException(Exception e) {
        // TODO Auto-generated method stub
         
    }
 
}
```
Then, implement a main() to instantiate the message handler and registering it with the client. 
```
package examples;
 
public RunClient {
 
    public static void main(String[] args) throws Exception {
            String serviceName = "/service=com.att.aft.dme2.ws.test.TestDME2SendTextMessageWithDirectURI/version=1.0.0/envContext=LAB/routeOffer=DME2_PRIMARY";
        try {
            Properties props = new Properties();
            DME2Manager mgr = new DME2Manager("TestDME2ClientWithDME2Protocol_DirectURI", props);
 
            String clientURI = String.format("ws://%s:%s%s%s",  InetAddress.getLocalHost().getCanonicalHostName(), "30906", serviceName, "?                        connecttimeoutinms=30000&wsConnIdleTimeout=30000");
             
            EchoTextMessageHandler msgHandler = new EchoTextMessageHandler();
             
            DME2Client sender = new DME2Client(mgr, new URI(clientURI), msgHandler, 5000, 5000);
             
            Thread t = new Thread()
            {
                @Override
                public void run() {
                    try {
                        Thread.sleep(5000);
                        EchoTextMessageHandler textHandler = (EchoTextMessageHandler ) msgHandler;
                        textHandler.sendTextMessage("How are you?");
                        System.out.println("Message sent");
                        Thread.sleep(2000);
                        textHandler.closeConnection();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            };
             
            try{
                sender.startWsConnection();    
                t.start();
                Thread.sleep(15000);
            } catch(Exception e){
                e.printStackTrace();
            } finally {
                sender.closeWsConnection();
            }  
        } finally {
            try {
                mgr.stop();
            } catch (Exception e) {
                //....
            }      
        }
            System.exit(0);
    }
}
``` 
 
Example2:
```
import java.io.File;
import java.io.FileInputStream;
import java.net.URI;
import java.util.Properties;
import com.att.aft.dme2.api.DME2Client;
import com.att.aft.dme2.api.DME2Manager;
public class TestApp {
    public static void main(String[] args) throws Exception {
        DME2Client sender = null;
        String msgFile = args[0]; //binary data file
        DME2Manager mgr = null;
         
        try {
            Properties props = new Properties();
            System.setProperty("AFT_ENVIRONMENT", "AFTUAT");
            System.setProperty("AFT_LATITUDE", "33.373900");
            System.setProperty("AFT_LONGITUDE", "-86.798300");
            System.setProperty("SCLD_PLATFORM", "SANDBOX-LAB");
            mgr = new DME2Manager("TestDME2ClientWithDME2Protocol_DirectURI", props);
             
            String serviceName = "/service=com.att.aft.dme2.ws.test.testServer/version=1.0.0/envContext=LAB/routeOffer=DME2_PRIMARY/partner=DME2_PARTNER";
            String clientURI = String.format("ws://DME2SEARCH%s%s", serviceName,
                    "?connecttimeoutinms=240000&wsConnIdleTimeout=240000");
             
            TestBinaryMessageHandler msgHandler = new TestBinaryMessageHandler();
             
            sender = new DME2Client(mgr, new URI(clientURI), msgHandler, 32768, 32768);
            FileInputStream fis = null;
             
            try {
                sender.startWsConnection();
                fis = new FileInputStream(new File(msgFile));
                byte[] fileData = new byte[(int) msgFile.length()];
                fis.read(fileData);
                msgHandler.sendBinaryMessage(fileData, 0, fileData.length );
                System.out.println("Message sent from client");
            } catch (Exception e) {
               e.printStackTrace();
            } finally {
               if (fis != null)
                   fis.close();
               sender.closeWsConnection();
            }
        } finally {
            try {
            mgr.stop();
            } catch (Exception e) {
            //....
            }      
        }
        System.exit(0);
    }
} 
```
### Programatically Configuring DME2 Client
You can override the default DME2 Manager configuration by providing an alternate DME2Manager instance:
// Initialize DME2Server and set attributes as required by application
```
Properties props1 =  new Properties();
props1.setProperty("AFT_DME2_PORT", "30316");
props1.setProperty("AFT_DME2_CLIENT_IGNORE_SSL_CONFIG", "false");
props1.setProperty("AFT_DME2_CLIENT_KEYSTORE", "xxxxx.jks");
props1.setProperty("AFT_DME2_CLIENT_KEY_PASSWORD", "xxxxxxx");
props1.setProperty("AFT_DME2_CLIENT_KEYSTORE_PASSWORD", "xxxxxxx");
// now try to call our https service   
clientManager = new DME2Manager("testSSLEnabledClient", props1);
EchoTextMessageHandler msgHandler = new EchoTextMessageHandler();  
// now we can also use this to create our new client
DME2Client sender = new DME2Client(mgr, new URI(clientURI), msgHandler, 5000, 0);
Adding QueryParams on client request
For application provider expecting query parameter inputs, DME2Client provides setQueryParams API as shown below
queryParameters as a string
// try to call a service we just registered
String uriStr = "http://DME2SEARCH/service=MyService/version=1.0.0/envContext=PROD/dataContext=205977/partner=TEST";
String queryParams = "queryParam1=1&queryParam2=2";
EchoTextMessageHandler msgHandler = new EchoTextMessageHandler();  
DME2Client sender = new DME2Client(mgr, new URI(uriStr ), msgHandler, 5000);
sender.setQueryParams(queryParams);
sender.startWsConnection();    
Thread.wait(MAX_CONVERSATION_TIME);

 queryParameters as Hashmap input on client 
// try to call a service we just registered
String uriStr = "http://DME2SEARCH/service=MyService/version=1.0.0/envContext=PROD/dataContext=205977/partner=TEST";
HashMap<String,String> queryParams = new HashMap<String,String>();
queryParams.put("queryParam1","1");
queryParams.put("queryParam2","2");
EchoTextMessageHandler msgHandler = new EchoTextMessageHandler();  
DME2Client sender = new DME2Client(mgr, new URI(uriStr ), msgHandler, 5000);
sender.setQueryParams(queryParams);
sender.startWsConnection();    
Thread.wait(MAX_CONVERSATION_TIME);
```
### ServiceURI for websocket client
The following name value pairs can be set in a websocket serviceURI…
Service
envContext
routeOffer
Version
preferredRouteOffer
partner
stickySelectorKey
Example: http://DME2SEARCH/service=com.att.aft.dme2.ws.test.TestEndpointStalenessInMin/version=1.0.0/envContext=LAB/partner=DME2_PARTNER?connecttimeoutinms=5000&wsConnIdleTimeout=5000

### Supported message types in Websocket client
Currently websocket client implementation supports text and binary message handling. The following base classes can be extended to implement the text and binary message handlers.
DME2WSCliTextMessageHandler
DME2WSCliBinaryMessageHandler

### Websocket Connection Identification 
Tracking_id is the key field that tracks the life cycle of a websocket communication on both client and server side (when dme2 is used to create client service). Note that the trackingId is sent as a queryparam in the endpoint url. A non dme2 websocket service can access this param and track the messages on the server end

## 4. DME2 JAX-WS CLIENT SERVICE IMPLEMENTATION
The DME2 JAX-WS Client API allows you to quickly integrate a JAX-WS client implementation with DME2. This allows you to fully utlize a JAX-WS implementation for message/xml marshalling and utilize DME2 for the underlying connectivity capablities.
### Configuring the Handler
To utlize, you need to add the com.att.aft.dme2.jaxw.client.DME2SOAPHandler class as the LAST handler in your client's handler chain. Consult the JAX-WS implementation documentation for instructions on how to add client-side handlers to a handler chain.

### Setting the Service URI
For most providers, you can simply set the BindingProvider.ENDPOINT_ADDRESS_PROPERTY to that of the DME2 service URI you are building a client for. In some cases, however, the provider may have a check to validate the URI is a valid network URL. In these cases, you will likely need to utilize a dummy URL and instead specify a MessageContext property of "com.att.aft.dme2.jaxw.client.uri" in order to set the endpoint.

### Additional Context Properties
In addition to the URI, you can set the following context properties:
com.att.aft.dme2.jaxws.client.connTimeoutMs
Timeout to use for each network connection attempt
com.att.aft.dme2.jaxws.client.readTimeoutMs
Timeout to use when waiting for a response from each endpoint

### Limitations
Currently the JAX-WS handler doesn't let you do a lot of other configurable options using the MessageContext. You can only configure DME2 via JVM or aft.properties and they will apply to the entire JVM. Future updates of this plugin will likely allow setup of custom DME2Manager's for each JAX-WS Handler instance so you can customize by client call.

# CONFIGURATION

Application properties are present in dme2-api/src/main/resources/dme-api_defaultConfigs.properties.

###DME2 Configuration Methods
DME2 has a flexiable configuration mechanism that allows configurations to be provided using two different methods.
Configuration by Manager Initialization
This method is the prefered model and allows multiple DME2Manager/DME2JMSManager instances to be instantiated in a single JVM with different configuration properties.
Configuring the JMS APIs
When using the JMS APIs you will not have direct access to the DME2 implementation classes. Instead, you will provide the property information to the InitialContext initialization code when acquiring the javax.jms.ConnectionFactory object.
This general technique can be used to configure DME2 JMS Providers in various containers (WebLogic, Websphere, Spring, etc).
```
Hashtable<String,Object> table = new Hashtable<String,Object>();
table.put("java.naming.factory.initial", jndiClass);
table.put("java.naming.provider.url", jndiUrl);
 
// set a custom manager name - otherwise it'll use the default manager
// this name should be unique in the JVM
table.put("AFT_DME2_MANAGER", "MyDME2Manager");
 
// example of setting an alternate max request dispatcher queue size
table.put("AFT_DME2_MAX_POOL_SIZE", "100");
 
// initialize the initial context object
InitialContext context = new InitialContext(table);
 
// continue with standard JMS code...
QueueConnectionFactory qcf = (QueueConnectionFactory) context.lookup(serverConn);
Configuring the Non-JMS APIs
To configure DME2Manager during initialization, you simply pass it a java.util.Properties object with the properties you wish to set during creation:
import java.util.Properties;
 
import com.att.aft.dme2.DME2Manager;
 
...
 
Properties properties = new Properties();
 
// load from file or set properties in code...\
 
// example of setting an alternate max request dispatcher queue size
properties.setProperty("AFT_DME2_MAX_POOL_SIZE", "100");
 
// initialize a manager instance with a custom name.
// This name should be unique in the JVM.
DME2Manager manager = new DME2Manager("MyDME2Manager", properties);
 
// this manager is thread safe - save it and utilize it for all interactions
// requiring the configuration provided through these properties
Configuring the Websocket APIs
To configure DME2Manager during initialization, you simply pass it a java.util.Properties object with the properties you wish to set during creation:
import java.util.Properties;
import com.att.aft.dme2.DME2Manager;
 
// example of setting an alternate keystore and other params
properties.setProperty("AFT_DME2_MAX_POOL_SIZE", "100");
props.setProperty("AFT_DME2_SSL_ENABLE", "true");
props.setProperty("AFT_DME2_KEYSTORE", "xxxx.jks");
props.setProperty("AFT_DME2_KEYSTORE_PASSWORD", "xxxxxx");
props.setProperty("AFT_DME2_DEF_WS_IDLE_TIMEOUT", "5000");
props.setProperty("AFT_DME2_WS_MAX_RETRY_COUNT", "2");
 
// initialize a manager instance with a custom name.
// This name should be unique in the JVM.
DME2Manager manager = new DME2Manager("MyDME2Manager", properties);
``` 
// this manager is thread safe - save it and utilize it for all interactions
// requiring the configuration provided through these properties
### JVM-Wide External Configuration File
This method allows a properties to be specified in a JVM-wide manner, either via the aft.properties configuration file, an OS environment variable, or by using a JVM System Property. In this method, the configuration file, OS environnment variables, or JVM System Properties are used automatically for any DME2Manager or DME2JMSManager instance created within the JVM. These properties will override any property provided directly to the DME2Manager via one of the code-based methods above.
 ```
# set an alternate default aft.properties during startup.
# If not provided, DME2 will always look for etc/aft.properties.
java -DAFT_CONFIG=/some/file.properties ....
 ```
 ```
# set OS environment variables.  This will not override anything
# specified in aft.properties, but will supercede anything provided
# via the code initilization methods listed above or the System Properties
# method mentioned below.
export AFT_DME2_MAX_POOL_SIZE=100
export AFT_DME2_GRM_USER=XXXXX
export AFT_DME2_GRM_PASS=XXXXX
java ....
```
```
# set explicit properties as JVM arguments.
# This will not override anything specified in aft.properties,
# but will supercede anything provided via the code initilization methods listed above.
java -DAFT_DME2_MAX_POOL_SIZE=100 -DAFT_DME2_GRM_USER=XXXXX -DAFT_DME2_GRM_PASS=XXXXXX ....
```
