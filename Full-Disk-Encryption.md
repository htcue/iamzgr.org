# 安装前配置
## 控制台字体
```
setfont ter-132b
```

## 设置root用户密码
```
passwd
```

## 开启ssh服务
```
systemctl start sshd
```
## 禁用 reflector 服务
```
systemctl stop reflector.service
systemctl status reflector.service
```
## 验证引导模式

```
ls /sys/firmware/efi/efivars
```

## 连接网络
```bash
iwctl # 进入交互式命令行
device list # 列出无线网卡设备名，比如无线网卡看到叫 wlan0
station wlan0 scan # 扫描网络
station wlan0 get-networks # 列出所有 wifi 网络
station wlan0 connect wifi-name # 进行连接，注意这里无法输入中文。回车后输入密码即可
exit # 连接成功后退出
```
##  编辑系统镜像
```
# nano /etc/pacman.d/mirrorlist
##
## Arch Linux repository mirrorlist
## Generated on 2023-07-19
##

## China
Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```
## 更新系统时间
```
# 更新系统时钟
timedatectl set-ntp true # 将系统时间与网络时间进行同步
timedatectl status # 检查服务状态
```

# 分区和格式化
### 分区
```bash
mkfs.fat -F32 /dev/   efi


```
### 格式化分区
```bash

```
## 加密
```bash

```

```bash
btrfs subvolume create /mnt/@home 
btrfs subvolume create /mnt/@snapshots 
btrfs subvolume create /mnt/@cache 
btrfs subvolume create /mnt/@libvirt 
btrfs subvolume create /mnt/@log 
btrfs subvolume create /mnt/@tmp
```

# 安装基础系统

```


```

# 安装桌面环境
```


```
