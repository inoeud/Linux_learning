# Linux使用记录

2022/01/06
开始记录我在使用/学习Linux（Ubuntu/Debian）过程中的心得体会以及遇到的一些问题。
在我再回过头看时，也许会感到收获很多吧！

## 01  [Ambian优化](https://github.com/Portuguass/Linux_learning/blob/main/Ambian优化.md)

我初次接触，又或者说正式接触使用Linux是源于斐讯N1盒子这么个东西。
在使用过程中，我常常会遇到一些莫名其妙的问题搞崩系统，因此我做了这么一份总结文件，方便自己再重新配置环境时不至于摸不着头脑，同时也减少到处百度/谷歌的时间。

由于Armbian版本的迭代更新，我本身就是一名追新党，文中的一些修改方式/处理方法可能不再适用于最新的系统，需要读者根据自身的实际情况进行更改~





## 02  Samba

对于我而言，服务器不仅仅是用于处理一些“计算任务”，人更加重要的是影音下载、分享，而在家庭局域网中，最方便最常见的分享方式莫过于Samba了。

### 02.1  安装

通过包管理器安装

（apt的使用）

```
sudo apt install Samba Samba-common 
```

注：说句题外话，新人一定要养好习惯，不要默认使用root账户去处理日常工作，不然你会倒大霉的！真不是我危言耸听，反正倒霉了你就知道了。

### 02.2  配置——匿名访问

修改smb.conf文件

（文本编辑器nano/vim）

```
nano /etc/Samba/smb.conf
```

注：可以删除[printers]及[profiles]

参考模板：

```
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

```
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

```
wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="2" DB_Root_Password="alt***ea" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp


wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="10" DB_Root_Password="strelizia" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp
```

注：这块可能有点啥问题但我不修改，就这样好了。

#### 03.1.2  pathinfo设置

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





## 04 Docker

### 04.1.1  官方安装方式

ubuntu安装docker流程参考[官方文档](https://docs.docker.com/engine/install/ubuntu/)

#### 04.1.1  更新软件包索引并使apt支持https

```
 sudo apt update
 sudo apt install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

#### 04.1.2  添加Docker官方的GPG密钥

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

#### 04.1.3  配置源文件

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 04.1.4  安装

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```



### 04.2  一键脚本

安装命令如下：

```
curl -fsSL https://get.docker.com | sudo bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```
curl -sSL https://get.daocloud.io/docker | sudo sh
```



### 04.3  Docker 卸载

```
后续补充，记得之前看见一篇很不错的教程。是csdn上面的。
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

