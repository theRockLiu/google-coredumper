# GetCoreDump #

## SYNOPSIS ##

```
   #include "google/coredumper.h"

   int GetCoreDump(void);

   int GetCompressedCoreDump(const struct CoredumperCompressor compressors[],
                             struct CoredumperCompressor **selected_compressor);
```

## DESCRIPTION ##

The **[GetCoreDump](GetCoreDump.md)**() function returns a file handle that can be read to obtain a snapshot of the current state of the calling process. The **[GetCompressedCoreDump](GetCoreDump.md)**() function returns a file handle to a core file that has been compressed on the fly. In _compressor_, the caller passes a pointer to an array of possible compressors:

```
   struct CoredumperCompressor {
     const char *compressor;  // File name of compressor; e.g. "gzip"
     const char *const *args; // execv()-style command line arguments
     const char *suffix;      // File name suffix; e.g. ".gz"
   };
```

The _suffix_ will be ignored by the **[GetCoreDump](GetCoreDump.md)**() and **[GetCompressedCoreDump](GetCoreDump.md)**() functions, and is only needed for the **[WriteCoreDump](WriteCoreDump.md)**() family of functions.

Array entries will be tried in sequence until an executable compressor has been found or the end of the array has been reached. The end is signalled by an entry that has been zero'd out completely. An empty string in place of the _compressor_ name signals that no compression should be performed.

There are several pre-defined compressor descriptions available:

  * **COREDUMPER\_COMPRESSED**

> Try compressing with either **bzip2**(1), **gzip**(1), or **compress**(1). If all of those fail, fall back on generating an uncompressed image.

  * **COREDUMPER\_BZIP2\_COMPRESSED** **COREDUMPER\_GZIP\_COMPRESSED** **COREDUMPER\_COMPRESS\_COMPRESSED**

> Try compressing with a specific compressor. Fail if no compressor could be found.

  * **COREDUMPER\_TRY\_BZIP2\_COMPRESSED** **COREDUMPER\_TRY\_GZIP\_COMPRESSED** **COREDUMPER\_TRY\_COMPRESS\_COMPRESSED**

> Try compressing with a specific compressor. Fall back on generating an uncompressed image, if the specified compressor is unavailable.

  * **COREDUMPER\_UNCOMPRESSED**

> Always create an uncompressed core file.

If _`selected_compressor`_ is non-NULL, it will be set to the actual _`CoredumperCompressor`_ object used.

## RETURN VALUE ##

Both **[GetCoreDump](GetCoreDump.md)**(), and **[GetCompressedCoreDump](GetCoreDump.md)**() return a non-seekable file handle on success. The copy-on-write snapshot will automatically be released, when the caller **close**()s this file handle.

On error -1 will be returned and _errno_ will be set appropriately.

## ERRORS ##

The most common reason for failure is for another process to already use the debugging API that is needed to generate the core files. This could, for instance, be **gdb**(1), or **strace**(1).

## NOTES ##

The coredumper functions momentarily suspend all threads, while creating a COW (copy-on-write) copy of the process's address space. The snapshot shows up as a new child process of the current process, but memory requirements are relatively small, as most pages are shared between parent and child.

The functions are neither reentrant nor async signal safe. Callers should wrap a mutex around their invocation, if necessary.

The current implementation tries very hard to behave reasonably when called from a signal handler, but no guarantees are made that this will always work. Most importantly, it is the caller's responsibility to make sure that there are never more than one instance of functions from the **[GetCoreDump](GetCoreDump.md)**() or **[WriteCoreDump](WriteCoreDump.md)**() family executing concurrently.

## SEE ALSO ##

**[WriteCoreDump](WriteCoreDump.md)**(3), **[WriteCoreDumpLimited](WriteCoreDump.md)**(3), and **[WriteCompressedCoreDump](WriteCoreDump.md)**(3).