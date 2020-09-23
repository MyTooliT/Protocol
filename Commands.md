[toc]

# Commands

## Blocks

| Block  | Short Description             | Extended Description                                                                                                                              |
| ------ | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x00` | [System](#block:system)       | System commands are used to modify/request the state of each unit (e.g. reset) or an the overall system state (e.g. transmission speed)           |
| `0x04` | [Streaming](#block:streaming) | Streaming commands are used to transmit data streams, but may be also used for single requests. The super frame is also located in this block.    |
| `0x08` | Statistical Data and Quantity | This command group is used to store statistical data that can be used for histograms such as operating time and the number of power on/off cycles |
| `0x28` | Configuration                 | This command block is used to set configuration data (e.g. you can set the sampling rate of acceleration data here).                              |
| `0x3D` | EEPROM                        | Used for writing and reading EEPROM data directly                                                                                                 |

<a name="block:system"></a>

## Block `System`

| Number | Block Command                                 | Access     | Permanently Stored |
| ------ | --------------------------------------------- | ---------- | ------------------ |
| `0x00` | [Verboten](#command:verboten)                 | –          | –                  |
| `0x01` | [Reset](#command:reset)                       | Event      | –                  |
| `0x02` | [Get/Set State](#command:get-set-state)       | Read/Write | –                  |
| `0x05` | [Get Node Status](#command:get-node-status)   | Read/Write | –                  |
| `0x06` | [Get Error Status](#command:get-error-status) | Read/Write | –                  |

<a name="command:verboten"></a>

### Command `Verboten`

This command is mainly used for initialization purposes

<a name="command:reset"></a>

### Command `Reset`

Reset the specified receiver. This command has no payload.

<a name="command:get-set-state"></a>

### Command `Get/Set State`

- Not fully implemented
- Startup state determines operating state
- Standby state works

#### Values

- <a name="value:get-set-state">`Get/Set State`</a>:

  | Value | Meaning   |
  | ----- | --------- |
  | `0`   | Get State |
  | `1`   | Set State |

- <a name="value:location">`Location`</a>:

  | Value | Meaning     |
  | ----- | ----------- |
  | `0`   | No Change   |
  | `1`   | Bootloader  |
  | `2`   | Application |
  | `3`   | Reserved    |

- <a name="value:state">`State`</a>:

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

- <a name="value:error-reason">`Error Reason`</a>:

  | Value | Meaning                                                     |
  | ----- | ----------------------------------------------------------- |
  | `1`   | Set state not available                                     |
  | `2`   | Wrong subscriber (e.g. accessing application as bootloader) |

#### Payload

| Byte 1                                  |          |                               |          |                         |
| --------------------------------------- | -------- | ----------------------------- | -------- | ----------------------- |
| Bit 7                                   | Bit 6    | Bit 5 – 4                     | Bit 3    | Bit 2 – 0               |
| [`Get/Set State`](#value:get-set-state) | Reserved | [`Location`](#value:location) | Reserved | [`State`](#value:state) |

#### Acknowledgment Payload

| Byte 1 |
| ------ |
| `0x08` |

#### Error Payload

| Byte 2                                |
| ------------------------------------- |
| [`Error Reason`](#value:error-reason) |

<a name="command:get-node-status"></a>

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

- <a name="value:error-bit">`Error Bit`</a>:

  | Value | Meaning  |
  | ----- | -------- |
  | `0`   | No Error |
  | `1`   | Error    |

- <a name="value:network-state">`Network State`</a>:

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

- <a name="value:radio-port">`Radio Port`</a>:

  | Value | Meaning             |
  | ----- | ------------------- |
  | `0`   | Radio Port Disabled |
  | `1`   | Radio Port Enabled  |

- <a name="value:can-port">`CAN Port`</a>:

  | Value | Meaning           |
  | ----- | ----------------- |
  | `0`   | CAN Port Disabled |
  | `1`   | CAN Port Enabled  |

- <a name="value:radio-activity">`Radio Activity`</a>:

  | Value | Meaning                     |
  | ----- | --------------------------- |
  | `0`   | Disconnected from Bluetooth |
  | `1`   | Connected to Bluetooth      |

#### Payload

- Setting the value `0` for the node status word mask means that we request the status word
- Currently the only supported payload should be 8 null (`0x00`) bytes

##### STH

| Byte 1    |                                         |                                 |
| --------- | --------------------------------------- | ------------------------------- |
| Bit 7 – 4 | Bit 3 – 1                               | Bit 0                           |
| Reserved  | [`Network State`](#value:network-state) | [`Error Bit`](#value:error-bit) |

##### STU

| Byte 1   |                                           |                                       |                                           |                                         |                                 |
| -------- | ----------------------------------------- | ------------------------------------- | ----------------------------------------- | --------------------------------------- | ------------------------------- |
| Bit 7    | Bit 6                                     | Bit 5                                 | Bit 4                                     | Bit 3 – 1                               | Bit 0                           |
| Reserved | [`Radio Activity`](#value:radio-activity) | [`CAN Port`](#value:can-port) Enabled | [`Radio Port`](#value:radio-port) Enabled | [`Network State`](#value:network-state) | [`Error Bit`](#value:error-bit) |

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

<a name="command: get-error-status"></a>

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

- <a name="value:transmission-failure">`Transmission Failure`</a> (Bluetooth for STH, CAN for STU):

  | Value | Meaning                 |
  | ----- | ----------------------- |
  | `0`   | No Transmission Failure |
  | `1`   | Transmission Failure    |

- <a name="value:adc-overrun">`ADC Overrun`</a>:

  | Value | Meaning              |
  | ----- | -------------------- |
  | `0`   | No ADC Overrun Error |
  | `1`   | ADC Overrun Error    |

#### Payload

- Setting the value `0` for the status word mask means that we request the error status word
- Currently the only supported payload should be 8 null (`0x00`) bytes

##### STH

| Byte 1    |                                     |                                                                 |
| --------- | ----------------------------------- | --------------------------------------------------------------- |
| Bit 7 – 2 | Bit 1                               | Bit 0                                                           |
| Reserved  | [`ADC Overrun`](#value:adc-overrun) | Bluetooth [`Transmission Failure`](#value:transmission-failure) |

##### STU

| Byte 1    |                                                           |
| --------- | --------------------------------------------------------- |
| Bit 7 – 2 | Bit 0                                                     |
| Reserved  | CAN [`Transmission Failure`](#value:transmission-failure) |

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

<a name="block:streaming"></a>

## Block `Streaming`

| Number | Block Command                         | Access | Permanently Stored |
| ------ | ------------------------------------- | ------ | ------------------ |
| `0x01` | [Acceleration](#command:acceleration) | Event  | –                  |
| `0x20` | [Voltage](#command:voltage)           | Event  | –                  |

### Values

- The <a name="value:data-sets">`Data Sets`</a> bits used in the sections below can have the following values:

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

- <a name="value:request">`Request`</a>:

  | Value | Meaning        |
  | ----- | -------------- |
  | `0`   | Stream         |
  | `1`   | Single Request |

- <a name="value:bytes">`Bytes`</a>:

  | Value | Meaning                     |
  | ----- | --------------------------- |
  | `0`   | 2 Bytes for each data point |
  | `1`   | 3 Byte for data point       |

- <a name="value:active">`Active`</a>

  | Value | Meaning                                                 |
  | ----- | ------------------------------------------------------- |
  | `0`   | Data for specified data point will not be measured/sent |
  | `1`   | Data for specified data point will be measured/sent     |

<a name="command:acceleration"></a>

### Command `Acceleration`

- Requesting while streaming is possible
- Only single stream allowed
- Requesting stream in different format stops last stream
- Tuple format (depending on active axis, see payload):
  - x/y/z
  - x/y
  - x/z
  - y/z

#### Payload

| Byte 1                      |                         |                                  |                                  |                                  |                                 |
| --------------------------- | ----------------------- | -------------------------------- | -------------------------------- | -------------------------------- | ------------------------------- |
| Bit 7                       | Bit 6                   | Bit 5                            | Bit 4                            | Bit 3                            | Bit 2 – 0                       |
| [`Request`](#value:request) | [`Bytes`](#value:bytes) | X-Axis [`Active`](#value:active) | Y-Axis [`Active`](#value:active) | Z-Axis [`Active`](#value:active) | [`Data Sets`](#value:data-sets) |

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

<a name="command:voltage"></a>

### Command `Voltage`

#### Notes

- Highest voltage sampling rate determines bit stream rate
- Requesting while streaming is possible

#### Payload

| Byte 1                      |                         |                                     |                                     |                                     |                                 |
| --------------------------- | ----------------------- | ----------------------------------- | ----------------------------------- | ----------------------------------- | ------------------------------- |
| Bit 7                       | Bit 6                   | Bit 5                               | Bit 4                               | Bit 3                               | Bit 2 – 0                       |
| [`Request`](#value:request) | [`Bytes`](#value:bytes) | Voltage 1 [`Active`](#value:active) | Voltage 2 [`Active`](#value:active) | Voltage 3 [`Active`](#value:active) | [`Data Sets`](#value:data-sets) |

#### Acknowledgment Payload

The command uses the same format as the “Acknowledgment Payload” of the `Acceleration` command.

## Block `Statistical Data and Quantity`

| Number | Block Command                     | Access | Permanently Stored |
| ------ | --------------------------------- | ------ | ------------------ |
| `0x00` | Power On Cycles, Power Off Cycles | Read   | x                  |
| `0x01` | Operating time                    | Read   | x                  |
| `0x02` | Under Voltage Counter             | Read   | x                  |
| `0x03` | Watchdog Reset Counter            | Read   | x                  |
| `0x04` | Production Date                   | Read   | x                  |

### Command `Power On Cycles, Power Off Cycles`

#### Notes

- Power off means power away e.g. Accumulator out of energy
- Power On includes resets

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB) |
| --------------------------- |
| Power On Cycles             |

| Byte 5 (MSB) - Byte 8 (LSB) |
| --------------------------- |
| Power Off Cycles            |

### Command `Operating time`

#### Notes

- Seconds since first power are stored each half an hour
- The STH also stored seconds since reset in disconnect case.

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB) |
| --------------------------- |
| Seconds since reset         |

| Byte 5 (MSB) - Byte 8 (LSB)  |
| ---------------------------- |
| Seconds since first power on |

### Command `Under Voltage Counter`

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB)                |
| ------------------------------------------ |
| Under voltage counter since first power on |

### Command `Watchdog Reset Counter`

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB)          |
| ------------------------------------ |
| Watchdog Resets since first power on |