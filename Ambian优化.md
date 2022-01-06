# Ambian探索记录



## 初始化



### 写入EMMC

```
nano-sata-install

#安装系统至eMMC
```



### RootFs扩容

```
以扩容 /dev/sda2 为例
df -h #查看当前磁盘大小

Filesystem      Size  Used Avail Use% Mounted on
rootfs          2.9G  2.8G   15M 100% /
/dev/root       2.9G  2.8G   15M 100% /
devtmpfs        214M     0  214M   0% /dev
tmpfs            44M  244K   44M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            88M     0   88M   0% /run/shm
/dev/mmcblk0p1   56M   19M   37M  34% /boot
tmpfs            88M     0   88M   0% /tmp

fdisk /dev/sda2   #使用fdisk操作磁盘
Command (m for help): p #查看分区信息

设备 启动 起点 末尾 扇区 大小 Id 类型
/dev/sdb1 8192 139263 131072 64M 83 Linux
/dev/sdb2 139264 2949119 2809856 1.3G 83 Linux
关键数字----139264

Command (m for help): d   #d，删除分区
Partition number (1-4): 2   # 2，删除第二分区

Command (m for help): n  #创建一个新分区
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
   
Select (default p): p  #创建主分区

Partition number (1-4, default 2): 2  #分区2

First sector (2048-7744511, default 2048): 139264  #输入第一次得到的第二分区起始扇区

Last sector, +sectors or +size{K,M,G} (139264-7744511, default 7744511):  #最后一个sector，默认即可Enter
Using default value 7744511

Command (m for help): w   #将上面的操作写入分区表
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.


reboot  #设置完成需要重启 reboot

重启完成之后，此时查询也还是没有变化的，还需要如下命令

resize2fs /dev/sda2

这时你再来查询树莓派的系统磁盘的容量就扩展啦！！！
```



### 更改 CPU 频率

某些板允许调整 CPU 速度

```
nano /etc/default/cpufrequtils
```

更改**min_speed**或**max_speed**变量。

```
service cpufrequtils restart
```

### 

### 连接Wi-Fi

ssh连接后执行

```
nmtui或armbian-config
```

选择第二个Activate a connection，按提示选择并输入密码即可

若Activate a connection中没有Wi-Fi选项卡则说明内核尚未开启wifi_dummy和dhd模块，执行如下指令即可

```
modprobe dhd && echo dhd >> /etc/modules
modprobe wifi_dummy && echo wifi_dummy >> /etc/modules
```



### 蓝牙连接

执行

```
armbian-config
```

进去后选择Network，接着选择BT Install，耐心等待蓝牙组件安装完毕，然后退出。

接着执行

```
sudo apt install pulseaudio-module-bluetooth #安装pulseaudio组件
armbian-config #安装pulseaudio组件
```

安装完成后，分别执行

```
sudo kill all pulseaudio
pulseaudio --start
#启动pulseaudio服务
```

开始进入蓝牙连接阶段，首先执行

```
hciconfig -a
#查看蓝牙控制器信息
```

确认无误后，执行

```
hciconfig hci0 up
#打开蓝牙控制器
```

然后执行

```
bluetoothctl
#打开蓝牙管理器
```

先后执行

```
power on
discoverable on
agent on
scan on
#搜集周围的蓝牙设备
```

记录下要连接的设备地址后，执行

```
trust <设备地址>
#信任设备
pair <设备地址>
#配对
```

此时，要配对的设备上可能会弹出提示，点确认。

如以上步骤都没有问题，则执行

```
connect <设备地址>
```

稍候即可顺利连接蓝牙，可以运行

```
info <设备地址>
#确认状态
```

至此，N1盒子蓝牙连接完毕。 



## 系统优化



### 修复MAC地址

建立脚本，root权限执行

```
#!/bin/bash

#update:2019-11-11
#author:alon2000

#LAN_MAC=`ifconfig eth0 | grep -w ether | awk '{print $2}'`
LAN_MAC=`cat /sys/class/net/eth0/address`
MAC_HEAD=`echo $LAN_MAC|cut -c1-15`
MAC_TAIL=`echo $LAN_MAC|cut -c16-17`
MAC_TAILn=$((16#${MAC_TAIL}-1))

WLAN_MAC="$(printf '%s%02x\n' $MAC_HEAD $[MAC_TAILn])"

if [[ -f "/lib/firmware/brcm/brcmfmac43455-sdio.phicomm,n1.txt" ]] ; then
  sed -i -e "s/^macaddr=b8:27:eb:74:f2:6c$/macaddr=$WLAN_MAC/" \
    "/lib/firmware/brcm/brcmfmac43455-sdio.phicomm,n1.txt"
fi

if [[ -f "/lib/firmware/brcm/brcmfmac43455-sdio.txt" ]] ; then
  sed -i -e "s/^macaddr=b8:27:eb:74:f2:6c$/macaddr=$WLAN_MAC/" \
    "/lib/firmware/brcm/brcmfmac43455-sdio.txt"
fi

echo "WiFi MAC address modified successfully! reboot..."
reboot
```



### 修改apt软件源为国内源

本文基于armbian 5.77 Debian

```
vim /etc/apt/sources.list
```

按 i 进入插入模式。

将文件内容替换成以下内容：

```
deb [ arch=arm64,armhf ] https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch main contrib non-free
#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch main contrib non-free

deb [ arch=arm64,armhf ] https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-updates main contrib non-free
#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-updates main contrib non-free

deb [ arch=arm64,armhf ] https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-backports main contrib non-free
#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-backports main contrib non-free

deb [ arch=arm64,armhf ] https://mirrors.tuna.tsinghua.edu.cn/debian-security/ stretch/updates main contrib non-free
#deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security/ stretch/updates main contrib non-free
#deb [ arch=arm64,armhf ] https://mirrors.tuna.tsinghua.edu.cn/debian/ sid main contrib non-free
```

编辑完成按 Esc 退出插入模式，按 :wq 保存退出。

使用 apt-get 更新和升级软件包：

```
apt-get update && apt-get upgrade
```

在实际操作过程中，可能由于某个软件包的更新导致文件系统变为只读状态，此时 reboot 重启后重新更新就可。



```
mv /etc/apt/sources.list /etc/apt/sources.list.bak  #备份

wget -O /etc/apt/sources.list https://repo.huaweicloud.com/repository/conf/Ubuntu-Ports-bionic.list

apt-get update
```

使用说明

Ubuntu的仓库地址为：https://mirrors.huaweicloud.com/ubuntu/
Ubuntu-CD的镜像地址为：https://mirrors.huaweicloud.com/ubuntu-cdimage/
Ubuntu-Cloud的镜像地址为：https://mirrors.huaweicloud.com/ubuntu-cloud-images/
Ubuntu-Ports的仓库地址为：https://mirrors.huaweicloud.com/ubuntu-ports/
Ubuntu-Releases的镜像地址为：https://mirrors.huaweicloud.com/ubuntu-releases/

1、备份配置文件：

sudo cp -a /etc/apt/sources.list /etc/apt/sources.list.bak

2、修改**sources.list**文件，将**http://archive.ubuntu.com**和**http://security.ubuntu.com**替换成**http://mirrors.huaweicloud.com**，可以参考如下命令：

sudo sed -i "s@http://.*archive.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list
sudo sed -i "s@http://.*security.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list

3、执行**apt-get update**更新索引





### 设置时区

执行以下命令：

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone
```

执行 date -R 查看时间是否正确。



### 开启 BBR

执行以下命令：

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

通过以下命令检查 BBR 是否启动：

```
sysctl net.ipv4.tcp_available_congestion_control
# 返回值应为 net.ipv4.tcp_available_congestion_control = reno cubic bbr
sysctl net.ipv4.tcp_congestion_control
# 返回值应为 net.ipv4.tcp_congestion_control = bbr
sysctl net.core.default_qdisc
# 返回值应为 net.core.default_qdisc = fq
lsmod | grep bbr
# 返回值应包含 tcp_bbr 模块
```



### 关闭 serial-getty@ttyS0 

syslog 中每 10s 出现一次 ttyS0 服务启动失败的日志。不理它也没关系，但看着不舒服，所以：

```
systemctl stop serial-getty@ttyS0
systemctl disable serial-getty@ttyS0
systemctl stop syslog.service
systemctl disable syslog.service
```



### 禁止/var/log日志

因为emmc存储是一种flash存储技术，其写入寿命非常有限，所以系统运行中应尽量避免数据写入。

如果我们没有装什么特殊程序的话，通常来说数据的主要写入就是/var/log目录的日志了，一天几十MB还是有的。

armbian其实已经考虑了这个问题，因为armbian就是给arm架构订制的debian发行版嘛，所以它默认是创建了一个内存盘（zram文件系统）挂载到了/var/log目录：

```
root@aml:/var/log# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            469M     0  469M   0% /dev
tmpfs           184M   22M  163M  12% /run
/dev/mmcblk1p2  6.4G  2.1G  4.3G  33% /
tmpfs           920M     0  920M   0% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           920M     0  920M   0% /sys/fs/cgroup
tmpfs           920M  8.0K  920M   1% /tmp
/dev/mmcblk1p1  122M   58M   64M  48% /boot
/dev/zram0       49M   15M   31M  32% /var/log
tmpfs           184M     0  184M   0% /run/user/00
```

所以频繁的日志写入并不会直接伤害到emmc。

解决方法：

修改/usr/lib/armbian/armbian-ramlog脚本

```
syncToDisk () {
	# no sync to protect emmc
	return 0
	isSafe
 
	echo -e "\n\n$(date): Syncing logs from $LOG_TYPE to storage\n" | $LOG_OUTPUT
 
	if [ "$USE_RSYNC" = true ]; then
		${NoCache} rsync -aXWv --delete --exclude armbian-ramlog.log --links $RAM_LOG $HDD_LOG 2>&1 | $LOG_OUTPUT
	else
		${NoCache} cp -rfup $RAM_LOG -T $HDD_LOG 2>&1 | $LOG_OUTPUT
	fi
 
	sync
}
```



### Swap



使用 dd if 命令生成一个名为 `swapfile` 的空文件，放在 File System 下。

```
dd if=/dev/zero of=/swapfile bs=1M count=2203
```

标记为 Swap 文件并挂载。

```
chmod 777 /swapfile
mkswap /swapfile
swapon /swapfile
```

执行 free -m，看到 Swap 已经被应用。

```
root@aml:/# free -m
              total        used        free      shared  buff/cache   available
Mem:           1838         187         925           5         725        1557
Swap:          1135           5        1129
```

由于需要开机挂载 /swapfile，很不方便，所以将其写入 /etc/fstab，实现开机自动挂载。

```
nano /etc/fstab
```

在该文件末尾追加挂载 /swapfile 的配置。

```
/swapfile	none		swap		defaults				0 0
```

保存并退出。

## 环境部署及网站搭建



### LNMPA

在这里我选择的是军哥的lnmp一键包，当然你可以直接选择apt安装速度会快很多，只是易用性差点。

军哥LNMP一键包主页：https://lnmp.org/install.html

#### 相关参数

无人值守模式 & LNMP架构&MySQL 5.5 &  启用InnoDB & 数据库Root 用户密码：strelizia & PHP 7.2 & 内存分配器 Jemalloc & Apache 2.4 & 管理员邮箱：**** & 离线安装

#### 命令

```
wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="2" DB_Root_Password="althaea" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp


wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="10" DB_Root_Password="strelizia" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp
```

#### pathinfo设置

解决typecho后台或其它404错误

修改/usr/local/nginx/conf/enable-php.conf 文件，添加pathinfo2.conf

```
location ~ [^/]\.php(/|$)
     {
            try_files $uri =404;
            fastcgi_pass  unix:/tmp/php-cgi.sock;
            fastcgi_index index.php;
            include fastcgi.conf;
            include pathinfo2.conf;
     }
```

pathinfo2.conf

```
set $real_script_name $fastcgi_script_name;
if ($fastcgi_script_name ~ "(.+?\.php)(/.*)") {
set $real_script_name $1;
set $path_info $2;
}
fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
fastcgi_param SCRIPT_NAME $real_script_name;
fastcgi_param PATH_INFO $path_info;
```

如果除首页外全部404，则为伪静态规则问题，请使用以下伪静态规则：

```
location /
{
index index.html index.php;
if (-f $request_filename/index.html){
rewrite (.*) $1/index.html break;
}
if (-f $request_filename/index.php){
rewrite (.*) $1/index.php;
}
if (!-f $request_filename){
rewrite (.*) /index.php;
}
}
```



### FRP外网映射

https://github.com/fatedier/frp

#### FRP自启设置

创建frpc进程（注意服务端改成frps）

```
vi /lib/systemd/system/frps.service
```

将下面的内容中路径修改为对应你服务器上的文件路径，并填进去（注意不要使用/root目录，可能权限不足无法启动）

```
[Unit]
Description=Frp Client Service
After=network.target 
[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/frp_0.27.0/frpc -c /usr/local/frp_0.27.0/frpc.ini
ExecReload=/usr/local/frp_0.27.0/frpc reload -c /usr/local/frp_0.27.0/frpc.ini 
[Install]
WantedBy=multi-user.target
```

启动FRPC

```
systemctl start frpc
```

启用开机自启（FRPS同理）

```
systemctl enable frpc
```



### 挂载NTFS/exFAT设备

获得NTFS分区设备名，执行下面的命令：

```
fdisk -l | grep NTFS		#结果如下所示：
/dev/sda1          65 1948282287 1948282223 929G 7 HPFS/NTFS/exFAT
```

建立装载点：

```
mkdir /disk      #建立/disk目录路径
```

给予特定的访问权限

```
chmod 777 /disk
```

装载NTFS分区，可以使用下面的命令以读写方式临时装载一个NTFS分区到装载点

```
mount -t ntfs-3g <NTFS Partition> <Mount Point>
其中：
   <NTFS Partition>  ------ NTFS所在分区的设备名，如/dev/sda1
   <Mount Point>    ------ 装载点，如/media/share

参考：
mount -t ntfs-3g /dev/sda1 /disk
```

系统启动时装载NTFS分区，编辑：

```
vi /etc/fstab
在文件最后增加如下格式的行
<NTFS Partition> <Mount Point> ntfs-3g defaults 0 0
其中：
 <NTFS Partition>  ------ NTFS所在分区的设备名，如1.1中的/dev/sda1
 <Mount Point>    ------ 装载点，如1.2中的/media/share

参考：
/dev/sda1 /disk ntfs-3g defaults 0 0
```

```
附录：卸载挂载区（备用）
   umount /dev/sda1
```



### 使用udev自动挂载

编辑 /etc/udev/udev.conf 在最后添加

```
udev_root="/dev/"
udev_rules="/etc/udev/rules.d"
udev_log="err"
```

编辑 /etc/udev/rules.d/11-usbmount.rules
文件名可以自定义 但要以.rules结尾

```
KERNEL=="sd[a-z][0-9]", RUN+="/etc/udev/mount_usb.sh $env{ACTION} %k"
```

```
mkdir -p /usbdisk		#新建挂载目录
```

编辑 /etc/udev/mount_usb.sh
由于系统问题导致不能使用udev自动挂载ntfs、exfat格式的usb存储设备，暂时未找到解决方法！ntfs格式需要安

装ntfs-3g exfat需要安装fuse-exfat和exfat-utils

```
#!/bin/bash

usbdisk=/usbdisk
if [ "$1" == "add" ]; then
        ID_FS_TYPE=$(blkid -sTYPE -ovalue /dev/$2)
        case $ID_FS_TYPE in
                vfat)
                        mount -t vfat -o noatime,umask=0,iocharset=utf8 /dev/$2 $usbdisk > /dev/null 2>&1
                        sync
                        ;;
                ext[2-4])
                        mount -o noatime /dev/$2 $usbdisk >/dev/null 2>&1
                        sync
                        ;;
        #        exfat)
        #                mount -t exfat -o noatime,umask=0,iocharset=utf8 /dev/$2 $usbdisk > /dev/null 2>&1
        #                sync
        #                ;;
        #        ntfs)
        #                mount -t ntfs-3g -o noatime,umask=0,iocharset=utf8 /dev/$2 $usbdisk > /dev/null 2>&1
        #                sync
        #                ;;
                *)
                        exit 0
                        ;;
        esac
elif [ "$1" == "remove" ]; then
        sync
        umount -f $usbdisk
fi
```

```
chmod a+x /etc/udev/mount_usb.sh		#给脚本执行权限
```

编辑 /lib/systemd/system/systemd-udevd.service

```
MountFlags=shared

systemctl restart udev		#重启服务
```



## 软件安装



### 网页版管理cockpit

```
apt-get install cockpit
#安装后浏览器打开IP:9090
```



### Samba

修改smb.conf文件

```
vi /etc/samba/smb.conf
```

可以删除[printers]及[profiles]

参考模板：

```
[global]
   security = user
   map to guest = bad user


[media]
   comment = Storage
   guest ok = yes
   writable = yes
   browseable = yes
   write list = share
   public = yes
```

重启服务

```
service smbd restart
```



### MiniDlna



#### 安装MiniDlna

```
armbian-config
```

#### 配置相关参数

查看硬盘挂载路径

```
df -h
```

编辑文件

```
vi /etc/minidlna.conf
```

参考：

```
media_dir=A,/sharedfolders/abc
#其中A,代表是音乐文件。/sharedfolders/abc  是挂载硬盘的路径，根据自己的查看结果修改，不指定媒体文件A,P,V,可以不加

举例：
media_dir=A,/sharedfolders/abc        #音乐文件目录
media_dir=P,/sharedfolders/abc        #图像文件目录
media_dir=V,/sharedfolders/abc        #视频文件目录

```

重启，访问localhost:8200检验数据



### Docker

#### 安装docker



```
curl -fsSL https://get.docker.com | bash
```

修改默认存储空间及镜像加速

修改docker默认存储空间到移动硬盘的方法.默认docker存储目录在/var/lib/docker，就是把这个文件夹移到移动硬盘下，然后创建一个软链接到移动硬盘(软连接貌似不好使，现修改如下)。

```
systemctl stop docker
cd /var/lib
mv /var/lib/docker /储存目录
ln -s /储存目录 /var/lib/docker
vim /etc/docker/daemon.json

{

	"graph": /储存目录",
	"storage-driver": "overlay" ,
	"registry-mirrors": ["https://wdq1zej0.mirror.aliyuncs.com"]
	
}
	
systemctl start docker
docker info 
#查看docker的默认存储目录-Docker Root Dir: /储存目录
```



#### 安装图形化管理工具Portainer 

```
docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer-ce:linux-arm64
```



#### docker镜像

80x86/typecho:latest

```
docker run -d -p 8000:80 --name typecho --restart=always -v /media/typecho:/data 80x86/typecho:latest
```



killadm/aarch64-emqx:30

```
docker run -d --restart=always --name="emqx" -v /etc/localtime:/etc/localtime -p 18083:18083 -p 1883:1883 killadm/aarch64-emqx:30
```



nodered/node-red

```
docker run -d --name="node-red" --net=bridge --restart always -e TZ="Asia/Shanghai" -p 1880:1880/tcp -v /home/node-red:/data:rw nodered/node-red
```



homeassistant/aarch64-homeassistant:latest

```
docker run -d --restart=always --name="home-assistant" -e TZ=Asia/Shanghai -v /home/homeassistant:/config -p 8123:8123 -v /etc/localtime:/etc/localtime:ro --net=hassio homeassistant/home-assistant:latest
```



80x86/filebrowser:latest

```
docker pull 80x86/filebrowser:latest
```



docker run -d --name=clash -v "/root/.config/clash/config.yaml:/root/.config/clash/config.yaml" --net=host --restart=unless-stopped dreamacro/clash



80x86/qbittorrent:latest

```
docker run -d  --name="qbittorrent" --restart=always -v /downloads:/downloads /media/qbittorrent/data:/data /media/qbittorrent/config:/config 80x86/qbittorrent:4.3.5-alpine-3.13.5-arm64-full
```



unifreq/openwrt-aarch64

```
1. 拉取镜像
这里使用的 F 大的镜像，感谢！ 原帖地址https://www.right.com.cn/forum/thread-958173-1-1.html
docker pull unifreq/openwrt-aarch64

默认拉取最新的镜像

2. 打开网卡的混杂模式
ip link set eth0 promisc on


3. 创建 macvlan 网络
docker network create -d macvlan --subnet=192.168.x.0/24 --gateway=192.168.x.1 -o parent=eth0 macnet

注意：这里需要根据实际网络来填写网关和子网掩码，如果主路由的 ip 地址为 192.168.0.1，则将上面的 192.168.x 改为 192.168.0

4. 运行 OpenWrt
docker run --name op --restart always -d --network macnet --privileged unifreq/openwrt-aarch64 /sbin/init


5. 修改 OpenWrt 的网络设置
docker exec -it op bash

vim /etc/config/network

```

![img](https://www.right.com.cn/forum/data/attachment/forum/202009/12/103327phhvothitots2xss.jpg)



修改图中两处红框，
第一处修改为需要访问 OpenWrt 的 ip 地址（前三未数字需要和主路由相同，最后一位数字随意修改，不要和其他设备冲突就行）
第二出修改为主路由 ip 地址

退出，保存：
（按 ESC 键，输入

1. :wq

*复制代码*


，回车）

\7. 重启 OpenWrt 的网络

1. /etc/init.d/network restart

*复制代码*



\8. 此时可以在浏览器访问第 6 步中第一个红框处填写的地址访问 OpenWrt
默认账户：root，默认密码：password

\9. 设置 OpenWrt
9.1 关闭 DHCP
网络 -> 接口 -> LAN/修改

![img](https://www.right.com.cn/forum/data/attachment/forum/202009/12/104608xp2u55mzwffb3zid.jpg)


基础设置

![img](https://www.right.com.cn/forum/data/attachment/forum/202009/12/105052oheteacspscctuxa.jpg)


9.2 关闭桥接
物理设置

![img](https://www.right.com.cn/forum/data/attachment/forum/202009/12/105208lnngan3nnfgabjga.jpg)


保存即可


![img](https://www.right.com.cn/forum/static/image/hrline/line4.png)


到这里 OpenWrt 安装并且已经设置完毕，可以日用了，下面还有一些附加设置可以选择。

\1. 设置 armbian 访问 OpenWrt
在 armbian 下修改 /etc/network/interfaces 文件，替换为以下内容

1. auto eth0
2. iface eth0 inet static
3. address 192.168.0.x
4. netmask 255.255.255.0
5. gateway 192.168.0.1

*复制代码*

x 代表的是你需要设置的 n1 armbian 系统的 Ip 地址
然后重新载入 networking

1. systemctl reload networking

*复制代码*

如果重载失败，请使用

1. systemctl status networking

*复制代码*

查看问题

\2. 低调上网
为什么低调上网需要单独说？因为我在使用过程中发现 PassWall 和 违禁软件 plus+ 设置好了节点之后，节点可以访问，状态也是正常，但是 op 却无法连接，甚至连接百度失败

原因：默认的 DNS 设置有问题
解决方案：

![img](https://www.right.com.cn/forum/data/attachment/forum/202009/12/110523sqpu4cun7trjz9zx.jpg)


PassWall 一定不要使用 【默认】和【运营商 DNS（自动分配）】，使用其他 DNS 即可

adguard/adguardhome:arm64-latest

```
docker pull adguard/adguardhome:arm64-latest
docker run --name adguardhome -p 53:53/tcp -p 53:53/udp -p 67:67/udp -p 69:68/tcp -p 69:68/udp -p 3339:80/tcp -p 444:443/tcp -p 853:853/tcp -p 3333:3000/tcp -d adguard/adguardhome:arm64-latest
```

访问localhost:3333 初始化设置 

访问localhost:3339 进入adguardhome控制面板

外加一个网址：https://github.com/otobtc/ADhosts  作者：[otobtc](https://github.com/otobtc)

 注意：https的443端口被映射成了444，启用https解析时端口不要写错

下面需要设置的地方：
上游 DNS 服务器：
114.114.114.114

Bootstrap DNS 服务器：
114.114.114.114:53

最后把要启用设备的DNS设置成 N1 的IP地址即可！！!



## JAVA

方法1.

```
a.下载jdk-8u151-linux-x64.tar.gz到download目录
http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html

b.安装jdk
cd download/
mkdir /usr/local/java
tar zxvf jdk-8u151-linux-x64.tar.gz -C /usr/local/java
ln -s /usr/local/java/jdk1.8.0_151/ /usr/local/java/latest

c. 添加环境变量：

vim /etc/profile
加入如下内容：
export JAVA_HOME=/usr/local/java/latest
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

执行
source /etc/profile

java -version #检验
```

方法2.

运行以下命令在您的Raspberry Pi上安装OpenJDK 11 JDK：

```
apt update && apt install default-jdk -y
```

安装完成后，通过检查Java版本进行验证：

```
java -version
```

输出应如下所示：

```
openjdk version "11.0.5" 2019-10-15
OpenJDK Runtime Environment (build 11.0.5+10-post-Raspbian-1deb10u1)
OpenJDK Server VM (build 11.0.5+10-post-Raspbian-1deb10u1, mixed mode)
```

方法3.

以前的Java LTS版本8仍受支持并被广泛使用。 如果您的应用程序需要Java 8，请输入以下内容进行安装：

```
sudo apt update && apt install openjdk-8-jdk -y
```

通过打印Java版本来验证安装：

```
java -version
```

输出应如下所示：

```
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-8u212-b01-1+rpi1-b01)
OpenJDK Client VM (build 25.212-b01, mixed mode)
```

## Python3.7

**改变默认python版本**

现成的aarch64的tensorflow轮子只支持python2.7和python3.5。系统默认使用的是2.7，所以要切换到3.7。

```
echo alias python=python3 >> ~/.bashrc
source ~/.bashrc
```

这时候可以查看一下版本

```
python -V
```

**SyntaxError**

```
# sudo vim /etc/profile
```

在末尾添上：

```
export LC_ALL="en_US.UTF-8"
export LANG="zh_CN.GBK"
```

**SSH 无法显示和输入中文**

```
vi /etc/environment
	ARCH=arm64
	LC_ALL=″en_US.UTF-8″

# 生效
source /etc/environment
```

**切换至3.7.7**

0.安装依赖

```
apt install libgdal-dev build-essential libatlas-base-dev libgdal-dev libdb-dev libdb5.3-dev libxslt1-dev  libncurses5-dev -y

apt install libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev zlibc zlib1g-dev libhdf5-dev libxml2-dev -y

apt install libdb-dev libdb5.3-dev libreadline-dev libpcap-dev libpcap0.8 libpcap0.8-dev libhdf5-dev libxml2-dev -y

apt dist-upgrade

apt install build-essential  libncursesw5-dev libgdbm-dev libc6-dev zlib1g-dev libsqlite3-dev tk-dev libssl-dev openssl gfortran  liblapack-dev libhdf5-dev libzbar-dev libbz2-dev -y
```

1.下载源码

```
wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz
```

2.命令行切换到上面压缩文件所在的目录（比如桌面），然后输入 `tar -xzf Python-3.7.7.tgz`

> 这里 tar表示解压缩，-x 表示从档案文件中释放文件，z 表示用 gzip 解压（用于 xx.tgz 以及 xx.tar.gz 格式的压缩包），f 后面是压缩文件名。

3.命令行目录切换到解压后的文件夹中，也就是 Python-3.7.7 文件夹。然后执行 

```
mkdir -p /usr/local/python3

./configure --prefix=/usr/local/python3 --with-ssl --enable-optimizations 

#执行这步是后面最好加上 --enable-optimizations 会自动安装pip3及优化配置
#在./configure过程中，如果没有加上–with-ssl参数时，默认安装的软件涉及到ssl的功能不可用，
#这个命令的作用是生成 Makefile 文件，以供下一步的 make 命令使用。
#Makefile 文件存储的时构建 (build) 顺序，linux build 程序组件时需要按照 Makefile 指定的顺序。
```

4.

```
make -j2 
#编译源代码，并生成执行文件
make install
#是把生成的执行文件拷贝到 linux 系统中必要的目录下，比如拷贝到 `usr/local/bin` 目录下，这样所有的用户都可以运行这个程序了。
```

5.删除原有的软连接

```
python3 -V

pip3 -V

rm -rf /usr/bin/python3

rm -rf /usr/bin/pip3
```

6.建立新的指向python3.7的软链接

```
#添加python3的软链接 

ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3 

#添加 pip3 的软链接 

ln -s /usr/local/python3/bin/pip3.7 /usr/bin/pip3
```

7.检测版本

```
root@arm-64:/usr/local/python3/bin# python3 -V
Python 3.7.7
root@arm-64:/usr/local/python3/bin# pip3 -V
pip 19.2.3 from /usr/local/python3/lib/python3.7/site-packages/pip (python 3.7)
```

8.升级pip/setuptool

```
pip3 install --upgrade pip setuptools
```



安装pip后，无法使用pip安装一些包，总是会出现上述错误

```
subprocess.CalledProcessError: Command '('lsb_release', '-a')' returned non-zero exit status 1.


```

```
解决方案：

找到lsb_release.py文件和CommandNotFound目录，把它们拷贝到报的错误中subprocess.py所在文件夹

cp  /usr/lib/python3/dist-packages/lsb_release.py /usr/local/python3/lib/python3.7/

同时还需要将CommandNotFound所在的目录复制到上面相同的目录下面

sudo cp -fr /usr/lib/python3/dist-packages/CommandNotFound   /usr/local/python3/lib/python3.7/

将__pycache__子目录中的文件名中带有38名字的文件更改为37


网上参考文献[2]中说，执行下面的命令也可以解决这个问题：

sudo rm /usr/bin/lsb_release

我个人不赞成这么做，因为这破坏了系统的完整性，将系统中的这个命令删去了。

```



**第三方库**

```
pip3 install wheel
pip3 install nose
pip3 install requests
pip3 install pyzbar
pip3 install Pillow
pip3 install prettytable
pip3 install selenium
pip3 install beautifulsoup4 
pip3 install lxml
pip3 install numpy
pip3 install parsel
pip3 install twisted
pip3 install w3lib
pip3 install cryptography
pip3 install pyOpenSSL
pip3 install protego
pip3 install Scrapy
pip3 install scipy
pip3 install mock
pip3 install sklearn
```



## TensorFlow

```
apt install libhdf5-dev cmake libgtk-3-dev libboost-all-dev -y
pip3 install dlib
pip3 install numpy
pip3 install h5py
pip3 install scipy
pip3 install keras
pip3 install nltk
pip3 install tensorflow   # 踩雷警告


```

检测一下安装是否成功

```
python3 -c "import keras,tensorflow"
```



## OpenCV

**依赖安装**

```
#安装所需要的工具和包：
apt install build-essential cmake git gcc libopencv-dev build-essential libgtk2.0-dev pkg-config libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev -y 

apt install libjpeg-dev libtiff5-dev libavcodec-dev libavformat-dev libswscale-dev libxine2-dev libtbb-dev libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev -y

apt install libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libavcodec-dev libv4l-dev libtheora-dev libvorbis-dev libxvidcore-dev libgtk-3-dev libavformat-dev -y

apt install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev libavresample-dev libswscale-dev libgphoto2-dev libopenblas-dev doxygen liblapacke-dev checkinstall libgtk2.0-dev libtbb2 -y
```

**方法1**

```
1.下载
wget https://github.com/opencv/opencv/archive/4.3.0.zip
wget https://github.com/opencv/opencv_contrib/archive/4.3.0.zip

2.解压
unzip opencv-4.3.0.zip -d ~opencv
unzip opencv_contrib-4.3.0.zip -d ~opencv

3.创建并进入 build 目录,执行 cmake 生成 makefile 文件

cd opencv/opencv-4.3.0
mkdir build && cd build/
sudo cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local -D OPENCV_EXTRA_MODULES_PATH= ..
例：

cd opencv/opencv-4.3.0
mkdir build && cd build/
cmake -D CMAKE_INSTALL_PREFIX=/usr/local -D CMAKE_BUILD_TYPE=Release -D OPENCV_EXTRA_MODULES_PATH=~/opencv/opencv_contrib-4.5.3/modules ..

screen -S opencv
make -j2 && make install

```

![img](https://images2017.cnblogs.com/blog/1070077/201801/1070077-20180106104253049-400444754.png)

**方法2**

```
unzip opencv-3.0.0.zip
cd opencv-3.0.0/
mkdir build
cd build/
cmake ..
make -j2
```

**方法3**

```

安装依赖库ffmpeg
    cd ffmpeg/
    ./configure --disable-yasm  --enable-shared --enable-pic --prefix=/usr/local/ffmpeg 生成可连接库，--prefix设置安装路径
    make
    make install 开始安装
vim /etc/profile 打开环境变量文档
    在文尾输入：
    export FFMPEG_HOME=/usr/local/ffmpeg
    export PATH=$FFMPEG_HOME/bin:$PATH
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib #添加动态库路径
    export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/ffmpeg/lib/pkgconfig:/usr/loal/lib
    ffmpeg -version  如果显示版本信息，就证明ffmpeg已经成功安装了
    
首先检测你的环境是否配置成功。
    pkg-config  ffmpeg --libs --cflags查看ffmpeg链接库是否配置好了如果没配置好也别着急，继续往下看）
    pkg-config opencv --libs --cflags查看opencv链接库和头文件配置
如果没有打印程序的链接库路径，说明链接库没有完整配置好
    这个时候首进入ffmpeg文件目录（不是源码目录，是软件安装目录/usr/local/ffmpeg/）/lib的目录的
    所有文件复制到/usr/local/lib目录下；然后打开ffmpeg/lib/的里有个pkgconfig，
    把里头的文件全部复制到/usr/local/lib/pkgconfig里头

编译源码：
    1.cd opencv-3.3.0
    2.mkdir build  
    3.cd build  
    4.cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..  编译通过
    
    4.1.cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local -D OPENCV_EXTRA_MODULES_PATH=/home/firstuser/depend/opencv-3.3.0/opencv_contrib-3.3.0/modules/ -D WITH_TBB=ON -D BUILD_SHARED_LIBS=OFF -D WITH_OPENMP=ON -D ENABLE_PRECOMPILED_HEADERS=OFF ..
    
    4.2.cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_TBB=ON -D BUILD_SHARED_LIBS=OFF -D WITH_OPENMP=ON -D ENABLE_PRECOMPILED_HEADERS=OFF ..
      
    5.make -j4 多线程
    6.sudo make install  
    
修改ippicv下载路径：
    vim /home/firstuser/depend/opencv-3.3.0/3rdparty/ippicv/ippicv.cmake #根据自己的路径填写
    将47行的
    "https://raw.githubusercontent.com/opencv/opencv_3rdparty/${IPPICV_COMMIT}/ippicv/"
    改为手动下载的文件的本地路径：
    "file:///home/firstuser/下载/" （根据自己的路径填写）
    到了下载ippicv那一步时会自动从本地下载。
错误及解决方法:
    0.error: ‘CODEC_FLAG_GLOBAL_HEADER’ was not declared in this scope
    error: ‘AVFMT_RAWPICTURE’ was not declared in this scope

    在/opt/opencv/opencv-3.3.0/modules/videoio/src/cap_ffmpeg_impl.hpp 里最顶端添加
    #define AV_CODEC_FLAG_GLOBAL_HEADER (1 << 22)
    #define CODEC_FLAG_GLOBAL_HEADER AV_CODEC_FLAG_GLOBAL_HEADER
    #define AVFMT_RAWPICTURE 0x0020

    1.-- No package 'gtk+-3.0' found
    sudo apt-get install libgtk-3-dev

    2.--   No package 'gstreamer-base-1.0' found
    --   No package 'gstreamer-video-1.0' found
    --   No package 'gstreamer-app-1.0' found
    --   No package 'gstreamer-riff-1.0' found
    --   No package 'gstreamer-pbutils-1.0' found
    sudo apt-get -y install libgstreamer-plugins-base1.0-dev
    sudo apt-get -y install libgstreamer1.0-dev

    3.--   No package 'libavresample' found
    --   No package 'libgphoto2' found
    sudo apt-get -y install libavresample-dev
    sudo apt-get -y install libgphoto2-dev

    4.-- Could not find OpenBLAS include. Turning OpenBLAS_FOUND off
    -- Could not find OpenBLAS lib. Turning OpenBLAS_FOUND off
    -- Could NOT find Atlas (missing:  Atlas_CBLAS_INCLUDE_DIR Atlas_CLAPACK_INCLUDE_DIR             Atlas_CBLAS_LIBRARY Atlas_BLAS_LIBRARY Atlas_LAPACK_LIBRARY)
    -- Could NOT find Doxygen (missing:  DOXYGEN_EXECUTABLE)
    -- Could NOT find JNI (missing:  JAVA_AWT_LIBRARY JAVA_JVM_LIBRARY JAVA_INCLUDE_PATH             JAVA_INCLUDE_PATH2 JAVA_AWT_INCLUDE_PATH)
    -- Could NOT find Matlab (missing:  MATLAB_MEX_SCRIPT MATLAB_INCLUDE_DIRS MATLAB_ROOT_DIR         MATLAB_LIBRARIES MATLAB_LIBRARY_DIRS MATLAB_MEXEXT MATLAB_ARCH MATLAB_BIN)
    -- VTK is not found. Please set -DVTK_DIR in CMake to VTK build directory, or to VTK install         subdirectory with VTKConfig.cmake file
    sudo apt-get install libopenblas-dev

    5.-- Could NOT find Doxygen (missing:  DOXYGEN_EXECUTABLE)
    sudo apt-get install doxygen

    6.-- Could NOT find JNI (missing:  JAVA_AWT_LIBRARY JAVA_JVM_LIBRARY JAVA_INCLUDE_PATH             JAVA_INCLUDE_PATH2 JAVA_AWT_INCLUDE_PATH)

    sudo mkdir /usr/local/java
    sudo tar zxvf jdk-8u151-linux-x64.tar.gz -C /usr/local/java
    sudo ln -s /usr/local/java/jdk1.8.0_151/ /usr/local/java/latest
    sudo vim /etc/profile
    export JAVA_HOME=/usr/local/java/latest
    export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$PATH:$JAVA_HOME/bin
    source /etc/profile

cd ~/opencv-3.3.0/build
make clean
重新编译：
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..  编译通过
    make -j8
    make install


附3.4版本以上有该问题：
/usr/bin/ld: warning: libswresample.so.3, needed by //usr/local/ffmpeg/lib/libavcodec.so.58, not found (try using -rpath or -rpath-link)
//usr/local/ffmpeg/lib/libavcodec.so.58：对‘swr_close@LIBSWRESAMPLE_3’未定义的引用
解决方法:
https://blog.csdn.net/guo_lei_lamant/article/details/82561312
vim /etc/ld.so.conf.d
/usr/local/ffmpeg/lib

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

重装新版本:先卸载旧版本
    1.删除安装文件
    cd /home/***/opencv/build
    sudo make uninstall
    cd  ..
    sudo rm -r build
    2.删除所有和opencv相关文件
    sudo rm -r /usr/local/include/opencv /usr/include/opencv /usr/include/opencv2
    cd /usr
    find . -name "*opencv*" | xargs sudo rm -rf
     3.删除代码包
    cd /home/***
    chmod a+x /home/***/opencv
    rm -r /home/***/opencv
    4.
    cd ~/opencv-3.4.6

error while loading shared libraries: libopencv_core.so.3.4: cannot open shared object file: No such file or directory
    1.打开路径:/etc/ld.so.conf.d
    2.创建文件:OpenCV.conf文件
    3.添加自己opencv的lib路径 ldconfig (通常为/usr/local/lib)
error: ./TopCamDetDL.so: undefined symbol: _ZN2cv3dnn23experimental_dnn_34_v143NetC1Ev

　　opencv版本问题
重新编译：
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..  编译通过
    make -j2
    make install
```

**No module named cv2**
原因在于 python2.7 找不到 cv2.so 文件

```
cp ~/opencv/opencv-3.2.0/build/lib/cv2.so ~/anaconda2/lib/python2.7/site-packages/
```

**测试**

```
$ python
Python 2.7.14 |Anaconda, Inc.| (default, Oct 16 2017, 17:29:19) 
[GCC 7.2.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import cv2
>>> cv2.__version__
>>> '3.2.0'
```

## 摄像头

**查看/摄像头设备**

```
root@arm-64:~# ls /dev/video*
/dev/video0  /dev/video1  /dev/video2
root@arm-64:~# lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 0781:5583 SanDisk Corp. Ultra Fit
Bus 001 Device 003: ID 1871:0101 Aveo Technology Corp. UVC camera (Bresser microscope)
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

```

**安装软件**检测一下安装是否成功

```
apt install fswebcam

执行命令试拍一张看看效果：

fswebcam -d /dev/video0 --no-banner -r 320x240 /home/pi/image.jpg

或者你可以直接用

fswebcam image.jpg

可以直接拍照，-d是配置使用哪个摄像设备，–no-banner应该是水印相关，不加的话，可能会报字体问题， -r后的是图片的宽与高，最后的是待保存的图片路径。
PS:!!!!!!!以下问题及解决百度没有答案….
我在实际使用时拍摄出的照片是纯黑，谷歌了半天找到原因，需要加-S参数，控制拍照延时。我使用的是10，实际它并不是命令等待10秒后才拍摄，大约只等待1s左右，不影响使用。

fswebcam -S 10 image.jpg

现在拍照正常了，yeah!
```

```
apt install subversion imagemagick libjpeg8 libjpeg8-dev libv4l-dev git cmake
```



## 杂项

#### 取消自动登录

/etc/lightdm/lightdm.conf.d/22-armbian-autologin.conf

```
[Seat:*]
autologin-user=orca
autologin-user-timeout=0
user-session=xfce

注释 autologin-user=<YOUR USER>
```



#### crontab

**安装**

如果没有安装crontab，请先安装。

```
apt install crontab
```

**设置**

接着可以使用下面命令开关crontab

```
service crond start/stop/restart/reload    #启动/关闭/重启/重载
```

查看crontab服务状态：

```
service crond status
```

手动启动crontab服务：

```
service crond start
```

查看crontab服务是否已设置为开机启动，执行命令：ntsysv

如果列表中有crond，而且前面有[*]就说明已经添加了开机启动。如果没有可以按照下面的步骤执行。

使用tab键可以切换到确认取消栏。

加入开机自动启动:

```
chkconfig crond on
```

**用法**

**功能说明**：设置计时器。

**语　　法**：crontab [-u <用户名称>][配置文件] 或 crontab [-u <用户名称>][-elr]

**补充说明**：cron是一个常驻服务，它提供计时器的功能，让用户在特定的时间得以执行预设的指令或程序。只要用户会编辑计时器的配置文件，就可以使 用计时器的功能。其配置文件格式如下：
Minute Hour Day Month DayOFWeek Command
**参　　数**：
*-e* 编辑该用户的计时器设置。
*-l* 列出该用户的计时器设置。
*-r* 删除该用户的计时器设置。
*-u<用户名称>* 指定要设定计时器的用户名称。

**基本格式** :

```
*　 *　 *　*　 *   command
分　时　日　月　周　 命令
第1列表示分钟1～59 每分钟用*或者 */1表示
第2列表示小时1～23（0表示0点）
第3列表示日期1～31
第4列 表示月份1～12
第5列标识号星期0～6（0表示星期天）
第6列要运行的命令
# Use the hash sign to prefix a comment
# +—————- minute (0 – 59)
# | +————- hour (0 – 23)
# | | +———- day of month (1 – 31)
# | | | +——- month (1 – 12)
# | | | | +—- day of week (0 – 7) (Sunday=0 or 7)
# | | | | |
# * * * * * command to be executed
```

**举例**

```
30 21 * * * /etc/init.d/nginx restart
每晚的21:30重启 nginx。

45 4 1,10,22 * * /etc/init.d/nginx restart
每月1、 10、22日的4 : 45重启nginx。

10 1 * * 6,0 /etc/init.d/nginx restart
每周六、周日的1 : 10重启nginx。

0,30 18-23 * * * /etc/init.d/nginx restart
每天18 : 00至23 : 00之间每隔30分钟重启nginx。

0 23 * * 6 /etc/init.d/nginx restart
每星期六的11 : 00 pm重启nginx。

\* */1 * * * /etc/init.d/nginx restart
每一小时重启nginx

\* 23-7/1 * * * /etc/init.d/nginx restart
晚上11点到早上7点之间，每 隔一小时重启nginx

0 11 4 * mon-wed /etc/init.d/nginx restart
每月的4号与每周一到周三 的11点重启nginx

0 4 1 jan * /etc/init.d/nginx restart
一月一号的4点重启nginx

*/30 * * * * /usr/sbin/ntpdate 210.72.145.20
每半小时同步一下时间n
```



#### 修改/删除 .user.ini 

```
首先
了解chattr命令：Linux chattr命令用于改变文件属性。
这项指令可改变存放在ext2文件系统上的文件或目录属性，这些属性共有以下8种模式：
a：让文件或目录仅供附加用途。
b：不更新文件或目录的最后存取时间。
c：将文件或目录压缩后存放。
d：将文件或目录排除在倾倒操作之外。
i：不得任意更动文件或目录。
s：保密性删除文件或目录。
S：即时更新文件或目录i。
u：预防意外删除。
我们主要用的是 i 的模式
第一：切换到 .user.ini 目录
第二：使用命令 chattr -i .user.ini  解除文件不可更动属性，之后就可以修改/删除.user.ini这个文件了
第三：chattr +i .user.ini   重新恢复文件不可更动属性
```



#### 编译安装libdrm

```
sudo tar -xvf libdrm-2.4.58.tar.gz  
cd libdrm-2.4.58/  
sudo ./configure  
sudo make  
sudo make install  
```



#### 编译安装mesa

**安装依赖**

```
apt install flex bison python3-mako libwayland-egl-backend-dev libxcb-dri3-dev libxcb-dri2-0-dev  -y
apt install libxcb-glx0-dev libx11-xcb-dev libxcb-present-dev libxcb-sync-dev libxxf86vm-dev -y
apt install libxshmfence-dev libxrandr-dev libwayland-dev libxdamage-dev libxext-dev libxfixes-dev -y
apt install x11proto-dri2-dev x11proto-dri3-dev x11proto-present-dev x11proto-gl-dev -y
apt install x11proto-xf86vidmode-dev libexpat1-dev libudev-dev gettext  mesa-utils xutils-dev -y
apt install libpthread-stubs0-dev ninja-build bc python-pip flex bison cmake git valgrind -y
apt install llvm llvm-8-dev python3-pip  pkg-config zlib1g-dev wayland-protocols meson -y
```

**编译安装**

```
git clone git://anongit.freedesktop.org/mesa/mesa
cd mesa
meson -Ddri-drivers= -Dvulkan-drivers= -Dgallium-drivers=panfrost,kmsro -Dlibunwind=false -Dprefix=/usr build/
ninja -C build/
sudo ninja -C build/ install
```

**运行 Glmark2 测试**

```
apt install libjpeg62-turbo-dev libpng-dev
git clone https://github.com/glmark2/glmark2.git
cd glmark2/
./waf configure --with-flavors=drm-gl,drm-glesv2,wayland-gl,wayland-glesv2
./waf
./waf install 
```



#### 安装python踩坑

**安装 numpy**

```
apt install python3-dev build-essential
pip install numpy  # 安装numpy

```

**安装 scipy**

```
# 安装gfortran,后面编译过程中会用到
apt-get install gfortran
# 安装blas,Ubuntu下对应的是libopenblas，其它操作系统可能需要安装其它版本的blas——这是个OS相关的。
apt-get install libopenblas-dev
# 安装lapack，Ubuntu下对应的是liblapack-dev，和OS相关。
apt-get install liblapack-dev
# 安装atlas，Ubuntu下对应的是libatlas-base-dev，和OS相关。
apt-get install libatlas-base-dev

pip install scipy
```

**安装 Pillow**

```
apt-get build-dep python-imaging   # 注意不是 sudo apt-get install build-dep python-imaging

pip install Pillow
```

**安装 h5py**

```
pip install cython   
apt-get install libhdf5-dev   #  PyPI里没有它，用 apt 下载 。 
pip install h5py
```



#### 升级openSSL和openSSH

**安装依赖**

```
apt-get install autoconf automake make build-essential libpam0g-dev libpcre3 libpcre3-dev zlib1g-dev libssl-dev libbz2-dev -y
```

```
下载的压缩包放在opt目录

cd /opt/

wget https://www.openssl.org/source/openssl-1.1.1h.tar.gz

wget http://ftp.jaist.ac.jp/pub/OpenBSD/OpenSSH/portable/openssh-7.7p1.tar.gz

解压

tar zxf openssl-1.1.1h.tar.gz

tar zxf openssh-7.7p1.tar.gz


进入openssl源码目录

cd openssl-1.1.1h/

./config shared zlib     #加上shared参数是生成共享库文件，不然后边源码安装openssl的时候会提示找不到文件

make -j $(cat /proc/cpuinfo| grep "processor"|wc -l) && make install

mv /usr/bin/openssl /root/    #移动原来的openssl执行文件到用户目录，这个可以随意放

rm -rf /usr/bin/openssl

ln -s /usr/local/bin/openssl /usr/bin/openssl     #新版的openssl做个软链

openssl version     #如果版本显示为新版则大功告成



删除/lib/x86_64-linux-gnu路径下旧版的libssl.so.1.0.0和libcrypto.so.1.0.0文件

rm -rf /lib/x86_64-linux-gnu/libssl.so.1.0.0
rm -rf /lib/x86_64-linux-gnu/libcrypto.so.1.0.0

软链个新版的

ln -s /usr/local/lib/libssl.so.1.1 /lib/x86_64-linux-gnu/libssl.so.1.1
ln -s /usr/local/lib/libcrypto.so.1.1 /lib/x86_64-linux-gnu/libcrypto.so.1.1

引擎目录同理

mv /usr/lib/x86_64-linux-gnu/openssl-1.0.0/engines/ /usr/lib/x86_64-linux-gnu/openssl-1.0.0/engines.old/

 ln -s /usr/local/ssl/lib/engines /usr/lib/x86_64-linux-gnu/openssl-1.0.0/engines



好，继续安装新版ssh

cd /opt/openssh-7.7p1/

./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords --with-pam --with-ssl-dir=/usr/local/ssl --with-zlib=/usr/local/include

make -j $(cat /proc/cpuinfo| grep "processor"|wc -l) && make install

ssh -V    #如版本显示为新版本则大功告成

```

**疑难杂症**

```
报错：
openssl: /usr/lib/x86_64-linux-gnu/libssl.so.1.1: version OPENSSL_1_1_1’ not found (required by openssl)
查了一下，主要是LD_LIBRARY_PATH这个环境变量没有指定导致openssl正在使用旧的系统OpenSSL库

解决方法：

echo "export LD_LIBRARY_PATH=/usr/local/lib" >> ~/.bashrc
export LD_LIBRARY_PATH=/usr/local/lib

小贴士：在1.1.0或以上版本的时候，最好显示地指定--prefix 和 --openssldir，之前我只是直接默认地./config.除此以外还需要设置额外的CFLAGS才能正常地运行，举个栗子./config --prefix=/usr --openssldir=/usr --libdir=lib shared zlib-dynamic -Wl,-R,'$(LIBRPATH)' -Wl,--enable-new-dtags
```

#### Chromium安装flash播放器

```
下载： chromium-pepper-flash-12.0.0.77-1-armv7h.pkg.tar.xz
wget http://odroidxu.leeharris.me.uk/repo/chromium-pepper-flash-12-12.0.0.77-1-armv7h.pkg.tar.xz

解压 tar xf chromium-pepper-flash-12.0.0.77-1-armv7h.pkg.tar.xz
       chmod +x ./usr/lib/PepperFlash/*
       sudo cp ./usr/lib/PepperFlash/* /usr/lib/chromium-browser/plugins
       然后安装Chromium浏览器插件。
      sudo vi /etc/chromium-browser/default
        如下编辑文件

  CHROMIUM_FLAGS="--ppapi-flash-path=/usr/lib/chromium-browser/plugins/libpepflashplayer.so --ppapi-flash-ersion=12.0.0.77 -password-store=detect -user-data-dir"

强制保存，重新打开chromium浏览器，终于OK了

```

#### 开启X11转发

.1. 配置服务器的sshd，重启服务

```
# vi /etc/ssh/sshd_config

X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes

# service sshd restart

/usr/bin/xfce4-session
```



#### screen命令



```
1.1 创建screen会话
可以先执行：screen -S lnmp ，screen就会创建一个名字为lnmp的会话。 
1.2 暂时离开，保留screen会话中的任务或程序
当需要临时离开时（会话中的程序不会关闭，仍在运行）可以用快捷键Ctrl+a d(即按住Ctrl，依次再按a,d)
1.3 恢复screen会话
当回来时可以再执行执行：screen -r lnmp 即可恢复到离开前创建的lnmp会话的工作界面。如果忘记了，或者当时没有指定会话名，可以执行：screen -ls
screen会列出当前存在的会话列表
1.4 关闭screen的会话
执行：exit ，会提示：[screen is terminating]，表示已经成功退出screen会话。VPS侦探 https://www.vpser.net/
2、远程演示
首先演示者先在服务器上执行 screen -S test 创建一个screen会话，观众可以链接到远程服务器上执行screen -x test 观众屏幕上就会出现和演示者同步。
3、常用快捷键
Ctrl+a c ：在当前screen会话中创建窗口
Ctrl+a w ：窗口列表
Ctrl+a n ：下一个窗口
Ctrl+a p ：上一个窗口
Ctrl+a 0-9 ：在第0个窗口和第9个窗口之间切换
```

#### 解决无线MAC相同的问题

```
其实N1盒子的无线是有自己的MAC地址的，拆机后写在PCB的标签上，先抄下来。
用dmesg命令查看,有以下几条错误:
[    8.870556] platform regulatory.0: Direct firmware load for regulatory.db failed with error -2
...
[    8.889520] bluetooth hci0: Direct firmware load for brcm/BCM4345C0.hcd failed with error -2
[    8.889534] Bluetooth: hci0: BCM: Patch brcm/BCM4345C0.hcd not found
...
[    9.016529] brcmfmac mmc2:0001:1: Direct firmware load for brcm/brcmfmac43455-sdio.phicomm,n1.txt failed with error -2
原因是缺少相关文件，路径是/lib/firmware以及 /lib/firmware/brcm两个目录，其中，

1.  regulatory.db（wifi监管数据库）应该在/lib/firmware下，下载地址是 https://mirrors.edge.kernel.org/pub/software/network/wireless-regdb/
     解压后把regulatory开头的几个文件拷进去
2. brcm/BCM4345C0.hcd,应该在/lib/firmware/brcm下面，但实际文件是在 /lib/firmware下，用mv移动一下即可, 这个文件是蓝牙的固件

3. brcm/brcmfmac43455-sdio.phicomm,n1.txt, 这个文件应该在/lib/firmware/brcm下面, 但实际上不存在,只有 brcmfmac43455-sdio.txt, 打开 brcmfmac43455-sdio.txt查看，发现了MAC地址：
sromrev=11
vendid=0x14e4
devid=0x43ab
manfid=0x2d0
prodid=0x06e4
#macaddr=00:90:4c:c5:12:38
macaddr=b8:27:eb:74:f2:6c
nocrc=1
boardtype=0x6e4
boardrev=0x1304

把 brcmfmac43455-sdio.txt 拷贝到  brcmfmac43455-sdio.phicomm,n1.txt ，并编辑修改MAC地址即可，问题解决。
```

#### 统计当前目录下的文件和目录

Linux统计当前目录下有多少文件和目录

仅仅做个笔记：

统计当前目录下文件的个数，不包括子目录：

```
ls -l |grep "^-"|wc -l
```

统计当前目录下目录的个数，不包括子目录：

```
ls -l |grep "^d"|wc -l
```

统计当前目录下文件的个数，包括子目录：

```
ls -lR|grep "^-"|wc -l
```

统计当前目录下目录的个数，包括子目录

```
ls -lR|grep "^d"|wc -l
```





```
systemctl start armbian-resize-filesystem
```

