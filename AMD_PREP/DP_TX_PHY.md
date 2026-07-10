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

# Understanding the DisplayPort Main Link Controller (Interview-Friendly)

One common challenge in DisplayPort interviews is remembering technical terms. You don't need to memorize every term in depth. Instead, understand **why each function exists**. Once you know its purpose, the terminology becomes much easier to remember.

---

# Think of the Main Link Controller as a Courier Service

Imagine you want to send a large package to a friend.

Before sending it, you need to:

1. Pack the items into boxes.
2. Label the boxes correctly.
3. Arrange the boxes into different trucks if multiple trucks are available.
4. Send the trucks on the road.

The **Main Link Controller** performs the same job for video data.

---

# 1. Packetizes Video Data

### Technical Term

**Packetization**

### Simple Meaning

It divides the continuous stream of pixels into smaller packets that DisplayPort can transmit.

### Analogy

```text
Large Book
    │
    ▼
Chapter 1
Chapter 2
Chapter 3
```

Instead of sending one huge block of data, it sends many smaller packets.

### Remember

> **Packetization = Breaking video into small packets.**

---

# 2. Scrambles Data

The word **scrambler** sounds complicated, but its purpose is simple.

Imagine sending this over a wire:

```text
11111111111111111111111
```

or

```text
00000000000000000000000
```

Long runs of identical bits make it difficult for the receiver to recover the clock accurately and can increase electromagnetic interference (EMI).

A **scrambler** changes the transmitted bit pattern so it looks more random.

Example:

```text
Before
1111111111111111

After Scrambling
101001110100101...
```

The receiver uses the same scrambling algorithm to recover the original data.

### Remember

> **Scrambler = Makes the bit stream look random for more reliable transmission.**

---

# 3. Line Coding (8b/10b)

DisplayPort does not transmit raw bytes directly.

Instead:

```text
8 Bits
   │
   ▼
10 Bits
```

The extra bits help with:

- Clock Recovery
- Error Detection
- Maintaining Signal Quality

### Analogy

Think of it as adding a **barcode** to every package so the receiver can identify it correctly.

### Remember

> **8b/10b = Adds extra information to make transmission reliable.**

---

# 4. Lane Distribution

DisplayPort supports:

- 1 Lane
- 2 Lanes
- 4 Lanes

### Analogy

#### One Lane

```text
→ → → → →
```

#### Four Lanes

```text
→ → → →
→ → → →
```

Instead of sending all data through one lane, DisplayPort splits the data across multiple lanes to increase bandwidth.

### Remember

> **Lane Distribution = Split the data across multiple lanes to send it faster.**

---

# 5. Controls the Main Link

The **Main Link** is the high-speed path that carries the actual video and audio.

The Main Link Controller is responsible for:

- Starting transmission
- Stopping transmission
- Sending idle patterns when required
- Maintaining synchronization
- Ensuring data is transmitted correctly

### Remember

> **Main Link Controller = Manages the high-speed DisplayPort data path.**

---

# Easy Memory Trick: **P-S-L-L-C**

Remember these five letters:

| Letter | Function |
|---------|----------|
| **P** | Packetize |
| **S** | Scramble |
| **L** | Line Coding |
| **L** | Lane Distribution |
| **C** | Control the Main Link |

### Memory Phrase

> **"Please Send Large Letters Carefully."**

Which maps to:

- **Please** → Packetize
- **Send** → Scramble
- **Large** → Line Coding
- **Letters** → Lane Distribution
- **Carefully** → Control the Main Link

---

# One-Line Interview Answer

> "The Main Link Controller prepares video for transmission. It packetizes the video, applies the 
required encoding and scrambling, distributes the data across one, two, or four lanes, and controls 
the high-speed DisplayPort Main Link."

This answer is technically accurate and usually sufficient in an interview.

---

# Don't Memorize—Understand the Flow

Whenever you encounter a high-speed protocol such as **DisplayPort TX**, **HDMI TX**, or **PCIe**, ask yourself these four questions:

| Question | Function |
|----------|----------|
| **How is the data prepared?** | Packetization |
| **How is the data made reliable?** | Scrambling and Line Coding |
| **How is the data sent faster?** | Lane Distribution |
| **Who manages the transmission?** | Main Link Controller |

If you remember this sequence, you'll be able to explain the Main Link Controller naturally instead of recalling isolated technical terms.
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

This is another topic that sounds complicated because of the terminology. The easiest way to remember it is to think of AUX as the "communication channel" between the DisplayPort source (FPGA/GPU) and the monitor.

One-line definition

AUX Channel = A low-speed bidirectional communication channel used to configure and manage the DisplayPort link.

Notice the keywords:

Low-speed (not for video)
Bidirectional (both source and monitor can send data)
Management (configuration, not video)
Think of this analogy

Imagine you're calling a hotel.

The Main Link is the truck delivering your luggage (video/audio).
The AUX Channel is the phone call where you ask:
"Do you have my reservation?" (EDID)
"Which room should I use?" (DPCD)
"Can I enter?" (HDCP)
"Can we communicate?" (I²C-over-AUX)

The phone call doesn't carry your luggage—it just coordinates everything.

AUX Controller Functions
1. Read DPCD Registers
What is DPCD?

DPCD = DisplayPort Configuration Data

These are registers inside the monitor (sink) that describe its DisplayPort capabilities.

The source reads them to learn:

Maximum link rate
Number of lanes supported
DisplayPort version
MST support
FEC support
Training status
Easy memory

Think of DPCD as the monitor's configuration file.

Interview answer:

"The AUX controller reads DPCD registers to discover the monitor's DisplayPort capabilities before link training."

2. Read EDID
What is EDID?

EDID = Extended Display Identification Data

EDID tells the source what the monitor can display.

Examples:

1920×1080
3840×2160
60 Hz
120 Hz
HDR support
Audio support

Without EDID, the source doesn't know which video mode to send.

Easy memory

Think of EDID as the monitor's resume or profile.

It says:

"Hi, I'm a 4K monitor. I support HDR and 120 Hz."

Interview answer:

"The AUX controller reads EDID so the source can configure the appropriate video resolution and refresh rate."

3. I²C-over-AUX

Normally, EDID is accessed over an I²C bus. In DisplayPort, the AUX channel can carry I²C transactions.

This is called I²C-over-AUX.

You don't need to explain the protocol details.

Just remember:

"AUX can transport I²C commands to communicate with the monitor."

4. HDCP Communication

Before sending protected content:

Source and monitor authenticate each other.
They exchange encryption information.

This communication happens over the AUX channel.

Think of it like:

"Are you an authorized display?"

If yes, encrypted video transmission begins.

Why is AUX Bidirectional?

The Main Link only sends data:

GPU  ------------->  Monitor

The AUX channel allows both ends to communicate:

GPU  <----------->  Monitor

For example:

GPU asks:

"What resolutions do you support?"

Monitor replies:

"I support 4K at 60 Hz."

This two-way exchange is why the AUX channel is bidirectional.

Easy Memory Trick

Remember the acronym DEIH:

D → DPCD
E → EDID
I → I²C-over-AUX
H → HDCP

You can think:

"Don't Ever Ignore HDCP."

30-Second Interview Answer

"The AUX Controller manages the low-speed bidirectional communication between the source and the DisplayPort sink. 
It reads the DPCD registers to determine the sink's capabilities, reads the EDID to identify supported display modes, 
carries I²C-over-AUX transactions, and handles HDCP authentication and other management operations. 
Unlike the Main Link, which carries high-speed video and audio, the AUX channel is used for control and configuration."

A simple way to separate the two in your mind
Main Link	AUX Channel
Carries video and audio	Carries control and configuration data
High speed	Low speed
Mainly one-way (source → monitor)	Bidirectional (source ↔ monitor)
Uses DP lanes	Uses dedicated AUX pins

If an interviewer asks, "What is the AUX channel used for?", a concise answer is:

"The AUX channel is DisplayPort's management channel. It is used for DPCD access, EDID reading, I²C-over-AUX communication, 
and HDCP authentication before and during video transmission."

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

# DisplayPort Link Training (Interview-Friendly Notes)

One of the most frequently asked DisplayPort interview topics is **Link Training**. Instead of memorizing the sequence, focus on **why Link Training is required**. Once you understand its purpose, the entire process becomes easy to explain.

---

# Why Do We Need Link Training?

Imagine you and your friend are talking over a phone call.

Before starting the conversation, you ask:

> **"Can you hear me clearly?"**

Your friend might respond:

- **"Too loud."** → Lower your voice.
- **"Too soft."** → Speak louder.
- **"Connection is poor."** → Slow down.

DisplayPort follows the same idea before sending video.

Since **cable quality, cable length, PCB routing, and connectors vary**, the transmitter cannot assume that the receiver will correctly receive high-speed data.

So, before transmitting video, the **DisplayPort Source** and **DisplayPort Sink** first **train the communication link**.

---

# One-Line Definition

> **Link Training is a negotiation between the DisplayPort Source and Sink to establish a reliable high-speed communication link before video transmission begins.**

---

# DisplayPort Link Training Flow

```text
Monitor Connected
        │
        ▼
1. HPD Detected
        │
        ▼
2. Read DPCD
        │
        ▼
3. Clock Recovery
        │
        ▼
4. Channel Equalization
        │
        ▼
5. Training Success
        │
        ▼
6. Video Transmission Starts
```

---

# Step 1: HPD (Hot Plug Detect)

When the monitor is connected, it asserts the **HPD (Hot Plug Detect)** signal.

```text
Monitor Connected
        │
        ▼
HPD = HIGH
```

The transmitter now knows:

> **"A DisplayPort monitor is connected."**

---

# Step 2: Read DPCD

The transmitter communicates with the monitor over the **AUX Channel** and reads the **DisplayPort Configuration Data (DPCD)** registers.

It asks questions such as:

- How many lanes do you support?
- What is your maximum link rate?
- Which DisplayPort version do you support?

### Example

The monitor reports:

- 4 Lanes
- HBR3 (8.1 Gbps)
- DisplayPort 1.4

Now the transmitter knows the capabilities of the receiver.

---

# Step 3: Clock Recovery

This is one of the most common interview questions.

## What is Clock Recovery?

The receiver must recover the transmitter's clock from the incoming serial data stream.

If it cannot synchronize to the transmitted clock, the received data cannot be decoded correctly.

### Analogy

Imagine two people clapping.

**Person A**

```text
👏 👏 👏 👏
```

**Person B** tries to clap at exactly the same rhythm.

If both clap together:

✔ Clock Recovery Success

If they are out of sync:

✘ Clock Recovery Failed

To help the receiver synchronize, the transmitter adjusts:

- Voltage Swing
- Link Rate

until the receiver locks onto the incoming data stream.

### Remember

> **Clock Recovery = Can the receiver synchronize to the transmitted data?**

---

# Step 4: Channel Equalization

Once the receiver has recovered the clock, it checks the quality of the received data.

High-speed cables introduce:

- Attenuation
- Signal Loss
- Distortion
- Inter-symbol Interference (ISI)

The transmitter adjusts:

- Pre-emphasis
- Voltage Swing

to compensate for these channel losses.

The receiver checks:

- Is the eye diagram sufficiently open?
- Are symbols being decoded correctly?

If yes:

✔ Channel Equalization Success

---

# Step 5: Training Success

When both:

- Clock Recovery
- Channel Equalization

are successful, the communication link is considered stable.

```text
Training Success
        │
        ▼
Enable Video Transmission
```

The DisplayPort source now begins transmitting video.

---

# Parameters Adjusted During Link Training

---

## 1. Voltage Swing

Voltage Swing represents the electrical signal strength.

### Example

- Long cable → Increase Voltage Swing
- Short cable → Lower Voltage Swing

### Analogy

Speaking louder when someone is farther away.

### Remember

> **Voltage Swing = Signal Strength**

---

## 2. Pre-emphasis

High-frequency components of the signal weaken over long cables.

The transmitter boosts these components before transmission.

### Analogy

Imagine speaking in a noisy room.

You naturally emphasize difficult words so the listener hears them clearly.

### Remember

> **Pre-emphasis = Boost high-frequency signal components before transmission.**

---

## 3. Link Rate

The Link Rate determines the transmission speed.

### Common DisplayPort Link Rates

| Link Rate | Speed |
|-----------|-------|
| RBR | 1.62 Gbps |
| HBR | 2.7 Gbps |
| HBR2 | 5.4 Gbps |
| HBR3 | 8.1 Gbps |

If the communication channel cannot reliably support the highest speed, the transmitter lowers the Link Rate.

### Analogy

Instead of driving at **120 km/h** on a rough road, slow down to **80 km/h**.

### Remember

> **Link Rate = Transmission Speed**

---

## 4. Number of Lanes

DisplayPort supports:

- 1 Lane
- 2 Lanes
- 4 Lanes

Examples:

- Monitor supports 2 lanes → Use 2 lanes.
- Monitor supports 4 lanes → Use all 4 lanes for maximum bandwidth.

### Remember

> **Lane Count = Number of parallel transmission paths**

---

# Easy Memory Trick

## Link Training Sequence

Remember:

**H-D-C-E-V**

| Letter | Meaning |
|---------|----------|
| **H** | HPD |
| **D** | DPCD Read |
| **C** | Clock Recovery |
| **E** | Channel Equalization |
| **V** | Video Starts |

### Memory Phrase

> **"Happy Drivers Can Enjoy Vacation."**

---

## Adjustable Parameters

Remember:

**V-P-L-L**

| Letter | Meaning |
|---------|----------|
| **V** | Voltage Swing |
| **P** | Pre-emphasis |
| **L** | Link Rate |
| **L** | Lane Count |

---

# 45-Second Interview Answer

> "When a DisplayPort monitor is connected, it asserts the HPD signal. The source then reads the monitor's DPCD registers over the AUX channel to determine its capabilities, such as the supported lane count and maximum link rate. Next, the Link Training Engine performs Clock Recovery so the receiver can synchronize with the transmitted data. It then performs Channel Equalization to ensure the received signal quality is sufficient. During these stages, the transmitter adjusts parameters such as voltage swing, pre-emphasis, link rate, and lane count based on feedback from the sink. Once Link Training succeeds, normal video transmission begins."

---

# One Sentence to Remember

> **"Link Training is a negotiation between the DisplayPort Source and Sink to find the fastest and most reliable settings before transmitting video."**

If you remember this single idea, you'll have a strong foundation for explaining the entire Link Training process and answering most follow-up interview questions.

---

## 3.4 Audio Packet Generator

DisplayPort transports audio using **Secondary Data Packets (SDPs).**

### Responsibilities

- Receives PCM Audio
- Generates Audio Packets
- Inserts Audio Packets into Blanking Intervals

This block is actually one of the easier ones to understand once you know how DisplayPort sends video and audio together.

First, what is PCM Audio?

PCM = Pulse Code Modulation

It is simply raw digital audio samples.

Examples:

Music
Voice
Movie audio

Think of PCM as:

"Digital audio data coming from the processor or audio codec."

You don't need to explain the PCM encoding details in an interview.

What does the Audio Packet Generator do?

The DisplayPort source receives:

Video pixels
Audio samples

The video and audio cannot just be mixed randomly.

So the Audio Packet Generator:

Takes the PCM audio samples.
Packs them into DisplayPort Secondary Data Packets (SDPs).
Inserts those packets into the DisplayPort stream at the appropriate times.
Why are they called Secondary Data Packets (SDPs)?

DisplayPort mainly exists to transmit video.

Anything other than the main video stream is sent as Secondary Data Packets.

Examples include:

Audio
Time stamps
Info packets
Other auxiliary information

So:

Main Stream → Video
Secondary Data Packets → Audio and metadata

Think of it like a train:

DisplayPort Train

==========================
| Video | Video | Video |
==========================

Between video packets

[Audio Packet]

==========================
| Video | Video | Video |
==========================
What are Blanking Intervals?

This is another common interview term.

When one video line finishes, there is a very short idle period before the next line starts.

Similarly, between video frames there is another idle period.

These idle periods are called blanking intervals.

During these intervals:

No active pixels are being transmitted.
DisplayPort uses the available bandwidth to send Secondary Data Packets, such as audio.
Simple analogy

Imagine a teacher writing on a whiteboard.

Writing = Video
Walking to the next line = Blanking interval

While walking, the teacher can answer a student's question.

Similarly:

Active Video → Pixel data
Blanking Interval → Audio packets and other metadata
Responsibilities
1. Receives PCM Audio

Input:

Audio Codec

↓

PCM Samples

The Audio Packet Generator receives the digital audio samples.

2. Generates Audio Packets

Instead of sending raw samples directly, it organizes them into DisplayPort-compliant Audio Packets (a type of Secondary Data Packet).

Think of it like putting loose letters into an envelope before mailing them.

3. Inserts Audio Packets into Blanking Intervals

The video stream has priority.

Audio packets are inserted into the blanking intervals so they do not interfere with active video transmission.

Video Line

██████████████████████

Blanking

[Audio Packet]

Next Video Line

██████████████████████
Easy Memory Trick

Remember P-G-I:

P → PCM Audio
G → Generate Audio Packets
I → Insert into Blanking Intervals

You can think:

"Please Generate Immediately."

30-Second Interview Answer

"The Audio Packet Generator receives PCM audio samples from the audio source, encapsulates them into DisplayPort Secondary Data Packets, and inserts these packets into the blanking intervals of the DisplayPort stream. This allows audio and video to be transmitted together while ensuring the active video stream is not disturbed."

One-line Summary

Audio Packet Generator = Takes PCM audio, converts it into DisplayPort Secondary Data Packets, and inserts them into blanking intervals so audio is transmitted alongside video without affecting the video stream.

Interview tip

If the interviewer asks, "Why are audio packets sent during blanking intervals?", you can answer:

"Because the active video period is reserved for pixel data. Blanking intervals provide available bandwidth to transmit audio and other secondary data without interrupting the video stream."

That answer shows you understand the concept rather than just memorizing the terminology.

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

HDCP is another block where understanding the purpose and flow is more important than memorizing the terms.

First, what is HDCP?

HDCP = High-bandwidth Digital Content Protection

It is a security mechanism used to protect copyrighted digital content.

Examples:

Movies
Streaming video
Premium content

The goal is:

Only an authorized display should be able to receive and display protected content.

Simple Analogy

Think of a cinema.

Before showing a movie:

The cinema checks your ticket.
If the ticket is valid, they allow you inside.
The movie projector sends the movie securely.

HDCP works similarly:

Source (GPU/FPGA)
        |
        |  "Are you an authorized display?"
        |
        ▼
Display (Monitor)
        |
        |  "Yes, I support HDCP"
        |
        ▼
Encrypted Video Transfer
HDCP Engine Responsibilities

Remember:

A-K-E

A → Authentication
K → Key Exchange
E → Encryption

Think:

"Authenticate → Exchange Keys → Encrypt"

1. Authentication
Question:

"Is this display a trusted device?"

The source communicates with the sink (monitor) through the AUX channel.

The source verifies:

Is the receiver HDCP capable?
Is it a valid device?
Can it decrypt protected content?

Example:

Source:

Are you HDCP capable?

        ↓

Monitor:

Yes, I support HDCP 2.2

If authentication fails:

Protected content is not transmitted.
2. Key Exchange

After authentication, both devices exchange security information.

The purpose is to create a shared secret used for encryption.

Think of it like:

Two people creating a secret password that nobody else knows.

Source Key

+

Sink Key

↓

Shared Secret
3. Encryption

After authentication and key exchange:

The video stream is encrypted before transmission.

Flow:

Video Data

     ↓

HDCP Encryption

     ↓

Encrypted DisplayPort Stream

     ↓

Monitor Decryption

     ↓

Display

The monitor decrypts the data using the negotiated keys.

HDCP Versions

You don't need to memorize deep differences unless asked.

Just remember:

Version	Common Usage
HDCP 1.3	Older DisplayPort/HDMI systems
HDCP 2.2	4K protected content
HDCP 2.3	Latest improved security

For an AMD DisplayPort TX interview:

Mention:

"Modern systems commonly use HDCP 2.x for high-resolution protected content."

Where does HDCP communicate?

Important interview point:

HDCP communication happens through the AUX channel.

Remember:

AUX Channel
     |
     |
     ▼
HDCP Authentication

Main Link
     |
     |
     ▼
Encrypted Video Transfer

So:

AUX → Security handshake
Main Link → Encrypted video data
Easy Memory Trick

Remember:

"Check → Create → Code"
Check

Authentication

"Are you trusted?"

Create

Key Exchange

"Create a shared secret."

Code

Encryption

"Encrypt the video."

45-Second Interview Answer

"The HDCP Engine provides content protection for DisplayPort transmission. Before sending protected video, 
it performs authentication with the sink through the AUX channel. After successful authentication, both source
 and sink exchange keys to generate encryption parameters. The video stream is then encrypted before transmission
 over the Main Link. The receiver decrypts the stream and displays the content only if HDCP authentication is successful."

One-line memory

HDCP Engine = Verify the receiver, exchange security keys, then encrypt the video stream.

For the DP TX block diagram, remember the position:

Video
  |
  ▼
DP TX Controller
  |
  ├── Main Link → Video transmission
  ├── AUX       → HDCP communication
  └── HDCP      → Protects content

This explanation is enough for an AMD/Xilinx DisplayPort TX subsystem interview unless the interviewer goes into HDCP protocol internals.
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
