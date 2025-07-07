---

typora-root-url: README.assets
---

# xdemo-rustdesk编译文档

## 一.环境搭建

rustc＋msvc(win10sdk)+vcpkg(uuid等lib)+python3.8(用于build.py自动化编译)

**过程容易遇到网络问题,请配置git全局代理**

### 1.从安装rust环境开始

参考 [RsProxy](https://rsproxy.cn/) 配置rust下载镜像

![1751529650495](/./../README.assets/1751529650495.png)

下载rustup-init,https://www.rust-lang.org/zh-CN/tools/install,搭建rustc环境,获取rust工具链!(/1751529650495.png)

按1即可![1751528523943](/./../README.assets/1751528523943.png)

会自动下载visual studio

### 2.vs安装选择

![1751528704404](/./../README.assets/1751528704404.png)

![1751528739893](/./../README.assets/1751528739893.png)

### 3.vcpkg安装

vpkg下载地址](https://github.com/microsoft/vcpkg) 

配置vcpkg用户环境变量

目录为自己vcpkg安装目录

![1751528879996](/./../README.assets/1751528879996.png)

path里添加%VCPKG_ROOT%

cmd内使用vcpkg安装需要的依赖

```
vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static ffmpeg:x64-windows-static
```



### 4.安装llvm

https://github.com/llvm/llvm-project/releases

安装时勾选添加环境变量(推荐)

### 5.安装flutterSDK

官方ci使用Flutter 3.22.3 [Archive | Flutter](https://docs.flutter.dev/install/archive) 

下载链接https://storage.googleapis.com/flutter_infra_release/releases/stable/windows/flutter_windows_3.22.3-stable.zip

添加环境变量

![1751530015564](/./../README.assets/1751530015564.png)

## 二.编译

### 1.根据官方ci修改版本

cargo.toml文件中修改为1.80.1版本

```
flutter_rust_bridge = { version = "=1.80.1", features = ["uuid"], optional = true}
```

pubspec.yaml中修改为13.0.0

```
extended_text: 13.0.0
```

### 2.拉取子模块hbb_common

 git submodule update --init --recursive 

### 3.生成flutter与rust的桥接文件

终端内先进flutter文件夹get flutter依赖

![1751530180431](/./../README.assets/1751530180431-1751531065199.png)

回到根目录生成桥接文件![1751530408726](/./../README.assets/1751530408726.png)

```
 flutter_rust_bridge_codegen --rust-input ./src/flutter_ffi.rs --dart-output ./flutter/lib/generated_bridge.dart 
```

### 4.开始编译

```powershell
python build.py --flutter --hwcodec --vram
```

拉取hwcodec源码可能遇到网络问题,自行设置代理.

### 关于可能会遇到的LNK1120报错

修改build.rs

```rust
#[cfg(windows)]
fn build_windows() {
    let file = "src/platform/windows.cc";
    let file2 = "src/platform/windows_delete_test_cert.cc";
    cc::Build::new().file(file).file(file2).compile("windows");
    println!("cargo:rustc-link-lib=WtsApi32");
    println!("cargo:rerun-if-changed={}", file);
    println!("cargo:rerun-if-changed={}", file2);
    //解决LNK1120错误
    // 1. 先加 Windows SDK 路径，保证 mfuuid/uuid 优先
    println!("cargo:rustc-link-search=native=C:/Program Files (x86)/Windows Kits/10/Lib/10.0.22621.0/um/x64");
    println!("cargo:rustc-link-lib=mfplat");
    println!("cargo:rustc-link-lib=mf");
    println!("cargo:rustc-link-lib=mfreadwrite");
    println!("cargo:rustc-link-lib=mfuuid");
    println!("cargo:rustc-link-lib=uuid");
    println!("cargo:rustc-link-lib=strmiids");
    // 2. 最后加 vcpkg 路径，只影响 swresample
    println!("cargo:rustc-link-search=native=C:/Program Files/Microsoft Visual Studio/2022/Community/VC/vcpkg/installed/x64-windows-static/lib");
    println!("cargo:rustc-link-lib=swresample");
}
```

以及src\platform\windows.cc

增加两个头文件,放在最顶上

```
#include <initguid.h>
#include <codecapi.h>
```

