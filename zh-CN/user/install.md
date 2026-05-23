# 安装指南

## 鸿蒙 PC

**1\. 卸载冲突软件**

如果 PC 中安装有 GitNext 和 DevBox 这两个应用，请将它们卸载。

**2\. 打开安全开关**

进入“设置”-> “系统”->“开发者选项”，打开“开发者选项”开关。

进入“设置”->“隐私和安全”->“高级”，打开“运行来自非应用市场的扩展程序”开关。

**3\. 安装 Homebrew**

在终端中执行这句命令进行安装

```sh
zsh -c "$(curl -fsSL https://harmonybrew.atomgit.com/install.sh)"
```

**4\. 配置环境变量**

按照安装脚本的提示，执行以下命令，将 Homebrew 加入到 PATH 中

```sh
echo >> ~/.zshrc
echo 'eval "$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)"' >> ~/.zshrc
eval "$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)"
```

现在可以开始使用 brew 命令了。

## 鸿蒙开发板

**1\. 配置所需目录**

Homebrew 运行过程中需要使用 `$HOME`、`/usr/bin`、`/storage/Users/currentUser` 等目录，我们需要手动配置这些目录

```sh
mount -o remount,rw /
mkdir -p /usr
mkdir -p /data/storage/Users/currentUser
ln -s /bin /usr/bin

# /storage 目录在 tmpfs 上，数据无法持久化，每次重启设备后需要重新创建软链接
ln -s /data/storage/Users /storage/Users

# hdc shell 环境下 HOME 变量默认指向 / 目录，这个目录可用空间很少，需要将其指向一个大容量的目录
# 此配置仅在当前终端有效，每次进入 hdc shell 需要重新设置
export HOME=/storage/Users/currentUser
```

**2\. 安装 curl 和 zsh**

在上位机（Windows 电脑）下载好 [鸿蒙版 curl](https://github.com/Harmonybrew/ohos-curl) 和 [鸿蒙版 zsh](https://github.com/Harmonybrew/ohos-zsh) ，并用 hdc 将 tar 包推到设备上。

在设备上解压，把里面的命令软链接到 `/usr/bin` 目录下

```sh
mount -o remount,rw /
tar -zxf curl-8.19.0-ohos-arm64.tar.gz -C /data
ln -s /data/curl-8.19.0-ohos-arm64/bin/curl /usr/bin/curl
tar -zxf zsh-5.9-ohos-arm64.tar.gz -C /data
ln -s /data/zsh-5.9-ohos-arm64/bin/zsh /usr/bin/zsh
```

**3\. 安装 Homebrew**

在终端中执行这句命令进行安装 _（别忘了给开发板联网）_

```sh
zsh -c "$(curl -fsSL https://harmonybrew.atomgit.com/install.sh)"
```

**4\. 配置环境变量**

手动将 Homebrew 加入到 PATH 中

```sh
# 此配置仅在当前终端有效，每次进入 hdc shell 需要重新设置
export PATH=/storage/Users/currentUser/.harmonybrew/bin:$PATH
```

现在可以开始使用 brew 命令了。

## 鸿蒙容器

**1\. 安装 zsh**

在容器内通过 curl 下载 zsh，将其软链接到 /usr/bin 目录下

```sh
curl -fLO https://github.com/Harmonybrew/ohos-zsh/releases/download/5.9/zsh-5.9-ohos-arm64.tar.gz
tar -zxf zsh-5.9-ohos-arm64.tar.gz -C /opt
ln -s /opt/zsh-5.9-ohos-arm64/bin/zsh /usr/bin/zsh
```

**2\. 安装 Homebrew**

在终端中执行这句命令进行安装

```sh
zsh -c "$(curl -fsSL https://harmonybrew.atomgit.com/install.sh)"
```

**3\. 配置环境变量**

按照安装脚本的提示，执行以下命令，将 Homebrew 加入到 PATH 中

```sh
echo >> ~/.mkshrc
echo 'eval "$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)"' >> ~/.mkshrc
eval "$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)"
```

现在可以开始使用 brew 命令了。
