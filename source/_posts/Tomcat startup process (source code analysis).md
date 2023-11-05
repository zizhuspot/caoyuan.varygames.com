---
title: Tomcat startup process (source code analysis)
date: 2023-11-05 00:10:00
categories: 
  - Technology
tags: 
  - Tomcat
  - method
  - startup
  - Bootstrap
  - Reflection
  - generates
  - finally
  - main
description: The main method of Tomcat startup is Bootstrap, Bootstrap#init method The init method does two main things: Initializes the class loader. Reflection generates a Catalina instance
cover: https://s2.loli.net/2023/11/05/2lxm4cRIXf3usqh.png
---
The main method of Tomcat startup is Bootstrap with the following source code:

```java

Bootstrap daemon;
Bootstrap bootstrap = new Bootstrap();
bootstrap.init();
daemon = bootstrap;
...

daemon.setAwait(true);
daemon.load(args);
daemon.start();
```

## 1. Bootstrap ## init method

The init method does two main things:

```java

initClassLoaders();
Thread.currentThread().setContextClassLoader(catalinaLoader);
SecurityClassLoad.securityClassLoad(catalinaLoader);
​
Class startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
Object startupInstance = startupClass.getConstructor().newInstance();
​
String methodName = "setParentClassLoader";
Class paramTypes[] = new Class[1];
paramTypes[0] = Class.forName("java.lang.ClassLoader");
Object paramValues[] = new Object[1];
paramValues[0] = sharedLoader;
Method method =
            startupInstance.getClass().getMethod(methodName, paramTypes);
method.invoke(startupInstance, paramValues);
​
catalinaDaemon = startupInstance;
```

```java

commonLoader = createClassLoader("common", null);
if (commonLoader == null) {
    commonLoader = this.getClass().getClassLoader();
}
catalinaLoader = createClassLoader("server", commonLoader);
sharedLoader = createClassLoader("shared", commonLoader);
```

## 2. Bootstrap #load

Call Catalina's load method via reflection (why via reflection?)

```java
String methodName = "load";
Method method =catalinaDaemon.getClass().getMethod(methodName, paramTypes);
method.invoke(catalinaDaemon, param);
```

Catalina's load method is divided into three main steps:

```java

initDirs();
initNaming();
​
Digester digester = createStartDigester();
InputSource inputSource = null;
InputStream inputStream = null;
inputSource.setByteStream(inputStream);
digester.push(this);
digester.parse(inputSource);
​

 getServer().init();
```

StandardServer's init method (which actually calls initInternal because it inherits from LifecycleBase, and the same for the methods that follow it), mainly calls the init method of Service, which is implemented as StandardService. (And of course scans for class loader (of course, will also scan the class loader for resources, ignored here)

```java

for (Service service : services) {
    service.init();
}
```

The StandardService's init method does a few things:

```java

engine.init();
​
mapperListener.init();
​
for (Connector connector : connectors) {
    connector.init();
}
```

StandardEngine in the init will call the parent class ContainerBase init, mainly to initialize a startup thread pool.

```java

BlockingQueue startStopQueue = new LinkedBlockingQueue<>();
startStopExecutor = new ThreadPoolExecutor(
                getStartStopThreadsInternal(),
                getStartStopThreadsInternal(), 10, TimeUnit.SECONDS,
                startStopQueue,
                new StartStopThreadFactory(getName() + "-startStop-"));
startStopExecutor.allowCoreThreadTimeOut(true);
```

Connector's init method is the main one:

```java

adapter = new CoyoteAdapter(this);
protocolHandler.setAdapter(adapter);
protocolHandler.init();
```

Endpoint is an interface, the abstract class for AbstractEndpoint, init method will call bindWithCleanup method, and then the internal will call the bind abstract method, different is the implementation of the class processing is not the same. Take NioEndpoint as an example: it mainly creates ServerSocketChannel and then binds the port.

```java

public void bind() throws Exception {
    initServerSocket();
    setStopLatch(new CountDownLatch(1));

    initialiseSsl();
}
```

```java

protected void initServerSocket() throws Exception {
    serverSock = ServerSocketChannel.open();
    socketProperties.setProperties(serverSock.socket());
    InetSocketAddress addr = new InetSocketAddress(getAddress(), getPortWithOffset());
    serverSock.socket().bind(addr,getAcceptCount());

    serverSock.configureBlocking(true);

}
```

## 3. Bootstrap#start

Calling Catalina's start method via reflection

```java

Method method = catalinaDaemon.getClass().getMethod("start", (Class [])null);
method.invoke(catalinaDaemon, (Object [])null);
```

There are three main steps in Catalina's START method:

```java

getServer().start();

if (shutdownHook == null) {
    shutdownHook = new CatalinaShutdownHook();
}
Runtime.getRuntime().addShutdownHook(shutdownHook);
```

```java
if (await) {
    await();
    stop();
}
```

The start method of Server mainly calls the start method of Service, which is internally implemented as startInternal method because it inherits LifecycBase. Some of the following classes are analyzed in the same way.

```java

for (Service service : services) {
    service.start();
}
```

Service's start method, which does three main things.

```java

engine.start();
​
mapperListener.start();
​
for (Connector connector: connectors) {
    connector.start();
}
```

Engine's start method, which mainly calls the startInternal method of the parent class ContainerBase, does the following:

```java

Container children[] = findChildren();
List> results = new ArrayList<>();
for (Container child : children) {
    results.add(startStopExecutor.submit(new StartChild(child)));
}
for (Future result : results) {
    result.get();
}

if (pipeline instanceof Lifecycle) {
    ((Lifecycle) pipeline).start();
}

threadStart();
```

Host (implementation class for StandardHost) in the start no special, will call the parent class ContainerBase startInternal method, will continue to look for child containers to call the start method

In the Context container, the execution logic of the start method will be more complex:

Create a Loader, which will contain a class loader internally, and call the start method.

```java

WebappLoader webappLoader = new WebappLoader();
webappLoader.setDelegate(getDelegate());
setLoader(webappLoader);
​
Loader loader = getLoader();
if (loader instanceof Lifecycle) {
    ((Lifecycle) loader).start();
}
```

Find the subcontainer (Wrapper) and call the start method.

```java
for (Container child : findChildren()) {
    if (!child.getState().isAvailable()) {
        child.start();
    }
}
```

Calling the onStartup method of the ServletContainerInitializer interface

```java
for (Map.Entry>> entry :initializers.entrySet()) {
    entry.getKey().onStartup(entry.getValue(),getServletContext());
}
```

Looking for subcontainers that need to be loaded at startup time, the subcontainer Wrapper loads the Servlet internally and calls the Servlet#init method. This kind is loaded at startup, by default, it is delayed loading.

```java
loadOnStartup(findChildren());
```

There is nothing special about the Wrapper container.

Connector internally calls the start method of ProtocolHandler

```java

protocolHandler.start();
```

ProtocolHandler is an interface with an abstract class AbstractProtocol, which internally implements the main logic:

```java

endpoint.start();
​
asyncTimeout = new AsyncTimeout();
Thread timeoutThread = new Thread(asyncTimeout, getNameInternal() + "-AsyncTimeout");
timeoutThread.start();
```

Endpoint is an interface , the abstract class is AbstractEndpoint, the internal will call startInternal, implemented by different implementation of the class . NioEndpoint as an example. The main logic is as follows::

```java

createExecutor();

initializeConnectionLatch();

poller = new Poller();
Thread pollerThread = new Thread(poller, getName() + "-Poller");
pollerThread.setPriority(threadPriority);
pollerThread.setDaemon(true);
pollerThread.start();
​

startAcceptorThread();
```

```java

public void createExecutor() {
    internalExecutor = true;
    TaskQueue taskqueue = new TaskQueue();
    TaskThreadFactory tf = new TaskThreadFactory(getName() + "-exec-", daemon, getThreadPriority());
    executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), 60, TimeUnit.SECONDS,taskqueue, tf);
    taskqueue.setParent( (ThreadPoolExecutor) executor);
}
```

run method of the runnable interface, which essentially calls Catalina's stop

```java

Catalina.this.stop();
```

Catalina#stop method, which essentially calls the Server's stop and destroy

```java

Server s = getServer();
LifecycleState state = s.getState();
if (LifecycleState.STOPPING_PREP.compareTo(state) 0
                    && LifecycleState.DESTROYED.compareTo(state) >= 0) {

} else {
    s.stop();
    s.destroy();
}
```

Internally, the Server's await method will actually be called. By default, the port has a value, a ServerSocket is created, and the main thread blocks in the accept() method.

```java

awaitSocket = new ServerSocket(port, 1,
                    InetAddress.getByName(address));
while (!stopAwait) {
    ServerSocket serverSocket = awaitSocket;
    Socket socket = serverSocket.accept();
}
```

In the stopServer method in Catalina (which can be called with the command stop, which will call that method), in addition to calling Stop on the Server, if the port is greater than 0, it will also create a socket and send the SHUTDOWN string.

```java

Socket socket = new Socket(s.getAddress(), s.getPort()；
OutputStream stream = socket.getOutputStream()；
String shutdown = s.getShutdown();
for (int i = 0; i < shutdown.length(); i++) {
    stream.write(shutdown.charAt(i));
}
stream.flush();
```

