---
title: vps
date: 2026-02-23 11:00:00
tags:
  - 优化
categories:
  - 技术向
---
# Oracle Cloud vm选择
## 我这里使用的rocklinux 9 x86，Arm就算了，抢不到的直接抢不到~，其他linux系统也类似，我感觉rocklinux比较稳定罢了。

---
## 免费套餐我们能用到的都是内存0.75G（可怜啊），我们依次优化一下
### 一、/boot瘦身
```bash
# 使用df命令查看/boot占用情况，一般而言，会有多个内核及救援镜像
df
Filesystem             1K-blocks    Used Available Use% Mounted on

devtmpfs                    4096       0      4096   0% /dev

tmpfs                     389064       0    389064   0% /dev/shm

tmpfs                     155628   14744    140884  10% /run

efivarfs                     256      21       231   9% /sys/firmware/efi/efivars

/dev/mapper/rocky-root  47636480 7450084  40186396  16% /

/dev/sda2                 958464  770680    187784  81% /boot

/dev/sda1                 101148    7190     93958   8% /boot/efi

tmpfs                      77812       0     77812   0% /run/user/1000
```
从上述结果可以看出，/boot已使用了81%，为防止下次 `dnf update` 更新内核时会报错中断，可能导致系统进入不完整更新的风险状态。(当然，vps一般是不会重启的，┑(￣Д ￣)┍)
```bash
# 以下输出结果均为假设，以实际为准。
# 1、查看所有救援文件
ls -lh /boot/initramfs-0-rescue* /boot/vmlinuz-0-rescue*
-rw-------. 1 root root 80M Aug 27 18:01 /boot/initramfs-0-rescue-2521a84c90644bf9b4b098fefb33f947.img
-rw-------. 1 root root 85M Jun  3  2025 /boot/initramfs-0-rescue-43fbf6126ccb4b77939c770eccfbe5b3.img
-rw-------. 1 root root 81M Jan 13 10:56 /boot/initramfs-0-rescue-fe26d7bf1a8d4e52a3d099527b5ee2fb.img
-rwxr-xr-x. 1 root root 15M Aug 27 18:00 /boot/vmlinuz-0-rescue-2521a84c90644bf9b4b098fefb33f947
-rwxr-xr-x. 1 root root 15M Jun  3  2025 /boot/vmlinuz-0-rescue-43fbf6126ccb4b77939c770eccfbe5b3
-rwxr-xr-x. 1 root root 15M Jan 13 10:55 /boot/vmlinuz-0-rescue-fe26d7bf1a8d4e52a3d099527b5ee2fb
# 2、保留日期最新的一个，删除其他的
sudo rm /boot/initramfs-0-rescue-2521a84c90644bf9b4b098fefb33f947.img
sudo rm /boot/initramfs-0-rescue-43fbf6126ccb4b77939c770eccfbe5b3.img
sudo rm /boot/vmlinuz-0-rescue-2521a84c90644bf9b4b098fefb33f947
sudo rm /boot/vmlinuz-0-rescue-43fbf6126ccb4b77939c770eccfbe5b3
# 3、如果你不需要调试内核崩溃，删除以下kdump镜像
sudo rm /boot/initramfs-*-kdump.img
# 4、确认当前运行的内核版本
uname -r
5.14.0-570.33.2.el9_6.x86_64
# 5、检查有无其他大文件
sudo du -sh /boot/* | sort -rh | head -n 10
85M     /boot/initramfs-0-rescue-43fbf6126ccb4b77939c770eccfbe5b3.img
83M     /boot/initramfs-5.14.0-611.30.1.el9_7.x86_64.img
83M     /boot/initramfs-5.14.0-611.16.1.el9_7.x86_64.img
82M     /boot/initramfs-5.14.0-570.33.2.el9_6.x86_64.img
81M     /boot/initramfs-0-rescue-fe26d7bf1a8d4e52a3d099527b5ee2fb.img
80M     /boot/initramfs-0-rescue-2521a84c90644bf9b4b098fefb33f947.img
49M     /boot/initramfs-5.14.0-570.33.2.el9_6.x86_64kdump.img
15M     /boot/vmlinuz-5.14.0-611.30.1.el9_7.x86_64
15M     /boot/vmlinuz-5.14.0-611.16.1.el9_7.x86_64
15M     /boot/vmlinuz-5.14.0-570.33.2.el9_6.x86_64
# 6、删除指定内核（可选）
sudo dnf remove kernel-core-5.14.0-611.30.1.el9_7.x86_64
# 7、刷新引导配置（重要）
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
### 二、dnf.conf配置优化
```bash
sudo vi /etc/dnf/dnf.conf
```
```conf
[main]
gpgcheck=1 
installonly_limit=2
clean_requirements_on_remove=True # 卸载包时自动删除不再需要的依赖 fastestmirror=True # 让 dnf 自动找最快的镜像源（虽然 OCI 内部通常已经很快了） max_parallel_downloads=10 # 允许同时下载 10 个包，显著提升更新安装速度
```
### 三、小内存Swap优化
```bash
# 1. 调整 Swappiness & 优化缓存压力
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
echo "vm.vfs_cache_pressure=50" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
# 2、使用 ZRAM 代替磁盘 Swap
sudo dnf install zram-generator -y
sudo vi /etc/systemd/zram-generator.conf
```
```conf
[zram0]
zram-size = ram / 2    # 使用物理内存的一半作为 ZRAM
compression-algorithm = zstd  # 使用 zstd 压缩算法，效率最高
```
```bash
sudo systemctl daemon-reload
sudo systemctl start /dev/zram0
# 3、如果你的 VPS 只有 1GB 内存且运行了 MySQL 等大内存应用，建议额外创建一个 2GB 的 Swap 文件作为兜底。
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo vi /etc/fstab
# 末尾添加
```
```conf
/swapfile none swap defaults 0 0
```

### 四、禁用不必要服务
```bash
# 1、**kdump (内核崩溃转储)：** 除非你要调试内核死机，否则可以关掉。
sudo systemctl stop kdump
sudo systemctl disable kdump
# 2、**Cockpit (Web 控制台)：** 如果你只用 SSH，可以关掉。
sudo systemctl stop cockpit.socket
sudo systemctl disable cockpit.socket
```
