# Commands

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

| Number | Command          | Access     | Permanently Stored |
| ------ | ---------------- | ---------- | ------------------ |
| `0x00` | Verboten         | –          | –                  |
| `0x01` | Reset            | Event      | –                  |
| `0x02` | Get/Set State    | Read/Write | –                  |
| `0x05` | Get Node Status  | Read/Write | –                  |
| `0x06` | Get Error Status | Read/Write | –                  |

### Command `Verboten`

This command is mainly used for initialization purposes

### Command `Reset`

Reset the specified receiver

### Command `Get/Set State`

- Not fully implemented
- Startup state determines operating state
- Standby state works

#### Values

- Get/Set state values:

  | Value | Meaning   |
  | ----- | --------- |
  | `0`   | Get State |
  | `1`   | Set State |

- Location values

  | Value | Meaning     |
  | ----- | ----------- |
  | `0`   | No Change   |
  | `1`   | Bootloader  |
  | `2`   | Application |
  | `3`   | Reserved    |

- State values

  | Value | Meaning                                                                         |
  | ----- | ------------------------------------------------------------------------------- |
  | `0`   | Failure (No acknowledgement will be sent; <br> Only power on resets this state) |
  | `1`   | Error (No active communication)                                                 |
  | `2`   | Turn Off/Standby                                                                |
  | `3`   | Graceful degradation level 2                                                    |
  | `4`   | Graceful degradation level 1                                                    |
  | `5`   | Operating                                                                       |
  | `6`   | Startup                                                                         |
  | `7`   | No change                                                                       |

- Error reason values

  | Value | Meaning                                                     |
  | ----- | ----------------------------------------------------------- |
  | `1`   | Set state not available                                     |
  | `2`   | Wrong subscriber (e.g. accessing application as bootloader) |

#### Payload

| Byte 1        |          |           |          |           |
| ------------- | -------- | --------- | -------- | --------- |
| Bit 7         | Bit 6    | Bit 5 – 4 | Bit 3    | Bit 2 – 0 |
| Get/Set State | Reserved | Location  | Reserved | State     |

#### Acknowledgment Payload

| Byte 1 |
| ------ |
| `0x08` |

#### Error Payload

| Byte 2       |
| ------------ |
| Error Reason |

### Command `Get Node Status`

- Note that the state may not be set instantly.
- The node status word is defined differently for STH and STU
- STH node status word:

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

- STU node staus word:

  ```c
    struct
    {
      uint32_t bError :1; /**< Indicates an overall Error */
      uint32_t u3NetworkState :3; /**< Which state has node in the network */
      uint32_t bEnabledRadio :1; /**< Radio port enabled(1) or disabled(0) */
      uint32_t bEnabledCan :1; /**< CAN port enabled(1) or disabled(0) */
      uint32_t bRadioActive :1; /**< Radio Active(Connected to Bluetooth) or not */
      uint32_t Reserved :25; /**< Reserved */
    };
    uint32_t u32Word;
    uint8_t au8Bytes[4U];
  } NodeStatusWord_t;
  ```

- Possible error bit values:

  | Value | Meaning  |
  | ----- | -------- |
  | `0`   | No Error |
  | `1`   | Error    |

- Possible network state values:

  | Value | Meaning                |
  | ----- | ---------------------- |
  | `0`   | Failure                |
  | `1`   | Error                  |
  | `2`   | Standby                |
  | `3`   | Graceful Degradation 2 |
  | `4`   | Graceful Degradation 1 |
  | `5`   | Operating              |
  | `6`   | Startup                |
  | `7`   | No Change              |

- Radio port bit values:

  | Value | Meaning             |
  | ----- | ------------------- |
  | `0`   | Radio Port Disabled |
  | `1`   | Radio Port Enabled  |

- CAN port bit values:

  | Value | Meaning           |
  | ----- | ----------------- |
  | `0`   | CAN Port Disabled |
  | `1`   | CAN Port Enabled  |

- Radio activity bit values

  | Value | Meaning                     |
  | ----- | --------------------------- |
  | `0`   | Disconnected from Bluetooth |
  | `1`   | Connected to Bluetooth      |

#### Payload

- Setting the value `0` for the node status word mask means that we request the status word
- Currently the only supported payload should be 8 null (`0x00`) bytes

##### STH

| Bit 7 – 4 | Bit 3 – 1     | Bit 0     |
| --------- | ------------- | --------- |
| Reserved  | Network State | Error Bit |

##### STU

| Bit 7    | Bit 6           | Bit 5            | Bit 4              | Bit 3 – 1     | Bit 0     |
| -------- | --------------- | ---------------- | ------------------ | ------------- | --------- |
| Reserved | Radio Connected | CAN Port Enabled | Radio Port Enabled | Network State | Error Bit |

##### STH & STU

| Byte 2   |
| -------- |
| Reserved |

| Byte 3   |
| -------- |
| Reserved |

| Byte 4   |
| -------- |
| Reserved |

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

#### Error Payload

- The (possibly incorrect) length of the status word (5 instead of 4 bytes) was taken from the original documentation.

| Byte 1                |
| --------------------- |
| Mask Used Not Allowed |

| Byte 4      |
| ----------- |
| Status Word |

| Byte 5      |
| ----------- |
| Status Word |

| Byte 6      |
| ----------- |
| Status Word |

| Byte 7      |
| ----------- |
| Status Word |

| Byte 8      |
| ----------- |
| Status Word |

### Command `Get Error Status`

- STH definition:

  ```c
  typedef union
  {
   struct
   {
    uint32_t bTxBluetoothFail :1; /**< Tx Fail Counter for Bluetooth (non single set) */
    uint32_t bAdcOverRun :1;  /**< Determines ADC over run (not able to shuffle data in time) */
    uint32_t Reserved :30;
   };
   uint32_t u32Word;
   uint8_t au8Bytes[4U];
  } ErrorStatusWord_t;
  ```

- STU definition:

  ```c
  typedef union
  {
  	struct
  	{
  		uint32_t bTxCanFail :1; /**< Tx Fail Counter for CAN (non single set) */
  		uint32_t Reserved :31; /**< DAC was not fed */
  	};
  	uint32_t u32Word;
  	uint8_t au8Bytes[4U];
  } ErrorStatusWord_t;
  ```

- Transmission failure bit (Bluetooth for STH, CAN for STU):

  | Value | Meaning                 |
  | ----- | ----------------------- |
  | `0`   | No Transmission Failure |
  | `1`   | Transmission Failure    |

- ADC overrun:

  | Value | Meaning              |
  | ----- | -------------------- |
  | `0`   | No ADC Overrun Error |
  | `1`   | ADC Overrun Error    |

#### Payload

- Setting the value `0` for the status word mask means that we request the error status word
- Currently the only supported payload should be 8 null (`0x00`) bytes

##### STH

| Bit 7 – 2 | Bit 1       | Bit 0                          |
| --------- | ----------- | ------------------------------ |
| Reserved  | ADC Overrun | Bluetooth Transmission Failure |

##### STU

| Bit 7 – 2 | Bit 0                    |
| --------- | ------------------------ |
| Reserved  | CAN Transmission Failure |

##### STH & STU

| Byte 2   |
| -------- |
| Reserved |

| Byte 3   |
| -------- |
| Reserved |

| Byte 4   |
| -------- |
| Reserved |

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

#### Error Payload

- Same structure as error payload for node status command

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
