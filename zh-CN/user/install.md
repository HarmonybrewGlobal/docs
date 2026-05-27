# Installation Guide

## HarmonyOS PC

**1. Uninstall Conflicting Software**

If GitNext and DevBox are installed on your PC, please uninstall them.

**2. Enable Developer Options**

Go to “Settings” -> ‘System’ -> “Developer Options” and toggle on “Developer Options”.

Go to “Settings” -> “Privacy and Security” -> “Advanced,” and enable the “Run extensions from outside the App Market” toggle.

**3. Install Homebrew**

Execute the following command in the terminal to install Homebrew

```sh
zsh -c “$(curl -fsSL https://harmonybrew.atomgit.com/install.sh)”
```

**4. Configure Environment Variables**

Follow the prompts in the installation script and run the following commands to add Homebrew to your PATH

```sh
echo >> ~/.zshrc
echo ‘eval “$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)”’ >> ~/.zshrc
eval “$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)”
```

You can now start using the `brew` command.

## HarmonyOS Development Board

**1. Configure Required Directories**

During Homebrew operation, directories such as `$HOME`, `/usr/bin`, and `/storage/Users/currentUser` are required. We need to manually configure these directories

```sh
mount -o remount,rw /
mkdir -p /usr
mkdir -p /data/storage/Users/currentUser
ln -s /bin /usr/bin

# The /storage directory is on tmpfs, so data is not persistent; you must recreate the symbolic link after each device reboot
ln -s /data/storage/Users /storage/Users

# In the hdc shell environment, the HOME variable defaults to the / directory, which has very little free space; it needs to be set to a directory with ample storage
# This configuration is only valid in the current terminal; it must be reset each time you enter the hdc shell
export HOME=/storage/Users/currentUser
```

**2. Install curl and zsh**

Download the [HarmonyOS version of curl](https://github.com/Harmonybrew/ohos-curl) and the [HarmonyOS version of zsh](https://github.com/Harmonybrew/ohos-zsh) on your host computer (Windows PC), then use hdc to push the tar packages to the device.

Extract the files on the device and create symbolic links to the commands in the `/usr/bin` directory

```sh
mount -o remount,rw /
tar -zxf curl-8.19.0-ohos-arm64.tar.gz -C /data
ln -s /data/curl-8.19.0-ohos-arm64/bin/curl /usr/bin/curl
tar -zxf zsh-5.9-ohos-arm64.tar.gz -C /data
ln -s /data/zsh-5.9-ohos-arm64/bin/zsh /usr/bin/zsh
`
