# AMD/Xilinx DisplayPort TX + PHY Architecture (Interview Notes)

For an **AMD/Xilinx DisplayPort TX** interview, the interviewer usually expects you to explain the **data flow 
from the video source to the DisplayPort connector**, while describing the function of each block. 
The following notes provide a structured explanation of the DisplayPort TX subsystem.

---

# DisplayPort TX + PHY Block Diagram

```text
             Video Source
          (AXI4-Stream / Native)
                    │
                    ▼
         Video Timing Controller
                    │
                    ▼
        DisplayPort TX Controller
  ┌────────────────────────────────┐
  │ Main Link Controller           │
  │ AUX Controller                 │
  │ Link Training Engine           │
  │ Audio Packet Generator         │
  │ HDCP Engine                    │
  │ MST/SST Manager                │
  └────────────────────────────────┘
                    │
           Video PHY Interface
                    │
                    ▼
             DisplayPort PHY
   ┌──────────────────────────────┐
   │ PLL                          │
   │ Serializer                   │
   │ Lane Drivers                 │
   │ Pre-emphasis                 │
   │ Output Buffers               │
   └──────────────────────────────┘
                    │
                    ▼
             DisplayPort Connector
```

> This reflects the DisplayPort TX subsystem architecture described in AMD's 
**DisplayPort TX Subsystem Product Guide (PG299)**, where the subsystem includes 
the DisplayPort TX core, Main Link, AUX channel, Secondary Data handling, and interfaces to the Video PHY.

---

# DisplayPort TX Architecture (Point-to-Point Explanation)

## 1. Video Source

### What it is

The Video Source generates the pixel data that is transmitted over DisplayPort.

### Examples

- Frame Buffer
- Video DMA (VDMA)
- Image Processing Pipeline
- Test Pattern Generator (TPG)

The video is typically transferred using:

- AXI4-Stream Interface
- Native Video Interface

### Interview Statement

> "The DisplayPort TX subsystem receives pixel data from the video pipeline through 
AXI4-Stream or the Native Video interface."

---

## 2. Video Timing Controller (VTC)

### Purpose

The Video Timing Controller (VTC) generates timing signals required for each video frame.

It generates:

- Horizontal Sync (HSYNC)
- Vertical Sync (VSYNC)
- Active Video
- Blanking Intervals

Without these timing signals, the DisplayPort transmitter cannot determine:

- Frame Start
- Frame End
- Active Pixels

### Interview Statement

> "The VTC provides synchronization signals like HSYNC and VSYNC, which are required 
by the DisplayPort controller to correctly packetize the video."

---

# 3. DisplayPort TX Controller

The DisplayPort TX Controller is the digital protocol engine.

It converts raw video into a DisplayPort-compliant stream.

It consists of several functional blocks.

---

## 3.1 Main Link Controller

This is the most important block inside the DisplayPort TX controller.

### Responsibilities

- Packetizes video data
- Scrambles data
- Performs line coding (8b/10b for DP 1.4 link rates)
- Distributes data across lanes
- Controls the Main Link

### Interview Statement

> "The Main Link Controller formats pixel data into DisplayPort symbols and distributes 
them across one, two, or four high-speed lanes."

---

## 3.2 AUX Controller

DisplayPort includes a separate bidirectional management channel called the **AUX Channel**.

### Functions

- Read DPCD Registers
- Read EDID
- I²C-over-AUX Communication
- HDCP Communication

Unlike the Main Link, the AUX channel is bidirectional.

### Interview Statement

> "The AUX controller communicates with the monitor to read capabilities and configure the link."

---

## 3.3 Link Training Engine

A commonly discussed interview topic.

Before transmitting video, the transmitter determines whether the communication channel is reliable.

### Link Training Sequence

```text
HPD Detected
      │
      ▼
Read DPCD
      │
      ▼
Clock Recovery
      │
      ▼
Channel Equalization
      │
      ▼
Training Success
      │
      ▼
Video Transmission
```

During Link Training, the transmitter adjusts:

- Voltage Swing
- Pre-emphasis
- Link Rate
- Number of Lanes

### Interview Statement

> "The Link Training Engine ensures reliable communication by negotiating the optimal 
lane count, link rate, voltage swing, and pre-emphasis with the sink."

---

## 3.4 Audio Packet Generator

DisplayPort transports audio using **Secondary Data Packets (SDPs).**

### Responsibilities

- Receives PCM Audio
- Generates Audio Packets
- Inserts Audio Packets into Blanking Intervals

### Interview Statement

> "The Audio Packet Generator encapsulates audio into Secondary Data Packets without affecting the video stream."

---

## 3.5 HDCP Engine

### Purpose

Provides copyright protection.

### Responsibilities

- Authentication
- Encryption
- Key Exchange

### Supported Versions

- HDCP 1.3
- HDCP 2.2
- HDCP 2.3

### Interview Statement

> "If protected content is transmitted, the HDCP engine encrypts the DisplayPort stream 
after successful authentication."

---

## 3.6 MST/SST Manager

DisplayPort supports two operating modes.

### SST (Single Stream Transport)

```text
GPU
 │
 ▼
Monitor
```

### MST (Multi-Stream Transport)

```text
GPU
 │
 ▼
Hub
├── Monitor 1
├── Monitor 2
└── Monitor 3
```

The MST Manager allocates bandwidth among multiple video streams.

### Interview Statement

> "The MST manager schedules multiple independent video streams over a single DisplayPort link."

---

# 4. Video PHY Interface

The DisplayPort controller generates **parallel digital data**, whereas the DisplayPort cable 
carries **high-speed serial data**.

The Video PHY Interface bridges these two layers.

### Transfers

- Parallel Symbols
- Clock Signals
- Reset Signals
- PHY Status

### Interview Statement

> "The Video PHY Interface bridges the protocol layer with the physical transceiver."

---

# 5. DisplayPort PHY

The PHY converts digital data into high-speed differential electrical signals for transmission.

---

## 5.1 PLL (Phase-Locked Loop)

### Purpose

Generates:

- Bit Clock
- Lane Clock
- Reference Clock

### Example

```text
Pixel Clock
      │
      ▼
     PLL
      │
      ▼
8.1 Gbps Lane Clock
```

### Interview Statement

> "The PLL multiplies the reference clock to generate the high-speed clocks required for serial transmission."

---

## 5.2 Serializer

The DisplayPort controller outputs parallel words.

The serializer converts them into a serial bit stream.

### Example

```text
Parallel Data

101011001100...

        │
        ▼

Serializer

        │
        ▼

Serial Bit Stream
```

Each DisplayPort lane has its own serializer.

### Interview Statement

> "The serializer converts parallel DisplayPort symbols into high-speed serial bit streams."

---

## 5.3 Lane Drivers

DisplayPort supports:

- 1 Lane
- 2 Lanes
- 4 Lanes

Each lane is transmitted as a differential pair:

```text
TX+
TX-
```

### Responsibilities

- Voltage Swing Control
- Current Drive
- Differential Signaling

### Interview Statement

> "The lane drivers transmit the serialized data over differential pairs to the DisplayPort connector."

---

## 5.4 Pre-emphasis

High-speed signals degrade over long PCB traces and cables.

Pre-emphasis boosts the high-frequency components of the transmitted signal.

It is dynamically adjusted during Link Training.

### Interview Statement

> "Pre-emphasis compensates for channel losses and improves eye opening at the receiver."

---

## 5.5 Output Buffers

These form the final analog stage before the DisplayPort connector.

### Responsibilities

- Drive Strength
- Impedance Matching
- Signal Integrity

### Interview Statement

> "Output buffers ensure proper electrical characteristics before transmitting the signal to the DisplayPort connector."

---

# 6. DisplayPort Connector

Finally, the following signals are transmitted through the DisplayPort connector:

- Four High-Speed Differential Lanes
- AUX Channel

The DisplayPort monitor performs the reverse operations:

- Receive Serial Data
- Recover Clock
- Deserialize Data
- Decode DisplayPort Symbols
- Display Pixels

---

# 2-Minute Interview Summary

> "The DisplayPort TX subsystem starts by receiving video through AXI4-Stream or a Native Video interface. 
The Video Timing Controller provides synchronization information such as HSYNC and VSYNC. 
The DisplayPort TX controller packetizes the video, manages the Main Link, generates audio packets, performs HDCP encryption 
if enabled, and controls the AUX channel for EDID, DPCD, and Link Training. 
Once Link Training is complete, the controller sends parallel data to the DisplayPort PHY. 
The PHY uses a PLL to generate high-speed clocks, serializes the parallel data, drives it through differential lane 
drivers with programmable voltage swing and pre-emphasis, and transmits it over one, two, or four DisplayPort lanes to the monitor."
