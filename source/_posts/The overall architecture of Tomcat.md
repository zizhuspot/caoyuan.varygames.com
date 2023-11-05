---
title: The overall architecture of Tomcat
date: 2023-11-05 00:09:00
categories: 
  - Technology
tags: 
  - Tomcat
  - Understanding
  - architecture
  - essential
  - introduced
  - technical
  - finally
  - Tomcat
description: Understanding the overall architecture of Tomcat can be learned from the working principle of Tomcat, is an essential part of learning Tomcat. The text will introduce the core components of Tomcat, and then finally introduced from the source code level how to put Tomcat process.
cover: https://s2.loli.net/2023/11/05/2lxm4cRIXf3usqh.png
---
# Tomcat overall architecture

Understanding the overall architecture of Tomcat is an essential part of learning how Tomcat works. The text will introduce the core components of Tomcat, and then finally introduce the source code level how to link Tomcat access to the Servlet process.

## 1. General Architecture

Tomcat has to fulfill 2 core functions:

1. handling socket connections, responsible for the conversion of network byte streams to Request and Response objects.
2. Load and manage Servlets, as well as specific processing Request requests.

Therefore, Tomcat designed two core components: connector (Connector) and container (Container) to deal with these two things. Connector is responsible for external communication , the container is responsible for internal processing .

Tocamt supports the following I/O models:

* NIO: non-blocking IO, implemented using the Java NIO class library .
* NIO2: asynchronous IO, using JDK7 latest NIO2 class library implementation.
* APR: implemented using the Apache portable runtime library, a native library written in C/C++.

Tomcat supports the application layer protocols are:

* Http/1.1: the access protocol used by most web applications.
* AJP: for integration with web servers (eg: Apache)
* HTTP/2: Http2.0 protocol substantially improves the performance of the Web.

Tomcat in order to support a variety of IO models and application layer protocols , a container may dock multiple connectors . But a single connector or container can not provide services to the outside world , they need to be combined to work , assembled as a whole is called Service component . Service is just outside the connector and the container more than one layer of the package , put them together . Tomcat may have more than one Service, so the design is also out of flexibility considerations. The relationship diagram is as follows:

![](https://s2.loli.net/2023/11/05/6o5bzxmNQq3DMJg.webp)

As you can see from the diagram:

* The top level is the Server, which in this case is also an instance of Tomcat.
* A Server has one or more Services, and a Service has multiple connectors and a container.
* Connectors and containers communicate with each other through the standard ServletRequest and ServletResponse.

## 2. connectors

Connector to the Servlet container shielding protocols and I/O model and other differences, whether HTTP or AJP, from the container always get a standard ServletRequest object.

Connector roughly to do 3 highly cohesive functions:

* Network communication.
* Application layer protocol parsing.
* Tomcat Request/Response and ServletRequest/ServletResponse conversion.

Corresponding Tomcat designed three components to achieve these three functions, respectively, Endpoint, Processor and Adapter. overall logic: EndPoint is responsible for providing byte streams to the Processor, Processor is responsible for providing Tomcat Request to the Adapter, Adapter is responsible for Provide ServletRequest object to the container.

As the I/O model and application protocols can be freely combined , such as NIO + HTTP or NIO2 + AJP. Tomcat will be the network communication and application layer protocol parsing considered together , designed an interface called ProtocolHandler to encapsulate these two changes. Specific combinations are: Http11NIOProtocol and AjpNioProtocol. of course, also designed a series of abstract base classes to encapsulate the generic part, such as: AbstractProtocol implements the ProtocolHandler interface, the entire inheritance relationship is as follows:

![](https://s2.loli.net/2023/11/05/LBDU4JxKs8hCGHN.webp)

To summarize: the three core components of the connector: EndPoint, Processor and Adapter to do three things, of which EndPoint and Processor together into an abstract PortocolHandler component, the relationship is shown in the following figure

![](https://s2.loli.net/2023/11/05/XtnJc2679eaz3LQ.webp)

### 2.1 ProtocolHandler Components

ProtocolHandler internally includes EndPoint and Processor, the following describes how they work:

**EndPoint**: communication endpoint, is a concrete Socket receive and send processor, is an abstraction of the transport layer. Therefore EndPoint is used to implement the TCP/IP protocol.EndPoint is an interface, AbstractEndPoint is its abstract implementation of the class, the abstract class specific subclasses, there are two important sub-components: Acceptor and SocketProcessor.

Acceptor is used to listen for socket connection requests. socketProcessor is used to process incoming socket requests, implements the Runnable interface, and calls the Processor in the run method for processing. In order to increase the processing power, SocketProcessor is submitted to a thread pool for execution. This thread pool is Executor.

**Processor**: If EndPoint is used to implement the TCP/IP protocol, then Processor is used to implement the HTTP protocol.Processor receives the Socket from Endpoint, reads the byte stream and parses it into Tomcat Requst and Response objects, and submits it to the container for processing via the Adapter will be submitted to the container processing , Processor is an abstraction of the application layer protocol . Processor is an interface that defines the processing of requests and other methods , its abstract implementation class AbstactProcessor encapsulates some of the properties common to the protocol. Specific implementations include : HTTP11Processor, which implements protocol-specific parsing methods and request processing.

The detailed component relationships in the connector are as follows: ![](https://s2.loli.net/2023/11/04/UM4t8egaPXIKZ7Q.webp)

### 2.2 Adapter

Because of the different protocols, the client sends different request information, so Tomcat defined its own Request class to "store" the request information. protocolHandler interface is responsible for that is the system request and generate TomcatRequest class. But this is not a standard HttpServletRequest, means that you can not use Tomcat Request as a parameter to invoke the container , Tomcat designer's solution is to introduce the CoyoteAdapter, which is a classic use of the adapter pattern , the connector calls the CoyoteAdapter service method , pass in the Tomcat Request object, converted to a ServletRequest object, in the call container service method.

## 3. Container

## 3.1 Hierarchy of containers

Tomcat has designed four types of containers, namely Engine, Host, Context and Wrapper, which are not parallel, but parent-child relationships. The schematic is as follows:

![](https://s2.loli.net/2023/11/05/cHz2RjfgN98VpUT.webp)

The reason why it is designed into so many layers is that Tomcat has a layered architecture, and the Servlet container is very flexible.

* Context represents a Web application; Wrapper represents a Servlet, a Web application may have more than one Servlet.
* Host represents a virtual host or a site , you can configure Tomcat multiple virtual host address , and a virtual host can be deployed under multiple Web applications .
* Engine represents the engine, used to manage multiple virtual sites, a Service can only have a maximum of one Engine.

You can use Tomcat's server.xml configuration file to deepen your understanding of the Tomcat container.Tomcat uses a componentized design, which consists of components that are configurable, the outermost layer is the Server, and other components are configured in this top-level container according to certain format requirements.

```java

```

How does Tomcat manage these containers? It is managed through the combination pattern. Specific implementation: all container components implement the Container interface , so the combination pattern can make the user but the container object and the combination of container objects have consistency in the use of container objects . Here the single container object refers to the bottom of the Wrapper, the combination of container objects refers to the top of the Context, Host or Engine. Container interface is defined as follows:

```java
public interface Container extends Lifecycle {
    public void setName(String name);
    public Container getParent();
    public void setParent(Container container);
    public void addChild(Container child);
    public void removeChild(Container child);
    public Container findChild(String name);
}
```

You can see the methods getParent, setParent, addChild and removeChild. It also implements the LiftCycle interface, which is used to centrally manage the life cycle of each component.

### 3.2 The request-localization servlet process

Designed so many levels of containers, Tomcat how to determine which request is handled by the Wrapper container Servlet, Tomcat uses the Mapper component to accomplish this task.

The function of the Mapper component is to locate the user request URL to a Servlet, it works: Mapper component saves the configuration information of the Web application, in fact, it is ** container components and access path mapping relationship **. For example, the Host container configuration of the domain name, the Context container of the Web application path and the Wrapper container Servlet mapping path.

When a request arrives, the Mapper component locates a Servlet by parsing the domain name and path in the request URL and looking in its own saved Map. ** A request URL will end up locating only one Wrapper container, which is a Servlet**.

An example of the process of how to locate. Suppose there are two systems running under the same Tomcat, configured with two virtual domains: manage.shopping.com is used to manage users and products, there are two internal Web applications; user.shopping.com is used for end-customers to search for products and purchase, there are also two internal Web applications.

For such a deployment, Tomcat will create a Service component, an Engine container component, two Host subcontainers, and two Context subcontainers under each Host container. As a Web application will usually have multiple Servlets, each Context also has multiple Wrapper containers. The schematic is as follows:

![](https://s2.loli.net/2023/11/05/4AwIVEsUkzC3cxO.webp)

Suppose: the URL to access is: `http://user.shopping.com:8080/order/buy`, how does Tomcat locate it?

* **First select Service and Engine based on protocol and port number**.Tomcat each connector listens to a different port, the default Http connector listens to 8080, the default AJP listens to 8009.The above example accesses port 8080, which is picked up by the Http connector, and a connector belongs to a Service component, so the The Service component is identified. there is only one Engine container within the Service, so the Engine is also identified.
* **Then according to the domain name of the selected Host **. Service and Engine identified, the Mapper component through the domain name in the URL to find the appropriate Host container. If you visit user.shopping.com in the example, you will find the Host2 container.
* **Find the Context component based on the URL** After Host is confirmed, Mapper matches the corresponding web application path based on the path in the URL. The example accesses /order, so Context4, the Context container, is found.
* **Finally, according to the URL path to find the Wrapper (Servlet)** . Context to determine, Mapper and then according to the web.xml configuration of the Servlet mapping path to find the specific Wrapper and Servlet.

Then there is a layer of parent-child containers to find a particular Servlet, but not only Servlet can handle the request. The actual parent and child containers on the lookup path will do some processing of the request. After the first Engine container does some processing on the request, it will pass the request to its own child container Host to continue processing, and finally the request will be passed to the Wrapper, and this calling process is **implemented through the Pipeline-Valve pipeline**.

### 3.3 Pipeline-Valve

Pipeline-Valve is a chain-of-responsibility pattern. The chain-of-responsibility pattern refers to the fact that there are a number of processors that process a request sequentially, and each processor is responsible for doing its own processing, and then calling the next processor to continue the processing after it is done.

Valve represents a processing point such as permission authentication and log printing. The interface is as follows:

```java
public interface Valve {
  public Valve getNext();
  public void setNext(Valve valve);
  public void invoke(Request request, Response response)
}
```

The invoke method is what handles the request, and getNext calls the next Valve.

The Pipeline interface is as follows.

```java
public interface Pipeline extends Contained {
  public void addValve(Valve valve);
  public Valve getBasic();
  public void setBasic(Valve valve);
  public Valve getFirst();
}
```

There is the addValve method, and the Pipeline maintains a chain of Valves, which can be inserted into the Pipeline to do some processing of the request.There is no invoke method in the Pipeline, because the entire call chain is triggered by the Valve.After the Valve completes its own processing, it will call getNext(). invoke triggers the next Valve call.

Each container has a Pipeline object, as long as the first Valve is triggered all Valves in this container will be called. Different containers are called through the getBasic method, the BasicValve is at the end of the Valve, it is an essential Valve of the Pipeline, responsible for calling the first Valve of the Pipeline of the next layer of containers. the whole process is shown in the following figure:![](https://s2.loli.net/2023/11/05/FvJgqbcG6u9xI4Q.webp)

1. The last Valve of the Wrapper container creates a Filter chain and calls the doFilter method, which finally calls the Servlet's service method.

   Filter and Valve difference:

   * Valve is a Tomce private mechanism , and Tomcat infrastructure / API is tightly coupled . Servlet API is a public standard , all Web containers including Jetty support Filter mechanism .
   * Valve work at the Web container level, intercepting all application requests. Filter work at the application level , can only intercept all the requests of a Web application . If you want to do the entire Web container interception, you must be realized through the Valve.

## 4. Tomcat access to the Servlet process

   Record the Tomcat access Servlet process to facilitate source code tracking .

   1. When Tomcat can read data from the network , it will be encapsulated into a Runnable, thrown to the thread pool processing, which implements the Runnnable method for SocketProcessorBase.

      ```java
       executor.execute(sc);
      ```

   2. Within the doRun method of NioEndpoint$SocketProcessor, the process method of ProtocolHandler is called.

      ```ini
      state = getHandler().process(socketWrapper, event)
      ```

   3. The AbstractProtocol method internally creates the Processor and then calls the Processor's process method.

      ```java
      processor = getProtocol().createProcessor();
      state = processor.process(wrapper, status);
      ```

   4. It will first go to the process method of the AbstractProcessorLight class, which will internally call the service method. This is an abstract class, implemented by concrete classes.

      ```ini
      state = service(socketWrapper)
      ```

   5. The Http11Processor will call the service method of the Adapter.

      ```java
       getAdapter().service(request, response);
      ```


   6. CoyoteAdapter's service method calls postParseRequest, which is called internally. Internally, the Mapper resolves the container (Host, Context) to which the request belongs.

      ```java
      connector.getService().getMapper().map(serverName, decodedURI,
      version, request.getMappingData())
      ```

   7. Then Pipeline will be called to execute the Valve chain, which will successively execute StandardEngineValve, StandardHostValve, StandardContextValve, and StandardWrapperValve.

      ```java
       connector.getService().getContainer().getPipeline().getFirst().invoke(
                               request, response);
      ```

      

   8. In the invoke method of StandardWrapperValve, the Servlet is initialized (if uninitialized)

      ```ini
       servlet = wrapper.allocate()
      ```

   9. The ApplicationFilterChain is then constructed and the FilterChain is invoked

      ```vbscript
       ApplicationFilterChain filterChain =
                       ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);
       //....
       
       filterChain.doFilter(request.getRequest(),
                                           response.getResponse());
      ```

      

   10. ApplicationFilterChain calls the Filter recursively.

       ```java
        filter.doFilter(request, response, this);
       ```

   11. After all Filters are called, the Servlet's service method is called.

       ```java
        servlet.service(request, response);
       ```



## 5. References

1. Tomcat & Jetty: A Deep Dive - Geek Time
2. Tomcat source code branch 8.5.x

![](https://s2.loli.net/2023/11/05/2lxm4cRIXf3usqh.png)
