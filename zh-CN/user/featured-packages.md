# Featured Software

Harmonybrew was originally intended solely for package management, but to help everyone get started with coding on OpenHarmony devices, we’ve gone the extra mile.

To address various system constraints (such as code signing and platform identification), we’ve curated a series of specialized software packages that optimize the development experience for C/C++, Rust, Python, and other scenarios.

We’ve smoothed out these low-level “hurdles” for you, all to ensure that the operational logic on OpenHarmony aligns with the Linux or macOS environments you’re familiar with.

## Package List

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ohos-sdk</td>
      <td>
        We also distribute ohos-sdk via the software repository; after installing ohos-sdk, users can use
        commands such as clang and llvm-ar. Additionally, we have created a simple script wrapper for the lld
        linker included in ohos-sdk, which enables
        <a
          href="https://atomgit.com/openharmony/third_party_llvm-project/pull/882"
          >linker signing</a
        > is enabled by default, allowing programs compiled with it to run directly on HarmonyOS PCs without the need for manual code signing.
      </td>
    </tr>
    <tr>
      <td><div style="min-width: max-content; display: block; white-space: nowrap;">llvm-gcc-compat</div></td>
      <td>
        This package generates symbolic links for cc, gcc, ld, and others, all pointing to the LLVM
        toolchain within the ohos-sdk. This allows us to compile various open-source software more smoothly, without having to manually specify environment variables such as CC and CXX
        and other environment variables. This approach mimics macOS, where commands like cc, gcc, and ld
        are also symbolic links pointing to LLVM.
      </td>
    </tr>
    <tr>
      <td>devel-base</td>
      <td>
        Similar to build-essential in Debian. This package contains no content itself; it simply sets
        ohos-sdk, llvm-gcc-compat, make, coreutils
        and other packages as cascading dependencies. Once this package is installed, these dependencies are automatically installed, providing you with a relatively complete
        C/C++ compilation environment. It is named devel-base to avoid naming conflicts with similar packages in major Linux
        distributions.
      </td>
    </tr>
    <tr>
      <td>uname-is-linux</td>
      <td>
        A system identifier spoofing tool designed specifically for HarmonyOS PC. It hijacks the
        <code>uname()</code> function in libc, ensuring that the system identifier returned to applications is always
        <code>Linux</code>. For detailed usage instructions, please refer to
        <a href="https:/
