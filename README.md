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

## Pinouts of nonstandard connectors

```bash
G = GND
x = NC?

J4003 (near power connectors)
[ G ? ? ? ? ? ?   ]
[ G ? ? ? ? ? G ? ]

J2000 and J2001 (next to J1000, a GPU power connector)
[ ? ? ? ? ]   [ ? ? ? ? ]
[ x G G G ]   [ G G G G ]

J4000 and Speaker/J5 (near front+bottom, a populated and unpopulated connector)
[ G ? ? ? ]     ? x x x
[ ? ? ?   ]     ? G x x 

J2 (backside, unpopulated, near front)
? ? ? ? ? ? x ? x x
? G G G ? ? ? ? G ?
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
