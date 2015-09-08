# Using NodeMCU on ESP8266 #

## Dependencies ##

I'm working on linux, but these tools are cross-platform, so they may (or may not, idk) work on Windows.

- Python
- A serial monitor (I use the one in the Arduino IDE http://www.arduino.cc/en/Main/Software)
- A serial adapter
- A level converter, if the serial adapter is not 3.3V

## Wiring the Serial Connection ##

You'll need a 3.3V serial connection. 5v serial risks damage the ESP-8266, so use a 3.3V device or a level converter. I use SparkFun's 3.3V FTDI breakout (https://www.sparkfun.com/products/9873).

If using a serial converter like mine, you will also need a separate 3.3V power supply for the ESP-8266. It requires more current than most serial converters will supply.

Wiring (pinouts for some ESP-8266 versions at https://github.com/esp8266/esp8266-wiki/wiki/Hardware_versions):
- Serial converter ground, ESP-8266 ground, and power supply ground should all be connected
- ESP-8266 3.3V to power supply 3.3V
- ESP-8266 Tx and Rx to serial converter Tx and Rx. The labeling varies. For my converter, connect Tx to Rx and Rx to Tx. Connecting them backwards won't do any damage, so try both ways if unsure
- ESP-8266 reset and chip enable to power supply 3.3V
- Only when flashing firmware, connect ESP-8266 GPIO 0 to ground

Make sure to power on the serial converter BEFORE the ESP-8266 chip.

## Connecting to AT Firmware (optional) ##

Most ESP-8266 chips come with firmware that responds to AT commands. Some come with other firmware. Some come with firmware that prints an error message and then does nothing else. If you want to try out the AT commands or see what firmware your chip came with, you can play 'find the serial settings':

ESP-8266 default baud rates can be  9600, 19200, 57600, or 115200 depending on what firmware version they come with. Line ending may be CR, LF, or both.

Power on the ESP-8266. It will print a message stating the current firmware version by serial. The message will be in plain text if your serial settings match, or random ascii characters if they don't.

If the serial connection doesn't work, try checking /dev so see if a new tty device is added when plugging in the serial converter, and lsusb to check if a USB device is recognized. The Tx and Rx connections may need to be reversed.

Once the serial connection is working, sending 'AT' to the ESP-8266 should get an 'OK' response. For a full list of AT commands see http://wiki.iteadstudio.com/ESP8266_Serial_WIFI_Module#AT_Commands

## Building NodeMCU ##

TODO

First get the latest NodeMCU binary, available from https://github.com/nodemcu/nodemcu-firmware

    wget https://github.com/nodemcu/nodemcu-firmware/raw/master/pre_build/latest/nodemcu_latest.bin

TODO: Document new NodeMCU location.build process

## Flashing with esptool

Get the esptool Python-based flasher

    git clone https://github.com/themadinventor/esptool
    python esptool/setup.py build
    sudo python esptool/setup.py install

Connect GPIO 0 to gound and restart the ESP-8266 to enter flashing mode.

Flash with esptool

    sudo python esptool/esptool.py --port /dev/ttyUSB0 --baud 9600 write_flash 0x00000 nodemcu_latest.bin

This is set to run rather slow to reduce the likelyhood of errors. It may take a few minutes to flash, during which you should see the ESP-8266 serial transmit LED flash every few seconds.

After the esptool command exits, disconnect GPIO 0 from ground and restart to go back to run mode.

With NodeMCU on the ESP-8266, it will run a lua interpreter, and execute lua commands sent to it over serial. For full details, see NodeMCU's Github (https://github.com/nodemcu/nodemcu-firmware) or docs (http://www.nodemcu.com/docs/).

The interpreter can be accessed over serial with 9600 baud rate and CR+LF line ending. To see it do something basic, run `print(2+2)`

On startup, NodeMCU checks its internal memory for a file named init.lua and runs it if found. There are lua commands for writing this file without needing to reflash the firmware, and the luatool utility offers a streamlined way to do this.

## Copy .lua Files With luatool ##

Fetch luatool from GitHub

    git clone https://github.com/4refr0nt/luatool

Then disconnect the serial monitor, and run

    python luatool/luatool/luatool.py --port /dev/ttyUSB0 --src your_lua_file.lua --dest your_lua_file.lua --verbose

To load a new lua file as init.lua, which will run automatically when the ESP-8266 starts

    python luatool/luatool/luatool.py --port /dev/ttyUSB0 --src your_lua_file.lua --dest init.lua --verbose

The `--restart` option cause luatool to restart the ESP-8266 after transferring a file. A NodeMCU system can also be restarted by sending `node.restart()` over serial.

## Wifi LED Example ##

The example .lua file in this repo creates a network, and runs a simple HTTP server that allows controlling a connected device over Wifi.

Download the example script

    wget https://raw.githubusercontent.com/Densaugeo/ESP-8266-NodeMCU-Tutorial/master/wifi_led.lua

Connect an LED to GPIO 2. On my ESP-8266, this is pin 4 in NodeMCU, but this may vary from one device to another. You can test whether a pin number is correct by sending `gpio.mode(4, gpio.OUTPUT)` and `gpio.write(4, gpio.LOW)` or `gpio.write(4, gpio.HIGH)` or serial to control pin 4, for example. To use different pin numbers, edit the example script.

Disconnect the serial monitor, then load the script and reboot. Loading a lua script is not flashing, and should be done in regular run mode (GPIO 0 not grounded)

    python luatool/luatool/luatool.py --port /dev/ttyUSB0 --src wifi_led.lua --dest init.lua --verbose -- restart

Connect to the 'ESP-test' wifi network (password: ESP-test) and open http://192.168.4.1/

    firefox 192.168.4.1

Clicking the button should toggle GPIO 2.
