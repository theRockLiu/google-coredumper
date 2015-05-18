# WriteCoreDump #

## SYNOPSIS ##

```
   #include "google/coredumper.h"

   int WriteCoreDump(const char *file_name);

   int WriteCoreDumpLimited(const char *file_name, size_t max_length);

   int WriteCompressedCoreDump(const char *file_name, size_t max_length,
                               const struct CoredumperCompressor compressors[],
                               struct CoredumperCompressor **selected_compressor);
```

## DESCRIPTION ##

The **[WriteCoreDump](WriteCoreDump.md)**() function writes a core file to _`file_name`_. The calling process continues running after the file has been written.

The **[WriteCoreDumpLimited](WriteCoreDump.md)**() function restricts the maximum length of the written file. This is similar to what setting calling **setrlimit**(2) does for system-generated core files.

The **[WriteCompressedCoreDump](WriteCoreDump.md)**() function attempts to compress the core file on the fly, if a suitable _compressor_ could be located.

If _selected`_`compressor_ is non-NULL, it will be set to the actual _`CoredumperCompressor`_ object used.

The _filename_ automatically has a suitable suffix appended to it. Normally this would be ".bz2" for **bzip2**(1) compression, ".gz" for **gzip**(1) compression, or ".Z" for **compress**(1) compression. This behavior can be changed by defining custom _`CoredumperCompressor`_ descriptions. See the manual page for **[GetCoreDump](GetCoreDump.md)**(3) for more details.

## RETURN VALUE ##

**[WriteCoreDump](WriteCoreDump.md)**(), **[WriteCoreDumpLimited](WriteCoreDump.md)**(), and **[WriteCompressedCoreDump](WriteCoreDump.md)**() return zero on success and -1 on failure. In case of failure, _errno_ will be set appropriately.

## ERRORS ##

The most common reason for failure is for another process to already use the debugging API that is needed to generate the core files. This could, for instance, be **gdb**(1), or **strace**(1).

## NOTES ##

The coredumper functions momentarily suspend all threads, while creating a COW (copy-on-write) copy of the process's address space.

The functions are neither reentrant nor async signal safe. Callers should wrap a mutex around their invocation, if necessary.

The current implementation tries very hard to behave reasonably when called from a signal handler, but no guarantees are made that this will always work. Most importantly, it is the caller's responsibility to make sure that there are never more than one instance of functions from the **[GetCoreDump](GetCoreDump.md)**() or **[WriteCoreDump](WriteCoreDump.md)**() family executing concurrently.

## SEE ALSO ##

**[GetCoreDump](GetCoreDump.md)**(3), and **[GetCompressedCoreDump](GetCoreDump.md)**(3).