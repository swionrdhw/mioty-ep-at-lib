<!-- PROJECT LOGO -->
<br/>
<p align="center">
  <a href="https://github.com/swionrdhw/mioty-ep-at-lib-releases">
    <img src="mioty-logo-617x207.png" alt="Logo" width="330" height="120">
  </a>

  <h3 align="center">
  mioty_ep_at_lib</h3>

  <p align="center">
    MIOTY endpoint Library for AT-Protocol over a serial connection (USART)
  </p>
</p>

[comment]: # (Table of Contents can be created with the visual studio code plugin "Markdown All in One") 
[comment]: # (Use command "Create Table of Contents" to create table of contents)
[comment]: # (Use command "Add/Update section number" to automatically number all sections)

- [1. About The Project](#1-about-the-project)
  - [1.1. Built With](#11-built-with)
    - [1.1.1. Software](#111-software)
    - [1.1.2. Communication](#112-communication)
    - [1.1.3. Others](#113-others)
    - [1.1.4. Dependencies](#114-dependencies)
- [2. Getting Started](#2-getting-started)
  - [2.1. Prerequisites](#21-prerequisites)
  - [2.2. Installation](#22-installation)
  - [2.3. Open Example](#23-open-example)
- [3. Usage](#3-usage)
- [4. Example](#4-example)
- [5. Application layer payload format](#5-application-layer-payload-format)
- [6. MIOTY-EP Class](#6-mioty-ep-class)
  - [6.1. Object constructor](#61-object-constructor)
    - [6.1.1. Initialization](#611-initialization)
      - [6.1.1.1. EUI](#6111-eui)
      - [6.1.1.2. Shortaddress](#6112-shortaddress)
      - [6.1.1.3. Networkkey](#6113-networkkey)
  - [6.2. get mioty endpoint infos](#62-get-mioty-endpoint-infos)
  - [6.3. Reset](#63-reset)
  - [6.4. Firmware update mode](#64-firmware-update-mode)
    - [6.4.1. States](#641-states)
      - [6.4.1.1. Busy](#6411-busy)
      - [6.4.1.2. Check if currently alive](#6412-check-if-currently-alive)
      - [6.4.1.3. Marked as Alive](#6413-marked-as-alive)
      - [6.4.1.4. Attached](#6414-attached)
        - [6.4.1.4.1. Attach](#64141-attach)
        - [6.4.1.4.2. Detach](#64142-detach)
      - [6.4.1.5. Bootloader Active](#6415-bootloader-active)
  - [6.5. Uplink Counter](#65-uplink-counter)
  - [6.6. Uplink Powerlevel](#66-uplink-powerlevel)
  - [6.7. Uplink Mode](#67-uplink-mode)
    - [6.7.1. Set Mode](#671-set-mode)
    - [6.7.2. Get Mode](#672-get-mode)
  - [6.8. Send carrier wave](#68-send-carrier-wave)
  - [6.9. Send/uplink data or text](#69-senduplink-data-or-text)
    - [6.9.1. Send Text](#691-send-text)
    - [6.9.2. Send Bytes](#692-send-bytes)
    - [6.9.3. Send in background](#693-send-in-background)
  - [6.10. Send and receive data or text](#610-send-and-receive-data-or-text)
    - [6.10.1. Send text or bytes](#6101-send-text-or-bytes)
    - [6.10.2. Send text or bytes in background](#6102-send-text-or-bytes-in-background)
  - [6.11. Send uplink witch acknowledge](#611-send-uplink-witch-acknowledge)
    - [6.11.1. Send text](#6111-send-text)
    - [6.11.2. Send bytes](#6112-send-bytes)
  - [6.12. Hex String to String](#612-hex-string-to-string)
  - [6.13. Structures and Enums](#613-structures-and-enums)
    - [6.13.1. MIOTYReturnCode](#6131-miotyreturncode)
    - [6.13.2. Reply](#6132-reply)
- [7. License](#7-license)
- [8. Contact](#8-contact)

<!-- ABOUT THE PROJECT -->
---

# 1. About The Project

The purpose of this library is to provide arduino-usable functions for a AT-command controlled mioty-endpoint, which is connected over a USART-interface.

The AT-command interface is based on [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.1.0.pdf).

It provides functions to setup the mioty-endpoint and to perform uplinks/downlinks.

Further more it transforms error messages from the mioty-endpoint in to a human readable format.
If a function of the mioty-endpoint is needed but not implemented in this
library, it's possible to send a raw serial message to the mioty-endpoint, using it as a HardwareSerial-object (Arduino Class representation of a USART interface).

This library is based on Arduino standard syntax.

## 1.1. Built With

### 1.1.1. Software

- Arduino IDE

- C++

### 1.1.2. Communication

- USART
- [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.1.0.pdf)

### 1.1.3. Others

- [Coding guidelines](docs/Coding-Richtlinien-fuer-Arduino-MIOTY.docx)

### 1.1.4. Dependencies

- Arduino HardwareSerial

<!-- GETTING STARTED -->

---

# 2. Getting Started

To get a local copy up and running follow these simple steps.

## 2.1. Prerequisites

1. Make sure the [Arduino IDE](https://www.arduino.cc/en/software) is installed.

## 2.2. Installation

1. Download this repo as a .ZIP
2. Open the Arduino IDE
3. [Select from Menu](https://www.arduino.cc/en/guide/libraries):
    - Sketch > Include Library > Add .ZIP Library...
    - Browse to the download path
    - Press open

Now it's ready to include in your library path.

## 2.3. Open Example

1. To open an example, open File>Examples>Mioty-EP-AT-Lib
2. Select the example you'd like to open.

---

# 3. Usage

With this library you're able to create an object representation of your mioty-endpoint.
The object can be used to send and receive text or byte-arrays.
Furthermore it provides functions to watch the states the endpoint is in.

to use the library in your sketch, include **"mioty_ep_at.h"**.

In general, if a function returns a bool, a value of false represent always a negative result.

---

# 4. Example

We will go through a short example of how to use this library.

First the Library has to be included into the sketch and an instance of MIOTY_EP created.

```cpp
#include "mioty_ep_at.h"

MIOTY_EP myMIOTY = MIOTY_EP(<SAVE_BOOT_PIN>, <RESET_PIN>, <Rx_pin>, <Tx_pin>);
```

Inside the constructor function, we initialize the mioty-endpoint default values.
To print this information we initialize the Serial connection to the PC.

```cpp
    //start the Serial connection to the PC
    Serial.begin(9600);
```

The mioty endpoint can now be fully initialized with the init function. This can take up to ~3 seconds. This makes sure the endpoint is connected.

```cpp
    //initialize the endpoint
    myMIOTY.init();
```

After the initialization we print some information about the endpoint over the Serial connection.

```cpp
    //check the connection
    if(myMIOTY.wasAlive()) {
        Serial.println("mioty-endpoint alive");

        //show some information
        Serial.println(myMIOTY.getMiotyInfo());   //information's about the endpoint
        Serial.println(myMIOTY.getEUI());         //print the EUI to check
        Serial.println(myMIOTY.getShortAddress());//print the shortaddress

        //send a string to the station
        myMIOTY.uplink("MIOTY");
    }
    else{
        Serial.println("mioty-endpoint is not available");
    }
```

For the loop function we take advantage of the sendInBackground function, to do something else while the endpoint is sending.

```cpp
    /*if the mioty-endpoint is not busy anymore*/
    if(!myMIOTY.isBusy()){
        /*check if the transmission was successful*/
        if(!myMIOTY.getReply().success){
            /*print the error message*/
            Serial.println(myMIOTY.getReply().message);
        }

        /*try start a sending command*/
        if(myMIOTY.uplinkInBackground("MIOTY")){
            Serial.println("started sending");
        }
        else{
            /*if something went wrong*/
            Serial.println("something went wrong");
            /*print the error message*/
            Serial.println(myMIOTY.getReply().message);
        }
    }
    /*now you can do something else*/
```

For the check if the transmission transmission was successful, it's also possible to check with the [*returnCode*](#reply) member for an [*MIOTYRETURN_CODE_OK*](#miotyreturncode). This method is most practically used if some type of error may be expected.

```cpp
if(!myMIOTY.getReply().returnCode)
```

instead of

```cpp
if(myMIOTY.getReply().success)
```

# 5. Application layer payload format

From the Application Center exist an option to add a payload format which allows automatic processing of uplink data.

It is defined with an json object on the base station, for e.g.:

```json
{
  "version": "2.0",
  "typeEui": "5c335cf982b21f6b",
  "meta": {
      "name": "miotyduino wheater sensor temp_humidity_pressure_battery",
      "vendor": "Swissphone"
  },
  "component": {
      "battery": {
        "func": "$/256*1.3+3.0", 
        "littleEndian": false, 
        "size": 8, 
        "type": "uint", 
        "unit": "V"
      },
      "humidity": {
        "func": "", 
        "littleEndian": false, 
        "size": 8, 
        "type": "uint", 
        "unit": "%"
      },
      "pressure": {
        "func": "$/10", 
        "littleEndian": false, 
        "size": 16, 
        "type": "uint", 
        "unit": "hPa"
      },
      "temperature": {
        "func": "$/10-273.15", 
        "littleEndian": false, 
        "size": 16, 
        "type": "uint", 
        "unit": "°C"
      }
  },
  "uplink": [
      {
      "id": 0,
      "payload": [
          {"component": "temperature","name": "temperature"},
          {"component": "humidity","name": "humidity"},
          {"component": "pressure","name": "pressure"}
      ]
      },
      {
          "id": 192,
          "payload": [
              {"component": "temperature","name": "temperature"},
              {"component": "humidity","name": "humidity"},
              {"component": "pressure","name": "pressure"},
              {"component": "battery","name": "battery"}
          ]
      }
  ]
}
```

With this payload format it's also possible to send different data formats on each uplink. The uplink format gets identified with the <code style="json"> "id":</code> tag of the json object.

So it's possible for example to send all 5 minutes a message with all sensor-data and the Application layer payload format id = 0. And all 15 minutes a massage with an additional battery value and the Application layer payload format id = 192

***Note***: the Application layer payload format id is divided in two ranges, where 0 to 191 is reserved and the range between 192-255 is for custom Application layer payload format.

# 6. MIOTY-EP Class

## 6.1. Object constructor

---

Every mioty-endpoint needs to have an EUI, a shorted 2-Byte version of the EUI and a network key. Often the mioty-endpoint has these parameters already preprogrammed.

```cpp
MIOTY_EP(int saveboot_pin, int reset_pin, int Rx, int Tx)
```

**Takes:**

- `saveboot_pin`: Save boot pin of the module

- `reset_pin`: Reset pin of the module

- `Rx`: UART receiver pin-number (e.g. PC5 on the MIOTYDuino20)

- `Tx`: UART transmission pin-number (e.g. PC4 on the MIOTYDuino20)

### 6.1.1. Initialization

To fully initialize the mioty endpoint, the `init` method has to be called.
It checks if the mioty endpoint is stuck in bootloader or has boot up normally. Additionally it's possible to give a new EUI/networkkey/shortaddress to overwrite the old EUI/networkkey/shortaddress.

To change the networkkey separately, [setNetworkkey](#networkkey) have to be used.

If the given EUI and shortaddress matches with the saved EUI and shortaddress on the mioty endpoint, the given EUI/networkkey/shortaddress will be ignored and not set again.

This prevents a reset of the [Uplink Counter](#uplink-counter) after reinizialisation/reboot.

**Attention:** This will cause a reset of the uplink counter.

***Note:*** It doesn't matter if the EUI/networkkey/shortaddress is uppercase or not.

The given EUI/networkkey/shortaddress gets saved on the endpoints memory and will only change if a factory reset is done or gets overwritten again.

```cpp
void init(String EUI /*= ""*/, String ShortAddr /*= ""*/, 
          String NetworkKey /*= ""*/)
```

**Takes:**

- `EUI`: A string with the EUI-64 (e.g. "5C335CFB929599F2")

- `ShortAddr`: A string with the 2-byte form of EUI (e.g. "99F2")

- `NetworkKey`  16-byte Access key to Network ("918f5657c391ec20aa087c1df34ebab8")

It's possible to set/get the EUI and shortaddress manually by calling the set/get functions:

<!----- SET EUI ----->
#### 6.1.1.1. EUI

---
Get EUI:

```cpp
String getEUI(void)
```

**Returns:**

- `String` EUI in hex format (e.g. "5C335CCB959399F2").

Set EUI:

```cpp
bool setEUI(String EUI)
```

**Takes:**

- `EUI`: String with the EUI in hex format (e.g. "5C335CCB959399F2").

**Returns:**

- `true`: When the EUI successfully has been set.
- `false`: If error occurred

<!----- SET SHORTADDRESS ----->
#### 6.1.1.2. Shortaddress

---
Get shortaddress:

```cpp
String getShortAddress(void)
```

**Returns:**

- `String` shortaddress in hex format (e.g. "99F2").

Set shortaddress:

```cpp
bool setShortAddress(String shortAddress)
```

**Takes:**

- `shortAddress`: String with the EUI in hex format (e.g. "99F2").

**Returns:**

- `true`: When the short address successfully has been set.
- `false`: If error occurred

<!----- SET NETWORKKEY ----->
#### 6.1.1.3. Networkkey

---
for safety reason it's not possible to request the network key.

Set networkkey:

```cpp
bool setNetworkKey(String NetworkKey)
```

**Takes:**

- `NetworkKey`: String with the network key in hex format (e.g. "9ee5e6d365b04679e21bd34146aa5ea1").

**Returns:**

- `true`: When the network key successfully has been set.
- `false`: If error occurred

To get more Information when an Error occurred, use
<a href="#reply">getReply</a>.

## 6.2. get mioty endpoint infos

This function returns string with the information about the Hardware of the endpoint. Plus what firmware-version runs on it.

```cpp
String getMiotyInfo();
```

**Returns:**

- `string` : e.g. "Swissphone Wireless AG;m.YON;v1.0.0-0;EFR32FG14PXXXF256;Apr 30 2021;14:05:49"

To get more detailed information about the firmware, the function following can be used:

```cpp
String getFirmwareVersion();
```

**Returns:**

- `string` : e.g. "Fraunhofer IIS;MIOTY_EP_CORE_LIB_BIDI;v2.2.1-0;cortex-m4;Apr 23 2021;13:00:16"

## 6.3. Reset

Reboots the mioty endpoint. This process takes ca. 1.5s.

**The uplink counter, EUI, short address and networkkey won't be lost in this process.**

```cpp
void reset();
```

## 6.4. Firmware update mode

The mioty module can be updated over the USART interface.
For this functionality are two functions provided.

`enterFirmwareUpdateMode` and `exitFirmwareUpdateMode`.

the enter function is used to reboot the mioty module into bootloader and the exit mode is used to exit the bootloader safely.

***Note***: the bootloader starts with its own baudrate. May consult the [firmware-update user guide](https://www.swissphone.com/download-item/61943) for more information.

```cpp
void enterFirmwareUpdateMode()
```

```cpp
void exitFirmwareUpdateMode()
```

### 6.4.1. States

---
The MIOTY_EP object has two internal state variables: busy and alive.

#### 6.4.1.1. Busy

The endpoint is marked as **busy** if it is transmitting or receiving something and is unable to take another command. This state can be requested:

```cpp
bool isBusy(void)
```

**Returns:**

- `true`: if the endpoint is busy
- `false`: otherwise.

#### 6.4.1.2. Check if currently alive

---
This function sends an AT-Command over the USART to check if the endpoint is responding and marks the endpoint as alive. So, with this function, the internal alive-state, which is accessed also by [wasAlive](#marked-as-alive), is refreshed.

This function requires more time as the [wasAlive](#marked-as-alive) function.

```cpp
bool isAlive();
```

***Returns:***

- `true`: endpoint right now responding over USART.
- `false`: endpoint not responsive over USART.

#### 6.4.1.3. Marked as Alive

During the initialization of the mioty endpoint object or at calling the [isAlive](#check-if-currently-alive) method, it is tested if the mioty endpoint answers over USART. If the USART test was successfully, the mioty-endpoint gets marked as alive.
To check if the last test was successful, the internal alive-state can be accessed with:

```cpp
bool wasAlive(void)
```

**Return:**

- `true`: if the serial-communication was successfully.
- `false`: if something went wrong.

Note: This function does only return an internal bool state and **does not perform a communication** to the endpoint. To perform a new communication to test the connection, use [isAlive](#check-if-currently-alive)

#### 6.4.1.4. Attached

---
During usage this mioty endpoint object keeps track of the attachment state automatically.

To get the current attachment state the `isAttached` function can be used:

```cpp
bool isAttached();
```

***Returns:***

- `true`: if the endpoint is attached.
- `false`: if the endpoint is detached.

##### 6.4.1.4.1. Attach

---
Attaches the endpoint locally.

```cpp
bool attach();
```

***Return:***

- `true`: if the successfully attached.
- `false`: if an error occurred.

##### 6.4.1.4.2. Detach

---

Detaches the endpoint locally.

```cpp
bool detach();
```

***Return:***

- `true`: if the successfully detached.
- `false`: if an error occurred.

#### 6.4.1.5. Bootloader Active

---
This function tests if the mioty endpoint is in bootloader.

This function internally changes the Serial speed to the bootloader baudrate and tests if some 'C' are received.

```cpp
bool isBootloaderActive();
```

***Returns:***

- `true`:  if bootloader active.
- `false`: if bootloader not active.

<!---- Send --->

---

## 6.5. Uplink Counter

---
The endpoint does keep track of the uplink count by itself and gets only reset if the endpoint gets detached.
To query the current uplink counter the `getUplinkCounter` function can be used:

```cpp
long long getUplinkCounter();
```

***Returns:***

- `long long`: current count of uplinks since last attached.

---

## 6.6. Uplink Powerlevel

---
The uplink power can be set by this function.

The output power of the MIOTY_EP can not directly be set by a dBm value. It is set by an internal representation of the power level.

This function employs an approximation function to set the power-output in dBm.

This formula is used to convert the dBm value into the internal representation of the power level:
$internal\_powerlevel =  \lfloor e^{\frac{dBm++21.519}{8.5121}} \rfloor$

Therefor it yields a possibility for a divergence of the real output to the calculated value by +/-1dBm.

The default output power range of the mioty-endpoint is from -2dBm up to 18dBm.

If no output power is configured, the output power is by default by 18dBm.

```cpp
void setTransmissionPowerLevel(float dBm)
```

**Takes:**

- `dBm`: Desired output power value.

## 6.7. Uplink Mode

---
There are three uplink modes normal uplink mode, high reliability mode or high speed mode.

### 6.7.1. Set Mode

---
The uplink mode can be set by this function.

```cpp
bool setUplinkMode(UplinkMode UM);
```

**Takes:**

- `UplinkMode`:
  - UM_STANDARD -> nomal uplink mode
  - UM_RETRANSMISSION -> high reliability mode
  - UM_LOWLATENCY -> high speed mode

**Returns:**

- `true`: If the uplink mode successfully has been set.
- `false`: If error occurred, (to get the error-information call getReply())

### 6.7.2. Get Mode

---
The current uplink mode can be sedisplayed by this function.

```cpp
UplinkMode getUplinkMode();
```

**Returns:**

- `UM_STANDARD`: uplink mode is set to normal uplink mode
- `UM_RETRANSMISSION`: uplink mode is set to high reliability mode
- `UM_LOWLATENCY`: uplink mode is set to high speed mode
- `UM_ERROR`:  something went wrong

---

## 6.8. Send carrier wave

---
This function sends an unmodulated carrier wave at the mioty center frequency (868.18Mhz).

```cpp
void sendCarrierWave(bool state);
```

**Takes:**

- `true`: Starts sending carrier wave
- `false`: Stop sending carrier wave

---

## 6.9. Send/uplink data or text

---
This library allows 2 different types of sending (uplink).

- In foreground > blocking function
- In background > non-blocking function

Sending something in foreground means to pause/blocks the program until the sending is finished. this takes about **~3.5 seconds**, this can take longer if the message is big.

Internally the timeout (in ms), is calculated as:

$(1+0.034)*164*[number of bytes]+1349$

The sending function accepts two variable types (due to overloading):

- For a String like "MIOTY"
- For a byte array like {0x4B,120,55}

### 6.9.1. Send Text

---
The function for sending a text in the foreground (blocking) is as following:

```cpp
bool uplink(String text, uint8_t MPFid = 0)
```

**Takes:**

- `text`: String like "Hello MIOTY" (max. 122 characters)
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `true`: if the transmission was successful.
- `false`: if an error occurred.

Note: for further information about the error, call <a href="#reply">getReply</a>.

### 6.9.2. Send Bytes

---
If a transmission of values is needed (e.g. sensor data), this byte-array send function in foreground (blocking) can be used..

```cpp
bool uplink(uint8_t bytes[], uint8_t length, uint8_t MPFid = 0)
```

**Takes:**

- `bytes`: Array of max. 245 bytes (e.g. {0x4B,120,55})
- `length`: number of elements. May directly use <code> sizeof(bytes) </code>.
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `true`: if the transmission was successful.
- `false`: if an error occurred.

Note: for further information about the error, call <a href="#reply">getReply</a>.

### 6.9.3. Send in background

---
If the transmission should run while the main-program does something else, this functions can be used.

After the command is sent to the mioty-endpoint, the endpoint will be marked as busy.

To check if the endpoint is still busy call <a href="#busy">isBusy</a>.

```cpp
bool uplinkInBackground(String msg, uint8_t MPFid = 0)
```

**Takes:**

- `text`: String like "Hello MIOTY" (max. 122 characters)
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `true`: if the transmission started successful.
- `false`: if an error occurred.

***or***

```cpp
bool uplinkInBackground(uint8_t bytes[], uint8_t length, uint8_t MPFid = 0)
```

**Takes:**

- `bytes`: Array of max. 245 bytes (e.g. {0x4B,120,55})
- `length`: number of elements. May directly use <code> sizeof(bytes) </code>.
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `true`: if the transmission started successful.
- `false`: if an error occurred.

After the transmission of the mioty-endpoint is finished the result can be inspected via the <a href="#reply">getReply</a> function.

## 6.10. Send and receive data or text

---
It's possible to uplink data or text and wait for a downlink message from the Base station.

This function doesn't have a dynamic timeout, because the AT-Command has it's own timeout

Like the uplink functionality does it allow 2 different types of sending and receiving (downlink).

- In foreground > blocking function
- In background > non-blocking function

Sending a downlink request in foreground means to pause/blocks the program until the sending and receiving is finished. this takes about **~9.5 seconds**, this can take longer if the message is big.

A downlink message can be interpreted as a Text or as A raw hex-string.
For that reason, all the send and receive functions do have the optional parameter `replyIsText` which tell if the received hex-string should be converted to normal text string with the [HexStringToString](#hex-string-to-string) function.

If a downlink with a specific format is expected, [getReply()](#reply) can be called and the format attribute extracted.

### 6.10.1. Send text or bytes

---
The send and receive functions has the same overload like the uplink function.
so it is able to accept a String or a Byte-Array with it's length.

```cpp
String sendAndReceive(String text, uint8_t MPFid = 0, bool replyIsText = false)
```

**Takes:**

- `text`: String like "Hello MIOTY" (max. 122 characters)
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)
- `replyIsText` (default: false): A bool which tells if the receiving message has to be converted to text.

**Returns:**

- `String`: Received message

```cpp
String sendAndReceive(uint8_t bytes[],uint8_t length, uint8_t MPFid = 0, bool replyIsText = false)
```

**Takes:**

- `bytes`: Array of max. 245 bytes (e.g. {0x4B,120,55})
- `length`: number of elements. May directly use <code> sizeof(bytes) </code>.
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)
- `replyIsText` (default: false): A bool which tells if the receiving message has to be converted to text.

**Returns:**

- `String`: Received message (empty string if no message received)

### 6.10.2. Send text or bytes in background

---
If the transmission and receiving should run while the main-program does something else, these functions can be used.

After the command is sent to the mioty-endpoint, the endpoint will be marked as busy. this function will keep the mioty module marked as busy until a downlink message has been received or the mioty module timed out.

After the mioty module is not marked as busy anymore, the received message and further information's can be read with the [getReply](#reply) function.

***Note:*** if the received hex string is a text, it has to be converted manually with the the [HexStringToString](#hex-string-to-string) function.

```cpp
bool sendAndReceiveInBackground(String msg, uint8_t MPFid = 0)
```

**Takes:**

- `msg`: String with the message for the Station
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `bool`: True if the transmission started successfully and false otherwise.

```cpp
bool sendAndReceiveInBackground(uint8_t bytes[],uint8_t length, uint8_t MPFid = 0)
```

**Takes:**

- `bytes`: byte array (max. 245)
- `length`: length of byte array
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `bool`: True if the started transmission successfully or false otherwise.

---

## 6.11. Send uplink witch acknowledge

---
Send uplink with text or bytes with an acknowledge request. Wait for acknowledge till timeout runs out.

---

### 6.11.1. Send text

```cpp
bool uplinkWithAck(String msg="", uint8_t MPFid = 0);
```

**Takes:**

- `msg`: String with the message for the Station
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `true`: An acknowledge from basestation got received
- `false`: An no acknowledge got received

---

### 6.11.2. Send bytes

```cpp
bool uplinkWithAck(uint8_t bytes[], uint8_t length, uint8_t MPFid = 0);
```

**Takes:**

- `bytes`: byte array (max. 245)
- `length`: length of byte array. May directly use <code> sizeof(bytes) </code>.
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**

- `true`: An acknowledge from basestatio got received
- `false`: An no acknowledge got received

---

## 6.12. Hex String to String

---
All the text messages over mioty are in a Hex-String format.
This function converts hex-strings who are an encoded string into a text string.

(e.g. "4D494F5459"=>"MIOTY")

```cpp
String HexStringToString(String hexString)
```

**Takes:**

- `hexString`: String with ASCII hex values (e.g. "4D494F5459")

**Returns:**

- `String`: String with decoded text (e.g. "MIOTY")

## 6.13. Structures and Enums

---

### 6.13.1. MIOTYReturnCode

---
Every time the MIOTY_EP object analyses a message from the mioty-endpoint or tries to send a AT-command to the endpoint, it analyses if an error occurred.

As e result it saves a numerical representation of the problem it encountered inside the returning MIOTYReply struct. This value can be accessed by the member [returnCode](#reply).

The enum goes as follows:

```cpp
typedef enum {
    MIOTYRETURN_CODE_OK,
    MIOTYRETURN_CODE_MacError,
    MIOTYRETURN_CODE_MacFramingError,
    MIOTYRETURN_CODE_ArgumentSizeMismatch,
    MIOTYRETURN_CODE_ArgumentOOR,
    MIOTYRETURN_CODE_BufferSizeInsufficient,
    MIOTYRETURN_CODE_MacNodeNotAttached,
    MIOTYRETURN_CODE_MacNetworkKeyNotSet,
    MIOTYRETURN_CODE_MacAlreadyAttached,
    MIOTYRETURN_CODE_ERR, //not in protocol
    MIOTYRETURN_CODE_MacDownlinkNotAvailable,
    MIOTYRETURN_CODE_UplinkPackingErr,
    MIOTYRETURN_CODE_MacNoDownlinkReceived,
    MIOTYRETURN_CODE_MacOptionNotAllowed,
    MIOTYRETURN_CODE_MacDownlinkErr,
    MIOTYRETURN_CODE_MacDefaultsNotSet,
    MIOTYRETURN_CODE_ATErr, //start of AT errors
    MIOTYRETURN_CODE_ATgenericErr,
    MIOTYRETURN_CODE_ATCommandNotKnown,
    MIOTYRETURN_CODE_ATParamOOB, 
    MIOTYRETURN_CODE_ATDataSizeMismatch,
    MIOTYRETURN_CODE_ATUnexpectedChar,
    MIOTYRETURN_CODE_ATArgInvalid,
    MIOTYRETURN_CODE_ATReadFailed,
    MIOTYRETURN_BUSY
};
```

The first code *MIOTYRETURN_CODE_OK* means every thing was successfully and has the value 0.

The code *MIOTYRETURN_CODE_ERR* is used in the case off a program code error and in the case of an unknown error reply from the mioty-endpoint.

The description of the next 15 return codes are found in the [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.1.0.pdf).

To unite all errorcodes in one enum, the AT-Errors have an offset value of 17 to the description in the [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.1.0.pdf).

Additionally, the  *MIOTYRETURN_CODE_ATErr*  has been added for the case, if the MIOTY_EP object received a invalid AT-Error, which can be identified as an AT-Error.

### 6.13.2. Reply

---
The library provides one Structure for the reply-message of the mioty-endpoint.

>**Attention: The replay-message refers to the last received message from the MIOTY_EP. If an error happens before a message is received, the replay-message will contain the information from the previous successful request/command to the MIOTY_EP.**

This has three members for the different information types of the reply message.

```cpp
struct MIOTYReply{
    bool  success = false;
    MIOTYReturnCode returnCode = MIOTYRETURN_CODE_ERR;
    String  message = "";
    byte format = 0;
};
```

- bool **`success`**: tells if the AT-Command was successful. If thats not the case it is false.
- [MIOTYReturnCode](#miotyreturncode) **`returnCode`**: numeric representation of the result.
- String **`message`**: the message of the reply from the mioty-endpoint. If the transmission was not successful, this string holds a human readable description of the error that was encountered. If an uplink has been successfully made, the message component carry's the value of the Package counter in decimal (e.g.: 1928).
- byte **`format`**: The downlink format id number if a Application layer payload format is used. (by default 0)

A received reply from a MIOTY_EP object could look like:

```cpp
MIOTYReply reply = myMIOTY_EP.uplink("<text-longer-than-122-characters>");
>> reply.message = "Uplink Message to long"
>> reply.success = false
>> reply.returnCode = MIOTYRETURN_CODE_UPLINK_MSG_TO_LONG
>> reply.format = 0
```

The purpose of the different members is to make it possible to check a reply in a fast way.

A possible use case could be:

```cpp
reply = getReply();
if(!reply.success){
    if(reply.returnCode == MIOTYRETURN_CODE_UPLINK_MSG_TO_LONG){
        //Show the problem to the user
        Serial.println("Message to long");
    }
    else{
      Serial.println(reply.message);
    }
}
```

<!-- LICENSE -->

---

# 7. License

<!-- CONTACT -->

# 8. Contact

Mioty Team - [mioty@swissphone.com](mioty@swissphone.com)

Project Link: [https://github.com/swionrdhw/mioty-ep-at-lib-releases](https://github.com/swionrdhw/mioty-ep-at-lib-releases)
