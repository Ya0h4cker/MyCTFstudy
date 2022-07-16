# CommonCollections6

## Gadget Chain

    HashSet.readObject()
        HashMap.put()
        HashMap.hash()
            TiedMapEntry.hashCode()
            TiedMapEntry.getValue()
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

    HashMap.readObject()
        HashMap.hash()
            TiedMapEntry.hashCode()
            TiedMapEntry.getValue()
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

## Analysis

### Key: TiedMapEntry

`TiedMapEntry`: a public `Entry` of `Map` class.

The `LazyMap` object is as `TiedMapEntry.map`. When method `TiedMapEntry.hashCode()`, `TiedMapEntry.toString()` or `TiedMapEntry.equals()` is called, the `TiedMapEntry.getValue()` method is called. In `TiedMapEntry.getValue()` method, invoke `TiedMapEntry.map.get()` method, and the parameter is `TiedMapEntry.key`.

The `CommonCollections6` uses the `TiedMapEntry` class to bypass limitations in `jdk>=8u71` and continues to  call `LazyMap.get()` method.

### Source: HashMap

The `TiedMapEntry` object is as the key of `HashMap` object. As the same as the `URLDNS` chain, when the `HashMap.readObject()` method is called, method `hash(Key)` is called. Therefore, the `TiedMapEntry.hashCode()` method is invoked.

### Another source: HashSet

The `TiedMapEntry` object is as the element of `HashSet` object. When the `HashSet.readObject()` method is called, a `HashMap` object is created. Then each element of `HashSet` object is traversed and added into the `HashMap` object by calling `HashMap.put()` method. In the `HashMap.put()` method, method `hash(Key)` is called.

## Note

- The another parameter of the constructor function of the `TiedMapEntry` class is the `TiedMapEntry.key`, so it is also controllable.

- To make the `TiedMapEntry` object become the key of `HashMap` object, we need invoke the `HashMap.put()` method. However, it will cause the gadget chain to be triggered in advance. Also the method `LazyMap.get()` is called in advance, and `LazyMap.put()` method will be invoked, which will cause that the key of `LazyMap` object is put, so the condition for calling the `LazyMap.factory.transform()` method can't be satisfied in the deserialization process. Therefore, we can use a trick, that is create a `LazyMap` object with a fake `Transformer` object firstly, like `new ConstantTransformer(1)`, then before serialization, modify it to the constructed `ChainedTransformer` by reflections. In addition, remove the key in the `LazyMap` before serialization by invoking `LazyMap.remove()` method.
