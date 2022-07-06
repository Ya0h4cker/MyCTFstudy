# CommonCollections3

## Gadget Chain

    AnnotationInvocationHandler.readObject()
        Map(Proxy).entrySet()
            AnnotationInvocationHandler.invoke()
                LazyMap.get()
                ChainedTransformer.transform()
                    ConstantTransformer.transform()
                        TrAXFilter.class
                    InstantiateTransformer.transform()
                        TrAXFilter.TrAXFilter()
                            TemplatesImpl.newTransformer()
                                TemplatesImpl.getTransletInstance() 
                                    TemplatesImpl.defineTransletClasses()
                                        ClassLoder.defineClass()

## Analysis

### Key: TrAXFilter

`TrAXFilter`: The constructor of the `TrAXFilter` class will call `TemplatesImpl.newTransformer()` method, but the `TrAXFilter` class isn't serializable.

`InstantiateTransformer`: a kind of `Transformer` class, calling the constructor of a input object, and return the instantiation result.

## Note

* Because the `TrAXFilter` class isn't serializable, we can use the `TrAXFilter.class` object as the input of `InstantiateTransformer` object instead. This `CommonCollections3` chain is suitable for the situation where the `InvokerTransformer` class is banned.
