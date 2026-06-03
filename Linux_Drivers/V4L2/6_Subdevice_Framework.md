# V4L2 Subdevice Framework - Complete Understanding

# Introduction

Modern multimedia hardware is no longer a simple device that captures data and sends it directly to an application.

A typical camera pipeline inside a modern SoC contains several hardware blocks:

```text
Image Sensor
     |
     V
CSI Receiver
     |
     V
ISP
     |
     V
Scaler
     |
     V
Video Device
```

Each block performs a specific task.

For example:

* Sensor captures raw pixels.
* CSI receives MIPI data.
* ISP performs image processing.
* Scaler resizes the image.
* Video Device provides the final stream to userspace.

Although all these blocks participate in the same camera pipeline, they have completely different responsibilities.

Linux therefore needs a mechanism to represent and manage each block independently.

This is the purpose of the V4L2 Subdevice Framework.

---

# What is a Subdevice?

A subdevice represents an internal multimedia processing component within a larger media pipeline.

Examples include:

```text
Camera Sensor
CSI Receiver
ISP
Scaler
HDMI Decoder
TV Decoder
Video Bridge
```

These components process, transform, receive, or generate multimedia data.

Unlike a video device, a subdevice usually does not provide frame buffers directly to applications.

Instead, it participates in building the complete data path.

---

# Understanding the Difference Between Video Device and Subdevice

A common source of confusion is understanding the role of:

```text
video_device
```

versus

```text
v4l2_subdev
```

Consider the following pipeline:

```text
Application
      |
      V
/dev/video0
      |
      V
Scaler
      |
      V
ISP
      |
      V
CSI
      |
      V
Sensor
```

The application only communicates with:

```text
/dev/video0
```

The application never directly communicates with:

```text
Sensor
CSI
ISP
Scaler
```

These internal components are represented as subdevices.

Therefore:

```text
video_device
=
User-facing interface

v4l2_subdev
=
Internal hardware block
```

---

# Why Separate Internal Blocks?

Consider a camera sensor.

The sensor may support:

```text
1920x1080
1280x720
640x480

RAW8
RAW10
RAW12
```

The ISP may support:

```text
RAW10
RAW12
RAW14
```

The scaler may support:

```text
Output resizing
Cropping
Rotation
```

Each block has its own configuration requirements.

Trying to manage everything through a single video device would become extremely complicated.

The subdevice framework allows each hardware block to manage its own functionality independently.

---

# The v4l2_subdev Structure

Every subdevice is represented by:

```c
struct v4l2_subdev
```

This structure acts as a generic representation of a multimedia component.

Conceptually:

```text
v4l2_subdev
    |
    +---- Sensor Driver
    |
    +---- CSI Driver
    |
    +---- ISP Driver
    |
    +---- Scaler Driver
```

Each driver fills in operation callbacks according to its functionality.

---

# Why Operation Callbacks Are Used

Different hardware blocks perform different tasks.

A sensor driver knows how to:

```text
Set exposure
Set gain
Set frame rate
```

An ISP driver knows how to:

```text
Configure image processing
Set white balance
Enable noise reduction
```

A scaler driver knows how to:

```text
Resize image
Crop image
```

Linux provides a common framework, while each driver implements only the operations it needs.

---

# Subdevice Operation Categories

Operations are grouped into three major categories:

```text
Core Operations
Video Operations
Pad Operations
```

---

# Core Operations

Core operations contain generic device management functionality.

These operations are not directly related to image streaming.

Examples include:

```text
Device Reset
Power Management
Status Reporting
Debug Support
```

A sensor may use core operations to:

```text
Reset sensor registers
Power down the device
Print status information
```

An ISP may use core operations to:

```text
Initialize internal hardware
Reset processing blocks
```

Core operations provide common management functionality shared across all multimedia devices.

---

# Video Operations

Video operations are responsible for controlling streaming behavior.

The most important operation is:

```c
s_stream()
```

This function starts or stops data flow.

When streaming starts:

```text
Sensor begins generating frames

CSI begins receiving data

ISP begins processing data

Scaler begins producing output
```

When streaming stops:

```text
All hardware blocks stop data flow
```

The operation propagates through the entire pipeline.

---

# Understanding Stream Start

Suppose a user starts video capture.

The pipeline:

```text
Sensor
   |
CSI
   |
ISP
   |
Video Device
```

must become active.

Linux therefore enables streaming in each component.

Conceptually:

```text
Enable Sensor

Enable CSI

Enable ISP

Enable Output Path
```

Only after every stage is ready can video flow successfully.

---

# Pad Operations

Pad operations are arguably the most important part of the subdevice framework.

They manage communication between connected multimedia blocks.

To understand why they are needed, consider:

```text
Sensor
   |
CSI
   |
ISP
```

The sensor generates:

```text
1920x1080
RAW10
```

The CSI must understand:

```text
1920x1080
RAW10
```

The ISP must also understand:

```text
1920x1080
RAW10
```

All blocks must agree on the format.

Pad operations make this possible.

---

# Pads

A pad represents a connection point.

Example:

```text
Sensor
   |
Source Pad
```

Data leaves through a source pad.

```text
Sink Pad
   |
CSI
```

Data enters through a sink pad.

A connection is established between pads.

---

# Format Negotiation

One of the main responsibilities of pad operations is format negotiation.

Imagine:

```text
Sensor supports:
RAW10
RAW12

ISP supports:
RAW12 only
```

The framework must determine:

```text
Which format can both devices use?
```

The answer is:

```text
RAW12
```

The format is propagated through the pipeline so every component uses a compatible configuration.

---

# get_fmt()

This operation retrieves the current format configuration.

Information may include:

```text
Width
Height
Media Bus Format
Colorspace
Field Type
```

Example:

```text
1920x1080
RAW10
```

This allows connected devices to understand how data is currently configured.

---

# set_fmt()

This operation changes the format.

Example request:

```text
1280x720
RAW12
```

The subdevice evaluates the request and configures itself accordingly.

Format changes typically propagate through neighboring subdevices to maintain compatibility.

---

# enum_mbus_code()

This operation enumerates supported media bus formats.

Examples:

```text
MEDIA_BUS_FMT_SRGGB10_1X10

MEDIA_BUS_FMT_SRGGB12_1X12

MEDIA_BUS_FMT_UYVY8_2X8
```

These formats describe how data moves between hardware blocks.

This is different from userspace pixel formats such as:

```text
V4L2_PIX_FMT_NV12

V4L2_PIX_FMT_YUYV
```

Media bus formats describe internal pipeline communication.

Pixel formats describe the final image format visible to applications.

---

# TRY Format and ACTIVE Format

The framework supports two format states.

---

## TRY Format

TRY format is temporary.

It allows validation.

Example:

```text
Can this sensor support:

3840x2160
RAW12 ?
```

The framework checks compatibility.

No hardware changes occur.

---

## ACTIVE Format

ACTIVE format represents the real hardware configuration.

After a format is accepted:

```text
Sensor registers updated

CSI configured

ISP configured
```

Streaming will use this format.

---

# Asynchronous Subdevice Registration

In a complex system, not all hardware appears at the same time.

For example:

```text
ISP Driver Loaded
```

while:

```text
Sensor Driver
Still Not Loaded
```

This is very common because sensors often reside on an I2C bus.

I2C probing may occur later.

Linux therefore needs a way to connect components dynamically.

---

# Async Notifier

The framework provides:

```c
struct v4l2_async_notifier
```

This object waits for missing devices.

Example:

```text
CSI Driver Waiting For:

IMX219 Sensor
```

The sensor eventually appears.

The framework automatically creates the relationship.

---

# Registration Flow

Step 1:

```text
Sensor Driver Loads
```

Step 2:

```text
Subdevice Created
```

Step 3:

```c
v4l2_async_register_subdev()
```

called.

Step 4:

```text
Framework Searches
For Matching Endpoints
```

Step 5:

```text
Sensor Connected To Pipeline
```

The media graph becomes complete.

---

# Relationship with Media Controller

The Media Controller Framework and Subdevice Framework work together.

Media Controller describes:

```text
Hardware topology
```

Subdevice Framework manages:

```text
Hardware behavior
```

Media Controller answers:

```text
Who is connected to whom?
```

Subdevice Framework answers:

```text
How should each component behave?
```

---

# Complete Pipeline Example

Consider:

```text
IMX219 Sensor
      |
CSI Receiver
      |
ISP
      |
Scaler
      |
Video Device
```

Media Controller creates:

```text
Entities
Pads
Links
```

Subdevices implement:

```text
Format negotiation

Streaming control

Power management

Device-specific configuration
```

Video Device provides:

```text
Buffer management

Queue/dequeue

Stream control

Userspace access
```

Together they form the complete Linux camera architecture.

---

# Architecture Summary

```text
Application
      |
      V
Video Device
      |
      V
Subdevice (Scaler)
      |
      V
Subdevice (ISP)
      |
      V
Subdevice (CSI)
      |
      V
Subdevice (Sensor)
```

Each layer has a specific responsibility.

The application interacts with the video device.

The video device depends on subdevices.

Subdevices communicate through pads and links.

The Media Controller Framework describes the topology.

The V4L2 framework manages streaming and buffer handling.

Together they provide a scalable architecture capable of supporting modern multimedia systems ranging from simple webcams to complex multi-camera automotive platforms.

