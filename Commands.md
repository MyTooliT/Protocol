# Commands

## Akronyms

**BP**: Byte Position
**MSB**: Most Significant Byte

## Blocks

| Block  | Short Description | Extended Description                                                                                                                           |
| ------ | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x00` | System            | System commands are used to modify/request the state of each unit (e.g. reset) or an the overall system state (e.g. transmission speed)        |
| `0x04` | Streaming         | Streaming commands are used to transmit data streams, but may be also used for single requests. The super frame is also located in this block. |

## Block `Streaming`

| Number | Command      |
| ------ | ------------ |
| `0x1`  | Acceleration |
| `0x20` | Voltage      |

The “Data Sets” bits used in the sections below can have the following values:

| Value | Data Amount   | Possible Data                                                                                                                                              |
| ----- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0     | Stop (stream) | • value 1 <br> • value 2 <br> • value 3 <br> • value 1 / value 2 / value 3 <br> • value 1 / value 2 <br> • value 1 / value 3 <br> • value 2 / value 3 <br> |
| 1     | 1 data set    |                                                                                                                                                            |
| 2     | 3 data sets   |                                                                                                                                                            |
| 3     | 6 data sets   |                                                                                                                                                            |
| 4     | 10 data sets  |                                                                                                                                                            |
| 5     | 15 data sets  |                                                                                                                                                            |
| 6     | 20 data sets  |                                                                                                                                                            |
| 7     | 30 data sets  | • value 1 <br> • value 2 <br> • value 3                                                                                                                    |

The chronological order starts with the oldest set (BP) and continues with newer values (BP + t), where t is the time point.

### Command `Acceleration`

- Access: Event Message
- Permanently Stored: –

#### Notes

- Requesting while streaming is possible
- Only single stream allowed
- Requesting stream in different format stops last stream
- Tuple format (depending on active axis, see payload):
  - x/y/z
  - x/y
  - x/z
  - y/z

#### Payload

| Byte 1                                   |                                             |               |               |               |           |
| ---------------------------------------- | ------------------------------------------- | ------------- | ------------- | ------------- | --------- |
| Bit 7                                    | Bit 6                                       | Bit 5         | Bit 4         | Bit 3         | Bit 2 – 0 |
| • `0`: Stream <br> • `1`: Single Request | • `0`: 2 Bytes/Axis <br> • `1`: 3 Byte/Axis | X-Axis Active | Y-Axis Active | Z-Axis Active | Data Sets |

#### Acknowledgment Payload

| Byte 1          |
| --------------- |
| Same as Payload |

| Byte 2           |
| ---------------- |
| Sequence Counter |

##### 2 Byte Format

| Byte 3   |
| -------- |
| MSB (BP) |

| Byte 4       |
| ------------ |
| LSB (BP + 1) |

##### 3 Byte Format

| Byte 3   |
| -------- |
| MSB (BP) |

| Byte 4   |
| -------- |
| (BP + 1) |

| Byte 5       |
| ------------ |
| LSB (BP + 2) |

### Command `Voltage`

- Access: Event Message
- Permanently Stored: –

#### Notes

- Highest voltage sampling rate determines bit stream rate
- Requesting while streaming is possible

#### Payload

| Byte 1                                   |                                              |                  |                  |                  |           |
| ---------------------------------------- | -------------------------------------------- | ---------------- | ---------------- | ---------------- | --------- |
| Bit 7                                    | Bit 6                                        | Bit 5            | Bit 4            | Bit 3            | Bit 2 – 0 |
| • `0`: Stream <br> • `1`: Single Request | • `0`: 2 Bytes Data <br> • `1`: 3 Bytes data | Voltage 1 Active | Voltage 2 Active | Voltage 3 Active | Data Sets |

#### Acknowledgment Payload

The command uses the same format as the “Acknowledgment Payload” of the `Acceleration` command.
