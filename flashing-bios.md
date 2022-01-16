## BIOS
BIOS on this device is a 1Mbit (128 Kb) FLASH, soldered on board which containa a proprietary Soekris Engineering comBIOS.

The net4501 comes with the Soekris Engineering netBIOS. The BIOS is designed especially for setup and
operation using the serial port as the console. The BIOS is located in Flash memory, and can be upgraded
over the serial port. Critical system setup parameters are also saved in the Flash memory, so the system will
not lose any setup information due to CMOS battery backup power loss.

Running newer Linux kernels and GRUB2 requires updating to BIOS v1.33.

## Updating the BIOS
Flashing new BIOS to your net4501 requires sending the new BIOS binary image over XMODEM protocol, over a serial connection. Binary image will be saved in memory at
4000:0000, from where it is flashed later in the process.

### Prerequisites
A null modem cable is required to communicate with your Soekris net4501. (separate page?)

#### GNU/Linux 
To communicate with your net4501 we can use `tio` a simple tty I/O application. To transfer the binary BIOS via XMODEM we will use `sx` which is a part of `lrzsz` package. Install it using your package manager.

For Debian-based distros:
```
sudo apt install tio lrzsz
```

Make sure XON/XOFF flow control is disabled.
To temporarily disable XON/XOF flow control in a current terminal session use:

`stty -ixon`

This will not persist beyond your current session.

Default baud rate for net4501 is `19200`, however, if working on modern hardware feel free to raise it to `115200`. If issues appear during this process lower the baud rate progressively until issues are gone.

### Flashing

1. Obtain the BIOS image from Soekris http://soekris.com/downloads.html
Be cautious, if running a BIOS prior to `1.20`, you will have to update to `1.26a` first, and then update to `1.33` (the latest known version).

2. Connect to you net4501 via serial console. If running `tio`:

```tio /dev/ttyS0 -b 115200```

Replace `115200` with the appropriate baud rate for your device (default is `19200`).

3. Upon successfully connecting to the device, enter Monitor mode by pressing `CTRL+P` prior to booting. After getting to the `>` prompt, issue `download` command.

Since BIOS v1.22 the download command defaults to CRC. By entering `download -` it's possible to revert to checksum.


4. Disconnect `tio` by pressing `CTRL+T Q` (release CTRL before pressing Q).

5. Send your new BIOS image to your net4501 using `sx`.

In most GNU/Linux distros:
```
sx -X xxx.bin > /dev/ttyS0 < /dev/ttyS0
```

replace `xxx.bin` with the name of your BIOS file and `/dev/ttyS0` to designate your serial connection.

6. Wait for the process to complete. Even faster baud rates are sluggish. e.g. `115200` **baud** is `14.0625` **Kbps**. When the file is transfered, you should get a message similar to `Transfer complete`.

7. Reconnect to serial console (see step 2.).

8. Issue the `flashupdate` command. In case your transfer has failed, or the BIOS image is corrupted, net4501 *should* check against a checksum. It *should* not allow you to flash a corrupt image.

9. Finally, issue a `reboot` command. Your net4501 will now reboot. After POST you should now see it is using a new version of BIOS.