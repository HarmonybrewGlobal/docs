# 特色软件

Harmonybrew 本该只负责包管理，但为了让大家能顺利地在 OpenHarmony 设备上写起代码，我们做了一些“分外之事”。

针对系统环境的诸多限制（如代码签名、平台标识等），我们专门维护了一系列特色软件包，优化了 C/C++ 与 Python 等场景的开发体验。

我们替你填平了这些底层的“坑”，只为让 OpenHarmony 上的操作逻辑能重新对齐你熟悉的 Linux 或 macOS。

## 软件包列表

<table>
  <thead>
    <tr>
      <th>名称</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ohos-sdk</td>
      <td>
        我们将 ohos-sdk 也放在软件仓库中分发，用户安装 ohos-sdk 后即可使用
        clang、llvm-ar 等命令。另外，我们对 ohos-sdk 里面的 lld
        链接器做了简单的脚本封装，默认启用了
        <a
          href="https://atomgit.com/openharmony/third_party_llvm-project/pull/882"
          >链接器签名</a
        >，使得它编出来的程序可以直接在鸿蒙 PC 上运行，无需手动进行代码签名。
      </td>
    </tr>
    <tr>
      <td><div style="min-width: max-content; display: block; white-space: nowrap;">llvm-gcc-compat</div></td>
      <td>
        这个包会生成 cc、gcc、ld 等软链接，全部指向 ohos-sdk 里面的 LLVM
        工具链。这使我们可以更顺利地编译各种开源软件，无需手动指定 CC、CXX
        等环境变量。此做法是模仿 macOS 的做法，macOS 中的 cc、gcc、ld
        等命令也是指向 LLVM 的软链接。
      </td>
    </tr>
    <tr>
      <td>devel-base</td>
      <td>
        类似于 Debian 里面的 build-essential。这个包本身没有任何内容，只是将
        ohos-sdk、llvm-gcc-compat、make、coreutils
        等软件包设置成了级联依赖。只要装了这个包，这些级联依赖就会被装进来，你就拥有了一个较为完善的
        C/C++ 编译环境。命名成 devel-base 是为了避免跟各大 Linux
        发行版里面的同类软件包重名。
      </td>
    </tr>
    <tr>
      <td>uname-is-linux</td>
      <td>
        一个专为鸿蒙 PC 设计的系统标识伪装工具。它通过劫持 libc 中的
        <code>uname()</code> 函数，使应用程序获取的系统标识始终为
        <code>Linux</code>。详细用法请查看
        <a href="https://atomgit.com/Harmonybrew/uname-is-linux">源码仓</a> 里面的
        README 文档。
      </td>
    </tr>
    <tr>
      <td><div style="min-width: max-content; display: block; white-space: nowrap;">ohos-pip-autosign</div></td>
      <td>
        自动对 pip 包里面的 .so 文件进行代码签名。详细用法请查看
        <a href="https://atomgit.com/Harmonybrew/ohos-pip-autosign">源码仓</a> 里面的
        README 文档。
      </td>
    </tr>
  </tbody>
</table>

## 场景演示

### 场景一：编译 C/C++ 程序

在鸿蒙 PC 的 HiShell 环境中编译开源软件 gzip：

```sh
# 安装编译工具链和环境伪装工具
brew install devel-base uname-is-linux

# HiShell 环境中，TMPDIR 环境变量的默认值是 /storage/Users/currentUser，这个目录承载在 HMDFS 上
# 修改这个环境变量，将其指向一个非 HMDFS 的目录，避免因 HMDFS 文件系统缺陷导致编译失败
export TMPDIR=/data/storage/el2/base/files/tmp
mkdir -p $TMPDIR

# 使用非 HMDFS 目录作为工作目录，避免因 HMDFS 文件系统缺陷导致编译失败
WORKDIR=/data/storage/el2/base/files/work
mkdir -p $WORKDIR
cd $WORKDIR

# gzip 安装目录，安装到 currentUser 下，方便我们使用它
PREFIX=/storage/Users/currentUser/gzip-1.14-ohos-arm64

# 启用全局环境伪装
export LD_PRELOAD=$(brew --prefix)/opt/uname-is-linux/lib/libuname.so

# 编译 gzip
curl -fLO https://ftp.gnu.org/gnu/gzip/gzip-1.14.tar.gz
tar -zxf gzip-1.14.tar.gz
cd gzip-1.14
./configure --prefix=$PREFIX  # 无需再显式指定目标平台（如 `--host=aarch64-linux` 等）
make -j$(nproc)
make install
cd ..

# 编译完成，取消环境伪装
unset LD_PRELOAD

# 验证编出来的 gzip 是否能正常运行
$PREFIX/bin/gzip --help

# 清理临时目录和工作目录
sh -c "rm -rf $TMPDIR/* $WORKDIR/*"
```

### 场景二：编译 Rust 程序

用法1：安装 llvm-gcc-compat，让 llvm-gcc-compat 提供 cc 命令供 rustc 使用。用户无需再额外通过 config.toml 来配置 linker 路径。

在不配置任何 config.toml 的情况下，rustc 默认会调用 cc 命令作为 linker。

这个 cc 命令实际上是个软链接，指向了 ohos-sdk 里面的 clang 驱动器。

clang 驱动器在进行链接操作时，调用的是经过我们封装的 ld.lld 脚本，因此最终 rustc 编译出来的二进制都会默认带有代码签名，可直接在鸿蒙 PC 上运行。

```sh
# 安装 rust 和 llvm-gcc-compat（ohos-sdk 会作为 llvm-gcc-compat 的级联依赖被自动引入）
brew install rust llvm-gcc-compat

# 创建一个简单的 cargo 工程并将其编译成二进制
cargo new hello_project
cd hello_project
cat > src/main.rs << 'EOF'
fn main() {
    println!("Hello, world!");
}
EOF
cargo build --release

# 测试编译产物是否能正常运行
./target/release/hello_project
```

用法 2：不安装 llvm-gcc-compat，仅安装 ohos-sdk。用户需要额外配置 config.toml 指定使用 ohos-sdk 里面的 clang 驱动器作为 linker。

clang 驱动器在进行链接操作时，调用的是经过我们封装的 ld.lld 脚本，因此最终 rustc 编译出来的二进制都会默认带有代码签名，可直接在鸿蒙 PC 上运行。

```sh
# 仅安装 rust 和 ohos-sdk，不安装 llvm-gcc-compat
brew install rust ohos-sdk

# 编写一个用户级的 config.toml
mkdir -p ~/.cargo/
cat > ~/.cargo/config.toml << 'EOF'
[target.aarch64-unknown-linux-ohos]
ar = "/storage/Users/currentUser/.harmonybrew/opt/ohos-sdk/native/llvm/bin/llvm-ar"
linker = "/storage/Users/currentUser/.harmonybrew/opt/ohos-sdk/native/llvm/bin/clang"
EOF

# 创建一个简单的 cargo 工程并将其编译成二进制
cargo new hello_project
cd hello_project
cat > src/main.rs << 'EOF'
fn main() {
    println!("Hello, world!");
}
EOF
cargo build --release

# 测试编译产物是否能正常运行
./target/release/hello_project
```

### 场景三：安装 Python 三方库

在鸿蒙 PC 的 HiShell 环境中安装 numpy：

```sh
# 安装 Python 运行时和自动签名工具
brew install python ohos-pip-autosign

# 激活 venv
python3 -m venv .venv
source .venv/bin/activate

# 激活自动签名工具
ohos-pip-autosign activate

# 安装 numpy
pip install numpy

# 编写一段 python 脚本，验证 numpy 是否能正常工作
cat <<EOF > test.py
import numpy as np
arr = np.array([1, 2, 3])
print(arr * 2)
EOF

# 执行脚本
python3 test.py
```
