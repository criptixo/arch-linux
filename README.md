# My Gentoo Linux System

<h1 align="center">
<img src="./screenshots/rice.png?raw=true">
</h1>

<p align="center">
    <a href="https://www.meetup.com/pro/umbraco">
        <img src="./screenshots/linux-powered.gif"
             alt="">
    </a>
    <a href="">
        <img src="./screenshots/linx.gif"
             alt="">
    </a>
    <a href="">
        <img src="./screenshots/nclinux.gif"
             alt="">
    </a>
</p>

Hey, this is my tested Step by Step complete gentoo linux desktop install.

key features:

* luks on lvm allow for mounting other encrypted drives
* systemd
* systemd-boot as the bootloader
* silent boot
* optimized

> Tested only on my hardware make adjustments to suit your needs.

**Content**

* [**Gentoo install guide**](#gentoo-install-guide)
  * [**boot gentoo live cd**](#boot-gentoo-live-cd)
  * [**Setting up the Hard Drive**](#setting-up-hard-drive)
    - [**Partitioning**](#partitioning)
    - [**Create LVM**](#create-lvm)
    - [**Create LUKS**](#create-luks)
    - [**Creating File Systems**](#create-file-systems)
    - [**Mount the filesystems**](#mount-the-filesystems)
  * [**Installing the Gentoo base system**](#installing-the-gentoo-base-system)
    - [**Install the stage3 tarball**](#install-the-stage3-tarball)
    - [**Chrooting**](#chrooting)
  * [**configuring portage**](#configuring-portage)
    - [**text**](#text)
    - [**text**](#text)
    - [**choosing-systemd-profile**](#choosing-systemd-profile)
    - [**re-compile**](#re-compile)
  * [**Configuring the base system**](#configuring-the-base-system)
    - [**Timezone/locale/hostname**](#Timezone/locale/hostnamee)
    - [**firmware/microcode**](#firmware/microcode)
    - [**Installing the kernel**](#installing-the-kernel)
        - [**gentoo sources**](#gentoo-sources)
        - [**genkernel**](#genkernel)
        - [**Configuring the modules**](#configuring-the-modules)
    - [**LVM Configuration**](#lvm-configuration)
    - [**Creating-Swap**](#creating-swap)
    - [**fstab**](#fstab)
    - [**crypttab**](#crypttab)
    - [**systemd-boot**](#systemd-boot)
    - [**Efibootmgr**](#efibootmgr)
    - [**passwd**](#passwd)
    - [**install needed software**](#install-needed-software)
  * [**Post Installation**](#post-installation)
    - [**First Boot**](#first-boot)
    - [**Add a user**](#add-a-user)
    - [**Configuring Network**](#configuring-network)
  * [**Sway-DE**](#sway-de)
    - [**installing-sway**](#installin-sway)
    - [**starting-sway**](#starting-sway)
    - [**auto login**](#auto-login)
    - [**audio**](#audio)
  * [**extra configurations**](#extra-configurations)
  * [**extra software**](#extra-software)
    - [**steam**](#steam)
    - [**qutebrowser**](#qutebrowser)
    - [**irssi**](#irssi)
    - [**gimp**](#gimp)
* [**Acknowledgments**](#acknowledgments)


## Boot Gentoo live cd

make sure that we boot on UEFI mode:
```shell
efivar -l
```
## Setting up Hard Drive

### Partitioning
if NVME disk `/dev/nvme0n1`
if SATA disk `/dev/sda/`

```shell
cfdisk /dev/sda
```

### Create LVM 
```shell
pvcreate /dev/sda2
vgcreate vg0 /dev/sda2
lvcreate -l 100%FREE -n troot vg0
```

### Create LUKS
```shell
cryptsetup luksFormat --type luks2 --cipher aes-xts-plain64 --key-size 256 --hash sha256 /dev/vg0/cryptroot
cryptsetup open /dev/vg0/root root
```

### Create filesystems
```shell
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/mapper/root
```
### Mount the filesystems
```shell
mount /dev/mapper/root /mnt/gentoo
mkdir /mnt/gentoo/boot
mount /dev/sda1 /mnt/gentoo/boot
```

## Installing the Gentoo base system
Change into our mounted directory:
```shell
cd /mnt/gentoo
```
Ensure the date and time are set correctly.
```shell
chronyd -q
```

### Install the stage3 tarball

Choose systemd stage3 profile:
```shell 
links https://www.gentoo.org/downloads/#other-arches
```
Unpack the tarball:
```shell
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

### Chrooting
Copy DNS info:
```shell
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

Chroot into the new environment:
```shell
arch-chroot /mnt/gentoo
source /etc/profile
export PS1="(chroot) $PS1"
```

## configuring portage
1.  make.conf
```shell
COMMON_FLAGS="-march=native -O3 -pipe -flto=7"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
MAKEOPTS="-j5 -l5"
USE="icu policykit webp gstreamer minimal screencast opengl openal opus png raw ffmpeg zeroconf tiff man dbus ipv4 vim-syntax lvm vaapi uefi wayland vulkan tray vdpau pulseaudio alsa asm avif bash-completion branding cak egl encode exif flac gif heif jpeg libnotify pgo graphite threads ithreads kms lto mp3 mp4 mpeg ogg x264 -seccomp -telemetry -ipv6"
ACCEPT_LICENSE="*"
L10N="en-US"
LINGUAS="en_US"
VIDEO_CARDS="amdgpu radeonsi"
INPUT_DEVICES="libinput"
CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3"
```
2. package.use/systemd
```shell
sys-apps/systemd 		        cryptsetup boot
```
package.use/util-linux

```shell
sys-apps/util-linux 	        cryptsetup 
```





TO DO:
```shell
emerge -av dev-vcs/git
cd /etc/portage/
rm -rf make.conf package.use package.env package.accept_keywords package.mask
mkdir /etc/porage/env
cd
git clone https://github.com/criptixo/gentoo-linux-desktop
cd gentoo-linux-desktop
mv gentoo-linux-desktop/portage/* /etc/portage/
```

### Choosing systemd profile
update the snapshot with the latest version of the repository:
`emerge-webrsync`

List the available profiles:
```shell
eselect profile list
```

```shell
[..]
[17]  default/linux/amd64/17.1/systemd (stable) *
[..]
```

```shell
eselect profile set 17 # or the number that you have on your list
```

### Re-compile

```shell
emerge --ask --verbose --update --deep --newuse @world
```

## Configuring the base system
In addition to Portage, some other options should be configured.

### Timezone/locale/hostname
setup timezone
```shell
echo Africa/Algiers > /etc/timezone
emerge --config sys-libs/timezone-data
hwclock --systohc
```

setup locale
```shell
nano -w /etc/locale.gen 
locale-gen
eselect locale list
eselect locale set 4
env-update
source /etc/profile
```

setup hostname 
```shell
hostnamectl hostname navi
```

### firmware/microcode
firmware:
```shell
emerge --ask sys-kernel/linux-firmware
```
microcode:

### Installing the kernel

#### Gentoo sources
```shell
emerge --ask sys-kernel/gentoo-sources
```

```shell
eselect kernel list
```

```shell
Available kernel symlink targets:
  [1]   linux-5.15.59-gentoo
```

```shell
eselect kernel set 1
```

#### genkernel

1. Install `genkernel`:

    ```shell
    emerge --ask sys-kernel/genkernel
    ```

2. Genkernel needs the `/boot` entry in the `/etc/fstab` file. So go to the [fstab](#fstab) section now and continue with point 2 when you're done.

3. Edit `/etc/genkernel.conf`:

    ```shell
    nano -w /etc/genkernel.conf
    ```

    Ensure that `LVM` and `LUKS` are set to `yes`; otherwise, the system will not boot. Leave the rest of the options as they are:

    ```shell
    # Add LVM support
    LVM="yes"

    # Add LUKS support
    LUKS="yes"
    ```

4. Once `genkernel` is configured, then run to generate the kernel binary:

    ```shell
    genkernel --loglevel=5 all
    ```

#### Configuring the modules
If we need to auto-load a kernel module each time to system boots, we should specify it in `/etc/conf.d/modules` file.
You can list your available modules with:
```shell
find /lib/modules/<kernel version>/ -type f -iname '*.o' -or -iname '*.ko' | less
```

### LVM Configuration
```shell
emerge --ask sys-fs/lvm2
```

```shell
nano -w /etc/lvm/lvm.conf
```

```shell
volume_list = ["vg0"]
```

### Creating Swap

### Fstab

Before editing `fstab` we need to know which UUID are using our devices inside and outside `lvm` and `luks` volumes:

```shell
blkid /dev/mapper/vg0-root | awk '{print $2}' | sed 's/"//g'
UUID="576e229c-cf68-4010-8d85-ff8149158416"
blkid /dev/mapper/vg0-home | awk '{print $2}' | sed 's/"//g'
UUID="95fa5807-ea57-4cf5-b717-74f4aba190e2"
```

```shell

/dev/mapper/root		/		    ext4		rw		    0 1
/dev/sda1			    /boot		vfat		rw		    0 2
/swapfile			    none 		swap 		sw	        0 0
```


### crypttab
Warning!!! As we don't have encrypted partitions other than root, which the `systemd` must mount before the whole system start, we don't need to set it up there, so our `crypttab` must be empty.

### systemd-boot

```shell
emerge --ask sys-libs/efivar
efivar -l
```

```shell
bootctl --path=/boot install
```

Every time there's a new version of the systemd you should copy the new binaries to that System Partition by running:

```shell
bootctl --path=/boot update

```

Add one entry into bootloader with this options:

```shell
nano -w /boot/loader/entries/gentoo.conf
```

```shell
title gentoo
linux /vmlinuz-6.6.30-gentoo-x86_64
initrd /initramfs-6.6.30-gentoo-x86_64.img
options crypt_root=/dev/mapper/vg0-root root=/dev/mapper/root dolvm quiet loglevel=3 vt.global_cursor_default=0 mitigations=off
```

Edit bootloader config:

```shell
nano -w /boot/loader/loader.conf
```

```shell
timeout 0
```

### Efibootmgr

```shell
emerge --ask sys-boot/efibootmgr
```

To list the current boot entries:

```shell
efibootmgr -v
```

```shell
BootCurrent: 0003
Timeout: 1 seconds
BootOrder: 0003
Boot0000* Linux Boot Manager  HD(1,GPT,3eb8effe-8e1d-4670-987c-9b49b5f605b2,0x800,0x1ff801)/File(\EFI\systemd\systemd-bootx64.efi)
Boot0001* gentoo  HD(1,GPT,02f231b8-8f9a-471c-b3a9-dc7edb1bd70e,0x800,0xee000)/File(\EFI\gentoo\grubx64.efi)
Boot0003* Gentoo Linux  PciRoot(0x0)/Pci(0x1f,0x2)/Sata(2,32768,0)/HD(1,GPT,73f682fe-e07b-4870-be82-d85077f8aaa2,0x800,0x100000)/File(\EFI\systemd\systemd-bootx64.efi)
```

I'm only Gentoo in my system so I don't really need anything but the Gentoo entry so I just delete everything with:

```shell
efibootmgr -b <entry_id> -B
```

### passwd
```shell
passwd
```

### install needed software
```shell
emerge -av net-misc/dhcpcd
```
```shell
exit
reboot
```

## Post-installation

### First Boot
```shell
systemd-machine-id+set
systemd-firstboot --prompt
systemctl preset-all --preset-mode=enable-only
```

### Setting bash

```shell
sys-fs/lvm2 app-shells/bash
```

### Add a user
```shell
useradd -m -G users,wheel,audio,video -s /bin/bash criptixo
passwd criptixo
```

### Setup doas
```shell
emerge -av app-admin/doas
touch /etc/doas.conf
chown -c root:root /etc/doas.conf
chmod -c 0400 /etc/doas.conf
```

```shell 
nvim /etc/doas.conf
permit :wheel
emerge --sync
```

### Configuring Network
```shell
systemctl enable --now dhcpcd.service
```
## sway de

### installing sway

```shell
emerge -av swaybg foot grim slurp terminus-font mako wl-clipboard playerctl bemenu ranger htop neovim zip unzip
```

package.use/sway
```shell
ibs/ncurses		                -minimal 
x11-base/xwayland		        libei
media-libs/libepoxy		        X
media-video/pipewire	        sound-server pipewire-alsa echo-cancel -ffmpeg 
media-libs/vulkan-loader        X layers
media-libs/vulkan-layers	    X
dev-libs/bemenu			        -ncurses
media-libs/libaom		        -examples 
x11-libs/cairo 			        X 
x11-libs/libdrm  		        video_cards_radeon
sys-process/htop		        lm-sensors
media-gfx/imv 			        -X
www-client/qutebrowser 	        qt6 adblock widevine -pdf
dev-python/PyQt6		        qml webchannel
sys-libs/zlib			        minizip
media-libs/libva		        X
x11-libs/libxkbcommon 	        X
media-libs/libglvnd		        X
gui-apps/swaybg		            gdk-pixbuf
gui-wm/sway			            X swaybar
gui-libs/wlroots		        X
media-libs/mesa  		        X
sys-apps/dbus 			        X
```

### Starting sway
nano /usr/bin/start-sway
```shell
#!/bin/sh
export XDG_SESSION_TYPE=wayland
export XDG_SESSION_DESKTOP=sway
export XDG_CURRENT_DESKTOP=sway

# Wayland stuff
export QT_QPA_PLATFORM=wayland
export SDL_VIDEODRIVER=wayland
export _JAVA_AWT_WM_NONREPARENTING=1

# Dark mode
GTK_THEME=Adwaita:dark
GTK2_RC_FILES=/usr/share/themes/Adwaita-dark/gtk-2.0/gtkrc 
QT_STYLE_OVERRIDE=Adwaita-Dark

exec dbus-run-session sway "$@"
```

```shell
chmod +x /usr/bin/start-sway
```
### autologin

```shell
emerge -av gui-libs/greetd`
```
nano /etc/greetd/config.toml

```shell
[terminal]
# The VT to run the greeter on. Can be "next", "current" or a number
# designating the VT.
vt = current
...
[initial_session]
command = "/usr/bin/start-sway"
user = "criptixo"
```
```shell
systemctl enable greetd.service
```

### audio
```shell
emerge -av pipewire libpulse wireplumber pulsemixer
systemctl --user enable --now pipewire-pulse.socket wireplumber.service
```

## extra configurations

## extra software

### steam 
```shell
emerge --ask --noreplace app-eselect/eselect-repository
eselect repository enable steam-overlay
emerge --sync
emerge steam-launcher
```
package.use/steam-launcher
```shell
sys-apps/systemd 		      abi_x86_32
sys-libs/zlib			        abi_x86_32
media-libs/libva		      abi_x86_32 
x11-libs/libX11  		      abi_x86_32
x11-libs/libXau  		      abi_x86_32
x11-libs/libxcb  		      abi_x86_32
x11-libs/libXdmcp  		    abi_x86_32
virtual/opengl  		      abi_x86_32
media-libs/mesa  		      abi_x86_32 
dev-libs/expat  		      abi_x86_32
media-libs/libglvnd     	abi_x86_32
x11-libs/libdrm  		      abi_x86_32
x11-libs/libxshmfence  		abi_x86_32
x11-libs/libXext  		    abi_x86_32
x11-libs/libXxf86vm  		  abi_x86_32
x11-libs/libXfixes  		  abi_x86_32
app-arch/zstd  			      abi_x86_32
sys-devel/llvm  		      abi_x86_32
x11-libs/libXrandr  		  abi_x86_32
x11-libs/libXrender  		  abi_x86_32
dev-libs/libffi  		      abi_x86_32
dev-libs/libxml2  		    abi_x86_32
dev-libs/icu  			      abi_x86_32
sys-libs/gpm  			      abi_x86_32
virtual/libelf  		      abi_x86_32
dev-libs/elfutils  		    abi_x86_32
app-arch/bzip2  		      abi_x86_32
dev-libs/nspr  			      abi_x86_32
dev-libs/nss  			      abi_x86_32
net-libs/libndp  		      abi_x86_32
x11-libs/extest 		      abi_x86_32
dev-libs/libevdev      		abi_x86_32
dev-lang/rust-bin		      abi_x86_32
dev-libs/wayland 		      abi_x86_32
virtual/rust 			        abi_x86_32
x11-libs/libpciaccess 		abi_x86_32
sys-devel/clang 	      	abi_x86_32
media-libs/fontconfig 		abi_x86_32
sys-libs/libudev-compat 	abi_x86_32
media-libs/libpulse 		  abi_x86_32
media-libs/libsndfile 		abi_x86_32
net-libs/libasyncns 	   	abi_x86_32
sys-apps/dbus 		      	abi_x86_32 
dev-libs/glib 			      abi_x86_32
dev-libs/libpcre2 	    	abi_x86_32
sys-apps/util-linux 		  abi_x86_32
media-libs/flac 		      abi_x86_32
media-libs/libogg 		    abi_x86_32
media-libs/libvorbis 		  abi_x86_32
media-libs/opus 		      abi_x86_32
media-sound/lame 		      abi_x86_32
media-sound/mpg123-base 	abi_x86_32
media-libs/freetype 		  abi_x86_32
media-libs/libpng 		    abi_x86_32
virtual/libintl 		      abi_x86_32
virtual/libudev 		      abi_x86_32
sys-apps/systemd-utils 		abi_x86_32
sys-libs/libcap 		      abi_x86_32
sys-libs/pam 			        abi_x86_32
virtual/libiconv 		       abi_x86_32
x11-libs/xcb-util-keysyms 	abi_x86_32
dev-db/sqlite			        abi_x86_32
sys-libs/readline		      abi_x86_32
sys-apps/lm-sensors		    abi_x86_32
dev-libs/libgcrypt		    abi_x86_32
app-arch/lz4			        abi_x86_32
dev-libs/libgpg-error		  abi_x86_32
sys-libs/ncurses		      abi_x86_32
media-sound/apulse		    abi_x86_32
media-libs/alsa-lib		    abi_x86_32
x11-libs/libvdpau 		    abi_x86_32
```
### qutebrowser
```shell
emerge --ask qutebrowser
```
### irssi
```shell
emerge --ask irssi
```
### gimp
```shell
emerge --ask gimp
```
package.use/gimp
```shell
app-text/poppler		cairo
media-libs/gegl			cairo
```

### nicotine+
```shell
emerge -av nicotine+
```

### rtorrent
```shell
emerge -av rtorrent
```




emerge -av vulkan-loader
## Acknowledgments