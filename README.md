# Ubuntu + Windows Dual Boot Guide

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/charZZH/ubuntu-windows-dual-boot-guide.svg)](https://github.com/charZZH/ubuntu-windows-dual-boot-guide/stargazers)

Best practice for installing **Ubuntu 26.04 LTS + Windows 11 Pro 25H2** dual boot with separate EFI partitions, completely avoiding Windows updates overwriting GRUB.

## 🌍 Choose Your Language

- [**中文版本 (Chinese)**](README-zh.md)
- [**English Version**](README-en.md)

## 🎯 Key Features

✅ **Boot completely isolated** - Windows updates never touch Ubuntu's EFI partition  
✅ **Systems independent** - Update either OS without breaking the other  
✅ **Data interoperable** - Read/write Windows partitions directly from Ubuntu  

## 📝 Background

The #1 problem with dual boot is that Windows Feature Updates (like 25H2) always overwrite the shared EFI partition and delete GRUB. This guide solves it permanently by using separate EFI partitions for each OS.

## ⭐️ Show Your Support

If this guide helped you, please give it a star! It helps others find it too.

## 🤝 Contributing

Issues and PRs are welcome!
