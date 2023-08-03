Disk Partitioning
We are going to cover two possible disk partitioning schemes:

[CaseA] - "Almost-Full" disk encryption, faster boot time

 /dev/sdX - physical disk with MBR partition table
 /dev/sdX1 - /boot unencrypted partition of 1 GB size
 /dev/sdX2 - encrypted with LUKS (Linux Unified Key Setup) and partitioned into a LVM (Logical Volume Manager) container
 |---> Logical volume 1 - /dev/mapper/lvm-volSwap - swap partition, the size of which is >= size of your RAM (i.e. 16 GB)
 |---> Logical volume 2 - /dev/mapper/lvm-volRoot - / root partition, which gets 100% of remaining free space
[/CaseA]

[CaseB] - Full disk encryption, slower boot time

 /dev/sdX - physical disk with MBR partition table
 /dev/sdX1 - encrypted with LUKS (Linux Unified Key Setup) and partitioned into a LVM (Logical Volume Manager) container
 |---> Logical volume 1 - /dev/mapper/lvm-volBoot - /boot encrypted partition of 1 GB size
 |---> Logical volume 2 - /dev/mapper/lvm-volSwap - swap partition, the size of which is >= size of your RAM (i.e. 16 GB)
 |---> Logical volume 3 - /dev/mapper/lvm-volRoot - / root partition, which gets 100% of remaining free space
[/CaseB]

Choose a case based on your speed/security preferences. The case-specific commands below - will be marked as [CaseA] or [CaseB] respectively!

Erase a Disk
Learn the X letter of your desired target installation drive:

 parted -l
Print its' partition table with

 parted -s /dev/sdX print
Ensure that there is nothing important on this drive, then erase its' partition table and overwrite all its' contents with

 dd bs=4096 if=/dev/urandom iflag=nocache of=/dev/sdX oflag=direct status=progress || true
For security, WAIT until this lengthy process completes: otherwise, if you interrupt it with Ctrl+C/Z, the not-overwritten data will be possible to restore. Then, run

 sync
to flush the disk operations. Also, it is recommended to reboot after doing this, and launch a Terminal with sudo su again.

NOTE: to be able to run the commands below by copy-pasting - without manually replacing X each time - you may create the symbolic links:

 ln -s /dev/sd? /dev/sdX
 ln -s /dev/sd?1 /dev/sdX1
 ln -s /dev/sd?2 /dev/sdX2
where ? should be substituted with a letter of your desired target installation drive.

Now, you can either continue below, or - if you are already familiar with this manual - just jump straight to Quick command summary

Create the Partitions
Create a new MBR partition table:

 parted -s /dev/sdX mklabel msdos
[CaseA] - "Almost-full" disk encryption, faster boot time

Set up a /dev/sdX1 partition for /boot - 1 GB should be enough - and set a boot flag:

 parted -s -a optimal /dev/sdX mkpart "primary" "fat16" "0%" "1024MiB"
 parted -s /dev/sdX set 1 boot on
Make a /dev/sdX2 partition which will take the rest of free space - after 1 GB of /boot - and set a lvm flag:

 parted -s -a optimal /dev/sdX mkpart "primary" "ext4" "1024MiB" "100%"
 parted -s /dev/sdX set 2 lvm on
Print the partition table of a drive and see if the alignment of your partitions is optimal:

 parted -s /dev/sdX print
 parted -s /dev/sdX align-check optimal 1
 parted -s /dev/sdX align-check optimal 2
[/CaseA]

[CaseB] - Full disk encryption, slower boot time

Make a /dev/sdX1 partition which will take the whole space, and set the boot and lvm flags:

 parted -s -a optimal /dev/sdX mkpart "primary" "ext4" "0%" "100%"
 parted -s /dev/sdX set 1 boot on
 parted -s /dev/sdX set 1 lvm on
Print the partition table of a drive and see if the alignment of your partition is optimal:

 parted -s /dev/sdX print
 parted -s /dev/sdX align-check optimal 1
[/CaseB]

Setup the Logical Volumes
The disk encryption will utilize the Linux Unified Key Setup (LUKS), which is now part of an enhanced version of cryptsetup, using dm-crypt (device-mapper crypt) as the disk encryption backend.

To force loading the Linux kernel modules related to Serpent and other strong encryptions from your LiveCD/LiveUSB, run

 cryptsetup benchmark
and, after it completes, use a command like

 # [CaseA]
 cryptsetup --verbose --type luks1 --cipher serpent-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 --use-random --verify-passphrase luksFormat /dev/sdX2
 # [CaseB]
 cryptsetup --verbose --type luks1 --cipher serpent-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 --use-random --verify-passphrase luksFormat /dev/sdX1
to create and format the LUKS partition (--type luks1 because of GRUB limitations) with your custom encryption flags. Open and mount it using the device mapper - into i.e. lvm-system :

 # [CaseA]
 cryptsetup luksOpen /dev/sdX2 lvm-system
 # [CaseB]
 cryptsetup luksOpen /dev/sdX1 lvm-system
Note: later you will encounter the following warnings - they happen because /run is not available inside the chroot - so you can ignore them:

 WARNING: Failed to connect to lvmetad. Falling back to device scanning.
 /run/lvm/lvmetad.socket: connect failed: No such file or directory
 WARNING: failed to connect to lvmetad: No such file or directory. Falling back to internal scanning.
Now it is possible to create a physical volume using the Logical Volume Manager (LVM) and the previously used id lvm-system as follows:

 pvcreate /dev/mapper/lvm-system
Having the physical volume, it is possible to create a logical volume group named lvmSystem as follows:

 vgcreate lvmSystem /dev/mapper/lvm-system
And having the logical volume group, the logical volumes can be created as follows:

 # [CaseA] 16GB for swap (volSwap) and the rest for the root partition (volRoot)
 lvcreate --contiguous y --size 16G lvmSystem --name volSwap
 lvcreate --contiguous y --extents +100%FREE lvmSystem --name volRoot
 # [CaseB] Similar, but also a boot partition (volBoot) - for which 1 GB should be enough
 lvcreate --contiguous y --size 1G lvmSystem --name volBoot
 lvcreate --contiguous y --size 16G lvmSystem --name volSwap
 lvcreate --contiguous y --extents +100%FREE lvmSystem --name volRoot
Format the Partitions
Having all physical and virtual disk partitions ready, now it is possible to format them.

Format a boot partition with

 # [CaseA]
 mkfs.fat -n BOOT /dev/sdX1
 # [CaseB]
 mkfs.fat -n BOOT /dev/lvmSystem/volBoot
Format a swap partition with

 mkswap -L SWAP /dev/lvmSystem/volSwap
This command will print a message like

 # Setting up swapspace version 1, size = 16 GiB (17179865088 bytes)
 # no label, UUID=6955244c-c72a-4dec-8dee-079ec743a818
Copy your swap UUID somewhere - you will need it later.

Format a root partition with

 mkfs.ext4 -L ROOT /dev/lvmSystem/volRoot
Mount the Partitions
Having each partition formatted, they can be mounted as follows:

 swapon /dev/lvmSystem/volSwap
 mount /dev/lvmSystem/volRoot /mnt
 mkdir /mnt/boot
 # [CaseA]
 mount /dev/sdX1 /mnt/boot
 # [CaseB]
 mount /dev/lvmSystem/volBoot /mnt/boot
Artix Installation
Avoid the Calamares Bugs
Calamares - graphical Artix Linux installer - helps to configure everything much faster. However, it has some critical bugs, which are active for some sophisticated partition schemes like ours. To ensure the successful Artix Linux installation with Calamares, you should do these temporary workarounds:

1) There is a high chance of getting a GRUB-related error

 Boost.Python error in job "bootloader".
at the end of Artix Linux installation. To avoid this error, open a related script with

 nano /usr/lib/calamares/modules/bootloader/main.py
and replace this line near the end of file at def run() function:

     prepare_bootloader(fw_type)
should become

     return None
This is fine, since we are going to install GRUB manually a bit later. Alternatively, you may simply run

 sed -i -e "s/    prepare_bootloader(fw_type)/    return None/g" /usr/lib/calamares/modules/bootloader/main.py
2) Calamares may unmount all the LVM volumes - including the target ones! - at the beginning of installation, causing its' failure. To avoid this problem, you need to somehow block the unmounting of the LVM volumes by putting them into use. The easiest way to do this: open the new Terminal tabs and

 # [CaseA]
 [2nd tab] cd /mnt
 # [CaseB]
 [2nd tab] cd /mnt
 [3rd tab] cd /mnt/boot
Then, you can launch a Calamares installer in the 1st Terminal tab, while the 2nd and 3rd tabs are still open and active.

Install with Calamares
Instead of double-clicking a Calamares shortcut on a Desktop, I recommend you to launch Calamares from a console - to get more logs, which could be really useful if any problems arise. Extract a launch command from this shortcut with

 cat /home/artix/Desktop/calamares.desktop | grep "Exec"
It should look like

 pkexec env DISPLAY=:0 XAUTHORITY=/home/artix/.Xauthority QT_QPA_PLATFORMTHEME=gtk2 calamares
however, my command may be outdated, so do NOT just copy-paste it. Extract your own! Then use it. Alternatively, you may simply run

 $(sed -n -e "s/^Exec=//p" /home/artix/Desktop/calamares.desktop)
While installing with Calamares: at "Partitions / Select storage device" screen - choose "lvmSystem (/dev/lvmSystem)" and "Manual partitioning", and at the next screen - set the mount points:

 # [CaseA]
 / for /dev/lvmSystem/volRoot
 # [CaseB]
 / for /dev/lvmSystem/volRoot
 /boot for /dev/lvmSystem/volBoot
"Install bootloader on" option will be ignored thanks to our earlier change of bootloader script. And the "Option to use GPT on BIOS" popup should be closed.

After completing the installation, close Calamares without choosing a "Restart now": we also need to install & configure the packages and a bootloader.

Configure the Packages
fstab
Open /mnt/etc/fstab with

 nano /mnt/etc/fstab
and remove all the uncommented lines - they have been created by the Calamares installer. Alternatively, you may simply run

 sed -i -n -e "/^#/p" /mnt/etc/fstab
Now, generate the new lines with

 fstabgen -U /mnt >> /mnt/etc/fstab
Optionally, all solid-state disk (SSD) mountpoints can be updated with the discard option to enable Continuous TRIM:

 sed -i "s/relatime/relatime,discard/g" /mnt/etc/fstab
However, there are opinions that recommend against TRIM. If in doubt about the hardware, the Periodic TRIM can be applied instead.

As the order of options at the configuration files might change, double check the results of all the sed commands to make sure that they really worked!

tmpfs is a temporary filesystem that resides in memory or swap partitions. Without systemd , only the /run directory uses tmpfs by default:

 findmnt --target /run
 # TARGET SOURCE FSTYPE OPTIONS
 # /run   run    tmpfs  rw,nosuid,nodev,relatime,mode=755,inode64
Optionally, to change the size of the tmpfs partition (e.g. of size 8GB, i.e. half RAM size), open a /mnt/etc/fstab

 nano /mnt/etc/fstab
and insert this line to the end of it, making sure that the TAB whitespace separators have not been converted to the regular spaces (TABs should be everywhere except a space between two last zeroes) and without a front space:

 tmpfs	/tmp	tmpfs	rw,nosuid,nodev,relatime,size=8G,mode=1777	0 0
chroot
Now, it is time to change root (chroot) to the newly installed environment:

 artix-chroot /mnt /bin/bash
Set up a root password with

 passwd
Update the database of packages by running:

 pacman -Sy
locale, timezone, hostname
System-wide locale (e.g. en_US.UTF-8), timezone and hostname - should have been configured by Calamares. Check it by doing

 cat /etc/locale.gen
 ls -l /etc/localtime
 cat /etc/conf.d/hostname
If not configured, configure a locale with

 echo -e "en_US.UTF-8 UTF-8" >> /etc/locale.gen
 locale-gen
 echo "LANG=en_US.UTF-8" > /etc/locale.conf
 export LANG="en_US.UTF-8"
 echo "LC_COLLATE=C" >> /etc/locale.conf
 export LC_COLLATE="C"
a timezone with

 ln -sf /usr/share/zoneinfo/Continent/City /etc/localtime
and a hostname (e.g. 4rt1x) with

 nano /etc/conf.d/hostname
 hostname="4rt1x"
mkinitcpio.conf
The /etc/mkinitcpio.conf file enables to set up various kernel parameters. Within the HOOKS part, the encrypt lvm2 needs to be put between block and filesystems keywords in order to enable the Full Disk Encryption. It may also be useful to include the resume keyword to enable suspend to disk options. However, this may not work at all times, such as with hardened kernels.

As the sed might be unreliable because of the possible changes to the options' order, open /etc/mkinitcpio.conf with

 nano /etc/mkinitcpio.conf
and insert encrypt and resume options manually to the following places:

 HOOKS="base udev autodetect modconf block keyboard keymap consolefont lvm2 filesystems fsck"
should become

 HOOKS="base udev autodetect modconf block encrypt keyboard keymap consolefont lvm2 resume filesystems fsck"
GRUB - Installation
To avoid a GRUB configuration problem described at the end of this post, remove an artix-grub-theme package with its' dependencies:

 pacman -Rc artix-grub-theme
For security, the standard Linux kernel and its' headers - should be replaced by a hardened version:

 pacman -Rc linux linux-headers
 pacman -S linux-hardened linux-hardened-headers
To avoid the cryptsetup installation errors, simultaneously upgrade pacman and openssl and get the legacy openssl:

 pacman -S openssl openssl-1.1 pacman
Now, you could install these packages:

 pacman -S lvm2 cryptsetup nano glibc mkinitcpio
During that, initramfs should be re-generated automatically with the encrypt/resume hooks. If not, re-generate initramfs manually:

 mkinitcpio -p linux-hardened
After that, a grub package could be installed with

 pacman -S grub
GRUB - Configuration
In order for a GRUB to find the LUKS-encrypted partitions, you will need to configure it:

 nano /etc/default/grub
Personally I have changed the following lines - without the front spaces:

1) Added a

 # GRUB boot loader configuration
to the top of a file

2) Increased a GRUB timeout from 3 to 15:

 GRUB_TIMEOUT="15"
3) Expanded a GRUB default command line:

 GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
should become

 GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=UUID=xxx:lvm-system loglevel=3 quiet resume=UUID=yyy net.ifnames=0"
or, if you are using a solid-state disk (SSD) and would like to be able to enable Continuous TRIM on a later step, - with an additional allow-discards parameter:

 GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=UUID=xxx:lvm-system:allow-discards loglevel=3 quiet resume=UUID=yyy net.ifnames=0"
where xxx UUID could be found out with

 # [CaseA]
 blkid -s UUID -o value /dev/sdX2
 # [CaseB]
 blkid -s UUID -o value /dev/sdX1
and yyy UUID - swap UUID - is already known by you from the previous steps.

4) Added

 # Uncomment to enable booting from LUKS encrypted devices
 GRUB_ENABLE_CRYPTODISK="y"

 # Set to 'countdown' or 'hidden' to change timeout behavior,
 # press ESC key to display menu.
 GRUB_TIMEOUT_STYLE="menu"
5) Changed GRUB_GFXMODE:

 GRUB_GFXMODE="1024x768,800x600"
should become

 GRUB_GFXMODE="auto"
6) Moved up the

 GRUB_DISABLE_LINUX_RECOVERY="true"
7) Commented out

 #GRUB_THEME="/usr/share/grub/themes/artix/theme.txt"
,

 #GRUB_SAVEDEFAULT="true"
and added the quote " symbols around the options.

Here is a final /etc/default/grub from artix-xfce-openrc-20220123-x86_64.iso . If this config is not outdated (still good as of 25th Dec 2022) - you could use it as a template, just remember to replace the UUID's with your own:

 swap UUID
- you already wrote it down on the previous steps,

 root UUID
- find it out with

 # [CaseA]
 blkid -s UUID -o value /dev/sdX2
 # [CaseB]
 blkid -s UUID -o value /dev/sdX1
Install these optional dependencies:

 pacman -S dosfstools freetype2 fuse2 gptfdisk libisoburn mtools os-prober
 pacman -S iw memtest86+ wpa_supplicant
 pacman -S device-mapper-openrc lvm2-openrc cryptsetup-openrc
Then, you can install GRUB to MBR and generate its' config:

 grub-install --target=i386-pc --boot-directory=/boot --bootloader-id=artix --recheck /dev/sdX
 grub-mkconfig -o /boot/grub/grub.cfg
Other Packages
In order to decrypt and use the LUKS/LVM volumes, the following services need to be installed and activated:

 rc-update add device-mapper boot
 rc-update add lvm boot
 rc-update add dmcrypt boot
The udev service (eudev/eudev-openrc) should be started by default in the sysinit runlevel. Its activation can be confirmed as follows:

 rc-status sysinit | grep udev
should print this output:

 udev                                                              [  stopped  ]
 udev-trigger                                                      [  stopped  ]
The dbus service should be installed and activated. Should it not, it can be done as follows:

 rc-update add dbus default
The SystemD project’s logind should be installed as part of the base meta package. :-P Should it not be activated, it can be done as follows:

 rc-update add elogind boot
The haveged service is a simple entropy daemon useful for unpredictable random number generation, which can be installed and activated as follows:

 pacman -S haveged haveged-openrc
 rc-update add haveged default
Cron job daemons (cronie, fcron etc.) can be installed and activated as follows (e.g. cronie):

 pacman -S cronie cronie-openrc
 rc-update add cronie default
If Network Manager GUI is the desired choice to manage network interfaces, the following needs to be run in order to install and activate the service:

 pacman -S gcr networkmanager networkmanager-openrc networkmanager-openvpn network-manager-applet
 rc-update add NetworkManager default
NTP, ACPI, Syslog-NG daemons can be installed and activated as follows:

 pacman -S ntp ntp-openrc acpid acpid-openrc syslog-ng syslog-ng-openrc
 rc-update add ntpd default
 rc-update add acpid default
 rc-update add syslog-ng default
Useful packages (will include samba, samba client):

 pacman -S artools bash-completion lsof strace
 pacman -S wget htop mc zip samba unrar p7zip unzip
 pacman -S hdparm smartmontools hwinfo dmidecode
 pacman -S whois rsync nmap inetutils net-tools ndisc6
Exit the chroot and unmount the volumes:

 exit
 umount -R /mnt
 swapoff -a
Flush the disk operations:

 sync
Now a system can be rebooted:

 reboot
First Boot
During the first boot, you might get the following error:

 A password is required to access the lvm-system volume:
 cryptsetup: /usr/lib/libjson-c.so.5: no version information available (required by /usr/lib/libcryptsetup.so.12)
 Enter passphrase for /dev/sdX2:
Ignore this error, enter your passphrase and boot. Then open a console, run

 sudo su
to get the root rights, enable swap

 swapon /dev/lvmSystem/volSwap
and fully upgrade your system with

 pacman -Suy
After the upgrade completes, instead of instantly rebooting, please open mkinitcpio.conf with

 nano /etc/mkinitcpio.conf
and make sure that all the HOOKS - especially encrypt --- have been preserved:

 HOOKS="base udev autodetect modconf block encrypt keyboard keymap consolefont lvm2 resume filesystems fsck"
If not - manually insert it, then run

 mkinitcpio -p linux-hardened
to apply this change. Now you can reboot safely and enjoy your encrypted Artix Linux ;-)

P.S. Currently, the pre-installed Epiphany web browser doesn't work with a linux-hardened kernel, so you may want to replace it with Midori:

 pacman -Rc epiphany
 pacman -S midori
For a better browser, try LibreWolf - a fork of Firefox with improved security.

Troubleshooting
If after a system update, instead of

 A password is required to access the lvm-system volume:
 Enter passphrase for /dev/sdX2:
you are getting

 ERROR: device '/dev/mapper/lvmSystem-volRoot' not found. Skipping fsck.
maybe encrypt has disappeared from HOOKS of mkinitcpio.conf. To fix this, boot from LiveCD/LiveUSB and do these commands:

 sudo su
get root rights,

 cryptsetup benchmark
to force loading the Serpent-related Linux kernel modules,

 # [CaseA]
 cryptsetup luksOpen /dev/sdX2 lvm-system
 mount /dev/lvmSystem/volRoot /mnt
 mount /dev/sdX1 /mnt/boot
 # [CaseB]
 cryptsetup luksOpen /dev/sdX1 lvm-system
 mount /dev/lvmSystem/volRoot /mnt
 mount /dev/lvmSystem/volBoot /mnt/boot
mount the partitions,

 artix-chroot /mnt /bin/bash
enter the chroot ,

 nano /etc/mkinitcpio.conf
insert the "encrypt" to HOOKS ,

 HOOKS="base udev autodetect modconf block encrypt keyboard keymap consolefont lvm2 resume filesystems fsck"
and, finally,

 mkinitcpio -p linux-hardened
to apply this change.

Quick com
