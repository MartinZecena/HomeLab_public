# Hardware
## Main Hardware

- Laptop:
  - RAM 40GB DDR5
  - SSD 1TB (Storage)
  - SSD 500GB (DebianOS)
  - Intel i7 12700H
  - OS Debian GNU/Linux 12 (Bookworm) 

- pc1:
  - RAM 40GB
  - SSD 1TB
  - Intel i5 10500T
  - OS Proxmox V.E 8.4

- pc2:
  - RAM 16GB
  - SSD 500GB
  - Intel i3 10300T
  - OS Security Onion 2.4.141

- pc3:
  - RAM 16 GB 
  - SSD 256 GB
  - Intel N100
  - 5 ETH ports
  - OS OPNsense

- RaspberryPi 5:
  - RAM 8GB
  - 2x microSD 128 GB
  - OS Kali Linux arm64(red team)
  - OS Debian 13 arm6(blue team)

 
## Other Hardware
  - switch: TP-Link TL-SG108E - 8 port managed
  - access point: TP-Link TL-WA3001 
  - NAS: QNAP NAS TS-219P 2x4 TB RAID 1
  - ETH: CAT6 cables
  - adapters: wifi alpha for laptop and USB to ETH adapter for pc2 micro HDMI to HDMI
  - Sticker printer to label, ports, usbs, sd adapters power station


## Setup Specs.
RAM: 120GB
CORES: 28  
THREADS: 44  
AVG. CLOCK: 4GHz  
TOTAL STORAGE: 6.5 TB 
NAS: 35MB/s avg. write speed 
LAN: 990Mbps
WAN: 250Mbps  
ELECTRICITY: 520W

## atheros AWUS036NHA wifi card
- download the drivers
```
apt edit-sources
deb http://httpredir.debian.org/debian/ jessie main contrib non-free
deb-src http://httpredir.debian.org/debian/ jessie main contrib non-free
apt update && apt install firmware-atheros
```

## Lilygo (esp32-s3 ligo t-embeded esp32-s3 cc1101)
Field pentesting tool for zubG frequencies.

### install bruce 
 **we need file Bruce-lilygo-t-embed-cc1101.bin 
 - find devices name
```ls -l /dev/ttyUSB* /dev/ttyACM* 2>/dev/null
```
connect the device to the pc, desconnect it and shutdown
press the boot button (center button) and hold while connecting the usb
relese button after a few seconds of being connected
the device has no lights or soever it looks like nothing actually happened

change permissions
sudo chmod a+rw /dev/ttyACM4
sudo usermod -aG dialout $USER

flash device with binary firmware
esptool.py --chip esp32s3 --port /dev/ttyACM4 write_flash -z 0x0 ~/Downloads/Bruce-lilygo-t-embed-cc1101.bin

press reset button at the back of the pcb

## HackRF mayhem portapak 
- no docu available just isntalled the oficial repo of device on  video of how to config for reference
https://www.youtube.com/watch?v=xGR_PMD9LeU

## Modded raspyjack
- follow official configuration https://github.com/7h30th3r0n3/Raspyjack
- change background image 
 download .avif it was just the format of the file i liked and converted it to logo.bmp format with this simple python script
 ```python
from PIL import Image
import pillow_avif
img = Image.open("logo.avif")
img.save("logo.bmp")
```
  * change the size of the logo to the same size as your screen size with a program like imageMagick or similar
  * replace file with file in /root/Raspyjack/img/logo.bmp

- configure my screen
  * im using a different screen and button layout as the original project so some things need to be adjusted values were changed to files LCD_1in44.py and raspyjack.py to adjust the screen and menu sizes
  * find the pin layout of screen and adjust the pin values for buttons in raspyjack.py

- configure my wifi card alpha AWUS036ACS
```
lsusb //check if wifi card is connected to device
sudo apt install -y dkms git build-essential bc //install build tools
git clone https://github.com/aircrack-ng/rtl8812au.git //clone drivers from aircrack-ng repo
cd rtl8812au
sudo make dkms_install
```
- find rtl88xxau.ko file and move it to /lib/modules/6.12.25+rpt-rpi-v7/kernel/drivers/net/wireless/realtek/rtl88xxau/
- reboot device

- change gui color in files raspyjack.py and gui_conf.json




## Bootable usb with Tails6.26 OS
- tool to increase anonimity in TOR exploration
- install tail os in bootable usb
- extra tools installed:

