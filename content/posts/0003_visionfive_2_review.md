+++ 
draft = false
date = 2023-02-22T12:36:00+05:30
title = "My review of the VisionFive 2"
description = "A potential Raspberry Pi competitor, for a developer"
slug = "visionfive-2-initial-review"
externalLink = ""
series = []
+++


Since a long time, RISC-V was in my news feed. I have no idea about RISC-V
other than that it is an open source ISA (not implementation). That made me
curious and I wanted to get something RISC-y to understand the hype first-hand.

In August of 2022, I came across the Kickstarter announcement from StarFive
([here](https://www.kickstarter.com/projects/starfive/visionfive-2)) about the
VisionFive 2. Following hardware features peaked my interest the *most*:

  - Dual Gigabit Ethernet ports (some _Super early bird_ models have 1x 1000M
  and 1x 100M ports)
  - Presence of QSPI flash for `u-boot`
  - The SBC was available with 8 GB of RAM (that is the minimum for me)

The VisionFive 2 also has one M.2 slot for an NVMe drive, but it doesn't
interest me as much. Particularly because it only has the bandwidth of **1x
PCIe 2.0** lane. Also because I can not afford a _"good"_ NVMe drive at the
moment.

I don't want to put in a cheap NVMe drive because the cheap ones usually omit
DRAM. That isn't a problem on a moderately fast computer. But on a computer
whose CPU (SiFive U74) [allegedly] performs similarly to an ARM A55 core, said
latency hit is more noticeable.


# Initial software setup

_For now, one is expected to form their opinions on hardware performance based
on the [Debian image provided the vendor](https://debian.starfivetech.com/)._

**Before you flash the image provided by the vendor, the board firmware
_needs_ to be updated. Please do that first.**


## Updating firmware

The firmware can be easily updated by following these steps:

  1. Download 3 assets from VisionFive 2's
  [latest SDK release](https://github.com/starfive-tech/VisionFive2/releases/latest):
  `sdcard.img`, `u-boot-spl.bin.normal.out`, `visionfive2_fw_payload.img`
  2. `sudo dd if=sdcard.img conv=sync status=progress bs=1M of=/dev/<YOUR_DEV_HERE>`
  3. `mkdir temp-dir`
  4. `sudo mount /dev/<YOUR_DEV_HERE>4 temp-dir`
  5. `sudo cp u-boot-spl.bin.normal.out visionfive2_fw_payload.img temp-dir/root/`
  6. `sudo umount /dev/<YOUR_DEV_HERE>4`
  7. Eject the SD Card from your computer, insert it in VisionFive 2 and power
  it up. The green LED should start blinking.
  8. Plug the network cable on the Ethernet port that is next to the HDMI port.
  9. `ssh root@<IP_ADDRESS>` (passwd: `starfive`)
  10. Run the command `cat /proc/mtd` and you should have the following output:
  ```
  dev: size erasesize name
  mtd0: 00020000 00001000 "spl"
  mtd1: 00300000 00001000 "uboot"
  mtd2: 00100000 00001000 "data"
  ```
  11. If the partition information is correct, update the `spl` and `uboot`
  firmware using the following commands:
  ```
  flashcp -v u-boot-spl.bin.normal.out /dev/mtd0
  flashcp -v visionfive2_fw_payload.img /dev/mtd1
  ```

Done! Now `systemctl poweroff` and flash the vendor's Debian image to your SD
card and boot it up.


## Setup with vendor's image

Although the SD Card image provided by the vendor worked fine for me, people
have reported two major things missing from their kernel. The modules for
BTRFS are not built and IPv6 isn't supported either. We will compile the 
kernel soon, but there are other problems with the image that need to be
tackled first.


### Manually expand the root partition

Unlike most images available for the Raspberry Pi, this image does not
automatically expand the root partition. So first, resize your `/` partition
using `parted`:

```bash
root@starfive:~# parted /dev/mmcblk1
GNU Parted 3.5
Using /dev/mmcblk1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) resizepart 3 100%
Warning: Partition /dev/mmcblk1p3 is being used. Are you sure you want to
continue?
Yes/No? Y
(parted) q
Information: You may need to update /etc/fstab.
```

Resize the filesystem using `resize2fs`:

```bash
root@starfive:~# resize2fs /dev/mmcblk1p3
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /d[ 192.744328] EXT4-fs (mmcblk1p3): resizing filesystem from 1280507 to
31186944 blocks
ev/mmcblk1p3 is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 15
[ 196.934822] EXT4-fs (mmcblk1p3): resized filesystem to 31186944
The filesystem on /dev/mmcblk1p3 is now 31186944 (4k) blocks long.
```

Verify the change using the 'df' command. Best reboot now to prevent any
soft-errors.


### Update Debian keyring

To run `apt update` without any errors, the Debian keyring needs to be updated.
This can be easily remedied by manually downloading the `.deb` file for the
Debian keyring package from [here](https://packages.debian.org/sid/all/debian-ports-archive-keyring/download) (don't worry, it is architecture agnostic).

Choose your nearest mirror and download it. Then, like any other package, do a
`dpkg -i debian-ports-archive-keyring*.deb`.

Now, you can `apt update` flawlessly :wink:


### OPTIONAL: Update APT sources

**Please note that there is a reason why the APT sources point to a snapshot.
That is because it is a "known good" state of the packages. I am not
accountable if your system breaks. I am not a sysadmin.**

But this is what I have done to make sure that I stay up-to-date with any
developments in the ecosystem.

```bash
root@starfive:~# cat <<EOF > /etc/apt/sources.list
deb http://deb.debian.org/debian-ports sid main
deb http://deb.debian.org/debian-ports unreleased main
deb-src http://deb.debian.org/debian sid main
EOF

root@starfive:~# apt update
```


### MISC

Better do some housekeeping now...

```bash
root@starfive:~# passwd # use a strong password
root@starfive:~# userdel -r user
root@starfive:~# useradd -m -G <GROUPS> -s /bin/bash <USER_NAME>
root@starfive:~# visudo # I enabled 'NOPASSWD' for my user
```


# Software status of the VisionFive 2

Other than the minor inconvenience of updating the board firmware, setting
up the vendor's Debian image, everything else seems to mostly work for me.

As new as this SBC is, there already are 5 total offerings for compatible
images.

  1. The vendor provided [Debian Sid image](https://debian.starfivetech.com)
  2. An "experimental" [Debian Sid image](https://forum.rvspace.org/t/experimental-debian-sid-image/1517)
  3. An [Arch Linux image](https://forum.rvspace.org/t/arch-linux-image-for-visionfive-2/1459)
  4. [Build your own Debian images](https://forum.rvspace.org/t/build-your-own-debain-images/1881)
  5. A community image of [OpenSUSE Tumbleweed](https://en.opensuse.org/HCL:VisionFive2)

The OpenSUSE image, at the time of writing this, _seems_ to be using the latest
Linux kernel that is available (6.2-rc8) and adding StarFive's patches that are
for the VisionFive 2. Every other image listed above is using the [StarFive 5.15.0 kernel](https://github.com/starfive-tech/linux/tree/JH7110_VisionFive2_devel).

I have tested images #1, #2 and #3 and I can confirm that the following things
are working as **_I_** expected:

  - The hardware reset switch works
  - The GPIO pins work[^1] (along with UART)
  - Both of the Gigabit Ethernet ports on my board work at the max speed[^2]
  - All 4 of the front USB 3.0 ports work (didn’t test speed)
  - HDMI port works[^3]

[^1]: I haven't tested _all the pins_. I used pins 4, 6, 8, 10, and 14. These
work without any issue so I assume all 40 pins work as intended.

[^2]: You can get 948-ish MBit/s of throughput from each port, which lines up
with real world performance of other Gigabit NICs. But, sending traffic in from
one port and receiving it from another port is limited to 500-ish MBit/s of
throughput due to the way the PHYs are wired ("multiplexed").

[^3]: I don't have a use for display out (on the VF2). I only plugged it in and
booted the SBC up. For what it is worth, the boot logs show up when connected
to my 2160p monitor. **No further graphics testing was performed**, since that
is not my primary aim with this SBC.

{{< notice info >}}
No offence, but I won't be trying out the OpenSUSE image anytime soon. I
despise camel case in a _package manager_ of all things. Firefox is called
`MozillaFirefox`. **WHY?!**
{{</ notice >}}

I am daily driving cwt's Arch Linux image. The kernel in this image is the
vendor's 5.15.0 Linux kernel. This kernel was compiled to enable the support
for IPv6 and BTRFS.

The only modification that I had to perform was to add the following line to
the `/etc/pacman.conf` file:

```
IgnorePkg = linux-api-headers
```

I had to do this because the Linux kernel installed is the vendor's 5.15.0
kernel and the version of the `linux-api-headers` package is `6.1.9`.

{{< notice warning >}}
This image has an issue with **_rebooting_**. When you try to reboot, the board
just shuts down. Though, I am looking into fixing this issue.
{{</ notice >}}

---

People on the RVSpace forum reported that the XU4 fan by Hardkernel fits on
their board ([buy here](https://www.hardkernel.com/shop/cooling-fan-xu4-blue/)).
So I ordered one. **The fan header on the fan and on the board are
incompatible.** So I removed the connector from the fan and soldered the wires
to the jumper wires and inserted it into my GPIO pins. It works now :smile:

With the fan on, and compiling the StarFive's Linux tree for the VisionFive 2
using `make all -j4`, I never saw the temps go above 42 C. This is quite an 
achievement because I am in India (it's very hot here) and in a room that is
without an AC.

![hello world](/static/images/visionfive_2_review_btop_screenshot.png)

The compilation of StarFive's VisionFive 2 Linux kernel tree using the 
following command completed in 2 hours and 15 minutes. That's _really quick_
for such a machine!

```bash
make clean
make mrproper
make starfive_visionfive2_defconfig
ARCH=riscv CFLAGS="-march=rv64imafdc_zicsr_zba_zbb -mcpu=sifive-u74 -mtune=sifive-7-series -O2 -pipe" make all -j4
```

{{< notice warning >}}
You will need to patch the file `arch/riscv/Makefile` to compile the kernel
successfully. Here is the [patch](https://github.com/hexdump0815/linux-starfive-visionfive2-kernel/blob/main/misc.vf2/patches/make-newer-binutils-work.patch).
{{</ notice >}}


# My thoughts so far

Since the RISC-V ISA is pretty new, there are still some packages that have
not been ported yet. As a Neovim user, the biggest hit I received was when
I noticed that RISC-V support from **upstream** LuaJIT is missing. Though, that
is [being worked on](https://github.com/LuaJIT/LuaJIT/issues/628).

The Arch Linux maintainer [felixonmars](https://github.com/felixonmars)--who is
also porting a lot of packages to RISC-V--has made the Neovim package depend on
`lua51` instead of `luajit` ([relevant git commit](https://github.com/felixonmars/archriscv-packages/commit/7b7f04a28cbe6a04f663a14c914ba4d63b081ede)).
Neovim wouldn't even install on Debian due to the missing dependency of LuaJIT,
but on Arch Linux, I can at least install and use Neovim, albeit without the
traditional Lua support.

I have installed the packages that I have listed below. I haven't tested _all_
of them, but of what I did test, they work as I expect them to do on my
Raspbery Pi or on my x86 computer.

```
android-tools arch-install-scripts aria2 bandwhich base base-devel bat btop cargo-audit cargo-auditable cargo-bloat cargo-depgraph cargo-outdated cargo-spellcheck cargo-watch choose chrony cifs-utils dhcpcd dog dua-cli dust exa fd figlet firewalld gcc git git-lfs groff hd-idle hdparm htop iotop iperf iperf3 linux-firmware lsb-release lsof man-db man-pages mkinitcpio mlocate namcap nano neofetch neovim networkmanager nfs-utils nload nvme-cli opendoas openssh parted paru ripgrep rsync rustup skim smartmontools sudo tealdeer tmux tre tree unrar unzip usbutils wget which wireguard-tools wireless-regdb wol xmlto yt-dlp zsh zsh-autosuggestions zsh-completions zsh-syntax-highlighting
```

{{< notice note >}}
**My current workflow does not involve making use of the iGPU in any capacity
whatsoever.** Hence I will not make any _personal comments_ on it yet. But,
people have reported some issues with the display output when the SBC is
connected to a 2160p monitor. On the first day, when I couldn't find my UART
cable, I had to rely on the display output of the board. So I connected the
HDMI port to my **2160p** monitor, and, for what it is worth, it at least
showed me the boot logs. Though, I did see a black screen instead of the login
manager. I haven't done any further testing in this area since. 
{{< /notice >}}

---

The state of hardware on the SBC seems good enough for my use: as a development
machine and to build software for RISC-V.

StarFive is [upstreaming](https://rvspace.org/en/project/JH7110_Upstream_Plan)
software components. The only thing StarFive is not upstreaming is the GPU
drivers. That commitment is the responsibility of Imagination, as per their
[post](https://developer.imaginationtech.com/open-source-gpu-driver/).

As of now, I have no gripes as the vendor is not withholding any patches; of
what is available, works, the vendor is making an effort to upstream as much as
they can.

To repeat myself for the _n<sup>th</sup>_ time, of what packages are available,
they work as one might expect from a PC/laptop. Having such a low cost device
with minor issues, like the display output not working as intended at 2160p
resolution, (**in the context of a developer/porting machine**) is a _STEAL_!

I am very happy with it and I will be using this to learn Linux kernel
development (using :crab:) and help port more ARM64/AMD64 software to it. Let
me know if you want Rocky Linux on this! :wink:
