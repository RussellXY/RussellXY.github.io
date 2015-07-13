---
layout: post
title: WebRTC下载和编译存档
date: {}
comments: true
categories: iOS
tags: 
  - WebRTC
keywords: iOS WebRTC
description: WebRTC下载和编译存档
published: true
---


  前段时间由于项目需要研究了一下`WebRTC`，功能确实强大：能很轻松的实现实时的视频和音频通话，并且还带有消除噪音、回声等功能，不过恶心的是虽然`WebRTC`开源也有好几年了，但是国内外的文档却非常的少并且还有点时间久远了，我光是下载源码和编译都花了很长一段时间去查阅资料和实践，中途也遇到过各种问题，所以在此存档一下整个下载和编译的过程。

####准备：
* 8G的内存
* 安装最新的[git](http://sourceforge.net/projects/git-osx-installer/)和[subversion](https://subversion.apache.org/download/)
* `xcode`+`command line tools`
* 在国内下载的话一定需要一个高速且稳定的VPN，或者使用ssh和shadowsocks来给终端代理

以下操作都需要处于翻墙状态,不能翻墙的同学就去找个vpn或者代理吧

打开终端，先创建好一个用于保存`WebRTC`的文件夹
{% highlight ruby %}
RussellY:~ linmin$ mkdir -p webrtc_source/
{% endhighlight %}

下载`depot_tools`，一个用于编译`Chromium`和`WebRTC`的工具

{% highlight ruby %}
RussellY:~ linmin$ cd webrtc_source/
RussellY:~ linmin$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
{% endhighlight %}

把`depot_tools`的路径加入系统环境变量
{% highlight ruby %}
RussellY:~ linmin$ echo "export PATH=$PWD/depot_tools:$PATH" > $HOME/.bash_profile
{% endhighlight %}

####开始下载
先新建一个存放`WebRTC`源代码的文件夹
{% highlight ruby %}
RussellY:webrtc_source linmin$ mkdir webrtc
RussellY:webrtc_source linmin$ cd webrtc/
{% endhighlight %}

生成配置文件
{% highlight ruby %}
RussellY:webrtc linmin$ gclient config --name src http://webrtc.googlecode.com/svn/trunk
RussellY:webrtc linmin$ echo "target_os = ['ios']" >> .gclient
RussellY:webrtc linmin$ gclient sync —force
{% endhighlight %}

然后就开始下载了，中间一定要注意不要让vpn掉线了(还是shadowsocks好用...不用当心断线的问题)

####开始编译
下载成功后就进入编译环节，首先创建一个脚本文件并赋予权限并进入文件开始编辑
{% highlight ruby %}
RussellY:webrtc linmin$ touch build.sh
RussellY:webrtc linmin$ chmod +x build.sh
RussellY:webrtc linmin$ vi build.sh
{% endhighlight %}

将下面这段代码拷贝进去
{% highlight ruby %}
function build_iossim_ia32() {
    echo "*** building WebRTC for the ia32 iOS simulator";
    export GYP_GENERATORS="ninja";
    export GYP_DEFINES="OS=ios target_arch=ia32";
    export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_ia32";
    export GYP_CROSSCOMPILE=1;
    pushd src;
    gclient runhooks;
    #若需要编译Debug版则将下面这条语句改成 ninja -C out_ios_ia32/Debug-iphoneos AppRTCDemo,同时在下面的libtool命令出修改路径为Debug-iphones
    ninja -C out_ios_ia32/Release-iphonesimulator iossim AppRTCDemo;
 
    echo "*** creating iOS ia32 libraries";
    pushd out_ios_ia32/Release-iphonesimulator/;
    rm -f  libapprtc_signaling.a;
    popd;
    mkdir -p out_ios_ia32/libs;
    libtool -static -o out_ios_ia32/libs/libWebRTC-ia32.a out_ios_ia32/Release-iphonesimulator/lib*.a;
    strip -S -x -o out_ios_ia32/libs/libWebRTC.a -r out_ios_ia32/libs/libWebRTC-ia32.a;
    rm -f out_ios_ia32/libs/libWebRTC-ia32.a;
    echo "*** result: $PWD/out_ios_ia32/libs/libWebRTC.a";
 
    popd;
}
 
function build_iossim_x86_64() {
    echo "*** building WebRTC for the x86_64 iOS simulator";
    export GYP_GENERATORS="ninja";
    export GYP_DEFINES="OS=ios target_arch=x64";
    export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_x86_64";
    export GYP_CROSSCOMPILE=1;
    pushd src;
    gclient runhooks;
    #若需要编译Debug版则将下面这条语句改成 ninja -C out_ios_x86_64/Debug-iphoneos AppRTCDemo,同时在下面的libtool命令出修改路径为Debug-iphones
    ninja -C out_ios_x86_64/Release-iphonesimulator iossim AppRTCDemo;
 
    echo "*** creating iOS x86_64 libraries";
    pushd out_ios_x86_64/Release-iphonesimulator/;
    rm -f  libapprtc_signaling.a;
    popd;
    mkdir -p out_ios_x86_64/libs;
    libtool -static -o out_ios_x86_64/libs/libWebRTC-x86_64.a out_ios_x86_64/Release-iphonesimulator/lib*.a;
    strip -S -x -o out_ios_x86_64/libs/libWebRTC.a -r out_ios_x86_64/libs/libWebRTC-x86_64.a;
    echo "*** result: $PWD/out_ios_x86_64/libs/libWebRTC.a";
 
    popd;
}
 
function build_iosdevice_armv7() {
    echo "*** building WebRTC for armv7 iOS devices";
    export GYP_GENERATORS="ninja";
    export GYP_DEFINES="OS=ios target_arch=arm";
    export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_armv7";
    export GYP_CROSSCOMPILE=1;
    pushd src;
    gclient runhooks;
    #若需要编译Debug版则将下面这条语句改成 ninja -C out_ios_armv7/Debug-iphoneos AppRTCDemo,同时在下面的libtool命令出修改路径为Debug-iphones
    ninja -C out_ios_armv7/Release-iphoneos AppRTCDemo;
 
    echo "*** creating iOS armv7 libraries";
    pushd out_ios_armv7/Release-iphoneos/;
    rm -f  libapprtc_signaling.a;
    popd;
    mkdir -p out_ios_armv7/libs;
    libtool -static -o out_ios_armv7/libs/libWebRTC-armv7.a out_ios_armv7/Release-iphoneos/lib*.a;
    strip -S -x -o out_ios_armv7/libs/libWebRTC.a -r out_ios_armv7/libs/libWebRTC-armv7.a;
    echo "*** result: $PWD/out_ios_armv7/libs/libWebRTC.a";
 
    popd;
}
 
function build_iosdevice_arm64() {
    echo "*** building WebRTC for arm64 iOS devices";
    export GYP_GENERATORS="ninja";
    export GYP_DEFINES="OS=ios target_arch=arm64";
    export GYP_GENERATOR_FLAGS="$GYP_GENERATOR_FLAGS output_dir=out_ios_arm64";
    export GYP_CROSSCOMPILE=1;
    pushd src;
    gclient runhooks;
    #若需要编译Debug版则将下面这条语句改成 ninja -C out_ios_arm64/Debug-iphoneos          AppRTCDemo,同时在下面的libtool命令出修改路径为Debug-iphones
    ninja -C out_ios_arm64/Release-iphoneos AppRTCDemo;
 
    echo "*** creating iOS arm64 libraries";
    pushd out_ios_arm64/Release-iphoneos/;
    rm -f  libapprtc_signaling.a;
    popd;
    mkdir -p out_ios_arm64/libs;
    libtool -static -o out_ios_arm64/libs/libWebRTC-arm64.a out_ios_arm64/Release-iphoneos/lib*.a;
    strip -S -x -o out_ios_arm64/libs/libWebRTC.a -r out_ios_arm64/libs/libWebRTC-arm64.a;
    echo "*** result: $PWD/out_ios_arm64/libs/libWebRTC.a";
 
    popd;
}
 
function combine_libs() 
{
    echo "*** combining libraries";
    lipo  -create   src/out_ios_ia32/libs/libWebRTC.a \
            src/out_ios_x86_64/libs/libWebRTC.a \
            src/out_ios_armv7/libs/libWebRTC.a \
            src/out_ios_arm64/libs/libWebRTC.a \
            -output libWebRTC.a;
    echo "The public headers are located in $PWD/src/talk/app/webrtc/objc/public/*.h";
}
 
function create_framework() {
    echo "*** creating WebRTC.framework";
    rm -rf WebRTC.framework;
    mkdir -p WebRTC.framework/Versions/A/Headers;
    cp ./src/talk/app/webrtc/objc/public/*.h WebRTC.framework/Versions/A/Headers;
    cp libWebRTC.a WebRTC.framework/Versions/A/WebRTC;
 
    pushd WebRTC.framework/Versions;
    ln -sfh A Current;
    popd;
    pushd WebRTC.framework;
    ln -sfh Versions/Current/Headers Headers;
    ln -sfh Versions/Current/WebRTC WebRTC;
    popd;
}
 
function clean() 
{
    echo "*** cleaning";
    pushd src;
    rm -rf out_ios_arm64 out_ios_armv7 out_ios_ia32 out_ios_x86_64;
    popd;
    echo "*** all cleaned";
}
 
function update()
{
    gclient sync --force
    pushd src
    svn info | grep Revision > ../svn_rev.txt
    popd
}
 
function build_all() {
    build_iossim_ia32 && build_iossim_x86_64 && \
    build_iosdevice_armv7 && build_iosdevice_arm64 && \
    combine_libs && create_framework;
}
 
function run_simulator_ia32() {
    echo "*** running webrtc appdemo on ia32 iOS simulator";
    src/out_ios_ia32/Release-iphonesimulator/iossim src/out_ios_ia32/Release-iphonesimulator/AppRTCDemo.app;
}
 
function run_simulator_x86_64() {
    echo "*** running webrtc appdemo on x86_64 iOS simulator";
    src/out_ios_x86_64/Release-iphonesimulator/iossim -d 'iPhone 6' -s '8.1'  src/out_ios_x86_64/Release-iphonesimulator/AppRTCDemo.app;
}
 
function run_on_device_armv7() {
    echo "*** launching on armv7 iOS device";
    ideviceinstaller -i src/out_ios_armv7/Release-iphoneos/AppRTCDemo.app;
    echo "*** launch complete";
}
 
function run_on_device_arm64() {
    echo "*** launching on arm64 iOS device";
    ideviceinstaller -i src/out_ios_arm64/Release-iphoneos/AppRTCDemo.app;
    echo "*** launch complete";
}

$@
{% endhighlight %}

执行脚本文件
{% highlight ruby %}
RussellY:webrtc linmin$ ./build.sh build_all
{% endhighlight %}

等待执行完毕，成功后会在当前文件夹产生WebRTC框架
{% highlight ruby %}
RussellY:webrtc linmin$ ls  
WebRTC.framework build_webrtc.sh  libWebRTC.a      src
{% endhighlight %}

到此WebRTC编译就已经完成，如果发现没有生成框架文件的话应该是编译过程中报错了，原因有很多种，比如我就遇到过因为我电脑里有一个无效的开发者证书导致的错误，所以这个时候最好是将`build.sh`脚本里面的`function`一个一个的执行然后分析错误信息，比如先执行这条命令只编译模拟器版
{% highlight ruby %}
RussellY:webrtc linmin$ ./build.sh build_iossim_ia32
{% endhighlight %}
查看执行过程中是否发现错误信息，如果没有则继续编译`x86_64`版
{% highlight ruby %}
RussellY:webrtc linmin$ ./build.sh build_iossim_x86_64
{% endhighlight %}
以此为例一直把`arm64`都成功编译完，如果中间抱错了则根据错误信息进行修改然后再删除文件夹并重新编译，最后执行合并静态库和创建框架命令
{% highlight ruby %}
RussellY:webrtc linmin$ ./build.sh combine_libs
RussellY:webrtc linmin$ ./build.sh create_framework
{% endhighlight %}

