# Mahalia BLE Server

This software is This software is intended to be run on the PC and translates 
data from an openMHA client to a remote openMHA process via a BLE connection.

# Installation Instructions:

The following instructions have been tried on Debian Sid & Ubuntu Bionic.

```
$ sudo apt-get install bluetooth bluez libbluetooth-dev libudev-dev nodejs npm
$ sudo npm install -g npm
```

## Bluetooth Adapter firmware

The onboard Bluetooth adapter for my ThinkPad X1 Carbon 3rd generation worked 
without any installation of any third-party firmware. You may have some issues 
loading the non-free firmware for your Bluetooth adapter. Your internal 
Bluetooth adapter should also work providing you can install the firmware.
This is out of the scope of the provided instructions.

The following instructions detail the installation of the firmware for the
 [Plugable USB-BT4LE USB Bluetooth 4.0 Low Energy Micro Adapter](https://www.amazon.co.uk/gp/product/B009ZIILLI).

```
$ sudo dmesg
...
usb 5-1.4.3: new full-speed USB device number 7 using xhci_hcd
usb 5-1.4.3: New USB device found, idVendor=0a5c, idProduct=21e8
usb 5-1.4.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 5-1.4.3: Product: BCM20702A0
usb 5-1.4.3: Manufacturer: Broadcom Corp
usb 5-1.4.3: SerialNumber: 5CF3708E4B42
Bluetooth: Core ver 2.22
NET: Registered protocol family 31
Bluetooth: HCI device and connection manager initialized
Bluetooth: HCI socket layer initialized
Bluetooth: L2CAP socket layer initialized
Bluetooth: SCO socket layer initialized
usbcore: registered new interface driver btusb
Bluetooth: BNEP (Ethernet Emulation) ver 1.3
Bluetooth: BNEP filters: protocol multicast
Bluetooth: BNEP socket layer initialized
Bluetooth: hci0: BCM: chip id 63
Bluetooth: hci0: BCM: features 0x07
Bluetooth: hci0: BCM20702A
Bluetooth: hci0: BCM20702A1 (001.002.014) build 0000
bluetooth hci0: firmware: failed to load brcm/BCM20702A1-0a5c-21e8.hcd (-2)
firmware_class: See https://wiki.debian.org/Firmware for information about missing firmware
bluetooth hci0: Direct firmware load for brcm/BCM20702A1-0a5c-21e8.hcd failed with error -2
Bluetooth: hci0: BCM: Patch brcm/BCM20702A1-0a5c-21e8.hcd not found
```

As you can see above, the Linux kernel has found no firmware.
Following the [Github Repository for various Broadcom Bluetooth firmware](https://github.com/winterheart/broadcom-bt-firmware) 
we should download the correct firmware into the `/lib/firmware` directory.

```
$ sudo mkdir -p /lib/firmware/brcm
$ sudo wget -O /lib/firmware/brcm/BCM20702A1-0a5c-21e8.hcd https://github.com/winterheart/broadcom-bt-firmware/raw/master/brcm/BCM20702A1-0a5c-21e8.hcd
```

Unplug the adapter, restart the computer then plug the adapter in and try again:

```
$ sudo dmesg
...
usb 5-1.4.3: new full-speed USB device number 8 using xhci_hcd
usb 5-1.4.3: New USB device found, idVendor=0a5c, idProduct=21e8
usb 5-1.4.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 5-1.4.3: Product: BCM20702A0
usb 5-1.4.3: Manufacturer: Broadcom Corp
usb 5-1.4.3: SerialNumber: 5CF3708E4B42
Bluetooth: hci0: BCM: chip id 63
Bluetooth: hci0: BCM: features 0x07
Bluetooth: hci0: BCM20702A
Bluetooth: hci0: BCM20702A1 (001.002.014) build 0000
bluetooth hci0: firmware: direct-loading firmware brcm/BCM20702A1-0a5c-21e8.hcd
Bluetooth: hci0: BCM20702A1 (001.002.014) build 1764
Bluetooth: hci0: Broadcom Bluetooth Device

$ hciconfig
hci0:	Type: Primary  Bus: USB
	BD Address: 5C:F3:70:8E:4B:42  ACL MTU: 1021:8  SCO MTU: 64:1
	UP RUNNING 
	RX bytes:3460 acl:0 sco:0 events:422 errors:0
	TX bytes:38142 acl:0 sco:0 commands:421 errors:0
```

Success! The software will connect using the address `hci0`.

# Download mahalia-ble-server
```
git clone git@git.64studio.net:mahalia/mahalia-ble-server.git
cd mahalia-ble-server
npm install
```

# Scan for Mahalia systems
```
$ sudo nodejs mahalia-ble-server.js <ble_device_name> --disable-wifi
found Mahalia BLE Peripheral 'Mahalia_BWAXXXXXXXXXXXXX' address: 38:d2:69:dc:ea:da
<press Ctrl-C to quit>
```
Please replace any instance of <ble_device_name> in the following commands with the name found above.


# Disable/Enable Wi-Fi support on the Mahalia system

Note this will disable Wi-Fi for the current session as well as every boot.

```
$ sudo nodejs mahalia-ble-server.js <ble_device_name> --disable-wifi
[BLE] Connected to Mahalia BLE Peripheral 'Mahalia_BWAXXXXXXXXXXXXX' address: 38:d2:69:dc:ea:da
Wi-Fi disabled.
```

To enable Wi-Fi support:
```
$ sudo nodejs mahalia-ble-server.js <ble_device_name> --enable-wifi
[BLE] Connected to Mahalia BLE Peripheral 'Mahalia_BWAXXXXXXXXXXXXX' address: 38:d2:69:dc:ea:da
Wi-Fi enabled.
```

# Start the BLE Server

```
sudo nodejs mahalia-ble-server.js <ble_device_name> --daemon
[BLE] Connected to Mahalia BLE Peripheral 'Mahalia_BWAXXXXXXXXXXXXX' address: 38:d2:69:dc:ea:da
...
```

Now you may connect via netcat interface on localhost port 33337. The BLE 
server will translate any commands to the Mahalia system.

Enjoy!
