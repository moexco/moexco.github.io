---
title: "记一次Rust静态编译"
subtitle: ""
date: 2023-08-07T21:35:05+08:00
lastmod: 2023-08-07T21:35:05+08:00
draft: false
author: "moexco"
authorLink: ""
description: ""

tags: ["Rust"]
categories: ["Rust"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

![miku镇楼 pid:110505636](/rust_build_musl/202308_110505636.jpg)

## 前景提要

Rust这门语言出了名的难学难懂，我也是前前后后多次企图入坑又多次不了了之，一直到前段时间才真的用Rust写一些东西，在这之前，我接触过的语言也偏大众，像Java、Python、C等，最近使用最多的则是Go。

在Go的影响下，我更喜欢用标准库的习惯，给我的Rust开发带来了不少麻烦，如果说Go的标准库是一个工具箱，什么都带一点的话，Rust则是有但不多，HashMap这些是存在的，但Regex正则这些是不存在的。

与此同时，Rust的包管理和项目管理是非常成熟且完善的，尽管go也提供一整套的命令和工具，但使用上的区别还是非常明显的，这也是我这次Rust静态链接踩坑的主要原因。

## 情景还原

虽然是一个踩坑记录，但是原始程序已经编译好了，这里仅模拟踩坑全过程。

### 开发环境

- 系统：`ArchLinux` ~~即Glibc环境~~
- 版本：`rustc 1.71.1` ~~实际开发使用的是nightly~~

### 模拟程序

- 主程序

模拟程序内容不重要，重要的是用上个第三方包，比方说`reqwest`


```rust
fn main() {
    let body = reqwest::blocking::get("https://www.rust-lang.org").unwrap()
    .text();

    println!("body = {:?}", body);
}
```

- Cargo.toml内容

```toml
[package]
name = "hello"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
reqwest = { version = "0.11.18", features = ["json", "blocking"] }
```

## 踩坑

`cargo build --release`打包程序，然后丢给别人运行，跑不起来，问了一下环境，`alpine`，好家伙，`musl`，这我熟，在Go经常遇到`glibc`动态链接的程序无法在`musl`上运行的情况了，一个静态链接就解决了嘛。

但是问题来了，不同于Go重写了runtime，rust是依赖libc的，所以rust想要完全静态链接就必须抛弃`glibc`，选择`musl`，好在rust交叉编译不是什么难事，说干就干。

### 似乎是找不到依赖路径

```sh
rustup target add x86_64-unknown-linux-musl
pacman -Sy musl
cargo build --target x86_64-unknown-linux-musl --release
```

一套连招一气呵成，然后rust就给我当头一棒
```sh
   Compiling libc v0.2.147
   Compiling autocfg v1.1.0
   Compiling vcpkg v0.2.15
   Compiling pkg-config v0.3.27
   Compiling proc-macro2 v1.0.66
   Compiling cfg-if v1.0.0
   Compiling once_cell v1.18.0
   Compiling pin-project-lite v0.2.11
   Compiling bytes v1.4.0
   Compiling futures-core v0.3.28
   Compiling unicode-ident v1.0.11
   Compiling tokio v1.29.1
   Compiling cc v1.0.82
   Compiling itoa v1.0.9
   Compiling mio v0.8.8
   Compiling num_cpus v1.16.0
   Compiling quote v1.0.32
   Compiling socket2 v0.4.9
   Compiling slab v0.4.8
   Compiling futures-task v0.3.28
   Compiling memchr v2.5.0
   Compiling syn v2.0.28
   Compiling indexmap v1.9.3
   Compiling tracing-core v0.1.31
   Compiling futures-util v0.3.28
   Compiling foreign-types-shared v0.1.1
   Compiling openssl v0.10.56
   Compiling fnv v1.0.7
   Compiling http v0.2.9
   Compiling tracing v0.1.37
   Compiling foreign-types v0.3.2
   Compiling openssl-sys v0.9.91
   Compiling serde v1.0.183
   Compiling futures-io v0.3.28
   Compiling futures-sink v0.3.28
error: failed to run custom build command for `openssl-sys v0.9.91`

Caused by:
  process didn't exit successfully: `/root/hello/target/debug/build/openssl-sys-04208c1c308a778d/build-script-main` (exit status: 101)
  --- stdout
  cargo:rerun-if-env-changed=X86_64_UNKNOWN_LINUX_MUSL_OPENSSL_LIB_DIR
  X86_64_UNKNOWN_LINUX_MUSL_OPENSSL_LIB_DIR unset
  cargo:rerun-if-env-changed=OPENSSL_LIB_DIR
  OPENSSL_LIB_DIR unset
  cargo:rerun-if-env-changed=X86_64_UNKNOWN_LINUX_MUSL_OPENSSL_INCLUDE_DIR
  X86_64_UNKNOWN_LINUX_MUSL_OPENSSL_INCLUDE_DIR unset
  cargo:rerun-if-env-changed=OPENSSL_INCLUDE_DIR
  OPENSSL_INCLUDE_DIR unset
  cargo:rerun-if-env-changed=X86_64_UNKNOWN_LINUX_MUSL_OPENSSL_DIR
  X86_64_UNKNOWN_LINUX_MUSL_OPENSSL_DIR unset
  cargo:rerun-if-env-changed=OPENSSL_DIR
  OPENSSL_DIR unset
  cargo:rerun-if-env-changed=OPENSSL_NO_PKG_CONFIG
  cargo:rerun-if-env-changed=PKG_CONFIG_ALLOW_CROSS_x86_64-unknown-linux-musl
  cargo:rerun-if-env-changed=PKG_CONFIG_ALLOW_CROSS_x86_64_unknown_linux_musl
  cargo:rerun-if-env-changed=TARGET_PKG_CONFIG_ALLOW_CROSS
  cargo:rerun-if-env-changed=PKG_CONFIG_ALLOW_CROSS
  cargo:rerun-if-env-changed=PKG_CONFIG_x86_64-unknown-linux-musl
  cargo:rerun-if-env-changed=PKG_CONFIG_x86_64_unknown_linux_musl
  cargo:rerun-if-env-changed=TARGET_PKG_CONFIG
  cargo:rerun-if-env-changed=PKG_CONFIG
  cargo:rerun-if-env-changed=PKG_CONFIG_SYSROOT_DIR_x86_64-unknown-linux-musl
  cargo:rerun-if-env-changed=PKG_CONFIG_SYSROOT_DIR_x86_64_unknown_linux_musl
  cargo:rerun-if-env-changed=TARGET_PKG_CONFIG_SYSROOT_DIR
  cargo:rerun-if-env-changed=PKG_CONFIG_SYSROOT_DIR
  run pkg_config fail: pkg-config has not been configured to support cross-compilation.

  Install a sysroot for the target platform and configure it via
  PKG_CONFIG_SYSROOT_DIR and PKG_CONFIG_PATH, or install a
  cross-compiling wrapper for pkg-config and set it via
  PKG_CONFIG environment variable.

  --- stderr
  thread 'main' panicked at '

  Could not find directory of OpenSSL installation, and this `-sys` crate cannot
  proceed without this knowledge. If OpenSSL is installed and this crate had
  trouble finding it,  you can set the `OPENSSL_DIR` environment variable for the
  compilation process.

  Make sure you also have the development packages of openssl installed.
  For example, `libssl-dev` on Ubuntu or `openssl-devel` on Fedora.

  If you're in a situation where you think the directory *should* be found
  automatically, please open a bug at https://github.com/sfackler/rust-openssl
  and include information about your system as well as this message.

  $HOST = x86_64-unknown-linux-gnu
  $TARGET = x86_64-unknown-linux-musl
  openssl-sys = 0.9.91

  ', /root/.cargo/registry/src/index.crates.io-6f17d22bba15001f/openssl-sys-0.9.91/build/find_normal.rs:190:5
  note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
warning: build failed, waiting for other jobs to finish...
```

考虑到我`openssl`肯定是安装了的，系统里也有，我就把环境变量给它
```sh
OPENSSL_DIR=/usr/include OPENSSL_LIB_DIR=/usr/lib OPENSSL_INCLUDE_DIR=/usr/include cargo build --target=x86_64-unknown-linux-musl
```

显然我理解的方向是错误的，并没有就这样编译通过
```sh
   Compiling openssl-sys v0.9.91
   Compiling openssl-macros v0.1.1
   Compiling tinyvec_macros v0.1.1
   Compiling pin-utils v0.1.0
   Compiling bitflags v1.3.2
   Compiling hashbrown v0.12.3
   Compiling openssl v0.10.56
   Compiling futures-channel v0.3.28
   Compiling native-tls v0.2.11
   Compiling httparse v1.8.0
   Compiling serde v1.0.183
   Compiling futures-util v0.3.28
   Compiling tinyvec v1.6.0
   Compiling tokio-util v0.7.8
   Compiling indexmap v1.9.3
   Compiling log v0.4.19
   Compiling try-lock v0.2.4
   Compiling openssl-probe v0.1.5
   Compiling percent-encoding v2.3.0
   Compiling unicode-normalization v0.1.22
   Compiling want v0.3.1
   Compiling form_urlencoded v1.2.0
   Compiling http-body v0.4.5
   Compiling httpdate v1.0.2
   Compiling tower-service v0.3.2
   Compiling serde_json v1.0.104
   Compiling unicode-bidi v0.3.13
   Compiling ryu v1.0.15
   Compiling idna v0.4.0
   Compiling encoding_rs v0.8.32
   Compiling h2 v0.3.20
   Compiling url v2.4.0
   Compiling mime v0.3.17
   Compiling tokio-native-tls v0.3.1
   Compiling serde_urlencoded v0.7.1
   Compiling base64 v0.21.2
   Compiling ipnet v2.8.0
   Compiling hyper v0.14.27
   Compiling hyper-tls v0.5.0
   Compiling reqwest v0.11.18
   Compiling hello v0.1.0 (/root/hello)
error: linking with `cc` failed: exit status: 1
  |
  = note: LC_ALL="C" PATH="/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin:/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin/self-contained:/root/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" VSLANG="1033" "cc" "-m64" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/self-contained/rcrt1.o" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/self-contained/crti.o" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/self-contained/crtbeginS.o" "/tmp/rustcaDaF3l/symbols.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.1lmq1ouv58w3rwz.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.1yizfmm3nj5x3q7w.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.2nzud2ndcojz7uk2.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.3a55l8llsp3r2tcl.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.4ok09fnd53o40xs8.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.4y92ixbrkcvb8jys.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.57guh5q81h8sa6ag.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.5cdk9g0lv2noid4s.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.5g88t7k0ybyitp8h.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.dh08tp5wn056bn5.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.dqlvcrgqkc7ceom.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.hiecjvjjv53un8g.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.zjk48m8k5yuqax7.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.3bzdcys5ws4an6mz.rcgu.o" "-Wl,--as-needed" "-L" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps" "-L" "/root/hello/target/debug/deps" "-L" "/usr/lib" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib" "-Wl,-Bstatic" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libreqwest-a4928d2796b7a20a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhyper_tls-6adcb3f682fd63be.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbase64-f032cdc26b2188fd.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libipnet-9fcf7a1b05f70428.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio_native_tls-84d0836c10c03084.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde_urlencoded-12f3954529516483.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmime-61cb5f21eb43cd09.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libencoding_rs-7191fbee27b32ca8.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde_json-b7863fea1f06fdc2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libryu-eb41dd4558394161.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde-b1bc13ea6b13e86a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libnative_tls-8cc19a344b2b97ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl_probe-33c3113c2ab0c38a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl-878c34367adf445f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbitflags-a35fc4f5e760ae0c.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libforeign_types-344debc7b4283ac5.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libforeign_types_shared-d4b5ae7f9901e3ef.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl_sys-d44d47f3a15f4314.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhyper-ad505ff30a83f00f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libwant-da6c59dcd2b430e8.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtry_lock-63e33292a4cab6b7.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttparse-86bb3be05875b2be.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libh2-86a6152341939650.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libindexmap-700fb0f243cbf770.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhashbrown-3e2a8dddddd69e47.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio_util-01f3c807bcba0656.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_sink-47ab71e3b2a9d652.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtower_service-55f09582429b3ad2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtracing-22c55ea82046148d.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libcfg_if-763b1ddbb00ab312.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtracing_core-8b484e32d3a618dc.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libonce_cell-df35a5bc77e9bad2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_channel-b3ab7d62489521ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio-0a1f54475be772d7.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libnum_cpus-ed3c6a13d6ca91c3.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libsocket2-37ebd4a8a7b2d956.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmio-2908ea1e129c91b6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liblibc-4a181a3269fd660d.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttp_body-b0a126980fc702e6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liblog-343afbef5c0c4dc0.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_util-95f015b9173c8184.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmemchr-f2f0db64792846a0.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_io-814bd2be1702f58b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libslab-7b5ee7683556de32.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpin_project_lite-f0335dd74864094f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_task-834669506822b441.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpin_utils-e9460a26d7509d0b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_core-59d6481d783432fe.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liburl-72c7256e70d64cc4.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libidna-f828dd91a3024d62.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libunicode_normalization-12be77ce4e4ef380.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtinyvec-ac95596c2aa9b76c.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtinyvec_macros-40bbf77c062383ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libunicode_bidi-3e4ce736fb98ad05.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libform_urlencoded-8a01bf4df8ff1a6b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpercent_encoding-efe5dc91ab199535.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttp-caefd3ce726da4f6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libitoa-6b7a72b344370fc1.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbytes-db81abd5b66cf450.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfnv-7fa02a81681331dc.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd-493936ddc336b3cf.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libpanic_unwind-3dd70cbadf3e1002.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libobject-3558ee77f8b1ad74.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libmemchr-10f2552578618c97.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libaddr2line-7b520da5a73d1918.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libgimli-3702969c0e889474.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_demangle-c0cc4c329ee15590.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd_detect-874514dffe09e03c.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libhashbrown-e3f573f6d5b58355.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_std_workspace_alloc-85dc525393d56c52.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libminiz_oxide-36ca93bbf21bf97c.rlib" "/root/.rustup/toolchains
/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libadler-1041082638612281.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libunwind-e11982fd4ee76f53.rlib" "-lunwind" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcfg_if-78634847bd9095da.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/liblibc-930f912cc77d37c5.rlib" "-lc" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/liballoc-8cc0612c0889f762.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_std_workspace_core-918df048a2f8e1f4.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcore-d1705156e6561f5f.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcompiler_builtins-9ad5603f71a0f0cc.rlib" "-Wl,-Bdynamic" "-lssl" "-lcrypto" "-Wl,--eh-frame-hdr" "-Wl,-z,noexecstack" "-nostartfiles" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/self-contained" "-o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475" "-Wl,--gc-sections" "-static-pie" "-Wl,-z,relro,-z,now" "-nodefaultlibs" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/self-contained/crtendS.o" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/self-contained/crtn.o"
  = note: /usr/sbin/ld: /root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd-493936ddc336b3cf.rlib(std-493936ddc336b3cf.std.7b49138af54fd575-cgu.0.rcgu.o): in function `std::sys::unix::os::home_dir::fallback':
          /rustc/eb26296b556cef10fb713a38f3d16b9886080f26/library/std/src/sys/unix/os.rs:642:(.text._ZN3std3env8home_dir17h451fdab77b1c257fE+0xc9): warning: Using 'getpwuid_r' in statically linked applications requires at runtime the shared libraries from the glibc version used for linking
          /usr/sbin/ld: /root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd-493936ddc336b3cf.rlib(std-493936ddc336b3cf.std.7b49138af54fd575-cgu.0.rcgu.o): in function `<std::sys_common::net::LookupHost as core::convert::TryFrom<(&str,u16)>>::try_from::{{closure}}':
          /rustc/eb26296b556cef10fb713a38f3d16b9886080f26/library/std/src/sys_common/net.rs:206:(.text._ZN104_$LT$std..sys_common..net..LookupHost$u20$as$u20$core..convert..TryFrom$LT$$LP$$RF$str$C$u16$RP$$GT$$GT$8try_from28_$u7b$$u7b$closure$u7d$$u7d$17h2fd855f3da3ccc3cE+0x5c): warning: Using 'getaddrinfo' in statically linked applications requires at runtime the shared libraries from the glibc version used for linking
          /usr/sbin/ld: /root/hello/target/x86_64-unknown-linux-musl/debug/deps/libreqwest-a4928d2796b7a20a.rlib(reqwest-a4928d2796b7a20a.reqwest.db4bc6ec345dcfb4-cgu.14.rcgu.o): undefined reference to symbol '__tls_get_addr@@GLIBC_2.3'
          /usr/sbin/ld: /usr/lib/ld-linux-x86-64.so.2: error adding symbols: DSO missing from command line
          collect2: error: ld returned 1 exit status

  = note: some `extern` functions couldn't be found; some native libraries may need to be installed or have their path specified
  = note: use the `-l` flag to specify native libraries to link
  = note: use the `cargo:rustc-link-lib` directive to specify the native libraries to link with Cargo (see https://doc.rust-lang.org/cargo/reference/build-scripts.html#cargorustc-link-libkindname)

error: could not compile `hello` (bin "hello") due to previous error
```

### 似乎是链接器错误

`cc`链接器？网上一通搜索，要指定`musl-gcc`，这还不简单
```sh
CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER=musl-gcc CC_x86_64_unknown_linux_musl=musl-gcc CC=musl-gcc OPENSSL_DIR=/usr/include OPENSSL_LIB_DIR=/usr/lib OPENSSL_INCLUDE_DIR=/usr/include cargo build --target=x86_64-unknown-linux-musl
```

结果和上面没有什么区别，只是`cc`变成了`musl-gcc`，显然我的努力方向有问题，搜索了不少这方面的文章帖子，要么是没遇到什么问题就成功了，要么是折腾无解后选择在容器里编译

> 诸如社区`cross`方案我也使用过，编译依旧是报错，为了排除我项目代码的问题，我新建了一个`alpine`容器，在里面运行了`cargo build --release`没有问题，确认了是我交叉编译姿势有问题，不过这里的编译只是动态链接，`ldd`下依旧能看到依赖性，因为文件已经删除，就不展示了

### 求助交叉编译新星 `zig cc`

放弃是不可能放弃的，我又搜到了一个使用`zig`完成编译的帖子，zig cc是zig团队给zig开发的一个全新的c编译器，传说中口碑极好，据说`uber`甚至投资zig团队让他们优先处理uber在使用zig cc上遇到的问题

我立刻把zig安排上了，`pacman -Sy zig`

然后根据帖子的内容，新建一个内容，为zig cc指定编译目标
```shell
#!/bin/sh
zig cc  -target x86_64-linux-musl $@
```
并且在项目里新建`./.cargo/config.toml`，用来配置编译相关项，在其中配置
```shell
rustflags = ["-C", "linker-flavor=gcc", "-C", "link-self-contained=no"]
linker = "zcc-x64-linux"
```

接着就是一个编译，`CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER=/root/zcc CC_x86_64_unknown_linux_musl=/root/zcc CC=/root/zcc OPENSSL_DIR=/usr/include OPENSSL_LIB_DIR=/usr/lib OPENSSL_INCLUDE_DIR=/usr/include cargo build --target=x86_64-unknown-linux-musl`

但奇迹并没有发生，它还是报错了
```shell
   Compiling cfg-if v1.0.0
   Compiling once_cell v1.18.0
   Compiling libc v0.2.147
   Compiling pin-project-lite v0.2.11
   Compiling bytes v1.4.0
   Compiling itoa v1.0.9
   Compiling futures-core v0.3.28
   Compiling foreign-types-shared v0.1.1
   Compiling fnv v1.0.7
   Compiling foreign-types v0.3.2
   Compiling tracing-core v0.1.31
   Compiling slab v0.4.8
   Compiling memchr v2.5.0
   Compiling futures-task v0.3.28
   Compiling tinyvec_macros v0.1.1
   Compiling bitflags v1.3.2
   Compiling futures-io v0.3.28
   Compiling http v0.2.9
   Compiling socket2 v0.4.9
   Compiling num_cpus v1.16.0
   Compiling mio v0.8.8
   Compiling tokio v1.29.1
   Compiling tracing v0.1.37
   Compiling openssl-sys v0.9.91
   Compiling hashbrown v0.12.3
   Compiling futures-sink v0.3.28
   Compiling pin-utils v0.1.0
   Compiling tinyvec v1.6.0
   Compiling futures-util v0.3.28
   Compiling indexmap v1.9.3
   Compiling openssl-probe v0.1.5
   Compiling openssl v0.10.56
   Compiling log v0.4.19
   Compiling try-lock v0.2.4
   Compiling percent-encoding v2.3.0
   Compiling unicode-normalization v0.1.22
   Compiling form_urlencoded v1.2.0
   Compiling want v0.3.1
   Compiling httparse v1.8.0
   Compiling futures-channel v0.3.28
   Compiling serde v1.0.183
   Compiling http-body v0.4.5
   Compiling tower-service v0.3.2
   Compiling ryu v1.0.15
   Compiling httpdate v1.0.2
   Compiling unicode-bidi v0.3.13
   Compiling encoding_rs v0.8.32
   Compiling idna v0.4.0
   Compiling url v2.4.0
   Compiling tokio-util v0.7.8
   Compiling h2 v0.3.20
   Compiling native-tls v0.2.11
   Compiling serde_urlencoded v0.7.1
   Compiling tokio-native-tls v0.3.1
   Compiling serde_json v1.0.104
   Compiling ipnet v2.8.0
   Compiling mime v0.3.17
   Compiling base64 v0.21.2
   Compiling hyper v0.14.27
   Compiling hyper-tls v0.5.0
   Compiling reqwest v0.11.18
   Compiling hello v0.1.0 (/root/hello)
error: linking with `/root/zcc` failed: exit status: 1
  |
  = note: LC_ALL="C" PATH="/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin:/root/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" VSLANG="1033" "/root/zcc" "-m64" "/tmp/rustceja6Ye/symbols.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.1lmq1ouv58w3rwz.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.1yizfmm3nj5x3q7w.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.2nzud2ndcojz7uk2.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.3a55l8llsp3r2tcl.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.4ok09fnd53o40xs8.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.4y92ixbrkcvb8jys.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.57guh5q81h8sa6ag.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.5cdk9g0lv2noid4s.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.5g88t7k0ybyitp8h.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.dh08tp5wn056bn5.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.dqlvcrgqkc7ceom.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.hiecjvjjv53un8g.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.zjk48m8k5yuqax7.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.3bzdcys5ws4an6mz.rcgu.o" "-Wl,--as-needed" "-L" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps" "-L" "/root/hello/target/debug/deps" "-L" "/usr/lib" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib" "-Wl,-Bstatic" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libreqwest-a4928d2796b7a20a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhyper_tls-6adcb3f682fd63be.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbase64-f032cdc26b2188fd.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libipnet-9fcf7a1b05f70428.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio_native_tls-84d0836c10c03084.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde_urlencoded-12f3954529516483.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmime-61cb5f21eb43cd09.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libencoding_rs-7191fbee27b32ca8.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde_json-b7863fea1f06fdc2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libryu-eb41dd4558394161.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde-b1bc13ea6b13e86a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libnative_tls-8cc19a344b2b97ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl_probe-33c3113c2ab0c38a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl-878c34367adf445f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbitflags-a35fc4f5e760ae0c.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libforeign_types-344debc7b4283ac5.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libforeign_types_shared-d4b5ae7f9901e3ef.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl_sys-d44d47f3a15f4314.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhyper-ad505ff30a83f00f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libwant-da6c59dcd2b430e8.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtry_lock-63e33292a4cab6b7.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttparse-86bb3be05875b2be.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libh2-86a6152341939650.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libindexmap-700fb0f243cbf770.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhashbrown-3e2a8dddddd69e47.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio_util-01f3c807bcba0656.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_sink-47ab71e3b2a9d652.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtower_service-55f09582429b3ad2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtracing-22c55ea82046148d.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libcfg_if-763b1ddbb00ab312.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtracing_core-8b484e32d3a618dc.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libonce_cell-df35a5bc77e9bad2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_channel-b3ab7d62489521ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio-0a1f54475be772d7.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libnum_cpus-ed3c6a13d6ca91c3.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libsocket2-37ebd4a8a7b2d956.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmio-2908ea1e129c91b6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liblibc-4a181a3269fd660d.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttp_body-b0a126980fc702e6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liblog-343afbef5c0c4dc0.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_util-95f015b9173c8184.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmemchr-f2f0db64792846a0.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_io-814bd2be1702f58b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libslab-7b5ee7683556de32.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpin_project_lite-f0335dd74864094f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_task-834669506822b441.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpin_utils-e9460a26d7509d0b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_core-59d6481d783432fe.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liburl-72c7256e70d64cc4.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libidna-f828dd91a3024d62.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libunicode_normalization-12be77ce4e4ef380.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtinyvec-ac95596c2aa9b76c.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtinyvec_macros-40bbf77c062383ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libunicode_bidi-3e4ce736fb98ad05.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libform_urlencoded-8a01bf4df8ff1a6b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpercent_encoding-efe5dc91ab199535.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttp-caefd3ce726da4f6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libitoa-6b7a72b344370fc1.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbytes-db81abd5b66cf450.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfnv-7fa02a81681331dc.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd-493936ddc336b3cf.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libpanic_unwind-3dd70cbadf3e1002.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libobject-3558ee77f8b1ad74.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libmemchr-10f2552578618c97.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libaddr2line-7b520da5a73d1918.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libgimli-3702969c0e889474.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_demangle-c0cc4c329ee15590.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd_detect-874514dffe09e03c.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libhashbrown-e3f573f6d5b58355.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_std_workspace_alloc-85dc525393d56c52.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libminiz_oxide-36ca93bbf21bf97c.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libadler-1041082638612281.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libunwind-e11982fd4ee76f53.rlib" "-lunwind" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcfg_if-78634847bd9095da.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/liblibc-930f912cc77d37c5.rlib" "-lc" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/liballoc-8cc0612c0889f762.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_std_workspace_core-918df048a2f8e1f4.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcore-d1705156e6561f5f.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcompiler_builtins-9ad5603f71a0f0cc.rlib" "-Wl,-Bdynamic" "-lssl" "-lcrypto" "-Wl,--eh-frame-hdr" "-Wl,-z,noexecstack" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib" "-o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475" "-Wl,--gc-sections" "-static-pie" "-Wl,-z,relro,-z,now" "-nodefaultlibs"
  = note: error: static library 'unwind' not found. search paths:
           /root/hello/target/x86_64-unknown-linux-musl/debug/deps/libunwind.a
           /root/hello/target/debug/deps/libunwind.a
           /usr/lib/libunwind.a
           /root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libunwind.a
           /root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libunwind.a
           suggestion: use full paths to static libraries on the command line rather than using -l and -L arguments


error: could not compile `hello` (bin "hello") due to previous error
```

我只能在设备上找到`libunwind.so`，找不到`.a`文件，别人也是卡在了这里。我尝试在系统里搜索这个文件，还真让我找到了
`./.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/self-contained/libunwind.a`

我毫不客气的就把它给链接到了上面报错里提到的路径里，比方说`/usr/lib`

`ln -s ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/
lib/self-contained/libunwind.a /usr/lib`

然后再次尝试编译，再次报错
```shell
Compiling hello v0.1.0 (/root/hello)
error: linking with `/root/zcc` failed: exit status: 1
  |
  = note: LC_ALL="C" PATH="/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin:/root/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" VSLANG="1033" "/root/zcc" "-m64" "/tmp/rustctAxXkT/symbols.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.1lmq1ouv58w3rwz.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.1yizfmm3nj5x3q7w.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.2nzud2ndcojz7uk2.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.3a55l8llsp3r2tcl.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.4ok09fnd53o40xs8.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.4y92ixbrkcvb8jys.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.57guh5q81h8sa6ag.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.5cdk9g0lv2noid4s.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.5g88t7k0ybyitp8h.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.dh08tp5wn056bn5.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.dqlvcrgqkc7ceom.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.hiecjvjjv53un8g.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.zjk48m8k5yuqax7.rcgu.o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475.3bzdcys5ws4an6mz.rcgu.o" "-Wl,--as-needed" "-L" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps" "-L" "/root/hello/target/debug/deps" "-L" "/usr/lib" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib" "-Wl,-Bstatic" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libreqwest-a4928d2796b7a20a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhyper_tls-6adcb3f682fd63be.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbase64-f032cdc26b2188fd.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libipnet-9fcf7a1b05f70428.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio_native_tls-84d0836c10c03084.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde_urlencoded-12f3954529516483.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmime-61cb5f21eb43cd09.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libencoding_rs-7191fbee27b32ca8.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde_json-b7863fea1f06fdc2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libryu-eb41dd4558394161.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libserde-b1bc13ea6b13e86a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libnative_tls-8cc19a344b2b97ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl_probe-33c3113c2ab0c38a.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl-878c34367adf445f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbitflags-a35fc4f5e760ae0c.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libforeign_types-344debc7b4283ac5.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libforeign_types_shared-d4b5ae7f9901e3ef.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libopenssl_sys-d44d47f3a15f4314.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhyper-ad505ff30a83f00f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libwant-da6c59dcd2b430e8.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtry_lock-63e33292a4cab6b7.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttparse-86bb3be05875b2be.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libh2-86a6152341939650.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libindexmap-700fb0f243cbf770.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhashbrown-3e2a8dddddd69e47.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio_util-01f3c807bcba0656.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_sink-47ab71e3b2a9d652.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtower_service-55f09582429b3ad2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtracing-22c55ea82046148d.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libcfg_if-763b1ddbb00ab312.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtracing_core-8b484e32d3a618dc.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libonce_cell-df35a5bc77e9bad2.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_channel-b3ab7d62489521ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtokio-0a1f54475be772d7.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libnum_cpus-ed3c6a13d6ca91c3.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libsocket2-37ebd4a8a7b2d956.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmio-2908ea1e129c91b6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liblibc-4a181a3269fd660d.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttp_body-b0a126980fc702e6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liblog-343afbef5c0c4dc0.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_util-95f015b9173c8184.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libmemchr-f2f0db64792846a0.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_io-814bd2be1702f58b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libslab-7b5ee7683556de32.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpin_project_lite-f0335dd74864094f.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_task-834669506822b441.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpin_utils-e9460a26d7509d0b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfutures_core-59d6481d783432fe.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/liburl-72c7256e70d64cc4.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libidna-f828dd91a3024d62.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libunicode_normalization-12be77ce4e4ef380.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtinyvec-ac95596c2aa9b76c.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libtinyvec_macros-40bbf77c062383ea.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libunicode_bidi-3e4ce736fb98ad05.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libform_urlencoded-8a01bf4df8ff1a6b.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libpercent_encoding-efe5dc91ab199535.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libhttp-caefd3ce726da4f6.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libitoa-6b7a72b344370fc1.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libbytes-db81abd5b66cf450.rlib" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/libfnv-7fa02a81681331dc.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd-493936ddc336b3cf.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libpanic_unwind-3dd70cbadf3e1002.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libobject-3558ee77f8b1ad74.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libmemchr-10f2552578618c97.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libaddr2line-7b520da5a73d1918.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libgimli-3702969c0e889474.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_demangle-c0cc4c329ee15590.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libstd_detect-874514dffe09e03c.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libhashbrown-e3f573f6d5b58355.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_std_workspace_alloc-85dc525393d56c52.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libminiz_oxide-36ca93bbf21bf97c.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libadler-1041082638612281.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libunwind-e11982fd4ee76f53.rlib" "-lunwind" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcfg_if-78634847bd9095da.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/liblibc-930f912cc77d37c5.rlib" "-lc" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/liballoc-8cc0612c0889f762.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/librustc_std_workspace_core-918df048a2f8e1f4.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcore-d1705156e6561f5f.rlib" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib/libcompiler_builtins-9ad5603f71a0f0cc.rlib" "-Wl,-Bdynamic" "-lssl" "-lcrypto" "-Wl,--eh-frame-hdr" "-Wl,-z,noexecstack" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-musl/lib" "-o" "/root/hello/target/x86_64-unknown-linux-musl/debug/deps/hello-1b96c31f4f6a4475" "-Wl,--gc-sections" "-static-pie" "-Wl,-z,relro,-z,now" "-nodefaultlibs"
  = note: ld.lld: error: undefined symbol: __gcc_personality_v0
          >>> referenced by iofclose.o:(DW.ref.__gcc_personality_v0) in archive /usr/lib/libc.a


error: could not compile `hello` (bin "hello") due to previous error
```

到这一步，我不得不承认，我努力的方向从一开始就是错误的

## 解决方案

> 虽然各种尝试都失败了，我直面困难撞得重伤倒地，但问题还得解决，我还是得把东西编译出来，于是有了以下两个解决方案

### 方案一：切换rustls
在尝试了`zigbuild`、`cross`等方案都失败之后，我开始考虑绕开`openssl`，~~虽然不能解决问题，但是我可以躲开问题~~

确认依赖项都提供了把`native-tls`换成`rustls`的features，于是我一个修改，`Cargo.toml`内容就变成了
```toml
[package]
name = "hello"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
reqwest = { version = "0.11.18", default-features = false, features = ["json", "blocking", "rustls"] }
```

然后删除了之前折腾的zig cc的遗留配置文件，防止干扰，然后一个编译，最简单的`cargo build --release --target=x86_64-unknown-linux-musl`

**然后就成功了，就这么简单**

确认一下
```shell
[root@1f8081b4b5b6 hello]# ldd ./target/x86_64-unknown-linux-musl/release/hello
        statically linked
```

> 感动来得太简单，虽然问题得到解决，但`openssl`就像个阴影一样挥之不去，经过多天的在群里跟各位大佬偷师学艺，从旁敲击，我找到了我编译不过的真正原因

### 方案二：找到原因

> 某天，群里有人交叉编译依赖了openssl的东西也失败了，然后大佬问"开启vendored了吗"，他表示没有，然后去试了下，回来说成功了

因此，我去谷歌搜了好多关于vendored关键字的内容，辅以rust/cargo等字眼，最终一无所获

因为编译失败的是`openssl`，于是我去看了一下它的文档`https://crates.io/crates/openssl-sys/0.9.36`，其中一段描述引起了我的注意
```
Vendored
[dependencies]
openssl = { version = "0.10", features = ["vendored"] }
```
这可不就是我在谷歌上没找到的东西嘛，先把这一段往依赖里加再说，于是`Cargo.toml`就变成这样
```toml
[package]
name = "hello"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
reqwest = { version = "0.11.18", features = ["json", "blocking"] }
openssl = { version = "0.10", features = ["vendored"] }
```

**编译走起，它终于编译成功了！**

#### 结论

> 经过一番讨论研究，我项目并没有直接依赖openssl但是为了引入了依赖的方式并不够直观和优雅，通过查看reqwest的features发现它其实提供了`native-tls-vendored`将openssl的vendored给暴露了出来

于是，`Cargo.toml`就应该是这样
```toml
[package]
name = "hello"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
reqwest = { version = "0.11.18", features = ["json", "blocking", "native-tls-vendored"] }
```

如此就直观多了，如果遇到依赖了openssl但是没有暴露它的features的方案，可以使用上面那种并不直观的方式，追加一个openssl的直接依赖去修改

## 写在最后

**不要硬撞**

该看文档看文档，该问大佬问大佬，在错误的方向上努力除了浪费时间之外最大的用处只有打击自信了
