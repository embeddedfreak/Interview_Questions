Copy the following directly into a file named
`buffer_overflow_string_termination.md` in your Git repository.

Writing

# Interview Answer: Buffer Overflow and String Termination in C

The main bug is that `strncpy()` does not guarantee null termination when the
source string length is greater than or equal to the destination buffer size.
As a result, `printf("%s\n", name)` may read beyond the bounds of the local
array, causing undefined behavior.

## 1\. Identify the Bug

### Original Code

c

```
#include <stdio.h>
#include <string.h>

void copy_name(const char *src)
{
    char name[8];

    strncpy(name, src, sizeof(name));
    printf("%s\n", name);
}
```

The destination buffer has 8 bytes, but a valid C string needs a terminating
`'\0'` byte in addition to its characters.

`strncpy()` copies at most 8 bytes here. If the source has 8 or more characters
before its terminator, the destination will not contain a null terminator.

## 2\. What Happens with Each Input?

### Case A: `src = "Qualcomm"`

- `"Qualcomm"` contains 8 characters.
- `strncpy()` copies all 8 characters into `name`.
- No null terminator is appended.
- `printf("%s\n", name)` continues reading memory beyond `name` until it
  encounters a zero byte or otherwise faults.

**Result:** Undefined behavior. It might print extra garbage characters, appear
to work, or crash.

### Case B: `src = "NVIDIA"`

- `"NVIDIA"` contains 6 characters.
- `strncpy()` copies the 6 characters and fills the remaining 2 bytes with
  `'\0'`.
- The resulting buffer is a valid null-terminated string.

**Output:**

text

```
NVIDIA
```

**Important distinction:** This is not necessarily a buffer write overflow. The
immediate bug is a possible out-of-bounds read when `printf()` processes a
non-terminated string.

## 3\. Correct the Code

A straightforward fix is to reserve one byte for the terminator.

### Corrected Code

c

```
#include <stdio.h>
#include <string.h>

void copy_name(const char *src)
{
    char name[8];

    if (src == NULL)
        return;

    strncpy(name, src, sizeof(name) - 1);
    name[sizeof(name) - 1] = '\0';

    printf("%s\n", name);
}
```

Now the function copies at most 7 characters and explicitly terminates the string.

- `"Qualcomm"` becomes `"Qualcom"` (7 characters).
- `"NVIDIA"` remains `"NVIDIA"`.

This version prevents `printf()` from reading beyond `name` due to a missing
terminator, assuming `src` points to a valid readable C string.

## 4\. An Alternative for Code Review

For a small fixed-size buffer, explicit length calculation and copying can be clearer.

c

```
#include <stdio.h>
#include <string.h>

void copy_name(const char *src)
{
    char name[8];

    if (src == NULL)
        return;

    size_t len = strlen(src);

    if (len >= sizeof(name))
        len = sizeof(name) - 1;

    memcpy(name, src, len);
    name[len] = '\0';

    printf("%s\n", name);
}
```

This explicitly calculates the number of characters copied and terminates the result.

However, `strlen()` requires `src` to be a valid, null-terminated string, and
it scans the entire source before truncation. For untrusted or potentially very
large input, a bounded approach may be preferable.

## 5\. What Else Would I Improve?

In a senior embedded interview, I would mention these additional considerations.

### Input Validation

A null pointer check does not validate an arbitrary non-null pointer. The
caller must still supply a valid, readable string.

### Truncation Handling

Silently truncating a device name, identifier, or configuration field may be
unacceptable. Return a status or report truncation if the caller needs the full
value.

### API Design

If the input is a byte buffer rather than a C string, pass its length
explicitly instead of calling `strlen()`.

### Portability

Avoid relying on adjacent stack memory containing a zero byte. Compiler
optimization, stack layout, and other runtime conditions can change the
observed failure.

## 6\. A Polished Interview Response

You can deliver the answer in roughly 45–60 seconds:

> The issue is that `strncpy()` does not always null-terminate the destination.
> Since `name` is 8 bytes, copying the 8-character string `"Qualcomm"` fills
> the entire buffer without a terminator. Passing it to `%s` then causes an
> out-of-bounds read and undefined behavior. For `"NVIDIA"`, which has only six
> characters, `strncpy()` pads the remaining bytes with zeros, so it works
> correctly.I would reserve one byte for the terminator, copy at most
> `sizeof(name) - 1` characters, and explicitly set the last byte to `'\0'`. In
> production code, I would also consider whether truncation is acceptable,
> validate the input contract, and return an error if the caller needs to know
> that the name was truncated.

## 7\. Senior-Level Takeaway

Always distinguish between the size of a character buffer and the maximum
number of characters that can fit in it as a null-terminated C string.

For an 8-byte buffer:

- **Buffer capacity:** 8 bytes
- **Maximum string length:** 7 characters
- **Null terminator:** 1 byte (`'\0'`)

**Key lesson:** Never assume `strncpy()` automatically null-terminates the
destination. Always ensure that the destination is properly terminated before
passing it to string-handling functions such as `printf("%s", ...)`.
