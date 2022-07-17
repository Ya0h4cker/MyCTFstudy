# SpringBoot SpEL RCE

## SpEL introduction

- `#{}`: It is xml style templates.The contents in this symbol is parsed as the SpEL expressions, and can use operators, variables, and bean references, properties, and methods.

- `${}`: It is expression templates, and simliar with `#{}`.

- Class type expression: use `T(Type)` to represent the instance of the corresponding type class, and can access class static methods and class static fields.

- Class instantiation: use java keyword `new`.

## Analysis

In white error page of the `SpringBoot`, the URL request  parameter will be recursively resolved through the `parseStringValue()` method, in which the contents in the symbol `${}` will be parsed as the SpEL expressions.
