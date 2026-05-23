# 如何贡献 formula

## 1 前言

向系统级软件仓库录入软件包并非易事。在业界，这项工作通常由深度用户或系统开发者完成，而非普通用户。

如果你目前的开发经验尚处于“只会调包，不会传包”的阶段，从未向官方 Homebrew 或主流 Linux 发行版（如 Debian、Arch Linux 等）贡献过软件包，我们不建议你直接尝试向 Harmonybrew 提交贡献。

向 Harmonybrew 录入软件包的过程并不比其他软件仓库简单。相反，由于操作系统环境的特殊性以及基础设施的现状，适配工作面临更多挑战。如果你在成熟平台上尚且无法独立完成贡献，在这里只会更加寸步难行。

如果你认为自己已经准备好迎接这些挑战，那就来吧。欢迎开启 OpenHarmony 命令行生态的拓荒之旅。

## 2 前置知识学习

在贡献 formula 之前，请务必掌握以下资料：

<table>
  <thead>
    <tr>
      <th>资料</th>
      <th>介绍</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://docs.brew.sh/">Homebrew Documentation</a></td>
      <td>
        Homebrew 官方文档。在“跑”之前请先学会“走”，请务必先成为 Homebrew
        深度用户，透彻了解其底层运作机制。
      </td>
    </tr>
    <tr>
      <td>
        <div style="min-width: max-content; display: block; white-space: nowrap;">
          <a href="https://docs.brew.sh/Adding-Software-to-Homebrew">Adding-Software-to-Homebrew</a>
        </div>
      </td>
      <td>
        Homebrew 官方贡献指南。Harmonybrew 的贡献流程尽可能与上游官方保持一致。
      </td>
    </tr>
    <tr>
      <td><a href="../user/FAQ.md">常见问题</a></td>
      <td>
        常见问题集。里面有介绍到 Harmonybrew 项目细节及 OpenHarmony
        平台的相关知识。
      </td>
    </tr>
  </tbody>
</table>

## 3 环境准备

### 3.1 准备 Docker 环境

为了解决 formula 构建过程中极其复杂的工具链依赖问题，并确保你的代码在提交后能够顺利通过流水线构建，Harmonybrew 的所有开发、调试与构建工作**必须在统一的 Docker 容器中完成。**

这种“容器化开发”模式有以下优势：

- **环境对齐**：你的本地环境与流水线环境 100% 一致，“在我的机器上能跑通”即意味着“在流水线上能跑通”。
- **工具集成**：容器内已预置好 ohos-sdk、make 等开发工具，无需手动配置复杂的编译环境。
- **无损撤销**：所有实验性修改均在容器内进行，不会污染你的宿主机系统。

<br>

因此，你首先需要准备一个能够运行 [DockerHarmony](https://github.com/hqzing/dockerharmony) 的环境。

相关要求如下：

- **硬件要求**：建议使用 arm64 原生硬件（如 arm 服务器或 Mac 电脑）。若必须使用 x86_64 设备，请参考 DockerHarmony 文档注册 QEMU 解释器。需注意：在编译构建场景下，QEMU 模拟运行的性能通常仅为原生硬件的 5% ~ 10%，效率极低。
- **网络要求**：构建过程中需实时从 GitHub 下载源码，请确保开发环境能够流畅访问 GitHub。建议选用香港或海外地域的 arm 服务器。

### 3.2 启动并进入容器

从华为云 SWR 拉取最新 [ci-runner](https://atomgit.com/Harmonybrew/ci/blob/main/ci-runner/Dockerfile) 镜像并启动：

```sh
docker pull swr.cn-north-4.myhuaweicloud.com/harmonybrew/ci-runner:latest
docker run -itd --name=ohos swr.cn-north-4.myhuaweicloud.com/harmonybrew/ci-runner:latest

# 进入容器环境，后续的所有步骤均在容器中进行
docker exec -it ohos sh
```

这个镜像是我们的流水线镜像。它是 DockerHarmony 的二次封装，预置了必要的开发工具。我们将这个镜像对外公开，作为标准化的开发环境供贡献者使用。

> **注意**：镜像会持续更新。为确保环境始终最新，建议在每次使用前执行 `docker pull`。

### 3.3 更新 Homebrew

执行一次手动更新，将 Homebrew 包管理器和核心 tap 更新至最新版本：

```sh
brew update
```

> 在这个容器中，默认设置了环境变量 `HOMEBREW_NO_AUTO_UPDATE=1`，全局禁用 Homebrew 自动更新。此举是为了避免开发者在工作过程中因 Homebrew 自动更新而丢失本地改动。因此，你每次工作前需要手动执行更新操作，以同步最新的包管理器和核心 tap 代码。

## 4 新增或修改 formula

### 4.1 新增 formula

**A. 搬运上游已有的 formula**

如果开源软件在上游 [Homebrew/homebrew-core](https://github.com/Homebrew/homebrew-core) 中已存在，**必须直接搬运上游 formula，严禁从零手写**。

这样做的目的是确保构建配置、依赖关系与上游保持一致，避免因打包差异导致连锁依赖失效。即便需要针对 OpenHarmony 环境做适配，也应在上游版本的基础上进行修改。若有特殊情况必须全量重写，请在 PR 中注明原因。

以 `sevenzip` 为例，搬运流程如下：

```sh
# 进入核心 tap 目录
cd $(brew --repo harmonybrew/core)

# 从 Homebrew 上游下载 formula 文件到对应的首字母子目录 (s)
# 若 Aliases 目录存在相关软链接，需一并创建
curl -fL -o Formula/s/sevenzip.rb https://raw.githubusercontent.com/Homebrew/homebrew-core/refs/heads/main/Formula/s/sevenzip.rb
ln -s ../Formula/s/sevenzip.rb Aliases/7-zip
ln -s ../Formula/s/sevenzip.rb Aliases/7zip

# 从源码安装（触发本地实时构建）
brew install -s -v --include-test sevenzip

# 执行冒烟测试，验证功能
brew test sevenzip
```

> **适配提示**：若上游 formula 在 OpenHarmony 环境下构建失败，可以尝试禁用不支持的可选功能，或制作专门的适配补丁（见附录）。

**B. 手写全新的 formula**

若上游仓库不存在该软件，则需手写全新的 formula，并遵循以下准则：

- **准入规则**：必须符合上游的 [Acceptable-Formulae](https://docs.brew.sh/Acceptable-Formulae) 标准，例如必须是开源软件、必须能从源码构建（不能录入你自己构建好的二进制，这是为了防止投毒）、不能是 fork 版本、不能过于冷门等。
- **代码规范**：必须通过 `brew style` 和 `brew audit` 检查。
- **语言要求**：为便于后续与上游社区交流，请使用英文注释。

<br>

如果你的软件包具备可移植性，建议先将其录入上游 [Homebrew/homebrew-core](https://github.com/Homebrew/homebrew-core)，再从上游搬运回来，而不是直接往下游仓库录入。如此一来，你的软件包不仅能覆盖更多操作系统，还能尽可能减少被我们拒收的可能性。因为从上游搬运过来的 formula 我们通常不会拒收。

如果你的软件包因闭源、无法从源码构建等原因不满足核心仓库准入规则，请通过自定义 tap 的方式进行分发。参考：[通过自定义 tap 分发软件包](./custom-tap.md)。

> **特例**：`ohos-sdk` 和 `rust` 是核心仓库中唯二的特例。这两套工具链目前无法在 OpenHarmony 平台上完成自举，因其作为生态基石的特殊地位，我们直接分发官方社区的构建版本。

### 4.2 修改 formula

如果有修改 formula 的需求，请直接在 homebrew-core 仓库目录进行修改。

以 sevenzip 为例，修改流程如下：

```sh
# 进入核心 tap 目录
cd $(brew --repo harmonybrew/core)

# 对 formula 文件进行修改
vim Formula/s/sevenzip.rb

# 从源码安装（触发本地实时构建）
brew install -s -v --include-test sevenzip

# 执行冒烟测试，验证功能
brew test sevenzip
```

## 5 提交 PR

### 5.1 配置 SSH 密钥对

为了能够推送代码，需生成 SSH 密钥并将其公钥录入 [AtomGit 设置中心](https://atomgit.com/setting/key-ssh)。

```sh
# 生成密钥对（建议直接一路回车）
ssh-keygen -t ed25519

# 查看公钥内容。请将其添加到你的 AtomGit 账号上。
cat ~/.ssh/id_ed25519.pub
```

### 5.2 推送代码至个人仓库

如果你尚未 fork 过 [Harmonybrew/homebrew-core](https://atomgit.com/Harmonybrew/homebrew-core)，请先进行 fork。

随后在容器内将个人仓库添加为远程地址，并提交代码：

```sh
# 进入核心 tap 目录
cd $(brew --repo harmonybrew/core)

# 给 git 做配置，设置你的用户名和邮箱
git config --global user.email "you@example.com"
git config --global user.name "Your Name"

# 添加你的个人远程仓库（以用户 hqzing 为例）
git remote add my-fork git@atomgit.com:hqzing/homebrew-core.git

# 规范提交信息
git add .
git commit -m "sevenzip 26.00 (new formula)"

# 推送至个人仓的新分支（请勿直接推送到 main 分支）
git push my-fork main:sevenzip
```

提交信息（Commit Message）规范：

| 场景         | 格式                                   | 示例                                 |
| :----------- | :------------------------------------- | :----------------------------------- |
| 新增 formula | `<formula> <version> (new formula)`    | `make 4.4.1 (new formula)`           |
| 版本升级     | `<formula> <version>`                  | `ldapvi 1.8`                         |
| 修订版本升级 | `<formula>: revision bump to <reason>` | `marp-cli: revision bump to node@24` |
| 修复/增强    | `<formula>: <action>`                  | `tcl-tk: update livecheck blocks`    |

### 5.3 提交 PR

去 AtomGit 网页上操作，从自己的个人仓往 [Harmonybrew/homebrew-core](https://atomgit.com/Harmonybrew/homebrew-core) 提交 PR。

PR 提交完成后，HarmonybrewBot 机器人会自动生成一个同名的“替代 PR”。

> **为什么要生成“替代 PR”？**
> 因为 AtomGit 平台的能力相比于 GitHub 还是有所欠缺，并不支持设置 “Allow edits by maintainers”，而我们的业务非常依赖该特性（机器人在跑完流水线后需往 PR 中添加新的 commit）。在平台能力有限的情况下，只能通过这种妥协方式实现。

这个“替代 PR”需要维护者进行评审。评审通过后（维护者点击“通过”或手动添加 `request-ci` 标签），流水线会自动执行构建、测试、打包等动作。构建结束后，机器人会将构建日志链接发至 PR 讨论区。

**如果流水线构建成功：**

- **“替代 PR”与原始 PR 均会被合并**：机器人会定期扫描并合并构建成功的 PR（当前扫描周期为 10 分钟）。机器人检查到构建成功后，会首先合并“替代 PR”。随后，你的原始 PR 也会随之自动变更为“已合并”状态。
- **软件包将会入库**：在 PR 合并后，软件包正式进入制品仓库。受 CDN 缓存刷新策略影响，新版本最长需要 10 分钟的生效延迟，届时其他用户即可下载到你录入或修改的软件包。

**如果流水线构建失败：**

- **原 PR 需关闭并重新提交**：请手动关闭原 PR，根据构建日志修改代码后重新提交一个新的 PR。在原始 PR 上做更新并不能改变流水线的运行结果，因为“替代 PR”不会随原始 PR 的更新而同步更新。

<br>

> **建议**：完成一轮贡献后建议删除容器。下次工作时创建新容器，以免“不干净”的环境对后续构建造成干扰。

## 6 进阶玩法：自动化搬运

如果你已经对这套提交流程驾轻就熟，你会发现：在大部分无需修改源码或打补丁的场景下，搬运工作其实是一套机械化的标准动作。

为了提升效率，我们开发了自动化迁移工具：[formula-migration-tool](https://atomgit.com/Harmonybrew/formula-migration-tool)。该工具能够自动完成这一系列流程：从上游同步、创建 Alias 软链接、本地构建验证、规范提交代码。

建议在熟悉手动流程后，按照该仓库 README 的指引配置使用，这能极大地节省大批量软件包录入时的重复劳动。

## 7 附录：补丁制作指南

### 7.1 进入 superenv

Homebrew 默认在 **superenv**（超级编译环境）中构建。它通过**环境变量注入**（如 `CFLAGS`）和**编译器劫持**（使用 shim 垫片脚本代替系统的 `gcc` 等命令）来屏蔽宿主环境的差异，确保构建环境的标准化与一致性。

为了复现编译报错，必须先进入该环境：

```sh
brew install -s -i <formula>
```

执行后，Homebrew 会完成下载、解压、应用已有补丁等前序动作，最后启动一个交互式终端并停留在源码目录。这相当于让你身处 formula 里面的 `install` 函数内部进行手工调试。

### 7.2 进行手工构建

进入 superenv 的交互式终端后，你就可以按照 formula 里面的 `install` 函数的逻辑进行手工构建（例如自己执行 `./configure`、`make` 等命令）。

手工构建时请务必确保自己的构建行为与 `install` 函数中的行为完全一致。如果你不确定它到底执行了哪些操作，可以先在 superenv 之外执行 `brew install -s -v <formula>` 来观察构建日志，从中得到具体的构建命令。

当你通过手工构建复现了之前的报错后，就可以着手处理它了。你可以直接在终端里对源码进行适配修改，改完代码后继续在当前环境下执行构建命令，直到构建顺利通过。

> 本项目不会专门提供鸿蒙适配相关的指导，仅在力所能及的范围内分享一些参考资料：
>
> 1. [aports](https://gitlab.alpinelinux.org/alpine/aports)：Alpine Linux 的软件仓库。由于其 C 标准库采用 musl libc，该仓库中积累了大量针对非 glibc 环境的兼容性补丁，是解决 C 标准库差异的重要参考。
> 2. [termux-packages](https://github.com/termux/termux-packages)：Termux 的软件仓库。Termux 在 Android 平台上积累了丰富的应用沙箱与非 FHS 路径的适配实践，相关补丁对于解决 OpenHarmony 应用沙箱环境下的难题极具借鉴价值。

### 7.3 制作补丁

适配完成后，需制作标准补丁文件。推荐做法如下：

1. **清理环境**：在 superenv 中执行 `make clean`（或同类命令）清理构建产物。
2. **生成 Diff**：另找一处存放同版本的“干净源码”，使用 `diff` 工具生成补丁：
   ```sh
   # -r: 递归目录; -u: 统一格式; -N: 处理缺失文件;
   diff -ruN path/to/clean_source path/to/modified_source > 0001-add-ohos-support.patch
   ```

### 7.4 本地验证

在将补丁正式贡献给社区之前，需在个人环境验证其有效性：

1. **放置补丁**：在核心 tap 仓库的 `Patches` 目录下创建一个与 formula 同名的子目录，将补丁放置其中。

   示例：`Patches/perl/0001-add-ohos-support.patch`

2. **引用补丁**：在本地 formula 中加入 `patch` 块，指向补丁文件：

   示例：

   ```rb
   patch do
     file "Patches/perl/0001-add-ohos-support.patch"
   end
   ```

3. **验证补丁**：退出 superenv，执行 `brew install -s -v <formula>` 验证构建是否通过。

### 7.5 提交 PR

在本地验证通过并确认无误后，将 formula 和补丁一同提交至 [Harmonybrew/homebrew-core](https://atomgit.com/Harmonybrew/homebrew-core) 仓库。
