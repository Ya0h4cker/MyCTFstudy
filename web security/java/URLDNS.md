# URLDNS

## Gadget Chain

    HashMap.readObject()
        HashMap.hash()
            URL.hashCode()
                URLStreamHandler.hashCode()
                    URLStreamHandler.getHostAddress()

## Note

- Object `URL` is as a key of object `HashMap` and `HashMap.readObject()` calls `hash(Key)`. Therefore, the `URL.hashCode()` method is called.

- To make `URL.hashCode()` calls `URLStreamHandler.hashCode()`, the value of `URL.hashcode` should be equal with `-1`. This value is modified by reflections.
