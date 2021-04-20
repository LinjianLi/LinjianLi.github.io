# Linux 系统安装后配置环境

不是全部执行，只是放在这里方便复制粘贴

## 各种基础应用

```shell
apt update -y
apt upgrade -y
apt install sudo
apt install -y openssh-server
apt install -y make cmake g++
apt install -y htop sysstat # for monitoring the system
apt install -y vim
apt install -y zsh zsh-syntax-highlighting
apt install -y git curl wget
apt install -y tmux
apt install screenfetch
apt install gnome-tweak-tool
```

## Timezone

查看系统当前时区

```shell
timedatectl
```

查看所有时区并设置时区

```shell
timedatectl list-timezones
timedatectl set-timezone <your_time_zone>
```

## 设置 Shell 为 zsh

```shell
chsh -s $(which zsh)
```

### 设置 zsh 的语法高亮

我不想用 oh-my-zsh ，因为遇到过一些奇怪的问题。

```shell
apt install zsh-syntax-highlighting
echo "source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ~/.zshrc
```

重新刷新之后没有语法高亮，那就执行以下语句找到 `zsh-syntax-highlighting.zsh` 文件的位置

```shell
whereis zsh-syntax-highlighting.zsh
```

然后在 `.zshrc` 文件里面加上（把 `the_path_to/zsh-syntax-highlighting.zsh` 替换成之前找到的文件路径）

```shell
source the_path_to/zsh-syntax-highlighting.zsh
```

### 设置 zsh 的指令历史记录

打开 `~/.zshrc` 文件然后添加以下内容

```shell
HISTFILE=~/.zsh_history
HISTSIZE=100000
SAVEHIST=100000
setopt appendhistory
setopt HIST_SAVE_NO_DUPS  # Don't write duplicate entries in the history file.
setopt HIST_IGNORE_DUPS # Do not enter command lines into the history list if they are duplicates of the previous event.

export PYTHONIOENCODING=utf-8
alias wn1="watch -n 1"
alias wn1ns="watch -n 1 nvidia-smi"
```

然后要创建一个 `~/.zsh_history` 来存放指令历史记录

```shell
touch ~/.zsh_history
```

## Anaconda

### 基于 Python 3.6 的 Anaconda3

这里的命令是安装基于 Python 3.6 的。要安装最新版的话，去 Anaconda 挂网找最新版的连接，替换掉就好了。

```shell
cd /tmp
# Download the Anaconda Bash Script
curl -O https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh
# Verify the Data Integrity of the Installer
sha256sum Anaconda3-5.2.0-Linux-x86_64.sh
# Run the Anaconda Script
bash Anaconda3-5.2.0-Linux-x86_64.sh
# 后续安装过程会询问几个问题
# agreement要yes
# VS code自己决定是否yes
# 增加到环境变量要yes
```

在国内，官方源的速度比较慢，可以添加源

```shell
# 添加清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
# conda config --set show_channel_urls yes
```

## 安装Sublime Text 3

```shell
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | apt-key add -
apt install apt-transport-https
echo "deb https://download.sublimetext.com/ apt/stable/" | tee /etc/apt/sources.list.d/sublime-text.list
apt update
apt install sublime-text
```

## pip

更换 pip 源，在用户个人目录下 (root 用户就在 root 目录下) 修改 `~/.pip/pip.conf` 文件 (如果没有就自己新建)，如下

```shell
[global]
index-url = https://mirrors.aliyun.com/pypi/simple
extra-index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

保存并退出

## liblinear

在(Unix环境)Python中使用liblinear需要先下载liblinear源码，然后解压，进入liblinear主目录，执行`make`，然后进入`python`子目录，执行`make`。然后又2种方法使得liblinear在Python环境变量中

1. 拷贝`liblinear/python`的文件夹目录，将目录添加至Python解释器的path中
2. 在命令行界面执行`export PYTHONPATH=<path_to_liblinear>/liblinear/python:$PAYTHONPATH`
