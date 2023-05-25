## 00  [Ambian优化](https://github.com/Portuguass/Linux_learning/blob/main/Ambian优化.md)

我初次接触，又或者说正式接触使用Linux是源于斐讯N1盒子这么个东西。
在使用过程中，我常常会遇到一些莫名其妙的问题搞崩系统，因此我做了这么一份总结文件，方便自己再重新配置环境时不至于摸不着头脑，同时也减少到处百度/谷歌的时间。

由于Armbian版本的迭代更新，我本身就是一名追新党，文中的一些修改方式/处理方法可能不再适用于最新的系统，需要读者根据自身的实际情况进行更改~


## 01  换源  

由于国内的网络原因，官方源用起来会非常慢。但是在安装的时候他会检测自动帮你换一个比较快的源。其实也没差，但是不排除你有强迫症，就是想用国内源。

**非常不推荐！！！**
- `GNU/Linux` 手动换源

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
**强烈推荐！！！**
- `GNU/Linux` 一键更换国内软件源脚本

```bash
bash <(curl -sSL https://gitee.com/SuperManito/LinuxMirrors/raw/main/ChangeMirrors.sh)
```

> **注意：**
>
> - *Debian 系 Linux 默认禁用了源码仓库和预发布软件源，若需启用可将 list 源文件中相关内容的所在行 `取消注释`。*

例行要安装的软件

```bash
sudo apt install -y mc curl wget git build-essential cmake glances byobu jq htop lsof ccze net-tools dnsutils # for dig

sudo apt install ssh samba # 基础服务，确保系统初装后能够被远程配置
```

## 02  Samba

对于我而言，服务器不仅仅是用于处理一些“计算任务”，人更加重要的是影音下载、分享，而在家庭局域网中，最方便最常见的分享方式莫过于Samba了。

### 02.1  安装

通过包管理器安装

（apt的使用）

```bash
sudo apt install samba samba-common 
```

注：说句题外话，新人一定要养好习惯，不要默认使用root账户去处理日常工作，不然你会倒大霉的！真不是我危言耸听，反正倒霉了你就知道了。

### 02.2  配置——匿名访问

修改smb.conf文件

（文本编辑器nano/vim）

```bash
nano /etc/samba/smb.conf
```

注：可以删除[printers]及[profiles]

参考模板：

```bash
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
亦或是
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
docker run -d \
-p 9000:9000 \
--name portainer \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
portainer/portainer-ce
```

#### 04.3.2  个人博客 Typecho

```bash
docker run -d \
-p 8000:80 \
--name typecho \
--restart=always \
-v /media/typecho:/data \
80x86/typecho:latest
```

#### 04.3.3  私人网盘 Filebrowser

http + 硬件编码：

```bash
docker run -d \
    --name filebrowser \
    --restart unless-stopped \
    --device=/dev/dri/renderD128:/dev/dri/renderD128 \
    --net=host \
    -v /media:/myfiles \
    -v /docker/config:/config \
    --mount type=tmpfs,destination=/tmp \
    80x86/filebrowser:2.9.4-amd64
```

#### 04.3.4  下载工具 qbittorrent

```bash
docker run -d \
    --name qbittorrent \
    --restart unless-stopped \
    --net=host \
    -v /downloads:/downloads \
    -v /docker/qbittorrent/config:/config \
    -v /docker/qbittorrent/data:/data \
    80x86/qbittorrent:4.3.5-alpine-3.13.5-amd64-full
    
sudo nano /docker/qbittorrent/config/qBittorrent.conf

将WebUI\HTTPS\Enabled=true改为WebUI\HTTPS\Enabled=false

sudo docker restart qbittorrent
```

#### 04.3.5  自建 DNS AdGuard Home 

创建 macvlan 网络（根据实际情况替换参数，并删除注释）

```bash
docker network create \
    -d macvlan \ # 使用 macvlan 网络驱动
    --subnet=10.0.0.0/24 \ # 指定网段
    --gateway=10.0.0.1 \ # 指定网关 IP
    -o parent=eth0 \ # 指定网卡
    openwrt # 网络名称，随意，自己记得就行
    
 实例：   
docker network create -d macvlan --subnet=10.0.0.0/24 --gateway=10.0.0.1 -o parent=enp0s31f6 AdGuard
```

启动容器（根据实际情况替换参数，并删除注释）

```bash
docker run -d \
    --name adguardhome \
    --restart unless-stopped \
    --log-opt max-size=1m \
    --network openwrt \ # 使用之前创建的 macvlan 网络
    --ip 10.0.0.53 \ # 设置本容器的 IP
    -v $PWD/adguardhome/work:/opt/adguardhome/work \
    -v $PWD/adguardhome/conf:/opt/adguardhome/conf \
    adguard/adguardhome

 实例： 
docker run -d \
    --name adguardhome \
    --restart unless-stopped \
    --log-opt max-size=1m \
    --network AdGuard \
    --ip 10.0.0.3 \
    -v /docker/adguardhome/work:/opt/adguardhome/work \
    -v /docker/adguardhome/conf:/opt/adguardhome/conf \
    adguard/adguardhome
```

配置，参考这篇[教程](https://p3terx.com/archives/use-adguard-home-to-build-dns-to-prevent-pollution-and-remove-ads-2.html)

#### 04.3.6  自动更新镜像与容器  Watch­tower

所有容器（包括 Watch­tower）都会自动更新，`--cleanup` 选项，这样每次更新都会把旧的镜像清理掉。

```bash
docker run -d \
    --name watchtower \
    --restart unless-stopped \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup
```

注：[参考教程](https://p3terx.com/archives/docker-watchtower.html)

#### 04.3.7  阿里云盘 webdav-aliyundriver

```bash
docker run -d \
--name=webdav-aliyundriver \
--restart=always -p 9080:8080 \
-v /etc/localtime:/etc/localtime \
-v /docker/aliyun-driver/:/etc/aliyun-driver/ \
-e TZ="Asia/Shanghai" \
-e ALIYUNDRIVE_REFRESH_TOKEN="your refreshToken" \
-e ALIYUNDRIVE_AUTH_PASSWORD="admin" \
-e JAVA_OPTS="-Xmx1g" \
zx5253/webdav-aliyundriver

# /etc/aliyun-driver/ 挂载卷自动维护了最新的refreshToken，建议挂载
# ALIYUNDRIVE_AUTH_PASSWORD 是admin账户的密码，建议修改
# JAVA_OPTS 可修改最大内存占用，比如 -e JAVA_OPTS="-Xmx512m" 表示最大内存限制为512m
```

#### 04.3.8  Awesome TTRSS

```bash
docker run -itd \
--name ttrss \
--restart unless-stopped \
-p 9090:80 \
wangqiru/ttrss \
-e SELF_URL_PATH = http://10.0.0.40:9090/ \
-e DB_HOST = 127.0.0.1 \
-e DB_PORT = 5432 \
-e DB_NAME = TTRSS \
-e DB_USER = orca \
-e DB_PASS = althaea

变量加后面就好了
还是运行不了，不弄了
```

#### 04.3.9  数据库 postgresql  

```bash
docker run --name postgres -e POSTGRES_PASSWORD=althaea -e POSTGRES_USER=orca -d postgres


POSTGRES_PASSWORD
使用此环境变量是使用 PostgreSQL 映像所必需的。它不能为空或未定义。此环境变量设置 PostgreSQL 的超级用户密码。默认超级用户由环境变量定义。POSTGRES_USER

注1：PostgreSQL 映像在本地设置身份验证，因此您可能会注意到从（在同一容器内）进行连接时不需要密码。但是，如果从其他主机/容器进行连接，则需要输入密码。trustlocalhost

注2：此变量定义 PostgreSQL 实例中的超级用户密码，由脚本在初始容器启动期间设置。它对客户端在运行时可能使用的环境变量没有影响，如https://www.postgresql.org/docs/current/libpq-envars.html中所述。，如果使用，将被指定为单独的环境变量。initdbPGPASSWORDpsqlPGPASSWORD

POSTGRES_USER
此可选环境变量与 一起使用，用于设置用户及其密码。此变量将创建具有超级用户能力的指定用户和具有相同名称的数据库。如果未指定，则将使用 的默认用户。POSTGRES_PASSWORDpostgres

请注意，如果指定了此参数，PostgreSQL 在初始化期间仍将显示。这是指运行守护程序的 Linux 系统用户（来自映像中），因此与该选项无关。有关更多详细信息，请参阅标题为"任意注释"的部分。The files belonging to this database system will be owned by user "postgres"/etc/passwdpostgresPOSTGRES_USER--user
```

#### 04.3.9  数据库管理 pgadmin4 

```bash
通过端口 80 运行一个简单的容器：
docker pull dpage/pgadmin4
docker run -p 80:80 \
    -e 'PGADMIN_DEFAULT_EMAIL=user@domain.com' \
    -e 'PGADMIN_DEFAULT_PASSWORD=SuperSecret' \
    -d dpage/pgadmin4
    
PGADMIN_DEFAULT_EMAIL
这是设置初始管理员帐户以登录到 pgAdmin 时使用的电子邮件地址。此变量是必需的，必须在启动时设置。
PGADMIN_DEFAULT_PASSWORD
这是设置初始管理员帐户以登录到 pgAdmin 时使用的密码。此变量是必需的，必须在启动时设置。
```

[容器部署 — pgAdmin 4 6.3 文档](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html)

#### 04.3.10  影音播放 jellyfin  

```bash
docker run -d \
    --name jellyfin \
    --restart unless-stopped \
    --net=host \
    -v /docker/jellyfin/data:/data \
    -v /docker/jellyfin/config:/config \
    -v /downloads:/media \
    nyanmisaka/jellyfin
```

#### 04.3.11  青龙面板 qinglong 

```bash
docker run -dit \
  -v /docker/ql/config:/ql/config \
  -v /docker/ql/log:/ql/log \
  -v /docker/ql/db:/ql/db \
  -v /docker/ql/repo:/ql/repo \
  -v /docker/ql/raw:/ql/raw \
  -v /docker/ql/scripts:/ql/scripts \
  -p 5700:5700 \
  --name qinglong \
  --hostname qinglong \
  --restart unless-stopped \
  whyour/qinglong:latest
```

#### 04.3.12  智能家居 homeassistant

```bash
docker run -dit \
  -v /docker/homeassistant:/ql/config \
  -v /etc/localtime:/etc/localtime:ro \
  -e TZ=Asia/Shanghai \
  --name homeassistant \
  --net=host \
  --restart unless-stopped \
  homeassistant/home-assistant:latest
```

#### 04.3.12  MQTT 消息服务器 emqx

```bash
docker run -d \
-v /docker/emqx/data:/opt/emqx/data \
-v /docker/emqx/etc:/opt/emqx/etc \
-v /docker/emqx/log:/opt/emqx/log \
-v /docker/emqx/lib:/opt/emqx/lib \
--name emqx \
--net=host \
emqx/emqx:v4.0.0

docker run -d --name emqx --net=host emqx/emqx:v4.0.0

docker run -d --name emqx -p 1883:1883 -p 8081:8081 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 18083:18083 emqx/emqx:v4.0.0
```



### 04.4  卸载

```bash
后续补充，记得之前看见一篇很不错的教程。是csdn上面的。
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
tar -zxf jdk-8u151-linux-x64.tar.gz -C /usr/local/java
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

0.安装依赖

```bash
apt install libgdal-dev build-essential libatlas-base-dev libgdal-dev libdb-dev libdb5.3-dev libxslt1-dev  libncurses5-dev -y

apt install libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev zlibc zlib1g-dev libhdf5-dev libxml2-dev -y

apt install libdb-dev libdb5.3-dev libreadline-dev libpcap-dev libpcap0.8 libpcap0.8-dev libhdf5-dev libxml2-dev -y

apt dist-upgrade

apt install build-essential  libncursesw5-dev libgdbm-dev libc6-dev zlib1g-dev libsqlite3-dev tk-dev libssl-dev openssl gfortran  liblapack-dev libhdf5-dev libzbar-dev libbz2-dev -y
```

1.下载源码

```bash
wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz
```

2.命令行切换到上面压缩文件所在的目录（比如桌面），然后输入 `tar -xzf Python-3.7.7.tgz`

> 这里 tar表示解压缩，-x 表示从档案文件中释放文件，z 表示用 gzip 解压（用于 xx.tgz 以及 xx.tar.gz 格式的压缩包），f 后面是压缩文件名。

3.命令行目录切换到解压后的文件夹中，也就是 Python-3.7.7 文件夹。然后执行 

```bash
mkdir -p /usr/local/python3

./configure --prefix=/usr/local/python3 --with-ssl --enable-optimizations 

#执行这步是后面最好加上 --enable-optimizations 会自动安装pip3及优化配置
#在./configure过程中，如果没有加上–with-ssl参数时，默认安装的软件涉及到ssl的功能不可用，
#这个命令的作用是生成 Makefile 文件，以供下一步的 make 命令使用。
#Makefile 文件存储的时构建 (build) 顺序，linux build 程序组件时需要按照 Makefile 指定的顺序。
```

4.

```bash
make -j2 
#编译源代码，并生成执行文件
make install
#是把生成的执行文件拷贝到 linux 系统中必要的目录下，比如拷贝到 usr/local/bin 目录下，这样所有的用户都可以运行这个程序了。
```

5.删除原有的软连接

```bash
python3 -V

pip3 -V

rm -rf /usr/bin/python3

rm -rf /usr/bin/pip3
```

6.建立新的指向python3.7的软链接

```bash
#添加python3的软链接 

ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3 

#添加 pip3 的软链接 

ln -s /usr/local/python3/bin/pip3.7 /usr/bin/pip3
```

7.检测版本

```bash
root@arm-64:/usr/local/python3/bin# python3 -V
Python 3.7.7
root@arm-64:/usr/local/python3/bin# pip3 -V
pip 19.2.3 from /usr/local/python3/lib/python3.7/site-packages/pip (python 3.7)
```

8.升级pip/setuptool

```bash
pip3 install --upgrade pip setuptools
```



安装pip后，无法使用pip安装一些包，总是会出现上述错误

```bash
subprocess.CalledProcessError: Command '('lsb_release', '-a')' returned non-zero exit status 1.


```

```bash
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

### 06.2  第三方库

**第三方库**

```bash
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





## 07  Node.js

### 07.1  安装Node.js v16.x

```bash
# Using Ubuntu
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
apt-get install -y nodejs
```

### 07.2  安装Yarn（包管理器）

```bash
## To install the Yarn package manager, run:
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
```

### 07.3  node-red

```bash
1.安装
sudo npm install -g node-red
2.安装pm2
sudo npm install pm2 -gd
3.使用PM2启动Node-red：
sudo pm2 start node-red
4.查看启动项列表：
sudo pm2 ls && sudo pm2 save
5.设置pm2为自启进程：
sudo pm2 startup
```





## 08  golang

### 08.1  安装

#### 08.1.1  下载 Go 压缩包

以 root 或者其他 sudo 用户身份运行下面的命令，下载并且解压 Go 二进制文件到`/usr/local`目录：

```bash
wget -c https://go.dev/dl/go1.17.6.linux-amd64.tar.gz -O - | sudo tar -xz -C /usr/local
```

#### 08.1.2  添加到`$PATH`环境变量

通过将 Go 目录添加到`$PATH`环境变量，系统将会知道在哪里可以找到 Go 可执行文件。

这个可以通过添加下面的行到`/etc/profile`文件（系统范围内安装）或者`$HOME/.profile`文件（当前用户安装）：

```bash
export PATH=$PATH:/usr/local/go/bin
```

保存文件，并且重新加载新的PATH 环境变量到当前的 shell 会话：

```bash
sudo source /etc/profile
```

#### 08.1.4  验证

通过打印 Go 版本号，验证安装过程。

```bash
go version
```

输出应该像下面这样：

```bash
go version go1.17.2 linux/amd64
```







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



### 09.3  统计文件和目录

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



### 09.4  中文 man 手册页

#### 09.4.1  通过包管理器安装

```bash
1.安装
sudo apt update
sudo apt install manpages-zh
2.查看manpages-zh库的安装路径，例如:/usr/share/man/zh_CN/
dpkg -L manpages-zh
3.简单测试，介绍为中文则无误
man -M /usr/share/man/zh_CN open
4.使用alias命令给中文man取名为cman,方便后期使用。
sudo su
echo "alias cman='man -M /usr/share/man/zh_CN'" >> /etc/profile.d/cman.sh
注意/usr/share/man/zh_CN路径的正确性。
5.刷新缓存
source /etc/profile.d/cman.sh
6.验证
cman open
如果此命令的效果和man -M /usr/share/man/zh_CN open命令的效果一致，则设置成功。
```

#### 09.4.1 使用

```bash
cman <命令>
```



### 09.5  Aria2-pro

```bash
wget https://github.com/P3TERX/aria2.sh/archive/refs/heads/master.zip
sudo unzip master.zip && cd aria2.sh-master
sudo chmod 777 *sh && sudo ./*sh

然后就等安装好再修改配置就行了~

前端面板
# host 网络模式（如果你需要使用 IPv6 网络访问，这是最简单的方式）
docker run -d \
    --name ariang \
    --log-opt max-size=1m \
    --restart unless-stopped \
    --network host \
    p3terx/ariang --port 6880 --ipv6
```



### 09.6  Docker 基本命令

```bash
停止容器：
docker stop <CONTAINER>
删除容器：
docker rm <CONTAINER>
更新镜像：
docker pull <IMAGE>
启动容器：
docker run <ARG> ... <IMAGE>
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

