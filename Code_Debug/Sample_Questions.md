# Senior Embedded C/C++ Interview Practice: Debugging and Code Review

For senior embedded software roles at companies such as Qualcomm, Intel,
NVIDIA, AMD, Texas Instruments, and similar product-based companies, you should
be comfortable doing more than finding syntax errors. You need to read
unfamiliar code, identify subtle bugs, explain why they occur, and suggest
improvements without changing the intended behavior.

The best way to prepare is to practice with short code snippets that look
almost correct but contain one or more defects.

You should train yourself to answer five questions for every snippet:

- What is the bug? Identify the exact line or assumption that is wrong.
- Why is it a bug? Explain the C/C++ language rule, hardware behavior, or
  concurrency issue involved.
- When will it fail? Describe the input, timing, compiler optimization, or
  hardware condition that triggers it.
- How would you fix it? Write the corrected code, not just a verbal
  explanation.
- What else would you improve? Consider performance, readability, portability,
  safety, and maintainability.

Below is a practice set designed for senior embedded interviews. These are
representative interview-style exercises, not claims that these exact snippets
were asked by any particular company.

## How to approach the exercises

Try each question for 5–10 minutes before reading the solution.

Assume a typical modern embedded target unless the question specifies otherwise.

For C/C++, distinguish between undefined behavior, implementation-defined
behavior, and hardware-dependent behavior.

For each fix, explain the trade-offs. Senior interviewers often care as much
about your reasoning as the final code.

## Level 1 — C fundamentals and common debugging traps

These are the first types of problems you should master. They test pointers,
memory, integer arithmetic, and bit manipulation.

### Question 1: Buffer overflow and string termination

**Difficulty:** Easy–medium | **Topic:** Memory safety

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

**Your task:**

- Find the bug.
- What happens if src contains "Qualcomm"?
- What happens if it contains "NVIDIA"?
- Can the code print garbage or read beyond the buffer?
- How would you fix it without introducing another buffer overflow?

### Question 2: Signed and unsigned comparison

**Difficulty:** Medium | **Topic:** Integer conversions

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

**Your task:**

- Predict the output on a typical 32-bit embedded target.
- Why can a negative value behave like a very large positive number?
- What changes if the variables are int64\_t and uint32\_t?
- How would you make the comparison safe?

### Question 3: Operator precedence in bit manipulation

**Difficulty:** Easy–medium | **Topic:** Registers and bit masks

c

```
#include <stdint.h>

uint32_t reg = 0xA5A50000;
uint32_t mask = 0x0000000F;

reg &= ~mask == 0;
```

**Your task:**

- Does this clear the lower four bits?
- Explain how the compiler groups the operators.
- Rewrite the expression to clear the bits correctly.
- How would you set bits 4–7 while preserving every other bit?

### Question 4: Returning the address of a local variable

**Difficulty:** Easy | **Topic:** Pointer lifetime

c

```
#include <stdint.h>

uint8_t *get_status(void)
{
    uint8_t status = 1;
    return &status;
}

int main(void)
{
    uint8_t *p = get_status();
    return *p;
}
```

**Your task:**

- What is the lifetime of status?
- Why might the code appear to work during debugging but fail in an optimized build?
- Would declaring status as volatile fix it?
- Give two valid alternative designs.

### Question 5: Incorrect array size calculation

**Difficulty:** Medium | **Topic:** Arrays and pointers

c

```
#include <stdint.h>
#include <string.h>

void process(void)
{
    uint32_t data[16];
    memset(data, 0, sizeof(data) / sizeof(data[0]));
}
```

**Your task:**

- Is this code correct?
- How many bytes does memset write?
- What would happen if the same expression were used after data was passed as a pointer to another function?
- Rewrite the code to avoid the most common mistake.

## Level 2 — Embedded C, hardware registers, and interrupts

These problems are particularly relevant to firmware, device drivers, BSPs, and low-level systems software.

### Question 6: Why volatile is not enough

**Difficulty:** Medium–hard | **Topic:** ISR communication

c

```
#include <stdint.h>

volatile uint32_t ticks = 0;

void SysTick_Handler(void)
{
    ticks++;
}

int main(void)
{
    while (ticks < 1000) {
        /* Wait */
    }

    /* Continue */
}
```

**Your task:**

- What is volatile doing here?
- Is this code necessarily safe on every embedded target?
- What if a 32-bit variable is accessed on an 8-bit MCU?
- Does volatile guarantee atomicity, thread safety, or memory ordering?
- How would you implement a portable solution for a target where a 32-bit read
  can be interrupted halfway through?

### Question 7: Read-modify-write on a hardware register

**Difficulty:** Hard | **Topic:** MMIO and register semantics

c

```
#include <stdint.h>

#define STATUS_REG (*(volatile uint32_t *)0x40001000u)

void clear_interrupt_flag(void)
{
    STATUS_REG |= (1u << 3);
}
```

Assume the hardware manual says that bits in STATUS\_REG are write-one-to-clear (W1C).

**Your task:**

- Why might this code clear more flags than intended?
- What happens during the read-modify-write sequence?
- What is the correct way to clear only bit 3?
- Why must you check the register's hardware specification before applying
  generic bit-manipulation patterns?

### Question 8: Race condition between an ISR and main code

**Difficulty:** Hard | **Topic:** Shared state

c

```
#include <stdint.h>

static volatile uint32_t received = 0;
static uint8_t rx_buffer[64];

void UART_IRQHandler(void)
{
    rx_buffer[received % 64] = UART_READ_DATA();
    received++;
}

void process_rx(void)
{
    uint32_t count = received;

    for (uint32_t i = 0; i < count; i++) {
        process_byte(rx_buffer[i % 64]);
    }

    received = 0;
}
```

Assume UART\_READ\_DATA() reads the UART receive register and process\_byte()
processes one byte.

**Your task:**

- Can bytes be lost?
- Can the main code process a byte while the ISR is overwriting it?
- Is volatile sufficient?
- What is wrong with resetting received to zero?
- Propose a ring-buffer design that can handle continuous reception.

### Question 9: Incorrect use of a timeout

**Difficulty:** Medium–hard | **Topic:** Timer rollover

c

```
#include <stdint.h>

int timed_out(uint32_t start, uint32_t timeout,
              uint32_t now)
{
    return now >= start + timeout;
}
```

**Your task:**

- What happens when the timer wraps around from UINT32\_MAX to zero?
- Can this function return the wrong result?
- Rewrite it using unsigned arithmetic.
- What assumption about timeout must hold for the corrected method to work?

## Level 3 — Senior-level C/C++ debugging

These questions test language semantics, resource management, alignment, and
the kinds of mistakes that are difficult to spot during code review.

### Question 10: Strict aliasing and type punning

**Difficulty:** Hard | **Topic:** C/C++ memory model

c

```
#include <stdint.h>

float get_float(uint32_t bits)
{
    return *(float *)&bits;
}
```

**Your task:**

- Is this a portable way to reinterpret the bits?
- What language rules can make this undefined behavior?
- Why might the code behave differently under optimization?
- How would you rewrite it in C and C++?

### Question 11: C++ resource leak and exception safety

**Difficulty:** Medium–hard | **Topic:** RAII and resource management

cpp

```
class Device
{
public:
    void start()
    {
        buffer = new uint8_t[1024];
        initialize_hardware();
    }

    ~Device()
    {
        delete[] buffer;
    }

private:
    uint8_t *buffer = nullptr;
};
```

Assume initialize\_hardware() can fail or throw an exception.

**Your task:**

- What happens if start() is called twice?
- What happens if initialization fails after allocation?
- What happens if the object is copied?
- How would you improve this class for production firmware?
- Would std::unique\_ptr\<uint8\_t\[\]\> be a suitable solution in a resource-constrained system?

### Question 12: Unaligned memory access

**Difficulty:** Hard | **Topic:** Architecture and portability

c

```
#include <stdint.h>

uint32_t read_u32(const uint8_t *buffer)
{
    return *(const uint32_t *)(buffer + 1);
}
```

**Your task:**

- Why is this potentially unsafe even if the buffer contains four valid bytes
  at that position?
- What happens on a CPU that does not support unaligned loads?
- Is the code automatically safe on a CPU that supports unaligned loads?
- Rewrite it portably, considering that the input might be a little-endian byte
  stream.

## Level 4 — Concurrency, RTOS, DMA, and performance

These are important for senior firmware and embedded systems interviews because
the code can look perfectly reasonable while failing only under load, with a
particular compiler, or on real hardware.

### Question 13: Broken single-producer/single-consumer queue

**Difficulty:** Hard | **Topic:** Concurrency and C++ atomics

cpp

```
#include <cstdint>

uint32_t buffer[128];
uint32_t head = 0;
uint32_t tail = 0;

void push(uint32_t value)
{
    buffer[head % 128] = value;
    head++;
}

bool pop(uint32_t &value)
{
    if (head == tail)
        return false;

    value = buffer[tail % 128];
    tail++;
    return true;
}
```

**Your task:**

- What happens when the queue becomes full?
- What if push() and pop() execute concurrently on separate CPU cores?
- Can the compiler or CPU reorder memory operations in a way that breaks the
  queue?
- Would making all three variables volatile solve the problem?
- How would you implement the queue correctly for a single producer and single
  consumer?

### Question 14: DMA buffer coherency

**Difficulty:** Hard | **Topic:** Cache and DMA

c

```
#include <stdint.h>

static uint8_t rx_buffer[256];

void receive_packet(void)
{
    start_dma_receive(rx_buffer, sizeof(rx_buffer));

    while (!dma_complete()) {
        /* Wait */
    }

    process_packet(rx_buffer, sizeof(rx_buffer));
}
```

Assume DMA writes incoming data directly to RAM and the CPU has a data cache.

**Your task:**

- Why might the CPU process stale data even though the DMA transfer completed?
- Is declaring rx\_buffer as volatile sufficient?
- What cache-maintenance operations might be required?
- Why can cache-line alignment and buffer placement matter?
- How would your answer change on a non-cache-coherent MCU?

### Question 15: Busy waiting and real-time behavior

**Difficulty:** Medium | **Topic:** Timing and CPU utilization

c

```
void wait_for_device(void)
{
    while ((DEVICE_STATUS & READY_BIT) == 0) {
    }
}
```

**Your task:**

- What happens if the device never becomes ready?
- Why can this be problematic in an RTOS task?
- What happens if DEVICE\_STATUS is not declared appropriately for MMIO?
- How would you add a timeout?
- When is busy waiting actually the right design choice?

### Question 16: Integer overflow in memory allocation

**Difficulty:** Hard | **Topic:** Integer overflow and bounds checking

c

```
#include <stdint.h>
#include <stdlib.h>

void allocate_buffer(uint32_t count)
{
    uint32_t bytes = count * sizeof(uint32_t);

    uint32_t *buffer = malloc(bytes);

    if (buffer != NULL) {
        initialize_buffer(buffer, count);
        free(buffer);
    }
}
```

**Your task:**

- Can bytes become smaller than the required allocation?
- What happens when count is very large?
- Why is the multiplication dangerous on a 32-bit target?
- How would you validate the size before allocating?
- What else should be checked if initialize\_buffer() writes count elements?

## Level 5 — Code review: find every issue you can

These are closer to the open-ended exercises you may encounter in a
senior-level interview. There may be several independent bugs, and the
interviewer may expect you to prioritize them.

### Question 17: A driver initialization routine

**Difficulty:** Hard | **Topic:** Error handling, initialization, cleanup

c

```
#include <stdint.h>
#include <stdlib.h>

typedef struct {
    uint8_t *buffer;
    uint32_t size;
    int initialized;
} Driver;

int driver_init(Driver *d, uint32_t size)
{
    d->buffer = malloc(size);
    d->size = size;

    hardware_init();

    d->initialized = 1;
    return 0;
}

void driver_deinit(Driver *d)
{
    free(d->buffer);
    d->initialized = 0;
}

int driver_write(Driver *d, const uint8_t *src, uint32_t len)
{
    if (!d->initialized)
        return -1;

    for (uint32_t i = 0; i < len; i++)
        d->buffer[i] = src[i];

    return 0;
}
```

**Your task:**

- Find all memory-safety and initialization bugs.
- What if malloc() fails?
- What if hardware\_init() fails?
- What if len \> d-\>size?
- What if d or src is NULL?
- What happens if driver\_deinit() is called twice?
- Rewrite the function contracts and error handling to make the driver safer.

### Question 18: C++ object lifetime and asynchronous callbacks

**Difficulty:** Hard | **Topic:** Lifetime, concurrency, asynchronous systems

cpp

```
class Sensor
{
public:
    void start()
    {
        register_callback([this]() {
            read_sensor();
        });
    }

    void stop()
    {
        unregister_callback();
    }

    void read_sensor()
    {
        /* Read sensor and update internal state */
    }
};
```

**Your task:**

- What happens if the Sensor object is destroyed while a callback is still pending?
- Does unregister\_callback() necessarily guarantee that no callback is running?
- What if the callback executes concurrently with stop()?
- How would you make the lifecycle safe?
- Would capturing this by value solve the problem?

## Answer key and explanations

Try the questions first. Then use the explanations below to check your
reasoning. In a real interview, explain the failure mechanism rather than
simply naming the bug.

## Solutions 1–5: C fundamentals

### 1

strncpy() does not guarantee null termination.

"Qualcomm" contains eight characters, so copying eight bytes fills the entire
array without a '\\0'. Passing name to %s can read past the array, causing
undefined behavior. "NVIDIA" is shorter, so strncpy() pads the remaining bytes
with zeros.

A simple fix:

c

```
void copy_name(const char *src)
{
    if (src == NULL)
        return;

    char name[8];
    size_t n = strlen(src);

    if (n >= sizeof(name))
        n = sizeof(name) - 1;

    memcpy(name, src, n);
    name[n] = '\0';

    printf("%s\n", name);
}
```

This assumes src points to a valid null-terminated string. In production,
consider bounded input lengths to avoid an unbounded strlen().

### 2

Signed/unsigned conversion changes the comparison.

On a typical target with 32-bit int32\_t and uint32\_t, count is converted to
unsigned. The value -1 becomes UINT32\_MAX, so the condition is false and the
program prints Limit exceeded.

Validate the sign before converting:

c

```
if (count >= 0 && (uint32_t)count < limit) {
    printf("Below limit\n");
}
```

With int64\_t and uint32\_t, the usual arithmetic conversions generally convert
the unsigned operand to int64\_t, since it can represent every uint32\_t value.
Then -1 \< 10 is true. Do not rely on implicit conversions to express intent.

### 3

Operator precedence makes the expression clear the entire register.

Equality binds more tightly than bitwise AND, so the expression is parsed as:

c

```
reg &= ((~mask) == 0);
```

The comparison is false, producing zero; therefore, reg becomes zero.

Correct code:

c

```
reg &= ~mask;          /* Clear bits 0–3 */
reg |= (0xFu << 4);    /* Set bits 4–7 */
```

In real MMIO code, these operations are appropriate only if the register
supports read-modify-write semantics.

### 4

The pointer becomes invalid when the function returns.

status has automatic storage duration. Its lifetime ends when get\_status()
returns, so dereferencing the returned pointer has undefined behavior. volatile
does not extend an object's lifetime.

Better alternatives include returning the value directly:

c

```
uint8_t get_status(void)
{
    return 1;
}
```

Or having the caller provide storage:

c

```
void get_status(uint8_t *out)
{
    if (out != NULL)
        *out = 1;
}
```

Static storage is another option when shared persistent state is actually
required, but it introduces shared-state considerations.

### 5

memset() takes a byte count, not an element count.

The array occupies 16 × 4 = 64 bytes on a target where uint32\_t is four bytes.
The expression evaluates to 16, so only the first 16 bytes are cleared.

Correct:

c

```
memset(data, 0, sizeof(data));
```

This works while data is an array in this scope. If passed to a function as
uint32\_t \*data, sizeof(data) returns the pointer size, not the original array
size. Pass the element count or byte length separately.

## Solutions 6–9: Embedded systems

### 6

volatile does not make an access atomic.

It tells the compiler that accesses to the volatile object are observable and
must follow the implementation's volatile-access rules. It does not guarantee
that a 32-bit load is indivisible on an 8-bit CPU.

A target-specific critical section can protect the snapshot:

c

```
disable_interrupts();
uint32_t snapshot = ticks;
enable_interrupts();
```

The actual implementation must preserve the previous interrupt state rather
than blindly enabling interrupts afterward. Alternatively, use an appropriate
atomic type if the target and toolchain support it correctly. C and C++ atomics
are not necessarily lock-free on every MCU.

For a simple polling loop, also consider a timeout so a missing tick source cannot block forever.

### 7

W1C registers must be handled according to their semantics.

STATUS\_REG |= (1u \<\< 3) reads the register and writes the result back. Every
flag that was already set is written back as 1, potentially clearing all those
flags.

For a W1C register, the usual operation is:

c

```
STATUS_REG = (1u << 3);
```

This writes a one only to bit 3. Check the hardware manual for reserved bits,
access width, and any special write requirements before applying this pattern.

### 8

The shared counter is doing too many jobs.

The ISR uses received as a write index and the main code treats it as a batch
count. Resetting it can race with an interrupt and lose newly received data. If
the counter grows beyond 64, older entries are overwritten, and a large count
can cause the main loop to process stale or duplicate bytes.

Use separate producer and consumer indices in a ring buffer. Define a
full-buffer policy (drop newest, drop oldest, or signal overflow). Ensure index
updates and data publication are safe for the target. On a single-core MCU, a
carefully designed ISR/main implementation may use atomic-width index updates
and short critical sections; on multicore systems, appropriate atomics and
memory ordering may be needed.

### 9

Adding the timeout to the start time can overflow.

start + timeout may wrap before the comparison. Use unsigned subtraction:

c

```
int timed_out(uint32_t start, uint32_t timeout,
              uint32_t now)
{
    return (uint32_t)(now - start) >= timeout;
}
```

This is the standard wraparound-safe elapsed-time pattern for a monotonic
unsigned counter, provided the actual elapsed interval is
unambiguous—typically, the caller guarantees it checks the timeout before a
full timer wrap has elapsed. For stricter designs, constrain timeout ranges and
document the clock assumptions.

## Solutions 10–12: C/C++ language rules

### 10 — Strict aliasing

Accessing a uint32\_t object through a float \* violates the permitted
type-access rules in C and C++. Use memcpy() to copy the bit pattern into a
float where the representations are appropriate. In C++20,
std::bit\_cast\<float\>(bits) is another option, subject to equal sizes and
valid representation assumptions. Verify the floating-point representation if
portability matters.

### 11 — Resource management

Calling start() twice overwrites the first pointer and leaks the original
allocation. If initialization throws, the object may remain partially
initialized. Copying the class copies the raw pointer, so two destructors may
attempt to free the same allocation. Prefer RAII, such as
std::unique\_ptr\<uint8\_t\[\]\>, delete copy operations or implement correct
ownership transfer, and make initialization failure leave the object in a
documented, recoverable state. Embedded systems may also use fixed-size storage
or a controlled allocator.

### 12 — Alignment and aliasing

buffer + 1 may not be aligned for a uint32\_t load. Even if the CPU supports
unaligned loads, the cast can violate C/C++ alignment and object-access rules.
Use memcpy() into a uint32\_t, or assemble the value from bytes. For a
little-endian stream, explicitly combine the bytes with shifts and casts, and
validate that at least four bytes are available.

## Solutions 13–18: Advanced systems debugging

### 13\. Single-producer/single-consumer queue

There are three main issues:

- No full-buffer detection: head % 128 overwrites unread entries if the
  producer gets too far ahead.
- Data races: Concurrent non-atomic accesses to head and tail in C++ are
  undefined behavior.
- Missing publication ordering: The consumer must not observe an updated head
  before the corresponding buffer write is visible.

A simplified SPSC queue using C++ atomics can look like this:

cpp

```
#include <atomic>
#include <cstdint>

constexpr uint32_t N = 128;

uint32_t buffer[N];
std::atomic<uint32_t> head{0};
std::atomic<uint32_t> tail{0};

bool push(uint32_t value)
{
    uint32_t h = head.load(std::memory_order_relaxed);
    uint32_t t = tail.load(std::memory_order_acquire);

    if (h - t >= N)
        return false;

    buffer[h % N] = value;
    head.store(h + 1, std::memory_order_release);
    return true;
}

bool pop(uint32_t &value)
{
    uint32_t t = tail.load(std::memory_order_relaxed);
    uint32_t h = head.load(std::memory_order_acquire);

    if (t == h)
        return false;

    value = buffer[t % N];
    tail.store(t + 1, std::memory_order_release);
    return true;
}
```

This is an illustrative SPSC implementation: it assumes exactly one producer
and one consumer, that unsigned index arithmetic remains valid for the queue's
operating interval, and that the atomic types are suitable for the target. For
ISR use, verify that the required atomics are lock-free and ISR-safe.
Otherwise, use a target-appropriate critical-section design.

### 14\. DMA buffer coherency

DMA completion does not necessarily mean the CPU cache contains the latest bytes.

If the DMA controller writes RAM while the CPU retains old cache lines, the CPU
may read stale data. volatile does not invalidate a cache or synchronize the
DMA controller with the CPU.

A typical receive sequence on a non-coherent, write-back-cache system is:

1. Prepare the destination buffer according to the platform's DMA rules.
2. Ensure the buffer is suitably aligned and does not share cache lines with
   unrelated dirty data.
3. Perform the required cache clean/invalidate operation before DMA if the
   platform requires it.
4. Start DMA and wait for documented completion.
5. Invalidate the relevant cache lines after DMA, using the platform's
   prescribed barriers and cache-maintenance API.
6. Process the received bytes.

Exact requirements differ by processor, cache architecture, DMA engine, and
operating system. Some platforms provide coherent DMA memory or dedicated
DMA-safe allocation APIs, which are preferable to guessing at cache operations.

### 15\. Busy waiting and real-time behavior

The loop can block forever if the device never becomes ready. In an RTOS task,
it may also consume CPU time and prevent lower-priority work from running.

A safer design uses a timeout:

c

```
int wait_for_device(uint32_t timeout_ticks)
{
    uint32_t start = get_tick_count();

    while ((DEVICE_STATUS & READY_BIT) == 0) {
        if ((uint32_t)(get_tick_count() - start)
                >= timeout_ticks) {
            return -1;
        }

        /* Optionally yield or sleep if permitted. */
    }

    return 0;
}
```

This assumes get\_tick\_count() is a suitable monotonic unsigned counter and
the timeout fits its documented range. DEVICE\_STATUS must also be accessed
using the platform's correct MMIO definitions.

Busy waiting is appropriate when the wait is very short and bounded, the
latency requirement justifies it, and the CPU is not needed for other work.

### 16\. Integer overflow in allocation

On a 32-bit target, the allocation size can overflow when count \*
sizeof(uint32\_t) is evaluated. The resulting allocation may be too small,
while initialize\_buffer() still attempts to write count elements.

Validate before multiplying:

c

```
#include <stdint.h>
#include <stdlib.h>
#include <stdint.h>

int allocate_buffer(uint32_t count)
{
    if (count > SIZE_MAX / sizeof(uint32_t))
        return -1;

    size_t bytes = (size_t)count * sizeof(uint32_t);

    uint32_t *buffer = malloc(bytes);
    if (buffer == NULL && bytes != 0)
        return -1;

    initialize_buffer(buffer, count);
    free(buffer);
    return 0;
}
```

For a strict embedded implementation, also define how zero-sized allocations
are handled and reject sizes that exceed the product's memory budget. In
practice, a fixed maximum allocation is often safer than accepting an arbitrary
count.

### 17\. Driver initialization

The important issues are:

- d may be NULL.
- malloc() failure is ignored.
- hardware\_init() has no checked failure path.
- driver\_write() does not validate src or len.
- len \> d-\>size causes an out-of-bounds write.
- Reinitialization may leak an existing allocation.
- Repeated deinitialization requires a defined state and cleanup contract.

A better implementation should validate arguments, allocate resources, check
every initialization step, and unwind already-acquired resources if a later
step fails. It should also specify whether reinitialization is supported.

For example, the write path could be:

c

```
int driver_write(Driver *d,
                 const uint8_t *src,
                 uint32_t len)
{
    if (d == NULL || !d->initialized)
        return -1;

    if (len > d->size)
        return -1;

    if (len != 0 && src == NULL)
        return -1;

    for (uint32_t i = 0; i < len; i++)
        d->buffer[i] = src[i];

    return 0;
}
```

This assumes the initialized driver's buffer is valid and the driver is not
concurrently being deinitialized. If concurrent access is possible,
synchronization and lifecycle rules must be added.

### 18\. Asynchronous callbacks and object lifetime

Capturing this does not extend the object's lifetime. If the object is
destroyed while a callback is pending, the callback can dereference a dangling
pointer.

Unregistering a callback is not automatically sufficient: the API must
guarantee whether unregistering waits for in-flight callbacks, prevents future
dispatch, or merely removes future registrations.

A robust design needs a documented shutdown protocol:

- Stop new callback dispatch.
- Wait for or cancel callbacks already in flight.
- Ensure no callback can access the object after destruction begins.
- Release associated resources only after callbacks have stopped.

A shared\_ptr/weak\_ptr design can help in suitable C++ systems, but
reference-counting and callback synchronization may be undesirable or
unavailable in some embedded environments. Capturing this by value simply
copies the pointer; it does not solve the lifetime problem.

## Your senior embedded debugging checklist

Use this checklist whenever you receive an unfamiliar code snippet in an
interview.

### Code review self-assessment

0/19 mastered

**Memory and pointers**

- Array bounds and buffer lengths
- Null pointers and dangling pointers
- Allocation failure and ownership
- String termination and object lifetime

**C/C++ semantics**

- Signed/unsigned conversions
- Integer overflow and shifts
- Operator precedence
- Strict aliasing and alignment
- Undefined behavior and compiler optimization

**Concurrency and RTOS**

- ISR/main shared state
- Atomicity versus volatile
- Race conditions and memory ordering
- Deadlocks, blocking, and priority inversion
- Timeouts and timer rollover

**Hardware and performance**

- MMIO register semantics
- DMA and cache coherency
- Interrupt latency and critical sections
- Stack usage and dynamic allocation
- Portability across CPU architectures

## How I recommend you practice

Start with Questions 1–5 and explain each bug aloud as if you were talking to
an interviewer. Then move to the interrupt, DMA, and concurrency problems. For
every exercise, try to produce three things: the bug, a corrected
implementation, and a short explanation of the trade-offs.

Once these fundamentals are solid, the next step is practicing larger,
realistic code-review exercises involving a UART driver, an SPI transaction
state machine, an RTOS queue, a memory allocator, or a multithreaded C++ device
driver. Those are useful for building the ability to read unfamiliar code
rather than just recognize isolated bugs.
