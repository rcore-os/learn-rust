# 第一个 HelloWorld 程序



*TODO: 以下是 Compiling and Linking in More Detail 的翻译，可能需要改进内容安排。*

## 程序的入口

Rust 和 C/C++ 一样，有一个主函数，通常是 `main()`.[^1]

不像 C/C++（返回 `int`，可能有参数 `int argc` 和 `char *argv[]`），Rust 的主函数不接受任何参数，也没有返回值。那么，怎么处理命令行参数呢？

### 处理命令行参数

回顾一下 C/C++，入口函数 `main` 接受两个参数：`argc` 和 `argv`. `argc` 是命令行参数的数量，`argv` 是一个 `char*` 数组，存储那些命令行参数。

```c++
int main(int argc, char *argv[]) {
  // ...
}
```

在 Rust 中，我们可以使用 `std::env::args()` 在任何地方访问命令行参数，而不仅限于主函数中）。这里的语法和 C++ 很像，`std` 是标准库内容所在的命名空间，`env` 命名空间的内容是标准库的一个组件，`args()` 函数就在其中。

`args()` 返回命令行参数构成的字符串数组。通常来说，第一个参数是执行的命令本身（但未必，参见[标准库文档](https://doc.rust-lang.org/std/env/fn.args.html)）。

```rust
use std::env;

fn main() {
  for argument in env::args() {
    println!("{}", argument);
  }
}
```

- 第一行，这和 C++ 中的 `using` 用法接近，把其它命名空间的符号引入进来。
- Rust 中的 `for` 循环比较接近 C++11 引入的 [range-for](https://zh.cppreference.com/w/cpp/language/range-for).
- `println!` 的用法比较接近 C++ 中的 `std::printf`，第一个参数是格式字符串，但是格式化语法是 Python 风格的（参见[标准库文档](https://doc.rust-lang.org/std/fmt/index.html#usage)）。

由于 `args()` 的返回值类型是 `Args`，这是一个迭代器（在 Rust 中，我们说其实现了 `Iterator` trait），我们可以将 `Args` 包含的元素 collect 到其它容器中：

```rust
use std::env;
use std::collections::HashSet;

fn main() {
  let args: HashSet<String> = env::args().collect();
  let verbose_flag = args.contains("--verbose");
}
```

- 如果你学过 TypeScript，会对 Rust 中的变量定义方式比较熟悉：

  ```rust
  let variable: Type = ...;
  ```

  `verbose_flag` 的定义说明，类型注解是可以省略的。但是 `args` 这一行中，`collect()` 函数是一个泛型函数（类似 C++ 中的函数模板），我们需要指定 `args` 的类型，否则编译器无法推断出 `collect()` 该返回什么类型。这一点和 C++ 不同，C++ 的模板参数推导不包括返回值类型。

- 即使这是你第一次学习 Rust，上面的代码也不难看懂。唯一需要说明的是 `collect()` 函数，你可以简单理解为使用 `for` 循环遍历迭代器，把其中的元素放入新的容器（这里就是 `args`，一个装有 `String` 的哈希表）。

同 C/C++ 的 `int argc, char *argv[]` 比较，Rust 这种风格更有好处：

- `argc` 和 `argv` 逻辑上是一体的。在 Rust 中，`Args` 类型既保有命令行参数，也记录了参数个数（数组长度），这一点在其它地方也会有体现。
- `args()` 函数可以在任何地方访问到，因此我们不需要把命令行参数传来传去了。

#### 使用第三方 crate

*crate* 意为一个 Rust 代码包，和 Python 的模块差不多。在 Rust 中引入第三方库比 C/C++ 容易得多，下面我们简单学习一下 Rust 中最流行的命令行参数处理库 [clap](https://crates.io/crates/clap).

clap 提供了一种描述式（descriptive）、声明式（declarative）的方式向代码中添加规则。当你的程序需要接受很多参数并且要进行错误检查时，这大为方便。

*TODO: 需要介绍一下 cargo 的使用。*

首先，在 `Cargo.toml` 中加入：

```toml
[dependencies]
clap = "2.33"
```

而后将 `main.rs` 修改为：

```rust
use clap::*;

fn main() {
  let matches = App::new("Sample App")
    .author("My Name <myname@foocorp.com>")
    .about("Sample application")
    .arg(Arg::with_name("T")
        .long("timetowait")
        .help("Waits some period of time for something to happen")
        .default_value("10")
        .takes_value(true)
        .possible_values(&["10", "20", "30"])
        .required(false))
    .get_matches();

  let time_to_wait = value_t_or_exit!(matches, "T", u32);
  println!("Time to wait value is {}", time_to_wait);
}
```

很容易猜出来这在干什么。我们的程序接受参数 `-T`（或长版本 `--timetowait`），它的值必须是 10, 20 或 30，默认是 10. 最后，我们输出这个参数的值，并退出程序。

此外，我们还提供了帮助信息，用户可以使用 `--help` 来查看。

### 退出状态码

在 C/C++ 中，主函数的返回值可以用于判断程序是否正常结束。在 Rust 中，主函数没有返回值，因此如果有需要，我们需要显式指定：

```rust
fn main() {
  // ...
  std::os::set_exit_status(1);
}
```

当 `main()` 执行完后，运行时会进行清理，并把状态码返回给运行环境。注意，并不一定非要在 `main()` 中设置状态码，你可以在任何地方设置，然后使用 `panic!()` 去结束程序，类似 C++ 的 `std::exit`.

## 编译优化

在开发过程中，我们通常按照：编辑、编译、调试，三者循环进行，这时并不需要编译出优化后的代码，因此 Rust 编译器默认不会执行优化。

同 C/C++ 一样，编译优化可能会导致编译时间的增加，还可能会对代码进行重排，导致回溯和断点调试可能不会指定到正确的行。

我们可以通过 `-O` 参数让 rustc 开启编译优化：

```shell
rustc -O hw.rs
```

这会在链接前调用 LLVM 优化器进行优化，生成运行速度更快的代码，当然，这以更长的编译时间为代价。

*TODO: 或许我们应该介绍一下使用 cargo 时的情况。*

## 增量编译

增量编译在「编辑、编译、调试」循环中很重要。增量编译只需要重新构建代码中被修改的部分，这可以大大减少整个构建时间。

C/C++ 本身不支持增量编译，需要额外的工具去处理（例如 make），它们会跟踪项目中的文件，并且维护了依赖关系，例如当 foo.h 被修改后，会重新编译所有依赖它的文件，而后链接。

Rust 最初的增量编译模型与这不同，它是 crate 级别而不是文件级别的增量编译。但是当单个 crate 很大时，表现很糟糕，因此 Rust 社区后来决定在 crate 级别的基础上添加[文件级别的增量编译支持](https://blog.rust-lang.org/2016/09/08/incremental.html)。在写这篇文档时，新发布的 rustc 1.54.0 刚刚恢复了默认情况下的增量编译支持，因为先前发布的 1.52.0 版本中增量编译出现了 bug，于是在 1.52.1 中这一功能被默认禁用了（详见 [1.52.1 发布信息](https://blog.rust-lang.org/2021/05/10/Rust-1.52.1.html)）。

## 管理项目

*TODO: 感觉这一节应该往前放放。*

在 C/C++ 中，我们通常用 `makefile` 之类的文件来管理项目。其中通常包括了项目中有哪些文件、它们之间的依赖关系、最终生成的目标文件是什么，以及编译和链接的参数。

Rust 大大简化了这件事，每个文件本身就是一个 `makefile`，其中包含了它的依赖项。[^2]例如下面这个例子：

```rust
// main.rs

mod pacman;

fn main() {
  let mut game = pacman::Game::new();
  game.start();
}
```

当我们在终端中执行 `rustc main.rs` 时，编译器会注意到对 `pacman` 的引用，并会搜索 `pacman.rs`（或者 `pacman/mod.rs`），将这个依赖一同编译，整个过程是递归进行的。

换句话说，即使我们有一个包含 1000 个文件的 Rust 项目，我们也可以简单地使用 `rustc main.rs` 去编译。

不过，如果当我们的项目依赖一些第三方库，或者我们就在写一个供其它项目使用的库时，该怎么办呢？

### Cargo

Cargo 是一个集构建和包管理为一体的工具，它可以拉取依赖项、构建它们、构建和链接你的项目、运行单元测试、安装二进制 crate、创建文档，以及将你的项目上传到仓库。

在 Rust 中，创建新项目最简单的方式是使用 `cargo` 命令：

```shell
cargo new hello_world
```

这会在上述命令的工作目录下创建出：

```plain
hello_world/
├── .git
│   └── ...
├── .gitignore
├── Cargo.toml
└── src
    └── main.rs
```

默认情况下，Cargo 会使用 git 进行版本控制。

构建项目同样十分简单，在项目的任意子目录下执行：

```shell
cargo build
```

如果你想编译出发行版本，可以添加 `--release` 参数，这会告诉编译器开启优化。

如果要运行单元测试，可以执行：

```shell
cargo test
```

### crate 和外部依赖

Cargo 不仅维护构建我们项目的一切，也管理着我们项目的依赖项。它会自动确保所有依赖项被下载下来，并进行构建。所有依赖项在 `Cargo.toml` 中定义。例如，当我们向项目中添加一个依赖项 `time` crate 时，只需修改 `Cargo.toml`：

```toml
[package]
name = "hello_world"
version = "0.1.0"
authors = ["Joe Blogs <jbloggs@somewhere.com>"]

[dependencies]
time = {version = "0.3.1", features = ["macros"]}
```

而后，执行 `cargo build`，Cargo 会从 [crates.io](https://crates.io/) 下载 `time` crate，以及所有 `time` 自身的依赖项，并会按合理的顺序构建这些依赖。第三方 crate 会下载并构建到你的 CARGO_HOME 目录。

使用就很简单，直接用就行了：

```rust
fn main() {
  let now = time::macros::date!(2021 - 8 - 11);
  println!("The time is {}", now);
}
```

所以，只需在修改 `Cargo.toml` 并在源代码中引用，就提供了足够的信息给 Cargo 来：

1. 拉取依赖的 crate.
2. 构建这些依赖。
3. 编译你的代码，并链接依赖项。

我们无需去试图弄清怎么正确编译这些依赖项，Cargo 会处理好的！

#### Cargo.lock

注意到，当我们第一次构建项目时，Cargo 会在项目根目录创建一个 `Cargo.lock` 文件。

这个文件是一个确定的列表，记录了哪些包的哪个版本应该被拉取并编译，这样一来，当我们再次构建项目时，构建出的仍是同样的东西，即使把缓存目录删除也不会有影响。如果你想强制让 Cargo 创建一个新的 `Cargo.lock` 文件（例如刚刚修改了 `Cargo.toml`），只需要运行 `cargo update` 命令。

[^1]: 你可以使用 `#[start]` 指令把入口函数修改为其它函数，不过默认情况下就是 `main()`.
[^2]: 事实上，例如 GCC 也可以生成 C/C++ 代码文件的依赖信息。
