# AMD Interview Analysis
## Senior Linux Display Driver Development Engineer (Bangalore)

Based on the AMD job description, Glassdoor interview experiences, and interview reports from Linux kernel/driver engineers, this role appears to follow AMD's standard **Senior Individual Contributor (IC)** hiring process with a heavy focus on:

- Linux kernel internals
- Display drivers
- Debugging
- Hardware-software interaction

rather than algorithm-heavy interviews.

---

# Expected Interview Process

From previous AMD candidates and Linux kernel interview experiences, you can expect **4–6 rounds**.

| Round | Interviewer | Duration | Probability |
|--------|-------------|----------|-------------|
| HR/Recruiter Screening | Recruiter | 20–30 mins | Almost Certain |
| Technical Round 1 | Senior Engineer | 60 mins | Certain |
| Technical Round 2 | Staff/Principal Engineer | 60–90 mins | Certain |
| Technical Round 3 | Display/Graphics Expert | 60 mins | Very Likely |
| Hiring Manager | Manager | 45–60 mins | Very Likely |
| Director / Bar Raiser (Sometimes) | Director / Senior Manager | 45 mins | Depends on Level |

For senior engineering roles, AMD interview processes commonly last **2–4 weeks**.

Some candidates report four interviews, while others (especially senior hires) report five or more including manager and director discussions.

---

# Round 1 — Resume + Linux Kernel Fundamentals

This round validates whether you have actually developed Linux kernel/display drivers.

## Resume Deep Dive

Expect questions like:

- Explain your current display driver architecture.
- Biggest debugging issue you've solved.
- Explain one kernel patch you wrote.
- What happens after your driver loads?
- Walk through `probe()`.

---

## Linux Kernel

Topics include:

- User space vs Kernel space
- Virtual memory
- Process scheduling
- Page tables
- Copy-on-write (COW)
- Slab allocator
- Kernel boot
- Device Tree
- PCI enumeration

---

## Driver Basics

Topics include:

- Platform driver
- PCI driver
- Character driver
- Module init/exit
- sysfs
- procfs
- debugfs

Glassdoor reports that AMD/Linux driver interviews begin almost immediately with resume discussion followed by kernel questions.

---

# Round 2 — Kernel Driver Deep Dive

This is usually the toughest round.

Expect detailed questions on:

## Driver Lifecycle

- `probe()`
- `remove()`
- `suspend()`
- `resume()`

---

## Interrupts

Topics include:

- Hard IRQ
- SoftIRQ
- Tasklets
- Workqueues
- Threaded IRQ

Typical question:

> Why can't you sleep inside interrupt context?

---

## DMA

Huge favorite at AMD.

Possible questions:

- DMA coherent vs streaming
- IOMMU
- Cache coherency
- Scatter-Gather
- `dma_map_single()`

Glassdoor candidates specifically mention DMA, interrupts, MMU/IOMMU, and driver probing as common topics.

---

## Synchronization

Expect questions on:

- Spinlock
- Mutex
- Semaphore
- Atomic variables
- RCU
- Completion

Scenario-based question:

> Two CPUs updating the same framebuffer.
>
> How do you synchronize?

---

# Round 3 — Display Driver Expertise

This role is specifically for Linux Display.

Expect almost every question around:

---

## DRM/KMS

They may ask you to explain:

```text
DRM
│
▼
CRTC
│
▼
Encoder
│
▼
Connector
│
▼
Plane
│
▼
Framebuffer
```

Difference between:

- Atomic Modesetting
- Legacy Modesetting

---

## Display Pipeline

Question:

> How does a frame travel from userspace to the monitor?

Expected explanation:

```text
Application
│
▼
Mesa
│
▼
DRM
│
▼
KMS
│
▼
GPU
│
▼
Display Controller
│
▼
DisplayPort
│
▼
Monitor
```

---

## DisplayPort

Very likely topics:

- Link Training
- AUX Channel
- HPD
- LTTPR
- MST
- SST
- DSC

Typical questions:

> Why does Link Training fail?

> How do you debug it?

---

## HDMI

Topics include:

- EDID
- CEC
- HDCP
- Hotplug

---

## eDP

Topics include:

- Panel Power Sequence
- Backlight
- PSR
- eDP AUX

---

## MIPI DSI

Basic understanding expected:

- DCS commands
- Video mode
- Command mode

---

# Round 4 — Debugging Round

AMD places a strong emphasis on debugging.

Expect scenario-based questions rather than textbook questions.

Examples:

### Scenario 1

> Monitor remains black after boot.

How will you debug?

---

### Scenario 2

> Display flickers only after suspend.

How do you isolate the issue?

---

Expected debugging flow:

```text
dmesg
│
▼
DRM logs
│
▼
tracepoints
│
▼
ftrace
│
▼
dynamic debug
│
▼
register dump
│
▼
oscilloscope / analyzer
```

Candidates also report dedicated debugging exercises involving race conditions and driver issues.

---

# Coding Round

Unlike Google,

Don't expect LeetCode Hard.

Instead, expect:

---

## C Programming

Topics include:

- Pointer questions
- Bit manipulation
- Memory allocation
- Linked List
- Ring Buffer
- Circular Queue

---

## Kernel Coding

Example implementations:

- Producer–Consumer
- Ring Buffer
- Circular Buffer
- Linked List
- Kernel List

Candidates consistently report C programming and systems-oriented coding rather than DSA-heavy interviews.

---

# Hiring Manager Round

This round mostly evaluates ownership.

Typical questions:

- Tell me about a difficult bug.
- Describe a cross-team conflict.
- Tell me about a missed deadline.
- How do you prioritize work?
- Why AMD?
- Open source contributions?
- Biggest technical achievement?
- Handling ambiguous requirements.

The AMD job description repeatedly emphasizes ownership, cross-functional collaboration, and driving issues to closure, so expect behavioral questions tied to real engineering examples.

---

# What They Usually Don't Ask

Compared to companies like Google or Amazon:

❌ Dynamic Programming

❌ Competitive Programming

❌ Graph Algorithms

❌ LeetCode Hard

Instead, expect:

✅ Linux Kernel

✅ Debugging

✅ C Programming

✅ Hardware Interaction

✅ Driver Architecture

✅ Display Pipeline

---

# Preparation Priority (Based on the JD)

The job description itself provides a strong signal for preparation.

Rank your study time roughly as follows:

| Priority | Topic | Chance of Being Asked |
|----------|-------|-----------------------|
| ⭐⭐⭐⭐⭐ | DRM/KMS Architecture | Very High |
| ⭐⭐⭐⭐⭐ | Linux Kernel Internals | Very High |
| ⭐⭐⭐⭐⭐ | C Programming | Very High |
| ⭐⭐⭐⭐⭐ | DisplayPort / eDP | Very High |
| ⭐⭐⭐⭐⭐ | Interrupts, DMA, Memory Management | Very High |
| ⭐⭐⭐⭐☆ | Debugging (ftrace, dmesg, dynamic debug) | High |
| ⭐⭐⭐⭐☆ | HDMI, MIPI DSI | High |
| ⭐⭐⭐⭐☆ | Wayland, X11, Mesa | High |
| ⭐⭐⭐☆☆ | PCIe, Device Tree | Medium |
| ⭐⭐☆☆☆ | General DSA | Low |

---

# Overall Prediction

For someone interviewing for **Senior Linux Display Driver Development Engineer**, a realistic interview process is:

1. Recruiter Screening
2. Technical Round 1
   - Linux kernel fundamentals
   - Resume discussion
3. Technical Round 2
   - Kernel drivers
   - DMA
   - Interrupts
   - Synchronization
   - Memory management
4. Technical Round 3
   - DRM/KMS
   - DisplayPort
   - HDMI
   - eDP
   - Debugging
5. Hiring Manager
   - Technical ownership
   - Behavioral questions
6. Director / Staff Interview (Optional)
   - Architecture
   - Cross-team collaboration
   - Technical leadership

---

# Final Assessment

Given the responsibilities in the job description, expect **80–90%** of the technical discussion to revolve around:

- Linux Kernel
- DRM/KMS
- Display Drivers
- DisplayPort
- HDMI
- eDP
- DMA
- Interrupts
- Synchronization
- Real-world Debugging
- Hardware–Software Interaction

with relatively little emphasis on traditional algorithmic coding or LeetCode-style problems.
