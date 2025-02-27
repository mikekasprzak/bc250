# How to BC-250?

## Practical Requirements

Sure you can do whatever you want, but IMO you want these:

* An NVMe drive
* A 250W+ PSU with 12V outputs (i.e. yellow wires in a PC power supply)
* A fan and shroud
* A gigabit ethernet connection
* A USB stick

At the time of this writing, installing Linux is a little wonky, and MESA drivers need to be patched.

Ubuntu setup notes: <https://github.com/chris2355/BC250-Proxmox->

### Questions

* Q: Does this use an 8pin CPU cable or 8pin GPU cable?
  * A: 8 pin GPU cable.
    * I probed the power connector, and found continuity with 5 of the 8 pins (i.e. ground), then 3 of other pins (12V).
    * `      __      `
    * `[.][.](.)(.)` <- GND GND GND GND
    * `(.)(.)(.)[.]` <- 12V 12V 12V GND

## Warnings

* the 8-pin GPU power connector on common power supplies is **only spec'd for 150W**!!
* I'm not sure about the 8 pin CPU connector (or dual 4 pin), but [this](https://www.moddiy.com/pages/Power-Supply-Connectors-and-Pinouts.html) says it's over 300W.

## PCI Device Map (`lspci`)

### 00:00 -- PCI ROOT

* 00:00.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Ariel Root Complex [1022:13e0]

### 00:01 -- Dummy PCI Bridge

* 00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Ariel PCIe Dummy Host Bridge [1022:13e2]

### 00:08 -- APU PCI Bridge

* 00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Ariel PCIe Dummy Host Bridge [1022:13e2]
* 00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Ariel Internal PCIe GPP Bridge 0 to Bus A [1022:13e5] (prog-if 00 [Normal decode])
  * LnkCap:	Port #0, Speed 16GT/s, Width x16, ASPM L0s L1, Exit Latency L0s <64ns, L1 <1us
  * LnkSta:	Speed 16GT/s, Width x16
  * Kernel driver in use: pcieport

### 00:10 -- USB 3.0 BUS (blue USB ports on back)

* 00:10.0 `lsusb`: Bus 006 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
* 00:10.0 `lsusb`: Bus 007 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub

### 00:11 -- SATA BUS (no visible connectors)

* 00:11.0 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7801] (rev 40) (prog-if 01 [AHCI 1.0])
 	* Kernel driver in use: ahci
 	* Kernel modules: ahci


### 00:12 -- USB 2.0/1.1 BUS

* 00:12.0 `lsusb`: Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
  * Kernel driver in use: ohci-pci
* 00:12.2 `lsusb`: Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
  * Kernel driver in use: ehci-pci

### 00:13 -- USB 2.0/1.1 BUS (black USB ports on back)

* 00:13.0 `lsusb`: Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
  * Kernel driver in use: ohci-pci
* 00:13.2 `lsusb`: Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
  * Kernel driver in use: ehci-pci

### 00:14 -- SMBus

* 00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:780b] (rev 16)
  *	Kernel driver in use: piix4_smbus
  *	Kernel modules: **i2c_piix4**, sp5100_tco

#### 00.14.3 -- ISA Bridge

* 00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:780e] (rev 11)

#### 00.14.4 -- PCI Bridge

* 00:14.4 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] FCH PCI Bridge [1022:780f] (rev 40) (prog-if 01 [Subtractive decode])

#### 00:14.5 -- USB 1.1 BUS

* 00:14.5 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] FCH USB OHCI Controller [1022:7809] (rev 11) (prog-if 10 [OHCI])
  * `lsusb`: 005 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    * Kernel driver in use: ohci-pci


### 00:15 -- PCI to PCI bridge (PCIE port 0)

* 00:15.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Hudson PCI to PCI bridge (PCIE port 0) [1022:43a0] (prog-if 00 [Normal decode])
  * LnkCap:	Port #0, Speed 5GT/s, Width x2, ASPM L0s L1, Exit Latency L0s <64ns, L1 <1us
  * LnkSta:	Speed 5GT/s, Width x2
  * Connected to 03:00.0 (NVMe Connector)
* 00:15.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Hudson PCI to PCI bridge (PCIE port 1) [1022:43a1] (prog-if 00 [Normal decode])
  * LnkCap:	Port #1, Speed 5GT/s, Width x1, ASPM L0s L1, Exit Latency L0s <64ns, L1 <1us
  * LnkSta:	Speed 2.5GT/s, Width x1
  * Connected to 04:00.0 (Ethernet)

### 00:18 -- Memory

* 00:18.0 - 00:18.7 -- 2GB GDDR6


### 01:00 -- APU Devices

* 01:00.0 -- VGA controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Cyan Skillfish [BC-250] [1002:13fe]
  * PCIe 4.0 x16 (16x 16GT/s)
* 01:00.1 -- Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Device [1002:13ff]
  * PCIe 4.0 x16 (16x 16GT/s)
* 01:00.2 -- Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Device [1022:143e]
  * PCIe 4.0 x16 (16x 16GT/s)


### 03:00 -- NVMe Connector

* PCIe 2.0 x2 BUS (2x 5GT/s)


### 04:00 -- Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8211/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 15)

* PCIe 2.0 x1 BUS (5 GT/s)
* Operating at 2.5GT/s

## Interesting Chips

* **M2U2**: NXP L04083B - a PCIe 3.0 Multiplexer
  * "Useful for PCIe Gen 3, DisplayPort 1.2, USB 3.0, SATA 6 Gbit"
  * <https://www.nxp.com/docs/en/data-sheet/CBTL04083A_CBTL04083B.pdf>

## Pinouts of nonstandard connectors

```bash
G = GND
x = NC (floating)
V = 12V
? = Unknown pin
A, B, C = Unknown common pin (might be 5V, 3.3V, etc)
1-9 = Specific pins

^ or v = Pin #1
[] = Socket (female)
() = Pin headers (male)
[==] = Chip

-- J1000 (8-pin GPU power), J2000, and J2001 --
    ___         v             v
[ G G G G ]   [ ? V V V ]   [ V V V ? ]
[ G V V V ]   [ x G G G ]   [ G G G ? ]
        ^     
        
-- TPMS1 --
( G ? ? ? ? G ? ? ? )
( ? ? ? ? ? ?   A G )
  ^

-- J4000 and Speaker/J5 (unpopulated)
( G 3 4 ? )     ? x x x
( A 1 2   )     A G x x 
  ^             ^

-- BIOS_A1 --
A ? 3 4
[=====]
1 2 ? G 
^

-- J4002 --
DisplayPort Connector

-- J4003 (rear, near power and fan) --
( G ? ? ? ? ? ?   )
( G ? ? ? ? ? G ? )
  ^


-- J2 (backside, unpopulated) --
? ? ? ? ? ? x ? x x
? G G G ? ? ? ? G ?
^
```

* NOTE: I would hae expected `G ? ? V` for a speaker

## PCIe Benchmarking

<https://cloud.google.com/compute/docs/disks/benchmarking-pd-performance-linux>

### Test write throughput by performing sequential writes with multiple parallel streams (16+), using an I/O block size of 1 MB and an I/O depth of at least 64:

```bash
write_throughput: (groupid=0, jobs=16): err= 0: pid=3041: Tue Feb 25 19:12:49 2025
  write: IOPS=634, BW=638MiB/s (669MB/s)(187GiB/300517msec); 0 zone resets
    slat (usec): min=55, max=2130.6k, avg=1807.76, stdev=33760.72
    clat (msec): min=11, max=12162, avg=1611.41, stdev=855.49
     lat (msec): min=19, max=12162, avg=1613.21, stdev=854.37
    clat percentiles (msec):
     |  1.00th=[  262],  5.00th=[  506], 10.00th=[  718], 20.00th=[  969],
     | 30.00th=[ 1083], 40.00th=[ 1200], 50.00th=[ 1418], 60.00th=[ 1703],
     | 70.00th=[ 1955], 80.00th=[ 2232], 90.00th=[ 2735], 95.00th=[ 3171],
     | 99.00th=[ 4144], 99.50th=[ 4530], 99.90th=[ 6007], 99.95th=[ 8557],
     | 99.99th=[12013]
   bw (  KiB/s): min=32772, max=3127296, per=100.00%, avg=800962.85, stdev=33412.02, samples=7834
   iops        : min=   32, max= 3054, avg=781.74, stdev=32.61, samples=7834
  lat (msec)   : 20=0.01%, 50=0.03%, 100=0.08%, 250=0.82%, 500=3.97%
  lat (msec)   : 750=6.07%, 1000=11.08%, 2000=49.81%, >=2000=28.61%
  cpu          : usr=0.21%, sys=0.45%, ctx=151453, majf=0, minf=582
  IO depths    : 1=0.0%, 2=0.1%, 4=0.3%, 8=0.7%, 16=2.5%, 32=16.9%, >=64=79.5%
     submit    : 0=0.0%, 4=98.6%, 8=0.4%, 16=0.4%, 32=0.4%, 64=0.2%, >=64=0.0%
     complete  : 0=0.0%, 4=98.8%, 8=0.3%, 16=0.4%, 32=0.3%, 64=0.3%, >=64=0.0%
     issued rwts: total=0,190797,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
  WRITE: bw=638MiB/s (669MB/s), 638MiB/s-638MiB/s (669MB/s-669MB/s), io=187GiB (201GB), run=300517-300517msec

Disk stats (read/write):
  nvme0n1: ios=498/203546, sectors=12928/395259600, merge=42/3425, ticks=534212/300461381, in_queue=301023972, util=88.44%
```

### Test write IOPS by performing random writes, using an I/O block size of 4 KB and an I/O depth of at least 256:

```bash
write_throughput: (groupid=0, jobs=16): err= 0: pid=5509: Tue Feb 25 19:19:13 2025
  write: IOPS=608, BW=611MiB/s (641MB/s)(180GiB/302292msec); 0 zone resets
    slat (usec): min=54, max=2268.1k, avg=6807.18, stdev=85039.35
    clat (msec): min=9, max=7997, avg=1669.10, stdev=898.67
     lat (msec): min=19, max=7998, avg=1675.87, stdev=894.07
    clat percentiles (msec):
     |  1.00th=[  194],  5.00th=[  502], 10.00th=[  751], 20.00th=[ 1003],
     | 30.00th=[ 1099], 40.00th=[ 1217], 50.00th=[ 1485], 60.00th=[ 1737],
     | 70.00th=[ 2005], 80.00th=[ 2299], 90.00th=[ 2937], 95.00th=[ 3339],
     | 99.00th=[ 4463], 99.50th=[ 5000], 99.90th=[ 5805], 99.95th=[ 6678],
     | 99.99th=[ 7684]
   bw (  KiB/s): min=32276, max=3119903, per=100.00%, avg=753711.71, stdev=33929.01, samples=7991
   iops        : min=   31, max= 3046, avg=735.96, stdev=33.13, samples=7991
  lat (msec)   : 10=0.01%, 20=0.06%, 50=0.13%, 100=0.24%, 250=1.02%
  lat (msec)   : 500=3.52%, 750=5.06%, 1000=9.74%, 2000=50.19%, >=2000=30.58%
  cpu          : usr=0.22%, sys=0.41%, ctx=144202, majf=0, minf=583
  IO depths    : 1=0.0%, 2=0.3%, 4=0.8%, 8=1.9%, 16=5.0%, 32=13.5%, >=64=78.4%
     submit    : 0=0.0%, 4=99.0%, 8=0.1%, 16=0.2%, 32=0.3%, 64=0.4%, >=64=0.0%
     complete  : 0=0.0%, 4=99.3%, 8=0.1%, 16=0.1%, 32=0.1%, 64=0.4%, >=64=0.0%
     issued rwts: total=0,183818,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
  WRITE: bw=611MiB/s (641MB/s), 611MiB/s-611MiB/s (641MB/s-641MB/s), io=180GiB (194GB), run=302292-302292msec

Disk stats (read/write):
  nvme0n1: ios=71/193276, sectors=600/381244928, merge=0/313, ticks=73007/291963575, in_queue=292056521, util=89.86%
```

### Test read throughput by performing sequential reads with multiple parallel streams (16+), using an I/O block size of 1 MB and an I/O depth of at least 64:

```bash
read_throughput: (groupid=0, jobs=16): err= 0: pid=5577: Tue Feb 25 19:32:15 2025
  read: IOPS=795, BW=798MiB/s (837MB/s)(235GiB/301305msec)
    slat (usec): min=42, max=605, avg=130.01, stdev=33.89
    clat (msec): min=268, max=6025, avg=1284.12, stdev=807.51
     lat (msec): min=268, max=6025, avg=1284.25, stdev=807.51
    clat percentiles (msec):
     |  1.00th=[  347],  5.00th=[  609], 10.00th=[  701], 20.00th=[  760],
     | 30.00th=[  827], 40.00th=[  844], 50.00th=[  927], 60.00th=[ 1045],
     | 70.00th=[ 1351], 80.00th=[ 1737], 90.00th=[ 2467], 95.00th=[ 3104],
     | 99.00th=[ 4077], 99.50th=[ 4245], 99.90th=[ 5537], 99.95th=[ 5873],
     | 99.99th=[ 5940]
   bw (  KiB/s): min=45072, max=2136576, per=100.00%, avg=849765.03, stdev=22519.21, samples=9144
   iops        : min=   44, max= 2086, avg=829.75, stdev=21.99, samples=9144
  lat (msec)   : 500=3.33%, 750=11.02%, 1000=43.22%, 2000=27.02%, >=2000=15.82%
  cpu          : usr=0.07%, sys=0.79%, ctx=219453, majf=0, minf=583
  IO depths    : 1=0.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=3.2%, >=64=96.8%
     submit    : 0=0.0%, 4=100.0%, 8=0.1%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.1%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=239544,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=798MiB/s (837MB/s), 798MiB/s-798MiB/s (837MB/s-837MB/s), io=235GiB (252GB), run=301305-301305msec

Disk stats (read/write):
  nvme0n1: ios=256907/65, sectors=495587632/840, merge=8/24, ticks=328812567/42586, in_queue=328869901, util=88.04%
```

### Test read IOPS by performing random reads, using an I/O block size of 4 KB and an I/O depth of at least 256:

```bash
read_iops: (groupid=0, jobs=1): err= 0: pid=5622: Tue Feb 25 19:39:38 2025
  read: IOPS=171k, BW=667MiB/s (700MB/s)(195GiB/300002msec)
    slat (usec): min=3, max=1053, avg=48.74, stdev=20.75
    clat (nsec): min=1836, max=22866k, avg=1446059.32, stdev=1305699.82
     lat (usec): min=124, max=22927, avg=1494.80, stdev=1299.06
    clat percentiles (usec):
     |  1.00th=[  149],  5.00th=[  194], 10.00th=[  219], 20.00th=[  310],
     | 30.00th=[  510], 40.00th=[  766], 50.00th=[ 1074], 60.00th=[ 1418],
     | 70.00th=[ 1827], 80.00th=[ 2343], 90.00th=[ 3261], 95.00th=[ 4113],
     | 99.00th=[ 5669], 99.50th=[ 6325], 99.90th=[ 8291], 99.95th=[ 9110],
     | 99.99th=[11207]
   bw (  KiB/s): min=390864, max=805992, per=100.00%, avg=683445.49, stdev=181832.96, samples=600
   iops        : min=97716, max=201498, avg=170861.23, stdev=45458.22, samples=600
  lat (usec)   : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.01%, 50=0.01%
  lat (usec)   : 100=0.01%, 250=14.42%, 500=15.19%, 750=9.92%, 1000=8.40%
  lat (msec)   : 2=25.62%, 4=20.99%, 10=5.44%, 20=0.03%, 50=0.01%
  cpu          : usr=12.31%, sys=76.48%, ctx=1511872, majf=0, minf=36
  IO depths    : 1=0.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=100.0%
     submit    : 0=0.0%, 4=54.0%, 8=3.8%, 16=41.6%, 32=0.6%, 64=0.1%, >=64=0.1%
     complete  : 0=0.0%, 4=53.6%, 8=2.5%, 16=43.3%, 32=0.6%, 64=0.1%, >=64=0.1%
     issued rwts: total=51236327,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=256

Run status group 0 (all jobs):
   READ: bw=667MiB/s (700MB/s), 667MiB/s-667MiB/s (700MB/s-700MB/s), io=195GiB (210GB), run=300002-300002msec

Disk stats (read/write):
  nvme0n1: ios=51592006/54, sectors=412778848/664, merge=5350/21, ticks=75346489/39, in_queue=75346548, util=94.40%
```

## Other Links

* Hardware breakdown by Chips and Cheese: <https://chipsandcheese.com/p/the-nerfed-fpu-in-ps5s-zen-2-cores>
