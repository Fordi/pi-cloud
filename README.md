# Pi Cloud

This tutorial will walk you through the steps needed to set up a Raspberry Pi with the necessary
software to run your own cloud server from your home network.  The goal is to minimize the required
changes to your main machine and router, while providing full access to your home network.

It will require the following hardware:

* A Raspberry Pi.  The RPi 4 series is the recommended hardware, but any Pi to which you can arrange
  a network connection will suffice.  I personally use a Pi 400, since having a ready keyboard is
  helpful.
* A suitable power supply.  For a Raspberry Pi 4, this is a 3A (15W) or better USB-C charger brick.  For earlier
  models, use a 2.5A (12.5W) micro USB charger.  For a Pi Zero, you can get away with a 1.2A (6W) micro USB charger.
* A microSD card.  Ideally, this will be a brand-name (SanDisk, Samsung, etc) XCII V90 card.  See the SD Association's
  [Speed Class](https://www.sdcard.org/developers/sd-standard-overview/speed-class/) page for more information.
* A way to write data to microSD cards.
* [Recommended] A keyboard, mouse, and monitor.  This isn't _absolutely_ necessary, but it's a lot easier to
  work with Rasperry Pi if you can directly control it.
* [Recommended] An ethernet cable.  Again, not absolutely necessary, but things will perform much better over
  the wire than over the air.
* [Recommended] A large chunk of USB 3.0 storage.

The main steps we will walk through:

1. [Basic setup](Basic%20setup.md) - Set up your Raspberry Pi and be able to consistently control it from your main computer.
2. [Dynamic DNS](Dynamic%20DNS.md) - This will allow your always-changing public IP to be addressable by an unchanging domain name.
3. [PiVPN](PiVPN.md) - This will enable you to access your local network from the outside world, through an authenticated, encrypted, and easy-to-use tunnel.
4. [Samba](Samba.md) - This will share your Big Storage with authenticated users, and also includes permanent mounting instructions.

If you've got all he required hardware, proceed to [Basic setup](Basic%20setup.md).