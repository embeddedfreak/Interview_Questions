# 2) V4L2 IOCTLs

## What is an IOCTL?

**IOCTL (Input Output Control)** is a mechanism used by user-space applications to communicate with kernel drivers and perform device-specific operations.

Unlike regular file operations such as:

* `read()`
* `write()`

multimedia devices require more advanced control such as:

* Querying device capabilities
* Setting video formats
* Allocating DMA buffers
* Starting and stopping streaming
* Controlling camera parameters
* Configuring memory types
* Retrieving frame metadata

These operations are performed through:

```c
ioctl(fd, command, arg);
```

where:

| Parameter | Description                            |
| --------- | -------------------------------------- |
| `fd`      | Device file descriptor                 |
| `command` | Requested operation                    |
| `arg`     | Structure containing input/output data |

---

## Why V4L2 Uses IOCTLs?

A camera is not a simple file.

The application must negotiate several parameters before receiving video frames:

* Resolution
* Pixel Format
* Frame Rate
* Buffer Count
* Memory Type
* Streaming Mode
* Colorspace Information

IOCTLs provide a standardized interface between applications and camera drivers.

### Example Applications

* GStreamer
* FFmpeg
* OpenCV
* Chrome Camera Application
* VLC
* WebRTC Applications

All communicate with camera drivers through V4L2 IOCTLs.

---

# 1. VIDIOC_QUERYCAP

## Purpose

Used to discover the capabilities of a V4L2 device.

This is usually the first IOCTL called after opening the device.

```c
struct v4l2_capability cap;

ioctl(fd, VIDIOC_QUERYCAP, &cap);
```

---

## Why is it Needed?

Before using a device, the application must know:

* Can it capture video?
* Can it output video?
* Does it support streaming?
* Does it support read/write access?
* Does it support multi-planar formats?
* Does it support metadata capture?

Without this check, the application may attempt unsupported operations.

---

## Important Structure

```c
struct v4l2_capability
{
    char driver[16];
    char card[32];
    char bus_info[32];
    __u32 capabilities;
};
```

### Fields Explanation

| Field          | Description              |
| -------------- | ------------------------ |
| `driver`       | Driver name              |
| `card`         | Device name              |
| `bus_info`     | Physical bus information |
| `capabilities` | Supported features       |

---

## Common Capability Flags

### Video Capture

```c
V4L2_CAP_VIDEO_CAPTURE
```

Meaning:

Device can capture video frames.

Examples:

* USB Camera
* MIPI CSI Camera
* Webcam

---

### Streaming Support

```c
V4L2_CAP_STREAMING
```

Meaning:

Supports buffer-based streaming.

Required for:

* `mmap()`
* DMA-based capture
* Zero-copy pipelines

---

### Common Additional Flags

```c
V4L2_CAP_VIDEO_CAPTURE_MPLANE
```

Multi-planar video capture support.

```c
V4L2_CAP_READWRITE
```

Supports traditional `read()` and `write()` operations.

```c
V4L2_CAP_DEVICE_CAPS
```

Indicates valid device capability information.

---

## Interview Question

### Why call QUERYCAP first?

**Answer:**

It verifies that the device supports the required functionality before performing format negotiation and buffer allocation.

It also helps applications determine the correct capture path and memory model supported by the driver.

---

# 2. Format Negotiation

## Why Format Negotiation is Required?

The camera and application must agree on:

* Image Width
* Image Height
* Pixel Format
* Field Type
* Color Space
* Buffer Layout

This process is called **format negotiation**.

---

## Example

Application wants:

* 1920 × 1080
* NV12

Camera may support:

### Resolutions

* 640 × 480
* 1280 × 720
* 1920 × 1080

### Formats

* YUYV
* NV12
* MJPEG

Application selects one supported combination.

---

# VIDIOC_ENUM_FMT

## Purpose

Enumerates all formats supported by the driver.

Example:

```c
struct v4l2_fmtdesc fmt;
```

Possible output:

* YUYV
* NV12
* MJPEG
* RGB24

---

## Internal Driver Operation

Driver returns formats supported by:

* Sensor
* ISP
* Hardware pipeline
* Decoder blocks

This information helps applications choose compatible formats.

---

# VIDIOC_G_FMT

## Purpose

Get current active format.

Example:

```c
ioctl(fd, VIDIOC_G_FMT, &fmt);
```

Driver returns:

* Current Width
* Current Height
* Current Pixel Format
* Bytes Per Line
* Colorspace

Useful for debugging and validation.

---

# VIDIOC_S_FMT

## Purpose

Set desired format.

Example:

```c
fmt.fmt.pix.width = 1920;
fmt.fmt.pix.height = 1080;
fmt.fmt.pix.pixelformat = V4L2_PIX_FMT_NV12;

ioctl(fd, VIDIOC_S_FMT, &fmt);
```

---

## What Happens Internally?

Driver validates:

* Resolution supported?
* Format supported?
* Alignment constraints satisfied?
* ISP requirements met?

If unsupported:

* Driver adjusts values

or

* Returns an error

---

## Important Structure

```c
struct v4l2_format
```

### width

Image width.

Example:

```text
1920 pixels
```

### height

Image height.

Example:

```text
1080 pixels
```

### pixelformat

Defines how pixel data is stored.

Examples:

* NV12
* YUYV
* MJPEG
* RGB24

### bytesperline

Number of bytes required for one image row.

Example:

```text
1920 pixels
2 bytes per pixel

1920 × 2 = 3840 bytes
```

### sizeimage

Total buffer size required for one frame.

Used by applications when allocating buffers.

---

# Buffer Management (Most Important Topic)

Video capture is impossible without buffers.

Frames are continuously produced by the camera.

The driver needs memory locations where incoming frames can be stored.

---

## Why Multiple Buffers?

Single buffer causes:

* Frame drops
* Pipeline stalls
* Poor performance
* DMA starvation

Using multiple buffers enables:

* Capture
* Processing
* Display

to run simultaneously.

This is called:

## Pipelining

While DMA writes frame N+1:

* Application processes frame N
* Display shows frame N−1

This significantly improves throughput.

---

# VIDIOC_REQBUFS

## Purpose

Request buffers from driver.

Example:

```c
req.count = 4;

ioctl(fd, VIDIOC_REQBUFS, &req);
```

---

## What Happens Internally?

Driver:

* Allocates DMA-capable memory
* Creates buffer queue
* Initializes metadata
* Prepares DMA descriptors

Applications typically request:

* 4 buffers
* 8 buffers
* 16 buffers

depending on latency requirements.

---

## Memory Types

```c
V4L2_MEMORY_MMAP
```

Kernel allocated buffers mapped to user space.

```c
V4L2_MEMORY_USERPTR
```

Application provides buffers.

```c
V4L2_MEMORY_DMABUF
```

Buffers shared across devices without copying.

Used extensively in:

* ISP
* GPU
* Display
* AI accelerators

---

# VIDIOC_QUERYBUF

## Purpose

Retrieve information about each allocated buffer.

Example:

```c
ioctl(fd, VIDIOC_QUERYBUF, &buf);
```

Returns:

* Buffer Offset
* Buffer Length
* Memory Location
* Buffer Flags

---

## Why Needed?

The application needs this information before calling:

```c
mmap()
```

---

# mmap()

## Purpose

Maps kernel buffer memory into user space.

```c
ptr = mmap(...);
```

---

## Why mmap is Important?

Without mmap:

```text
Kernel → Copy → User Space
```

Extra CPU overhead.

With mmap:

```text
Kernel Memory ↔ User Space
```

Same memory shared.

Benefits:

* Lower CPU usage
* Lower latency
* Higher throughput
* Zero-copy access

This is one reason V4L2 achieves high-performance capture.

---

# VIDIOC_QBUF

## Purpose

Queue an empty buffer to driver.

Example:

```c
ioctl(fd, VIDIOC_QBUF, &buf);
```

Meaning:

> This buffer is available. Please fill it with video data.

---

## Internal Driver Action

Buffer enters:

```text
Incoming Capture Queue
```

DMA engine can now write frame data into it.

---

# VIDIOC_DQBUF

## Purpose

Retrieve a completed buffer.

Example:

```c
ioctl(fd, VIDIOC_DQBUF, &buf);
```

---

## Internal Driver Action

Driver removes buffer from:

```text
Completed Queue
```

and returns it to application.

Returned metadata often contains:

* Timestamp
* Buffer Index
* Frame Sequence Number
* Bytes Used
* Capture Flags

---

## Interview Question

### Why must DQBUF be followed by QBUF?

**Answer:**

After processing a frame, the buffer must be returned to the driver so it can be reused for future frame capture.

Failing to requeue buffers eventually causes the driver to run out of available buffers and streaming will stall.

---

# VIDIOC_STREAMON

## Purpose

Starts streaming.

```c
ioctl(fd, VIDIOC_STREAMON, &type);
```

---

## What Happens Internally?

Driver starts:

* Sensor Streaming
* CSI Receiver
* ISP Processing
* DMA Engine
* Buffer Queue Handling
* Interrupt Generation

Frames begin arriving into queued buffers.

---

## Important Note

At least one buffer must already be queued before calling:

```c
VIDIOC_STREAMON
```

Otherwise streaming may fail.

---

# VIDIOC_STREAMOFF

## Purpose

Stops streaming.

```c
ioctl(fd, VIDIOC_STREAMOFF, &type);
```

---

## Internal Driver Action

Stops:

* Sensor
* DMA
* Frame Scheduling
* Interrupt Handling

Buffers are returned to application.

Driver also flushes pending queues.

---

# Complete V4L2 Capture Flow

```text
Application                Driver

open()
  |
  +-------> Device Open

QUERYCAP
  |
  +-------> Check Capabilities

S_FMT
  |
  +-------> Configure Resolution
             Configure Pixel Format

REQBUFS
  |
  +-------> Allocate DMA Buffers

QUERYBUF
  |
  +-------> Get Buffer Information

mmap()
  |
  +-------> Map Buffers

QBUF
  |
  +-------> Queue Empty Buffers

STREAMON
  |
  +-------> Start Sensor
             Start DMA

DQBUF
  |
  +-------> Receive Captured Frame

Process Frame
  |
  +-------> Display
             Encode
             AI Processing

QBUF
  |
  +-------> Return Buffer

Repeat DQBUF/QBUF Loop

STREAMOFF
  |
  +-------> Stop Streaming

close()
```

---

# Simplified Buffer Queue Diagram

```text
              Camera Sensor
                    |
                    v
              DMA Engine
                    |
                    v

      +-------------------------+
      | Driver Buffer Queue     |
      +-------------------------+

         QBUF     QBUF
           |         |
           v         v

      [Buffer0] [Buffer1]

           ^
           |
         DQBUF

      Application
```

---

# Senior Interview Key Points

When discussing V4L2, always mention:

## Format Negotiation

Application and driver must agree on:

* Resolution
* Pixel Format
* Colorspace

before streaming begins.

---

## Buffer Queue Mechanism

V4L2 uses producer-consumer queues:

* Driver produces frames
* Application consumes frames

using the `QBUF/DQBUF` mechanism.

---

## mmap Zero-Copy Access

Buffers are shared between kernel and user-space, reducing CPU overhead and memory bandwidth usage.

---

## DMA-based Capture

Frames are transferred directly from hardware into memory without CPU involvement.

Benefits:

* Low latency
* High throughput
* Reduced CPU load

---

## Streaming State Machine

Typical states:

```text
OPEN
  ↓
CONFIGURED
  ↓
BUFFERS_ALLOCATED
  ↓
STREAMING
  ↓
STOPPED
  ↓
CLOSED
```

Understanding state transitions is important in driver debugging.

---

## DQBUF/QBUF Capture Loop

Core runtime capture loop:

```text
DQBUF
   ↓
Process Frame
   ↓
QBUF
   ↓
Repeat
```

This loop runs continuously while streaming is active.

---

## Latency vs Buffer Count Trade-off

More Buffers:

* Higher throughput
* Better stability
* Higher latency

Fewer Buffers:

* Lower latency
* Greater risk of frame drops

Choosing the correct buffer count is a common optimization task in embedded camera systems.

---

# One-Line Interview Summary

**V4L2 video capture is fundamentally a DMA-driven buffer queue system where applications negotiate formats, allocate buffers, queue them to the driver, start streaming, repeatedly perform DQBUF/QBUF operations to exchange frames, and stop streaming when capture is complete.**

