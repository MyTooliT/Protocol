# Commands

## Blocks

| Block  | Short Description | Extended Description                                                                                                                           |
| ------ | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x00` | System            | System commands are used to modify/request the state of each unit (e.g. reset) or an the overall system state (e.g. transmission speed)        |
| `0x04` | Streaming         | Streaming commands are used to transmit data streams, but may be also used for single requests. The super frame is also located in this block. |

## `Streaming`

| Number | Command      |
| ------ | ------------ |
| 1      | Acceleration |

The “Data Sets” bits used in the sections below can have the following values:

| Value | Data Amount                                  |
| ----- | -------------------------------------------- |
| 0     | Stop (stream)                                |
| 1     | 1 data set (x, y, z, x-y-z, x-y, x-z or y-z) |
| 2     | 3 data sets                                  |
| 3     | 6 data sets                                  |
| 4     | 10 data sets                                 |
| 5     | 15 data sets                                 |
| 6     | 20 data sets                                 |
| 7     | 30 data sets (x, y, or z)                    |

### Acceleration

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
