# Ubuntu + Windows 双系统安装最佳实践

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/char7ez/ubuntu-windows-dual-boot-guide.svg)](https://github.com/char7ez/ubuntu-windows-dual-boot-guide/stargazers)

[← 返回主页](README.md) | [English Version](README-en.md)

适用于 **Ubuntu 26.04 LTS + Windows 11 Pro 25H2**，采用独立EFI分区方案，彻底避免Windows大版本更新覆盖GRUB引导。

## 🎯 核心原则

Windows更新时会强行覆盖共享EFI分区里的GRUB引导，**必须让Ubuntu和Windows各用各的EFI分区**，彻底隔离。

## ✅ 第一步：Windows下准备空间（无损分区）

1. 右键「此电脑」→ 管理 → 磁盘管理
2. 找到你的Windows系统盘（最大的NTFS分区）
3. 右键 → 「压缩卷」，等待系统计算可压缩空间
4. 输入要分给Ubuntu的大小（推荐至少100GB），点击「压缩」

## ✅ 第二步：制作Ubuntu安装U盘

1. 下载最新Ubuntu桌面版镜像
2. 用Rufus烧录到U盘（分区类型选GPT）
3. 重启电脑，BIOS里设置从U盘启动

## ✅ 第三步：最关键！强制Ubuntu创建独立EFI分区

⚠️ 这步跳过的话，Ubuntu会直接用Windows的EFI分区，后续必被覆盖！

1. 进入Ubuntu安装界面后，**先不要点安装**，按 `Ctrl+Alt+T` 打开终端
2. 找到你的安装目标盘：
   sudo fdisk -l
   比如目标盘是 /dev/nvme0n1
3. 用parted临时关闭Windows EFI的boot标志：
   sudo parted /dev/nvme0n1
  • 输入 p 查看分区，找到带 boot 标志的Windows EFI分区（一般是分区1）
  • 输入 set 1 boot off 关闭boot标志
  • 再输入 p 确认boot标志消失
  • 输入 q 退出
4. 关闭终端，开始正常安装Ubuntu

## ✅ 第四步：手动分区安装

1. 到「安装类型」这步，必须选「手动分区」
2. 找到刚才压缩出来的「空闲空间」，点击「+」
3. 挂载点选 /，文件系统用ext4，点确定
4. ✅ 你会看到Ubuntu自动创建了自己的EFI分区，继续安装即可

## ✅ 第五步：安装后恢复引导（不要重启！）

1. 安装完成提示重启时，先不要重启，再次打开终端
2. 恢复Windows EFI的boot标志：
   sudo parted /dev/nvme0n1
  • 输入 p 查看分区
  • 输入 set 1 boot on 恢复boot标志
  • 输入 p 确认，输入 q 退出
3. 确认Ubuntu引导优先：
   efibootmgr -v
   确保 BootOrder 里Ubuntu在Windows前面，不对的话用 efibootmgr -o 调整
4. 现在可以重启电脑了

## ✅ 第六步：让GRUB识别Windows
   
   Ubuntu默认禁用了os-prober，需要手动开启：

1. 安装GRUB到Ubuntu自己的EFI分区
   sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu --recheck
2. 编辑GRUB配置
   sudo nano /etc/default/grub
找到 GRUB_DISABLE_OS_PROBER=false，去掉前面的#，按 Ctrl+X → `Y` → 回车保存。
然后扫描并更新GRUB：
   sudo os-prober
   sudo grub-mkconfig -o /boot/grub/grub.cfg

## ✅ 第七步：验证双系统正常工作
   
重启后，你会看到GRUB菜单，里面有 "Ubuntu" 和 "Windows Boot Manager" 两个选项，说明引导正常。

1. 先进Ubuntu验证
  • 打开文件管理器，应该能看到Windows的磁盘分区
  • 点击挂载，尝试读写文件，确认权限正常
2. 再进Windows验证
  • 正常进入Windows系统
  • 确认所有文件都在，没有丢失
  • 检查Windows更新，确认不会影响Ubuntu引导

## 🎯 最终效果

✅ 引导完全隔离：Windows大版本更新不会碰Ubuntu的EFI分区，GRUB永远不会消失
✅ 系统互不影响：两边各自独立，随便更新都不会搞挂对方
✅ 数据互通：可以在Ubuntu里直接读写Windows分区的文件

## 🚨 避坑提醒

1. 绝对不要用共享EFI分区：Windows 25H2大版本更新100%会覆盖GRUB
2. 分区顺序：先装Windows，再装Ubuntu，不要反过来
3. BIOS里关闭「Secure Boot」，否则很多驱动（包括ROCm）会出问题

---

**本教程基于实际操作验证，适用于2026年最新的Windows 11 25H2 + Ubuntu 26.04组合。**

如果帮到了你，请给个Star ⭐️ 支持一下！
