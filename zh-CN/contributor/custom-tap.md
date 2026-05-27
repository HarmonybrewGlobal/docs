# Distributing Packages via Custom Taps

## Introduction

The admission criteria for the Harmonybrew core tap (Harmonybrew/homebrew-core) are quite strict. For details, please refer to [How to Contribute a Formula](./contribute-formula.md). For software that does not meet the core tap’s admission rules (such as closed-source software), we recommend distributing it via a [custom tap](https://docs.brew.sh/How-to-Create-and-Maintain-a-Tap).

This distribution method still leverages Harmonybrew’s package manager ecosystem. Users only need to execute an additional command to add the custom tap when downloading packages, which has minimal impact on the user experience.

This document will use the AtomGit platform as an example to demonstrate how to set up a custom tap based on this platform and distribute a “closed-source software” named `busybox`.

> The reason “closed-source software” is in double quotes: This software is actually open-source, and the source code is available [here](https://busybox.net/downloads/). For the purposes of this demonstration, we will download pre-built artifacts directly from the Releases page of the [Harmonybrew/ohos-busybox](https://github.com/Harmonybrew/ohos-busybox) repository and distribute them, assuming it is a closed-source software.

## Prerequisites

Maintaining a custom tap is an advanced operation. Before proceeding, please ensure you have mastered the basics of contributing formulas. If you need guidance, please refer to [How to Contribute a Formula](./contribute-formula.md).

## Preparing the Pre-built Package

Using the distribution of the “proprietary software” `busybox` as an example, assume the pre-built package is named `busybox-1.37.0-ohos-arm64.tar.gz`.

Its directory structure after extraction is as follows:

```text
busybox-1.37.0-ohos-arm64
├── AUTHORS
├── bin
│   └── busybox
└── LICENSE
```

## Create a Git Repository

Each tap corresponds to a Git repository; you must create a new repository on a code hosting platform to serve as the container for the tap.

This example uses the AtomGit platform, with the repository URL: `https://atomgit.com/Harmonybrew/custom-tap-example`

## Upload the Pre-built Package

Since the pre-built package must be downloaded during the formula build process, it must be uploaded to a publicly accessible storage location.

Example: You can use the “Releases” feature of the AtomGit repository to store the pre-built package. In this scenario, create a release named `busybox-1.37.0` under the `https://atomgit.com/Harmonybrew/custom-tap-example` repository and upload the pre-built package to it.

After the upload is complete, record the public download link: `https://atomgit.com/Harmonybrew/custom-tap-example/releases/download/busybox-1.37.0/busybox-1.37.0-ohos-arm64.tar.gz`

## Create and Upload the Formula

Clone the Git repository to your local machine, create a `Formula` directory within the repository, organize it into subdirectories by first letter, and finally create the formula file within that subdirectory.

**Steps:**

```sh
git clone git@atomgit.com:Harmonybrew/custo
