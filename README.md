# Linux使用记录

2022/01/06
开始记录我在使用/学习Linux（Ubuntu/Debian）过程中的心得体会以及遇到的一些问题。
在我再回过头看时，也许会感到收获很多吧！

## 01.[Ambian优化](https://github.com/Portuguass/Linux_learning/blob/main/Ambian优化.md)

我初次接触，又或者说正式接触使用Linux是源于斐讯N1盒子这么个东西。
在使用过程中，我常常会遇到一些莫名其妙的问题搞崩系统，因此我做了这么一份总结文件，方便自己再重新配置环境时不至于摸不着头脑，同时也减少到处百度/谷歌的时间。

由于Armbian版本的迭代更新，我本身就是一名追新党，文中的一些修改方式/处理方法可能不再适用于最新的系统，需要读者根据自身的实际情况进行更改~

## 02.Samba匿名访问

对于我而言，服务器不仅仅是用于处理一些“计算任务”，人更加重要的是影音下载、分享，而在家庭局域网中，最方便最常见的分享方式莫过于Samba了。

### 02.1.Samba的安装

通过包管理器安装

```
sudo apt install Samba Samba-common 
```

注：说句题外话，新人一定要养好习惯，不要默认使用root账户去处理日常工作，不然你会倒大霉的！真不是我危言耸听，反正倒霉了你就知道了。

### 02.2.Samba的配置（匿名访问）

修改smb.conf文件

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

```
sudo service smbd restart
抑或是
sudo /etc/init.d/smbd restart
```

## 03.环境部署及网站搭建

纯属瞎闹着玩，这块我是一窍不通，反正就装上呗~

以后在学？仿佛听见了下次一定。23333

### 03.1LNMP

在这里我选择的是军哥的lnmp一键包，当然你可以直接选择apt安装速度会快很多，只是易用性差点。（迷信编译出来的性能更优秀，走出一条中国特色社会主义道路~）

军哥LNMP一键包主页：https://lnmp.org/install.html

#### 03.1.1相关参数

无人值守模式 & LNMP架构&MySQL 5.5 &  启用InnoDB & 数据库Root 用户密码：strelizia & PHP 7.2 & 内存分配器 Jemalloc & Apache 2.4 & 管理员邮箱：**** & 离线安装

```
wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="2" DB_Root_Password="alt***ea" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp


wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && LNMP_Auto="y" DBSelect="10" DB_Root_Password="strelizia" InstallInnodb="y" PHPSelect="8" SelectMalloc="2" CheckMirror="n" ./install.sh lnmp
```

注：这块可能有点啥问题但我不修改，就这样好了。

#### 03.1.2pathinfo设置

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

