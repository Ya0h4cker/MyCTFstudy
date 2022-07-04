# CommonCollections1

## Gadget Chain

    AnnotationInvocationHandler.readObject()
        Map(Proxy).entrySet()
            AnnotationInvocationHandler.invoke()
                LazyMap.get()
                    ChainedTransformer.transform()
                        ConstantTransformer.transform()
                            Runtime.class
                        InvokerTransformer.transform()
                            Method.invoke()
                                Class.getMethod()
                        InvokerTransformer.transform()
                            Method.invoke()
                                Runtime.getRuntime()
                        InvokerTransformer.transform()
                            Method.invoke()
                                Runtime.exec()

or

    AnnotationInvocationHandler.readObject()
        TransformedMap.Entry.setValue()
            TransformedMap.checkSetValue()
                ChainedTransformer.transform()
                    ConstantTransformer.transform()
                        Runtime.class
                    InvokerTransformer.transform()
                        Method.invoke()
                            Class.getMethod()
                    InvokerTransformer.transform()
                        Method.invoke()
                            Runtime.getRuntime()
                    InvokerTransformer.transform()
                        Method.invoke()
                            Runtime.exec()

## Analysis

### Sink: Transformer

`Transformer`: is a interface which transforms the input object into some output object by calling `Transformer.transform()` method.

`ConstantTransformer`: return a object itself.

`InvokerTransformer`: call an arbitrary method of a object by reflections and return the result.

`ChainedTransformer`: call each `Transformer` in `Transformer[]` in turns like a chain, and the ouput of previous `Transformer` is the input of next `Transformer`.

### Key: TransformedMap

`TransformedMap`: is a object which decorates the Java standard class `Map` with key and value `Transformer` objects and generate a decorated `Map`.

The object `Transformer` is as the `TransformedMap.valueTransformer`. When `TransformedMap.Entry.setValue()` method is called, the method `TransformedMap.checkSetValue()` is called and `TransformedMap.valueTransformer.transform()` method is actually called in this method.

### Another Key: LazyMap

`LazyMap`: is a object which decorates the Java standard class `Map` with a `Factory` object, and the `Factory` object is actually a `Transformer` class.

The object `Transformer` is as the `LazyMap.factory`. When `LazyMap.get()` method is called and there is no such a key in `LazyMap`, the method `LazyMap.factory.transform()` will be called.

### Source: AnnotationInvocationHandler

`AnnotationInvocationHandler`: is a kind of `handler` class for Java dynamic proxy.

The decorated `Map` is as the `AnnotationInvocationHandler.memberValues`.

In method `AnnotationInvocationHandler.readObject()`, use `AnnotationInvocationHandler.memberValues.entrySet()` to traverse entries. Then for each entry, `AnnotationInvocationHandler.memberValues.Entry.setValue()` method is called.

As we mentioned, `AnnotationInvocationHandler` class is a kind of `handler` class for Java dynamic proxy. So an arbitraty method of proxied class is called means that the `handler.invoke()` method is called. Coincidentally, in method `AnnotationInvocationHandler.invoke()`, the method `AnnotationInvocationHandler.memberValues.get()` is called.

Therefore, we can use a `Proxy` class with the constructed `AnnotationInvocationHandler` class to proxy a `Map` object. Then the proxied `Map` object is as `AnnotationInvocationHandler.memberValues` of another new `AnnotationInvocationHandler` object. When the method `AnnotationInvocationHandler.readObject()` of the new `AnnotationInvocationHandler` object is called, method `AnnotationInvocationHandler.memberValues.entrySet()` is called, which means a method of the proxied `Map` object is called.

## Note

* The `Runtime` class can't be serialized, so use `Runtime.class` object and invoke `Runtime,getRuntime()` method instead .

* The `AnnotationInvocationHandler` class is a Java internal class, we need to construct an object by reflections.

* The another parameter of the constructor function of `AnnotationInvocationHandler` class is an `Annotation` class except for `Map` class. In method `AnnotationInvocationHandler.readObject()`, a condition that the key of the decorated `Map` object is a method name of the `Annotation` object
, should be satisfied to excute method `AnnotationInvocationHandler.memberValues.Entry.setValue()`. So use `Retention` or `Target Annotation` object.

* In method `AnnotationInvocationHandler.invoke()`, a condition that the method of the proxied object is a non-parametric method should be satisfied to excute method `AnnotationInvocationHandler.memberValues.get()`.

* Patch for `jdk>=8u71`: in method `AnnotationInvocationHandler.readObject()`, invoke `entrySet()` of the `ObjectInputStream.readFields().get("memberValues", null)` object instead invoke `AnnotationInvocationHandler.memberValues.entrySet()` directly; Delete `AnnotationInvocationHandler.memberValues.Entry.setValue()` call instead use a new `LinkedHashMap` object to store the key and value. The patch can prevent `TransformedMap.Entry.setValue()` and `LazyMap.get()` methods being called. However, the `CommonCollections6` chain can bypass.
