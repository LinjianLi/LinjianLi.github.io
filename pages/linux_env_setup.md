# Linux 系统安装后配置环境

不是全部执行，只是放在这里方便复制粘贴

## Ubuntu 18.04

```shell
sudo apt update -y
sudo apt upgrade -y
sudo apt install -y make cmake g++
sudo apt install -y htop zsh git vim
sudo apt install python3-dev python3-pip
sudo pip3 install --upgrade virtualenv
sudo apt install screenfetch
sudo apt install gnome-tweak-tool
```

## 安装基于Python 3.6的Anaconda3

```shell
cd /tmp
curl -O https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh
sha256sum Anaconda3-5.2.0-Linux-x86_64.sh
bash Anaconda3-5.2.0-Linux-x86_64.sh
# 后续安装过程会询问几个问题
# agreement要yes
# VS code自己决定是否yes
# 增加到环境变量要yes
```

在国内，官方源的速度比较慢，可以添加源

```shell
# 添加清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --set show_channel_urls yes
```

## 安装Sublime Text 3

```shell
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
sudo apt install apt-transport-https
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt update
sudo apt install sublime-text
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

## CUDA

查看 CUDA 版本

```shell
nvcc --version
```

或

```shell
/usr/local/cuda/bin/nvcc --version
```

或

```shell
cat /usr/local/cuda/version.txt
```
