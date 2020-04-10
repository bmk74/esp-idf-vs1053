# esp-idf-vs1053
VS1053 Driver for esp-idf.    
I ported from from [here](https://github.com/baldram/ESP_VS1053_Library).

# Software requirements
esp-idf ver4.1 or later.   

# Configure   
You have to set this config value with menuconfig.   
- CONFIG_ESP_WIFI_SSID   
- CONFIG_ESP_WIFI_PASSWORD   
- CONFIG_ESP_MAXIMUM_RETRY   
- CONFIG_GPIO_CS   
- CONFIG_GPIO_DCS   
- CONFIG_GPIO_DREQ   
- CONFIG_GPIO_RESET   
- CONFIG_VOLUME   
- CONFIG_SERVER_HOST   
- CONFIG_SERVER_PORT   
- CONFIG_SERVER_PATH   
```
git clone https://github.com/nopnop2002/esp-idf-vs1053
cd esp-idf-vs1053
make menuconfig
make flash
```

![config-1](https://user-images.githubusercontent.com/6020549/76663983-3415df00-65c6-11ea-93db-9ec83e2601df.jpg)

![config-2](https://user-images.githubusercontent.com/6020549/76910307-047e1400-68f1-11ea-8907-c7e9a085c0b2.jpg)

# Wireing  

# for Blue board(VS1003)   

|VS1003||ESP32|
|:-:|:-:|:-:|
|5V|--|VIN(*1)|
|DGND|--|GND|
|MISO|--|GPIO19|
|MOSI|--|GPIO23|
|SCK|--|GPIO18|
|DREQ|--|GPIO4(*2)|
|RST|--|EN(*2)|
|CS|--|GPIO5(*2)|
|DCS|--|GPIO16(*2)|

(*1) External power if no VIN Pin.   
(*2) You can change any GPIO using menuconfig.   

![vs1053-1](https://user-images.githubusercontent.com/6020549/76676544-0782b700-6608-11ea-8b79-6e77598f7b8e.JPG)

![vs1053-2](https://user-images.githubusercontent.com/6020549/76676546-09e51100-6608-11ea-8a78-104490b97406.JPG)


# for Red board(VS1053)   

|VS1053||ESP32|
|:-:|:-:|:-:|
|5V|--|VIN(*1)|
|GND|--|GND|
|CS|--|N/C|
|MISO|--|GPIO19|
|SI|--|GPIO23|
|SCK|--|GPIO18|
|XCS|--|GPIO5(*2)|
|XRESET|--|EN(*2)|
|XDCS|--|GPIO16(*2)|
|DREQ|--|GPIO4(*2)|

(*1) External power if no VIN Pin.   
(*2) You can change any GPIO using menuconfig.   

![vs1053-1](https://user-images.githubusercontent.com/6020549/78961282-e42a2980-7b2b-11ea-97d1-7fbf7317189a.JPG)

---

# About embedded metadata
SHOUTCast server can put a Metadata Chunk in the middle of StreamData.   
The Metadata Chunk contains song titles and radio station information.

## Request embedded metadata chunk
Include the following in the HTTP Request.
```
Icy-MetaData: 1
```

## Metadata chunk interval
The SHOUTcast server will notify the metadata interval below.
```
icy-metaint:45000
```
This means that embedded metadata is sent from the server every 45000 bytes.
```
 --------------------------------------------------------------------------------
 |<---45000Byte Stream data---><Metadata><---45000Byte Stream data---><Metadata>
 --------------------------------------------------------------------------------
```

## Metadata chunk format
[Here](https://stackoverflow.com/questions/44050266/get-info-from-streaming-radio) is a detailed explanation.

The very first byte of the metadata chunk tells us how long the metadata chunk is.   
However, most are 0.   
0 indicates that the metadata chunk is 0 blocks(=0 byte)
```
 --------------------------------------------------------------------------------
 |<---45000Byte Stream data---><0><---45000Byte Stream data---><0>
 --------------------------------------------------------------------------------
```

## Display Metadata
The detected Metadata is sent to the CONSOLE task via RingBuffer.   
By changing the CONSOLE task, the received Metadata can be displayed on an external monitor.   
These pages will be helpful.

- HD44780(GPIO)
https://github.com/UncleRus/esp-idf-lib/tree/master/examples/hd44780_gpio

- HD44780(I2C)
https://github.com/UncleRus/esp-idf-lib/tree/master/examples/hd44780_i2c

- SSD1306
https://github.com/nopnop2002/esp-idf-ssd1306

__Note__:   
SPI Interface cannot be used for this purpose.   
Because VS1053 occupies SPI.   

---

# About Transfer-Encoding: chunked
There is some radio station return [Transfer-Encoding: chunked].   
This is one of them.

```
host = "icecast.radiofrance.fr";
path = "/franceculture-lofi.mp3";
Port = 80;
```

Even if you request [Icy-MetaData: 1], there is no [Icy-metaint] in the responce.   
Chunks are padded periodically.   
Details of [Transfer-Encoding: chunked] is [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding).  
```
 -------------------------------------------------------------------------------------------------
 |<StreamSize><CR><LF><---Stream data---><CR><LF><StreamSize><CR><LF><---Stream data---><CR><LF>
 -------------------------------------------------------------------------------------------------

<StreamSize> is Hexadecimal string.

<1234> is 4660 bytes.
```

---

# About 128K bit rate radio station
In this example, radio stations larger than 128K bit rate cannot be played for a long time.   
Click [here](https://github.com/nopnop2002/esp-idf-vs1053/issues/2) for details.

