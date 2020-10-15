# Commands

## Blocks

| Block  | Short Description                                                     | Extended Description                                                                                                                                |
| ------ | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x00` | [System](#block:system)                                               | System commands are used to modify/request the state of each unit (e.g. reset) or an the overall system state (e.g. transmission speed)             |
| `0x04` | [Streaming](#block:streaming)                                         | Streaming commands are used to transmit data streams, but may be also used for single requests. The super frame is also located in this block.      |
| `0x08` | [Statistical Data and Quantity](#block:Statistical-Data-and-Quantity) | This command group is used to store statistical data that can be used for histograms such as operating time and the number of power on/off cycles   |
| `0x28` | [Configuration](#block:Configuration)                                 | This command block is used to set configuration data (e.g. you can set the sampling rate of acceleration data here).                                |
| `0x3D` | [EEPROM](#block:EEPROM)                                               | Used for writing and reading EEPROM data directly                                                                                                   |
| `0x3E` | [ProductData and RFID](#block:ProductionData-and-RFID)                | Used to store product data like a serial number. Furthermore, this block provides access to RFID information that is supported via connected tools. |
| `0x3F` | [Test](#block:Test)                                                   | Test Config Page                                                                                                                                    |

<a name="block:system"></a>

## Block `System`

| Number | Block Command                                 | Access     | Permanently Stored |
| ------ | --------------------------------------------- | ---------- | ------------------ |
| `0x00` | [Verboten](#command:verboten)                 | –          | –                  |
| `0x01` | [Reset](#command:reset)                       | Event      | –                  |
| `0x02` | [Get/Set State](#command:get-set-state)       | Read/Write | –                  |
| `0x05` | [Get Node Status](#command:get-node-status)   | Read/Write | –                  |
| `0x06` | [Get Error Status](#command:get-error-status) | Read/Write | –                  |
| `0x0B` | [Bluetooth](#command:bluetooth)               | Read       | –                  |

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

<a name="command:get-error-status"></a>

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

<a name="command:bluetooth"></a>

### Command `Bluetooth`

- <a name="value:bluetooth-subcommand">`Bluetooth Subcommand`</a>

  | Value | Meaning                                                                         |
  | ----- | ------------------------------------------------------------------------------- |
  | `0`   | Reserved                                                                        |
  | `1`   | Connect                                                                         |
  | `2`   | Get number of available devices                                                 |
  | `3`   | Write device name #1 and set device name #2 to `NULL`                           |
  | `4`   | Write device name #2 and push it to STH (read will be equivalent in the future) |
  | `5`   | Read first part (6 bytes) of device name                                        |
  | `6`   | Read second part (2 bytes) of device name                                       |
  | `7`   | Connect to device (with device number)                                          |
  | `8`   | Check if connected                                                              |
  | `9`   | Disconnect                                                                      |
  | `10`  | Get send counter                                                                |
  | `11`  | Received RX frames                                                              |
  | `12`  | Absolute RSSI indicator (number must be interpreted as negative)                |
  | `13`  | Read energy mode reduced                                                        |
  | `14`  | Write energy mode reduced                                                       |
  | `15`  | Read energy mode lowest                                                         |
  | `16`  | Write energy mode lowest                                                        |
  | `17`  | Bluetooth MAC address                                                           |
  | `18`  | Connect to device (with Bluetooth MAC address)                                  |

- <a name="value:bluetooth-device-number">`Device Number`</a>: Sequential positive number assigned by STU to available STH nodes; For a single STH this number will be `0`.

- <a name="value:bluetooth-value">`Bluetooth Value`</a>

  | [`Bluetooth Subcommand`](#value:bluetooth-subcommand) | Value                                                                                                                                                  |
  | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | `0`                                                   | –                                                                                                                                                      |
  | `1`                                                   | –                                                                                                                                                      |
  | `2`                                                   | –                                                                                                                                                      |
  | `3`                                                   | ASCII string                                                                                                                                           |
  | `4`                                                   | ASCII string (`NULL`)                                                                                                                                  |
  | `5`                                                   | –                                                                                                                                                      |
  | `6`                                                   | –                                                                                                                                                      |
  | `7`                                                   | –                                                                                                                                                      |
  | `8`                                                   | –                                                                                                                                                      |
  | `9`                                                   | –                                                                                                                                                      |
  | `10`                                                  | –                                                                                                                                                      |
  | `11`                                                  | –                                                                                                                                                      |
  | `12`                                                  | –                                                                                                                                                      |
  | `13`                                                  | –                                                                                                                                                      |
  | `14`                                                  | Byte 3 – 6: Time form normal to reduced energy mode in ms <br> Byte 7 – 8: Advertisement time for reduced energy mode in ms <br> Big endian            |
  | `15`                                                  | –                                                                                                                                                      |
  | `16`                                                  | Byte 3 – 6: Time form reduced to lowest energy mode in ms <br> Byte 7 – 8: Advertisement time for lowest energy mode in ms <br> Little endian 0 = read |
  | `17`                                                  | Byte 3: Value `255` (self addressing)                                                                                                                  |

- <a name="value:bluetooth-return-value">`Bluetooth Return Value`</a>

  | [`Bluetooth Subcommand`](#value:bluetooth-subcommand) | Value                                                                                                                                         |
  | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
  | `0`                                                   | `NULL`                                                                                                                                        |
  | `1`                                                   | `NULL`                                                                                                                                        |
  | `2`                                                   | ASCII string containing the number of available devices                                                                                       |
  | `3`                                                   | ASCII string containing the first 6 characters of the Bluetooth advertisement name                                                            |
  | `4`                                                   | ASCII string containing the last 2 characters of the Bluetooth advertisement name                                                             |
  | `5`                                                   | ASCII string                                                                                                                                  |
  | `6`                                                   | ASCI string of connected device or `NULL` if not connected                                                                                    |
  | `7`                                                   | `NULL` (Byte 3 is `true` if in search mode, at least single device was found, no legacy mode and scanning mode active; `false` otherwise)     |
  | `8`                                                   | `NULL` if not connected <br> Not `NULL` otherwise                                                                                             |
  | `9`                                                   | `NULL`                                                                                                                                        |
  | `10`                                                  | 6 Byte `unsigned int`                                                                                                                         |
  | `11`                                                  | 6 Byte `unsigned int`                                                                                                                         |
  | `12`                                                  | Byte 3: `uint8_t` deviceType <br> Byte 4: `int8_t` <br> Byte 5 – 8: `NULL`                                                                    |
  | `13`                                                  | Byte 3 – 6: Time form normal to reduced energy mode in ms <Br> Byte 7 – 8: Advertisement time for reduced energy mode in ms <br> Big Endian   |
  | `14`                                                  | Byte 3 – 6: Time form normal to reduced energy mode in ms <Br> Byte 7 – 8: Advertisement time for reduced energy mode in ms <br> Big Endian   |
  | `15`                                                  | Byte 3 – 6: Time form reduced to lowest energy mode in ms <Br> Byte 7 – 8: Advertisement time for lowest energy mode in ms <br> Little Endian |
  | `16`                                                  | Byte 3 – 6: Time form reduced to lowest energy mode in ms <Br> Byte 7 – 8: Advertisement time for lowest energy mode in ms <br> Little Endian |
  | `17`                                                  | Byte 3 – 6: Bluetooth MAC address in little endian format                                                                                     |

#### Payload

| Byte 1                                                |
| ----------------------------------------------------- |
| [`Bluetooth Subcommand`](#value:bluetooth-subcommand) |

| Byte 2                                            |
| ------------------------------------------------- |
| [`Device Number`](#value:bluetooth-device-number) |

| Byte 3 – 8                                  |
| ------------------------------------------- |
| [`Bluetooth Value`](#value:bluetooth-value) |

**Note:** Use `0` bytes if Device Number or [`Bluetooth Value`](#value:bluetooth-value) are not applicable (e.g. when you use the `Connect` command)

#### Acknowledgement Payload

| Byte 1          |
| --------------- |
| Same as Payload |

| Byte 2          |
| --------------- |
| Same as Payload |

| Byte 3 – 8                                                |
| --------------------------------------------------------- |
| [`Bluetooth Return Value`](#value:bluetooth-return-value) |

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

<a name="block:Statistical-Data-and-Quantity"></a>

## Block `Statistical Data and Quantity`

| Number | Block Command                                                                  | Access | Permanently Stored |
| ------ | ------------------------------------------------------------------------------ | ------ | ------------------ |
| `0x00` | [Power On Cycles, Power Off Cycles](#command:Power-On-Cycles-Power-Off-Cycles) | Read   | x                  |
| `0x01` | [Operating time](#command:Operating-time)                                      | Read   | x                  |
| `0x02` | [Under Voltage Counter](#command:Under-Voltage-Counter)                        | Read   | x                  |
| `0x03` | [Watchdog Reset Counter](#command:Watchdog-Reset-Counter)                      | Read   | x                  |
| `0x04` | [Production Date](#command:Production-Date)                                    | Read   | x                  |

<a name="command:Power-On-Cycles-Power-Off-Cycles"></a>

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

<a name="command:Operating-time"></a>

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

<a name="command:Under-Voltage-Counter"></a>

### Command `Under Voltage Counter`

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB)                |
| ------------------------------------------ |
| Under voltage counter since first power on |

<a name="command:Watchdog-Reset-Counter"></a>

### Command `Watchdog Reset Counter`

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB)          |
| ------------------------------------ |
| Watchdog Resets since first power on |

<a name="command:Production-Date"></a>

### Command `Production Date`

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB)                                                              |
| ---------------------------------------------------------------------------------------- |
| ASCII String of the Production Date in the format: yyyymmdd where y=year, m=month, d=day |

<a name="block:Configuration"></a>

## Block `Configuration`

| Number | Block Command                                                                     | Access     | Permanently Stored |
| ------ | --------------------------------------------------------------------------------- | ---------- | ------------------ |
| `0x00` | [Get/Set Acceleration Configuration](#command:Get-Set-Acceleration-Configuration) | Read/Write | x                  |
| `0x60` | [Get/Set Calibration Factor k](#command:Get-Set-Calibration-Factor-k)             | Read/Write | x                  |
| `0x61` | [Get/Set Calibration Factor d](#command:Get-Set-Calibration-Factor-d)             | Read/Write | x                  |
| `0x62` | [Calibration Measurement](#command:Callibration-Measurement)                      | Read/Write | x                  |
| `0xC0` | [HMI Configuration](#command:HMI-Configuration)                                   | Read/Write | x                  |

<a name="command:Get-Set-Acceleration-Configuration"></a>

### Command `Get/Set Acceleration Configuration`

#### Notes

##### Sampling Rate

$$ \frac{f_{CLOCK}}{(Prescaler+1)·(AcquisitionTime + 12+1) · OverSamplingRate} $$

$$f_{clock}=38400000 Hz$$

##### Prescaler

- Byte2

##### Acquisition Time

- Sample and Hold Time i.e. Time to charge capacitor that is cut off and measured at digital quantisation
- 2^(Byte3-1) iff Byte3 > AdcAcquisitionTime4
- (Byte3+2) iff Byte3 < =AdcAcquisitionTime4

##### Over Sampling Rate

- 2^Byte4; No Over Sampling if Byte4=0

##### ADC Reference Voltage

- 1V25
- 1V65
- 1V8
- 2V1
- 2V2
- 2V5
- 2V7
- 3V3(VDD)
- 5V
- 6V6

##### Setting at Reset

- 2/Aqu8(4)/OverSampling64(6)/VDD

#### Values

- <a name="value:get-set-state2">`Get/Set State`</a>:

  | Value | Meaning   |
  | ----- | --------- |
  | `0`   | Get State |
  | `1`   | Set State |

#### Payload

| Byte 1                                   |           |
| ---------------------------------------- | --------- |
| Bit 7                                    | Bit 6 – 0 |
| [`Get/Set State`](#value:get-set-state2) | Reserved  |

| Byte 2        |
| ------------- |
| ADC Prescaler |

| Byte 3                      |
| --------------------------- |
| Acquisition time (See Note) |

| Byte 4                                                                          |
| ------------------------------------------------------------------------------- |
| Power of over sampling rate e.g. 10->1024 OverSampling Rate, 0=no Over Sampling |

| Byte 5                               |
| ------------------------------------ |
| Reference: Voltage\*20 e.g. 3.3V->66 |

| Byte 6 - Byte 8 |
| --------------- |
| Reserved        |

#### Acknowledgment Payload

- Same structure as payload

<a name="command:Get-Set-Calibration-Factor-k"></a>

### Command `Get/Set Calibration Factor k`

#### Values

- <a name="value:calibration-element">`Calibration Element`</a>:

  | Value | Meaning      |
  | ----- | ------------ |
  | `0`   | Acceleration |
  | `1`   | Temperature  |
  | `32`  | Voltage      |

- <a name="value:number-of-axis">`Number or axis`</a>:

  | Value | Meaning                       |
  | ----- | ----------------------------- |
  | `0`   | Reserved                      |
  | `1`   | x-Axis / First measure point  |
  | `2`   | y-Axis / Second measure point |
  | `3`   | z-Axis / Third measure point  |

- <a name="value:get-set-value">`Get/Set Value`</a>:

  | Value | Meaning   |
  | ----- | --------- |
  | `0`   | Get Value |
  | `1`   | Set Value |

#### Payload

| Byte 1                                              |
| --------------------------------------------------- |
| [`Calibration Element`](#value:calibration-element) |

| Byte 2                                    |
| ----------------------------------------- |
| [`Number or axis`](#value:number-of-axis) |

| Byte 3                                  |           |
| --------------------------------------- | --------- |
| Bit 7                                   | Bit 6 – 0 |
| [`Get/Set Value`](#value:get-set-value) | Reserved  |

| Byte 4   |
| -------- |
| Reserved |

| Byte 5 (MSB) - Byte 8 (LSB)                                                                                                            |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| k (Slope) according to IEEE 754 single precision (float)<br /><br />Calibration=kx+d (Also calculation to SI value or any other value) |

#### Acknowledgment Payload

| Byte 1                                              |
| --------------------------------------------------- |
| [`Calibration Element`](#value:calibration-element) |

| Byte 2                                    |
| ----------------------------------------- |
| [`Number or axis`](#value:number-of-axis) |

| Byte 3   |
| -------- |
| Reserved |

| Byte 4   |
| -------- |
| Reserved |

| Byte 5 (MSB) - Byte 8 (LSB)                                                                                                            |
| -------------------------------------------------------------------------------------------------------------------------------------- |
| k (Slope) according to IEEE 754 single precision (float)<br /><br />Calibration=kx+d (Also calculation to SI value or any other value) |

<a name="command:Get-Set-Callibration-Factor-d"></a>

### Command `Get/Set Calibration Factor d`

Payload and Acknowledgment Payload have the same Structure as [`Get/Set Calibration Factor k`](#command:Get-Set-Calibration-Factor-k) but with `d (Offset)` instead of `k (Slope)` from `kx+d`.

<a name="command:Calibration-Measurement"></a>

### Command `Calibration Measurement`

#### Values

- <a name="value:calibration-Get-Set">`Calibration Get/Set`</a>:

  | Value | Meaning                                       |
  | ----- | --------------------------------------------- |
  | `0`   | Get (Ignores the remaining bits of this byte) |
  | `1`   | Set                                           |

- <a name="value:calibration-Method">`Calibration Method`</a>:

  | Value | Meaning  |
  | ----- | -------- |
  | `0`   | Reserved |
  | `1`   | Inject   |
  | `2`   | Eject    |
  | `3`   | Measure  |

- <a name="value:calibration-Measurement-Element">`Calibration Measurement Element`</a>:

  | Value | Meaning                                                        |
  | ----- | -------------------------------------------------------------- |
  | `0`   | Acceleration                                                   |
  | `1`   | Temperature (for VREF=1250 temperature gets calculated to m°C) |
  | `32`  | Voltage                                                        |
  | `96`  | VSS(Ground)                                                    |
  | `97`  | VDD (Supply)                                                   |
  | `98`  | Regulated Internal Power                                       |
  | `99`  | Operation Amplifier Output                                     |

- <a name="value:Dimension">`Dimension`</a>:

  | Value | Meaning         |
  | ----- | --------------- |
  | `0`   | Reserved        |
  | `1`   | 1. Dimension(x) |
  | `2`   | 2. Dimension(y) |
  | `3`   | 3. Dimension(z) |

#### Payload

| Byte 1                                              |                                                   |       |               |
| --------------------------------------------------- | ------------------------------------------------- | ----- | ------------- |
| Bit 7                                               | Bit 6 - Bit 5                                     | Bit 4 | Bit 3 - Bit 0 |
| [`Calibration Get/Set`](#value:calibration-Get-Set) | [`Calibration Method`](#value:calibration-Method) | Reset | Reserved      |

| Byte 2                                                                      |
| --------------------------------------------------------------------------- |
| [`Calibration Measurement Element`](#value:calibration-Measurement-Element) |

| Byte 3                          |
| ------------------------------- |
| [`Dimension`](#value:Dimension) |

| Byte 4   |
| -------- |
| Reserved |

| Byte 5 - Byte 8 |
| --------------- |
| Reserved        |

#### Acknowledgment Payload

| Byte 1 - Byte 4 |
| --------------- |
| Same as Payload |

| Byte 5 - Byte 8 |
| --------------- |
| Result          |

<a name="command:HMI-Configuration"></a>

### Command `HMI Configuration`

#### Values

- <a name="value:Get-Set-Sampling-Rate">`Get/Set Sampling Rate`</a>:

  | Value | Meaning           |
  | ----- | ----------------- |
  | `0`   | Get Sampling Rate |
  | `1`   | Set Sampling Rate |

- <a name="value:LED">`LED`</a>:

  | Value | Meaning  |
  | ----- | -------- |
  | `0`   | Reserved |
  | `1`   | LED      |

- <a name="value:ON-OFF">`ON/OFF`</a>:

  | Value | Meaning          |
  | ----- | ---------------- |
  | `0`   | Reserved         |
  | `1`   | On (Reset value) |
  | `2`   | Off              |

#### Payload

| Byte 1                                                  |                     |
| ------------------------------------------------------- | ------------------- |
| Bit 7                                                   | Bit 6 - Bit 0       |
| [`Get/Set Sampling Rate`](#value:Get-Set-Sampling-Rate) | [`LED`](#value:LED) |

| Byte 2         |
| -------------- |
| Number (0-255) |

| Byte 3                    |
| ------------------------- |
| [`ON/OFF`](#value:ON-OFF) |

| Byte 4   |
| -------- |
| Reserved |

| Byte 5 - Byte 8 |
| --------------- |
| Reserved        |

#### Acknowledgment Payload

- Same structure as payload

<a name="block:EEPROM"></a>

## Block `EEPROM`

| Number | Block Command                         | Access | Permanently Stored |
| ------ | ------------------------------------- | ------ | ------------------ |
| `0x00` | [EEPROM Read](#command:EEPROM-Read)   | Read   | x                  |
| `0x01` | [EEPROM Write](#command:EEPROM-Write) | Write  | x                  |

<a name="command:EEPROM-Read"></a>

### Command `EEPROM Read`

#### Notes

- Used to read data from EEPROM directly.

#### Payload

| Byte 1 | Byte 2 | Byte 3 | Byte 4   | Byte 5 - Byte 8 |
| ------ | ------ | ------ | -------- | --------------- |
| Page   | Offset | Length | Reserved | Reserved        |

#### Acknowledgment Payload

| Byte 1 | Byte 2 | Byte 3 | Byte 4   | Byte 5 - Byte 8  |
| ------ | ------ | ------ | -------- | ---------------- |
| Page   | Offset | Length | Reserved | Data (MSB first) |

<a name="command:EEPROM-Write"></a>

### Command `EEPROM Write`

#### Notes

- Used to write data to EEPROM directly.
- It is not allowed to write everything if the byte 0 is locked(0xCA)

#### Payload

| Byte 1 | Byte 2 | Byte 3 | Byte 4   | Byte 5 - Byte 8  |
| ------ | ------ | ------ | -------- | ---------------- |
| Page   | Offset | Length | Reserved | Data (MSB first) |

#### Acknowledgment Payload

- Same structure as payload

<a name="block:ProductionData-and-RFID"></a>

## Block `ProductData and RFID`

| Number          | Block Command                                                           | Access | Permanently Stored |
| --------------- | ----------------------------------------------------------------------- | ------ | ------------------ |
| `0x00`          | [Global Trade Identification Number (GTIN)](#command:GTIN)              | Read   | x                  |
| `0x01`          | [Hardware Revision](#command:Hardware-Revision)                         | Read   | x                  |
| `0x02`          | [Firmware Version](#command:Firmware-Version)                           | Read   | x                  |
| `0x03`          | [Release Name](#command:Release-Name)                                   | Read   | x                  |
| `0x04` - `0x07` | [Serial Number 1-4](#command:Serial-Number)                             | Read   | x                  |
| `0x08` - `0x17` | [Name 1-16](#command:Name)                                              | Read   | x                  |
| `0x18` - `0x1F` | [OEM Free Use 0-7](#command:OEM-Free-Use)                               | Read   | x                  |
| `0x80`          | [Tool RFID product information](#command:Tool-RFID-product-information) | Read   | -                  |

<a name="command:GTIN"></a>

### Command `Global Trade Identification Number (GTIN)`

#### Acknowledgment Payload

- 8 Byte GTIN
- Format: uint64_t (unsigned int)
- MSB: Byte 0
- LSB: Byte 7

<a name="command:Hardware-Revision"></a>

### Command `Hardware Revision`

#### Acknowledgment Payload

- 8 Bytes totally
  - 5 Bytes Reserved
  - 1 Byte Major Revision
  - 1 Byte Minor Revision
  - 1 Byte Build Revision

<a name="command:Firmware-Version"></a>

### Command `Firmware Version`

#### Acknowledgment Payload

- 8 Bytes totally
  - 5 Bytes Reserved
  - 1 Byte Major Revision
  - 1 Byte Minor Revision
  - 1 Byte Build Revision

<a name="command:Release-Name"></a>

### Command `Release Name`

#### Acknowledgment Payload

- 8 Byte ASCII Code
- NULL terminated or 8 Byte long

<a name="command:Serial-Number"></a>

### Command `Serial Number`

#### Notes

| Command | Purpose         |
| ------- | --------------- |
| `0x04`  | Serial Number 1 |
| `0x05`  | Serial Number 2 |
| `0x06`  | Serial Number 3 |
| `0x07`  | Serial Number 4 |

#### Acknowledgment Payload

- UTF-8 String (8 Byte)

<a name="command:Name"></a>

### Command `Name`

#### Notes

- Multiple Strings in different languages possible

| Command | Purpose |
| ------- | ------- |
| `0x08`  | Name 1  |
| `0x09`  | Name 2  |
| `0x0A`  | Name 3  |
| `0x0B`  | Name 4  |
| `0x0C`  | Name 5  |
| `0x0D`  | Name 6  |
| `0x0E`  | Name 7  |
| `0x0F`  | Name 8  |
| `0x10`  | Name 9  |
| `0x11`  | Name 10 |
| `0x12`  | Name 11 |
| `0x13`  | Name 12 |
| `0x14`  | Name 13 |
| `0x15`  | Name 14 |
| `0x16`  | Name 15 |
| `0x17`  | Name 16 |

#### Acknowledgment Payload

- UTF-8 String (8 Byte)

<a name="command:OEM-Free-Use"></a>

### Command `OEM Free Use`

#### Notes

| Command | Purpose        |
| ------- | -------------- |
| `0x18`  | OEM Free Use 0 |
| `0x19`  | OEM Free Use 1 |
| `0x1A`  | OEM Free Use 2 |
| `0x1B`  | OEM Free Use 3 |
| `0x1C`  | OEM Free Use 4 |
| `0x1D`  | OEM Free Use 5 |
| `0x1E`  | OEM Free Use 6 |
| `0x1F`  | OEM Free Use 7 |

#### Acknowledgment Payload

- 8 Byte

<a name="command:Tool-RFID-product-information"></a>

### Command `Tool RFID product information`

#### Acknowledgment Payload

- to be determined

<a name="block:Test"></a>

## Block `Test`

| Number | Block Command                       | Access | Permanently Stored |
| ------ | ----------------------------------- | ------ | ------------------ |
| `0x00` | Reserved                            | -      | -                  |
| `0x01` | [Test signal](#command:Test-signal) | -      | -                  |

<a name="command:Test-signal"></a>

### Command `Test signal`

#### Payload

##### Byte 1:

| Value | Meaning  |
| ----- | -------- |
| 0     | Reserved |
| 1     | Line     |
| 2     | Ramp     |

##### Byte 2:

Module (Module specific)

##### Byte 3-8:

Module specific

#### Acknowledgment Payload

##### Byte 1:

| Value | Meaning  |
| ----- | -------- |
| 0     | Reserved |
| 1     | Line     |
| 2     | Ramp     |

##### Byte 2-3:

Module (Module specific)

##### Byte 4-8:

Module specific

## Errors

| Value | Description                     | Example                                    |
| ----- | ------------------------------- | ------------------------------------------ |
| `0`   | Specific Error                  |                                            |
| `1`   | Not available                   |                                            |
| `2`   | General Error                   |                                            |
| `3`   | Write not allowed               | Setting of memory area in word not allowed |
| `4`   | Unsupported format              | 64 Byte Data via CAN2.0 is not possible    |
| `5`   | Wrong key/magic number          |                                            |
| `6`   | No SuperFrame inside SuperFrame |                                            |
| `7`   | EEPROM defect                   |                                            |
