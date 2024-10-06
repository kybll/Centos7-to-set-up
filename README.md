## 准备

本教程中多次使用vi编辑器，现作基本使用解释

```
编辑：i
跳转到段尾并编辑：A
退出编辑：Esc
保存并退出：    :wq
保存：      :w
退出：      :q
强制退出：     :q!
强制保存并退出：      :wq!
```

## 关闭SELinux

```
vi /etc/selinux/config
vi /etc/sysconfig/selinux
```

将这两个文件中的`SELINUX=enforcing`修改为`SELINUX=disabled`
## 换源

### 一键脚本

```
bash <(curl -sSL https://linuxmirrors.cn/main.sh)
```

### 手动换源

**阿里云**

备份软件源

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/

```
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

生成yum缓存

```
yum makecache
```

**华为云**

备份软件源

```
cp -a /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/

```
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.huaweicloud.com/artifactory/os-conf/centos/centos-7.repo
```

生成yum缓存

```
yum makecache
```

## 更新内核

手动下载至服务器

```
yum install wget perl -y && wget https://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-ml-6.9.7-1.el7.elrepo.x86_64.rpm && wget https://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-ml-devel-6.9.7-1.el7.elrepo.x86_64.rpm
```

使用rpm安装内核

```
rpm -ivh kernel-ml-6.9.7-1.el7.elrepo.x86_64.rpm && rpm -ivh kernel-ml-devel-6.9.7-1.el7.elrepo.x86_64.rpm 
```

确认是否安装成功

```
rpm -qa | grep kernel
```

如果输出内容中包含`kernel-ml-6.9.7-1.el7.elrepo.x86_64` 和 `kernel-ml-devel-6.9.7-1.el7.elrepo.x86_64` 则已经安装成功

```
[root@centos7 ~]# rpm -qa | grep kernel
kernel-ml-6.9.7-1.el7.elrepo.x86_64
kernel-3.10.0-1160.71.1.el7.x86_64
kernel-tools-3.10.0-1160.71.1.el7.x86_64
kernel-ml-devel-6.9.7-1.el7.elrepo.x86_64
kernel-tools-libs-3.10.0-1160.71.1.el7.x86_64
```

查看现有内核启动顺序

```
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
```

修改默认启动项

> [!NOTE]
> xxx 为序号数字，以指定启动列表中第x项为启动项，x从0开始计数
> 

```
# 更改启动项命令
grub2-set-default xxx

# 使用6.9.7安装
grub2-set-default 0
```

重启系统

```
reboot
```

删除旧内核

```
yum remove $(rpm -qa | grep kernel | grep -v $(uname -r))
```

## 挂载新硬盘

### 一键脚本

说明：
1：默认将数据盘挂载到`/www`目录  
2：如有`NTFS/FAT32`分区可选格式化自动挂载  
3：若您的硬盘已分区，且未挂载，工具会自动将分区挂载到`/www`  
4：若您的硬盘是新硬盘，工具会自动分区并格式化成`xfs/ext4`文件系统  
5：只自动挂载一个分区，若您有多块数据盘，请手动挂载未被自动挂载的硬盘  
6：此脚本只适用于新硬盘挂载，若数据盘已有数据请勿使用此脚本

```
bash <(wget --no-check-certificate -qO- https://download.bt.cn/tools/auto_disk.sh)
```

### 手动挂载

查看新添加的硬盘

```
fdisk -l
```

建立分区

```
fdisk /dev/sdb
# 输入
n ->创建分区
p ->选择主分区
回车
回车
w -> 保存
```

分区创建操作系统

```
mkfs.xfs /dev/sdb1
```

挂载到`/data`目录

```
mount /dev/sdb1 /data
```

设置开机自启动

```
vi /etc/fstab
# 添加
/dev/sdb1    /data           xfs      defaults   0   0
```

重启系统`reboot`，验证挂载是否生效

```
# 重启后查看分区
df -h
```
## 安装常用软件包

```
yum install tree nmap dos2unix lrzsz nc lsof tcpdump iotop sysstat psmisc net-tools vim tar git gcc unzip epel-release -y
```

换源epel

```
wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

## 配置时间同步

安装ntp

```
yum install -y ntpdate
```

同步时间

```
ntpdate cn.pool.ntp.org
```

## 安装Docker

### 一键脚本

```
bash <(curl -sSL https://linuxmirrors.cn/docker.sh)
```

### 手动安装

卸载旧版Docker

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

```

换源华为云

```
yum install -y yum-utils && wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.huaweicloud.com/docker-ce/linux/centos/docker-ce.repo && sed -i 's+download.docker.com+mirrors.huaweicloud.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo && yum makecache
```

安装Docker

```
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

启动docker/设置开机自启动

```
systemctl start docker && systemctl enable docker
```

Docker换源

```
# 编辑docker配置文件
vi /etc/docker/daemon.json

# 在文档中加入以下内容
{
    "registry-mirrors": [
        "https://dockerproxy.1panel.live",
        "https://docker.1panel.live",
        "https://proxy.1panel.live"
    ]
}
```

## 安装宝塔面板/aapanel

### 宝塔面板

稳定版9.0.0

```
url=https://download.bt.cn/install/install_lts.sh;if [ -f /usr/bin/curl ];then curl -sSO $url;else wget -O install_lts.sh $url;fi;bash install_lts.sh ed8484bec
```

正式版9.2.0

```
if [ -f /usr/bin/curl ];then curl -sSO https://download.bt.cn/install/install_panel.sh;else wget -O install_panel.sh https://download.bt.cn/install/install_panel.sh;fi;bash install_panel.sh ed8484bec
```

旧版7.7.0

```
curl -sSO https://raw.githubusercontent.com/8838/btpanel-v7.7.0/main/install/install_panel.sh && bash install_panel.sh
```

### aaPanel

```
URL=https://www.aapanel.com/script/install_7.0_en.sh && if [ -f /usr/bin/curl ];then curl -ksSO "$URL" ;else wget --no-check-certificate -O install_7.0_en.sh "$URL";fi;bash install_7.0_en.sh aapanel
```

## 安装1Panel

```
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sh quick_start.sh
```

## 改中文显示

安装语言包

```
yum install kde-l10n-Chinese -y
```

确认安装

```
locale -a|grep zh_CN
```

临时修改语言显示

```
LANG="zh_CN.UTF-8"
```

永久修改语言显示

```
vi /etc/locale.conf
```

```
LANG="zh_CN.UTF-8"
```

重启系统

```
reboot
```

## 基础安全配置

更改root密码为强密码

```
passwd
```

新建用户

```
adduser xxxx
```

设置用户密码

```
passwd xxxx
```

将`sudoers`文件的权限修改成可编辑

```
chmod -v u+w /etc/sudoers
```

使用`vi`编辑`sudoers`文件

```
vi /etc/sudoers
```

在`sudoes`文件中添加如下的内容

找到 `root ALL=(ALL) ALL`
然后在下方添加 `xxxx ALL=(ALL) ALL`

将`sudoers`文件的权限修改成不可编辑

```
chmod -v u-w /etc/sudoers
```

添加用户目录

```
mkdir /home/xxxx
```

切换用户

```
su xxxx
```

以后SSH登录时就直接登录新建的用户即可，然后再进行`sudo -i`

重新进入root用户

```
sudo -i
```

编辑Openssh-server的配置文件

```
vi /etc/ssh/sshd_config
```

根据如下命令修改

```
修改ssh端口
#Port 22
去除上方#符号，修改22为其它没有被占用的端口，例如22229或33322等

关闭ssh的root登录
#PermitRootLogin yes
将这条命令（类似）修改为：
PermitRootLogin no

开启上次ssh登录显示
#PrintLastLog no
将这条命令（类似）修改为：
PrintLastLog yes
```

安装防火墙

```
yum install firewalld -y
```

启动/开机自启防火墙

```
systemctl start firewalld && systemctl enable firewalld
```

开放ssh端口（22修改为上面更改后的ssh端口）

```
firewall-cmd --zone=public --add-port=22/tcp --permanent
```

重新载入防火墙配置

```
firewall-cmd --reload
```

防火墙基本命令

```
显示状态：firewall-cmd --state
查看所有打开的端口：firewall-cmd --zone=public --list-ports
更新防火墙规则：firewall-cmd --reload
添加端口：firewall-cmd --zone=public --add-port=80/tcp --permanent
删除端口：firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

