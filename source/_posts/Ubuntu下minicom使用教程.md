---
title: Ubuntu下minicom使用教程
date: 2019-01-21 18:12:45
tags:
---
## 1.安装

```
#sudo apt-get install minicom
```
## 2.配置与使用<!--more-->

```
#sudo minicom -s
```
选择Serial port setup，如果要调试usb串口，设置Serial Device为/dev/ttyUSB0，然后设置Hardware Flow Contrl 为No，回车，Exit。

