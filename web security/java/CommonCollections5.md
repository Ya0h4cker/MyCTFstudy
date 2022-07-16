# CommonCollections5

## Gadget Chain

    BadAttributeValueExpException.readObject()
        TiedMapEntry.toString()
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

### Source: BadAttributeValueExpException

The `TiedMapEntry` object is as `BadAttributeValueExpException.value`. In method `BadAttributeValueExpException.readObject()`, `BadAttributeValueExpException.value.toString()` method is called.

## Note

- When instantiating a `BadAttributeValueExpException` object, `BadAttributeValueExpException.value.toString()` method will also be called. Therefore, we need to avoid triggering the gadget chain in advance by reflections.
