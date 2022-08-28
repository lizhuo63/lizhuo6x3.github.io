# 用户授权

1. 切换到root用户下 ：`su`
2. 编辑/etc/sudoers文件，需先添加sudoers文件的写权限`chmod u+w /etc/sudoers`
3. `vi /etc/sudoers` 找行 **root ALL = (ALL) ALL**，换行添加 `用户名 ALL=(ALL) ALL 
   `
4. 移除写权限，`chmod u-w /etc/sudoers`
5. 切换用户 `su yakong`

# 配置软件源

```
1.安装wget工具；yum -y install wget
2.备份原配置文件；
  cd /etc/yum.repos.d/ 
  yum.repos.d]# cp CentOS-Base.repo CentOS-Base.repo.back
3.加载阿里yum源
  yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
4.清理原缓存，建立新的缓存
  yum clean all
  yum makecache
5.搜索测试yum加载成功
  yum search tomcat
```

# 网络配置

```
1.安装网络管理工具
sudo yum install net-tools
2.配置文件
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens33

UUID="ba6bd190-c634-46d6-b96d-b945a0e53162"
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
# 设置
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
DEVICE="ens33"
ONBOOT="yes"
NETNASK=255.255.255.0
# 设置
IPADDR="192.168.154.143"
PREFIX="24"
# 设置
GATEWAY="192.168.154.2
DNS1="223.5.5.5"
DNS2="119.29.29.29"

3.重启网卡
sudo systemctl restart network
```

# 防火墙

```
启动： systemctl start firewalld
查看状态： systemctl status firewalld 
禁用，禁止开机启动： systemctl disable firewalld
停止运行： systemctl stop firewalld

开放端口
sudo vim /usr/lib/firewalld/services/ssh.xml
  <port protocol="tcp" port="3306"/>

```

# SSH服务

```
systemctl status sshd.service   #查看ssh状态
systemctl start sshd.service    #启动ssh服务
systemctl restart sshd.service  #重启ssh服务 
systemctl enable sshd.service   #开机ssh自启
systemctl stop sshd.service　　　#关闭ssh服务
```

