# JDBC deserialization

## Analysis

When the MySQL client connects to the MySQL server using specific configuration, specific query statements will be executed and the result will be deserialized.

if an attacket can set the JDBC URL to the evil MySQL server, the constructed data will be deserialized.

## ServerStatusDiffInterceptor

When the property `queryInterceptors` is set to `ServerStatusDiffInterceptor` class name and the property `autoDeserialize` is set to true in the JDBC URL, the `preProcess()` method of the `ServerStatusDiffInterceptor` class will be called during the connection process. Then the `ServerStatusDiffInterceptor.populateMapWithSessionStatusValues()` method will be called. In this method, execute `SHOW SESSION STATUS` quey statement and invoke `readObject()` method.

## detectCustomCollations

If the property `detectCustomCollations` and the property `autoDeserialize` are set to true, the `buildCollationMapping()` method of the `buildCollationMapping` class will be called. In this method, execute `SHOW COLLATION` quey statement and invoke `readObject()` method.
