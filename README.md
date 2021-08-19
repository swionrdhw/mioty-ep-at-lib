<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://git.swissphone.com/libraries/cpp/mioty-ep-miotyduino20/-/tree/master">
    <img src="https://radiocrafts.com/wp-content/uploads/2020/04/mioty-logo-617x207.png" alt="Logo" width="330" height="120">
  </a>

  <h3 align="center">mioty_ep_at_lib</h3>

  <p align="center">
    MIOTY endpoint Library for AT-Protocol over a serial connection (USART)
  </p>
</p>



<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
        <li><a href="#open-example">Open Example</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#example">Example</a></li>
    <li><a href="#application-layer-payload-format">Application layer payload format</a></li>
    <li><a href="#MIOTY-EP-Class">MIOTY-EP Class</a></li>
      <ul>
        <li><a href="#firmware-update-mode">Firmware update mode</a>
        <li><a href="#object-constructor">Object constructor</a>
            <ul>
                <li><a href="#eui">EUI (MAC)</a>
                <li><a href="#shortaddress">Short Address</a>
                <li><a href="#networkkey">Network key</a>
            </ul>
        </li>
        <li><a href="#states">States</a>
            <ul>
                <li><a href="#busy">Busy</a>
                <li><a href="#alive">Alive</a>
            </ul>
        </li>
        <li><a href="#senduplink-data-or-text">Send/uplink data or text</a>
            <ul>
                <li><a href="#send-text">Send text</a>
                <li><a href="#send-bytes">Send Bytes</a>
                <li><a href="#send-in-background">Send in background</a>
            </ul>   
        </li>
        <li><a href="#send-and-receive-data-or-text">Send and receive data or text</a>
            <ul>
                <li><a href="#send-text">Send text</a>
                <li><a href="#send-bytes">Send Bytes</a>
                <li><a href="#send-in-background">Send in background</a>
            </ul>   
        </li>
        <li><a href=#hex-string-to-string>Hex-string to string</li>
        <li><a href="#structures-and-enums">Structures and Enums</a>
            <ul>
                <li><a href="#miotyreturncode">MIOTYReturnCode</a>
                <li><a href="#reply">Reply</a>
            </ul>   
         </li>
      </ul>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
---
## About The Project
---
The purpose of this library is to provide an arduino-usable functions for a AT-command controlled mioty-endpoint, which is connected over a USART-interface.

The AT-command interface is based on [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.0.9.pdf). 

It provides functions to setup the mioty-endpoint and to perform uplinks and downlinks.

Further more it transforms error messages from the mioty-endpoint in to a human readable format.
If a function of the mioty-endpoint is needed but not implemented in this 
class, it's possible to send the mioty-endpoint a raw serial message using it as a HardwareSerial-device (Arduino Class representation of a USART interface).

This library is based on Arduino standard functions.

### Built With
#### Software
* Arduino IDE
* C++

#### Communication
* USART
* [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.0.9.pdf)

#### Others
* [Coding guidelines](docs/Coding-Richtlinien-fuer-Arduino-MIOTY.docx)



<!-- GETTING STARTED -->

---
## Getting Started
---
To get a local copy up and running follow these simple steps.

### Prerequisites
1. Make sure the [Arduino IDE](https://www.arduino.cc/en/software) is installed.


### Installation

1. Download this repo as a .ZIP
2. Open the Arduino IDE
3. [Select from Menu](https://www.arduino.cc/en/guide/libraries): 
    - Sketch > Include Library > Add .ZIP Library...
    - Browse to the download path
    - Press open

Now it's ready to include in your library path.

### Open Example
1. To open an example, open File>Examples>Mioty-EP-AT-Lib
2. Select the example you'd like to open.

---
## Usage
---
With this library you're able to create an Object representation of your mioty-endpoint.
The object can be used to send and receive Text or Byte-Arrays.
Furthermore it provides functions to watch the states the endpoint is in.

to use the library in your sketch, include "mioty_ep_at.h".

In general, if a function returns a bool, a value of false represent always an error. 

---
## Example
---
We will go through a short example of how to use this library.

First the Library has to be included into the sketch and create an instance of MIOTY_EP created.

```cpp
#include "mioty_ep_at.h"

MIOTY_EP myMIOTY = MIOTY_EP(<Rx_pin>,<Tx_pin>);
```

Inside the setup function, we're checking if the mioty-endpoint is correctly to the board connected and print some information.

If the connection was successful send a "MIOTY" String as a "Hello World".

To print this information we initialize the Serial connection to the PC.
```cpp
    //start the Serial connection to the PC
    Serial.begin(9600);
    
    //check the connection
    if(myMIOTY.isAlive()) {
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
For the check if the transmission transmission was successful, it's also possible to check with the [*returnCode*](#reply) member for an [*MIOTYRETURN_CODE_OK*](#miotyreturncode). This method is most practically used if some type of error me be expected.

```cpp
if(myMIOTY.getReply().returnCode)
```
instead of

```cpp
if(myMIOTY.getReply().success)
```

## Application layer payload format
---
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

With this payload format it's also possible to send different data formats on each uplink. The uplink format gets identified with the <code style="json">"id":</code> tag of the json object.

So it's possible for example to send all 5 minutes a message with all sensor-data and the Application layer payload format id = 0. And all 15 minutes a massage with an additional battery value and the Application layer payload format id = 192

***Note***: the Application layer payload format id is divided in two ranges, where 0 to 191 is reserved and the range between 192-255 is for custom Application layer payload format.

## Firmware update mode
The mioty module can be updated over the USART interface.
For this functionality are two functions provided.

`enterFirmwareUpdateMode` and `exitFirmwareUpdateMode`.

the enter function is used to reboot the mioty module into bootloader and the exit mode is used to exit the bootloader safely.

(At the moment the firmware update mode will be exited with the baudrate 9600 enabled and the saveboot pin configured as input)

***Note***: the bootloader starts with its own baudrate. May consult the [user guide](xxxxxx) for further information.

```cpp
enterFirmwareUpdateMode(int saveboot_pin, int reset_pin)
```
**Takes:**
- `saveboot_pin`: Save Boot pin number.
- `reset_pin`: Reset pin number.

```cpp
exitFirmwareUpdateMode(int reset_pin)
```
**Takes:**
- `reset_pin`: Reset pin number.

## MIOTY EP Class
---
### Object constructor

Every mioty-endpoint needs to have an EUI (MAC), a shorted 2-Byte version of the EUI and a network key. Often the mioty-endpoint has these parameters already preprogrammed.

```cpp
MIOTY_EP(int Rx = PC4, int Tx = PC5, long baudrate=9600)
```
The EUI, short address and network key can be overwritten in separate functions.

**Takes:**

- `Rx`: UART receiver pin-number (e.g. PC5 on the MIOTYDuino20)

- `Tx`: UART transmission pin-number (e.g. PC4 on the MIOTYDuino20)

- `baudrate`: UART communication speed (default 9600)

Or create the mioty-endpoint object with this constructor to set user defined EUI, short address and network key:

```cpp
MIOTY_EP(String EUI, String ShortAddr,String NetworkKey,int Rx,int Tx, long baudrate = 9600)
```
Where the EUI, short-address and the network-key string are set on creation of the object. The sending and receiving pin also have to be given.

**Takes:**
- `EUI`: A string with the EUI-64 (e.g. "5C335CFB929599F2")

- `ShortAddr`: A string with the 2-byte form of EUI (e.g. "99F2")

- `NetworkKey`  16-byte Access key to Network ("918f5657c391ec20aa087c1df34ebab8")

- `Rx`: UART receiver pin-number (e.g. PC5 on the MIOTYDuino20)

- `Tx`: UART transmission pin-number (e.g. PC4 on the MIOTYDuino20)

- `baudrate`: UART communication speed (default 9600)

Note: The baudrate represents the baudrate of the mioty-endpoint. it does not change the baudrate of the endpoint.

It's possible to set/get the EUI and shortaddress manually by calling the set/get functions:

<!----- SET EUI ----->
#### EUI
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
#### Shortaddress
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
#### Networkkey
---
for safety reason it's not possible to request the network key.

Set networkkey:

```cpp
bool setNetworkKey(String NetworkKey)
```
**Takes:**
- `NetworkKey`: String with the network key in hex format (e.g. "5C335CCB959399F2").

**Returns:**
- `true`: When the network key successfully has been set.
- `false`: If error occurred

To get more Information when an Error occurred, use
<a href="#reply">getReply</a>.


### States
---
The MIOTY_EP object has two internal state variables: busy and alive.

#### Busy:
The endpoint is marked as **busy** if it is transmitting or receiving something and is unable to take another command. This state can be requested:
```cpp
bool isBusy(void)
```
**Returns:**
- `true`: if the endpoint is busy
- `false`: otherwise.

#### Alive:
The MIOTY_EP object does a small test if the communication to the mioty-endpoint is working. If the USART connection was successfully, the mioty-endpoint gets marked as alive.

This state can be queried with the function:
```cpp
bool isAlive(void)
```
**Return:**
- `true`: if the serial-communication was successfully.
- `false`: if something went wrong.

Note: for further information about the error, call <a href="#reply">getReply</a>.



<!---- Send --->

---
## Send/uplink data or text
---
This library allows 2 different types of sending (uplink).
* In foreground > blocking function
* In background > non-blocking function

Sending something in foreground means to pause/blocks the program until the sending is finished. this takes about **~3.5 seconds**, this can take longer if the message is big.

Internally the timeout (in ms), is calculated as:

$(1+0.034)*164*[number of bytes]+1349$

The sending function accepts two variable types (due to overloading):
* For a String like "MIOTY"
* For a byte array like {0x4B,120,55}

### Send Text
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

### Send Bytes
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

### Send in background
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

**or**
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

## Send and receive data or text
---
It's possible to uplink data or text and wait for a downlink message from the Base station.

This function doesn't have a dynamic timeout, because the AT-Command has it's own timeout

Like the uplink functionality does it allow 2 different types of sending and receiving (downlink).
* In foreground > blocking function
* In background > non-blocking function

Sending a downlink request in foreground means to pause/blocks the program until the sending and receiving is finished. this takes about **~9.5 seconds**, this can take longer if the message is big.

A downlink message can be interpreted as a Text or as A raw hex-string.
For that reason, all the send and receive functions do have the optional parameter `replyIsText` which tell if the received hex-string should be converted to normal text string with the [HexStringToString](#hex-string-to-string) function.

If a downlink with a specific format is expected, [getReply()](#reply) can be called and the format attribute extracted.

### Send text or bytes
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

### Send text or bytes in background
---
If the transmission and receiving should run while the main-program does something else, these functions can be used.

After the command is sent to the mioty-endpoint, the endpoint will be marked as busy. Unlike the uplink in background, with this function will keep the mioty module busy until a downlink message has been received or the mioty module timed out.

After the mioty module is not marked as busy anymore, the received message and further information's can be read with the [getReply](#Reply) function.

***Note:*** if the received hex string is a text, it has to be converted manually with the the [HexStringToString](#hex-string-to-string) function.

```cpp
bool sendAndReceiveInBackground(String msg, uint8_t MPFid = 0)
```
**Takes:**
- `msg`: String with the message for the Station
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**
- `bool`: True if the started transmission successfully or false otherwise.


```cpp
bool sendAndReceiveInBackground(uint8_t bytes[],uint8_t length, uint8_t MPFid = 0)
```
**Takes:**
- `bytes`: byte array (max. 245)
- `length`: length of byte array
- `MPFid`: Id of the format from the [Application layer payload format](#application-layer-payload-format) on the Base Station (default 0)

**Returns:**
- `bool`: True if the started transmission successfully or false otherwise.

## Hex String to String
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

## Structures and Enums
---
### MIOTYReturnCode
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

The description of the next 15 return codes are found in the [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.0.9.pdf).

To unite all errorcodes in one enum, the AT-Errors have an offset value of 17 to the description in the [Reference manual for mioty module m.YON](docs/Reference-manual-for-mioty-module-m.YON_Rev.0.9.pdf).

Additionally, the  *MIOTYRETURN_CODE_ATErr*  has been added for the case, if the MIOTY_EP object received a invalid AT-Error, which can be identified as an AT-Error.


### Reply
---
The library provides one Structure for the reply-message of the mioty-endpoint.

This has three members for the different information types of the reply message.

```cpp
struct MIOTYReply{
    bool  success = false;
    MIOTYReturnCode returnCode = MIOTYRETURN_CODE_ERR;
    String  message = "";
    byte format = 0;
};
```
* bool **`success`**: tells if the AT-Command was successful. If thats not the case it is false.
* [MIOTYReturnCode](#MIOTYReturnCode) **`returnCode`**: numeric representation of the result.
* String **`message`**: the message of the reply from the mioty-endpoint. If the transmission was not successful, this string holds a human readable description of the error that was encountered. If an uplink has been successfully made, the message component carry's the value of the Package counter in decimal (e.g.: 1928).
* byte **`format`**: The downlink format id number if a Application layer payload format is used. (by default 0)

A received reply from a MIOTY_EP object could look like:
```cpp
MIOTYReply reply = myMIOTY_EP.uplink("<text-longer-than-122-characters>");
>> reply.message = "Uplink Message to long"
>> reply.success = false
>> reply.returnCode = MIOTYRETURN_CODE_UPLINK_MSG_TO_LONG
>> reply.format = 2
```

The purpose of the different members is to make it possible to check a reply in a fast way.

A possible use case could be:
```cpp
reply = getReply();
if(!reply.success){
    if(reply.returnCode != MIOTYRETURN_BUSY){
        //Show the problem to the user
        Serial.println(reply.message);
    }
}
```

<!-- LICENSE -->

---
## License
---

<!-- CONTACT -->

---
## Contact
---

Tendai Rondof - [tendai.rondof@swissphone.com](tendai.rondof@swissphone.com)

Christian Gwerder - [christian.gwerder@swissphone.com](christian.gwerder@swissphone.com)

Project Link: [https://git.swissphone.com/libraries/cpp/mioty-ep-miotyduino20/-/tree/master](https://git.swissphone.com/libraries/cpp/mioty-ep-miotyduino20/-/tree/master)


