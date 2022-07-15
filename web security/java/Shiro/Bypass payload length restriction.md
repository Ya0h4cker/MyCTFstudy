# Bypass payload length restriction

## WAF restriction

Modify the HTTP request method to an unkown method.

**Principle:** The cookie in HTTP header is read before the HTTP request method is processed. However the WAF only filters partial HTTP request methods.

## Tomcat restriction

* Modify the max size of HTTP request header in `Tomcat` by reflection.

* The payload in cookie realizes obtaining the `Request` object, thereby acquiring the data in the HTTP request body. Then the payload realizes calling `ClassLoader.defineClass()` method to load the evil class dynamically with the bytecodes in the HTTP request body.
