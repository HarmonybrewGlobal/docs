# Harmonybrew

The Harmonybrew project is an initiative to port Homebrew to the OpenHarmony platform, providing HarmonyOS users with an out-of-the-box package manager and accompanying software repositories. It supports various types of HarmonyOS devices, including HarmonyOS PCs, HarmonyOS development boards, and HarmonyOS containers.

## Device Support

<table>
  <thead>
    <tr>
      <th>Device Type</th>
      <th>Representative Products</th>
      <th>Minimum System Version</th>
      <th>Command-Line Environment</th>
      <th>Architecture</th>
      <th>Support Level</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>HarmonyOS PC</td>
      <td>HUAWEI MateBook Pro</td>
      <td>HarmonyOS 6.1.0.117 SP68</td>
      <td>HiShell</td>
      <td>arm64</td>
      <td>Tier 2 (Low)</td>
    </tr>
    <tr>
      <td>HarmonyOS Development Board</td>
      <td>dayu200(rk3568)</td>
      <td>OpenHarmony 6.1</td>
      <td>hdc shell</td>
      <td>arm64</td>
      <td>Tier 1 (High)</td>
    </tr>
    <tr>
      <td>HarmonyOS Container</td>
      <td>
        <a href="https://github.com/hqzing/dockerharmony">DockerHarmony</a>
      </td>
      <td>OpenHarmony 6.1</td>
      <td>Any</td>
      <td>arm64</td>
      <td>Tier 1 (High)</td>
    </tr>
  </tbody>
</table>

> **💡 About Platform Compatibility**
>
> As the commercial distribution of OpenHarmony, HarmonyOS can theoretically inherit its ecosystem, so this project can also run on HarmonyOS.
>
> However, please note: the ability to run does not imply full support. Some software packages function normally on development boards (hdc shell) or in containers, but may be restricted in the HiShell environment on HarmonyOS PC due to factors such as **system security restrictions**. This is a native system-level limitation and not an issue with the project itself.

## Installation Guide

- [HarmonyOS PC](./zh-CN/user/install.md#Harmony-PC)
- [HarmonyOS Development Board](./zh-CN/user/install.md#HarmonyOS-Development-Board)
- [HarmonyOS Container](./zh-CN/user/install.md#HarmonyOS-Container)

## Common Operations

The following are some common Homebrew operations. For more information, please refer to the [Homebrew Official Documentation](https://docs.brew.sh/)

```sh
zsh -c “$(curl -fsSL https://harmonybrew.atomgit.com/install.sh)”        # Install HarmonyOS version of Homebrew
zsh -c “$(curl -fsSL https://harmonybrew.atomgit.com/uninstall.sh)” 

Translated with DeepL.com (free version)
