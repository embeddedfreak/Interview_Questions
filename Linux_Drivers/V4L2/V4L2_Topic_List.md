
# V4L2 Senior Embedded Linux Study Roadmap

## Table of Contents

1. [V4L2 Architecture](1_V4L2_Architecture.md)
2. [IOCTLs](2_V4L2_IOCTL.md)
3. [VB2 Framework](3_vb2_Framework.md)
4. [Buffer Management](4_Buffer_Management.md)
5. [Media Controller Framework](5_Media_Ctl.md)
6. [Subdevice Framework](6_Subdevice_Framework.md)
7. [DMA](#7-dma-very-important)
8. [Device Tree for V4L2](#8-device-tree-for-v4l2)
9. [I2C Sensor Drivers](#9-i2c-sensor-drivers)
10. [Final Senior Interview Checklist](#10-final-senior-interview-checklist)

---


# V4L2 Senior Embedded Linux Study Roadmap

Below is a focused Senior Embedded Linux V4L2 study roadmap for the exact topics you mentioned.

# 1. V4L2 Architecture (Must Know First)

Understand who talks to whom.

## Architecture

```text
Userspace Application
      |
      v
V4L2 API / ioctl()
      |
      v
Video Node Driver
      |
      v
V4L2 Framework
      |
      +---- Media Controller
      +---- VB2
      +---- Subdevices
      |
      v
Hardware
```

## Study these concepts

### Purpose of V4L2

Why Linux introduced V4L2.

Used for:

* Video capture
* Video output
* Camera pipeline
* Codec devices
* Radio devices

### Driver Types

Study difference.

#### Video Device Driver

Creates `/dev/videoX`

Examples:

* Capture device
* Video output device

Structure:

```c
struct video_device
```

#### Subdevice Driver

Internal hardware blocks.

Examples:

* Sensor
* CSI receiver
* ISP
* HDMI decoder

Structure:

```c
struct v4l2_subdev
```

### Core Structures

Study purpose of each.

```c
struct v4l2_device
```

Top-level V4L2 container.

```c
struct video_device
```

Represents:

```text
/dev/video0
```

```c
struct v4l2_subdev
```

Represents internal components.

```c
struct media_device
```

Media framework root object.

### Registration Flow

Study complete flow.

```c
v4l2_device_register()

video_register_device()

v4l2_async_register_subdev()
```

### File Operations

Study:

* open()
* release()
* poll()
* mmap()
* unlocked_ioctl()

### Interview Question

Difference between video node and subdevice?

---

# 2. IOCTLs (Very Important)

This is V4L2 userspace ↔ driver communication.

Application uses:

```c
ioctl(fd, command, arg);
```

## Capability IOCTL

### VIDIOC_QUERYCAP

Purpose:

Driver capability query.

Study:

```c
struct v4l2_capability
```

Example:

* Capture support?
* Streaming support?

## Format Negotiation

Very important.

### Enumerate formats

```c
VIDIOC_ENUM_FMT
```

Driver supported formats.

### Get current format

```c
VIDIOC_G_FMT
```

### Set format

```c
VIDIOC_S_FMT
```

Study structure:

```c
struct v4l2_format
```

Fields:

* width
* height
* pixelformat
* bytesperline

## Buffer Management IOCTLs

Must master.

### Allocate buffers

```c
VIDIOC_REQBUFS
```

### Query buffer

```c
VIDIOC_QUERYBUF
```

### Queue buffer

```c
VIDIOC_QBUF
```

### Dequeue buffer

```c
VIDIOC_DQBUF
```

## Streaming Control

### Start

```c
VIDIOC_STREAMON
```

### Stop

```c
VIDIOC_STREAMOFF
```

Must understand complete sequence:

```text
open()

QUERYCAP

S_FMT

REQBUFS

QUERYBUF

mmap()

QBUF

STREAMON

DQBUF

STREAMOFF
```

Interview favorite.

---

# 3. VB2 Framework (Extremely Important)

VB2 = Videobuf2

Core V4L2 buffer framework.

Almost guaranteed interview topic.

Purpose:

Handles:

* buffer allocation
* queues
* synchronization
* streaming state

Study:

```c
struct vb2_queue
```

Main buffer queue object.

## Important callbacks

Must know.

### queue_setup()

Purpose:

Decide:

* buffer count
* plane count
* buffer size

### buf_prepare()

Purpose:

Validate buffer.

### buf_queue()

Purpose:

Buffer enters driver queue.

### start_streaming()

Purpose:

Hardware DMA starts.

### stop_streaming()

Purpose:

Stop DMA.

### wait_prepare()

Sleep handling.

### wait_finish()

Resume.

## VB2 Lifecycle

Very important.

Understand flow.

```text
REQBUFS

↓

VB2 allocates buffers

↓

QBUF

↓

buf_queue()

↓

Driver queue

↓

STREAMON

↓

start_streaming()

↓

DMA starts

↓

IRQ frame complete

↓

DQBUF
```

Questions:

* Explain complete VB2 lifecycle.
* Difference between VB2 and legacy videobuf.

---

# 4. Buffer Management (Very Important)

Study deeply.

## Memory Models

### MMAP

Kernel allocates buffers.

Userspace maps.

Most common.

### USERPTR

Userspace allocates memory.

Kernel uses pointer.

### DMABUF

Cross-device buffer sharing.

GPU ↔ V4L2 ↔ Display.

Very important for senior roles.

### Interview Question

MMAP vs USERPTR vs DMABUF.

## Buffer Lifecycle

Study.

```text
Allocate

↓

Queue

↓

DMA write/read

↓

Interrupt completion

↓

Dequeue

↓

Reuse
```

## Multi-planar Buffers

Study.

Formats:

* NV12
* YUV420

Need:

```c
struct v4l2_plane
```

## Zero Copy Pipeline

Very important.

Goal:

Avoid memcpy.

Questions:

* Why video avoids memcpy?
* How DMABUF enables zero copy?

---

# 5. Media Controller Framework

Senior-level must know.

Used for complex camera pipelines.

Study concepts.

## Media Graph

Represents hardware topology.

Example:

```text
Sensor
   |
CSI Receiver
   |
ISP
   |
Scaler
   |
Video Node
```

## Entity

Hardware block.

Structure:

```c
struct media_entity
```

Examples:

* sensor
* ISP
* decoder

## Pad

Input/output connection point.

Types:

* source pad
* sink pad

## Link

Connects entities.

## Pipeline

Active streaming path.

Study APIs:

```c
media_entity_pads_init()

media_create_pad_link()
```

Tool:

```bash
media-ctl -p
```

Must know.

Questions:

* Why media controller needed?
* What problem does it solve?

---

# 6. Subdevice Framework

Very important.

Subdevice = internal pipeline block.

Examples:

* Camera sensor
* CSI
* ISP
* HDMI decoder

Structure:

```c
struct v4l2_subdev
```

## Subdevice Operations

Study.

### Core Ops

```c
v4l2_subdev_core_ops
```

### Video Ops

```c
v4l2_subdev_video_ops
```

### Pad Ops

Very important.

```c
v4l2_subdev_pad_ops
```

Study:

```c
get_fmt()
set_fmt()
enum_mbus_code()
```

## Async Framework

Must know.

Used because:

Sensor may probe later.

Study:

```c
v4l2_async_notifier
```

Registration:

```c
v4l2_async_register_subdev()
```

Questions:

Difference:

```text
video_device
vs
v4l2_subdev
```

---

# 7. DMA (Very Important)

Multimedia lives on DMA.

Purpose:

CPU cannot copy huge video frames efficiently.

DMA moves frames.

Study.

## DMA Basics

CPU setup.

DMA transfers.

Interrupt completion.

## DMA Types

### Coherent DMA

Cache synchronized.

### Streaming DMA

Explicit sync needed.

Study APIs:

```c
dma_alloc_coherent()

dma_map_single()

dma_unmap_single()
```

## Scatter Gather DMA

Very important.

Non-contiguous memory.

## DMA Flow in V4L2

Study.

```text
VB2 buffer

↓

Physical address

↓

DMA programmed

↓

Frame capture

↓

IRQ

↓

Buffer complete
```

Questions:

* Why DMA essential in video?
* Cache coherency issue?

---

# 8. Device Tree for V4L2

Very important.

Study basics.

## Device Node

Example.

```dts
sensor@1a {
        compatible = "sony,imx219";
        reg = <0x1a>;
};
```

## Properties

Study.

* compatible
* reg
* interrupts
* clocks
* gpios
* power-domains

## Graph Binding

Must know for media.

Endpoints describe topology.

Example:

```text
Sensor endpoint
    |
CSI endpoint
```

Study:

* ports
* endpoint
* remote-endpoint

### Interview Question

How sensor connected to CSI in DT?

---

# 9. I2C Sensor Drivers

Very important.

Most camera sensors use I2C.

Study driver flow.

## Probe

Study.

```c
i2c_driver
```

Callbacks:

```c
probe()
remove()
```

## Register Access

Study.

Write register.

```c
i2c_smbus_write_byte_data()
```

Read register.

```c
i2c_smbus_read_byte_data()
```

### regmap framework

Senior interviews ask.

Study:

```c
regmap_read()

regmap_write()
```

## Sensor Initialization

Study typical flow.

```text
Power ON

↓

Reset

↓

Clock enable

↓

Write sensor registers

↓

Configure format

↓

Start streaming
```

## Sensor Controls

Study.

Examples:

* exposure
* gain
* frame rate

Uses:

```c
v4l2_ctrl_handler
```

## Streaming Functions

Study.

Usually:

```c
s_stream()
```

start/stop capture.

## Power Management

Important.

Study:

* runtime PM
* suspend/resume

---

# Final Senior Interview Checklist

Master these explanations.

* ✓ V4L2 architecture
* ✓ ioctl flow
* ✓ VB2 lifecycle
* ✓ MMAP / USERPTR / DMABUF
* ✓ Media graph
* ✓ Subdevice + async notifier
* ✓ DMA + cache coherency
* ✓ DT graph bindings
* ✓ I2C sensor probe + register programming

## Complete streaming path

```text
Userspace

→ ioctl

→ VB2

→ DMA

→ Sensor

→ IRQ

→ DQBUF
```

If you can explain this entire streaming pipeline end-to-end on a whiteboard, you are in strong shape for a Senior Multimedia Linux Driver interview.

