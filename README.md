# ID-s for USB-serial adapters

Unique semi-human-friendly IDs for USB-UART adapters (e.g. FTDI, CP) generated from serial numbers

This repository holds scripts to generate unique serial numbers for dual-port FTDI chips commonly used as JTAG programmers and TTL level UART adapters. The serial numbers are stored in the EEPROM of each device and they show up as a symlinks in the `/dev` directory. No more detective work to figure out which `/dev/ttyUSB?` device belongs to your USB-serial adapter!

Example: if your ESP Prog board has been flashed with serial `EP012` then its two ports would always show up as symlinks `/dev/EP012_uart` and `/dev/EP012_jtag` regardless of the order in which USB serial devices were plugged into the computer.

## Install prerequisites

Most devices have factory-issued serial numbers, e.g. "FTYI1XTW". To use those, you don't need anything extra - just follow the instructions in next section.

To program FTDI devices which don't have factory-issued serial numbers, you need the `ftdi_eeprom` utility:

```
$ sudo apt install ftdi-eeprom
```

To see the serial number and other attributes of the device, plug it in and query using `lsusb`, e.g.:

```
lsusb -d 0403:6015 -v
```

## Install the udev rules for device names

To install udev rules which create the symlink with device name, link the following files to `/etc/udev/rules.d` and restart udev:

```
$ cd ftdi_ids
$ sudo find "$(pwd)" -name "*.rules" -exec ln -s "{}" /etc/udev/rules.d/ \;
$ sudo systemctl restart udev
```

Now plug in your USB-serial adapter (which has a unique serial number assigned to it) and see nice names like `/dev/EP001_uart` or `/dev/EWK001_jtag` show up as symlinks to the correct `/dev/ttyUSB?` devices.

## Flash a new FTDI device with a unique serial number

Before the names appear, the FTDI device must have a unique serial number programmed into it (the factory default for some ESP devices is a blank EEPROM, which we need to program ourselves). This is easy enough.

* Open the file `ESP-Prog/eeprom_ft2232h_ESP-Prog.conf` (if using an ESP Prog board) or `ESP-WROVER-KIT/eeprom_ft2232h_ESP-WROVER-KIT.conf` (if using an ESP WROVER Kit board)
* Increment the number in field `serial` by one. Save and close, commit the file to git and push.
* Follow instructions in the header of same file to flash the EEPROM
* Reconnect the board, watch the new symlink appear in `/dev`
* Did you remember to commit the file to git and push?
