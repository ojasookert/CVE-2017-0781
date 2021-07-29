# CVE-2017-0781 PoC

## Overview

This is an implementation of the CVE-2017-0781 Android heap overflow vulnerability described in the Blueborne whitepaper released by Armis. Further reading: https://www.armis.com/blueborne/

In the current state, this code only demonstrates the overflow and the ability of crashing the bluetooth service. Again, this is not a fully developed remote code execution, but it can be.

## Instructions

Get pwntools.

```
apt-get update
apt-get install python2.7 python-pip python-dev git libssl-dev libffi-dev build-essential
pip install --upgrade pip
pip install --upgrade pwntools
```

Get pybluez.

```
apt-get install bluetooth libbluetooth-dev
pip install pybluez==0.22
```

I have used the `hciconfig` and `btmgmt` tools for this, both are included in the bluez package. If you get your bluetooth module locked, `rfkill` might help.

Run `btmgmt`. 

The `info` command will show the indices of your devices.

Entering `select 0` will make the first bluetooth controller active. A shortcut for this is to launch the tool with `btmgmt --index 0`.

Make sure you can discover devices with the `find` command. Your Android's screen must be on and the bluetooth settings view must be open for it to be discoverable. Note that discoverability is not a prerequisite for exploiting this vulnerability as detailed in the whitepaper released by Armis.

For the exploit to work without manual pairing, you must set the IO capabilities of your host with `io-cap 0x03` in the btmgmt tool.

With this set, run the code with `python CVE-2017-0781.py TARGET=XX:XX:XX:XX:XX:XX` and your Android device's bluetooth service should crash. It might take a few tries. Currently the code sends 30 of these invalid packets to corrupt enough memory for the process to crash.

Happy hacking ;)

## Troubleshooting

Make sure you see your bluetooth device on the host with the command `hciconfig`.

```
user-pc user # hciconfig
hci0:	Type: BR/EDR  Bus: USB
	BD Address: XX:XX:XX:XX:XX:XX  ACL MTU: 310:10  SCO MTU: 64:8
	DOWN 
	RX bytes:580 acl:0 sco:0 events:31 errors:0
	TX bytes:368 acl:0 sco:0 commands:30 errors:0
```

If it is not UP but rather DOWN as shown here, then fix it with `hciconfig <intf> up`

```
user-pc user # hciconfig hci0 up
Can't init device hci0: Operation not possible due to RF-kill (132)
```

If you get messages about rf-kill, try the `rfkill list` command.

```
user-pc user # rfkill list
0: phy0: Wireless LAN
	Soft blocked: no
	Hard blocked: no
2: asus-wlan: Wireless LAN
	Soft blocked: no
	Hard blocked: no
3: asus-bluetooth: Bluetooth
	Soft blocked: no
	Hard blocked: no
4: hci0: Bluetooth
	Soft blocked: yes
	Hard blocked: no
```

Unblocking can be done with `rfkill unblock <id>`.

If you see that your device still asks for pairing code after setting the IO capabilities on the host, then this method is probably not currently possible on your device.
