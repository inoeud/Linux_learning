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
sudo apt install samba samba-common 
```

注：说句题外话，新人一定要养好习惯，不要默认使用root账户去处理日常工作，不然你会倒大霉的！真不是我危言耸听，反正倒霉了你就知道了。

### 02.2.Samba的配置（匿名访问）

修改smb.conf文件

```
nano /etc/samba/smb.conf
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

