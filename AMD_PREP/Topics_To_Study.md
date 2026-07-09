# Linux Display Driver Interview Preparation Roadmap

## Target Role
Linux Display Driver Engineer (DRM/KMS, DisplayPort, eDP, HDMI, MIPI DSI, Wayland, X11, Linux Kernel)

---

# 1. Linux Display Architecture (Highest Priority)

## Study

- Complete Linux graphics stack
- Graphics architecture overview
- Display pipeline
- Framebuffer vs DRM
- Display controller
- GPU vs Display Engine
- Display Manager
- Display Hardware blocks

## Understand

Application
↓
Wayland / X11
↓
Mesa
↓
libdrm
↓
DRM/KMS
↓
Display Driver
↓
Display Controller
↓
HDMI / DP / eDP / DSI
↓
Display Panel / Monitor

## Interview Questions

- Explain Linux Display Architecture.
- What happens when an application draws a frame?
- Difference between GPU and Display Controller?
- Explain end-to-end graphics pipeline.

---

# 2. DRM/KMS (Most Important)

## DRM Overview

- Why DRM?
- DRM Architecture
- DRM Subsystems
- DRM Core
- DRM Driver

## DRM Objects

- CRTC
- Connector
- Encoder
- Plane
- Framebuffer
- Bridge
- Panel
- Property

## KMS

- Modesetting
- Atomic Modesetting
- Legacy Modesetting
- Universal Planes
- Atomic Commit
- Page Flip
- VBlank

## DRM Memory

- GEM
- TTM (Basics)
- Dumb Buffers
- DMA-BUF
- PRIME
- Scatter-Gather

## DRM Helpers

- drm_bridge
- drm_panel
- drm_connector
- drm_encoder

## Important APIs

- drm_dev_register()
- drm_mode_config_init()
- drm_atomic_helper_commit()
- drm_connector_funcs
- drm_crtc_funcs
- drm_plane_funcs

## Interview Questions

- Explain DRM Architecture.
- Difference between DRM and KMS.
- Explain Atomic Modesetting.
- Explain Page Flip.
- Explain VBlank.
- What is CRTC?
- Why Encoder?
- Difference between Connector and Encoder?
- Explain Plane.
- Primary Plane vs Overlay Plane vs Cursor Plane.
- Explain DRM Pipeline.
- What happens during Modesetting?
- What is Atomic Commit?
- Explain GEM.
- What is DMA-BUF?

---

# 3. Linux Display Pipeline

## Study

Display Initialization

Display Enablement

Display Bring-up

Framebuffer Creation

Modesetting

Display Refresh

Buffer Management

Synchronization

Display Composition

Overlay

Hardware Planes

Pipeline Configuration

## Interview Questions

- Explain complete Display Pipeline.
- Display Bring-up sequence.
- What happens after boot?
- How is framebuffer displayed?
- Explain Overlay Plane.
- Explain Display Composition.

---

# 4. DisplayPort (Very Important)

## DP Basics

- DP Architecture
- Physical Layer
- Protocol Layer
- Main Link
- AUX Channel
- HPD
- LTTPR

## Link Training

- Clock Recovery
- Channel Equalization
- Training Pattern 1
- Training Pattern 2
- Training Pattern 3
- Link Rate
- Lane Count
- Voltage Swing
- Pre-emphasis

## DP Components

- EDID
- DPCD
- AUX Transactions
- Sink
- Source
- PHY

## Features

- SST
- MST
- DSC
- HDCP (Basics)
- Adaptive Sync (Basics)

## Compliance

- VESA Compliance
- Link Training Tests
- PHY Tests
- Electrical Tests
- Protocol Compliance
- CTS

## Interview Questions

- Explain DisplayPort Architecture.
- Explain Link Training.
- Why Link Training fails?
- AUX Channel?
- HPD?
- DPCD?
- EDID?
- Difference between HDMI and DP?
- Explain MST.
- Explain SST.
- Explain DSC.
- Explain Compliance Testing.

---

# 5. eDP

## Study

- eDP Architecture
- Difference between DP and eDP
- Internal Display
- Panel Interface
- Backlight Control
- Power Sequencing
- PSR (Panel Self Refresh)
- eDP AUX

## Interview Questions

- Difference between DP and eDP?
- Explain eDP Power Sequence.
- Explain Panel Self Refresh.

---

# 6. HDMI

## Study

- HDMI Architecture
- TMDS
- EDID
- CEC
- HPD
- HDCP (Basics)
- HDMI Initialization
- HDMI PHY

## Interview Questions

- HDMI Initialization.
- Explain EDID.
- Explain HPD.
- Difference between HDMI and DP.
- What is TMDS?

---

# 7. MIPI DSI

## Study

- DSI Architecture
- D-PHY
- Command Mode
- Video Mode
- Packet Types
- Virtual Channels
- Timing
- Panel Initialization

## Interview Questions

- Explain MIPI DSI.
- Difference between Video Mode and Command Mode.
- D-PHY?
- Packet Types?

---

# 8. Linux Kernel Internals

## Memory

- kmalloc
- kzalloc
- vmalloc
- DMA Allocation
- Cache Coherency

## Interrupts

- Interrupt Flow
- IRQ
- Threaded IRQ
- Shared IRQ

## Synchronization

- Spinlock
- Mutex
- Semaphore
- Completion
- Wait Queue

## Execution Context

- Process Context
- Interrupt Context
- Bottom Half
- SoftIRQ
- Tasklet
- Workqueue

## Driver Model

- Platform Driver
- Probe
- Remove
- Suspend
- Resume
- Runtime PM

## Device Tree

- Nodes
- Compatible
- GPIO
- Clock
- Regulator
- Interrupt

## Interview Questions

- Interrupt vs Polling.
- Spinlock vs Mutex.
- kmalloc vs vmalloc.
- Explain DMA.
- Explain Probe.
- Explain Device Tree.
- Explain Runtime PM.

---

# 9. Linux Graphics Stack

## Components

- Wayland
- Weston
- X11
- XWayland
- Mesa
- libdrm
- EGL
- GBM
- OpenGL
- Vulkan (Basics)

## Study

How these interact.

## Interview Questions

- Explain Wayland.
- Difference between Wayland and X11.
- Why XWayland?
- What is Weston?
- What is Mesa?
- What is libdrm?
- Explain EGL.

---

# 10. Kernel Debugging

## Tools

- printk
- dmesg
- trace_printk
- ftrace
- perf
- kgdb
- gdb
- crash
- debugfs
- sysfs
- procfs
- dynamic debug

## Debugging

- Display not detected
- Black Screen
- Wrong Resolution
- Hotplug Failure
- Link Training Failure
- Driver Probe Failure
- Kernel Panic

## Interview Questions

- How do you debug Display Driver?
- Debug black screen.
- Debug Probe Failure.
- Debug Interrupt issue.

---

# 11. Display Bring-up

## Sequence

Bootloader

↓

Kernel

↓

Driver Probe

↓

Device Tree

↓

Clock Enable

↓

Power Enable

↓

GPIO

↓

Panel Init

↓

DRM Registration

↓

Connector Detection

↓

Modesetting

↓

Framebuffer

↓

Display ON

## Interview Questions

- Explain Display Bring-up.
- Explain Panel Initialization.
- Explain Driver Probe.

---

# 12. Display Compliance

## Study

- VESA
- CTS
- Compliance Testing
- DP Compliance
- HDMI Compliance
- Link Training Tests
- DPCD Validation
- EDID Validation

## Interview Questions

- Explain Compliance Testing.
- Why Compliance is required?
- What is CTS?

---

# 13. Root Cause Analysis

Prepare Real Examples

- Black Screen
- Flickering
- Wrong Resolution
- HPD Failure
- DP Failure
- HDMI Failure
- Dock Issues
- Multi-monitor
- Regression
- Wayland Issue
- X11 Issue
- Kernel Crash

Prepare

Problem

↓

Analysis

↓

Logs

↓

Debugging

↓

Root Cause

↓

Fix

↓

Validation

---

# 14. Regression Analysis

## Study

- Regression Testing
- Root Cause
- Git Bisect
- Kernel Regression
- Driver Regression

## Interview Questions

- Explain Regression.
- How do you identify regression?
- How do you fix regression?

---

# 15. C Programming

## Topics

Pointers

Double Pointer

Function Pointer

Structure

Union

Enum

Bit Field

Bit Manipulation

Memory Layout

const

volatile

typedef

Macros

Inline Functions

Preprocessor

Linked List

Dynamic Memory

Interview Coding

- Reverse Linked List
- String Reverse
- Bit Operations
- Memory Questions
- Structure Alignment

---

# 16. Linux Device Drivers

## Study

Character Driver

Platform Driver

PCI Driver (Basics)

Driver Model

Probe

Remove

Interrupt

DMA

Clock Framework

Regulator Framework

GPIO

Pinctrl

Device Tree

---

# 17. DMA

## Topics

- DMA Basics
- Coherent DMA
- Streaming DMA
- Cache Coherency
- Scatter Gather
- IOMMU (Basics)

---

# 18. Linux Memory Management

## Study

- Virtual Memory
- Physical Memory
- Page Tables
- Buddy Allocator
- Slab Allocator
- CMA
- MMU

---

# 19. Interrupts

## Study

- IRQ
- ISR
- Threaded IRQ
- Bottom Half
- SoftIRQ
- Tasklet
- Workqueue

---

# 20. Synchronization

## Topics

- Mutex
- Spinlock
- Semaphore
- Completion
- Wait Queue
- Atomic Variables

---

# 21. Embedded Linux

## Study

- Boot Process
- U-Boot
- Device Tree
- Kernel
- RootFS
- Init
- BSP
- PetaLinux

---

# 22. BSP Development

## Study

- Kernel Configuration
- Device Tree
- Bootloader
- RootFS
- Package Integration

---

# 23. Performance Optimization

## Study

- DMA Optimization
- AXI Throughput
- DDR Bandwidth
- Cache
- Zero Copy
- Buffer Optimization

---

# 24. Multimedia

## Study

- V4L2
- GStreamer
- Media Controller
- Video Pipeline
- H.264
- H.265

---

# 25. System Design

Prepare

- Design Linux Display Driver
- Design DP Driver
- Design HDMI Driver
- Design Display Pipeline
- Design DRM Driver

---

# 26. Behavioral Questions

Prepare STAR format examples for:

- Ownership
- Leadership
- Regression Handling
- Root Cause Analysis
- Production Issue
- Debugging
- Cross-team Collaboration
- Prioritization
- Conflict Resolution
- Mentoring
- Technical Presentations
- Customer Issues

---

# 27. Resume Deep Dive

Be prepared to explain every line of your resume.

## Capgemini

- Weston
- Wayland
- X11
- XWayland
- libdrm
- backend-drm
- renderer-gl
- eDP
- HDMI
- Hotplug
- Regression Investigation
- Root Cause Analysis

## Softnautics

- DisplayPort Compliance
- Link Training
- Multimedia
- GStreamer
- H.264/H.265

## iWave

- V4L2
- Driver Development
- BSP
- HDMI
- DP
- Video Pipeline
- Hotplug
- AXI
- DDR
- DMA

---

# 28. Practical Debugging Scenarios

Practice explaining how you would debug:

- Black screen after boot
- Monitor not detected
- eDP panel not powering up
- HDMI hotplug not working
- DP link training failure
- Wrong resolution selected
- Display flickering
- Overlay corruption
- Wayland rendering issue
- X11 display issue
- Driver probe failure
- Interrupt not triggering
- DMA timeout
- Memory corruption
- Kernel panic after display initialization

---

# 29. Recommended Resources

## Linux Kernel Documentation

- DRM/KMS Documentation
- Device Driver Documentation
- DMA API Documentation

## Books

- Linux Device Drivers (LDD3)
- Linux Kernel Development – Robert Love
- Understanding the Linux Kernel
- Professional Embedded ARM Development

## YouTube / Talks

- XDC (X.Org Developers Conference)
- Embedded Linux Conference (ELC)
- Linux Plumbers Conference
- DRM/KMS talks by Intel, AMD, Collabora, and Bootlin

---

# Final Interview Checklist

- ✅ DRM/KMS Architecture
- ✅ DRM Objects (CRTC, Connector, Encoder, Plane)
- ✅ Atomic Modesetting
- ✅ Display Pipeline
- ✅ Display Bring-up
- ✅ DisplayPort (Link Training, AUX, DPCD, EDID)
- ✅ eDP
- ✅ HDMI
- ✅ MIPI DSI
- ✅ Wayland
- ✅ Weston
- ✅ X11
- ✅ XWayland
- ✅ Mesa
- ✅ libdrm
- ✅ Linux Kernel Internals
- ✅ Device Drivers
- ✅ DMA
- ✅ Interrupts
- ✅ Memory Management
- ✅ Device Tree
- ✅ Kernel Debugging
- ✅ Display Compliance
- ✅ Regression Analysis
- ✅ Root Cause Analysis
- ✅ BSP Development
- ✅ GStreamer
- ✅ V4L2
- ✅ C Programming
- ✅ Behavioral Questions
- ✅ Resume Deep Dive
- ✅ Practical Debugging Scenarios
