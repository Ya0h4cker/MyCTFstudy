# CommonCollections3

## Gadget Chain

    AnnotationInvocationHandler.readObject()
        Map(Proxy).entrySet()
            AnnotationInvocationHandler.invoke()
                LazyMap.get()
                ChainedTransformer.transform()
                    ConstantTransformer.transform()
                    InstantiateTransformer.transform()
                        TrAXFilter.TrAXFilter()
                        TemplatesImpl.newTransformer()
                            TemplatesImpl.getTransletInstance() 
                                TemplatesImpl.defineTransletClasses()
                                    ClassLoder.defineClass()

## Analysis

### Key: TrAXFilter

`TrAXFilter`: a class whose constructor will call `TemplatesImpl.newTransformer()` method, but it isn't serializable.

`InstantiateTransformer`: a kind of `Transformer` class, calling the constructor of a input object, and return the instantiation result.

## Note

* Because the `TrAXFilter` class isn't serializable, we can use the `TrAXFilter.class` object as the input of `InstantiateTransformer` object instead. This `CommonCollections3` chain is suitable for the situation where the `InvokerTransformer` class is banned.
