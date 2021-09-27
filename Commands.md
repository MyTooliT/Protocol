# Commands

## Blocks

|  Block | Short Description                                                     | Extended Description                                                                                                                                |
| -----: | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
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
| -----: | --------------------------------------------- | ---------- | ------------------ |
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

- <a name="value:get-set-state"></a>`Get/Set State`:

  | Value | Meaning   |
  | ----: | --------- |
  |   `0` | Get State |
  |   `1` | Set State |

- <a name="value:location"></a>`Location`:

  | Value | Meaning     |
  | ----: | ----------- |
  |   `0` | No Change   |
  |   `1` | Bootloader  |
  |   `2` | Application |
  |   `3` | Reserved    |

- <a name="value:state"></a>`State`:

  | Value | Meaning                                                                          |
  | ----: | -------------------------------------------------------------------------------- |
  |   `0` | Failure (No acknowledgement will be sent; <br/> Only power on resets this state) |
  |   `1` | Error (No active communication)                                                  |
  |   `2` | Turn Off/Standby                                                                 |
  |   `3` | Graceful degradation level 2                                                     |
  |   `4` | Graceful degradation level 1                                                     |
  |   `5` | Operating                                                                        |
  |   `6` | Startup                                                                          |
  |   `7` | No change                                                                        |

- <a name="value:error-reason"></a>`Error Reason`:

  | Value | Meaning                                                     |
  | ----: | ----------------------------------------------------------- |
  |   `1` | Set state not available                                     |
  |   `2` | Wrong subscriber (e.g. accessing application as bootloader) |

#### Payload

|                 Byte 1                  |          |                               |          |                         |
| :-------------------------------------: | :------: | :---------------------------: | :------: | :---------------------: |
|                  Bit 7                  |  Bit 6   |           Bit 5 – 4           |  Bit 3   |        Bit 2 – 0        |
| [`Get/Set State`](#value:get-set-state) | Reserved | [`Location`](#value:location) | Reserved | [`State`](#value:state) |

#### Acknowledgment Payload

|                 Byte 1                  |          |                               |          |                         |
| :-------------------------------------: | :------: | :---------------------------: | :------: | :---------------------: |
|                  Bit 7                  |  Bit 6   |           Bit 5 – 4           |  Bit 3   |        Bit 2 – 0        |
| [`Get/Set State`](#value:get-set-state) | Reserved | [`Location`](#value:location) | Reserved | [`State`](#value:state) |

#### Error Payload

|                Byte 2                 |
| :-----------------------------------: |
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

- STU node status word:

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

- <a name="value:error-bit"></a>`Error Bit`:

  | Value | Meaning  |
  | ----: | -------- |
  |   `0` | No Error |
  |   `1` | Error    |

- <a name="value:network-state"></a>`Network State`:

  | Value | Meaning                |
  | ----: | ---------------------- |
  |   `0` | Failure                |
  |   `1` | Error                  |
  |   `2` | Standby                |
  |   `3` | Graceful Degradation 2 |
  |   `4` | Graceful Degradation 1 |
  |   `5` | Operating              |
  |   `6` | Startup                |
  |   `7` | No Change              |

- <a name="value:radio-port"></a>`Radio Port`:

  | Value | Meaning             |
  | ----: | ------------------- |
  |   `0` | Radio Port Disabled |
  |   `1` | Radio Port Enabled  |

- <a name="value:can-port"></a>`CAN Port`:

  | Value | Meaning           |
  | ----: | ----------------- |
  |   `0` | CAN Port Disabled |
  |   `1` | CAN Port Enabled  |

- <a name="value:radio-activity"></a>`Radio Activity`:

  | Value | Meaning                     |
  | ----: | --------------------------- |
  |   `0` | Disconnected from Bluetooth |
  |   `1` | Connected to Bluetooth      |

#### Payload

- Setting the value `0` for the node status word mask means that we request the status word
- Currently the only supported payload should be 8 null (`0x00`) bytes

##### STH

|  Byte 1   |                                         |                                 |
| :-------: | :-------------------------------------: | :-----------------------------: |
| Bit 7 – 4 |                Bit 3 – 1                |              Bit 0              |
| Reserved  | [`Network State`](#value:network-state) | [`Error Bit`](#value:error-bit) |

##### STU

|  Byte 1  |                                           |                                       |                                           |                                         |                                 |
| :------: | :---------------------------------------: | :-----------------------------------: | :---------------------------------------: | :-------------------------------------: | :-----------------------------: |
|  Bit 7   |                   Bit 6                   |                 Bit 5                 |                   Bit 4                   |                Bit 3 – 1                |              Bit 0              |
| Reserved | [`Radio Activity`](#value:radio-activity) | [`CAN Port`](#value:can-port) Enabled | [`Radio Port`](#value:radio-port) Enabled | [`Network State`](#value:network-state) | [`Error Bit`](#value:error-bit) |

##### STH & STU

|  Byte 2  |
| :------: |
| Reserved |

|  Byte 3  |
| :------: |
| Reserved |

|  Byte 4  |
| :------: |
| Reserved |

|      Byte 5      |
| :--------------: |
| Status Word Mask |

|      Byte 6      |
| :--------------: |
| Status Word Mask |

|      Byte 7      |
| :--------------: |
| Status Word Mask |

|      Byte 8      |
| :--------------: |
| Status Word Mask |

#### Acknowledgement Payload

- Same structure as payload

#### Error Payload

- The (possibly incorrect) length of the status word (5 instead of 4 bytes) was taken from the original documentation.

|        Byte 1         |
| :-------------------: |
| Mask Used Not Allowed |

|   Byte 4    |
| :---------: |
| Status Word |

|   Byte 5    |
| :---------: |
| Status Word |

|   Byte 6    |
| :---------: |
| Status Word |

|   Byte 7    |
| :---------: |
| Status Word |

|   Byte 8    |
| :---------: |
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

- <a name="value:transmission-failure"></a>`Transmission Failure` (Bluetooth for STH, CAN for STU):

  | Value | Meaning                 |
  | ----: | ----------------------- |
  |   `0` | No Transmission Failure |
  |   `1` | Transmission Failure    |

- <a name="value:adc-overrun"></a>`ADC Overrun`:

  | Value | Meaning              |
  | ----: | -------------------- |
  |   `0` | No ADC Overrun Error |
  |   `1` | ADC Overrun Error    |

#### Payload

- Setting the value `0` for the status word mask means that we request the error status word
- Currently the only supported payload should be 8 null (`0x00`) bytes

##### STH

|  Byte 1   |                                     |                                                                 |
| :-------: | :---------------------------------: | :-------------------------------------------------------------: |
| Bit 7 – 2 |                Bit 1                |                              Bit 0                              |
| Reserved  | [`ADC Overrun`](#value:adc-overrun) | Bluetooth [`Transmission Failure`](#value:transmission-failure) |

##### STU

|  Byte 1   |                                                           |
| :-------: | :-------------------------------------------------------: |
| Bit 7 – 2 |                           Bit 0                           |
| Reserved  | CAN [`Transmission Failure`](#value:transmission-failure) |

##### STH & STU

|  Byte 2  |
| :------: |
| Reserved |

|  Byte 3  |
| :------: |
| Reserved |

|  Byte 4  |
| :------: |
| Reserved |

|      Byte 5      |
| :--------------: |
| Status Word Mask |

|      Byte 6      |
| :--------------: |
| Status Word Mask |

|      Byte 7      |
| :--------------: |
| Status Word Mask |

|      Byte 8      |
| :--------------: |
| Status Word Mask |

#### Acknowledgement Payload

- Same structure as payload

#### Error Payload

- Same structure as error payload for node status command

<a name="command:bluetooth"></a>

### Command `Bluetooth`

- In general you need at least the following commands to connect to an STH

  1. `Activate`: Activate Bluetooth on the STU
  2. `Get number of available devices`: Check which STHs are available at the STU
  3. `Connect to device (with Bluetooth MAC address)` or `Connect to device (with device number)`: Connect to the STH at the specified STU

  Connecting to the STH will not work, if you do not **check for available devices first**

- <a name="value:bluetooth-subcommand"></a>`Bluetooth Subcommand`

  |                                 Value | Meaning                                                                         |
  | ------------------------------------: | ------------------------------------------------------------------------------- |
  |   <a name=command:bluetooth:0></a>`0` | Reserved                                                                        |
  |   <a name=command:bluetooth:1></a>`1` | Activate                                                                        |
  |   <a name=command:bluetooth:2></a>`2` | Get number of available devices                                                 |
  |   <a name=command:bluetooth:3></a>`3` | Write device name #1 and set device name #2 to `NULL`                           |
  |   <a name=command:bluetooth:4></a>`4` | Write device name #2 and push it to STH (read will be equivalent in the future) |
  |   <a name=command:bluetooth:5></a>`5` | Read first part (6 bytes) of device name                                        |
  |   <a name=command:bluetooth:6></a>`6` | Read second part (2 bytes) of device name                                       |
  |   <a name=command:bluetooth:7></a>`7` | Connect to device (with device number)                                          |
  |   <a name=command:bluetooth:8></a>`8` | Check if connected                                                              |
  |   <a name=command:bluetooth:9></a>`9` | Deactivate                                                                      |
  | <a name=command:bluetooth:10></a>`10` | Get send counter                                                                |
  | <a name=command:bluetooth:11></a>`11` | Received RX frames                                                              |
  | <a name=command:bluetooth:12></a>`12` | Get RSSI (Received Signal Strength Indication)                                  |
  | <a name=command:bluetooth:13></a>`13` | Read energy mode reduced                                                        |
  | <a name=command:bluetooth:14></a>`14` | Write energy mode reduced                                                       |
  | <a name=command:bluetooth:15></a>`15` | Read energy mode lowest                                                         |
  | <a name=command:bluetooth:16></a>`16` | Write energy mode lowest                                                        |
  | <a name=command:bluetooth:17></a>`17` | Get Bluetooth MAC address                                                       |
  | <a name=command:bluetooth:18></a>`18` | Connect to device (with Bluetooth MAC address)                                  |

- <a name="value:bluetooth-device-number"></a>`Device Number`: Sequential positive number assigned by STU to available STH nodes

  - For a single STH this number will be `0`
  - The number `255` (`0xff`) is reserved for “self addressing” (used for example when we ask a connected STH for its own MAC address). **Note**: A connected STH also returns its own name, if you use the read name subcommands ([`5`](#command:bluetooth:5) and [`6`](#command:bluetooth:6)) and a device number other than `0xff`.

- <a name="value:bluetooth-value"></a>`Bluetooth Value`

  | [`Bluetooth Subcommand`](#value:bluetooth-subcommand) | Value                                                                                                                                                    |
  | ----------------------------------------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
  |                           [`0`](#command:bluetooth:0) | –                                                                                                                                                        |
  |                           [`1`](#command:bluetooth:1) | –                                                                                                                                                        |
  |                           [`2`](#command:bluetooth:2) | –                                                                                                                                                        |
  |                           [`3`](#command:bluetooth:3) | ASCII string                                                                                                                                             |
  |                           [`4`](#command:bluetooth:4) | ASCII string (`NULL`)                                                                                                                                    |
  |                           [`5`](#command:bluetooth:5) | –                                                                                                                                                        |
  |                           [`6`](#command:bluetooth:6) | –                                                                                                                                                        |
  |                           [`7`](#command:bluetooth:7) | –                                                                                                                                                        |
  |                           [`8`](#command:bluetooth:8) | –                                                                                                                                                        |
  |                           [`9`](#command:bluetooth:9) | –                                                                                                                                                        |
  |                         [`10`](#command:bluetooth:10) | –                                                                                                                                                        |
  |                         [`11`](#command:bluetooth:11) | –                                                                                                                                                        |
  |                         [`12`](#command:bluetooth:12) | –                                                                                                                                                        |
  |                         [`13`](#command:bluetooth:13) | –                                                                                                                                                        |
  |                         [`14`](#command:bluetooth:14) | Byte 3 – 6: Time from normal to reduced energy mode in ms <br/> Byte 7 – 8: Advertisement time for reduced energy mode in ms <br/> Little endian         |
  |                         [`15`](#command:bluetooth:15) | –                                                                                                                                                        |
  |                         [`16`](#command:bluetooth:16) | Byte 3 – 6: Time from reduced to lowest energy mode in ms <br/> Byte 7 – 8: Advertisement time for lowest energy mode in ms <br/> Little endian 0 = read |
  |                         [`17`](#command:bluetooth:17) | –                                                                                                                                                        |
  |                         [`18`](#command:bluetooth:18) | Bytes of Bluetooth MAC address in reversed order (from right to left)                                                                                    |

- <a name="value:bluetooth-return-value"></a>`Bluetooth Return Value`

  | [`Bluetooth Subcommand`](#value:bluetooth-subcommand) | Value                                                                                                                                                           |
  | ----------------------------------------------------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  |                           [`0`](#command:bluetooth:0) | `NULL`                                                                                                                                                          |
  |                           [`1`](#command:bluetooth:1) | 6 Bytes containing `NULL` (`0`)                                                                                                                                 |
  |                           [`2`](#command:bluetooth:2) | ASCII string containing the number of available devices                                                                                                         |
  |                           [`3`](#command:bluetooth:3) | ASCII string                                                                                                                                                    |
  |                           [`4`](#command:bluetooth:4) | ASCII string                                                                                                                                                    |
  |                           [`5`](#command:bluetooth:5) | ASCII string containing the first 6 characters of the Bluetooth advertisement name                                                                              |
  |                           [`6`](#command:bluetooth:6) | • ASCII string containing the last 2 characters of the Bluetooth advertisement name <br/> • `NULL` if not connected                                             |
  |                           [`7`](#command:bluetooth:7) | First byte is: <br/> •`true` (`1`) if in search mode, at least single device was found, no legacy mode and scanning mode active <br/> • `false` (`0`) otherwise |
  |                           [`8`](#command:bluetooth:8) | First byte is: <br/> • `true` (`1`) if connected <br/> • `false` (`0`) otherwise <br/> Followed by 5 bytes containing `NULL` (`0`)                              |
  |                           [`9`](#command:bluetooth:9) | 6 Bytes containing `NULL` (`0`)                                                                                                                                 |
  |                         [`10`](#command:bluetooth:10) | 6 Byte `unsigned int`                                                                                                                                           |
  |                         [`11`](#command:bluetooth:11) | 6 Byte `unsigned int`                                                                                                                                           |
  |                         [`12`](#command:bluetooth:12) | • First byte contains RSSI as signed number <br/> • All other bytes are `NULL` (`0`)                                                                            |
  |                         [`13`](#command:bluetooth:13) | Byte 3 – 6: Time form normal to reduced energy mode in ms <br/> Byte 7 – 8: Advertisement time for reduced energy mode in ms <br/> Big Endian                   |
  |                         [`14`](#command:bluetooth:14) | Byte 3 – 6: Time form normal to reduced energy mode in ms <br/> Byte 7 – 8: Advertisement time for reduced energy mode in ms <br/> Big Endian                   |
  |                         [`15`](#command:bluetooth:15) | Byte 3 – 6: Time form reduced to lowest energy mode in ms <br/> Byte 7 – 8: Advertisement time for lowest energy mode in ms <br/> Little Endian                 |
  |                         [`16`](#command:bluetooth:16) | Byte 3 – 6: Time form reduced to lowest energy mode in ms <br/> Byte 7 – 8: Advertisement time for lowest energy mode in ms <br/> Little Endian                 |
  |                         [`17`](#command:bluetooth:17) | Bytes of Bluetooth MAC address in reversed order (from right to left)                                                                                           |
  |                         [`18`](#command:bluetooth:18) | Bytes of Bluetooth MAC address in reversed order (from right to left)                                                                                           |

#### Payload

|                        Byte 1                         |
| :---------------------------------------------------: |
| [`Bluetooth Subcommand`](#value:bluetooth-subcommand) |

|                      Byte 2                       |
| :-----------------------------------------------: |
| [`Device Number`](#value:bluetooth-device-number) |

|                 Byte 3 – 8                  |
| :-----------------------------------------: |
| [`Bluetooth Value`](#value:bluetooth-value) |

**Note:** Use `0` bytes if Device Number or [`Bluetooth Value`](#value:bluetooth-value) are not applicable (e.g. when you use the `Activate` command)

#### Acknowledgement Payload

|     Byte 1      |
| :-------------: |
| Same as Payload |

|     Byte 2      |
| :-------------: |
| Same as Payload |

|                        Byte 3 – 8                         |
| :-------------------------------------------------------: |
| [`Bluetooth Return Value`](#value:bluetooth-return-value) |

<a name="block:streaming"></a>

## Block `Streaming`

| Number | Block Command                         | Access | Permanently Stored |
| ------ | ------------------------------------- | ------ | ------------------ |
| `0x00` | [Acceleration](#command:acceleration) | Event  | –                  |
| `0x20` | [Voltage](#command:voltage)           | Event  | –                  |

### Values

- The <a name="value:data-sets"></a>`Data Sets` bits used in the sections below can have the following values:

  | Value | Data Amount   |
  | ----: | ------------- |
  |     0 | Stop (stream) |
  |     1 | 1 data set    |
  |     2 | 3 data sets   |
  |     3 | 6 data sets   |
  |     4 | 10 data sets  |
  |     5 | 15 data sets  |
  |     6 | 20 data sets  |
  |     7 | 30 data sets  |

  The streaming data itself can have the following structure:

  - value 1
  - value 2
  - value 3
  - value 1 / value 2 / value 3
  - value 1 / value 2
  - value 1 / value 3
  - value 2 / value 3

  The chronological order starts with the oldest value (BP) and continues with newer values (BP + t), where t is the time point.

- <a name="value:request"></a>`Request`:

  | Value | Meaning        |
  | ----: | -------------- |
  |   `0` | Stream         |
  |   `1` | Single Request |

- <a name="value:bytes"></a>`Bytes`:

  | Value | Meaning                     |
  | ----: | --------------------------- |
  |   `0` | 2 Bytes for each data point |
  |   `1` | 3 Bytes for each data point |

- <a name="value:active"></a>`Active`

  | Value | Meaning                                                 |
  | ----: | ------------------------------------------------------- |
  |   `0` | Data for specified data point will not be measured/sent |
  |   `1` | Data for specified data point will be measured/sent     |

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

|           Byte 1            |                         |                                  |                                  |                                  |                                 |
| :-------------------------: | :---------------------: | :------------------------------: | :------------------------------: | :------------------------------: | :-----------------------------: |
|            Bit 7            |          Bit 6          |              Bit 5               |              Bit 4               |              Bit 3               |            Bit 2 – 0            |
| [`Request`](#value:request) | [`Bytes`](#value:bytes) | X-Axis [`Active`](#value:active) | Y-Axis [`Active`](#value:active) | Z-Axis [`Active`](#value:active) | [`Data Sets`](#value:data-sets) |

#### Acknowledgment Payload

|     Byte 1      |
| :-------------: |
| Same as Payload |

|      Byte 2      |
| :--------------: |
| Sequence Counter |

##### Streaming Data Bytes

- Data is sent in little endian order (at least for 2 byte format)
- Older streaming data is stored in first bytes, newer data in later bytes
- Values are stored in first available bytes,
  - first value 1 (`x`) (if requested),
  - then value 2 (`y`) (if requested),
  - then value 3 (`z`) (if requested)
- Data length depends on requested values and number of sets

###### Examples

- Request first value (`x`)
- Single data set
- 2 Byte format

| Byte 3  |
| :-----: |
| x (LSB) |

| Byte 4  |
| :-----: |
| x (MSB) |

- Request second (`y`) and third value (`z`)
- Single data set
- 2 Byte format

| Byte 3  |
| :-----: |
| y (LSB) |

| Byte 4  |
| :-----: |
| y (MSB) |

| Byte 5  |
| :-----: |
| z (LSB) |

| Byte 6  |
| :-----: |
| z (MSB) |

<a name="command:voltage"></a>

### Command `Voltage`

#### Notes

- Highest voltage sampling rate determines bit stream rate
- Requesting while streaming is possible

#### Payload

|           Byte 1            |                         |                                     |                                     |                                     |                                 |
| :-------------------------: | :---------------------: | :---------------------------------: | :---------------------------------: | :---------------------------------: | :-----------------------------: |
|            Bit 7            |          Bit 6          |                Bit 5                |                Bit 4                |                Bit 3                |            Bit 2 – 0            |
| [`Request`](#value:request) | [`Bytes`](#value:bytes) | Voltage 1 [`Active`](#value:active) | Voltage 2 [`Active`](#value:active) | Voltage 3 [`Active`](#value:active) | [`Data Sets`](#value:data-sets) |

#### Acknowledgment Payload

The command uses the same format as the “Acknowledgment Payload” of the `Acceleration` command.

<a name="block:Statistical-Data-and-Quantity"></a>

## Block `Statistical Data and Quantity`

| Number | Block Command                                                                  | Access | Permanently Stored |
| -----: | ------------------------------------------------------------------------------ | ------ | ------------------ |
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
| :-------------------------: |
|       Power On Cycles       |

| Byte 5 (MSB) - Byte 8 (LSB) |
| :-------------------------: |
|      Power Off Cycles       |

<a name="command:Operating-time"></a>

### Command `Operating time`

#### Notes

- Seconds since first power are stored each half an hour
- The STH also stored seconds since reset in disconnect case.

#### ACK Payload

| Byte 1 (MSB) - Byte 4 (LSB) |
| :-------------------------: |
|     Seconds since reset     |

| Byte 5 (MSB) - Byte 8 (LSB)  |
| :--------------------------: |
| Seconds since first power on |

<a name="command:Under-Voltage-Counter"></a>

### Command `Under Voltage Counter`

#### ACK Payload

|        Byte 1 (MSB) - Byte 4 (LSB)         |
| :----------------------------------------: |
| Under voltage counter since first power on |

<a name="command:Watchdog-Reset-Counter"></a>

### Command `Watchdog Reset Counter`

#### ACK Payload

|     Byte 1 (MSB) - Byte 4 (LSB)      |
| :----------------------------------: |
| Watchdog Resets since first power on |

<a name="command:Production-Date"></a>

### Command `Production Date`

#### ACK Payload

|                               Byte 1 (MSB) - Byte 4 (LSB)                                |
| :--------------------------------------------------------------------------------------: |
| ASCII String of the Production Date in the format: yyyymmdd where y=year, m=month, d=day |

<a name="block:Configuration"></a>

## Block `Configuration`

| Number | Block Command                                                                     | Access     | Permanently Stored |
| ------ | --------------------------------------------------------------------------------- | ---------- | ------------------ |
| `0x00` | [Get/Set Acceleration Configuration](#command:Get-Set-Acceleration-Configuration) | Read/Write | x                  |
| `0x60` | [Get/Set Calibration Factor k](#command:Get-Set-Calibration-Factor-k)             | Read/Write | x                  |
| `0x61` | [Get/Set Calibration Factor d](#command:Get-Set-Calibration-Factor-d)             | Read/Write | x                  |
| `0x62` | [Calibration Measurement](#command:Calibration-Measurement)                       | Read/Write | x                  |
| `0xC0` | [HMI Configuration](#command:HMI-Configuration)                                   | Read/Write | x                  |

<a name="command:Get-Set-Acceleration-Configuration"></a>

### Command `Get/Set Acceleration Configuration`

#### Notes

##### Sampling Rate

<!-- prettier-ignore -->
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

- <a name="value:get-set-state2"></a>`Get/Set State`:

  | Value | Meaning   |
  | ----- | --------- |
  | `0`   | Get State |
  | `1`   | Set State |

#### Payload

|                  Byte 1                  |           |
| :--------------------------------------: | :-------: |
|                  Bit 7                   | Bit 6 – 0 |
| [`Get/Set State`](#value:get-set-state2) | Reserved  |

|    Byte 2     |
| :-----------: |
| ADC Prescaler |

|           Byte 3            |
| :-------------------------: |
| Acquisition time (See Note) |

|                                     Byte 4                                      |
| :-----------------------------------------------------------------------------: |
| Power of over sampling rate e.g. 10->1024 OverSampling Rate, 0=no Over Sampling |

|                Byte 5                |
| :----------------------------------: |
| Reference: Voltage\*20 e.g. 3.3V->66 |

| Byte 6 - Byte 8 |
| :-------------: |
|    Reserved     |

#### Acknowledgment Payload

- Same structure as payload

<a name="command:Get-Set-Calibration-Factor-k"></a>

### Command `Get/Set Calibration Factor k`

#### Values

- <a name="value:calibration-element"></a>`Calibration Element`:

  | Value | Meaning      |
  | ----: | ------------ |
  |   `0` | Acceleration |
  |   `1` | Temperature  |
  |  `32` | Voltage      |

- <a name="value:number-of-axis"></a>`Number or axis`:

  | Value | Meaning                       |
  | ----: | ----------------------------- |
  |   `0` | Reserved                      |
  |   `1` | x-Axis / First measure point  |
  |   `2` | y-Axis / Second measure point |
  |   `3` | z-Axis / Third measure point  |

- <a name="value:get-set-value"></a>`Get/Set Value`:

  | Value | Meaning   |
  | ----: | --------- |
  |   `0` | Get Value |
  |   `1` | Set Value |

#### Payload

|                       Byte 1                        |
| :-------------------------------------------------: |
| [`Calibration Element`](#value:calibration-element) |

|                  Byte 2                   |
| :---------------------------------------: |
| [`Number or axis`](#value:number-of-axis) |

|                 Byte 3                  |           |
| :-------------------------------------: | :-------: |
|                  Bit 7                  | Bit 6 – 0 |
| [`Get/Set Value`](#value:get-set-value) | Reserved  |

|  Byte 4  |
| :------: |
| Reserved |

|                                                      Byte 5 (MSB) - Byte 8 (LSB)                                                       |
| :------------------------------------------------------------------------------------------------------------------------------------: |
| k (Slope) according to IEEE 754 single precision (float)<br /><br />Calibration=kx+d (Also calculation to SI value or any other value) |

#### Acknowledgment Payload

|                       Byte 1                        |
| :-------------------------------------------------: |
| [`Calibration Element`](#value:calibration-element) |

|                  Byte 2                   |
| :---------------------------------------: |
| [`Number or axis`](#value:number-of-axis) |

|  Byte 3  |
| :------: |
| Reserved |

|  Byte 4  |
| :------: |
| Reserved |

|                                                      Byte 5 (MSB) - Byte 8 (LSB)                                                       |
| :------------------------------------------------------------------------------------------------------------------------------------: |
| k (Slope) according to IEEE 754 single precision (float)<br /><br />Calibration=kx+d (Also calculation to SI value or any other value) |

<a name="command:Get-Set-Callibration-Factor-d"></a>

### Command `Get/Set Calibration Factor d`

Payload and Acknowledgment Payload have the same Structure as [`Get/Set Calibration Factor k`](#command:Get-Set-Calibration-Factor-k) but with `d (Offset)` instead of `k (Slope)` from `kx+d`.

<a name="command:Calibration-Measurement"></a>

### Command `Calibration Measurement`

#### Values

- <a name="value:calibration-Get-Set"></a>`Calibration Get/Set`:

  | Value | Meaning                                       |
  | ----: | --------------------------------------------- |
  |   `0` | Get (Ignores the remaining bits of this byte) |
  |   `1` | Set                                           |

- <a name="value:calibration-Method"></a>`Calibration Method`:

  | Value | Meaning    |
  | ----: | ---------- |
  |   `0` | Reserved   |
  |   `1` | Activate   |
  |   `2` | Deactivate |
  |   `3` | Measure    |

- <a name="value:calibration-Measurement-Element"></a>`Calibration Measurement Element`:

  | Value | Meaning                                                               |
  | ----: | --------------------------------------------------------------------- |
  |   `0` | Acceleration                                                          |
  |   `1` | Temperature (for $V_{REF}=1.25~V$ the temperature is returned in m°C) |
  |  `32` | Voltage                                                               |
  |  `96` | VSS (Ground)                                                          |
  |  `97` | VDD (Supply)                                                          |
  |  `98` | Regulated Internal Power                                              |
  |  `99` | Operation Amplifier Output                                            |

- <a name="value:Dimension"></a>`Dimension`:

  | Value | Meaning          |
  | ----: | ---------------- |
  |   `0` | Reserved         |
  |   `1` | 1. Dimension (x) |
  |   `2` | 2. Dimension (y) |
  |   `3` | 3. Dimension (z) |

#### Payload

|                       Byte 1                        |                                                   |       |               |
| :-------------------------------------------------: | :-----------------------------------------------: | :---: | :-----------: |
|                        Bit 7                        |                   Bit 6 - Bit 5                   | Bit 4 | Bit 3 - Bit 0 |
| [`Calibration Get/Set`](#value:calibration-Get-Set) | [`Calibration Method`](#value:calibration-Method) | Reset |   Reserved    |

|                                   Byte 2                                    |
| :-------------------------------------------------------------------------: |
| [`Calibration Measurement Element`](#value:calibration-Measurement-Element) |

|             Byte 3              |
| :-----------------------------: |
| [`Dimension`](#value:Dimension) |

|  Byte 4  |
| :------: |
| Reserved |

| Byte 5 - Byte 8 |
| :-------------: |
|    Reserved     |

#### Acknowledgment Payload

| Byte 1 - Byte 4 |
| :-------------: |
| Same as Payload |

| Byte 5 - Byte 8 |
| :-------------: |
|     Result      |

<a name="command:HMI-Configuration"></a>

### Command `HMI Configuration`

#### Values

- <a name="value:Get-Set-Sampling-Rate"></a>`Get/Set Sampling Rate`:

  | Value | Meaning           |
  | ----: | ----------------- |
  |   `0` | Get Sampling Rate |
  |   `1` | Set Sampling Rate |

- <a name="value:LED"></a>`LED`:

  | Value | Meaning  |
  | ----: | -------- |
  |   `0` | Reserved |
  |   `1` | LED      |

- <a name="value:ON-OFF"></a>`ON/OFF`:

  | Value | Meaning          |
  | ----: | ---------------- |
  |   `0` | Reserved         |
  |   `1` | On (Reset value) |
  |   `2` | Off              |

#### Payload

|                         Byte 1                          |                     |
| :-----------------------------------------------------: | :-----------------: |
|                          Bit 7                          |    Bit 6 - Bit 0    |
| [`Get/Set Sampling Rate`](#value:Get-Set-Sampling-Rate) | [`LED`](#value:LED) |

|     Byte 2     |
| :------------: |
| Number (0-255) |

|          Byte 3           |
| :-----------------------: |
| [`ON/OFF`](#value:ON-OFF) |

|  Byte 4  |
| :------: |
| Reserved |

| Byte 5 - Byte 8 |
| :-------------: |
|    Reserved     |

#### Acknowledgment Payload

- Same structure as payload

<a name="block:EEPROM"></a>

## Block `EEPROM`

| Number | Block Command                                                     | Access | Permanently Stored |
| -----: | ----------------------------------------------------------------- | ------ | ------------------ |
| `0x00` | [EEPROM Read](#command:EEPROM-Read)                               | Read   | x                  |
| `0x01` | [EEPROM Write](#command:EEPROM-Write)                             | Write  | x                  |
| `0x20` | [Read Write Request Counter](#command:Read-Write-Request-Counter) | Read   | x                  |

<a name="command:EEPROM-Read"></a>

### Command `EEPROM Read`

#### Notes

- Used to read data from EEPROM directly

#### Payload

| Byte 1 | Byte 2 | Byte 3 | Byte 4 - Byte 8 |
| :----: | :----: | :----: | :-------------: |
|  Page  | Offset | Length |    Reserved     |

#### Acknowledgment Payload

| Byte 1 | Byte 2 | Byte 3 |  Byte 4  | Byte 5 - Byte 8 |
| :----: | :----: | :----: | :------: | :-------------: |
|  Page  | Offset | Length | Reserved |      Data       |

<a name="command:EEPROM-Write"></a>

### Command `EEPROM Write`

#### Notes

- Used to write data to EEPROM directly
- You are not allowed to change all values, if the EEPROM is locked ([byte `0` is set to value `0xca`](https://github.com/MyTooliT/EEPROM/blob/master/EEPROM.md))

#### Payload

| Byte 1 | Byte 2 | Byte 3 |  Byte 4  | Byte 5 - Byte 8 |
| :----: | :----: | :----: | :------: | :-------------: |
|  Page  | Offset | Length | Reserved |      Data       |

#### Acknowledgment Payload

- Same structure as payload

<a name="command:Read-Write-Request-Counter"></a>

### Command `Read Write Request Counter`

#### Notes

- The current documentation of this command is based on the (old) code of ICOc

#### Payload

| Byte 1 - Byte 8 |
| :-------------: |
|        0        |

#### Acknowledgment Payload

| Byte 1 - Byte 4 |    Byte 5 - Byte 8    |
| :-------------: | :-------------------: |
|    Undefined    | EEPROM Write Requests |

<a name="block:ProductionData-and-RFID"></a>

## Block `Product Data and RFID`

| Number          | Block Command                                                           | Access | Permanently Stored |
| --------------- | ----------------------------------------------------------------------- | ------ | ------------------ |
| `0x00`          | [Global Trade Identification Number (GTIN)](#command:GTIN)              | Read   | x                  |
| `0x01`          | [Hardware Version](#command:Hardware-Version)                           | Read   | x                  |
| `0x02`          | [Firmware Version](#command:Firmware-Version)                           | Read   | x                  |
| `0x03`          | [Release Name](#command:Release-Name)                                   | Read   | x                  |
| `0x04` - `0x07` | [Serial Number 1-4](#command:Serial-Number)                             | Read   | x                  |
| `0x08` - `0x17` | [Product Name 1-16](#command:Product-Name)                              | Read   | x                  |
| `0x18` - `0x1F` | [OEM Free Use 0-7](#command:OEM-Free-Use)                               | Read   | x                  |
| `0x80`          | [Tool RFID product information](#command:Tool-RFID-product-information) | Read   | -                  |

<a name="command:GTIN"></a>

### Command `Global Trade Identification Number (GTIN)`

#### Acknowledgment Payload

| Byte 1 (MSB) – Byte 8 (LSB) |
| :-------------------------: |
|    GTIN (`unsigned int`)    |

<a name="command:Hardware-Version"></a>

### Command `Hardware Version`

#### Acknowledgment Payload

| Byte 1 – Byte 5 |    Byte 6     |    Byte 7     |    Byte 8     |
| :-------------: | :-----------: | :-----------: | :-----------: |
|    Reserved     | Major Version | Minor Version | Patch Version |

<a name="command:Firmware-Version"></a>

### Command `Firmware Version`

#### Acknowledgment Payload

| Byte 1 – Byte 5 |    Byte 6     |    Byte 7     |    Byte 8     |
| :-------------: | :-----------: | :-----------: | :-----------: |
|    Reserved     | Major Version | Minor Version | Patch Version |

<a name="command:Release-Name"></a>

### Command `Release Name`

#### Acknowledgment Payload

|        Byte 1 – Byte 8        |
| :---------------------------: |
| Firmware Release Name (ASCII) |

- `NULL` terminated or 8 bytes long

<a name="command:Serial-Number"></a>

### Command `Serial Number`

#### Notes

| Number | Purpose                          |
| ------ | -------------------------------- |
| `0x04` | Get first part of serial number  |
| `0x05` | Get second part of serial number |
| `0x06` | Get third part of serial number  |
| `0x07` | Get last part of serial number   |

#### Acknowledgment Payload

- UTF-8 string (8 bytes for each part)
- The whole serial number is a concatenation of its parts starting with the first part of the serial number

<a name="command:Product-Name"></a>

### Command `Product Name`

#### Notes

- Multiple Strings in different languages possible

| Command | Purpose                      |
| ------- | ---------------------------- |
| `0x08`  | Get 1. part of product name  |
| `0x09`  | Get 2. part of product name  |
| `0x0A`  | Get 3. part of product name  |
| `0x0B`  | Get 4. part of product name  |
| `0x0C`  | Get 5. part of product name  |
| `0x0D`  | Get 6. part of product name  |
| `0x0E`  | Get 7. part of product name  |
| `0x0F`  | Get 8. part of product name  |
| `0x10`  | Get 9. part of product name  |
| `0x11`  | Get 10. part of product name |
| `0x12`  | Get 11. part of product name |
| `0x13`  | Get 12. part of product name |
| `0x14`  | Get 13. part of product name |
| `0x15`  | Get 14. part of product name |
| `0x16`  | Get 15. part of product name |
| `0x17`  | Get 16. part of product name |

#### Acknowledgment Payload

- UTF-8 string (8 bytes)

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

- 8 bytes for each block command

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
| ----: | -------- |
|     0 | Reserved |
|     1 | Line     |
|     2 | Ramp     |

##### Byte 2:

Module (Module specific)

##### Byte 3-8:

Module specific

#### Acknowledgment Payload

##### Byte 1:

| Value | Meaning  |
| ----: | -------- |
|     0 | Reserved |
|     1 | Line     |
|     2 | Ramp     |

##### Byte 2-3:

Module (Module specific)

##### Byte 4-8:

Module specific

## Errors

| Value | Description                     | Example                                    |
| ----: | ------------------------------- | ------------------------------------------ |
|   `0` | Specific Error                  |                                            |
|   `1` | Not available                   |                                            |
|   `2` | General Error                   |                                            |
|   `3` | Write not allowed               | Setting of memory area in word not allowed |
|   `4` | Unsupported format              | 64 Byte Data via CAN2.0 is not possible    |
|   `5` | Wrong key/magic number          |                                            |
|   `6` | No SuperFrame inside SuperFrame |                                            |
|   `7` | EEPROM defect                   |                                            |
