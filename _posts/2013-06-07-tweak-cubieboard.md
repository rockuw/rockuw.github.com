---
layout: post
title: "cubieboard 折腾手记"
description: "703n内存太小了，所以入手cubieboard"
category:
excerpt: "
曾经有一段时间我一直想有个linux box，但是当时市面上还没有这个东西卖，所
以一直没机会搞。当时问搞嵌入式的同学，他们也不知道，只知道有开发板。记
得当时Marvell公司搞过一个比赛，用他们自己的一个叫guru plug的产品来开发
应用。进了复赛会免费给你寄一个，但是我们连初赛都没有过，所以遗憾……后来
又迷了一段时间tplink的703n，有openwrt的官方支持，这个东西其实也挺强大
了，上网一搜才知道，已经有很多人用它做各种各样的事情。所以我在淘宝买了
一个内存和flash都加大了703n，想用它用pt下载，但是试验过才发现这就是个
悲剧，64M的内存，开着tranmission一会儿路由就重启了，挂我的移动硬盘，一
"
tags: [酷, linux]
---
{% include JB/setup %}

曾经有一段时间我一直想有个linux box，但是当时市面上还没有这个东西卖，所
以一直没机会搞。当时问搞嵌入式的同学，他们也不知道，只知道有开发板。记
得当时Marvell公司搞过一个比赛，用他们自己的一个叫guru plug的产品来开发
应用。进了复赛会免费给你寄一个，但是我们连初赛都没有过，所以遗憾……后来
又迷了一段时间tplink的703n，有openwrt的官方支持，这个东西其实也挺强大
了，上网一搜才知道，已经有很多人用它做各种各样的事情。所以我在淘宝买了
一个内存和flash都加大了703n，想用它用pt下载，但是试验过才发现这就是个
悲剧，64M的内存，开着tranmission一会儿路由就重启了，挂我的移动硬盘，一
插上就重启。**于是我一怒之下就买了个拥有1G内存的cubieboard**。

拿回来之后就是各种折腾，在写毕业论文的压力下，我硬是有两天多什么都没做，
一直在搞它。想想这是种什么样的心情，同学们都早出晚归地去实验室写论文，
而我由于折腾不得法，在寝室对着一团乱的桌面脑袋里也一团乱。这一方面是我
的原因，一方面也是因为网上的教程乱七八糟，我没有找到一篇真正优质的，从
零开始的教程。连[archlinux官网上的方法][archlinux-bug]都不对！所以我打
算写这篇文章，从零开始，一步一步折腾cubieboard。
![我的桌面][messy-desktop]

###硬件准备

- 一根HDMI线，是正常的HDMI，不是micro HDMI
- 一张TF卡，其实就是手机上用的那种小卡
- 一个cubieboard

###刷系统

刚拿到机器的时候，通电开机体验一下自带的android系统。然后寻思如何装个
linux。由于我朝悲剧的网速，官方介绍的berryboot联网下载安装ubuntu的教程
是不敢去做的。在[这位仁兄][sina-cubieboard-create-ubuntu-image]的教程帮
助下，还是很顺利地制作了启动的TF卡。但是由于我从linaro的官网下的是比较
新的系统，刷完之后能看到屏幕上出现启动信息，但是在某处出现了错误，最终
不能启动，后来我又去下了一个较早的版，
[linaro-precise-alip-20121124-519][linaro-tar-link]，这个系统的内核是
3.0。按教程刷好后，可以进入桌面，一切正常。值得一提的是，制作系统卡的时
候，要在linux环境下，因为那个`sunxi-media-create.sh`的脚本是为linux系统
写的，如果用mac的话会有很多问题，即使你把bash脚本都改成mac下可用的了也
还是不行，因为到后来有个工具叫`sfdisk`Mac下是没有的。

刚刷好的系统一点也不好用：没有sshd，默认的locale竟然是`C.UTF-8`，中文
字体欠缺。所以，首先修改locale：

    /usr/share/locales/install-language-pack en_US.UTF-8

然后将`/etc/default/locale`中的内容改成`LANG=en_US.UTF-8`。重启一下，
就可以安装别的软件了。

    sudo apt-get update
    sudo apt-get install openssh-server
    sudo apt-get install ttf-wqy-zenhei

这样应该系统就基本可以了，ssh过去终端下也能正常显示中文了。

###无线网卡

这是我折腾最久的部分，有两点需要抱怨：一是cubieboard的淘宝卖家提供了水
星MW150US的网卡，在cubieboard的介绍页面写的是：

> 如果您使用的是无线网络，需要一个无线网卡，该款MINI网卡是不错的选择，
> 即插即用，免驱动。

在网卡的介绍页面贴了一个驱动的链接，是github上的代码库的一个地址。没有
任何说明应该如何安装驱动。
二是刷的linaro ubuntu系统是自带有8192cu的驱动，但是谁也没有告诉我**到
底哪个网卡是使用这个芯片的**，我google上找一下，还真找到了，是阿里巴巴
中国的一个页面。我就想，这么重要的信息，居然都没有人写篇博客总结一下吗？
cubieboard有哪些无线网卡可以免驱直接使用。

后来还是自己编译的驱动，网上有不少关于编译8188eu驱动的文章，但正是它们
让我困惑，`make mrproper`, `CONFIG_XXX=m`，这些东西完全不知道是什么意
思。我后来回过来看，发现刚开始时**不应该看到教程立即去做，而是应该适当
地停一下，补充一些必要的知识**。我后来知道了什么叫内核模块，如何编译内
核，即[内核的编译构建系统是什么样的][kbuild-intro]，如果我当时在动手前
去看一下这篇文章，就能大大地缩短我折腾的时候，也能提升折腾的愉悦度。所
以，完整的步骤是：

1. 如果你的内核是3.0版本的（通过`uname -a`查看内核版本），就去下载
   sunxi-3.0的代码，如果是3.4版本的，就下载3.4版本的代码。
   [linux-sunxi][linux-sunxi-github]
   你可以交叉编译（如果不知道是什么意思，是时候停下来，补充一下知识了），
   也可以直接用cubieboard编译，我当时是直接编译的，编译过程花了1个多小
   时，交叉编译会快很多，大概10多分钟就能编译完。
3. 先编译内核。在cubieboard上`zcat /proc/config.gz > .config`，然后将
   生成的`.config`文件拷贝到linux-sunxi-3.0的根目录下，然后可以编译内
   核了：

        make prepare
        make modules_prepare
        make

   如果是交叉编译的情况，需要先安装交叉编译器：

        sudo apt-get install gcc-arm-linux-gnueabi
        sudo apt-get install g++-arm-linux-gnueabi

   在进行所有的`make`前，需要在`.config`中设置
   `CONFIG_CROSS_COMPILE="arm-linux-gnueabi-"`。
4. 编译8188eu的驱动，从[这里][8188eu-driver-github]下载驱动的代码。
   遗憾的是git对于下载单独的子目录是需要技巧的，我看了半天也没看明白，
   干脆直接把整个代码库都下下来了。将rtl8188eu目录放在一个地方，修改其
   中的Makefile：
   
        CONFIG_PLATFORM_I386_PC = y 改为 = n
        CONFIG_PLATFORM_ARM_SUN4I = n 改为 = y
   
   然后：

        cd /path/to/rtl8188eu
        CONFIG_RTL8188EU=m make -C /path/to/linux-sunxi-3.0 M=`pwd`
        
   如果编译时出现usb_intf.c 报找不到mach/sys_config.h的错，找到这个位
   置，将mach改为plat，再进行编译，应该就没有问题了。参考
   [这里][compile-8188eu]
5. 编译完成后将生成的8188eu.ko拷入到
   `/lib/modules/kernel/3.xxx/drivers/net/wireless`下，然后执行：

        depmod -a
        modprobe 8188eu

   大功告成，用`ifconfig`查看可以看到已经有了`wlan0`这个设备了。然后是
   设置IP，连接无线路由器了。具体参考[这篇文章][cubieboard-wifi-setup]，
   亲测有效。

###PT下载

用cubieboard实现pt下载，需要具备两个条件：能正常工作的移动硬盘和能正常
工作的BT客户端。移动硬盘要特别注意，**你需要一个带电源的USB Hub才能带
得起来**，可以去买[这个][ssk-usb-hub]，感觉还不错。然后就是安装ntfs-3g
的驱动，mount等等了。

BT客户端的话，好像网上大家都用tranmission，我开始用的它，但是发现它特
别占资源，使用起来很卡，不稳定。后来我一找，发现了一票的linux下可用的
BT客户端，包括deluge, qBittorrent, ktorrent等。试了下deluge，感觉很不
错，剩下的就没有去试了。deluge的设置如下：

1. 下载deluge的源码，然后安装相关的依赖。参考
   [官方的指导][deluge-install]
2. 先运行`deluged`启动进程，然后运行`deluge-web`，这样就可以在浏览器中
   用http://ip:8112 的地址进行web管理了。**需要注意的是，`deluged`是根
   据当前用户在其`home`目录下建立一个`.config/deluge`的目录，这里面保
   存了deluge的所有设置和下载的状态信息等。所以要保证只用同一个用户去
   运行deluge。**
3. 设置blocklist，像我们在学校ipv6是不限速的，而ipv4是限速的，如果开着
   BT的话，其他室友就不能好好上网了，所以需要设置blocklist，将所有的
   ipv4流量都禁掉。deluge提供这个功能，但是具体如何设置好像是
   undocumented。首先blocklist是一个插件，需要先进行安装：

        cd /path/to/deluge/source/deluge/plugins/blocklist
        python setup.py build

   然后可以看到在dist目录下有一个叫`Blocklist-1.2-py2.7.egg`的文件，这
   就是插件文件了，将它拷入配置文件夹中：

        cp dist/Blocklist-1.2-py2.7.egg ~/.config/deluge/plugins/

   修改`.config/deluge/core.conf`，设置启用这个插件：

        "enabled_plugins": [
          "Blocklist"
        ]

   编辑一个过滤规则文件，`vi .config/deluge/ipfilter.dat`：

        IPv4:0.0.0.0-255.255.255.255

   编辑插件配置文件，`vi .config/deluge/blocklist.conf`：

        {
          "file": 1,
          "format": 1
        }{
          "check_after_days": 4,
          "timeout": 180,
          "url": "/home/XXX/.config/deluge/ipfilter.dat",
          "try_times": 3,
          "list_size": 0,
          "last_update": 0.0,
          "list_type": "PeerGuardian",
          "list_compression": "",
          "load_on_start": true
        }

   完成。这些都是我通过在图形界面中运行`deluge-gtk`配置，然后总结出来
   的。core.conf中还有一些其他的参数，比如下载目录的设定、关闭dht等等，
   可以自己研究。

###iptables

这个3.0的内核是没有iptables模块的，因此netfilter的一些功能比如网络转发，
NAT等无法使用，如果要将cubieboard作为网关来使用，就需要将iptables的功
能加入内核。我已经通过交叉编译做好了内核的uImage，但是由于这段时间在挂
PT，所以等过段时间再试试把内核换掉，看有没有效果。

[archlinux-bug]: http://archlinuxarm.org/platforms/armv7/cubieboard
[messy-desktop]: http://rockuw.com/images/messy-desktop.jpg
[sina-cubieboard-create-ubuntu-image]: http://blog.sina.com.cn/s/blog_5459f60d0101h0j3.html
[linaro-tar-link]:http://releases.linaro.org/12.11/ubuntu/precise-images/alip
[kbuild-intro]: http://www.linuxjournal.com/content/kbuild-linux-kernel-build-system
[linux-sunxi-github]: https://github.com/linux-sunxi/linux-sunxi
[8188eu-driver-github]: https://github.com/mmplayer/linux-sunxi/tree/sunxi-3.4/drivers/net/wireless/rtl8188eu
[compile-8188eu]: http://cn.cubieboard.org/forum.php?mod=viewthread&tid=475&highlight=%E7%BD%91%E5%8D%A1
[cubieboard-wifi-setup]: http://cubieboard.info/thread-3-1-1.html
[ssk-usb-hub]: http://www.amazon.cn/gp/product/B0030XLWDS/ref=oh_details_o00_s00_i00?ie=UTF8&psc=1
[deluge-install]: http://dev.deluge-torrent.org/wiki/Installing/Source
