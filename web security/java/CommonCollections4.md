# CommonCollections4

## Gadget Chain

    PriorityQueue.readObject()
        TransformingComparator.compare()
            ChainedTransformer.transform()
                ConstantTransformer.transform()
                    TrAXFilter.class
                InstantiateTransformer.transform()
                    TrAXFilter.TrAXFilter()
                        TemplatesImpl.newTransformer()
                            TemplatesImpl.getTransletInstance()
                                TemplatesImpl.defineTransletClasses()
                                    ClassLoder.defineClass()

## Note

- Combination of `CommonCollections2` chain and `CommonCollections3` chain. This `CommonCollections4` chain is suitable for the situation where the `InvokerTransformer` class is banned and only suitable for `commons-collections4` version.
