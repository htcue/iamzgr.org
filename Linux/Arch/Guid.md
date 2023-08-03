# 安装前的准备
### 设置字体
```
setfont ter-132b
```
###  验证引导模式
```
ls /sys/firmware/efi/efivars
```
### 连接到互联网
```
iwctl 
device list 
station wlan0 scan 
station wlan0 get-networks
station wlan0 connect wifi-name
exit 
```
### 

使用 timedatectl(1) 确保系统时间是准确的：

# timedatectl
创建硬盘分区
系统如果识别到计算机的内置硬盘、U盘或者移动硬盘等类型磁盘，就会将其分配为一个块设备，如 /dev/sda、/dev/nvme0n1 或 /dev/mmcblk0。可以使用 lsblk 或者 fdisk 查看：

# fdisk -l(此处为小写字母l)
结果中以 rom、loop 或者 airoot 结尾的设备可以被忽略。

提示：在分区之前，请您检查 NVMe 驱动器和 Advanced Format 硬盘是否使用了最佳逻辑扇区大小。需要注意的是，更改逻辑扇区大小后，可能会导致在Windows系统中出现兼容性问题。
对于一个选定的设备，以下分区是必须要有的：

一个根分区（挂载在 根目录）/；
要在 UEFI 模式中启动，还需要一个 EFI 系统分区。
如果您需要创建多级存储例如 LVM、磁盘加密 或 RAID，请您在这时候完成。

请使用 fdisk 或 parted 修改分区表。例如：

# fdisk /dev/the_disk_to_be_partitioned（要被分区的磁盘）
注意：
如果您想要的磁盘没有显示出来， 确保您的磁盘控制器未处于RAID模式。
如果要启动的磁盘已经有一个EFI系统分区，就不要再新建 EFI 分区了，而是使用现有的EFI分区。
如果文件系统支持，交换空间 可以通过 交换文件 实现。
分区方案示例
对于 UEFI 与 GPT 分区表的磁盘分区方案
挂载点	分区	分区类型	建议大小
/mnt/boot1	/dev/efi_system_partition	EFI 系统分区	至少 300 MiB。如果您打算安装多个内核，那就是至少 1 GiB。
[SWAP]	/dev/swap_partition	Linux swap (交换空间)	大于 512 MiB。或者根据您的计算机的内存大小来决定。
/mnt	/dev/root_partition	Linux x86-64 根目录 (/)	剩余空间
如果使用的引导加载程序能够从根磁盘卷中加载内核和 initramfs 映像，则可以使用其他挂载点（例如 /mnt/efi）。请您参阅引导加载程序中的警告部分。
对于传统 BIOS 与 MBR 分区表的磁盘分区方案
挂载点	分区	分区类型	建议大小
[SWAP]	/dev/swap_partition	Linux swap (交换空间)	大于 512 MiB
/mnt	/dev/root_partition	Linux	剩余空间
另请参阅布局示例。

格式化分区
创建分区后，必须使用合适的文件系统对每个新创建的分区进行格式化。详情请参阅文件系统#创建文件系统。

例如，要在根分区 /dev/root_partition 上创建一个 Ext4 文件系统，请运行：

# mkfs.ext4 /dev/root_partition（根分区）
如果创建了交换分区，请使用 mkswap(8) 将其初始化：

# mkswap /dev/swap_partition（交换空间分区）
注意： 对于堆叠式块设备（stacked block devices）请使用恰当的块设备路径替换上文中的 /dev/*_partition 处。
如果你要创建一个 EFI 系统分区，使用 mkfs.fat(8) 将其格式化为 Fat32。

警告： 只有在分区步骤中创建 EFI 系统分区时才需要格式化。如果这个磁盘上已经有一个 EFI 系统分区了，将它重新格式化会破坏其他已安装操作系统的引导加载程序。
# mkfs.fat -F 32 /dev/efi_system_partition（EFI 系统分区）
挂载分区
将根磁盘卷挂载到 /mnt，例如：

# mount /dev/root_partition（根分区） /mnt
然后使用 mkdir(1) 创建其他剩余的挂载点（比如 /mnt/boot）并按层级顺序挂载其相应的磁盘卷。

提示：使用 --mkdir 选项运行 mount(8) 来创建指定的挂载点。或者，先使用 mkdir(1) 创建挂载点再挂载。
注意： 挂载分区一定要遵循顺序，先挂载根（root）分区（到 /mnt），再挂载引导（boot）分区（到 /mnt/boot 或 /mnt/efi，如果单独分出来了的话），最后再挂载其他分区。否则您可能遇到安装完成后无法启动系统的问题。参见 en:Talk:Installation guide#Clarify root mount。
对于 UEFI 系统，挂载 EFI 系统分区：

# mount --mkdir /dev/efi_system_partition（EFI 系统分区） /mnt/boot
如果创建了交换空间卷，请使用 swapon(8) 启用它：

# swapon /dev/swap_partition（交换空间分区）
稍后 genfstab(8) 将自动检测挂载的文件系统和交换空间。

开始安装系统
选择镜像站
系统的文件 /etc/pacman.d/mirrorlist 中定义了软件包会从哪个镜像站下载。在 LiveCD 启动的系统上，且在连接到互联网后，reflector 会通过选择 20 个最新同步的 HTTPS 镜像站并按下载速率对其进行排序来更新镜像列表（由于只考虑最新的20个镜像站，其结果大多数时候都不怎么好用）。

在列表中，越靠前的镜像站在下载软件包时，就会有越高的优先级。请您检查 /etc/pacman.d/mirrorlist 文件，看看列出的镜像站的顺序是否合适。如果不合适，可以手动修改，将离您所处地理位置最近的镜像挪到文件的头部，同时也应该考虑一些其他标准。

要获取 pacman-mirrorlist包 的按国家分列的原始镜像列表，在挑选了能用的镜像之后，可以执行

# pacman -Sy pacman-mirrorlist
再将 /etc/pacman.d/mirrorlist.pacnew 复制到 /etc/pacman.d/mirrorlist 并进行编辑。

这个文件接下来还会被 pacstrap 拷贝到新系统里，所以请您确保设置正确。

安装必需的软件包
注意： 除了 /etc/pacman.d/mirrorlist 之外的软件或配置不会从 Live 环境传递到安装的系统中。
使用 pacstrap(8) 脚本，安装 base包 软件包和 Linux 内核以及常规硬件的固件：

# pacstrap -K /mnt base linux linux-firmware
这时候可以同时额外安装计算机的 CPU 微码包。如果计算机是 Intel 的 CPU ，使用 intel-ucode包，AMD CPU 则使用 amd-ucode包。也可以暂时都不安装，等到进入系统后再安装。

提示：
可以将 linux包 替换为内核页面中介绍的其他内核软件包。
在虚拟机或容器中安装时，可以不安装固件与微码包。
新安装的系统中是没有文本编辑器的，所以请您先安装文本编辑器如 nano包 或者 vim包 等。
base包 软件包并没有包含 Live 环境中的全部程序。因此要获得一个功能齐全的基本系统，可能需要安装更多软件包。要安装其他软件包或软件包组（比如 base-devel包组），请将它们的名字追加到下面的 pacstrap 命令后（用空格分隔），或者也可以在 Chroot 进入新系统后使用 pacman 手动安装。特别要考虑安装：

管理所用文件系统的用户工具（比如 XFS 和 Btrfs 文件系统对应的管理工具）；
访问RAID或LVM分区的工具；
未包含在 linux-firmware包 中的额外固件(如用于声卡的sof-firmware包)；
联网所需要的程序（例如网络管理器或独立 DHCP 客户端，以及 Wi-Fi 认证软件和移动宽带连接所需的 ModemManager）；
文本编辑器（如：nano包、vim包 等）；
访问 man 和 info 页面中文档的工具：man-db包，man-pages包 和 texinfo包。
pkglist.x86_64.txt 中包含 Live 系统安装的软件包列表。

配置系统
生成 fstab 文件
通过以下命令生成 fstab 文件 (用 -U 或 -L 选项设置 UUID 或卷标)：

# genfstab -U /mnt >> /mnt/etc/fstab
强烈建议在执行完以上命令后，检查一下生成的 /mnt/etc/fstab 文件是否正确。

chroot 到新安装的系统
通过以下命令 chroot 到新安装的系统：

# arch-chroot /mnt
提示：此处使用的是arch-chroot而不是直接使用chroot，注意不要输错了。
设置时区
通过以下命令设置时区：

# ln -sf /usr/share/zoneinfo/Region（地区名）/City（城市名） /etc/localtime
提示：例如，在中国大陆需要将时区设置为上海这个城市，那么请运行 # ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime。
然后运行 hwclock(8) 以生成 /etc/adjtime：

# hwclock --systohc
这个命令假定已设置硬件时间为 UTC 时间。详细信息请查看 系统时间#时间标准。

区域和本地化设置
程序和库如果需要本地化文本，都依赖区域设置，后者明确规定了地域、货币、时区日期的格式、字符排列方式和其他本地化标准。

需要设置这两个文件：locale.gen 与 locale.conf。

编辑 /etc/locale.gen，然后取消掉 en_US.UTF-8 UTF-8 和其他需要的区域设置前的注释（#）。

接着执行 locale-gen 以生成 locale 信息：

# locale-gen
然后创建 locale.conf(5) 文件，并 编辑设定 LANG 变量，比如：

/etc/locale.conf
LANG=en_US.UTF-8
另外对于中文用户：

提示：
用户可以设置自己的 locale，详情请参阅 在用户会话中覆盖系统区域设置 或 设置当前区域；
将系统 locale 设置为 en_US.UTF-8 ，系统的 log 就会用英文显示，这样更容易判断和处理问题；
也可以设置为 en_GB.UTF-8 或 en_SG.UTF-8，附带以下优点：
进入桌面环境后以 24 小时制显示时间；
LibreOffice 等办公软件的纸张尺寸会默认为 A4 而非 Letter(US)；
可尽量避免不必要且可能造成处理麻烦的英制单位。
设置的 LANG 变量需与 locale 设置一致，否则会出现以下错误：
Cannot set LC_CTYPE to default locale: No such file or directory
警告： 并不推荐在此设置任何中文 locale，这可能会导致 tty 上中文显示为方块。如果您不经常使用 tty ，或是稍后需要安装桌面环境，则在不使用 tty 后可以设置为中文的 locale 。
如果需要修改#控制台键盘布局和字体，可编辑 vconsole.conf(5) 使其长期生效，例如：

/etc/vconsole.conf
KEYMAP=de-latin1
网络配置
创建 hostname 文件：

/etc/hostname
myhostname（主机名）
请接着完成新安装的环境的网络配置，配置过程中可能需要安装合适的网络管理软件。

警告： 请按上述网络配置指引正确配置好网络后再重新启动，否则系统重新启动后可能无法连接网络（不过可以用 LiveCD 重新进入 arch-chroot 进行配置）。例如在虚拟机软件 VirtualBox 安装并使用桥接模式时就需要配置 DHCP 。
关于 initramfs
通常不需要自己创建新的 initramfs，因为在执行 pacstrap 时已经安装 linux包，这时已经运行过 mkinitcpio 了。

如果是 LVM、 系统加密或 RAID 等分区配置，请修改 mkinitcpio.conf 并用以下命令重新创建一个 Initramfs：

# mkinitcpio -P
设置 root 密码
使用以下命令设置 root 密码：

# passwd
安装引导程序
需要安装 Linux 引导加载程序，才能在安装后启动系统，可以使用的的引导程序已在启动加载器中列出，请选择一个安装并配置它，GRUB 是最常见的选择。

如果有 Intel 或 AMD 的 CPU，请另外启用微码更新。

警告： 这是安装的最后一步也是至关重要的一步，请按上述指引正确安装好引导加载程序后再重新启动。否则计算机重新启动后将无法正常进入系统。
重新启动计算机
输入 exit 或按 Ctrl+d 退出 chroot 环境。

可选用 umount -R /mnt 手动卸载被挂载的分区：这有助于发现任何“繁忙”的分区，并通过 fuser(1) 查找原因。

最后，通过执行 reboot 重启系统，systemd 将自动卸载仍然挂载的任何分区。这时候不要忘记移除安装介质，然后使用 root 帐户登录到新系统。

安装后的工作
创建非特权账户、图形用户界面的安装、声音管理、触摸板支持等系统管理教程和后期工作参见建议阅读。

感兴趣的各类程序，请参见应用程序列表。
