# Linux使用记录

2022/01/06
开始记录我在使用/学习Linux（Ubuntu/Debian）过程中的心得体会以及遇到的一些问题。
在我再回过头看时，也许会感到收获很多吧！

## 00  [Ambian优化](https://github.com/Portuguass/Linux_learning/blob/main/Ambian优化.md)

我初次接触，又或者说正式接触使用Linux是源于斐讯N1盒子这么个东西。
在使用过程中，我常常会遇到一些莫名其妙的问题搞崩系统，因此我做了这么一份总结文件，方便自己再重新配置环境时不至于摸不着头脑，同时也减少到处百度/谷歌的时间。

由于Armbian版本的迭代更新，我本身就是一名追新党，文中的一些修改方式/处理方法可能不再适用于最新的系统，需要读者根据自身的实际情况进行更改~





## 01  换源  

由于国内的网络原因，官方源用起来会非常慢。但是在安装的时候他会检测自动帮你换一个比较快的源。其实也没差，但是不排除你有强迫症，就是想用国内源。
```bash
1.备份原有源（万一有啥毛病也能替换回去）
sudo cp -v /etc/apt/sources.list /etc/apt/sources.list.backup
2.清除缓存文件
sudo apt clean
3.删除原有源
sudo rm -rf /etc/apt/sources.list
4.更换源
sudo nano /etc/apt/sources.list
以下为修改内容，因地制宜，自行替换。
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
5.更新、升级
sudo apt update && sudo apt upgrade -y
```





## 02  Samba

对于我而言，服务器不仅仅是用于处理一些“计算任务”，人更加重要的是影音下载、分享，而在家庭局域网中，最方便最常见的分享方式莫过于Samba了。

### 02.1  安装

通过包管理器安装

（apt的使用）

```bash
sudo apt install Samba Samba-common 
```

注：说句题外话，新人一定要养好习惯，不要默认使用root账户去处理日常工作，不然你会倒大霉的！真不是我危言耸听，反正倒霉了你就知道了。

### 02.2  配置——匿名访问

修改smb.conf文件

（文本编辑器nano/vim）

```bash
nano /etc/Samba/smb.conf
```

注：可以删除[printers]及[profiles]

参考模板：

```bash
[global]
   security = user
   map to guest = bad user

[share]
   comment = Storage
   path = /share
   guest ok = yes
   writable = yes
   browseable = yes
   write list = share
   public = yes
```

重启Samba服务

（Linux进程管理）

```bash
sudo service smbd restart
抑或是
sudo /etc/init.d/smbd restart
```





## 03  环境部署及网站搭建

纯属瞎闹着玩，这块我是一窍不通，反正就装上呗~

以后在学？仿佛听见了下次一定。23333

### 03.1  LNMP

在这里我选择的是军哥的lnmp一键包，当然你可以直接选择apt安装速度会快很多，只是易用性差点。（迷信编译出来的性能更优秀，走出一条中国特色社会主义道路~）

军哥LNMP一键包主页：https://lnmp.org/install.html

#### 03.1.1  相关参数

无人值守模式 & LNMP架构&MySQL 5.5 &  启用InnoDB & 数据库Root 用户密码：strelizia & PHP 7.2 & 内存分配器 Jemalloc & Apache 2.4 & 管理员邮箱： & 离线安装

(wget的使用方法)

```bash
wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="2" DB_Root_Password="alt***ea" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp


wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="10" DB_Root_Password="strelizia" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp
```

注：这块可能有点啥问题但我不修改，就这样好了。

#### 03.1.2  pathinfo设置

解决typecho后台或其它404错误

修改/usr/local/nginx/conf/enable-php.conf 文件，添加pathinfo2.conf

```bash
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

```bash
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

```bash
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





## 04 Docker

### 04.1.1  官方安装方式

ubuntu安装docker流程参考[官方文档](https://docs.docker.com/engine/install/ubuntu/)

#### 04.1.1  更新软件包索引并使apt支持https

```bash
 sudo apt update
 sudo apt install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

#### 04.1.2  添加Docker官方的GPG密钥

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

#### 04.1.3  配置源文件

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 04.1.4  安装

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```



### 04.2  一键脚本

安装命令如下：

```bash
curl -fsSL https://get.docker.com | sudo bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```bash
curl -sSL https://get.daocloud.io/docker | sudo sh
```



### 04.3  常用容器

#### 04.3.1  图形化管理工具 Portainer 

```bash
docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer-ce
```

#### 04.3.2  个人博客 Typecho

```bash
docker run -d -p 8000:80 --name typecho --restart=always -v /media/typecho:/data 80x86/typecho:latest
```

#### 04.3.3  私人网盘 Filebrowser

```bash
docker pull 80x86/filebrowser:latest
```

#### 04.3.4 下载工具 qbittorrent

```bash
docker run -d  --name="qbittorrent" --restart=always -v /downloads:/downloads /media/qbittorrent/data:/data /media/qbittorrent/config:/config 80x86/qbittorrent
```



### 04.4  卸载

```bash
或许补充，记得之前看见一篇很不错的教程。是csdn上面的。
```





## 05  Java

### 05.1  安装

#### 0.5.1.1  包管理器安装

```bash
a.
安装OpenJDK 11:
sudo apt update && sudo apt install default-jdk -y
安装完成后，通过检查Java版本进行验证:
java -version

openjdk version "11.0.13" 2021-10-19
OpenJDK Runtime Environment (build 11.0.13+8-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.13+8-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)

b.
安装OpenJDK指定版本，如：Java 8
sudo apt update && sudo apt install openjdk-8-jdk -y
安装完成后，通过检查Java版本进行验证:
java -version

```

#### 05.1.2  编译安装

```bash
1.下载jdk-8u151-linux-x64.tar.gz到download目录
http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html

2.安装jdk
cd download/
mkdir /usr/local/java
tar zxvf jdk-8u151-linux-x64.tar.gz -C /usr/local/java
ln -s /usr/local/java/jdk1.8.0_151/ /usr/local/java/latest

3. 添加环境变量：
vim /etc/profile
加入如下内容：
export JAVA_HOME=/usr/local/java/latest
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

4.执行并检验
source /etc/profile
java -version #检验
```



### 0.5.2  卸载

```bash
暂时没遇上要卸载，有需要再写
```





## 06  Python

一般而言，系统就自带了python3/2，重要自己补全开发环境（附加包）就行。

### 06.1  安装

#### 06.1.1  包管理器安装

```bash
1.安装依赖
sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev
2.安装开发环境
sudo apt update
sudo apt install python3-dev python3-pip
3.升级
pip3 install --upgrade pip setuptools wheel
4.ipython3
ipython3是Python 交互工具，比Python自带环境更智能：
sudo apt install ipython3

```

#### 06.1.2  编译安装











## 09  杂七杂八

### 09.1 .user.ini 

```bash
chattr命令用于改变文件属性。
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



### 09.2  crontab

#### 09.2.1  安装

如果没有安装crontab，请先安装。

```
apt install crontab
```

#### 09.2.2  设置

接着可以使用下面命令开关crontab

```bash
service crond start/stop/restart/reload    #启动/关闭/重启/重载
```

查看crontab服务状态：

```bash
service crond status
```

手动启动crontab服务：

```bash
service crond start
```

查看crontab服务是否已设置为开机启动，执行命令：ntsysv

如果列表中有crond，而且前面有[*]就说明已经添加了开机启动。如果没有可以按照下面的步骤执行。

使用tab键可以切换到确认取消栏。

加入开机自动启动:

```bash
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

```bash
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

```bash
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



#### 09.2.3  统计文件和目录

Linux统计当前目录下有多少文件和目录

仅仅做个笔记：

统计当前目录下文件的个数，不包括子目录：

```bash
ls -l |grep "^-"|wc -l
```

统计当前目录下目录的个数，不包括子目录：

```bash
ls -l |grep "^d"|wc -l
```

统计当前目录下文件的个数，包括子目录：

```bash
ls -lR|grep "^-"|wc -l
```

统计当前目录下目录的个数，包括子目录

```bash
ls -lR|grep "^d"|wc -l
```





## 10  文末小剧场

2022/01/06/21:17

困了，睡觉了，今天系统不知道为啥崩了，从头再来，随便整理一下相关的知识/文件。

当前系统信息以及配置

```
                          ./+o+-       orca@ubuntu
                  yyyyy- -yyyyyy+      OS: Ubuntu 20.04 focal
               ://+//////-yyyyyyo      Kernel: x86_64 Linux 5.4.0-92-generic
           .++ .:/++++++/-.+sss/`      Uptime: 1h 28m
         .:++o:  /++++++++/:--:/-      Packages: 854
        o:+o+:++.`..```.-/oo+++++/     Shell: bash 5.0.17
       .:+o:+o/.          `+sssoo+/    Disk: 17G / 232G (8%)
  .++/+:+oo+o:`             /sssooo.   CPU: Intel Core i5-6500T @ 4x 3.1GHz [55.0°C]
 /+++//+:`oo+o               /::--:.   GPU: Intel Corporation HD Graphics 530 (rev 06)
 \+/+o+++`o++o               ++////.   RAM: 1816MiB / 15897MiB
  .++.o+++oo+:`             /dddhhh.  
       .+.o+oo:.          `oddhhhh+   
        \+.++o+o``-````.:ohdhhhhh+    
         `:o+++ `ohhhhhhhhyo++os:     
           .o:`.syhhhhhhh/.oo++o`     
               /osyyyyyyo++ooo+++/    
                   ````` +oo+++o\:    
                          `oo++.  
                          
Filesystem                         Size  Used Avail Use% Mounted on
udev                               7.8G     0  7.8G   0% /dev
tmpfs                              1.6G  2.9M  1.6G   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  223G   18G  195G   9% /
tmpfs                              7.8G     0  7.8G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p2                     976M  108M  801M  12% /boot
/dev/loop1                          71M   71M     0 100% /snap/lxd/21029
/dev/loop0                          56M   56M     0 100% /snap/core18/2128
/dev/loop2                          33M   33M     0 100% /snap/snapd/12704
tmpfs                              1.6G  4.0K  1.6G   1% /run/user/1000
```

