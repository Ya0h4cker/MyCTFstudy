# CommonCollections6-Shiro

## Gadget Chain

    HashMap.readObject()
        HashMap.hash()
            TiedMapEntry.hashCode()
            TiedMapEntry.getValue()
                LazyMap.get()
                    InvokerTransformer.transform()
                        TemplatesImpl.newTransformer()
                            TemplatesImpl.getTransletInstance() 
                                TemplatesImpl.defineTransletClasses()
                                    ClassLoder.defineClass()

## Note

* Because the `Shiro` framework overloads the `readObject` method, it calls `ClassLoader.loadClass()` method to load non-java native classes dynamically. Compared with `Class.forName()` method, the `ClassLoader.loadClass()` method doesn't support the array type. Therefore, the `ChainedTransformer` class can't be exploited and we use the `TemplatesImpl` chain instead.
