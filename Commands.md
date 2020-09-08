# Commands

## Akronyms

**GD1**: Graceful Degradation Level 1
**GD2**: Graceful Degradation Level 2
**BP**: Byte Position
**MSB**: Most Significant Byte

## Terms

**Event (Message)**: Even messages transport information about signals and events/states

## Blocks

| Block  | Short Description             | Extended Description                                                                                                                              |
| ------ | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x00` | System                        | System commands are used to modify/request the state of each unit (e.g. reset) or an the overall system state (e.g. transmission speed)           |
| `0x04` | Streaming                     | Streaming commands are used to transmit data streams, but may be also used for single requests. The super frame is also located in this block.    |
| `0x08` | Statistical Data and Quantity | This command group is used to store statistical data that can be used for histograms such as operating time and the number of power on/off cycles |
| `0x08` | Statistical Data and Quantity | This command group is used to store statistical data that can be used for histograms such as operating time and the number of power on/off cycles |
| `0x28` | Configuration                 | This command block is used to set configuration data (e.g. you can set the sampling rate of acceleration data here).                              |
| `0x3D` | EEPROM                        | Used for writing and reading EEPROM data directly                                                                                                 |

## Block `System`

| Number | Command       | Access     | Permanently Stored |
| ------ | ------------- | ---------- | ------------------ |
| `0x00` | Verboten      | –          | –                  |
| `0x01` | Reset         | Event      | –                  |
| `0x02` | Get/Set State | Read/Write | –                  |
| `0x05` | Get Status 1  | Read/Write | –                  |

### Command `Verboten`

This command is mainly used for initialization purposes

### Command `Reset`

Reset the specified receiver

### Command `Get/Set State`

#### Notes

- Not fully implemented
- Startup state determines operating state
- Standby state works

#### Payload

| Byte 1                                 |          |                                                                                      |          |                                                                                                                                                                                                                                                                                                              |
| -------------------------------------- | -------- | ------------------------------------------------------------------------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Bit 7                                  | Bit 6    | Bit 5 – 4                                                                            | Bit 3    | Bit 2 – 0                                                                                                                                                                                                                                                                                                    |
| • `0`: Get State <br> • `1`: Set State | Reserved | • `0`: No Change <br> • `1`: Bootloader <br> • `2`: Application <br> • `3`: Reserved | Reserved | • `0`: Failure (No acknowledgement will be sent; Only power on resets this state) <br> • `1`: Error (No active communication) <br> • `2`: Turn Off/Standby <br> • `3`: Graceful degradation level 2 <br> • `4`: Graceful degradation level 1 <br> • `5`: Operating <br> • `6`: Startup <br> • `7`: No change |

#### Acknowledgment Payload

| Byte 1 |
| ------ |
| `0x08` |

#### Error Payload

| Byte 2                                                                                                 |
| ------------------------------------------------------------------------------------------------------ |
| • `1`: Set state not available <br> • `2`: Wrong subscriber (e.g. accessing application as bootloader) |

### Command `Get Status 1`

#### Notes

- Note that the state may not be set instantly. This is the node state status word:

  ```c
  typedef union
  {
   struct
   {
    uint32_t bError :1; /**< Error or healthy */
    uint32_t u3NetworkState :3; /**< Which state has node in the network */
    uint32_t Reserved :28; /**< Reserved */
   };
   uint32_t u32Word;
   uint8_t au8Bytes[4U];
  } NodeStatusWord_t;
  ```

#### Payload

- Setting the value `0` for the status word mask means that we request the status word

| Byte 1      |
| ----------- |
| Status Word |

| Byte 2      |
| ----------- |
| Status Word |

| Byte 3      |
| ----------- |
| Status Word |

| Byte 4      |
| ----------- |
| Status Word |

| Byte 5           |
| ---------------- |
| Status Word Mask |

| Byte 6           |
| ---------------- |
| Status Word Mask |

| Byte 7           |
| ---------------- |
| Status Word Mask |

| Byte 8           |
| ---------------- |
| Status Word Mask |

#### Acknowledgement Payload

- Same structure as payload

## Block `Streaming`

| Number | Command      | Access | Permanently Stored |
| ------ | ------------ | ------ | ------------------ |
| `0x1`  | Acceleration | Event  | –                  |
| `0x20` | Voltage      | Event  | –                  |

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
