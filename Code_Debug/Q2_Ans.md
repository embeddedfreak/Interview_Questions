Here is a clean Markdown (`.md`) version you can upload directly to GitHub. It
combines both explanations, removes repetition, and formats the code blocks,
equations, and tables correctly.

Writing

# Signed and Unsigned Comparison in C

**Difficulty:** Medium\
**Topic:** Integer Conversions

## 1\. Why is `int32_t` converted to `uint32_t`?

`int32_t` is converted to `uint32_t` because of C's **usual arithmetic
conversions**. Both types have the same integer conversion rank, but one is
signed and the other is unsigned.

Consider the following comparison:

c

```
int32_t count = -1;
uint32_t limit = 10;

if (count < limit)
```

Before comparing the values, the C compiler determines a common type for both operands.

### Conversion rules

For integer operands, C applies the usual arithmetic conversions.

In this case:

- `int32_t` is a signed integer type.
- `uint32_t` is an unsigned integer type.
- Both types have the same integer conversion rank.
- `int32_t` cannot represent every value of `uint32_t`.

Therefore, the signed operand is converted to `uint32_t`.

Conceptually, the comparison becomes:

c

```
if ((uint32_t)count < limit)
```

## 2\. Predict the output

Consider the complete program:

c

```
#include <stdint.h>
#include <stdio.h>

int main(void)
{
    int32_t count = -1;
    uint32_t limit = 10;

    if (count < limit)
        printf("Below limit\n");
    else
        printf("Limit exceeded\n");

    return 0;
}
```

### Output

text

```
Limit exceeded
```

### Explanation

During the comparison:

- `count = -1` becomes `4294967295` (`UINT32_MAX`).
- `limit = 10` remains `10`.
- The comparison becomes `4294967295U < 10U`.
- The condition evaluates to false, so the `else` branch executes.

## 3\. Why can a negative value behave like a very large positive number?

Unsigned integer conversions follow modulo arithmetic.

Converting an integer to an unsigned type produces the value congruent to the
original value modulo $2^N$, where $N$ is the width of the unsigned type.

For a 32-bit unsigned integer:

$$
-1 \bmod 2^{32} = 2^{32}-1
$$

$$
2^{32}-1 = 4294967295
$$

Therefore, converting `-1` to `uint32_t` produces `4294967295`.

On a typical two's-complement system, `-1` has all 32 bits set to `1`.
Interpreting that bit pattern as an unsigned integer gives `4294967295` instead
of `-1`.

The conversion changes the value's interpretation, not necessarily its underlying bit pattern.

## 4\. Why doesn't C convert `uint32_t` to `int32_t` instead?

The reason is that `int32_t` cannot represent all possible values of `uint32_t`.

| Type | Minimum | Maximum |
| --- | --- | --- |
| `int32_t` | -2,147,483,648 | 2,147,483,647 |
| `uint32_t` | 0 | 4,294,967,295 |

If an unsigned value greater than `2,147,483,647` were converted to `int32_t`, it could not be represented.

Under the usual arithmetic conversion rules, when the signed and unsigned
operands have the same rank and the signed type cannot represent every value of
the unsigned type, the signed operand is converted to the unsigned type.

## 5\. What changes with `int64_t` and `uint32_t`?

Consider this example:

c

```
#include <stdint.h>
#include <stdio.h>

int main(void)
{
    int64_t count = -1;
    uint32_t limit = 10;

    if (count < limit)
        printf("Below limit\n");
    else
        printf("Limit exceeded\n");

    return 0;
}
```

### Output

text

```
Below limit
```

### Explanation

The signed `int64_t` type can represent every possible value of `uint32_t`.

Therefore, `limit` is converted to `int64_t`, and the comparison is effectively:

c

```
if (-1LL < 10LL)
```

This condition is true.

### Comparison summary

| Variable types | Conversion during comparison | Result |
| --- | --- | --- |
| `int32_t` and `uint32_t` | `int32_t` → `uint32_t` | False |
| `int64_t` and `uint32_t` | `uint32_t` → `int64_t` | True |

The result depends on C's integer conversion rules, not simply on the processor's native word size.

## 6\. How can you make the comparison safe?

The correct approach depends on whether negative values are valid in your application.

### Option A: Check for negative values first

If `count` must be nonnegative, validate it before converting it to an unsigned type.

c

```
#include <stdint.h>
#include <stdio.h>

int main(void)
{
    int32_t count = -1;
    uint32_t limit = 10;

    if (count < 0) {
        printf("Invalid negative count\n");
    } else if ((uint32_t)count < limit) {
        printf("Below limit\n");
    } else {
        printf("Limit reached or exceeded\n");
    }

    return 0;
}
```

Output:

text

```
Invalid negative count
```

The cast is safe because the negative case is handled first.

### Option B: Convert both operands to a suitable signed type

If the values should be compared as signed integers, use `int64_t` as the common type.

c

```
if ((int64_t)count < (int64_t)limit) {
    printf("Below limit\n");
}
```

This works for `int32_t` and `uint32_t` because `int64_t` can represent the full range of both types.

### Option C: Use unsigned types consistently

If negative counts are never valid, use an unsigned type from the beginning.

c

```
uint32_t count = 5;
uint32_t limit = 10;

if (count < limit) {
    printf("Below limit\n");
}
```

This is appropriate only when negative values are not meaningful. Changing a
signed variable to unsigned does not automatically make invalid input safe.

## 7\. Key takeaways

- **Usual arithmetic conversions:** C determines a common type before
  evaluating mixed signed and unsigned comparisons.
- **Unsigned conversion:** Converting `-1` to `uint32_t` produces `4294967295`.
- **Integer rank matters:** `int32_t` and `uint32_t` typically have the same
  rank, while `int64_t` can represent all `uint32_t` values.
- **Validate inputs:** Check for negative values before converting them to
  unsigned types.
- **Choose appropriate types:** Use consistent integer types for counters,
  lengths, limits, and indexes in embedded C.

**Remember:** Never assume that comparing a signed integer with an unsigned
integer preserves the signed value. A negative number can become a very large
unsigned value before the comparison takes place.
