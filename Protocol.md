# MyTooliT Communication Protocol

This document defines the MyTooliT network protocol. The MyTooliT network protocol exchanges information over data link layers like Bluetooth or Controller Area Network (CAN).

CAN (2.0) logically splits a message into

- a payload, and
- an identifier.

The identifier contains

- a sender field to define the node of origin of each message,
- a receiver field to define a message receiver, and
- the command number to
  - specify actions,
  - answers to actions,
  - or specify errors.

Each command, defined by its number, will be acknowledge via the same command number. A

- request bit defines request (acknowledgement) commands, and
- an error bit defines errors.

Please note that errors must not requested.

The MyTooliT communication protocol may also exchange information via Bluetooth. For that purpose CAN messages will be stored into the payload of a data link layer like Bluetooth. The identifier field is handled via a 4 byte header and the payload by an additional payload that follows each message header. Note that a message may have a larger payload than 8 bytes (up to 64 Bytes per message as defined by the CAN-FD specification) but the length is limited to 8 bytes, if CAN 2.0 is used in the transport chain.

The MyTooliT protocol can also use other data link layer formats like CAN-FD. For example, you can use the protocol for IP application because it is an end-to-end based network protocol.

## Introduction

CAN was introduced by BOSCH in the 1980s in the automotive industry to exchange short real time messages between Electronic Control Units (ECU). Each ECU may act as a master i.e. send frames and thus each ECU may control the system

- by inserting error frames,
- acknowledging frames,
- sending information or
- processing information.

A standard base format (11 bit identifier) and an extended format (29 bit identifier) exist. The MyTooliT communication protocol is based on the extended format. The following figure describes the extended format:

![CAN Frame](Figures/CAN%20Frame.png)

For more information, please take a look at the [Wikipedia article about CAN](https://en.wikipedia.org/wiki/CAN_bus) or other available literature (e.g. [Experimental Framework for Controller Area Network based on a Multi-Processor-System-on-a- Chip](https://repositum.tuwien.at/retrieve/27430)).

A main feature of CAN are prioritized messages i.e. if two or more senders try to send messages simultaneously, the message with the highest priority (lowest identifier) will be sent instantly and the remaining ones afterwards (CSMA/CR).

This design requires that each message identifier must be unique (each sender has a set of messages) and subscribers must queue messages according to their priority.

The priority-based concept of messages is a key feature of the MyTooliT network protocol. The protocol uses CAN 2.0, Bluetooth and other data link layer protocols to transport messages between end nodes. Thus, MyTooliT transport messages between end nodes over diverse data link protocols. The flow control is managed by the prioritization of messages, the end-to-end-communication and by limiting the overall traffic to 40%/60% of the total bandwidth.

## Protocol Specification

Each CAN 2.0 frame consists of

- an identifier,
- a payload,
- a data length code (DLC), and
- physical transport bits.

The following figure shows the essential parts of an extended CAN 2.0 frame:

| Identifier | DLC    | Payload     |
| ---------- | ------ | ----------- |
| 29 Bits    | 4 Bits | 0 – 7 Bytes |

The

- identifier describes the message,
- the data length code stores the length of the payload (CAN 2.0: 0 – 8 Byte, CAN-FD 0 – 64 bytes), and
- the payload stores message data.

### Identifier

|     | V   | Command | R1  | Sender  | R2  | Receiver |
| --- | --- | ------- | --- | ------- | --- | -------- |
| Bit | 0   | 1 – 16  | 17  | 18 – 22 | 23  | 24 – 28  |

The following table describes the identifier field.

| Field    | Purpose                                                                                                                                                                       |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| V        | Version number <br> • Must be `0` or the frame will be discarded                                                                                                              |
| Command  | Command to be executed or acknowledged                                                                                                                                        |
| R1/R2    | Reserved                                                                                                                                                                      |
| Sender   | Number of the original sender (frames may hop) <br> • `0` Not allowed                                                                                                         |
| Receiver | Number of the target receiver (frames may hop) <br> • `0` broadcasts at field bus (local network) with ACK <br> • `0x1F` broadcasts at field d bus(local network) without ACK |

### Command

|     | Command Number | A   | E   |
| --- | -------------- | --- | --- |
| Bit | 0 – 13         | 14  | 15  |

The command number contains the command block and the block command:

| Block | Block Command |
| ----- | ------------- |
| 0 – 5 | 6 – 13        |

The following table describes the whole command field.

| Field          | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Command Number | • 64 command blocks (6 Bit) <br> • A command block supports up to 256 (8 Bit) block commands <br> • Values: 1 – 16383 (14 bit), `0` is not valid <br> • Commands are described [here](Commands.md)                                                                                                                                                                                                                                   |
| A              | Acknowledge field <br> • `1` for a request <br> • `0` for an acknowledgement <br> Note that a single command may trigger multiple acknowledges (streaming).                                                                                                                                                                                                                                                                          |
| E              | Error Bit <br> • Indicates an error <br> • `1` if it is an error <br> • `0` if it is not an error <br> • An error code is supported via the payload <br> • The error format is b bytes long. The first byte describes the error number and the following 7 bytes are used for an error description. Furthermore, there are general errors (1 – 255) that are followed by `0` and specific errors that are followed by variable bits. |

### Abstracted CAN Messages

As mentioned in the introduction the MyTooliT protocol derives the priorities message concept from CAN 2.0. Therefore, the CAN header (identifier and DLC) are abstracted by a 4 byte header as described in the table below.

Note: The `DLC0` bit is at position 0 and the command resides in the 2 bytes at the highest addresses.

| Bit     | Name                   | Description                                                                |
| ------- | ---------------------- | -------------------------------------------------------------------------- |
| 0 – 3   | Data Length Code (DLC) | Length of message as described by the CAN-FD standard                      |
| 4 – 8   | Receiver               | End subscriber to be addressed as described in the Section “Identifier”    |
| 9       | Reserved               | Reserved                                                                   |
| 10 – 14 | Sender                 | End subscriber that sends message as described in the Section “Identifier” |
| 15      | Reserved               | Reserved                                                                   |
| 16 – 31 | Command                | Command as described in Section “Command Field”                            |

The transport of messages over a data link layer (except CAN 2.0) are fulfilled by putting messages consisting of header and payload in a row up to the length of the data link layer payload. Each node manage the prioritization of messages in each send queue by a prioritized message queue.

### Addressing

A network consists of two or more subscribers and each subscriber use a unique number (1 – 30; 0 = Broadcast with ACK; 31 = Broadcast without ACK) called address. The address targets a specific subscriber (or all subscribers). Note that the send number is important for the acknowledgement.

This addressing scheme yields an end-to-end management of the communication state i.e. the internal states of elements inside the end-to-end subscribers do not influence the logical communication state. Thus, only a single channel must be supported for a MyTooliT information exchange i.e. an incoming message that does not address the subscriber is discarded or forwarded. This means the MyTooliT commands can be used over other communication protocols like Bluetooth. Note that the simultaneous transport via CAN 2.0 may not possible due to the replication of the sender and receiver (and the command) at the data link layer.

In the MyTooliT protocol the subscribers manage the error handling e.g. re-request something after a timeout. If that is not the case, then other counter measurements must be fulfilled.

The following figure shows the overall idea of network addressing.

![End To End Communication](Figures/End%20To%20End%20Communication.png)

### DLC

The MyTooliT protocol uses the DLC as described the CAN-FD standard. The DLC must transfer over other protocols in the same format. Thus the DLC is limited by the data link layer i.e. requesting a command via CAN 2.0 and Bluetooth yields a limit of 8 bytes.

### Payload

The payload transports user data and/or sub-payloads.

### Startup an Backup Strategy

This is currently not implemented. CAN transmits at 1 MBit gross and Bluetooth transmits the payload at 1Mbit net. Note, that Bluetooth is a CSMA/CD protocol that will cause jitter without taking any other actions. Collisions also reduce the total bandwidth.

### Transmission Speed

The transmission speed depends of the supported data link layer formats.

### Bluetooth

The actual Bluetooth transmissions speed is 1 MBit gross. However, a message transmission might be delayed due to CSMA/CD. CSMA/CD prevents transmission, if an ongoing transport is in process in the corresponding transport frequency interval. Each collision delays the transport time exponentially. Note that simultaneously sending or any radio interference may destroy any radio frame and the actual Bluetooth configuration avoids re-requests at the protocol stack level (application must do this).

Bluetooth supports a net bandwidth of about 700 kBit if each frame is 255 bytes long. However, Bluetooth applications supports a maximum net bandwidth of about 420 kbit/s.

### CAN 2.0

The transmission speed should be aligned to a maximum of 40% of the total bandwidth. However, in any case there must not be any higher utilization than 60% of the overall bandwidth. In the case of fair message distribution with many nodes and many sporadic messages, the limit should be a utilization of 40%. In cases with many permanent messages the limit may be set to 60%. The 40% utilization for CAN2.0 with bit stuffing is calculated as follows:

$$
U = \frac{m·79+ \sum_{m=0}^{m} \left( 8·p_m + \lfloor{p_m·\frac{8}{5}} \rfloor \right)}{B}
$$

Here

- B is the gross bandwidth per second (e.g. 1Mbit/s),
- m is the overall number of send messages per second,
- $p_m$ the payload length in bytes for each message and
- U is the overall utilization.

The 60% utilization without bit stuffing is calculated as follows:

$$
U = \frac{m·67+ \sum_{m=0}^{m} \left( 8·p_m \right)}{B}
$$

The 40% utilization for CAN-FD with bit stuffing is calculated as follows:

$$
U = \frac{m·79}{B_{ID}} + \frac{\sum_{m=0}^{m} \left( 8·p_m + \lfloor{p_m·\frac{8}{5}} \rfloor \right)}{B_p}
$$

Here

- $B_{ID}$ is the gross identifier bandwidth per second (e.g. 1Mbit/s), and
- $B_p$ is the gross payload bandwidth per second (e.g. 8 Mbit/s).

The 60% utilization for CAN-FD without bit stuffing is calculated as follows:

$$
U = \frac{m·67}{B_{ID}}+ \frac{\sum_{m=0}^{m} \left( 8·p_m \right)}{B_p}
$$

Thus the bandwidth consumption for a streaming message (64 bytes payload each ms) calculates as follows at 1Mbit/8Mbit:

$$
U_{Stuff} = \frac{1000·79}{1000000} + \frac{1000·(512 + 102)}{8000000} = 0.079 + 0.07675 = 0.15575 (15.6\%)
$$

and

$$
U = \frac{1000·67}{1000000} + \frac{1000·512}{8000000} = 0.067 + 0.064 = 0.131 (13.1\%)
$$

Alarm messages – they will be periodically repeated until muted or alarm off event occurs e.g. temperature drops under a certain limit after reaching certain alarm limit – and streaming messages are periodic messages.

Sporadic messages trigger on demand e.g. setting a program status word requires a request and an acknowledgement. The acknowledgement and the request are sporadic messages.

Sporadic messages should have a reserved bandwidth of at least 10% (in an alarm shower case, the alarm messages will be prioritized). An overload case must be handled at the application level e.g. turn off all streaming messages and go to a graceful degradation state or a fail-save state. Note that time triggered communication eliminates such cases because each message transmission is pre-scheduled.

### Reserved Bits

Reserved Bits must be transmitted as `0`. This is required for compatibility.
