# 通过自定义 tap 分发软件包

## 前言

Harmonybrew 核心 tap（Harmonybrew/homebrew-core）的准入条件较为严格，详情请参阅 [如何贡献 formula](./contribute-formula.md)。对于无法满足核心 tap 准入规则的软件（例如闭源软件），建议通过 [自定义 tap](https://docs.brew.sh/How-to-Create-and-Maintain-a-Tap) 的方式进行分发。

这种分发方式仍然能够复用 Harmonybrew 的包管理器生态，用户下载软件包时仅需额外执行一条添加自定义 tap 的命令，对用户体验影响极小。

本文档将以 AtomGit 平台为例，演示如何基于该平台搭建自定义 tap，并分发一个名为 `hello-brew` 的闭源软件。

## 前置能力准备

维护自定义 tap 属于高级操作。在开始操作之前，请确保自己已掌握贡献 formula 的基本能力。若需学习，请参考 [如何贡献 formula](./contribute-formula.md)。

## 准备预构建包

以分发闭源软件 `hello-brew` 为例，假设预构建包名为 `hello-brew-1.0.0-ohos-arm64.tar.gz`。

其解压后的目录结构如下：
```text
hello-brew-1.0.0-ohos-arm64
├── bin
│   └── hello-brew
└── lib
    └── libhello.so
```

## 创建 Git 仓库

每个 tap 均对应一个 Git 代码仓，需在代码托管平台创建一个新的仓库作为 tap 载体。

本示例使用 AtomGit 平台，仓库地址为：`https://atomgit.com/Harmonybrew/custom-tap-example`

## 上传预构建包

由于构建 formula 的过程中需要下载该预构建包，因此需将其上传至可公开访问的存储地址。

操作示例：可直接利用 AtomGit 代码仓库的“发行版（Releases）”功能来存储预构建包。在本场景中，可在 `https://atomgit.com/Harmonybrew/custom-tap-example` 仓库下创建名为 `hello-brew-1.0.0` 的发行版，并将预构建包上传至其中。

上传完成后，记录该公开下载链接：`https://atomgit.com/Harmonybrew/custom-tap-example/releases/download/hello-brew-1.0.0/hello-brew-1.0.0-ohos-arm64.tar.gz`

## 制作并上传 Formula

将 Git 仓库克隆至本地，在仓库中创建 `Formula` 目录，并按首字母划分子目录，最后在该子目录中创建 formula 文件。

**操作步骤：**
```sh
git clone git@atomgit.com:Harmonybrew/custom-tap-example.git
cd custom-tap-example
mkdir -p Formula/h
vim Formula/h/hello-brew.rb
```

**Formula 模板示例：**
```rb
class HelloBrew < Formula
  desc "Hello world program with bin and so"
  homepage "https://atomgit.com/Harmonybrew/custom-tap-example"
  url "https://atomgit.com/Harmonybrew/custom-tap-example.git", branch: "main"
  version "1.0.0"

  resource "ohos_arm64_dist" do
    url "https://atomgit.com/Harmonybrew/custom-tap-example/releases/download/hello-brew-1.0.0/hello-brew-1.0.0-ohos-arm64.tar.gz"
    sha256 "06a1388a56d0b1bfad1f02936fe8a379bc3dcff2975bf69c3c4a39074edc31ba"
  end

  def install
    resource("ohos_arm64_dist").stage do
      prefix.install Dir["*"]
    end
  end
end
```

> 注意：请根据实际情况替换其中的下载地址及元数据。若压缩包结构为非标准结构，需自行调整 install 函数。

Formula 编写完成后，提交至远程仓库：
```sh
git add .
git commit -m "hello-brew 1.0.0 (new formula)"
git push
```

## 制作并上传 Bottle

进入 `ci-runner` 容器，添加自定义 tap：
```sh
# 启动并进入 ci-runner 容器
docker pull swr.cn-north-4.myhuaweicloud.com/harmonybrew/ci-runner:latest
docker run -itd --name=ohos swr.cn-north-4.myhuaweicloud.com/harmonybrew/ci-runner:latest
docker exec -it ohos sh

cd ~
brew update

# 添加自定义 tap
brew tap harmonybrew/custom-tap-example https://atomgit.com/Harmonybrew/custom-tap-example
```

配置 SSH 密钥、Git 用户信息及仓库信息，确保具备提交权限：
```sh
ssh-keygen -t ed25519       # 建议一路回车
cat ~/.ssh/id_ed25519.pub   # 将此公钥添加至 AtomGit 设置

# 配置 Git 全局信息
git config --global user.email "you@example.com"
git config --global user.name "Your Name"

# 为自定义 tap 添加 SSH 远程地址，以便在容器内直接提交更新
cd $(brew --repository harmonybrew/custom-tap-example)
git remote add ssh-repo git@atomgit.com:Harmonybrew/custom-tap-example.git
cd -
```

制作 bottle 并更新 formula：
```sh
# 以 build-bottle 模式从源码安装（实际流程为下载预构建包）
brew install -v --build-bottle hello-brew

# 生成 bottle 和软件包元数据
# --root_url 需指向发行版的下载路径，以便 Homebrew 正确拼接 bottle 地址
ROOT_URL="https://atomgit.com/Harmonybrew/custom-tap-example/releases/download/hello-brew-1.0.0"
brew bottle -v --skip-relocation --json --root-url=$ROOT_URL hello-brew

# 更新 formula 文件（自动合入 bottle 信息）并推送到远程仓库
JSON_FILE=$(ls hello-brew-*.json)
brew bottle --write --merge $JSON_FILE
cd $(brew --repository harmonybrew/custom-tap-example)
git push ssh-repo
cd -
```

执行完成后，当前目录将生成 bottle 文件（如 `hello-brew-1.0.0.arm64_ohos.bottle.1.tar.gz`）。将其上传至 AtomGit 对应发行版中即可。

## 验证 Bottle 有效性

启动全新容器验证安装流程：
```sh
# 清理旧容器
docker rm -f ohos

# 启动新容器
docker run -itd --name=ohos swr.cn-north-4.myhuaweicloud.com/harmonybrew/ci-runner:latest
docker exec -it ohos sh

cd ~
brew update

# 添加自定义 tap
brew tap harmonybrew/custom-tap-example https://atomgit.com/Harmonybrew/custom-tap-example

# 安装自定义 tap 中的软件包
brew install hello-brew

# 执行软件包里面的命令，验证它是否正常工作
hello-brew
```

## 注意事项

*   **命名冲突**：自定义 tap 中的软件包名称建议避免与核心 tap 重名。若存在重名包，Homebrew 默认优先安装核心 tap 版本，除非用户显式指定 tap 路径。

## 完整样例

完整的自定义 tap 示例已在以下仓库中开源，供参考：https://atomgit.com/Harmonybrew/custom-tap-example
