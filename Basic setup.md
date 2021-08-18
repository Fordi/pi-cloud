# Pi Cloud : Basic Setup

## Initialize your Raspberry Pi's SD card

* Download and install [Raspberry Pi Imager](https://www.raspberrypi.org/software/) to your main machine.
    * \[Optional\]: if you're using `dd`, Balena etcher, or some other imaging software, you can grab a
      [Raspbian image](https://www.raspberrypi.org/software/operating-systems/) directly.
* Plug in your microSD card reader, loaded with the card you intend to use for your OS.
* Click "CHOOSE OS", and select "Raspberry Pi OS (32-bit)"
* Click "CHOOSE STORAGE" and select your microSD card.
* Click "WRITE", and wait until the process finishes
* Unplug your microSD reader.

## Set up WiFi

\[Optional\]

If you're not connecting via ethernet, you'll need to set up WiFi on your Pi before powering it on.

* Plug your microSD reader back in.
* Find the drive labeled "boot"
* Create a new text file in the root of boot called: `wpa_supplicant.conf`, and fill it as appropriate, using 
  this template:

```
country={two-letter country code, e.g., US}
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
  ssid="{WiFi network name}"
  psk="{WiFi network password}"
}
```

## Set up SSH

If you're not using a keyboard/mouse/monitor, you'll need to communicate with your Pi via SSH.

\[Optional\], \[Recommended\]

* Plug your microSD reader back in.
* Find the drive labeled "boot"
* Create a new, empty text file in the root of boot called: `ssh` (no file extension)

## Boot the Pi

* Plug in your keyboard, mouse, monitor, and ethernet as appropriate.
* Plug in your power supply.
* Wait about two minutes.  If you have a monitor connected, you'll see that the first thing it does is expand the system
  partition to fill your micro SD card.  This is normal, but it does mean a boot cycle that a non-monitor user doesn't 
  get notification for.
* Once you've waited (or it's visibly settled to the PIXEL desktop), you're ready to move on.

## Verify connectivity

Ping your Pi from your main machine's command line.  The system doesn't matter here, the command is the same:

    ping raspberrypi.local

On OS-X, you may need to use `ping raspberrypi.home` instead.

You should see something like...

    PING raspberrypi.local (192.168.1.15) 56(84) bytes of data.
    64 bytes from raspberrypi.local (192.168.1.15): icmp_seq=1 ttl=64 time=0.065 ms
    64 bytes from raspberrypi.local (192.168.1.15): icmp_seq=2 ttl=64 time=0.047 ms
    64 bytes from raspberrypi.local (192.168.1.15): icmp_seq=3 ttl=64 time=0.065 ms
    64 bytes from raspberrypi.local (192.168.1.15): icmp_seq=4 ttl=64 time=0.057 ms

For Linux and OS-X, you can hit Ctrl+C to stop.  Windows will stop on its own.  That dotted number, `192.168.x.y`, 
is your Pi's IP address.  ***Write it down***

If this doesn't work, you'll need to grab a keyboard / mouse / monitor and get your Raspberry Pi's IP address manually:

* Open the terminal (the icon that looks like a black computer console)
* ***This is the Pi's console***
* run `ip addr show eth0` if using ethernet, or `ip addr show wlan0` if using wifi.
* It will return something like this:

        2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
            link/ether dc:a6:32:d1:0d:bc brd ff:ff:ff:ff:ff:ff
            inet 192.168.1.15/24 brd 192.168.1.255 scope global noprefixroute eth0
              valid_lft forever preferred_lft forever
            inet6 fe80::13c4:336c:2a93:4196/64 scope link
              valid_lft forever preferred_lft forever

Your Pi's IP address is the four-octet number immediately following `inet` and starting with `192.`.  Drop the `/24` (netmask).
Again, ***write it down***.

## Sign in

If you're not using a keyboard/mouse/monitor, you'll need to sign in via SSH.

\[Optional\]

### Windows + PuTTY

* Install [PuTTY](https://www.putty.org/)
* Open PuTTY
* In "Host Name (or IP address)", enter "raspberrypi.local" (or your Pi's IP address, if that fails)
* Click "Yes" for any security prompts.
* Enter "pi" as the username
* Enter the default password, "raspberry" for the password
* ***This is the Pi's console***

### Linux / OS-X

* Open a terminal
* Run `ssh pi@raspberry.local` (or `ssh pi@raspberry.home` or `ssh pi@192.168...`, filling in your Pi's IP address).
* Type `yes` for any security prompts.
* Enter the default password, "raspberry" for the password
* ***This is the Pi's console***



## Change the password

Since the default password for Raspberry Pis is well-known, you're definitely going to want to change it now.

* Get to your Pi's console
* type `sudo passwd pi`
* type your new password
* type it again

## Change the hostname

You'll want your Pi Cloud's name to be something memorable.  It may contain lowercase `a`-`z`, `0`-`9`, and `-`, and must not
begin with a number or `-`, and cannot end with `-`.  Spaces, uppercase, and other punctuation aren't allowed.

Use the following commands to update the hostname, then reboot.

    HOST="new-hostname"
    echo "$HOST" | sudo tee /etc/hostname
    sudo sed -e 's/^\(127\.0\.1\.1[\s\t]\+\)[a-zA-Z0-9-]\+/\1'"$HOST"'/' -i /etc/hosts
    sudo reboot now

> Note: you've just changed your hostname.  If you've changed it to `razzle-dazzle`, Anything that used to be 
`raspberrypi.local` or `raspberrypi.home` will now be `razzle-dazzle.local` or `razzle-dazzle.home`.

## Localize

You'll want to let the Pi know where it exists in the world.  Each of the following steps uses `raspi-config`.

### Locale

* type `sudo raspi-config` and press \[Enter\]
* Go to `5 Localization Options` and press \[Enter\]
* Go to `L1 Locale` and press \[Enter\]
* Unless you're in the UK, deselect `en_GB.UTF-8 UTF-8`, then find your own locale (for me, it's `en_US.UTF-8 UTF-8`).  It is almost 
  certainly going to be some variant of `lang_COUNTRY.UTF-8 UTF-8`.
* Hit \[Tab\], select `<OK>`, and press \[Enter\]
* Select `C.UTF-8` as the default locale, and press \[Enter\].

### Timezone

* type `sudo raspi-config` and press \[Enter\]
* Go to `5 Localization Options` and press \[Enter\]
* Go to `L2 Timezone` and press \[Enter\]
* Drill down to your local timezone (For me, it's `America`, `New York`).

### Keyboard layout

> Note: if you're not using a physical keyboard, this doesn't matter

* type `sudo raspi-config` and press \[Enter\]
* Go to `5 Localization Options` and press \[Enter\]
* Go to `L3 Keyboard` and press \[Enter\]
* You probably want to pick `Generic 105-key PC`
* Pick your layout.  Mine was `English (US)`
* The defaults are probably fine (`The default for the keyboard layout`, `No compose key`, `<No>`)

### WiFi Country

> Note: if you set up a `wpa_supplicant.conf` in your boot partition at the beginning, you don't need to do this.

* type `sudo raspi-config` and press \[Enter\]
* Go to `5 Localization Options` and press \[Enter\]
* Go to `L4 WLAN Country` and press \[Enter\]
* Find and select your country

### Enable SSH

> Note: if you created the file `ssh` in your boot partition at the beginning, you don't need to do this.

* type `sudo raspi-config` and press \[Enter\]
* Go to `3 Interface Options`
* Go to `P2 SSH`
* Select `<Yes>` and press \[Enter\]

### Enable VNC

Enabling VNC will allow you to access your Pi's desktop for administration without having a keyboard/mouse/monitor connected.  It's a nice convenience, 
but entirely optional.  I recommend [RealVNC](https://www.realvnc.com/en/connect/download/viewer/) as your viewer.

\[Optional\]

* type `sudo raspi-config` and press \[Enter\]
* Go to `3 Interface Options`
* Go to `P3 VNC`
* Select `<Yes>` and press \[Enter\]

If you plan on running your Pi headless, but want VNC access, you'll need to force HDMI on.  Exit `raspi-config`, and do the following:

* `sudo nano /boot/config.txt`
* Uncomment (removing the leading `#` from) the line `hdmi_force_hotplug=1`
* Press \[Ctrl+X\], \[Y\] then \[Enter\]
* `sudo reboot now`


### Update

You'll want your system to be as up to date as possible.

* `sudo apt update`
* `sudo apt -yq upgrade`

Your next step is to set up [Dynamic DNS](Dynamic%20DNS).
