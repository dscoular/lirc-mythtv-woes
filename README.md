# lirc-mythtv-woes

Since upgrading from Mythbuntu 16.04.5 to 18.04, my mythtv config files have two issues:

1. Any remote key press loops until I hit a key on the keyboard, then the remote works perfectly.
2. Hitting the KEY_SLEEP on the remote sends my server to sleep! This is a 24x7 mythtv server so that is bad news.

The 16.04.5 config worked perfectly. I see lots of advice about removing lirc but then I read that the reason people think this is the case is due to the fact that lirc config files have changed radically since 0.9.4 and that all you really need to do is manually rewrite the configs. I followed what I could of the guide at https://lirc.org to update to 0.10.0 (the version on mythbuntu 18.04).


This repo is just to let people browse my config files and give their 2 cents.

Oh, and here is some of the relevant output from various input related commands:

## lsusb: List our USB devices and we find the Infrared Transceiver
```
# lsusb
Bus 004 Device 003: ID 046d:c52b Logitech, Inc. Unifying Receiver
Bus 004 Device 006: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port
Bus 004 Device 008: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port
Bus 004 Device 007: ID 067b:2305 Prolific Technology, Inc. PL2305 Parallel Port
Bus 004 Device 005: ID 04cc:1122 ST-Ericsson Hub
Bus 004 Device 004: ID 0a12:0001 Cambridge Silicon Radio, Ltd Bluetooth Dongle (HCI mode)
Bus 004 Device 002: ID 0409:005a NEC Corp. HighSpeed Hub
Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 002 Device 003: ID 0471:060c Philips (or NXP) Consumer Infrared Transceiver (HP)
Bus 002 Device 002: ID 0d8c:013c C-Media Electronics, Inc. CM108 Audio Controller
Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 001 Device 002: ID 05a7:1020 Bose Corp.
Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
```
Our device is the "Philips (or NXP) Consumer Infrared Transceiver (HP)".

## ir-keytables has mapped the device as a remote control: rc0 (event4)
```
# ir-keytable
Found /sys/class/rc/rc0/ (/dev/input/event4) with:
        Name: Media Center Ed. eHome Infrared Remote Transceiver (0471:060c)
        Driver: mceusb, table: rc-rc6-mce
        lirc device: /dev/lirc0
        Supported protocols: lirc rc-5 rc-5-sz jvc sony nec sanyo mce_kbd rc-6 sharp xmp
        Enabled protocols: lirc rc-5 rc-6
        bus: 3, vendor/product: 0471:060c, version: 0x0101
        Repeat delay = 500 ms, repeat period = 125 ms
```

We see lirc, rc-5 and rc-6 protocols are enabled and the recommended repeat delay and repeat period are in use.

## xinput shows the device as id=9

```
$ xinput
⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ Logitech K400                             id=10   [slave  pointer  (2)]
⎜   ↳ lircd-uinput                              id=11   [slave  pointer  (2)]
⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
    ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
    ↳ Power Button                              id=6    [slave  keyboard (3)]
    ↳ Power Button                              id=7    [slave  keyboard (3)]
    ↳ C-Media Electronics Inc.       USB PnP Sound Device       id=8    [slave  keyboard (3)]
    ↳ Media Center Ed. eHome Infrared Remote Transceiver (0471:060c)    id=9    [slave  keyboard (3)]
    ↳ Logitech K400                             id=12   [slave  keyboard (3)]
    ↳ lircd-uinput                              id=13   [slave  keyboard (3)]
```
We can get more details on our IR remote transceiver (id=9):

```
$ xinput list-props 9
Device 'Media Center Ed. eHome Infrared Remote Transceiver (0471:060c)':
        Device Enabled (153):   1
        Coordinate Transformation Matrix (155): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000
        libinput Send Events Modes Available (272):     1, 0
        libinput Send Events Mode Enabled (273):        0, 0
        libinput Send Events Mode Enabled Default (274):        0, 0
        Device Node (275):      "/dev/input/event4"
        Device Product ID (276):        1137, 1548
```
Attempts to disable the remote seem to have no impact. I've tried using various combinations of:
```
$ xinput disable 9
$ xinput set-props 9 153 0
```

# xev - after booting, if we hit any key on the remote we enter a fast loop of generated X events.

We try the remote's up arrow and get the Up key event reported by the X server (weird - I expected all remote keys to be handled by lirc):
```
KeyPress event, serial 37, synthetic NO, window 0x4400001,
    root 0x1e1, subw 0x0, time 1321786, (89,64), root:(960,540),
    state 0x0, keycode 111 (keysym 0xff52, Up), same_screen YES,
    XLookupString gives 0 bytes:
    XmbLookupString gives 0 bytes:
    XFilterEvent returns: False
```
As soon as I hit a key on my Logitech K400 keyboard, the looping 	immediately  stops and I see the single space key event from the 	keyboard:
```
KeyRelease event, serial 38, synthetic NO, window 0x4400001,
    root 0x1e1, subw 0x0, time 1764624, (89,64), root:(960,540),
    state 0x0, keycode 65 (keysym 0x20, space), same_screen YES,
    XLookupString gives 1 bytes: (20) " "
    XFilterEvent returns: False
```
Interestingly, from this point onwards xev sees no more remote key presses.

# irw
Lirc's irw client sees kernel key events:
```
$ irw
000100ae00000001 00 KEY_EXIT devinput-64
0001006700000001 00 KEY_UP devinput-64
0001006c00000001 00 KEY_DOWN devinput-64
```

Doug Scoular

