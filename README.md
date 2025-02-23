# How to BC-250?

## Practical Requirements

Sure you can do whatever you want, but IMO you want these:

* An NVMe 2.0 drive
* A 250W+ PSU with 12V outputs (i.e. yellow wires in a PC power supply)
* A fan and shroud
* A gigabit ethernet connection
* A USB stick

At the time of this writing, installing Linux is a little wonky, and MESA drivers need to be patched.

Ubuntu setup notes: <https://github.com/chris2355/BC250-Proxmox->

## Questions

* Does this use an 8pin CPU cable or 8pin GPU cable?

## Warnings

* the 8-pin GPU power connector on common power supplies is only spec'd for 150W!!
  * Technically the pinout of the 8pin and 6pin are the same, but the extra 2 pins are grounded or used as sense pins
  * 
