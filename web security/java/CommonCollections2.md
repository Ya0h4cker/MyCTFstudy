# CommonCollections2

## Gadget Chain

    PriorityQueue.readObject()
        TransformingComparator.compare()
            InvokerTransformer.transform()
                TemplatesImpl.newTransformer()
                    TemplatesImpl.getTransletInstance() 
                        TemplatesImpl.defineTransletClasses()
                            ClassLoder.defineClass()

## Analysis

### Sink: TemplatesImpl

In `TemplatesImpl.getTransletInstance()` method, `TemplatesImpl.defineTransletClasses()` method is called, which invokes `ClassLoder.defineClass()` method to dynamically load class bytecodes for each `TemplatesImpl._bytecodes[]`. These loaded classes are saved into `TemplatesImpl._class[]` array. Then these classes are instantiated by calling `TemplatesImpl._class[].newInstance()`. And in the public method `TemplatesImpl.newTransformer()`, the `TemplatesImpl.getTransletInstance()` method is called.

### Key: TransformingComparator

`TransformingComparator`: a comparator class for comparing two objects after transforming.

The `Transformer` object is as `TransformingComparator.transformer`. When the method `TransformingComparator.compare()` is called, `TransformingComparator.transformer.transform()` method will be called with a compared object as paramter.

### Source: PriorityQueue

The `TransformingComparator` object is as `PriorityQueue.comparetor`. And the `TemplatesImpl` object is as the first added element of the `PriorityQueue` object. When method `PriorityQueue.readObject()` is called, `PriorityQueue.heapify()` method is called. Then the `PriorityQueue.siftDown()` method is called, at last call `PriorityQueue.siftDownUsingComparator()` method to call `PriorityQueue.comparetor.compare()` method for each group of parent node and two child nodes in `PriorityQueue` object. Therefore, `TransformingComparator.transformer.transform(TemplatesImpl)` is called.

## Note

* For calling `PriorityQueue.siftDown()` method in the `PriorityQueue.heapify()` method, a condition that the size of the `PriorityQueue` object is larger than `2` should be satisfied by using `PriorityQueue.add()` method. However, the `PriorityQueue.add()` method will call `PriorityQueue.offer()` method. Then `PriorityQueue.siftUp()` method is called, which leads `PriorityQueue.comparetor.compare()` method to be invoked. Therefore, we need to avoid triggering the gadget chain in advance by reflections.

* To excute `TemplatesImpl.defineTransletClasses()` method, a condition that the filed of `TemplatesImpl._class` is null and the fields of `TemplatesImpl._name` and `TemplatesImpl._tfactory` aren't null, should be satisfied. It can be achieved by reflections except for `TemplatesImpl._tfactory` field. Because the `TemplatesImpl._tfactory` field is decorated by `transient` keyword, so it will not be serialized. However, it doesn't matter, because the `TemplatesImpl._tfactory` field will be created in in Java native `TemplatesImpl.readObject()` method. In `TemplatesImpl.defineTransletClasses()` method, the loaded class should inherit the `AbstractTranslet` abstract class, and implement corresponding methods for loading class correctlty.

* Only suite for `commons-collections4` version, because the `TransformingComparator` class isn't serializable in `commons-collections3` version.
