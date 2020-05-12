DSMR/ESMR telegram to MQTT pusher
=================================

Sketch to read data from the [P1 port] of DSMR compliant electricity meters.
It verifies the CRC16 of the telegram, greatly reducing the chance for errors.

The Rx on a DSMR compliant meter is an open collector output, which means that
the current must be supplied by the receiving device and that the resulting
output signal is inverted. In the sketch, the Rx is inverted using SoftwareSerial,
so no further hardware inversion is needed.

## Compatibility & Hardware

Written for and tested on Wemos D1 mini v2, should work on any ESP8266 derived device.

The RTS needs 5V input to be triggered. This means that if you use an ESP or other
microcontroller that works on 3.3V, you will need a level shifter or an RTL inverter
to control it.

I have designed a PCB to handle the hardware-level interfacing with the P1 port,
contact me if you want one or check out my [Pwuts/Metertrekker] repo.

[Pwuts/Metertrekker]: https://gitlab.com/Pwuts/Metertrekker

## How to use

Assuming you have the repository files:
1. copy `settings.example.h` -> `settings.h`
2. adjust device specific firmware settings, setting defaults (if you need to)
3. upload to your device
4. power up device, connect to its access point
5. configure
6. restart device
7. connect to meter

To reliably publish Influx lines to MQTT, the `MAX_PACKET_SIZE` constant in
`PubSubClient.h` must be set to at least 1024. When using PlatformIO, *this
will be taken care of by the included `platformio.ini`*, otherwise you must
edit the library file.

### Note for DSMR <4.0 users
Your meter does not include a CRC in its telegrams, this sketch will (currently)
not work with your meter. You can manually disable the verification code, but
keep in mind that you may get garbage data out of it and it may also crash,
since the firmware will not be able to compensate for transmission errors.

## Dependencies

* CRC16 library retrieved from [vinmenn/Crc16] on Github
* [ESP-WiFiSettings] library

## Developing / contributing

If you want to work on, derive from, or tinker with this firmware or its functionality,
the following snippet may come in handy:
```C++
/* Valid telegram for testing purposes */
char bufferIn[768] = "/ISK5\\2M550E-1012\r\n"
                     "\r\n"
                     "1-3:0.2.8(50)\r\n"
                     "0-0:1.0.0(190827155511S)\r\n"
                     "0-0:96.1.1(4D455445525F53455249414C235F484558)\r\n"
                     "1-0:1.8.1(000057.460*kWh)\r\n"
                     "1-0:1.8.2(000037.300*kWh)\r\n"
                     "1-0:2.8.1(000000.000*kWh)\r\n"
                     "1-0:2.8.2(000000.000*kWh)\r\n"
                     "0-0:96.14.0(0002)\r\n"
                     "1-0:1.7.0(00.498*kW)\r\n"
                     "1-0:2.7.0(00.000*kW)\r\n"
                     "0-0:96.7.21(00008)\r\n"
                     "0-0:96.7.9(00002)\r\n"
                     "1-0:99.97.0()\r\n"
                     "1-0:32.32.0(00005)\r\n"
                     "1-0:32.36.0(00001)\r\n"
                     "0-0:96.13.0()\r\n"
                     "1-0:32.7.0(235.4*V)\r\n"
                     "1-0:31.7.0(002*A)\r\n"
                     "1-0:21.7.0(00.454*kW)\r\n"
                     "1-0:22.7.0(00.000*kW)\r\n"
                     "0-1:24.1.0(003)\r\n"
                     "0-1:96.1.0(4D455445525F53455249414C235F484558)\r\n"
                     "0-1:24.2.1(190827155507S)(00004.380*m3)\r\n"
                     "!";
int readLength = strlen(bufferIn);
char receivedCRC[5] = "ECDF";
```
This telegram is ESMR 5.0 compliant and the given CRC16 is valid for this telegram.


[vinmenn/Crc16]: https://github.com/vinmenn/Crc16
[ESP-WiFiSettings]: https://platformio.org/lib/show/7251/ESP-WiFiSettings

[P1 port]: https://www.netbeheernederland.nl/_upload/Files/Slimme_meter_15_a727fce1f1.pdf
