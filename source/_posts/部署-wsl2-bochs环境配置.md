---
title: '[部署]wsl2-bochs环境配置'
date: 2023-04-21 19:45:48
tags: 
- 部署 
- wsl2
- bochs
- 操作系统
categories: 部署
---

wsl2 - Ubuntu 22.04 + VSCode + bochs + xfce4 + VcXsrv

**笔者环境 wsl2 - Ubuntu 22.04**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133109488-1483721842.png)



## 0. 安装WSL2 & VSCode & 终端

网上教程千千万，请自行查找

**WSL2**：[ WSL2安装教程_pengege666的博客-CSDN博客](https://blog.csdn.net/weixin_42888638/article/details/127129727)
​  	切换清华源：[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)
​	  	备份：`sudo cp /etc/apt/sources.list /etc/apt/sources.bak`
​		修改：`sudo vim /etc/apt/sources.list`
​		更新：`sudo apt update`

**VSCode**：[Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)

​	安装插件：WSL

​	然后点击左下绿色按钮，按提示连接WSL

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133145985-1139654339.png)


**终端**：Microsoft Store就有
![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230421130405538-1624760680.png)



## 1. 安装软件包

以下命令以行为单位粘贴到终端运行

```shell
sudo apt update
sudo apt upgrade

sudo apt-get install -y neofetch
sudo apt-get install -y gcc
sudo apt-get install -y vim
sudo apt-get install -y build-essential
sudo apt-get install -y g++
sudo apt-get install -y libgtk2.0-dev
sudo apt-get install -y nasm
sudo apt-get install -y gdb
```


## 2. 配置 WSL2 图形界面

采用 xfce4 + VcXsrv

> xfce4是一个轻量级的类Unix的桌面系统，提供桌面环境
>
> VcXsrv提供图形界面，使在windows子系统wsl里的操作能够图形化显示

### 2.1 安装VcXsrv

下载地址：https://sourceforge.net/projects/vcxsrv/files/latest/download

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134429054-358612362.png)
![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134449834-650116383.png)

选择**one large window**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134501575-1115983286.png)
![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134510694-933931507.png)


**一定勾选Disable access control**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134524214-513922816.png)

看到下图即为成功

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134537056-251641745.png)

***解决高DPI模糊问题**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134625246-755424886.png)

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134638305-1079137345.png)

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134648360-304047365.png)


### 2.2 安装xfce4

`sudo apt install -y xfce4`

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134720388-1427608973.png)


### 2.3 配置

打开 .bashrc：`cd ~ & vim .bashrc`

在 .bashrc 文件最后添加

```shell
# 配置xfce4
export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
```

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134741381-1188389658.png)


**添加后执行`source ~/.bashrc`命令。**

### 2.4 启动

`sudo startxfce4`

**此外，当看到防火墙选项时，请同意其通过**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134759256-1237532277.png)

**小技巧**：当在终端执行`sudo startxfce4`后，xfce4会在前台输出log无法执行其他命令。此时可以再开一个终端窗口执行其他命令（比如开bochs什么的）
![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230421125101116-687537558.png)

**问题解决**：
当连接到图形化桌面时，如果一阵未使用桌面，会导致桌面没有反应直接卡死
**原因**：因为xfc4锁屏了....
**解决**：把锁屏删掉  `sudo apt purge xfce4-screensaver`



## 3. 安装bochs

bochs 2.6.2：https://sourceforge.net/projects/bochs/files/bochs/2.6.2/bochs-2.6.2.tar.gz

### 3.1 下载

在Linux下使用wget命令下载

`wget https://sourceforge.net/projects/bochs/files/bochs/2.6.2/bochs-2.6.2.tar.gz`

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133314035-1670243439.png)

### 3.2 解压

<u>* **非必须**</u>

*移动源码到合适的目录（笔者这里放在 ~/OS/实验3 下）

`mv bochs-2.6.2.tar.gz OS/实验3`

*打开源码所在目录

 `cd OS/实验3`

**解压**

`tar -zxvf bochs-2.6.2.tar.gz`

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133239873-693223833.png)

### 3.3 配置

**1.进入解压出来的目录**

`cd bochs-2.6.2/`

**2.生成 Makefile**

请在终端粘贴下列命令（请整块粘贴）
**！！！！！注意注意`--prefix=/your_path/bochs \`处的`your_path`要改成你自己想安装的路径**

```shell
./configure \
--prefix=/your_path/bochs \
--enable-debugger \
--enable-disasm \
--enable-iodebug \
--enable-x86-debugger \
--with-x \
--with-x11 \
LDFLAGS='-pthread' \
LIBS='-lX11'
```

下对配置命令进行解析 来源：《操作系统真相还原》

```shell
--prefix=/your_path/bochs \			# 指定安装目录，安装目录替换your_path
--enable-debugger \				# 打开bochs自身调试器
--enable-disasm \				# 使bochs支持反汇编
--enable-iodebug \				# 启动io接口调试器
--enable-x86-debugger \				# 使bochs支持x86调试器
--with-x \					# 使用x windows
--with-x11 \					# 使用x11图像用户接口
```

下给出笔者所用命令

```shell
./configure \
--prefix=/home/fwm-0100/bochs \
--enable-debugger \
--enable-disasm \
--enable-iodebug \
--enable-x86-debugger \
--with-x \
--with-x11 \
LDFLAGS='-pthread' \
LIBS='-lX11'
```

***3. 修改Makefile**

`vim Makefile`

在92行添加

```makefile
IBS =-lm -lgtk-x11-2.0 -lgdk-x11-2.0 -latk-1.0 -lgio-2.0 -lpangoft2-1.0 -lgdk_pixbuf-2.0 -lpangocairo-1.0 -lcairo -lpango-1.0 -lfreetype -lfontconfig -lgobject-2.0 -lgmodule-2.0 -lglib-2.0 -lpthread
```


为啥要搞这步捏？ 来源：《操作系统真相还原》![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133518098-220819599.jpg)


### 3.4 编译安装

编译：`make`

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133649322-1267685748.png)

安装：`sudo make install`

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133758733-164739630.png)

bochs安装目录如下：

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420133835119-509379829.png)

### 3.5 配置bochs

打开bochs程序目录，笔者这里是`/home/fwm-0100/bochs/bin`

创建`bochsrc.disk`

`sudo vim bochsrc.disk` 

输入以下内容(注意路径/home/fwm-0100要改成自己的bochs安装目录)

```shell
###############################################
######## Configuration file for Bochs #########
###############################################

# 第一步
# 设置Bochs在运行过程中能够使用的内存，本例为32MB
megs: 32


# 第二步
# 设置对应真实机器的BIOS和VGA BIOS
# 对应两个关键字：romimage 和 vgaromimage
# 注意这里的/home/fwm-0100 要替换为自己的安装目录
romimage: file=/home/fwm-0100/bochs/share/bochs/BIOS-bochs-latest
vgaromimage: file=/home/fwm-0100/bochs/share/bochs/VGABIOS-lgpl-latest


# 第三步
# 设置Bochs所使用的磁盘
# 软盘的关键字为floppy。
# 若只有一个软盘，则使用floppya即可，若有多个，则为floppya，floppyb…
# floppya: 1_44=a.img, status=inserted


# 第四步
# 选择启动盘符
# 默认从软盘启动，将其注释，我们使用从硬盘启动
# boot: floppy
boot: disk


# 第五步
# 设置日志文件的输出
log: bochs.out


# 第六步
# 开启或关闭某些功能

# 关闭鼠标
mouse: enabled=0

# 打开键盘
keyboard_mapping: enabled=1,map=/home/fwm-0100/bochs/share/bochs/keymaps/x11-pc-us.map

# 硬盘设置
ata0: enabled=1,ioaddr1=0x1f0,ioaddr2=0x3f0,irq=14

# gdb支持(需要在配置的时候就开启，不然会报错)
# 这样gdb便可以远程连接到此机器的1234端口调试
# gdbstub: enabled=1, port=1234, text_base=0, data_base=0, bss_base=0

################### 配置结束 ###################
```

### 3.6 运行bochs

**以下操作更加建议直接在图形化界面下的终端执行命令**

进入bochs安装目录下的bin目录，运行`./bochs`

此时在VcXsrv出现一个bochs的黑色窗口

所有需要输入的地方请见下图中框出部分

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230421131109117-322949858.png)


**在终端输入`c` 在VcXsrv下的bochs的黑色窗口出现bochs的UI**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230421131213785-1865061577.png)


看到下面的窗口，证明已经成功啦！！！！

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230421124224354-1088943758.png)


---

**常见问题**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134105984-2002691368.png)

**原因**：disk有tab（空格）

**解决**：删除配置文件空格即可



![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134127151-533292238.png)

**原因**：看图，不应该**换行**

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134249062-685439539.png)

**解决**：不换行喽

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134303458-579618952.png)




![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134340786-1518101424.png)

**原因**：配置编译的时候没写gdb

**解决**：配置文件就不要加gdb喽，注释掉

![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230420134359230-750212926.png)



![](https://img2023.cnblogs.com/blog/3129870/202304/3129870-20230421125345618-1610024171.jpg)

**原因**：配置文件没有放在bochs安装目录的bin目录下

**解决**：移动到bin目录下，如：`/home/fwm-0100/bochs/bin`



## 参考文档

《操作系统真相还原》

[通过 VcXsrv 在 WSL2 上使用图形化界面（xfce4） - bluenlq - 博客园 (cnblogs.com)](https://www.cnblogs.com/blauendonau/p/14166062.html)

[WSL2(Ubuntu 22.04.2 LTS) + Win11 + Bochs-Gui_wsl安装bochs_物与我皆无尽也的博客-CSDN博客](https://blog.csdn.net/qq_45746571/article/details/129846595)

[Linux下bochs打开黑屏解决方法](https://blog.csdn.net/qq_45740212/article/details/113469718)

[WSL2 Ubuntu + Xfce4 一段时间 Xfce4 卡死不动](https://blog.csdn.net/noto_xj/article/details/129929555)

**特别鸣谢：ZGY**