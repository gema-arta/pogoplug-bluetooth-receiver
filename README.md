#Pogoplug Bluetooth Speaker

Setup Pogoplug to receive and play audio stream over Bluetooth from device such as smartphone. Galaxy S5 is used as a music source in this project.

#Hardware Requirements

* Pogoplug Pink with Arch Linux (Linux 4.4.6-1-ARCH #1 PREEMPT armv5tel)
* Audio USB dongle (CMedia CM119 tested)
* Bluetooth USB dongle (Random low cost Bluetooth dongle tested)
* Stereo PC Speaker
* Music source (Any smartphone with bluetooth would work fine)

#Installation

##Assumption

* Your login id is 'blue'
* Audio USB, Bluetooth USB shoud be detected by system

```
# lsusb
Bus 001 Device 004: ID 0d8c:0008 C-Media Electronics, Inc. 
Bus 001 Device 003: ID 1131:1001 Integrated System Solution Corp. KY-BT100 Bluetooth Adapter
```

##Audio USB dongle

####As root

```
# pacman -Syu alsa-utils alsa-lib
# gpasswd -a blue audio
```

####As blue

```
<Log out, then log back in again after gpasswd command>
$ alsamixer # Set Master volume to max
$ speaker-test -c 2 # Left and right speaker should make sound alternatively
```

####As root

```
# pacman -Syu pulseaudio pulseaudio-alsa pulseaudio-bluetooth
# useradd -m pulse # Important! pulseaudio daemon creates config files in 'pulse' home directiory
# gpasswd -a pulse audio
```

Create systemd service file(/etc/systemd/system/pulseaudio.service) for pulseaudio

```
/etc/systemd/system/pulseaudio.service
[Unit]
Description=pulseaudio service
Requires=bluetooth.target

[Service]
User=pulse
ExecStart=/usr/bin/pulseaudio -v
Restart=always

[Install]
WantedBy=multi-user.target
```

Install systemd service file and enable it

```
# systemctl enable pulseaudio
# systemctl start pulseaudio
# systemctl status pulseaudio # Make sure pulseaudio service is running without any error message
```

##Bluetooth USB dongle

####As root

```
# pacman -Syu bluez bluez-libs bluez-utils
# gpasswd -a blue lp
# gpasswd -a pulse lp
# systemctl enable bluetooth
# systemctl start bluetooth
# hciconfig hci0 up
```

* Members in lp group can access bluetooth according to /etc/dbus-1/system.d/bluetooth.conf

###Pairing

```
# bluetoothctl
[bluetooth]# agent KeyboardOnly
[bluetooth]# default-agent
[bluetooth]# discoverable on
[bluetooth]# pairable on
[bluetooth]# trust [dev]
```

###Verifying a2dp is properly configured

```
[bluetooth]# show
Controller 00:11:67:12:34:56
...
	UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
...
```
* If 'Audio Sink' doesn't show up, then, check [pulseaudio](README.md#as-root-1) configuration again

###Auto start Bluetooth USB on reboot

```
/etc/udev/rules.d/10-bluetooth.rules
ACTION=="add", KERNEL=="hci0", RUN+="/usr/bin/hciconfig hci0 up"
```

#Future works

* How can CPU load of pulseaudio be lowered?. It is around 75% while playing streaming audio.
 * high-priority = no, realtime-scheduling = no in /etc/pulse/daemon.conf didn't affect on load
 
![High CPU LOAD](/cpu-load.png?raw=true "High CPU Load")

* iPhone issues(iPhone 4, iOS 6.0.1)
 * iPhone can't discover Pogoplug, therefore, pairing process should be initiated from Pogoplug.
 * iPhone can't control volume. Galaxy S5 has no problem.

#References

* https://wiki.archlinux.org/index.php/Bluetooth
* https://wiki.archlinux.org/index.php/PulseAudio
* https://delx.net.au/blog/2014/01/bluetooth-audio-a2dp-receiver-raspberry-pi/
