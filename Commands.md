# Commands

## Blocks

| Block  | Short Description                                            | Extended Description                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `0x00` | [System](#block:system)                                      | System commands are used to modify/request the state of each unit (e.g. reset) or an the overall system state (e.g. transmission speed) |
| `0x04` | [Streaming](#block:streaming)                                | Streaming commands are used to transmit data streams, but may be also used for single requests. The super frame is also located in this block. |
| `0x08` | [Statistical Data and Quantity](#block:Statistical-Data-and-Quantity) | This command group is used to store statistical data that can be used for histograms such as operating time and the number of power on/off cycles |
| `0x28` | [Configuration](#block:Configuration)                        | This command block is used to set configuration data (e.g. you can set the sampling rate of acceleration data here). |
| `0x3D` | [EEPROM](#block:EEPROM)                                      | Used for writing and reading EEPROM data directly            |
| `0x3E` | ProductData and RFID                                         | Used to store product data like a serial number. Furthermore, this block provides access to RFID information that is supported via connected tools. |
| `0x3F` | Test                                                         | Test Config Page                                             |

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

<a name="block:Statistical-Data-and-Quantity"></a>

## Block `Statistical Data and Quantity`

| Number | Block Command                                                | Access | Permanently Stored |
| ------ | ------------------------------------------------------------ | ------ | ------------------ |
| `0x00` | [Power On Cycles, Power Off Cycles](#command:Power-On-Cycles-Power-Off-Cycles) | Read   | x                  |
| `0x01` | [Operating time](#command:Operating-time)                    | Read   | x                  |
| `0x02` | [Under Voltage Counter](#command:Under-Voltage-Counter)      | Read   | x                  |
| `0x03` | [Watchdog Reset Counter](#command:Watchdog-Reset-Counter)    | Read   | x                  |
| `0x04` | [Production Date](#command:Production-Date)                  | Read   | x                  |

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

| Byte 1 (MSB) - Byte 4 (LSB)                                  |
| ------------------------------------------------------------ |
| ASCII String of the Production Date in the format: yyyymmdd where y=year, m=month, d=day |

<a name="block:Configuration"></a>

## Block `Configuration`

| Number | Block Command                                                | Access     | Permanently Stored |
| ------ | ------------------------------------------------------------ | ---------- | ------------------ |
| `0x00` | [Get/Set Acceleration Configuration](#command:Get-Set-Acceleration-Configuration) | Read/Write | x                  |
| `0x60` | [Get/Set Calibration Factor k](#command:Get-Set-Calibration-Factor-k) | Read/Write | x                  |
| `0x61` | [Get/Set Calibration Factor d](#command:Get-Set-Calibration-Factor-d) | Read/Write | x                  |
| `0x62` | [Calibration Measurement](#command:Callibration-Measurement) | Read/Write | x                  |
| `0xC0` | [HMI Configuration](#command:HMI-Configuration)              | Read/Write | x                  |

<a name="command:Get-Set-Acceleration-Configuration"></a>

### Command `Get/Set Acceleration Configuration`

#### Notes

##### Sampling Rate

- CLOCK/((Prescaler+1)*(AcquisitionTime + 12+1) * OverSamplingRate )
- CLOCK=38400000 Hz

##### Prescaler

- Byte2

##### Acquisition Time

- Sample and Hold Time i.e. Time to charge capacitor that is cut off and meassured at digital quantisation
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

| Byte 4                                                       |
| ------------------------------------------------------------ |
| Power of over sampling rate e.g. 10->1024 OverSampling Rate, 0=no Over Sampling |

| Byte 5                              |
| ----------------------------------- |
| Reference: Voltage*20 e.g. 3.3V->66 |

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

| Byte 5 (MSB) - Byte 8 (LSB)                                  |
| ------------------------------------------------------------ |
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

| Byte 5 (MSB) - Byte 8 (LSB)                                  |
| ------------------------------------------------------------ |
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

  | Value | Meaning                                                      |
  | ----- | ------------------------------------------------------------ |
  | `0`   | Acceleration                                                 |
  | `1`   | Temperature (for VREF=1250 temperature gets calculated to m°C) |
  | `32`  | Voltage                                                      |
  | `96`  | VSS(Ground)                                                  |
  | `97`  | VDD (Supply)                                                 |
  | `98`  | Regulated Internal Power                                     |
  | `99`  | Operation Amplifier Output                                   |

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

| Byte 2                                                       |
| ------------------------------------------------------------ |
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