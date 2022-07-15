# Memshell detection

The key is to use the RASP technology to monitor the runtime environment, whose principle is based on the java agent.

## Dynamic addition

Hook the dynamic addtion method and storage location.

## Classloader

The memory shell is usually loaded through some unsafe classloaders, so check the classloader.

## Dumpclass

Traverse all `Servlet` and `Filter`, decompile bytecode and check.

## Class file

The memory shell has no corresponding class file, so it's also a feature for detection.

## Mbeans

When adding a `Filter`, the `registerJMX()` method will be triggered and register a `Mbean` object. Therefore, moniter the `Mbean` object to detect the memory shell.