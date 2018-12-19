[TOC]

------

### j2ee_servlet

#### 1.servlet 3.1规范

######  1.1 What is servlet

	A servlet is a JavaTM technology-based Web component, managed by a container, that generates dynamic content. 

> From servlet 3.1

 ###### 1.2 History

![](https://ws1.sinaimg.cn/large/006tNbRwly1fy9oye7wq4j31pm0hyarl.jpg)

> From wiki

######  1.3 Servlet Life Cycle

- Loading and Instantiation

	When the servlet engine is started, needed servlet classes must be located by the
servlet container(WEB-INF/lib)

- Initialization

      The container initializes the servlet instance by calling the init method of the Servlet interface with a unique (per servlet declaration) object implementing the ServletConfig interface

（ServletConfig used by Servlet）

- Request Handling

     After a servlet is properly initialized, the servlet container may use it to handle client
requests. 

- End of Service

###### 1.4 Servlet 继承结构

![image-20181219221816845](https://ws2.sinaimg.cn/large/006tNbRwly1fycf0mk9d5j30u00u6dqo.jpg)

###### 1.5 ServletContext

		The ServletContext interface defines a servlet’s view of the Web application within which the servlet is running.  (web.xml) The Container Provider is responsible for providing an implementation of the ServletContext interface in the servlet container.

```markdown
InitParameter
config 
	-Filter
	-Listenr
	-Servlet
Attribute
Resource
...
```

> see : ApplicationContext、ApplicationContextFacade (tomcat)

###### 1.6 Request 

HttpServletRequest

- HTTP Protocol Parameters

  > ■ getParameter
  >
  > ■ getParameterNames 
  >
  > ■ getParameterValues 
  >
  > ■ getParameterMap

- File upload 

  content-type : multipart/form-data 

- Attributes 

- Headers

- Request Path Elements

  > requestURI = contextPath + servletPath + pathInfo

- Path Translation Methods

  > ■ ServletContext.getRealPath  
  >
  > ■ HttpServletRequest.getPathTranslated

- Non Blocking IO

  Non-blocking IO only works with async request processing in Servlets and Filters

- Cookies

- SSL

- Internationalization

  Accept-Language : zh-cn

- > ■ getLocale
  > ■ getLocales

- Request data encoding 

    The default encoding of a request the container uses to create the request reader and parse POST data must be “ISO-8859-1” if none has been specified by the client request. 

- Lifetime of the Request Object 

   Each request object is valid only within the scope of a servlet’s service method, or within the scope of a filter’s doFilter method, unless the asynchronous processing is enabled for the component and the startAsync method is invoked on the request object. 

###### 1.7 Response

![image-20181219221656097](https://ws1.sinaimg.cn/large/006tNbRwly1fyceza35scj30lc0luwf3.jpg)

###### 1.8 Filter

 	what is Filter? 

	A filter is a reusable piece of code that can transform the content of HTTP requests,responses, and header information.
> ```
> org.springframework.web.servlet.HandlerInterceptor
> ```

###### 1.9 Lifecycle Events

```json
Event
	-Servlet
	-Session
	-Request
EventListener
	-Servlet
	-Session
	-Request
```

Example : `ServletContextListener`

```java
public class ContextLoaderListener implements ServletContextListener {
    private ContextLoader contextLoader;

    public ContextLoaderListener() {
    }

    public void contextInitialized(ServletContextEvent event) {
        this.contextLoader = this.createContextLoader();
        this.contextLoader.initWebApplicationContext(event.getServletContext());
    }

    protected ContextLoader createContextLoader() {
        return new ContextLoader();
    }

    public ContextLoader getContextLoader() {
        return this.contextLoader;
    }

    public void contextDestroyed(ServletContextEvent event) {
        if(this.contextLoader != null) {
            this.contextLoader.closeWebApplicationContext(event.getServletContext());
        }

    }
}
```

###### 1.10 Session



#### 2. Servlet 

	Server + Applet 的缩写，表示一个服务器应用

###### 2.1 Servlet接口 

```java
package javax.servlet;
import java.io.IOException;
public interface Servlet {
    public void init(ServletConfig config) throws ServletException;
    
    public ServletConfig getServletConfig();
    
    public void service(ServletRequest req, ServletResponse res)
   throws ServletException, IOException;

    public String getServletInfo();
    
    public void destroy();
    
}
```

> Load-on-startup 为负的话不会在容器启动调用

###### 2.2 ServletConfig接口

```java
package javax.servlet;
import java.util.Enumeration;
/**
 * A servlet configuration object used by a servlet container
 * to pass information to a servlet during initialization. 
 */
 public interface ServletConfig {
    
    public String getServletName();

    public ServletContext getServletContext();

    public String getInitParameter(String name);

    public Enumeration<String> getInitParameterNames();

}
```

如下配置

```xml
<!--web.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<web-app ...>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>application-context.xml</param-value>
    </context-param>
<servlet>
    <servlet-name>demoDispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.Dispatcher</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>demo-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
</web-app>
```

在Servlet中 可以分别通过它们的getInitParameter方法获取，比如：

```java
String contextLocation = getServletConfig().getServletContext().getInitParameter(
    "contextConfigLocation");
String servletLocation = getServletConfig().getInitParameter("contextConfigLocation");
```

###### 2.3 GenerieServlet

	`Servlet`的默认实现，同时实现了`ServletConfig`接口、`Serializable`接口，所以可以直接调用`ServletConfig`里面的方法。详细可参考如下类注释。

```java
package javax.servlet;

import java.io.IOException;
import java.util.Enumeration;

/**
 *
 * Defines a generic, protocol-independent
 * servlet. To write an HTTP servlet for use on the
 * Web, extend {@link javax.servlet.http.HttpServlet} instead.
 *
 * <p><code>GenericServlet</code> implements the <code>Servlet</code>
 * and <code>ServletConfig</code> interfaces. <code>GenericServlet</code>
 * may be directly extended by a servlet, although it's more common to extend
 * a protocol-specific subclass such as <code>HttpServlet</code>.
 *
 * <p><code>GenericServlet</code> makes writing servlets
 * easier. It provides simple versions of the lifecycle methods 
 * <code>init</code> and <code>destroy</code> and of the methods 
 * in the <code>ServletConfig</code> interface. <code>GenericServlet</code>
 * also implements the <code>log</code> method, declared in the
 * <code>ServletContext</code> interface. 
 *
 * <p>To write a generic servlet, you need only
 * override the abstract <code>service</code> method. 
 *
 */
public abstract class GenericServlet 
    implements Servlet, ServletConfig, java.io.Serializable
{
    private transient ServletConfig config;

    public GenericServlet() {}

    public void destroy() {}
    
    public String getInitParameter(String name) {
		return getServletConfig().getInitParameter(name);
    }
    
    public Enumeration getInitParameterNames() {
		return getServletConfig().getInitParameterNames();
    }   
    
    public ServletConfig getServletConfig() {
		return config;
    }
    
    public ServletContext getServletContext() {
		return getServletConfig().getServletContext();
    }
    
    public String getServletInfo() {
		return "";
    }

    public void init(ServletConfig config) throws ServletException {
		this.config = config;
		this.init();
    }

    public void init() throws ServletException {
    }
    
    public void log(String msg) {
		getServletContext().log(getServletName() + ": "+ msg);
    }
   
    public void log(String message, Throwable t) {
		getServletContext().log(getServletName() + ": " + message, t);
    }
    
    public abstract void service(ServletRequest req, ServletResponse res)
	throws ServletException, IOException;
    
    public String getServletName() {
        return config.getServletName();
    }
}

```

附：为什么需要实现 `java.io.Serializable` 接口？

------

答：在 Servlet 2.4 规范的 7.7.2 Distributed Environments 章节中，有一句这样的描述：

```markdown
The distributed servlet container must support the mechanism necessary for
migrating objects that implement Serializable.
```

> 按照规范的设计，Servlet 有一个钝化的特性，类似于 Servlet 持久化到文件，然后当容器 Crash 回复后，可以重新恢复保存前的状态。



###### 2.4 HttpServlet

```java
package javax.servlet.http;

import ....;

/**
 * Provides an abstract class to be subclassed to create
 * an HTTP servlet suitable for a Web site. A subclass of
 * <code>HttpServlet</code> must override at least 
 * one method, usually one of these:
 *
 * <ul>
 * <li> <code>doGet</code>, if the servlet supports HTTP GET requests
 * <li> <code>doPost</code>, for HTTP POST requests
 * <li> <code>doPut</code>, for HTTP PUT requests
 * <li> <code>doDelete</code>, for HTTP DELETE requests
 * <li> <code>init</code> and <code>destroy</code>, 
 * to manage resources that are held for the life of the servlet
 * <li> <code>getServletInfo</code>, which the servlet uses to
 * provide information about itself 
 * </ul>
 *
 * <p>There's almost no reason to override the <code>service</code>
 * method. <code>service</code> handles standard HTTP
 * requests by dispatching them to the handler methods
 * for each HTTP request type (the <code>do</code><i>XXX</i>
 * methods listed above).
 *
 * <p>Likewise, there's almost no reason to override the 
 * <code>doOptions</code> and <code>doTrace</code> methods.
 *
 */
public abstract class HttpServlet extends GenericServlet
    implements java.io.Serializable
{
    private static final String METHOD_DELETE = "DELETE";
    private static final String METHOD_HEAD = "HEAD";
    private static final String METHOD_GET = "GET";
    private static final String METHOD_OPTIONS = "OPTIONS";
    private static final String METHOD_POST = "POST";
    private static final String METHOD_PUT = "PUT";
    private static final String METHOD_TRACE = "TRACE";

    private static final String HEADER_IFMODSINCE = "If-Modified-Since";
    private static final String HEADER_LASTMOD = "Last-Modified";
    
    /**
    * Resource bundles contain locale-specific objects.
    */
    private static final String LSTRING_FILE =
	"javax.servlet.http.LocalStrings";
    private static ResourceBundle lStrings =
	ResourceBundle.getBundle(LSTRING_FILE);

    public HttpServlet() { }
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
	throws ServletException, IOException{
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_get_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, msg);
        } else {
            resp.sendError(HttpServletResponse.SC_BAD_REQUEST, msg);
        }
    }

    protected long getLastModified(HttpServletRequest req) {
		return -1;
    }

    protected void doHead(HttpServletRequest req, HttpServletResponse resp)
	throws ServletException, IOException{
        NoBodyResponse response = new NoBodyResponse(resp);

        doGet(req, response);
        response.setContentLength();
    }
  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
	throws ServletException, IOException{
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_post_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, msg);
        } else {
            resp.sendError(HttpServletResponse.SC_BAD_REQUEST, msg);
        }
    }
  
    protected void doPut(HttpServletRequest req, HttpServletResponse resp)
	throws ServletException, IOException{
       //略,类似doPost
    }
    
    protected void doDelete(HttpServletRequest req,
			    HttpServletResponse resp)
	throws ServletException, IOException{
       //略,类似doPost
    }
  
    protected void doOptions(HttpServletRequest req, HttpServletResponse resp)
	throws ServletException, IOException{
      //略,主要用于调试,输出允许类型
    }
    protected void service(HttpServletRequest req, HttpServletResponse resp)
	throws ServletException, IOException{
        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
            // servlet doesn't support if-modified-since, no reason
            // to go through further expensive logic
            doGet(req, resp);
            } else {
            long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
            if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                // If the servlet mod time is later, call doGet()
                        // Round down to the nearest second for a proper compare
                        // A ifModifiedSince of -1 will always be less
                maybeSetLastModified(resp, lastModified);
                doGet(req, resp);
            } else {
                resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
            }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);
        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);	
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
    
    public void service(ServletRequest req, ServletResponse res)
	throws ServletException, IOException{
        HttpServletRequest	request;
        HttpServletResponse	response;

        try {
            request = (HttpServletRequest) req;
            response = (HttpServletResponse) res;
        } catch (ClassCastException e) {
            throw new ServletException("non-HTTP request or response");
        }
        service(request, response);
    }
}

/*
 * A response that includes no body, for use in (dumb) "HEAD" support.
 * This just swallows that body, counting the bytes in order to set
 * the content length appropriately.  All other methods delegate directly
 * to the HTTP Servlet Response object used to construct this one.
 */
// file private
class NoBodyResponse extends HttpServletResponseWrapper {
    private NoBodyOutputStream		noBody;
    private PrintWriter			writer;
    private boolean			didSetContentLength;

    // file private
    NoBodyResponse(HttpServletResponse r) {
        super(r);
	noBody = new NoBodyOutputStream();
    }
	// ....
}
/*
 * Servlet output stream that gobbles up all its data.
 */
 
// file private
class NoBodyOutputStream extends ServletOutputStream {
	//...
}

```

	doXXX 都是模板方法，如果子类没有实现将抛出异常
	
	doGet 方法前还会对是否过期做检查，如果没有过期，则直接返回304状态码做缓存。
	
	doHead调用了doGet的请求，然后返回空body的response
	
	doOptions和doTrace 主要是用来做一些调试工作 

#### 3.servlet容器 tomcat

![wiki_tomcat](https://ws1.sinaimg.cn/large/006tNbRwly1fy9tr0l2swj31kt0u0qed.jpg)

##### 3.1 Tomcat的顶层结构

```
Catalina 管理整个Tomcat的管理类

Server 最顶层容器，代表整个服务器

	Service 提供具体服务 （多个）

		Connector 负责网络连接、request/response的创建（可以有多个连接，从servet.xml的配置也可以看出，同时提供http和https，也可以提供相同协议不同端口的连接）

		Container 具体处理Servlet
```



##### 3.2 Tomcat的启动过程

	`org.apache.catalina.startup.Bootstrap` 是Tomcat的入口，作用类似一个`CatalinaAdptor`，具体处理还是Catalina来完成，这样做的好处是可以把启动的入口和具体的管理类分开，从而可以很方便地创建出多种启动方式。 

> BootStrap不在Tomcat依赖包下 ，而是在bin目录 通过反射 完全松耦合

```java
package org.apache.catalina.startup;

import ...;

public final class Bootstrap {
    private static final Log log = LogFactory.getLog(Bootstrap.class);
    /**
     * Daemon object used by main.
     */
    private static Bootstrap daemon = null;
    /**
     * Daemon reference.
     */
    private Object catalinaDaemon = null;

    ClassLoader commonLoader = null;
    ClassLoader catalinaLoader = null;
    ClassLoader sharedLoader = null;
    private void initClassLoaders() {
        try {
            commonLoader = createClassLoader("common", null);
            if( commonLoader == null ) {
                // no config file, default to this loader - we might be in a 'single' env.
                commonLoader=this.getClass().getClassLoader();
            }
            catalinaLoader = createClassLoader("server", commonLoader);
            sharedLoader = createClassLoader("shared", commonLoader);
        } catch (Throwable t) {
            handleThrowable(t);
            log.error("Class loader creation threw exception", t);
            System.exit(1);
        }
    }

    private ClassLoader createClassLoader(String name, ClassLoader parent)
        throws Exception {

        String value = CatalinaProperties.getProperty(name + ".loader");
        if ((value == null) || (value.equals("")))
            return parent;

        value = replace(value);

        List<Repository> repositories = new ArrayList<>();

        String[] repositoryPaths = getPaths(value);

        for (String repository : repositoryPaths) {
            // Check for a JAR URL repository
            try {
                @SuppressWarnings("unused")
                URL url = new URL(repository);
                repositories.add(new Repository(repository, RepositoryType.URL));
                continue;
            } catch (MalformedURLException e) {
                // Ignore
            }

            // Local repository
            if (repository.endsWith("*.jar")) {
                repository = repository.substring
                    (0, repository.length() - "*.jar".length());
                repositories.add(
                        new Repository(repository, RepositoryType.GLOB));
            } else if (repository.endsWith(".jar")) {
                repositories.add(
                        new Repository(repository, RepositoryType.JAR));
            } else {
                repositories.add(
                        new Repository(repository, RepositoryType.DIR));
            }
        }
        return ClassLoaderFactory.createClassLoader(repositories, parent);
    }

    /**
     * Initialize daemon.
     * @throws Exception Fatal initialization error
     */
    public void init() throws Exception {
        initClassLoaders();

        Thread.currentThread().setContextClassLoader(catalinaLoader);

        SecurityClassLoad.securityClassLoad(catalinaLoader);

        // Load our startup class and call its process() method
        if (log.isDebugEnabled())
            log.debug("Loading startup class");
        Class<?> startupClass =
            catalinaLoader.loadClass
            ("org.apache.catalina.startup.Catalina");
        Object startupInstance = startupClass.newInstance();

        // Set the shared extensions class loader
        if (log.isDebugEnabled())
            log.debug("Setting startup class properties");
        String methodName = "setParentClassLoader";
        Class<?> paramTypes[] = new Class[1];
        paramTypes[0] = Class.forName("java.lang.ClassLoader");
        Object paramValues[] = new Object[1];
        paramValues[0] = sharedLoader;
        Method method =
            startupInstance.getClass().getMethod(methodName, paramTypes);
        method.invoke(startupInstance, paramValues);

        catalinaDaemon = startupInstance;
    }
    
    /**
     * Load daemon.
     */
    private void load(String[] arguments)
        throws Exception {

        // Call the load() method
        String methodName = "load";
        Object param[];
        Class<?> paramTypes[];
        if (arguments==null || arguments.length==0) {
            paramTypes = null;
            param = null;
        } else {
            paramTypes = new Class[1];
            paramTypes[0] = arguments.getClass();
            param = new Object[1];
            param[0] = arguments;
        }
        Method method =
            catalinaDaemon.getClass().getMethod(methodName, paramTypes);
        if (log.isDebugEnabled())
            log.debug("Calling startup class " + method);
        method.invoke(catalinaDaemon, param);
    }

    // ----------------------------------------------------------- Main Program

    /**
     * Load the Catalina daemon.
     * @param arguments Initialization arguments
     * @throws Exception Fatal initialization error
     */
    public void init(String[] arguments)
        throws Exception {
        init();
        load(arguments);

    }
    
    /**
     * Start the Catalina daemon.
     * @throws Exception Fatal start error
     */
    public void start()
        throws Exception {
        if( catalinaDaemon==null ) init();

        Method method = catalinaDaemon.getClass().getMethod("start", (Class [] )null);
        method.invoke(catalinaDaemon, (Object [])null);
    }

    /**
     * Stop the Catalina Daemon.
     * @throws Exception Fatal stop error
     */
    public void stop()
        throws Exception {
		//实现略,主要通过反射调用了catalina的stop
    }

    /**
     * Stop the standalone server.
     * @throws Exception Fatal stop error
     */
    public void stopServer()
        throws Exception {
		//实现略,主要通过反射调用了catalina的stopServer
    }
    
    /**
     * Set flag.
     * @param await <code>true</code> if the daemon should block
     * @throws Exception Reflection error
     */
    public void setAwait(boolean await)
        throws Exception {
		//实现略 ,主要通过反射调用了catalina的setAwait
    }

    public boolean getAwait()
        throws Exception{
        //实现略 ,主要通过反射调用了catalina的getAwait
    }

    /**
     * Destroy the Catalina Daemon.
     */
    public void destroy() {
        // FIXME
    }
    /**
     * Main method and entry point when starting Tomcat via the provided
     * scripts.
     *
     * @param args Command line arguments to be processed
     */
    public static void main(String args[]) {

        if (daemon == null) {
            // Don't set daemon until init() has completed
            Bootstrap bootstrap = new Bootstrap();
            try {
                bootstrap.init();
            } catch (Throwable t) {
                handleThrowable(t);
                t.printStackTrace();
                return;
            }
            daemon = bootstrap;
        } else {
            // When running as a service the call to stop will be on a new
            // thread so make sure the correct class loader is used to prevent
            // a range of class not found exceptions.
            Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
        }
        try {
            String command = "start";
            if (args.length > 0) {
                command = args[args.length - 1];
            }
            if (command.equals("startd")) {
                args[args.length - 1] = "start";
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stopd")) {
                args[args.length - 1] = "stop";
                daemon.stop();
            } else if (command.equals("start")) {
                daemon.setAwait(true);
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stop")) {
                daemon.stopServer(args);
            } else if (command.equals("configtest")) {
                daemon.load(args);
                if (null==daemon.getServer()) {
                    System.exit(1);
                }
                System.exit(0);
            } else {
                log.warn("Bootstrap: command \"" + command + "\" does not exist.");
            }
        } catch (Throwable t) {
            // Unwrap the Exception for clearer error reporting
            if (t instanceof InvocationTargetException &&
                    t.getCause() != null) {
                t = t.getCause();
            }
            handleThrowable(t);
            t.printStackTrace();
            System.exit(1);
        }

    }
}
```

	Tomcat 启动脚本 startup.bat 是从main方法中开始的。其中主要做了:

 - 准备容器环境，`init()`初始化类加载器, 
 - 初始化容器，调用`load()` 实际是调用catalina里的`init()`
 - 启动容器，通过引用`catalinaDaemon` 反射射调用`start()`方法（实际还是通过catalina操作容器）

         关于类加载，我们都知道 J2EE 默认的类加载机制是双亲委派原则（详细查看如下🔎https://www.cnblogs.com/miduos/p/9250565.html）

	通过debug可以发现  commonLoader、catalinaLoader 、sharedLoader 其实三个是同一个（底层都是URLClassLoader），原因是因为`catalina.properties` 的配置中默认是空的。

	另外在`init()` 中 `Thread.currentThread().setContextClassLoader(catalinaLoader);` 



###### 3.2.1Catalina的启动过程

	Catalina的启动主要是调用`setAwait()`、`load()`和`start()`方法来完成。

- `setAwait()` 方法用于设置Server启动完成后是否进入等待状态的标记
- `load() `方法主要是用来加载配置文件`conf/server.xml`创建Server对象 （解析是通过Digester），然后调用Server的`init()`
- `start()` 主要是调用Server的 `start()`



###### 3.2.2 Server的启动过程	

	Server的默认实现`org.apache.catalina.core.StandardServer` ,在其父类中`org.apache.catalina.util.LifecycleBase` 中的`init() ` 实现如下

```java
@Override
public final synchronized void init() throws LifecycleException {
    if (!state.equals(LifecycleState.NEW)) {
        invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
    }

    try {
        setStateInternal(LifecycleState.INITIALIZING, null, false);
        initInternal();
        setStateInternal(LifecycleState.INITIALIZED, null, false);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(
                sm.getString("lifecycleBase.initFail",toString()), t);
    }
}
```

 `start()` 实现如下

```java
/**
 * {@inheritDoc}
 */
@Override
public final synchronized void start() throws LifecycleException {

    if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||LifecycleState.STARTED.equals(state)) {

        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
        } else if (log.isInfoEnabled()) {
            log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
        }
        return;
    }

    if (state.equals(LifecycleState.NEW)) {
        init();
    } else if (state.equals(LifecycleState.FAILED)) {
        stop();
    } else if (!state.equals(LifecycleState.INITIALIZED) &&
            !state.equals(LifecycleState.STOPPED)) {
        invalidTransition(Lifecycle.BEFORE_START_EVENT);
    }

    try {
        setStateInternal(LifecycleState.STARTING_PREP, null, false);
        startInternal();
        if (state.equals(LifecycleState.FAILED)) {
            // This is a 'controlled' failure. The component put itself into the
            // FAILED state so call stop() to complete the clean-up.
            stop();
        } else if (!state.equals(LifecycleState.STARTING)) {
            // Shouldn't be necessary but acts as a check that sub-classes are
            // doing what they are supposed to.
            invalidTransition(Lifecycle.AFTER_START_EVENT);
        } else {
            setStateInternal(LifecycleState.STARTED, null, false);
        }
    } catch (Throwable t) {
        // This is an 'uncontrolled' failure so put the component into the
        // FAILED state and throw an exception.
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(sm.getString("lifecycleBase.startFail", toString()), t);
    }
}
```

	其中 `startInternal() 和 initInternal()` 为模版方法 ，查看其实现类 可以发现是循环调用了每个`service`的`start()`和`init()`

###### 3.2.3 Service的启动过程

	类似于Server , `StandardService`的`initInternal()`和 `startInternal()`的方法主要调用`container`、`executors`、`mapperListener`、`connectors`的`init()`和`start()`方法。

> mapperListener是Mapper的监听器，可以监听container容器多变化
>
> executors是用在connectors中管理线程的线程池，在server.xml配置文件中tomcatThreadPool

下图为整个启动流程

![image-20181218123224591](https://ws4.sinaimg.cn/large/006tNbRwgy1fyasgs52ycj30wk0k6jxv.jpg)

##### 3.3 请求处理

​		Connector用于接收请求并将请求封装成`Request`和`Response`来处理，最底层是使用Socket来进行连接的，封装完后交给Container进行处理（通过pipeline-Value管道来处理），Container处理完之后返回给Connector，最后Connector使用Socket将处理结果返回给客户端，整个请求就处理完了。

###### 3.3.1 Connector的结构

​	`Connector`中具体是用`ProtocolHandler`来处理请求的，有多种不同的连接类型（Ajp、HTTP和Spdy）。

其中有3个非常重要的组件。（`Connector`的创建过程主要是初始化`ProtocolHandler`,serverx.xml中可配置）

- Endpoint  用于处理底层的Socket的网络连接，用来实现TCP/IP协议
- Processor  用于将Endpoint接收到的Socket封装成Request，用来实现HTTP协议(BIO、NIO、APR)
- Adapter 用于将封装好的Request交给Container进行具体处理，将适配到Servlet容器（转换`org.apache.coyote.Request`为`org.apache.catalina.connector.Request` ）

> Ajp ：连接器监听8009端口，负责和其他的HTTP服务器建立连接。在把Tomcat与其他HTTP服务器集成时，就需要用到这个连接器。
>
> Apr : 是从操作系统级别解决异步IO问题，大幅度提高服务器的并发处理性能 http://tomcat.apache.org/tomcat-8.5-doc/apr.html
>
> 协议升级：在servlet 3.1之后新增 WebSocket协议，如果Processor处理之后Socket的状态是UPGRADING,则EndPoint中的handler回接着创建并调用upgrade包中的processor进行处理
>
> request转换：Adapter转换后的request，其实就是封装了一层，  `public class Request implements org.apache.catalina.servlet4preview.http.HttpServletRequest`

###### 3.3.2 Pipeline-Value管道

​	Piepeline的管道模型和普通的责任链模式稍微有点不同，区别主要如下

- 每个pipeline都有特定的value，而且是在管道的最后一个执行，这个Value叫做BaseValue

- 上层的BaseValue（类似配货车到中转站的感觉）会调用下层容器的管道

> 每个BaseValue都有一个StandarValue的实现，例如WrapperValue的标准实现是StanderWrpperValue

​	经过所有管道（EnginePipeline、HostPipeline、ContextPipeline、WrapperPieline）后，在最后的WrapperPieline的BaseValue中会创建`FilterChain`并调用其doFilter来处理请求，FilterChain包含着我们配置的与请求相匹配的`Filter`和`Servlet`。

#### 4. springMVC 核心servlet —DispatcherServlet

​	基于spring-webmvc 4.0.3.RELEASE

##### 4.1 整体结构

![image-20181219135235499](https://ws3.sinaimg.cn/large/006tNbRwgy1fyc0ein61sj31m20u0adx.jpg)

> Aware：如果某个类想使用Spring的一些东西，就可以实现Aware接口
>
> Capable:具有spring某个类的能力

##### 4.2 创建过程

###### 4.2.1 HttpServletBean的创建

​	    实现了`EnvironmentCapable`、`EnvironmentAware` ，主要作用是将Servlet中配置的参数设置到相应的

###### 4.2.2 FramewordkServlet的创建

​	    实现了`ApplicationContextAware`，其主要作用是调用`initWebApplicationContext()`初始化`WebApplicationContext`

- 获取spring的根工期rootContext
- 设置webApplicationContext并根据情况调用onRefresh方法
- 将WebApplicationContext设置到ServletContext中			

###### 4.2.3 DispatcherServlet的创建

​	onRefresh方法是DispatcherServlet的入口方法，onRefresh中调用了initStrategies()方法来初始化一些策略组件

```java

protected void onRefresh(ApplicationContext context) {
    this.initStrategies(context);
}

/**
 * Initialize the strategy objects that this servlet uses.
 * <p>May be overridden in subclasses in order to initialize further strategy objects.
 */
protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```
如果没有配置相关，会使用默认的配置，在DispatcherServlet.properties文件中

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
   org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
   org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
   org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
   org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
   org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

> Tips：If no bean is defined with the given name in the BeanFactory for this namespace, no multipart handling is provided.

##### 4.3 组件概览

###### 4.3.1 HandlerMapping

​	它的作用是根据request找到对应的处理器handler和Interceptors ,首先看下接口信息

```java
package org.springframework.web.servlet;
import javax.servlet.http.HttpServletRequest;
/**
 * Interface to be implemented by objects that define a mapping between
 * requests and handler objects.
 * @see org.springframework.core.Ordered
 * @see org.springframework.web.servlet.handler.AbstractHandlerMapping
 * @see org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping
 * @see org.springframework.web.servlet.handler.SimpleUrlHandlerMapping
 */
public interface HandlerMapping {

   String PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE = HandlerMapping.class.getName() + ".pathWithinHandlerMapping";

   String BEST_MATCHING_PATTERN_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingPattern";

   String INTROSPECT_TYPE_LEVEL_MAPPING = HandlerMapping.class.getName() + ".introspectTypeLevelMapping";

   String URI_TEMPLATE_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".uriTemplateVariables";

   String MATRIX_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".matrixVariables";

   String PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE = HandlerMapping.class.getName() + ".producibleMediaTypes";

   /**
    * Return a handler and any interceptors for this request. The choice may be made
    * on request URL, session state, or any factor the implementing class chooses.
    * <p>The returned HandlerExecutionChain contains a handler Object, rather than
    * even a tag interface, so that handlers are not constrained in any way.
    * For example, a HandlerAdapter could be written to allow another framework's
    * handler objects to be used.
    * <p>Returns {@code null} if no match was found. This is not an error.
    * The DispatcherServlet will query all registered HandlerMapping beans to find
    * a match, and only decide there is an error if none can find a handler.
    * @param request current HTTP request
    * @return a HandlerExecutionChain instance containing handler object and
    * any interceptors, or {@code null} if no mapping found
    * @throws Exception if there is an internal error
    */
   HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;

}
```

以常用接口实现为例，RequestMappingHandlerMapping

![image-20181219181838287](https://ws4.sinaimg.cn/large/006tNbRwly1fyc839v8vdj31l20u043g.jpg)

> 这里提供了Ordered接口，可以确定匹配的顺序，同时通过继承WebApplicationObjectSupport抽象类，可以获取相关Bean（handler）

###### 4.3.2 HandlerAdapter

​	之所以要使用HandlerAdapter，是为SpringMVC中并没有对处理器做任何的限制，可以是个类，方法（HandlerMethod）这点从hanlder是Object就可以看出，首先是接口实现

```java
package org.springframework.web.servlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * MVC framework SPI interface, allowing parameterization of core MVC workflow.
 *
 * <p>Interface that must be implemented for each handler type to handle a request.
 * This interface is used to allow the {@link DispatcherServlet} to be indefinitely
 * extensible. The DispatcherServlet accesses all installed handlers through this
 * interface, meaning that it does not contain code specific to any handler type.
 *
 * <p>Note that a handler can be of type {@code Object}. This is to enable
 * handlers from other frameworks to be integrated with this framework without
 * custom coding, as well as to allow for annotation handler objects that do
 * not obey any specific Java interface.
 *
 * <p>This interface is not intended for application developers. It is available
 * to handlers who want to develop their own web workflow.
 *
 * <p>Note: HandlerAdaptger implementators may implement the
 * {@link org.springframework.core.Ordered} interface to be able to specify a
 * sorting order (and thus a priority) for getting applied by DispatcherServlet.
 * Non-Ordered instances get treated as lowest priority.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @see org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter
 * @see org.springframework.web.servlet.handler.SimpleServletHandlerAdapter
 */
public interface HandlerAdapter {

   /**
    * Given a handler instance, return whether or not this HandlerAdapter can
    * support it. Typical HandlerAdapters will base the decision on the handler
    * type. HandlerAdapters will usually only support one handler type each.
    * <p>A typical implementation:
    * <p>{@code
    * return (handler instanceof MyHandler);
    * }
    * @param handler handler object to check
    * @return whether or not this object can use the given handler
    */
   boolean supports(Object handler);

   /**
    * Use the given handler to handle this request.
    * The workflow that is required may vary widely.
    * @param request current HTTP request
    * @param response current HTTP response
    * @param handler handler to use. This object must have previously been passed
    * to the {@code supports} method of this interface, which must have
    * returned {@code true}.
    * @throws Exception in case of errors
    * @return ModelAndView object with the name of the view and the required
    * model data, or {@code null} if the request has been handled directly
    */
   ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

   /**
    * Same contract as for HttpServlet's {@code getLastModified} method.
    * Can simply return -1 if there's no support in the handler class.
    * @param request current HTTP request
    * @param handler handler to use
    * @return the lastModified value for the given handler
    * @see javax.servlet.http.HttpServlet#getLastModified
    * @see org.springframework.web.servlet.mvc.LastModified#getLastModified
    */
   long getLastModified(HttpServletRequest request, Object handler);

}
```

其中一个实现类`SimpleControllerHandlerAdapter`

```java
package org.springframework.web.servlet.mvc;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.HandlerAdapter;
import org.springframework.web.servlet.ModelAndView;

/**
 * Adapter to use the plain {@link Controller} workflow interface with
 * the generic {@link org.springframework.web.servlet.DispatcherServlet}.
 * Supports handlers that implement the {@link LastModified} interface.
 * <p>This is an SPI class, not used directly by application code.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @see org.springframework.web.servlet.DispatcherServlet
 * @see Controller
 * @see LastModified
 * @see HttpRequestHandlerAdapter
 */
public class SimpleControllerHandlerAdapter implements HandlerAdapter {
   @Override
   public boolean supports(Object handler) {
      return (handler instanceof Controller);
   }
   @Override
   public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
      return ((Controller) handler).handleRequest(request, response);
   }

   @Override
   public long getLastModified(HttpServletRequest request, Object handler) {
      if (handler instanceof LastModified) {
         return ((LastModified) handler).getLastModified(request);
      }
      return -1L;
   }

}
```

一般默认使用的是 RequestMappingHandlerAdapter ,在其父类中有这样一句设置Order

`private int order = Ordered.LOWEST_PRECEDENCE;`

![image-20181219180136155](https://ws1.sinaimg.cn/large/006tNbRwly1fyc7lnlrzmj318o0u042e.jpg)

> HandlerAdaptger implementators may implement the Order，使用时候是遍历 handlerAdapters ，调用supports判断直接返回（除了设置的，默认的实现有3个）。注意Order是越低优先级越高的。

###### 4.3.3 HandlerExceptionResolver

​	根据异常设置ModelAndView，之后交给render方法去渲染，这里要注意它是在render之前工作，所以只作用于解析对请求做处理过程中的产生的异常。

```java
package org.springframework.web.servlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * Interface to be implemented by objects than can resolve exceptions thrown
 * during handler mapping or execution, in the typical case to error views.
 * Implementors are typically registered as beans in the application context.
 *
 * <p>Error views are analogous to the error page JSPs, but can be used with
 * any kind of exception including any checked exception, with potentially
 * fine-granular mappings for specific handlers.
 *
 * @author Juergen Hoeller
 * @since 22.11.2003
 */
public interface HandlerExceptionResolver {
   /**
    * Try to resolve the given exception that got thrown during on handler execution,
    * returning a ModelAndView that represents a specific error page if appropriate.
    * <p>The returned ModelAndView may be {@linkplain ModelAndView#isEmpty() empty}
    * to indicate that the exception has been resolved successfully but that no view
    * should be rendered, for instance by setting a status code.
    * @param request current HTTP request
    * @param response current HTTP response
    * @param handler the executed handler, or {@code null} if none chosen at the
    * time of the exception (for example, if multipart resolution failed)
    * @param ex the exception that got thrown during handler execution
    * @return a corresponding ModelAndView to forward to,
    * or {@code null} for default processing
    */
   ModelAndView resolveException(
         HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex);

}
```

其实现SimpleMappingExceptionResolver 可以通过扩展配置，设置自定义类型

![image-20181219173520333](https://ws4.sinaimg.cn/large/006tNbRwly1fyc6u9xp9nj30zn0u0ag3.jpg)

> 可以配合定制自定义Exception，前端统一展示等

###### 4.3.4 ViewResolver

​	通过viewName来找到 对应的View，View是用来渲染页面的，主要解决用什么模版和用什么规则填入参数

```java
package org.springframework.web.servlet;
import java.util.Locale;

/**
 * Interface to be implemented by objects that can resolve views by name.
 *
 * <p>View state doesn't change during the running of the application,
 * so implementations are free to cache views.
 *
 * <p>Implementations are encouraged to support internationalization,
 * i.e. localized view resolution.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @see org.springframework.web.servlet.view.InternalResourceViewResolver
 * @see org.springframework.web.servlet.view.ResourceBundleViewResolver
 * @see org.springframework.web.servlet.view.XmlViewResolver
 */
public interface ViewResolver {

   /**
    * Resolve the given view by name.
    * <p>Note: To allow for ViewResolver chaining, a ViewResolver should
    * return {@code null} if a view with the given name is not defined in it.
    * However, this is not required: Some ViewResolvers will always attempt
    * to build View objects with the given name, unable to return {@code null}
    * (rather throwing an exception when View creation failed).
    * @param viewName name of the view to resolve
    * @param locale Locale in which to resolve the view.
    * ViewResolvers that support internationalization should respect this.
    * @return the View object, or {@code null} if not found
    * (optional, to allow for ViewResolver chaining)
    * @throws Exception if the view cannot be resolved
    * (typically in case of problems creating an actual View object)
    */
   View resolveViewName(String viewName, Locale locale) throws Exception;

}
```

常用的jsp解析

![image-20181219195241323](https://ws1.sinaimg.cn/large/006tNbRwly1fycat4mrudj310q0u043a.jpg)

按照类图可以把继承`AbstractCachingViewResolver`分成两类

- `ResourceBundleViewResolver`  可以同时支持多种类型的视图，需要将每一个视图名和对应的视图类型配置到相应的properties文件中,该类的注释中，解释了其用法,

```java
 /* <p>This {@code ViewResolver} supports localized view definitions,
 * using the default support of {@link java.util.PropertyResourceBundle}.
 * For example, the basename "views" will be resolved as class path resources
 * "views_de_AT.properties", "views_de.properties", "views.properties" -
 * for a given Locale "de_AT"./
```

- `UrlBasedViewResolver`系列的解析器都是对单一视图进行解析的，只需找到模版就行，例如

`InternalResourceViewResolver` 是专门用来解析jsp，`FreeMarkerViewResolver`只针对FreeMarker

> 相关扩展ExecelViewResolver、SimpleMapViewResolver
>
> 默认实现是 InternalResourceViewResolver

###### 4.3.5 RequestToViewNameTranslator

比较简单，当handler处理完后，没有View和ViewName的会调用该方法。

```java
/**
 * Do we need view name translation?
 */
private void applyDefaultViewName(HttpServletRequest request, ModelAndView mv) throws Exception {
   if (mv != null && !mv.hasView()) {
      mv.setViewName(getDefaultViewName(request));
   }
}
/**
 * Translate the supplied request into a default view name.
 * @param request current HTTP servlet request
 * @return the view name (or {@code null} if no default found)
 * @throws Exception if view name translation failed
 */
protected String getDefaultViewName(HttpServletRequest request) throws Exception {
    return this.viewNameTranslator.getViewName(request);
}
```

> ​	默认实现是RequestToViewNameTranslator

###### 4.3.6 LocaleResolver

​	用于从request中解析出Locale,默认实现是

```java
package org.springframework.web.servlet.i18n;

import java.util.Locale;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.LocaleResolver;

/**
 * {@link LocaleResolver} implementation that simply uses the primary locale
 * specified in the "accept-language" header of the HTTP request (that is,
 * the locale sent by the client browser, normally that of the client's OS).
 *
 * <p>Note: Does not support {@code setLocale}, since the accept header
 * can only be changed through changing the client's locale settings.
 *
 * @author Juergen Hoeller
 * @since 27.02.2003
 * @see javax.servlet.http.HttpServletRequest#getLocale()
 */
public class AcceptHeaderLocaleResolver implements LocaleResolver {

   public Locale resolveLocale(HttpServletRequest request) {
      return request.getLocale();
   }

   public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
      throw new UnsupportedOperationException(
            "Cannot change HTTP accept header - use a different locale resolution strategy");
   }

}
```

​	在DispatcherServlet#doSerive()方法中， `request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);` 放进request中，这里除了视图解析的时候需要用到，在使用国际化主题的时候也会用到

> 切换相关，可查看 LocaleChangeInterceptor

###### 4.3.7 ThemeResolver

​	SpringMVC中和主题相关的类有如下

- ThemeResolver 作用是从request中解析出主题名,默认实现FixedThemeResolver
- ThemeSource 根据主题名找到对应的主题 ，默认使用WebApplicationContext
- Theme 就是主题，包含了具体资源

```java
package org.springframework.web.servlet.theme;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * {@link org.springframework.web.servlet.ThemeResolver} implementation
 * that simply uses a fixed theme. The fixed name can be defined via
 * the "defaultThemeName" property; out of the box, it is "theme".
 *
 * <p>Note: Does not support {@code setThemeName}, as the fixed theme
 * cannot be changed.
 *
 * @author Jean-Pierre Pawlak
 * @author Juergen Hoeller
 * @since 17.06.2003
 * @see #setDefaultThemeName
 */
public class FixedThemeResolver extends AbstractThemeResolver {
   @Override
   public String resolveThemeName(HttpServletRequest request) {
      return getDefaultThemeName();
   }

   @Override
   public void setThemeName(HttpServletRequest request, HttpServletResponse response, String themeName) {
      throw new UnsupportedOperationException("Cannot change theme - use a different theme resolution strategy");
   }

}
```

> 切换相关，可查看 ThemeChangeInterceptor

###### 4.3.8 MultipartResolver

​	用于处理文件上传，注意这个组件在DispatcherServlet.properties 中 没有默认实现,首先是接口实现

```java
package org.springframework.web.multipart;

import javax.servlet.http.HttpServletRequest;

/**
 * A strategy interface for multipart file upload resolution in accordance
 * with <a href="http://www.ietf.org/rfc/rfc1867.txt">RFC 1867</a>.
 * Implementations are typically usable both within an application context
 * and standalone.
 *
 * <p>There are two concrete implementations included in Spring, as of Spring 3.1:
 * <ul>
 * <li>{@link org.springframework.web.multipart.commons.CommonsMultipartResolver} for Jakarta Commons FileUpload
 * <li>{@link org.springframework.web.multipart.support.StandardServletMultipartResolver} for Servlet 3.0 Part API
 * </ul>
 *
 * <p>There is no default resolver implementation used for Spring
 * {@link org.springframework.web.servlet.DispatcherServlet DispatcherServlets},
 * as an application might choose to parse its multipart requests itself. To define
 * an implementation, create a bean with the id "multipartResolver" in a
 * {@link org.springframework.web.servlet.DispatcherServlet DispatcherServlet's}
 * application context. Such a resolver gets applied to all requests handled
 * by that {@link org.springframework.web.servlet.DispatcherServlet}.
 *
 * <p>If a {@link org.springframework.web.servlet.DispatcherServlet} detects
 * a multipart request, it will resolve it via the configured
 * {@link MultipartResolver} and pass on a
 * wrapped {@link javax.servlet.http.HttpServletRequest}.
 * Controllers can then cast their given request to the
 * {@link MultipartHttpServletRequest}
 * interface, which permits access to any
 * {@link MultipartFile MultipartFiles}.
 * Note that this cast is only supported in case of an actual multipart request.
 *
 * <pre class="code">
 * public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {
 *   MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
 *   MultipartFile multipartFile = multipartRequest.getFile("image");
 *   ...
 * }</pre>
 *
 * Instead of direct access, command or form controllers can register a
 * {@link org.springframework.web.multipart.support.ByteArrayMultipartFileEditor}
 * or {@link org.springframework.web.multipart.support.StringMultipartFileEditor}
 * with their data binder, to automatically apply multipart content to command
 * bean properties.
 *
 * <p>As an alternative to using a
 * {@link MultipartResolver} with a
 * {@link org.springframework.web.servlet.DispatcherServlet},
 * a {@link org.springframework.web.multipart.support.MultipartFilter} can be
 * registered in {@code web.xml}. It will delegate to a corresponding
 * {@link MultipartResolver} bean in the root
 * application context. This is mainly intended for applications that do not
 * use Spring's own web MVC framework.
 *
 * <p>Note: There is hardly ever a need to access the
 * {@link MultipartResolver} itself
 * from application code. It will simply do its work behind the scenes,
 * making
 * {@link MultipartHttpServletRequest MultipartHttpServletRequests}
 * available to controllers.
 *
 * @author Juergen Hoeller
 * @author Trevor D. Cook
 * @since 29.09.2003
 * @see MultipartHttpServletRequest
 * @see MultipartFile
 * @see org.springframework.web.multipart.commons.CommonsMultipartResolver
 * @see org.springframework.web.multipart.support.ByteArrayMultipartFileEditor
 * @see org.springframework.web.multipart.support.StringMultipartFileEditor
 * @see org.springframework.web.servlet.DispatcherServlet
 */
public interface MultipartResolver {

   /**
    * Determine if the given request contains multipart content.
    * <p>Will typically check for content type "multipart/form-data", but the actually
    * accepted requests might depend on the capabilities of the resolver implementation.
    * @param request the servlet request to be evaluated
    * @return whether the request contains multipart content
    */
   boolean isMultipart(HttpServletRequest request);

   /**
    * Parse the given HTTP request into multipart files and parameters,
    * and wrap the request inside a
    * {@link org.springframework.web.multipart.MultipartHttpServletRequest} object
    * that provides access to file descriptors and makes contained
    * parameters accessible via the standard ServletRequest methods.
    * @param request the servlet request to wrap (must be of a multipart content type)
    * @return the wrapped servlet request
    * @throws MultipartException if the servlet request is not multipart, or if
    * implementation-specific problems are encountered (such as exceeding file size limits)
    * @see MultipartHttpServletRequest#getFile
    * @see MultipartHttpServletRequest#getFileNames
    * @see MultipartHttpServletRequest#getFileMap
    * @see javax.servlet.http.HttpServletRequest#getParameter
    * @see javax.servlet.http.HttpServletRequest#getParameterNames
    * @see javax.servlet.http.HttpServletRequest#getParameterMap
    */
   MultipartHttpServletRequest resolveMultipart(HttpServletRequest request) throws MultipartException;

   /**
    * Cleanup any resources used for the multipart handling,
    * like a storage for the uploaded files.
    * @param request the request to cleanup resources for
    */
   void cleanupMultipart(MultipartHttpServletRequest request);

}
```

常用实现CommonsMultipartResolver

![image-20181219214146380](https://ws1.sinaimg.cn/large/006tNbRwly1fycdyq4kmxj31ta0u0ahy.jpg)



> 对于上传类型判断是 multipart/form-data

###### 4.3.9 FlashMapManager

​	FlashMap主要用于redirect中传递参数，FlashMapManage是用来管理FlashMap的

```java
package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * A strategy interface for retrieving and saving FlashMap instances.
 * See {@link FlashMap} for a general overview of flash attributes.
 *
 * @author Rossen Stoyanchev
 * @since 3.1
 * @see FlashMap
 */
public interface FlashMapManager {

   /**
    * Find a FlashMap saved by a previous request that matches to the current
    * request, remove it from underlying storage, and also remove other
    * expired FlashMap instances.
    * <p>This method is invoked in the beginning of every request in contrast
    * to {@link #saveOutputFlashMap}, which is invoked only when there are
    * flash attributes to be saved - i.e. before a redirect.
    * @param request the current request
    * @param response the current response
    * @return a FlashMap matching the current request or {@code null}
    */
   FlashMap retrieveAndUpdate(HttpServletRequest request, HttpServletResponse response);

   /**
    * Save the given FlashMap, in some underlying storage and set the start
    * of its expiration period.
    * <p><strong>NOTE:</strong> Invoke this method prior to a redirect in order
    * to allow saving the FlashMap in the HTTP session or in a response
    * cookie before the response is committed.
    * @param flashMap the FlashMap to save
    * @param request the current request
    * @param response the current response
    */
   void saveOutputFlashMap(FlashMap flashMap, HttpServletRequest request, HttpServletResponse response);

}
```

​	两个方法，分别是retrieveAndUpdate 用于恢复参数，saveOutputFlashMap用于保存参数

类图也比较简单

![image-20181219214559988](https://ws4.sinaimg.cn/large/006tNbRwly1fyce346qynj30zc0u0q9t.jpg)

> 默认实现是SessionFlashMapManager

整个redirects 的参数通过FlashMap传递的过程分三步

1. 首先，RequestMappingHandlerAapter会将其设置到`outputFlashMap`中，如果是redirect类型的返回类型值（将需要传递的参数设置到`outputFlashMap`中，也可以是RedirectAttributes类型的参数中）
2. 在RedirectView中会调用`#saveOutputFlashMap()`方法，将`outputFlashMap`中的参数设置到Session
3. 请求redirect后，DispatcherServlet会调有你`#retrieveAndUpdate()`方法从Session中获取`inputFlashMap`并设置到Request的属性中备用，同时从Session中删除

##### 4.4 处理请求

###### 4.4.1 FrameworkServlet

​	Servlet的处理过程，首先是从Servlet接口的service，然后在HttpServlet的service方法中根据请求的类型不同将请求路由到了doGet、doHead等方法。

​	FrameworkServlet中重写了doGet等方法。将请求集中到processRequest方法进行统一处理

```java
/**
 * Override the parent class implementation in order to intercept PATCH
 * requests.
 */
@Override
protected void service(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

    String method = request.getMethod();
    if (method.equalsIgnoreCase(RequestMethod.PATCH.name())) {
        processRequest(request, response);
    }
    else {
        super.service(request, response);
    }
}
/**
 * Delegate GET requests to processRequest/doService.
 * <p>Will also be invoked by HttpServlet's default implementation of {@code doHead},
 * with a {@code NoBodyResponse} that just captures the content length.
 * @see #doService
 * @see #doHead
 */
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

    processRequest(request, response);
}
```

​	processRequest这个方法主要做了两件事情(当然还有doService，和对异步请求对处理)

- 对LocaleContext（获取locale）和RequestAttributes(管理request和session属性)的设置及恢复
- 处理完后发布ServletRequestHandledEvent消息

LocaleContextHolder 、RequestContextHolder 

```java
public abstract class LocaleContextHolder {
    private static final ThreadLocal localeContextHolder = new NamedThreadLocal("Locale context");
    private static final ThreadLocal inheritableLocaleContextHolder = new NamedInheritableThreadLocal("Locale context");
    ....
}
```

 这里比较有意思的地方是这是个抽象类，里面的方法实现都是静态方法，只能调用不能实例化。RequestContextHolder的实现类似，封装的是ServletRequestAttributes

> 这里之所以需要对LocaleContext和RequestAttributes恢复，是因为在Servlet外面可能还有别等操作，例如Filter（Spring-MVC的HandlerInterceptor）,为了不影响那些操作。

###### 4.4.2 DispatcherServlet

​	DispatcherServlet是FrameworkServlet的子类，实现了doService方法

```java
/**
 * Exposes the DispatcherServlet-specific request attributes and delegates to {@link #doDispatch}
 * for the actual dispatching.
 */
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) 
    throws Exception {
    // Keep a snapshot of the request attributes in case of an include,
    // to be able to restore the original attributes after the include.
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<String, Object>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // Make framework objects available to handlers and view objects.
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    //Pass paramter use to Redirect
    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
    if (inputFlashMap != null) {
        request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
    }
    request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
    request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

    try {
        doDispatch(request, response);
    }
    finally {
        if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            return;
        }
        // Restore the original attribute snapshot, in case of an include.
        if (attributesSnapshot != null) {
            restoreAttributesAfterInclude(request, attributesSnapshot);
        }
    }
}
```

[^]: 移除了部分日志代码

在doDispatch之前做了

- 判断是否include请求，是的话保存快照（attributesSnapshot）
- 为request设置默认的属性，在后面的handlers和view中需要使用

下面是doDispatch的方法的实现，主要的功能是 Process the actual dispatching to the handler.

```java
/**
 * Process the actual dispatching to the handler.
 * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
 * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
 * to find the first that supports the handler class.
 * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
 * themselves to decide which methods are acceptable.
 * @param request current HTTP request
 * @param response current HTTP response
 * @throws Exception in case of any kind of processing failure
 */
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) 
    throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = processedRequest != request;

            // Determine handler for the current request.
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (logger.isDebugEnabled()) {
                    logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                }
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }

            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            try {
                // Actually invoke the handler.
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            }
            finally {
                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }
            }
            applyDefaultViewName(request, mv);
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }catch (Exception ex) {
            dispatchException = ex;
        }
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }catch (Error err) {
        triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
    }finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            return;
        }
        // Clean up any resources used by a multipart request.
        if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
        }
    }
}
```

整个处理过程最核心的代码只有4句

- 根据request请求在HandlerMapping找到对应的Handler 
- 根据Handler找到对应的HandlerAdapter
- 用HandlerAdapter处理Handler,返回一个ModelAndView(Controller就是在这个地方执行)
- 根据ModelAndView的信息（或异常处理），通过ViewResolver找到View,并根据Model中的数据渲染输出后给用户

整个处理流程如下

![image-20181219111943655](https://ws4.sinaimg.cn/large/006tNbRwgy1fybvzj6p72j30u00ynju2.jpg)