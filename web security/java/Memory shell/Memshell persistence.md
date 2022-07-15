# Memshell persistence

The memory shell will not exist after the web server is restarted. Therefore, for the persistence of memory shell, it is still necessary to leave the file.

## @Servlet/@Filter annotation

The `Tomcat` server can register the `Servlet` or the `Filter` through the `@Servlet` and `@Filter` annotations at startup.

So write an evil `Servlet` or `Filter` class with the `@Servlet` or `@Filter` annotation.

## addShutdownHook()

The `addShutdownHook()` method can hook a thread, which will be executed before the web server is shut down.

So construct a thread, which writes the jar package that registers a memory shell, and execute the jar package after the JVM is restarted.

## ServletContainerInitializer

When the web server is initialized, the `onStartup()` method of the `ServletContainerInitializer` interface will be called to register the `Servlet` or `Filter`. And the `ServletContainerInitializer` class is registered through `META-INF/services/javax.servlet.ServletContainerInitializer` file according to the SPI mechanism.

So construct a new `ServletContainerInitializer` class and write code in the `onStartup()` method, which registers an evil `Servlet` or `Filter`. Then write the corresponding file according to the SPI mechanism and package them to a jar package.

## startUpClass

After the `WebLogic` server is started, the specified class will be executed according to the parameters of the `startUpClass` class.

## JarFile

The `JarFile` is a java class library in `java.util` package, which can modify the jar package even when the JVM is running. For example, modify the `tomcat-websocket.jar`.
