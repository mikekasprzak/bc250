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

### Questions

* Does this use an 8pin CPU cable or 8pin GPU cable?

## Warnings

* the 8-pin GPU power connector on common power supplies is **only spec'd for 150W**!!
* I'm not sure about the 8 pin CPU connector (or dual 4 pin), but [this](https://www.moddiy.com/pages/Power-Supply-Connectors-and-Pinouts.html) says it's over 300W.
