# Harmonybrew

Harmonybrew 项目是一个将 Homebrew 移植到 OpenHarmony 平台的项目，为鸿蒙用户提供开箱即用的包管理器以及配套的软件仓库，支持运行在鸿蒙 PC、鸿蒙开发板、鸿蒙容器等不同形态的鸿蒙设备上。

## 设备支持

<table>
  <thead>
    <tr>
      <th>设备形态</th>
      <th>代表产品</th>
      <th>最低系统版本</th>
      <th>命令行环境</th>
      <th>架构</th>
      <th>支持等级</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>鸿蒙 PC</td>
      <td>HUAWEI MateBook Pro</td>
      <td>HarmonyOS 6.1.0.117 SP68</td>
      <td>HiShell</td>
      <td>arm64</td>
      <td>Tier 2（低）</td>
    </tr>
    <tr>
      <td>鸿蒙开发板</td>
      <td>dayu200(rk3568)</td>
      <td>OpenHarmony 6.1</td>
      <td>hdc shell</td>
      <td>arm64</td>
      <td>Tier 1（高）</td>
    </tr>
    <tr>
      <td>鸿蒙容器</td>
      <td>
        <a href="https://github.com/hqzing/dockerharmony">DockerHarmony</a>
      </td>
      <td>OpenHarmony 6.1</td>
      <td>任意</td>
      <td>arm64</td>
      <td>Tier 1（高）</td>
    </tr>
  </tbody>
</table>

> **💡 关于平台兼容性**
>
> HarmonyOS 作为 OpenHarmony 的商业发行版，理论上可以继承其生态，因此本项目也能在 HarmonyOS 上运行。
>
> 但请注意：能运行不代表完美支持。部分软件包在开发板（hdc shell）或容器中表现正常，但在鸿蒙 PC 的 HiShell 环境下，可能会因**系统安全限制**等因素受限。这属于系统级原生限制，并非项目本身的问题。

## 安装指南

- [鸿蒙 PC](./zh-CN/user/install.md#鸿蒙-pc)
- [鸿蒙开发板](./zh-CN/user/install.md#鸿蒙开发板)
- [鸿蒙容器](./zh-CN/user/install.md#鸿蒙容器)

## 常用操作

以下是一些常用的 Homebrew 操作，更多资料请查看 [Homebrew 官方文档](https://docs.brew.sh/)

```sh
zsh -c "$(curl -fsSL https://harmonybrew.atomgit.com/install.sh)"        # 安装鸿蒙版 Homebrew
zsh -c "$(curl -fsSL https://harmonybrew.atomgit.com/uninstall.sh)"      # 卸载鸿蒙版 Homebrew
brew update                 # 更新 Homebrew 包管理器和包索引
brew formulae               # 列出软件仓库中可用的软件包列表
brew search [keyword]       # 在软件仓库中通过关键词搜索软件包
brew install [formula]      # 安装软件包
brew uninstall [formula]    # 卸载软件包
brew list                   # 查看已安装的软件包列表
rm -rf $(brew --cache)      # 清除缓存
rm -rf /storage/Users/currentUser/.harmonybrew   # 彻底删除 Homebrew 安装目录（比卸载脚本删得更干净）
```

> **注意**：Homebrew 是一套“滚动更新”模式的包管理系统，它对软件包版本的维护策略并不同于 Ubuntu 里面的 apt 或 Red Hat 里面的 yum。如果你未使用过 Homebrew 或其他滚动更新包管理系统，请先知悉它们的特点，以免使用时造成困惑。

## 其他文档

- [贡献指南](./zh-CN/contributor/contributing.md)
- [问题反馈](./zh-CN/user/issues.md)
- [社区说明](./zh-CN/community.md)
- [常见问题](./zh-CN/user/FAQ.md)
- [特色软件](./zh-CN/user/featured-packages.md)
