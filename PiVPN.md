# Pi Cloud : PiVPN

Here, we'll set up PiVPN and WireGuard

* Open your Pi's console
* `wget -O- https://install.pivpn.io | bash`

> Note: if you make a mistake, re-run this command, and select `Reconfigure`

* Read and proceed until you're given a choice.
* If you're going to use ethernet, select `eth0`, if WiFi, use `wlan0`.  Use \[Space\] to select, then press \[Enter\].
* When asked for your DHCP reservation, select `<No>`.  This doesn't actually matter for us, since we'll be setting up UPnP.
* The user for your VPN service is `pi`
* Select `WireGuard` with \[Space\], and press \[Enter\]
* Use the default port, `51820`
* The DNS provider for your clients should be your router, which is normally `192.168.1.1`.  Select `Custom` with \[Space\], then enter `192.168.1.1`.
* For public IP, select `DNS Entry`, and enter the domain name you set up in [Dynamic DNS](Dynamic%20DNS)
* When asked about unattended upgrades, select `<Yes>`.
* Do not reboot yet; select `<No>`.

## UPnP

This part will enable your Pi to open up an incoming port in your router without you needing to go into your router's admin console (which, if your router doesn't support UPnP, I can't actually help you with - more than saying, "Open a port from your router for TCP and UDP ports 51820 into your Pi").

* `sudo apt install miniupnpc`
* `sudo systemctl edit --force --full wg-upnp.service`

* Paste the following:

        [Unit]
        Description=UPnP forwarding for WireGuard
        After=wg-quick@wg0.service
        Requires=wg-quick@wg0.service

        [Service]
        Type=oneshot
        # Delays execution until after the Pi has an default route to the gateway (router).
        # Apparently, `Requires=network-online.target` doesn't do this.  Who knew?
        ExecStartPre=/bin/sh -c 'until ip route list | head -1 | grep -Po '"'"'(?<=default via )([0-9\\.]+)'"'"'; do sleep 1; done'
        # Starts upnpc; the grep command is looking up WireGuard's listenPort from its config file.
        ExecStart=/bin/sh -c '/usr/bin/upnpc -e WireGuard -r "$(grep -Po '"'"'(?<=ListenPort = )([0-9]+)'"'"' /etc/wireguard/wg0.conf)" UDP'
        ExecStop=/bin/sh -c '/usr/bin/upnpc -d "$(grep -Po '"'"'(?<=ListenPort = )([0-9]+)'"'"' /etc/wireguard/wg0.conf)" UDP'
        RemainAfterExit=true

        [Install]
        WantedBy=wg-quick@wg0.service

    
* \[Ctrl+X\], \[Y\], then \[Enter\]
* `sudo systemctl enable wg-upnp.service`
* `sudo systemctl daemon-reload`

## User init

Now, you'll need to create each user for your VPN.

* `pivpn add -n username`

Once that's done, you'll need to register the user on the devices you want to be able to tunnel in.  But first, reboot:

* `sudo reboot now`

## Mobile device setup

* Install WireGuard via your phone or tablet's store
* In the WireGuard app, tap the (+) button, then select "Scan from QR code"
* In your Pi console, run `pivpn -qr username`
* Scan the resulting QR code.
* Run `clear`

## Laptop / Desktop setup

* Install [WireGuard](https://www.wireguard.com/install/) from the website.
* Copy the configuration from your Pi to your local machine via SCP:
    * in Windows, you'd use something like [WinSCP](https://winscp.net/eng/index.php).  Connect with the same host and credentials you use for PuTTY,
    and copy `/home/pi/configs/{username}.conf` to your main machine
    * On OS-X or Linux, run
      `scp pi@{hostname}:/home/pi/configs/{vpn username}.conf .`
* In WireGuard, click `Import tunnel(s) from file`, and import the file you copied over.
* Delete the conf file.  It's a security risk.

## Validate

From whatever device you've set up, activate the tunnel, then run

* `pivpn -c`

If you're all set up, you should see something like this:

    ::: Connected Clients List :::
    Name           Remote IP                 Virtual IP      Bytes Received      Bytes Sent      Last Seen
    fordi          123.45.67.89:22620      10.6.0.2        7.0KiB              17KiB           Jun 01 2021 - 15:43:30

You should be ready to install [Samba](Samba) now.

## Security

These `{username}.conf` files and QR codes now represent an open door into your private network.  Make sure you don't leave copies of them around and you don't leave them on your screen in public places.
