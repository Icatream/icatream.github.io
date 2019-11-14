---
title: Rust windows环境下的编译与debug
date: 2019-11-14 11:10:59
tags: Rust
categories: essay
---

# MSVC
[Rust官网](https://www.rust-lang.org/)提供了*rustup-init.exe*.
除此之外, 还需要一个c编译器. windows环境说的也不是很清楚.
好在9102年了, 微软出了一个简单的安装工具 *vs_buildtools*.
在这个页面下[Visual Studio Download](https://visualstudio.microsoft.com/zh-hans/downloads/), 找到 *Visual Studio 生成工具* 下载运行, 选择需要的功能安装即可, 无需安装整个*Visaul Studio*.

## Proxy
国内嘛, *cargo* 当然是需要proxy的.
在```C:\Users\username\.cargo```下创建```config```文件.
内容如下:
```
[http]
proxy = "127.0.0.1:1080"

[https]
proxy = "127.0.0.1:1080"
```

* ```rustup``` 也需要proxy.

* cmd proxy

```
set http_proxy=127.0.0.1:1080
set https_proxy=127.0.0.1:1080
```

* powershell proxy

```
$ENV:HTTP_PROXY=127.0.0.1:1080
$ENV:HTTPS_PROXY=127.0.0.1:1080
```

## Vscode Debug
vscode 安装 *Rust Extension Pack* 插件(全套Rust插件), *C/C++*, *Native Debug* 插件, 就可以用msvc工具链debug了. 但每次都得改需要debug的.exe文件名, 很麻烦(不知道有没有更好的办法). 而且, 在查看```Option<Box<T>>```的时候会出些问题, 明明是```Some```却显示```None```.

# GNU
Jetbrains的Clion只能用gnu工具链进行debug.

## 工具链切换

```
rustup default stable-msvc
rustup default stable-gnu
rustup default nightly-msvc
rustup default nightly-gnu
```

## 安装mingw64
windows下安装gnu环境需要安装mingw64.

[mingw-w64.org](http://mingw-w64.org/doku.php/download)里有大量toolchains, 选择 *MingW-W64-builds* 安装即可.
*MingW-W64-builds* 是一个installer, 也就是实际内容还是得再下载的, 然而国内显然无法下载. 这次proxy也失败了(可能得路由器proxy了). 不过可以直接找到文件源下载.

[mingw-w64/files](https://sourceforge.net/projects/mingw-w64/files/)里有2 * 2 * 2个版本. installer的功能仅仅是在这些版本中选择.
其中, *x86_64* 和 *i686* 分别是64位和32位, 我们当然是选择64位了.
*seh* 和 *sjlj* 是异常处理, 其中*seh* 是windows原生的, 性能更好, 我们反正不用在意其他细节, 选 *seh* 就完事了.
*posix* 和 *win32* 是多线程相关的, 具体差距我没弄清楚, 大多数还是选 *posix*. 所以选择最新的 *x86_64-posix-seh* 下载就好.

再将rustup切换到gnu工具链, Clion中设置一下Toolchains就可以使用Clion进行debug了. Clion的debug界面就清晰很多了, 也不用每次在配置文件中修改.exe文件名.

但是, ```Option<Box<T>>```完全没有内容, 之前vscode存在异常, Clion里直接看不了堆内存...

# 踩坑
最开始装mingw的时候, 找来找去, 看推荐装win-builds, 就装了win-builds. 第一次走了狗屎运了, 居然把win-builds装完了.

切换到gnu工具链, 可以编译运行, 但开始debug就卡住, 过一会, Clion异常, *commands timed out*. 网上也没找到原因, 暂时就把这事放了一段时间.

后面想想是不是之前win-builds安装的问题, 在[mingw-w64.org](http://mingw-w64.org/doku.php/download)里找到[win-builds](http://win-builds.org/doku.php/download_and_installation_from_windows). 按照下面的 *Proxies* 说明, 重新安装了一次( *win-builds.exe* 的命名不同, 运行界面居然不一样!!). 这次用 wget 把文件全下载下来再装, 完了还是一样, debug就卡住.

网上查了查明白了, 确实是win-builds的问题, win-builds更全面, 主要是用于交叉编译的, mingw-w64的兼容性更好.

## 总结
这一趟下来, 别的没啥, proxy命令学了很多.