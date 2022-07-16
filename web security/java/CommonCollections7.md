# CommonCollections7

## Gadget Chain

    Hashtable.readObject()
    Hashtable.reconstitutionPut()
        AbstractMapDecorator.equals()
        AbstractMap.equals()
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

### Source: Hashtable

The `LazyMap` object is as the key of the `Hashtable` object. In method `Hashtable.readObject()`, the method `Hashtable.reconstitutionPut()` is called. If the hash of the new key is equal with the hash of the key that already exists in the `Hashtable` object, then `equals()` method of the key that already exists in the `Hashtable` object will be invoked to judge whether the two keys are equal. But the `LazyMap` class doesn't have `equals()` method, so this method of its parent class `AbstractMap` will be called. In this method `AbstractMap.equals()`, the method `get()` of the new key will be called, which is `LazyMap.get()` and its parameter is the key of the `LazyMap` class that already exists in the `Hashtable` object.

### Note

- The condition that the hash values of the two keys of the `Hashtable` object are same, should be satisfied. Therefore, put two `LazyMap` objects as the key of the `Hashtable` object and their values are same while the hash of their keys are same. Here, the keys can be `"yy"` and `"zZ"`. As for why not keeping their keys are same, because only the input key doesn't exist in the `LazyMap` object, can `LazyMap.get()` call `LazyMap.factory.transform()` method.

- When invoking `Hashtable.put()` method, the `equals()` method will also be called. So we need to avoid the gadget chain to be triggered in advance by setting a fake `ChainedTransformer` object. Then modity it to true `ChainedTransformer` object before serialization by reflections. What's more, the `LazyMap.get()` method is invoked in advance, which causes a key to be added to the `LazyMap` object. This make the condition for `AbstractMap.equals()` to call `LazyMap.get()` unsatisfied. Therefore, we need to remove the key of the first `LazyMap` object from the second `LazyMap` object before serialization by reflections.
