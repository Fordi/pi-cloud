# Pi Cloud : Samba

The final step is setting up your storage and sharing it.

## Setting up your drive

First, let's make sure we have the widest filesystem support possible.  We'll be installing support for NTFS, exFAT, and HFS+:

* `sudo apt install -y ntfs3g exfat-fuse exfat-utils hfsplus hfsutils`

You'll need to locate your drive.  Plug it in, open your Pi console, then run:

* `lsblk`

Under there, you should see `mmcblk0` (your Pi's microSD card), and `sda` (your USB drive).  You might see several partitions; remember the one with the largest entry in the `SIZE` column (in my case, `sda2`).

Anywhere I write `sda2`, substitute in your drive's name:

Next, you may need to format your drive.  If your disk is already formatted, skip this.  So you can use your drive elsewhere, we're going to go with exFAT.

* `mkfs.exfat -n "BigChonk" /dev/sda2`

    mkexfatfs 1.3.0
    Creating... done.
    Flushing... done.
    File system created successfully.

Next, we'll need some parameters:

* `blkid /dev/sda2`

    /dev/sda2: LABEL="BigChonk" UUID="A1B2C3D4E5F60718" TYPE="exfat" 
    PTTYPE="dos" PARTLABEL="Basic data partition" PARTUUID="837a4475-6f18-4b46-908e-212c2ab10353"

Make a note of **PARTUUID** and **TYPE**, then create a directory in the root filesystem for the drive.  We'll use the drive's label, but you can name it whatever you want.  This will be the **mount path**

* `sudo mkdir /BigChonk`

Finally, make an entry in `/etc/fstab` for the drive, so that it's always mounted on-boot and by root.  The entry is of this format:

PARTUUID={PARTUUID from above} {mount path} {TYPE} [options] [dump] [passno]

For options, we'll use: `defaults,nofail,noatime,rw,users,user_id=0,group_id=0,default_permissions,allow_other,umask=000`.  These will mount the drive as root, and allow any user to make changes to the drive (important for Samba to
work with little fuss; it's not great for a multi-user or public system, but
you're configuring your Pi as a private server with one user, `pi`).

We never want to dump, so that's `0`, and we want the drive scanned on the
last boot-time pass, or `2`.

> Note: if TYPE is `ntfs`, you'll want to use `ntfs-3g` instead.

So, the line we're adding to `/etc/fstab` should look something like:

* `sudo nano /etc/fstab`

        # ... The below should be the new last line of the file
        PARTUUID=837a4475-6f18-4b46-908e-212c2ab10353 /BigChonk exfat defaults,nofail,noatime,rw,users,user_id=0,group_id=0,default_permissions,allow_other,umask=000 0 2
  
## Samba

Install Samba

* `sudo apt install samba samba-common-bin`

* Create a Public directory on your drive: `mkdir /BigChonk/Public`

Edit Samba's config

* `sudo nano /etc/samba/smb.conf`

    Just under `workgroup = WORKGROUP`, add the line
    `netbios name = {your host name}`

    Now, roll all the way to the bottom.

    For the `BigChonk` share we created earlier, we'll create a share.  Add
    the lines:

        [BigChonk]
        path=/BigChonk/Public
        browsable=yes
        writable=yes
        read only=no
        guest ok=yes
        guest only=yes
        create mask=0644
        directory=0755
        public=yes

    This will create an SMB share that's public within your network (which is
    accessed from the outside world only via your VPN).

    If you want to create a share that's secured to your Pi's password, first create the directory, and share it by adding the following to `/etc/samba/smb.conf`.  If you create more smb users (see below), `valid users` accepts a comma-delimited list of users and @groups.

        [Secrets]
        path=/BigChonk/Secrets
        browsable=yes
        valid users=pi
        writable=yes
        read only=no
        guest ok=no
        guest only=no
        create mask=0644
        directory mask=0755
        public=no

    Last, if you want to restrict a share to certain _VPN_ users, first look up the user's VPN IP:

    `pivpn -c`

        ::: Connected Clients List :::
        Name           Remote IP                 Virtual IP      Bytes Received      Bytes Sent      Last Seen
        jim            123.45.67.89:22620        10.6.0.2        55KiB               201MiB          Jun 01 2021 - 16:47:24
        bob            12.345.67.89:22620        10.6.0.3        87KiB               143KiB          Jun 01 2021 - 16:56:13
        amy            87.65.43.210:22620        10.6.0.4        51MiB               128KiB          Jun 01 2021 - 17:22:55

    Find the Virtual IP for the users you want to grant access to the share, and place that in a `hosts allow` line.  You may also want to add
    `hide unreadable=yes`, though I haven't found that hides VPN user shares.
    Like `valid users`, `hosts allow` permits a comma-delimited list.

        [Bob]
        path=/BigChonk/Bob
        browsable=yes
        hosts allow=10.6.0.3
        guest ok=yes
        guest only=yes
        create mask=0644
        directory mask=0755
        public=no
        hide unreadable=yes

Each time you tweak `/etc/samba/smb.conf`, you'll need to restart Samba:

* `sudo systemctl restart smbd`

If you've messed with `netbios name` or `workgroup`, you'll also need to restart NMB:

* `sudo systemctl restart nmbd`

### Adding new Samba users / groups

If you need to provide password-protected access for multiple users on your shares, you can create new users and groups.

To create a user:

* `sudo useradd -M -N cookie`
* `sudo smbpasswd -a cookie`
* Enter the user's new password

To create a group (which can be referenced in `smb.conf` with a leading `@`):

* `sudo groupadd cookiejar`

To add a user to a group:

* `sudo usermod -a -G cookiejar cookie`

To delete a user

* `sudo userdel cookie`

Finally, to change a user's SMB password,

* `sudo smbpasswd cookie`
* Enter the user's new password

See the headings above for how to set up a user- or group-resticted share.

## WSDD

You'll find that Windows 10 computers can't see your Raspberry Pi, either locally or through the VPN.  This is because Windows 10 no longer supports the NMB protocol for security reasons.  To fix it, we have to install and configure `wsdd`:

* `echo "deb https://pkg.ltec.ch/public/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/wsdd.list`
* `apt-key adv --fetch-keys https://pkg.ltec.ch/public/conf/ltec-ag.gpg.key`
* `sudo apt update`
* `sudo apt install -y wsdd`

Now configure it to match your Samba config:

* `WORKGROUP=$(grep -oP '(?<=workgroup ?= ?).*' /etc/samba/smb.conf)`
* `SMBHOST=$(grep -oP '(?<=netbios name ?= ?).*' /etc/samba/smb.conf)`
* `echo 'WSDD_PARAMS="'"-v -w $WORKGROUP -n $SMBHOST -p"'"' | sudo tee /etc/wsdd.conf`

## Validate

### Local access

Go to another machine on your local network, and look in its network places.  For Windows, this is near the bottom on Explorer's left sidebar; for OS-X, it will be in the left sidebar under "Locations".  It varies on Linux, but I'm already kinda talking down to Linux users; you know where it is.

You should see a computer in your network places corresponding to the hostname you chose.  If you double-click it, you should see the shares you configured.  Drill down deeper, and you'll get the files on those shares.

### Remote access

With a phone or tablet, turn off WiFi and activate your VPN.  Install VLC Media Player (it has good SMB browsing support), and tap "Browse" on the bottom tab bar (looks like a folder).  In the "Local Network" section, you should see your Pi appear.  Make sure you can browse in, and, for funsies, put a movie file on your Pi so you can try playing it through your VPN.

### Conclusion

If you're here, and everything's good, that's about the last of it.  You should now have a working Cloud server you can access from anywhere!

If you're here and everything's _not_ working, please [file an issue](../../issues), and I'll be happy to help you debug the problem.
