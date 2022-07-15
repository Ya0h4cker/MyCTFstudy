# Context acquisition

The context acquisition is a basic step of building a memory shell. Because the dynamic registration of the `Filter` or the `Servlet` is based on the context operation.

The context to obtain is the `StandardContext` object in the `Tomcat`, the `ServletContext` object in the `WebLogic` or the `WebApplicationContext` object in the `SpringMVC` and `SpringBoot`.

## The Request object already owned

This situation usually occurs in the JSP file. We can obtain the `Request` object directly in the JSP file. So the `StandardContext` object can be found through the `Request` object.

    javax.servlet.ServletContext servletContext = request.getServletContext();
 
    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);
 
    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

## The Request object not owned yet

In this situation, there are two ways, one is to find the `Request` object and get the `StandardContext` object through it, another is to get the `StandardContext` object directly from the thread.

### ContextClassLoader

This method is only applicable to `Tomcat 8` and `Tomcat 9`.

    org.apache.catalina.loader.WebappClassLoaderBase webappClassLoaderBase =(org.apache.catalina.loader.WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
 
    StandardContext standardContext = (StandardContext)webappClassLoaderBase.getResources().getContext();
 
    System.out.println(standardContext);
