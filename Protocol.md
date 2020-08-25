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

## Definitions

- **Header**: Supplemental data placed at the beginning of a block
- **Jitter**: Difference between best-case time and worst-case time
- **Node**: Self-contained unit that interacts with other nodes via the MyToolIt communication protocol
- **Payload**: Transmitted user data
- **Trailer**: Terminating part of a message; May support check functionality

## Acronyms

List of Acronyms

- **CAN**: Controller Area Network
- **CAN-FD**: CAN Flexible Data Rate
- **CSMA/CR**: Carrier Sense Multiple Access/Collision Resolution
- **CSMA/CA**: Carrier Sense Multiple Access/Collision Avoidance
- **LSB**: Least Significant Bit
- **MSB**: Most Significant Bit
- **NOK**: Not OK
- **NA**: Not Available
- **NV**: Not Valid
- **HW**: Hardware
- **SW**: Software
- **TBD**: To Be Defined

## Introduction

CAN was introduced by BOSCH in the 1980s in the automotive industry to exchange short real time messages between Electronic Control Units (ECU). Each ECU may act as a master i.e. send frames and thus each ECU may control the system by inserting error frames, acknowledging, send information or process information. Furthermore, a standard base format and an extended format exists and the MyToolIt communication protocol bases at the extended format. The following figure describes the extended format:

![CAN Frame](Figures/CAN%20Frame.png)

The available literature e.g. Wikipedia, Master Thesis: Experimental Evaluation of Performance in Controller Area Network by Walther Operenyi, etc. supports further information.

A main feature of CAN are prioritized messages i.e. if two or more senders try to send messages simultaneously, the message with the highest priority (lowest identifier) transports instantly and the remaining afterwards (CSMA/CR).

Consequential, each message identifier must be unique (each sender has a set of messages) and subscribers must queue messages to be send by their priority.

The priority-based concept of messages is a key feature of the MyToolIt network protocol. This protocol uses CAN20, Bluetooth and other data link layer protocols to transport messages between end nodes. Thus, MyToolIt transport messages between end nodes over diverse data link protocols. The flow control is managed by the prioritization of messages, the end-to-end-communication and by limiting the overall traffic to 40%/60% of the total bandwidth.

Furthermore, CAN-FD is the extension of CAN2.0 and the main difference is the speed up in the payload field (DLC and CRC are extended too). Thus, the net bandwidth increases by multiplicating the payload field and speeding it up simultaneously (64Bytes takes approximately the same time to transport as 8Bytes if the transmission speed gets increases by 10). Thus transporting the same amount of messages yields into transporting more payload data. Note that MyToolIt actually does not support CAN-FD and CAN-FD will be used as a data link layer that transports abstracted CAN-messages.

## Protocol Specification

Each CAN20 frame consists of an identifier, a payload, a Data Length Code and physical transport bits. The following figure shows the essential parts of an extended CAN20 frame:

| Identifier     | DLC | Payload         |
| -------------- | --- | --------------- |
| Bit 0 – Bit 29 |     | Byte 0 – Byte 7 |

The Identifier describes the message, the Data Length Code the Length of the payload (CAN 2.0: 0 - 8 Byte, CAN-FD 0-64Bytes) and the payload holds information.

### Identifier

| V   | Command | R1  | Sender  | R2  | Receiver |
| --- | ------- | --- | ------- | --- | -------- |
| 0   | 1 – 16  | 17  | 18 – 22 | 23  | 24 – 28  |

The following table describes the identifier field.

| Field    | Purpose                                                                                                                                                         |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| V        | Version Number. Must be 0. Otherwise frame will be discarded.                                                                                                   |
| Command  | Command to be executed or acknowledged.                                                                                                                         |
| R        | Reserved                                                                                                                                                        |
| Sender   | Number of the original sender (Frames may hop). 0 Not allowed.                                                                                                  |
| Receiver | Number of the target receiver (Frames may hop). 0 broadcasts at field d bus(local network) with ack. 0x1F broadcasts at field d bus(local network) without ack. |

### Command Field

The following figure describes the command field.

| Number         | A   | E               |
| -------------- | --- | --------------- |
| Bit 0 – Bit 29 |     | Byte 0 – Byte 7 |

The following table describes the command field.

| Field  | Purpose                                                                                                                                                                                                                                                                                                                                                                                         |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Number | 64 Command Groups and a command group supports up to 256 commands (1 – 16383, 0 not valid). The command are described in the CommandGroups.xls document.                                                                                                                                                                                                                                        |
| A      | Acknowledge field, 1 if it’s a request and 0 if it’s an acknowledge. Note that a single command may trigger multiple acknowledges (See Streaming Bit S).                                                                                                                                                                                                                                        |
| E      | Error Bit that indicates an error. An error code is supported via the payload. 0 if it is an error and 1 if it’s not an error. The error format is 8 Byte long where the first Bytes describes the error number and the following 7 Bytes a sub error description. Furthermore, there are general errors (1-255) that are followed by 0 and specific errors that are followed by variable bits. |

### Abstracted CAN Messages

As mentioned in the Introduction the MyToolIt protocol derives the priorities message concept from CAN20. Therefore, the CAN-Header (Identifier and DLC) gets abstracted by a 4Byte header as follows (The DLC0 Bit is at position 0 and the command resided in the 2 Bytes at the highest addresses):

| Bit   | Name                  | Description                                               |
| ----- | --------------------- | --------------------------------------------------------- |
| 0-3   | Data Length Code(DLC) | Length of message as described by the CAN-FD standard.    |
| 4-8   | Receiver              | End subscriber to be addressed as described in table1.    |
| 9     | Reserved              | Reserved                                                  |
| 10-14 | Sender                | End subscriber that sends message as described in table 1 |
| 15    | Reserved              | Reserved                                                  |
| 16-31 | Command               | Command as described in table 2.                          |

Furthermore, the transport of messages over a data link layer (except CAN20) are fulfilled by putting messages consisting of header and payload in a row up to the length of the data link layer payload. Each node manage the prioritization of messages in each send queue by a prioritised message queue .

### Addressing

A network consists of two or more subscribers and each subscriber owns a unique number (1 – 30; 0= Broadcast with ACK and 31=Broadcast without ACK) called address. The address targets a specific subscriber (or all). Note that the send number is important to acknowledge.

Furthermore, the addressing scheme yields to an end-to-end management of the communication state i.e. the internal states of elements inside the end-to-end subscribers does not influence the logical communication state. Thus, only a single channel must be supported for a MyToolIt information exchange i.e. an incoming message that does not address the subscriber is discarded or forwarded. Thus, the MyToolIt commands transmits via other communication protocols like BlueTooth. Note that the simultaneously transport via CAN20 may not possible due to replicating the send and receiver (and the command) at the data link layer.

Furthermore, the subscribers manage the error handling e.g. re-request something after a time out is done only by the sender or other counter measurements must be fulfilled.

The following figure shows the overall idea of network addressing.

![End To End Communication](Figures/End%20To%20End%20Communication.png)

### DLC

DLC as described the CAN-FD standard. The DLC must transfer over other protocols in the same format. Thus the DLC gets limited by the Data Link Layer i.e. Requesting a command via CAN20 and Bluetooth yields into a limit of 8 Bytes in the DLC.

### Payload

The payload transports user data and/or subpayloads.

### Startup an backup strategy

Actually, not implemented. CAN transmits at 1Mbit gross and Bluetooth transmits the payload at 1Mbit net. Note, that Bluetooth is a CSMA/CD protocol that will yield into jitter without taking any other actions and also reduces the total bandwidth.

### Transmission Speed

The transmission speed depends of the supported data link layer formats.

### Bluetooth

The actual Bluetooth transmissions speed is 1Mbit gross but a message transmission delays due to CSMA/CD. CSMA/CD prevents transmitting if an ongoing transport is in process (radio energy at all counts!) in the corresponding transport frequency interval and each prevent delays the transport time exponentially. Note that simultaneously sending or any radio interference may destroy any radio frame and the actual Bluetooth configuration avoids re-requests at the protocol stack level (Application must do this). Furthermore, Bluetooth supports a net bandwidth of about 700kBit if each frame is 255Bytes long. However, applicatory Bluetooth supports a maximum net bandwidth of about 420kbit/s.

### CAN20

The transmission speed should be aligned to a maximum of 40% of the total bandwidth. However, in any case there must not be any higher utilization then 60% of the overall bandwidth. In the case of fair message distribution with many nodes and many sporadic messages, the limit should be the 40% utilization. In cases with many permanent messages the limit may be set to 60%. The 40% utilization for CAN2.0 with bit stuffing is calculated as follows:

$$
U = \frac{m·79+ \sum_{m=0}^{m} \left( 8·p_m + \lfloor{p_m·\frac{8}{5}} \rfloor \right)}{B}
$$

Where B is the gross bandwidth per second (e.g. 1Mbit/s), m is the overall number of send messages per second, $p_m$ the payload length in Bytes for each message and the overall Utilization U.

The 60% utilization without bit stuff is calculated as follows:

$$
U = \frac{m·67+ \sum_{m=0}^{m} \left( 8·p_m \right)}{B}
$$

The 40% utilization for CAN-FD with bit stuffing is calculated as follows:

$$
U = \frac{m·79}{B_{ID}} + \frac{\sum{m=0}^{m} \left( 8·p_m + \lfloor{p_m·\frac{8}{5}} \rfloor \right)}{B_p}
$$

Where $B_ID$ is the gross identifier bandwidth per second (e.g. 1Mbit/s) and B_p is the gross payload bandwidth per second (e.g. 8Mbit/s).

The 60% utilization for CAN-FD without bit stuffing is calculated as follows:

$$
U = \frac{m·67}{B_{ID}}+ \frac{\sum_{m=0}^{m} \left( 8·p_m \right)}{B_p}
$$

Thus the bandwidth consumption for a streaming message (64Bytes payload each 1ms) calculates as follows at 1Mbit/8Mbit:

$$
U_{Stuff} = \frac{1000·79}{1000000} + \frac{1000·(512 + 102)}{8000000} = 0.079 + 0.07675 = 0.15575 (15.6\%)
$$

and

$$
U = \frac{1000·67}{1000000} + \frac{1000·512)}{8000000} = 0.067 + 0.064 = 0.131 (13.1\%)
$$

Alarm messages (They will be periodically repeated until muted or alarm off event occurs e.g. temperature drops under a certain limit after reaching certain alarm limit) and streaming messages are periodic messages. Furthermore, sporadic messages triggers on demand e.g. setting a program status word requires a request and an acknowledgement. The acknowledgement and the request are sporadic messages. Sporadic messages should have a reserved bandwidth of at least 10% (in an alarm shower case, the alarm messages will be prioritized). Furthermore, an overload case must be handled at the application level e.g. turn off all streaming messages and go to a graceful degradation state or a fail-save state. Note that a time triggered communication eliminates such cases because each message transmission is pre-scheduled.

### Reserved Bits

Reserved Bits must be transmitted as 0. This is required for compatibility.
