# How to Contribute a Formula

## 1 Introduction

Submitting packages to a system-level software repository is no easy task. In the industry, this work is typically carried out by power users or system developers, rather than ordinary users.

If your current development experience is still at the stage where you “can install packages but not submit them,” and you have never contributed packages to official Homebrew or mainstream Linux distributions (such as Debian, Arch Linux, etc.), we do not recommend that you attempt to submit contributions to Harmonybrew directly.

The process of adding packages to Harmonybrew is no simpler than that of other software repositories. On the contrary, due to the unique nature of the operating system environment and the current state of the infrastructure, the adaptation process presents even greater challenges. If you are unable to independently complete contributions on mature platforms, you will find it even more difficult here.

If you believe you are ready to take on these challenges, then come on in. We welcome you to embark on a pioneering journey to expand the OpenHarmony command-line ecosystem.

## 2 Prerequisite Knowledge

Before contributing a formula, please make sure to review the following resources:

<table>
  <thead>
    <tr>
      <th>Resources</th>
      <th>Introduction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://docs.brew.sh/">Homebrew Documentation</a></td>
      <td>
        Homebrew Official Documentation. Before you “run,” learn to “walk” first. Be sure to become a
        power user of Homebrew and thoroughly understand its underlying mechanisms.
      </td>
    </tr>
    <tr>
      <td>
        <div style="min-width: max-content; display: block; white-space: nowrap;">
          <a href="https://docs.brew.sh/Adding-Software-to-Homebrew">Adding-Software-to-Homebrew</a>
        </div>
      </td>
      <td>
        Homebrew Official Contribution Guide. Harmonybrew’s contribution process aligns as closely as possible with the official upstream process.
      </td>
    </tr>
    <tr>
      <td><a href="../user/FAQ.md">Frequently Asked Questions</a></td>
      <td>
        A collection of frequently asked questions. It covers details about the Harmonybrew project and knowledge related to the OpenHarmony
        platform.
      </td>
    </tr>
  </tbody>
</table>

## 3 Environment Setup

### 3.1 Setting Up the Docker Environment

To resolve the extremely complex toolchain dependencies during formula builds and ensure your code passes the pipeline build successfully after submission, all Harmonybrew development, debugging, and build work **must be performed within a unified Docker container.**

This “containerized development” model offers the following advantages:

- **Environment Alignment**: Your local environment is 100% consistent with the pipeline environment; “it runs on my machine” means “it runs on the pipeline.”
- **Tool Integration**: Development tools such as ohos-sdk and make are pre-installed in the container, eliminating the need to manually configure complex build environments.
- **Lossless rollback**: All experimental changes are made within the container, so they won’t pollute your host system.

<br>

Therefore, you should first

Translated with DeepL.com (free version)
