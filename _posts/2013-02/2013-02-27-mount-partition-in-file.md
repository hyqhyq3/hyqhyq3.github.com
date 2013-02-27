---
layout: post
title: mount partition in file
category: 
tags: 
---

在linux下挂载loop文件很简单

    $ mount -o loop disk.img /mnt
    
但是如果disk.img是包含分区表的文件呢？

    $ mount -o loop disk.img /mnt
    mount: wrong fs type, bad option, bad superblock on /dev/loop0,
    missing codepage or helper program, or other error
    In some cases useful info is found in syslog - try
    dmesg | tail or so

这咋办呢？
经过我N多次google，终于搜索到一个解决办法：

    ## 先查看一下分区表
    $ fdisk -l disk.img
    Disk disk.img: 1073 MB, 1073741824 bytes, 2097152 sectors
    Units = 扇区 of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0xec30dacc
    
        设备 Boot      Start         End      Blocks   Id  System
    disk.img1            2048     1050623      524288   83  Linux
    disk.img2         1050624     2097151      523264   83  Linux

    ## 然后挂载第一个分区
    $ mount -o loop,offset=$((512*2048)) disk.img /mnt/part1
    ## 然后挂载第二个分区
    $ mount -o loop,offset=$((512*1050624)) disk.img /mnt/part2
    
也就是说在mount时，要加一个offset参数，值就是在fdisk的start×512