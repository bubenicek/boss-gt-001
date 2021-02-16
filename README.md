# boss-gt-001

 Boss GT-100 on Linux
Tags: linux, kernel, guitar
	

27 04 2012 #038

The amazing newest model of the famous Boss GT series has recently been released by Roland, called the GT-100. It's not functional on Linux out of the box, so ALSA has to be tweaked in order to get it working as USB in/out sound interface which also supports MIDI.

The chipset itself isn't entirely new, but the USB device ID changed, which is the reason for Linux not readily recognizing the floorboard.

Get your kernel source and head to the ALSA USB headers and edit the quirks table for USB interfaces:

cd /usr/src/linux-source-`uname -r`/sound/usb
vi quirks-table.h

According to lsusb -v its device ID is 0582:014d, so we simply add a new entry to the end of the tables header:www
    USB_DEVICE(0x0582, 0x014d),
    .driver_info =
      (unsigned long) & (const struct snd_usb_audio_quirk) {
        .ifnum = QUIRK_ANY_INTERFACE,
        .type = QUIRK_COMPOSITE,
        .data = (const struct snd_usb_audio_quirk[]) {
            {
                .ifnum = 1,
                .type = QUIRK_AUDIO_STANDARD_INTERFACE
            },
            {
                .ifnum = 2,
                .type = QUIRK_AUDIO_STANDARD_INTERFACE
            },
            {
                .ifnum = 3,
                .type = QUIRK_MIDI_FIXED_ENDPOINT,
                .data = & (const struct snd_usb_midi_endpoint_info) {
                    .out_cables = 0x0001,
                    .in_cables  = 0x0001
                }
            },
            {
                .ifnum = -1
            }
        }
    }
},

We don't need to compile the whole new kernel, just the USB sound modules. Remember that these commands will need root privileges.

make -C /usr/src/linux-headers-`uname -r` M=`pwd` modules
make -C /usr/src/linux-headers-`uname -r` M=`pwd` modules_install
depmod -a
update-initramfs -u

The very last step is only needed if you want your newly patched module during the boot process. Note: There is a very interesting OSS project called FxFloorBoard, already offering a development preview for managing the GT-100, although it's not quite working with the new device at this time. I'm really looking forward to this piece of software in the near future. Enjoy!

Update: The code has finally made its way to the official ALSA kernel sources. So you either compile ALSA from Git or wait until your distribution includes the kernel, but this may still take a while.
