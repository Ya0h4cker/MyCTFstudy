# Fastjson deserialization

## Analysis

In `fastjson`, the `JSON.parse()` method deserializes the json string to the class specified by `@type` key, and the `JSON.parseObject()` method deserializes the json string to the class specified by the method parameter.

When creating a class instance, it will call the `setter` and `getter` methods that meet the conditions in the class through reflection. And it is just the source of the vulnerability.

## TemplatesImpl chain

The `getOutputProperties()` method of the `TemplatesImpl` class calls the `newTransformer()`, so it's obvious.

## JdbcRowSetImpl chain

The `setAutoCommit()` method of the `JdbcRowSetImpl` class calls the `connect()` method of the `Connection` class. In the `connect()` method, the `InitialContext.lookup()` method is called, which means the JNDI injection. And the `lookup()` method parameter is the `dataSource` field of the `JdbcRowSetImpl` class.

## Fastjson-1.2.47

In the `fastjson-1.2.47`, it doesn't just set the whitelist and blacklist to filter the types of deserialized classes. And if the class is in the blacklist, it will lookup the corresponding class from `TypeUtils.mappings` and `deserializers`. However it can be exploited to bypass the whitelist and blacklist.

If the `@type` key is the `Class`, it will be handled through `MiscCodec.deserialze()` and the corresponding value will be the parameter of the `TypeUtils.loadClass()` to be called. In the `loadClass()` method, the value will be added into the `TypeUtils.mappings`.

Therefore, set the value to the evil class name, so that we can load the evil class to bypass the whitelist and blacklist.
