# Overview #

The coredumper library can be compiled into applications to create core dumps of the running program -- without terminating. It supports both single- and multi-threaded core dumps, even if the kernel does not natively support multi-threaded core files.

Coredumper is distributed under the terms of the BSD License.

# Example #

This is by no means a complete example; it simply gives you a feel for what the coredumper API looks like.

```
  #include <google/coredumper.h>
  ...
  WriteCoreDump('core.myprogram');
  /* Keep going, we generated a core file,
   * but we didn't crash.
   */
```