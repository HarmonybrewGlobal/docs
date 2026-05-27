# Issue Reporting

## Prerequisites

Before submitting an issue, please be aware of the following:

1. **Non-commercial nature**: Harmonybrew is a completely free open-source project. You have not paid for this project and are not entitled to demand any immediate response or specific services from the developers.
2. **Disclaimer of Affiliation**: This project is not affiliated with Huawei products or part of their after-sales support. We have no affiliation with the OpenHarmony community or Huawei. Please resolve any after-sales issues regarding hardware devices or operating systems through official channels.

## Issue Categories

We categorize issues as follows:

1. **Package Manager Not Working Properly**: The package manager fails to function due to unavailability of cloud-side infrastructure (CDN, pipelines, etc.) or client-side infrastructure (Git, Ruby, etc.), or due to omissions in HarmonyOS adaptation.
2. **Packages in the software repository do not function properly**: Packages installed via the package manager do not function correctly, such as returning errors like `Permission denied` or `Operation not permitted`.
3. **Desired packages are missing from the software repository**: The package you wish to use has not yet been added to the repository.
4. **Documentation issues**: There are gaps in the process; following the documentation does not yield the expected results.
5. **Security issues**: Critical incidents such as key leaks or supply chain attacks.
6. **Other issues**: Issues not covered by the categories above.

## How to Provide Feedback

**1. Package manager not working properly**

- **Critical issues**: You can email the Harmonybrew maintenance team directly. See the [Community Guide](../community.md) for the email address.
- **Non-urgent issues**: Submit an issue to the [Harmonybrew/brew](https://atomgit.com/Harmonybrew/brew) repository, or submit a PR after fixing the issue.

<br>

**2. Packages in the software repository are not working properly**

This community allows the submission of such issues for community discussion, but **the maintenance team is not responsible for resolving them.** We only provide the platform capabilities; we are not responsible for adapting thousands of packages to HarmonyOS, nor do we have the authority to direct other communities or vendors to modify their operating system implementations.

Our package inclusion policy aligns with that of upstream Homebrew: as long as a package passes `brew test` in the pipeline, it is allowed to be included.

A package passing `brew test` means it can at least pass smoke tests in an “**OpenHarmony + non-app sandbox environment + ext4 filesystem**” (such as a development board or container environment). However, for the “**HarmonyOS + app sandbox environment + HMDFS filesystem**” (i.e., the HarmonyOS PC HiShell environment), we do not conduct dedicated testing and cannot guarantee compatibility.

> **Note**: Since vendors have not provided a cloud-based HarmonyOS system environment, developers cannot access cloud testing resources; therefore, no testing has been conducted. In contrast, Windows, Linux, and macOS are standardized base environments on mainstream CI/CD platforms (such as GitHub Actions) and offer mature runtime support for open-source projects.

We recommend that users with technical expertise identify and resolve these issues on their own:

- **Requires in-depth adaptation**: Please contribute patches or formulas to the [Harmonybrew/homebrew-core](https://atomgit.com/Harmonybrew/homebrew-core) repository.
- **System environment issues**: Please compile a minimal reproducible test case and report it to the relevant community (OpenHarmony) or vendor (HarmonyOS).

<br>

If you lack the technical capability to resolve the issue yourself, you may submit an issue to [Harmonybrew/homebrew-core](https://atomgit.com/Harm
