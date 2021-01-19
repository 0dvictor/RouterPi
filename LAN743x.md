# LAN 7430 and LAN 7431 Bring-Up

LAN 7430 and LAN 7431 seems to be good choices to build a router with Raspberry Pi Compute Module 4.

## Performance

### Direct Connection

```
+---------+---------+        +------------+        +----------+
| RPi CM4 | LAN7430 | <====> | GbE Switch | <====> | Computer |
+---------+---------+        +------------+        +----------+
```

On the Pi:

```
iperf3 -s --bind <IP of PI>
```

On the Computer:

```
iperf3 -c <IP of PI>

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.08 GBytes   927 Mbits/sec  571             sender
[  5]   0.00-10.03  sec  1.08 GBytes   921 Mbits/sec                  receiver
```
```
iperf3 -c <IP of PI> -R

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.03  sec  1.09 GBytes   934 Mbits/sec    1             sender
[  5]   0.00-10.00  sec  1.09 GBytes   934 Mbits/sec                  receiver
```
```
iperf3 -c <IP of PI> --bidir

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID][Role] Interval           Transfer     Bitrate         Retr
[  5][TX-C]   0.00-10.00  sec   475 MBytes   399 Mbits/sec   72             sender
[  5][TX-C]   0.00-10.03  sec   473 MBytes   396 Mbits/sec                  receiver
[  7][RX-C]   0.00-10.00  sec   932 MBytes   782 Mbits/sec    1             sender
[  7][RX-C]   0.00-10.03  sec   929 MBytes   778 Mbits/sec                  receiver
```

### Via WireGuard

```
+---------+---------+        +------------+        +----------+
| RPi CM4 | LAN7430 | <====> | GbE Switch | <====> | Computer |
+---------+---------+        +------------+        +----------+
                ^                                    ^
                +------------ WireGuard --------------
```


On the Pi:

```
iperf3 -s --bind <IP of PI>
```

On the Computer:

```
iperf3 -c <IP of PI>

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   620 MBytes   520 Mbits/sec  324             sender
[  5]   0.00-10.00  sec   619 MBytes   519 Mbits/sec                  receiver
```
```
iperf3 -c <IP of PI> -R

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   674 MBytes   565 Mbits/sec  589             sender
[  5]   0.00-10.00  sec   670 MBytes   562 Mbits/sec                  receiver
```
```
iperf3 -c <IP of PI> --bidir

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID][Role] Interval           Transfer     Bitrate         Retr
[  5][TX-C]   0.00-10.00  sec   287 MBytes   241 Mbits/sec  334             sender
[  5][TX-C]   0.00-10.00  sec   287 MBytes   240 Mbits/sec                  receiver
[  7][RX-C]   0.00-10.00  sec   492 MBytes   413 Mbits/sec  626             sender
[  7][RX-C]   0.00-10.00  sec   489 MBytes   410 Mbits/sec                  receiver
```

### NAT

```
+--------+        +--------------+---------+---------+        +------------+        +--------+
| Client | <====> | Ob-board Eth | RPi CM4 | LAN7430 | <====> | GbE Switch | <====> | Server |
+--------+        +--------------+---------+---------+        +------------+        +--------+
```

On the Server:

```
iperf3 -s --bind <IP of Server>
```

On the Client:

```
iperf3 -c <IP of Server>

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   806 MBytes   676 Mbits/sec  753             sender
[  5]   0.00-10.00  sec   803 MBytes   674 Mbits/sec                  receiver
```
```
iperf3 -c <IP of Server> -R

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   815 MBytes   684 Mbits/sec  176             sender
[  5]   0.00-10.00  sec   812 MBytes   681 Mbits/sec                  receiver
```
```
iperf3 -c <IP of Server> --bidir

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID][Role] Interval           Transfer     Bitrate         Retr
[  5][TX-C]   0.00-10.00  sec   633 MBytes   531 Mbits/sec   85             sender
[  5][TX-C]   0.00-10.00  sec   629 MBytes   528 Mbits/sec                  receiver
[  7][RX-C]   0.00-10.00  sec   330 MBytes   277 Mbits/sec  295             sender
[  7][RX-C]   0.00-10.00  sec   329 MBytes   276 Mbits/sec                  receiver
```

## Hardware

- [Raspberry Pi Compute Module 4](https://www.raspberrypi.org/products/compute-module-4/) - 2GM RAM, 8GM eMMC
- [LAN7430 Evaluation Board](https://www.microchip.com/Developmenttools/ProductDetails/EVB-LAN7430)

## Software

- [Ubuntu Server 20.10](https://ubuntu.com/download/raspberry-pi)
  - LAN743x driver is not enabled, and hence requiring extra work.

### Setup Build Environment

Follow https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel, but I had to use this command to obtain the source code:
```
apt source linux-image-$(uname -r)
```

### Compile LAN7430x driver

1. Modify `Makefile` to "hack" the version signature:
```
diff --git a/Makefile b/Makefile
index 516f2a016..55371d1e0 100644
--- a/Makefile
+++ b/Makefile
@@ -1,8 +1,8 @@
 # SPDX-License-Identifier: GPL-2.0
 VERSION = 5
 PATCHLEVEL = 8
-SUBLEVEL = 18
-EXTRAVERSION =
+SUBLEVEL = 0
+EXTRAVERSION = -1011-raspi
 NAME = Kleptomaniac Octopus

 # *DOCUMENTATION*
```
2. Avoid having a traililng '+' in the version signature:
```
touch ./.scmversion
```
3. Obtain ".config" and "Module.symvers"
```
cp /boot/config-$(uname -r) ./.config
cp /usr/src/linux-headers-$(uname -r)/Module.symvers ./
```
4. Prepare config file => Change "LAN743x support" to "M"
```
make oldconfig
make menuconfig
```
5. Cherry-pick bug fixes
    - LAN743x driver from Ubuntu 20.10's source is very buggy.
    - I had to pull in all fixes from [the kernel upstream](https://github.com/torvalds/linux/tree/master/drivers/net/ethernet/microchip).
6. Compile the module
```
make scripts
make prepare
make modules_prepare
make -C . M=./drivers/net/ethernet/microchip/ modules
```
7. Install the module
```
sudo insmod ./drivers/net/ethernet/microchip/lan743x.ko
```

## FAQ

### What about LAN7431?

With the [LAN7431 Evaluation Board](https://www.microchip.com/Developmenttools/ProductDetails/EVB-LAN7431), it functionally works. I do not have a GbE PHY to test its performance.

### Why Ubuntu Server 20.10, not Raspberry Pi OS?

No particular reason, the goal is to make sure LAN743x works as expected before starting my design. My Raspberry Pi 4 (as my router) runs Ubuntu, so I just picked the same.
