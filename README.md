nodemcu d/l:
https://github.com/nodemcu/nodemcu-firmware
wget https://github.com/nodemcu/nodemcu-firmware/raw/master/pre_build/latest/nodemcu_latest.bin

esptool command:
https://github.com/themadinventor/esptool
sudo python esptool.py write_flash 0x00000 nodemcu_latest.bin --port /dev/ttyUSB0 --baud 9600

luatool command:
disconnect serial monitor, then...
python luatool.py  --port /dev/ttyUSB0 --src init.lua --dest init.lua --verbose # --restart
