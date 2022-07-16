# CommonBeanutils1

## Gadget Chain

    PriorityQueue.readObject()
        BeanComparator.compare()
            TemplatesImpl.getOutputProperties()
                TemplatesImpl.newTransformer()
                    TemplatesImpl.getTransletInstance() 
                        TemplatesImpl.defineTransletClasses()
                            ClassLoder.defineClass()

## Analysis

### Key: BeanComparator

The string `outputProperties` is as the field `BeanComparator.property`. In method `BeanComparator.compare()`, `PropertyUtils.getProperty()` method will be called with a compared object and `BeanComparator.property` field as parameters. And the `PropertyUtils.getProperty()` method will invoke `getter` method of the compared object, whose name is the same as `BeanComparator.property` field value.

## Note

- This chain is similar with `CommonCollections2` chain, and we need to avoid triggering the gadget chain in advance by reflections.

- This chain also depends on `commons-collections`, because the constructor of the `BeanComparator` class will call a method of the `ComparableComparator` class in `commons-collections` by default. To avoid depending on `commons-collections`, we can set `String.CASE_INSENSITIVE_ORDER` as the constructor parameter of the `BeanComparator` class which means that get a `CaseInsensitiveComparator` object instead of the default `ComparableComparator` class.
