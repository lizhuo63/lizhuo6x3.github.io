# 关于WSL

**wsl：** (Windows Subsystem for Linux)。 可直接在Windows上运行GNU/Linux环境，无需修改，无需传统虚拟机或双引导设置的开销。😘😘😘WSL，Microsoft👍👍👍

#  安装 WSL [官网教程](https://docs.microsoft.com/en-us/windows/wsl/install)

1. 前提条件

    Windows10_2004及更高版本（或内部版19041及更高版本）或 Windows 11。

2. 正式安装(管理员身份打开 PowerShell)

    ```bash
    #1 启用“适用于 Linux 的 Windows 子系统”可选功能。
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    #2 启用虚拟机平台可选功能
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    #3 重启电脑
    #4 安装Linux内核更新包https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
    #5 将WSL2设置为默认版本 
    wsl --set-default-version 2
    #6 微软商店选择并安装Linux(以下安装的是ubuntu) 创建用户和密码。
    ```

# 优化Ubuntu-18.04

## 换源

```bash
#1、备份 /etc/apt/sources.list
cp /etc/apt/sources.list /etc/apt/sources.list.bak
#2.重写
#Ubuntu国内阿里源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
#3.设置
sudo apt-get update
sudo apt-get upgrade
```

## 安装Zsh

```bash
#查看本地的 shell
cat /etc/shells
#切换到 zsh
chsh -s /bin/zsh

#1 安装zsh
sudo apt install zsh
#2 安装oh-my-zsh
前往Gitee任意oh-my-zsh镜像库下载tools/install.sh文件 然后 `sh install.sh`安装
#3 安装插件
#* 
sudo apt install autojump
#* 
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
#* 
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/plugins/zsh-autosuggestions
#* 
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/.oh-my-zsh/themes/powerlevel10k

#4 配置
vim ~/.zshrc
ZSH_THEME="powerlevel10k/powerlevel10k"
plugins=(
	git
	zsh-syntax-highlighting
	zsh-autosuggestions
	autojump
)
#5 应用配置
source  ~/.zshrc
```

## 安装docker

```bash
#安装依赖 
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
#添加GPG公钥 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
#添加软件仓库
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
#安装 
sudo apt-get update
sudo apt-get install -y docker-ce

# 免sudo使用
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo service docker restart
```

