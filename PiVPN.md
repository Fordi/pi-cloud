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
    > Note: not all routers act as DNS servers, but most modern ones do.  If after you connect to your VPN, you can't, say, browse to Google, start over here,
    > and select one of the default DNS providers.  If this becomes necessary, you'll also need to reconfigure all your VPN clients.
* For public IP, select `DNS Entry`, and enter the domain name you set up in [Dynamic DNS](Dynamic%20DNS)
* When asked about unattended upgrades, select `<Yes>`.
* Do not reboot yet; select `<No>`.

## UPnP

This part will enable your Pi to open up an incoming port in your router without you needing to go into your router's admin console (which, if your router doesn't support UPnP, I can't actually help you with - more than saying, "Open a port from your router for TCP and UDP ports 51820 into your Pi").

* Install the `upnpc` command:  
    `sudo apt install miniupnpc`
* Create a new service:  
    `sudo systemctl edit --force --full wg-upnp.service`

* Paste the following:

        [Unit]
        Description=UPnP forwarding for WireGuard
        # Ensures this runs after wireguard is started
        After=wg-quick@wg0.service
        # Ensures this stops when wireguard is stopped
        Requires=wg-quick@wg0.service

        [Service]
        # Lets the service remain in the "running" state, even though the actual "run" exits immediately
        Type=oneshot
        RemainAfterExit=true
        # Delays execution until after the Pi has an default route to the gateway (router).
        # Apparently, `Requires=network-online.target` doesn't do this.  Who knew?
        ExecStartPre=/bin/sh -c 'until ip route list | head -1 | grep -Po '"'"'(?<=default via )([0-9\\.]+)'"'"'; do sleep 1; done'
        # Runs upnpc to map the port; the grep command is looking up WireGuard's listenPort from its config file.
        ExecStart=/bin/sh -c '/usr/bin/upnpc -e WireGuard -r "$(grep -Po '"'"'(?<=ListenPort = )([0-9]+)'"'"' /etc/wireguard/wg0.conf)" UDP'
        # Runs upnpc to unmap the port
        ExecStop=/bin/sh -c '/usr/bin/upnpc -d "$(grep -Po '"'"'(?<=ListenPort = )([0-9]+)'"'"' /etc/wireguard/wg0.conf)" UDP'

        [Install]
        # Ensures that starting WireGuard also starts this.
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

If your VPN is all set up, you should see something like this:

    ::: Connected Clients List :::
    Name           Remote IP                 Virtual IP      Bytes Received      Bytes Sent      Last Seen
    fordi          123.45.67.89:22620      10.6.0.2        7.0KiB              17KiB           Jun 01 2021 - 15:43:30

If your UPnP forwarding is set up, you should see something like this after running `systemctl status wg-upnp`

    ??? wg-upnp.service - UPnP forwarding for WireGuard
       Loaded: loaded (/etc/systemd/system/wg-upnp.service; enabled; vendor preset: enabled)
       Active: active (exited) since Thu 2021-06-03 02:13:04 EDT; 16s ago
      Process: 2183 ExecStartPre=/bin/sh -c until ip route list | head -1 | grep -Po '(?<=default via )([0-9\.]+)'; do sleep 1; done (code=exited, sta
      Process: 2187 ExecStart=/bin/sh -c /usr/bin/upnpc -e WireGuard -r "$(grep -Po '(?<=ListenPort = )([0-9]+)' /etc/wireguard/wg0.conf)" UDP (code=e
     Main PID: 2187 (code=exited, status=0/SUCCESS)

    Jun 03 02:13:04 razzle-dazzle sh[2187]:  desc: http://192.168.1.1:2555/upnp/b996b6fb-e181-3ec2-873d-9964b542029b/desc.xml
    Jun 03 02:13:04 razzle-dazzle sh[2187]:  st: urn:schemas-upnp-org:device:InternetGatewayDevice:1
    Jun 03 02:13:04 razzle-dazzle sh[2187]:  desc: http://192.168.1.8:5555/rootDesc.xml
    Jun 03 02:13:04 razzle-dazzle sh[2187]:  st: urn:schemas-upnp-org:device:InternetGatewayDevice:1
    Jun 03 02:13:04 razzle-dazzle sh[2187]: Found valid IGD : http://192.168.1.1:2555/upnp/72e44ef4-f12a-3886-80d5-df06b929214f/WANIPConn1.ctl
    Jun 03 02:13:04 razzle-dazzle sh[2187]: Local LAN ip address : 192.168.1.15
    Jun 03 02:13:04 razzle-dazzle sh[2187]: ExternalIPAddress = 100.11.89.31
    Jun 03 02:13:04 razzle-dazzle sh[2187]: InternalIP:Port = 192.168.1.15:51820
    Jun 03 02:13:04 razzle-dazzle sh[2187]: external 100.11.89.31:51820 UDP is redirected to internal 192.168.1.15:51820 (duration=0)
    Jun 03 02:13:04 razzle-dazzle systemd[1]: Started UPnP forwarding for WireGuard.

If you don't see `duration=0` (e.g., my new router reports `duration=86400`), you'll need to add a cron job to restart the service periodically.

Run:

    $ sudo crontab -e

If you're unsure, pick `nano` as your editor.

In this file, add the line:

    0 6 * * * systemctl restart wg-upnp.service

That will restart the service once a day.  If your `duration` is some other value than `86400` (1 day), you can use [Crontab.guru](https://crontab.guru/) to help you design a scheduling string to match.
    

You should be ready to install [Samba](Samba) now.

## Security

These `{username}.conf` files and QR codes now represent an open door into your private network.  Make sure you don't leave copies of them around and you don't leave them on your screen in public places.
