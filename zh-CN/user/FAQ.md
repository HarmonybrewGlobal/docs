# 常见问题

目录
* 非故障现象
    * [卸载 Homebrew 时，卸载脚本没有自动清理 zshrc](#not-clean-zshrc)
    * [通过后装软件包覆盖系统命令后，调用到的仍是系统命令](#failed-to-override-system-command)
* 项目实现
    * [Tier 1 和 Tier 2 平台的区别](#difference-between-tier1-and-tier2)
    * [我们对 Homebrew 做了哪些改动？](#what-changes-have-we-made)
    * [我们为什么能继承大部分上游的 formula？](#why-can-we-inherit-formula)
    * [为什么选择 Homebrew 而不是别的包管理器？](#why-homebrew)
* 系统环境解析
    * [OpenHarmony、HarmonyOS 与 GNU/Linux 的关系](#relationship-between-OH-HO-linux)
    * [代码签名机制对我们的影响](#effect-of-code-signing)
    * [应用沙箱环境和非应用沙箱环境的区别](#difference-between-sandbox-and-non-sandbox)
    * [HMDFS 对我们的影响](#effect-of-hmdfs)
* 贡献和维护
    * [如何访问流水线？](#how-to-access-pipeline)
    * [如何加入维护团队？](#join-the-maintenance-team)
* 支持和规划
    * [对旧版本系统的兼容性](#compatibility-with-older-system)
    * [如何提供商用支持？](#provide-commercial-support)
    * [是否能支持其他架构？](#can-it-support-other-architectures)
    * [什么时候能回馈上游？](#contribute-to-upstream)

<div id="not-clean-zshrc"></div>

## 卸载 Homebrew 时，卸载脚本没有自动清理 zshrc

*   **现象描述**：

    在卸载 Homebrew 后重新打开 HiShell，终端可能会提示报错：`no such file or directory: /storage/Users/currentUser/.harmonybrew/bin/brew`。

*   **原因分析**：

    这不是 bug，上游版本（官方 Homebrew）的行为便是如此。

    在首次安装 Homebrew 时，用户通常会根据脚本提示，手动向 `~/.zshrc`（或 `~/.mkshrc`）中添加如下环境变量初始化配置：
    `eval "$(/storage/Users/currentUser/.harmonybrew/bin/brew shellenv)"`
    
    由于该配置项是由用户**手动写入**（或手动执行命令追加）的，而非安装脚本自动生成的私有配置，Homebrew 的卸载程序不会擅自修改或清理用户的个人 Shell 配置文件。因此，当 Homebrew 被卸载后，残留的 `eval` 指令因找不到目标文件而报错。

*   **解决方案**：

    如需彻底清除报错，请手动编辑您的配置文件：
    1.  打开配置文件：`vim ~/.zshrc`（或使用您偏好的编辑器）。
    2.  删除包含 `.harmonybrew` 或 `brew shellenv` 关键字的相关行。
    3.  保存并退出，重新打开终端即可。


<div id="failed-to-override-system-command"></div>

## 通过后装软件包覆盖系统命令后，调用到的仍是系统命令

*   **现象描述**：

    在鸿蒙 PC 的 HiShell 环境下，通过 Homebrew 安装了与系统重名的软件包（如 `curl`）后，即便 Homebrew 的 bin 路径已在 PATH 之中，执行命令时仍会调用系统内置版本，而非 Homebrew 安装的版本。
    ```sh
    # 场景复现
    curl -V              # 调用系统自带 curl
    brew install curl    # 安装新版本覆盖
    command -v curl      # 返回仍为 /usr/bin/curl
    ```

*   **原因分析**：

    该现象由 Zsh 的 **Command Hashing（命令哈希）** 机制引起，属于 Shell 的正常行为而非故障。

    为了优化性能，Zsh 会将首次调用过的命令路径缓存至内部的“命令哈希表”中。当下一次执行相同命令时，Zsh 会优先从缓存表中读取路径，而不再重新遍历 PATH 环境变量。因此，在安装新软件包之前若已调用过系统同名命令，缓存记录会导致后装的软件包被“屏蔽”。

*   **解决方案（二选一）**：

    *  **重置缓存**：在当前 Shell 中执行 `hash -r` 命令，强制清除并刷新命令哈希表。
    *  **重启环境**：关闭当前的 HiShell 窗口并重新打开，使 Zsh 重新加载环境并扫描 PATH。

<div id="difference-between-tier1-and-tier2"></div>

## Tier 1 和 Tier 2 平台的区别

这两个等级的区别主要体现在对 Formula 的**维护策略**与**质量保障**上：

*   **Tier 1：核心支持平台**
    *   **准入机制**：在该平台上，Formula 必须通过 `brew test` 自动化测试。若测试失败，则该 Formula 会被拒绝合入核心软件仓库。
    *   **典型环境**：社区版 OpenHarmony 系统。
    *   **覆盖设备**：鸿蒙开发板、鸿蒙容器。
*   **Tier 2：次级支持平台**
    *   **维护策略**：采取“生态随行”策略。我们致力于提供基础运行能力，但不会针对这些平台进行强制性的自动化测试或人工回归。
    *   **典型环境**：HarmonyOS 系统。
    *   **覆盖设备**：鸿蒙 PC。

通过这种分层机制，我们能够确保在核心开发环境（Tier 1）的稳定性，同时兼顾更多商用终端（Tier 2）的生态覆盖。

<div id="what-changes-have-we-made"></div>

## 我们对 Homebrew 做了哪些改动？

为了让 Homebrew 适配 OpenHarmony（尤其是鸿蒙 PC 的环境限制），我们对包管理器客户端和安装脚本进行了一系列定制修改。

若你对修改点感兴趣，可对相关仓库进行 diff 操作，查看比较结果。

包管理器客户端：
```sh
mkdir upstream downstream
git clone --depth 1 -b 5.1.11 https://github.com/Homebrew/brew.git upstream/brew
git clone --depth 1 https://atomgit.com/Harmonybrew/brew.git downstream/brew
diff -ruN -x ".git" upstream/brew downstream/brew > out.diff
vim out.diff
```

安装脚本：
```sh
mkdir upstream downstream
git clone https://github.com/Homebrew/install.git upstream/install
git clone https://atomgit.com/Harmonybrew/install.git downstream/install
cd upstream/install
git reset --hard 39c012e6db58f84f7469e9346adba57f8302110b
cd -
diff -ruN -x ".git" upstream/install downstream/install > out.diff
vim out.diff
```

这里给出一个简略版的修改点总结：

*   **基础设施重定向**：将硬编码的 GitHub 代码仓与制品仓（GHCR）地址切换至 AtomGit 仓库与自定义 CDN，以保障国内环境下的顺畅访问与极速下载。
*   **平台识别与路径重构**：新增 `ohos` 平台标志变量，在复用 Linux 业务逻辑的同时，可以针对 OpenHarmony 平台的特有差异进行处理；将所有安装、缓存及临时目录重定义至 `/storage/Users/currentUser` 下，以规避鸿蒙 PC 系统目录的只读限制。
*   **安全与权限适配**：针对鸿蒙 PC 的单用户及安全特性，剔除所有 `sudo` 相关逻辑。同时在构建流程中注入自动代码签名机制，确保所有 ELF 文件符合系统校验要求，避免运行被拦截。
*   **工具链与 C 库定制**：将默认编译器由 gcc 切换为 clang；禁用自包含 glibc 机制，强行链接系统原生的 `musl libc`，以确保在鸿蒙 PC 上的兼容性并降低环境侵入。
*   **核心机制补全（portable-git）**：借鉴 `portable-ruby` 理念，我们为鸿蒙开发了 `portable-git`。当检测到系统未预置 git 时，Homebrew 会自动下载 git 工具供自己使用，实现“开箱即用”。
*   **Shell 运行环境兼容**：由于鸿蒙 PC 缺少 `/bin/bash`，我们对项目中所有 Shell 脚本的 shebang 进行了 Zsh 适配修改，并修复了大量 Bash 专有语法在 Zsh 环境下的执行异常。
*   **系统命令适配（toybox）**：重写了 `find`、`ps`、`file` 等外部命令的调用逻辑，确保在系统仅内置 `toybox` 精简工具集的情况下，Homebrew 包管理器的各项功能依然能正常工作。
*   **稳定性增强与特性裁剪**：为了确保软件包稳定可靠，我们禁用了在鸿蒙上存在兼容风险的“软件包重定位”与“隐式依赖检测”特性。
*   **开发者生态支持**：适配 AtomGit API，使 `bump-formula-pr` 等维护命令能够自动向 AtomGit 提交 PR。

<div id="why-can-we-inherit-formula"></div>

## 我们为什么能继承大部分上游的 formula？

核心逻辑在于，我们没有重复造轮子，而是把“蹭生态”的艺术玩到了极致。

能够完全无缝继承 Homebrew 海量的上游 Formula，取决于两个层面的设计：

*   **业务设计层面**：我们对 Homebrew 业务代码做适配的时候，尽可能复用了 Linux 平台上的业务逻辑。
    * **系统身份泛化**：在代码底层，我们将 OpenHarmony 视为一种特殊的 Linux 发行版。通过走 Linux 系统的业务路径，大部分 Formula 无需修改即可直接在 OpenHarmony 上进入构建流程。
    * **Superenv 垫片适配**：我们对 Homebrew 的 `superenv` 垫片（cc, gcc, ld 等）进行了适配。当 Homebrew 启动构建时，这些垫片能自动拦截编译指令并重定向至 ohos-sdk 中的 LLVM 编译器，实现工具链的无感切换。

*   **构建环境层面**，我们基于 [DockerHarmony](https://github.com/hqzing/dockerharmony) 封装了专用的 [流水线镜像](https://atomgit.com/Harmonybrew/ci/blob/main/ci-runner/Dockerfile)，通过模拟标准开发环境扫清构建障碍。
    * **编译器欺骗机制**：效仿 macOS 的做法，在镜像中将 `cc`、`gcc`、`ld` 等指令软链接至 LLVM。即使部分软件绕过了 `superenv`，依然能在 `configure` 阶段自动识别并调用正确的编译器，无需手动干预环境变量。
    * **内核级平台兼容**：由于鸿蒙容器直接运行在 Linux 内核上，系统 `uname` 原生返回 `Linux` 标识。我们利用这一特性，使绝大多数开源软件能自动匹配成熟的 Linux 构建分支，避免因识别为“未知系统”而导致构建中断。
    * **标准 GNU 工具链补齐**：镜像预置了完整版的 `tar`、`grep`、`awk` 等标准 GNU 工具。这确保了那些默认系统已装有上述工具的 Formula 能够正常运行，无需在每一个 Formula 中额外声明构建依赖。
    * **隐式构建依赖注入**：Homebrew 默认运行环境已具备 `make`、`perl` 等基础开发工具。我们在流水线镜像中对标补齐了这些隐式依赖，从而保障了上游 Formula 在鸿蒙环境下的构建行为与原生 Linux/macOS 保持高度一致。

<div id="why-homebrew"></div>

## 为什么选择 Homebrew 而不是别的包管理器？

首先需要明确：Harmonybrew 是一个由社区发起的开源项目，其技术选型由项目创建者 [@hqzing](https://atomgit.com/hqzing) 基于个人调研独立做出。此选型与 OpenHarmony 社区或相关 PC 产品的官方规划均无直接或间接关系。应避免误解为“鸿蒙选择了 Homebrew”或“鸿蒙 PC 选择了 Homebrew”。

在针对 OpenHarmony 系统进行包管理器方案调研时（详见 @hqzing 编写的技术博客：[Linux包管理器生态漫谈](https://blog.csdn.net/hqzing/article/details/158005274)），@hqzing 认为 Homebrew 的交互逻辑和生态特征更契合个人 PC 用户的使用习惯。考虑到 PC 终端在鸿蒙生态中的重要性，决定重点开展 Homebrew 的移植工作。

Homebrew 并非我们的唯一路径。目前，@hqzing 已同步完成了 pkgsrc 的移植并创建了 [pkgsrc-ohos](https://github.com/pkgsrc-ohos/docs) 项目，该方案同样能够支持鸿蒙 PC 环境，为用户提供了另一种轻量化的选择。

这两套系统呈现出截然不同的技术特性，分别代表了包管理器设计的两个极端，旨在为 OpenHarmony 生态提供全方位的技术参考：

1. **机制深度：极繁 vs 极简**
    *   **Homebrew**：侧重于强大的自动化机制。它混合了大量的 Shell 与 Ruby 脚本，实现了如 `superenv`（超级环境）、自包含 glibc 以及软件包动态重定位等复杂特性。为了在鸿蒙上完美运行，我们对其业务逻辑进行了深度的适配。
    *   **pkgsrc**：追求极致的轻量与纯粹。它由一套构建系统和一套管理工具组成，逻辑简洁，通过源码重编译即可快速接入 OpenHarmony。

2. **实现方式：脚本实现 vs 二进制实现**
    *   **Homebrew**：纯脚本实现，高度依赖 Ruby 解释器及大量的外部工具链（`git`, `curl`, `file` 等），灵活性高，适合资源充足的 PC 环境。
    *   **pkgsrc**：其核心组件（如 `bmake`, `pkg_install`）完全由 C 语言编写，编译生成的二进制文件可直接运行，无需额外依赖，对系统环境的需求极低。

3. **发布模式：滚动更新 vs 周期发布**
    *   **Homebrew**：典型的滚动更新模式，源源不断地为用户推送最新版本的软件包，保持生态的实时性。
    *   **pkgsrc**：遵循严格的季度发布计划，通过固化版本分支来确保环境的高度稳定与可预期。

通过对上述两套路线迥异的方案进行实践，我们已经完成了初步的“技术打样”。这两套方案覆盖了包管理设计的不同形态，后续开发者在移植其他包管理器时（如 Nix 或 Pacman 等），将有迹可循。

<div id="relationship-between-OH-HO-linux"></div>

## OpenHarmony、HarmonyOS 与 GNU/Linux 的关系

关于这方面的背景知识，请参考 [@hqzing](https://atomgit.com/hqzing) 编写的技术博客：[OpenHarmony、HarmonyOS 与 GNU/Linux 的关系](https://blog.csdn.net/hqzing/article/details/160717501)

<div id="effect-of-code-signing"></div>

## 代码签名机制对我们的影响

关于这方面的背景知识，请参考 [@hqzing](https://atomgit.com/hqzing) 编写的技术博客：[代码签名机制对我们的影响](https://blog.csdn.net/hqzing/article/details/160746583)

这里仅补充讲解 Harmonybrew 项目里面的情况：在 Harmonybrew 项目中，“二进制签名工具签名”和“链接器签名”这两种签名方式我们都有使用到。

首先是二进制签名工具签名：我们的流水线在执行编译构建的时候不会启用链接器签名，而是等整体编译构建任务完成后，在打包 bottle 之前统一用二进制签名工具签一遍。

这是因为，如果仅靠链接器签名，会有一些场景覆盖不到：
1. 在我们的仓库中，并非所有软件包都是使用 ohos-sdk 里面的 LLVM 来编译的。比如 `gh` 是用 Go 编译器来进行编译的、`node` 是用 Alpine Linux 软件仓库里面的 GCC 编译器来进行编译的。对于这些软件包，ohos-sdk 的链接器签名是照顾不到的。
2. 部分软件包会在编译完成后再次对二进制文件进行修改。比如 `brotli` 这个软件包，它的构建系统（cmake）会在编译完成后对二进制文件进行修改。即使我们为它做了链接器签名，经过 cmake 修改之后签名也会失效，仍无法通过验签。

因此我们分发的软件包全部都是用二进制签名工具做的签名。

其次是链接器签名：我们虽然有用到这种方式，但并不是用来分发软件包的，而是给用户提供方便的。

我们在制作 [ohos-sdk](https://atomgit.com/Harmonybrew/homebrew-core/blob/main/Formula/o/ohos-sdk.rb) 这个 formula 的时候，对 ohos-sdk 里面的 lld 链接器做了简单的脚本封装，默认启用了 [链接器签名](https://gitcode.com/openharmony/third_party_llvm-project/pull/882)，使得它编出来的程序可以直接在鸿蒙 PC 上运行，无需手动进行代码签名。

<div id="difference-between-sandbox-and-non-sandbox"></div>

## 应用沙箱环境和非应用沙箱环境的区别 

关于这方面的背景知识，请参考 [@hqzing](https://atomgit.com/hqzing) 编写的技术博客：[应用沙箱环境和非应用沙箱环境的区别](https://blog.csdn.net/hqzing/article/details/160748691)

<div id="effect-of-hmdfs"></div>

## HMDFS 对我们的影响 

关于这方面的背景知识，请参考 [@hqzing](https://atomgit.com/hqzing) 编写的技术博客：[HMDFS 对我们的影响](https://blog.csdn.net/hqzing/article/details/160773683)

这里仅补充讲解 Harmonybrew 项目里面的情况：我们的 Harmonybrew 正是安装在 `/storage/Users/currentUser` 目录下，在HiShell 环境中，它正是承载在 HMDFS 上的。

既然 HMDFS 有这么多缺陷，为什么 Harmonybrew 还要执意安装在这里呢？

这其实是一个极其无奈的选择。在鸿蒙 PC 的应用沙箱内，系统管控非常严格，**这个家目录是目前唯一一个既能让用户有写权限、又能实现跨应用共享的目录**。如果我们想让 Harmonybrew 安装的软件在 HiShell 里能用，在 CodeArts IDE 这种开发工具里也能调用，我们就只能选这里，别无他处。

我们曾多次向厂商反馈，希望能在 PC 上新增一些基于普通文件系统的 FHS 目录（比如 `/usr/local` 或 `/opt`），但这些意见最终都没被采纳。作为开发者，我们没有更好的目录可以选择。

这就引出了一个为了全局稳定性而做的牺牲：全平台路径统一。

为了让 Harmonybrew 的生态在所有设备上保持一致，无论是在 PC、开发板还是容器中，我们都强制统一了安装路径。这意味着，即使在**鸿蒙开发板**或**鸿蒙容器**中，我们本可以随心所欲地使用更标准的 `/usr/local`、`/opt` 等目录，但为了迁就**鸿蒙 PC** 这个“最小公约数”，大家不得不统一使用 `/storage/Users/currentUser`。

不过有一点可以让开发者放心：虽然路径看起来一样，但底层的“土壤”不同。
* **在鸿蒙开发板上**，我们首推的使用方法是在 hdc shell 调试终端中使用 Harmonybrew，并没有进入到应用沙箱中。因此，我们也就没有使用应用沙箱提供的家目录，而是直接在常规文件系统中创建了这个路径，所以它不受 HMDFS 限制。
* **在鸿蒙容器中**，这个路径也是在容器自己的 rootfs 上创建的，同样没有这些限制。

<div id="how-to-access-pipeline"></div>

## 如何访问流水线？

本项目使用华为云 CodeArts Pipeline 负责 CI/CD 任务。由于平台权限限制，流水线后台目前无法直接对公众开放，但我们通过以下方式确保流程的透明与公开：

*   **实时反馈：** 我们通过 Webhook 实现了代码仓与流水线的联动。当您提交 PR 后，机器人会将运行日志自动同步至 PR 讨论区，您可以直接在评论区查看构建结果。
*   **逻辑公开：** 流水线的核心业务逻辑并非“黑盒”。所有构建脚本与配置代码均在 [Harmonybrew/ci](https://atomgit.com/Harmonybrew/ci) 仓库开源，任何人都可以审计或参考我们的 CI 流程。

<div id="join-the-maintenance-team"></div>

## 如何加入维护团队？

目前项目的协作机制尚在完善中，暂无正式的增员计划。

但开源的魅力在于，**无需“管理员”身份也一样能参与核心建设**。相比于权限，维护团队更多承担的是评审 PR、例行巡检等繁琐的责任。我们非常欢迎您先以贡献者的身份提交 PR，当深度参与成为常态时，加入团队便是水到渠成的事。

<div id="compatibility-with-older-system"></div>

## 对旧版本系统的兼容性

与上游 Homebrew 针对多个 macOS 版本（如 macOS 14、macOS 15）分别构建制品的做法不同，Harmonybrew 目前采取 **单版本、向前滚动** 的维护策略：

*   **唯一版本制品**：我们为每个软件包仅构建一个 `ohos` 版本的 Bottle 制品，暂不区分具体的系统小版本（如 OpenHarmony 6.1、OpenHarmony 7.0 等）。

*   **激进的版本跟随策略**：由于 OpenHarmony 目前仍处于快速迭代期，本项目将始终跟随官方最新的正式版本。当新版本系统发布且鸿蒙 PC 开始大规模推送更新后，我们会同步升级工具链与流水线执行机，并重新构建所有软件包。

*   **兼容性边界**：在迭代期内，我们的工作重心是确保软件在最新系统上的可用性。一旦环境完成迁移，我们将不再关注软件包在旧版本系统上的兼容情况或运行表现。

这种“向前看”的策略能够让我们集中精力解决新系统带来的技术挑战，确保包管理器始终与鸿蒙生态的最新特性保持同步。

<div id="provide-commercial-support"></div>

## 如何提供商用支持？

Harmonybrew 作为一个由社区发起的开源项目，在商用支持与责任边界上遵循以下原则：

*   **免责声明与许可证**：本项目对标上游 Homebrew 的运作模式，采用 BSD 2-Clause License 开源许可证。项目提供的所有代码与软件包均按“现状”（As-is）提供，不提供任何形式的明示或暗示保证，包括但不限于对商用结果、特定用途适用性或系统稳定性的保障。

*   **非营利与社区驱动模式**：我们是一个纯粹的非营利性开源社区，不从事任何商业经营活动。项目的发展由社区兴趣与共识驱动，**不承接任何来自企业端的定向定制需求**。若您的业务场景需要特定的软件包支持，欢迎通过提交 Pull Request 的方式贡献至社区仓库，我们将一视同仁地进行技术评审。

*   **关于企业“可信”诉求**：本项目不为任何企业的“可信软件”、“可信构建”等内部合规诉求背书。由于开源项目的公开性质，我们无法满足特定企业私有的安全合规审计要求。对于有极高合规性或内网运行要求的企业，我们建议您 Fork 本项目并在企业内网独立运营。您可以基于我们的开源底座，自行构建符合企业内部安全标准的软件源与流水线。

<div id="can-it-support-other-architectures"></div>

## 是否能支持其他架构？

目前 Harmonybrew 仅原生支持 arm64 架构，且暂无主动适配其他架构的计划。

若相关硬件厂商或社区组织有强烈意愿将特定架构（如 x86_64）融入本社区生态，我们持开放合作态度，但合作方需具备独立承担**工程适配**与**运行成本**的能力。

新增架构的准入需完成以下工作：

*   **构建环境适配**：
    *   **鸿蒙容器**：对 [DockerHarmony](https://github.com/hqzing/dockerharmony) 项目进行 fork 并迭代升级，使其能通过 `docker buildx` 构建多架构镜像（包含目标架构），将改动回馈至上游。
    *   **原生编译工具链**：提供可在目标架构原生运行的 ohos-sdk。出于构建质量考虑，我们的流水线仅接受原生编译，不接受交叉编译方案。
    *   **流水线镜像**：对 [Harmonybrew/ci](https://atomgit.com/Harmonybrew/ci) 项目进行 fork 并迭代升级，使其 ci-runner 能通过 `docker buildx` 构建多架构镜像（包含目标架构），将改动回馈至上游。
*   **流水线工程改造**：
    *   **脚本兼容性**：对 [Harmonybrew/ci](https://atomgit.com/Harmonybrew/ci) 项目进行 fork 并迭代升级，使流水线能够支持多架构 Bottle 的并行构建与分发，将改动回馈至上游。
*   **持续的资源投入**：
    *   **硬件与流量成本**：合作方需承担因新增架构产生的额外构建机开销及 CDN 下行流量成本。
*   **长期维护承诺**：
    *   **测试与排障**：合作方需指派专人维护 Formula 的架构兼容性。若某软件包在 arm64 上通过但在新增架构上失败，合作方有义务及时修复，严禁因架构差异长期阻塞社区 PR 的合入流程。

如果您能满足上述所有条件并希望发起合作，可通过邮件与我们联系。

<div id="contribute-to-upstream"></div>

## 什么时候能回馈上游？

Harmonybrew 参考了 [Linuxbrew](https://github.com/Linuxbrew) 的早期演进路径：即先通过独立 Fork 版本进行快速迭代与验证，待生态成熟后再寻求合入 Homebrew 上游。

然而，基于当前的客观技术环境与社区治理现状，**本项目在短期内暂无回馈上游的计划**。

主要的挑战在于以下三个层面：

*   **架构的侵入性变更**：

    为了优先适配 OpenHarmony 特有的底层限制，我们对 Homebrew 核心代码进行了一些非兼容性修改。这些变更目前尚未完全实现“平台解耦”，若直接回馈，可能会对 Homebrew 在 macOS 或标准 Linux 平台上的既有行为产生副作用。

*   **平台定义的路线抉择**：

    目前的工程实践将 OpenHarmony 视为一个“特殊的 Linux 发行版”以实现快速迁移。但在寻求上游合入时，必须确立明确的平台定义标准——是将 OpenHarmony 视为一个特殊的 Linux 发行版，还是视为一个完全独立的新平台？

    我们需要在这两条路线之间做出抉择：
    *   **方案 A（兼容路径）**：继续沿用 Linux 兼容路线（类似 Ruby 对 Android 的处理）。
    *   **方案 B（原生路径）**：将 OpenHarmony 定义为完全独立的全新平台架构（类似 Node.js 对 Android 的处理）。

    这两条路线在技术上均可行，但涉及重大的架构顶层设计决策，需与上游社区进行长期的技术论证与共识构建。

*   **开源治理与成本分担**：

    合入上游不仅是代码的合并，更涉及长期运维责任的分担：
    *   **流水线压力**：新增平台意味着上游社区需额外维护一套针对 OpenHarmony 的构建与测试流水线（CI/CD），这将显著增加其核心维护团队的工作负载。
    *   **基础设施成本**：多平台构建涉及算力资源等成本，目前尚无明确的资金支持方案来覆盖这部分由上游承担的额外开支。
    *   **构建环境合规性**：上游社区通常要求官方认可的、标准化的云化构建环境。目前我们使用的鸿蒙容器方案虽能满足功能需求，但该鸿蒙容器并非 OpenHarmony 官方提供，而是个第三方解决方案，在合规性与上游认可度方面仍需长期的沟通与背书。

总结：我们目前的重心在于 **“生态可用性”** 的打磨。回馈上游是一个涉及技术、资金、治理规则的复杂系统工程，我们将在 OpenHarmony 桌面生态进一步壮大、相关构建基础设施更趋标准化后，再适时启动与上游社区的正式磋商。
