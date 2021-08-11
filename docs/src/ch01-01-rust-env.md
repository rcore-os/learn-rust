# 安装和配置Rust开发环境

这一节将告诉你如何安装和更新Rust。

Rust安装上手非常容易，但是有些细节会因为你的目标平台而不同。Rust可以在Windows、Linux和macOS上运行。此外，你可能希望交叉编译代码以便在其他平台上使用。

## 使用Rustup

安装Rust最简单的方法就是下载并运行 `rustup-init`，你可以在[Rustup的官网](https://www.rustup.rs/)下载到它。

Windows系统和类Unix操作系统中的安装流程是不同的：

* Windows系统的 `rustup-init` 是一个exe安装包。
* Unix / macOS / Linux系统的 `rustup-init` 是一个shell脚本。

不管哪一种，只要按照指示走，安装程序都会下载并安装rustc、cargo和rustup。安装路径在类Unix系统中是 `~/.cargo/bin` ，在Windows系统中是 `%USERPROFILE%.cargo.\bin`。

安装程序还会设置 `PATH` 环境变量以便于运行rustc、cargo和rustup。

当 `rustup` 安装完成后，你就可以用它来做很多事情了：

* 安装额外的Rust工具链（如果你在交叉编译，或者希望支持多个平台，那么你可能需要不止一个工具链）
* 更改 `rustc` 和 `cargo` 使用的默认工具链。Rustup会创建软链接和脚本来确保它们调用了正确的工具链
* 在Rust版本更新时，升级工具链
* 获得源代码和文档

### Unix / Linux系统

运行 `rustup-init.sh`的步骤如下：

1. 打开命令行环境
2. 输入 `curl https://sh.rustup.rs -sSf | sh`
3. 以上命令会下载并且运行 `rustup-init.sh` 脚本，这个脚本将会检查你的系统环境，向你推荐下载的工具链版本，而且可以修改你的 `PATH` 环境变量。
4. 按1继续（如果想要修改就自行修改）
5. 等待下载完成
6. 大功告成

如果你没有安装 `curl`，你可以先安装它，或者从浏览器下载[脚本](https://sh.rustup.rs)到硬盘然后执行

在Linux系统中安装 `curl`，你可以这样做：

* Debian / Ubuntu - `sudo apt get curl`
* Fedora / Redhat - `sudo dnf install curl`

### Windows系统

1. 从 https://rustup.rs 下载 rustup-init.exe
2. 双击安装包，将会出现一个命令行环境。
3. 按1继续（如果想要修改就自行修改）
4. 等待下载完成
5. 大功告成

如果不想采用默认设置，可以参考以下几个建议：

1. 32/64位版本，当前大多数Windows系统都是64位的，但也可能有特殊的安装32位版本的理由。
2. GNU/MSVC ABI（应用二进制接口），这取决于你希望兼容的工具链和运行时：

* 如果你不关心动态库链接，或者你希望链接 MingW / MSYS 生成的DLL，请选择GNU ABI。
* 如果你安装了Visual Studio，或者你希望在Rust程序中调用Visual Studio生成的DLL，那应当选择MSVC ABI。这一选择的额外好处在于编译器会生成.pdb文件，因此你可以在Visual Studio中调试Rust。

### 保持Rust更新

Rust的新版本发布节奏大致介于频繁和偶尔之间。想升级Rust到最新版本，只需要这样：

```
rustup update
```

有时候 `rustup` 自身需要更新，此时只需要这样：

```
rustup self update
```

### 添加Rust源代码

Rustup会安装Rust工具链，但如果你要写代码或者调试Rust，你可能需要获取Rust的源代码以便于探究内部实现：

```
rustup component add rust-src
```

## 手动安装

如果你喜欢手动安装，在[Rust官网](https://forge.rust-lang.org/infra/other-installation-methods.html)上有安装包和流程。

但请注意，Rust的发布周期还是相对比较短的，所以只有在你计划一直停在一个版本的情况下我们才推荐这样做，否则你会被迫每6周卸载再安装一次Rust。

## 设置调试器


### Unix / Linux

调试Rust和调试C/C++几乎完全相同。

首先，你需要安装gdb，然后你可以从命令行或者你喜欢的调试器前端调用它来调试Rust代码。

在Linux系统下，gdb可以这样安装：

* Debian / Ubuntu - `sudo apt get gdb`
* Fedora / Redhat - `sudo dnf install gdb`

如果你想用lldb（Rust的编译器后端LLVM的姊妹项目），可以参考[lldb的网站](http://lldb.llvm.org/)。

Rust还提供了一些包装gdb和lldb的脚本，从而可以在调试时启用美观打印。你可以调用 `rust-gdb` 或者 `rust-lldb` 来启用这个功能。

### Windows

如果安装时选择了MSVC ABI，那么可以用Visual Studio来调试Rust代码。在以debug模式构建代码时，编译器会额外创建一个.pdb文件。这样你就可以在Visual Studio中执行你的二进制程序，然后进行单步执行、变量查看等调试操作。

#### GDB

Windows系统中，GDB可以通过 MSYS 或者 MingW 来获取。

以MSYS为例，TDM-GCC的发行版可以从[这里](http://tdm-gcc.tdragon.net/download)获取。截至译稿时，你可以选择TDM-GCC 10.3.0、9.2.0等多个版本，并根据你的Rust安装选择32/64位编译器。我们以`gdb-7.9.1-tdm64-2.zip`为例。

将压缩包提取到一个路径下，如`C:\tools\gdb-7.9.1-tdm64-2`，然后将它添加到 `PATH` 环境变量：

```
set PATH=%PATH%;C:\tools\gdb-7.9.1-tdm64-2\bin\
```

（译注：也可以在右击我的电脑-高级-编辑环境变量，在GUI中修改）

这样你就可以从命令行使用 `gdb` 了，但一般情况下我们更喜欢加一个编译器前端。

截至写稿时，支持最好的编辑器是Visual Studio Code，它有GDB调试和Rust的插件，因此你可以在同一个IDE中编写和调试代码。

##### 启用美观打印

Rust支持在GDB中变量检查时启用美观打印。美观打印器是一个Python脚本，GDB会调用它来显示变量。

首先保证环境变量中有Python 2.7。

如果你用的是 `rustup` 安装，那么可以在 `%USERPROFILE%\.rustup` 目录下找到它：

e.g.

```
C:\users\MyName\.rustup\toolchains\stable-x86_64-pc-windows-gnu\lib\rustlib\src\rust\src\etc
```

否则它在你解压Rust源代码的路径下 `src\rust\src\etc`.

记下它所在的绝对路径，然后编辑 `C:\tools\gdb-7.9.1-tdm64-2\bin\gdbinit` 来将它加入path。注意，请用*正*斜杠（也就是`/`）

```
python
print "---- Loading Rust pretty-printers ----"
 
sys.path.insert(0, "C:/users/MyName/.rustup/toolchains/stable-x86_64-pc-windows-gnu/lib/rustlib/src/rust/src/etc")
import gdb_rust_pretty_printing
gdb_rust_pretty_printing.register_printers(gdb)
 
end
```

## 设置集成开发环境

在IDE支持上，Rust相比其他语言还稍显落后，但已经有很多编辑器/IDE支持以插件方式提供开发Rust所需的功能了。

Eclipse、IntelliJ、Visual Studio等流行IDE都有Rust集成的插件。

* [Visual Studio Code](https://code.visualstudio.com/) (别和Visual Studio搞混) 是一个跨平台的编辑器，而且提供大量的插件。[这个教程]((https://sherryummen.in/2016/09/02/debugging-rust-on-windows-using-visual-studio-code/)展示了如何将Visual Studio Code配置为完整的Rust开发环境。（译注：这个链接已失效，后续将补充Visual Studio Code的Rust开发环境配置）
* [IntelliJ IDEA的Rust插件](https://intellij-rust.github.io/)正在积极开发中。这个插件有很强的吸引力，基本上每周发布一个新版本。它提供语法高亮、自动补全、cargo构建集成，未来还会提供其他功能。[IntelliJ IDEA](https://www.jetbrains.com/idea/download/#section=windows)是商业软件，但免费的社区版也足够用来开发。
* [Visual Rust plugin for Microsoft Studio](https://github.com/PistonDevelopers/VisualRust) . 提供语法高亮、自动补全和交互式调试。
* [RustDT for Eclipse](https://github.com/RustDT/RustDT) 也在积极开发中。它有语法高亮、自动补全（通过Racer实现）、cargo构建集成和rustfmt功能。
* Atom 是一个有丰富插件的流行编辑器。这些插件对于开发Rust非常有用：
  * [language-rust](https://atom.io/packages/language-rust)提供了基本语法高亮
  * [racer](https://atom.io/packages/racer)提供自动补全
  * [atom-beautify](https://atom.io/packages/atom-beautify)调用rustfmt来美化代码
  * [build-cargo](https://atom.io/packages/build-cargo)调用cargo来实现错误和警告的内联显示。

对于其他编辑器和IDE，可以参考[Rust and IDEs](https://forge.rust-lang.org/ides.html)。

（译注：根据Jetbrains的2021开发者调查，在Rust开发者中最受欢迎的IDE是Visual Studio Code，超过了Jetbrains自家的CLion和IntelliJ IDEA，因此我们推荐使用Visual Studio Code来开发。）

## Racer / Rustfmt

上面提到的一些插件使用了 Racer 或者 Rustfmt，其中Racer用于自动补全，Rustfmt用于代码格式化。只需要在命令行中执行以下命令就可以安装 racer 和 rustfmt：

```
cargo install racer
cargo install rustfmt
```
