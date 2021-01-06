# Windows下CLion配置Android NDK编译环境

这里记录怎么在Windows下的CLion中配置Android NDK，以备不时之需，也希望能帮到他人。

1.安装MinGW

下载安装文件（链接[https://download.csdn.net/download/ICANOFCSU/12093062，](https://download.csdn.net/download/ICANOFCSU/12093062)https://download.csdn.net/download/justinanyes/10205184，这里是MinGW5.1.6），为了省事，把所有的组件全部安装上。

注意：这里一定要安装MinGW，不要安装Cygwin！！！我之前就是在Cygwin上折腾了一下午，最终失败。

2.配置Toolchains

![toolchains配置](https://img-blog.csdnimg.cn/20200109180034170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0lDQU5PRkNTVQ==,size_16,color_FFFFFF,t_70)

这里使用的CLion版本为2019.3。从File>>Settings>>Build,Execution,Deployment进入Toolcains设置。然后添加一个MinGW配置，设置好前面安装的MinGW主目录，CMake路径选择Bundled，再设置Make，gcc，g++，Debugger为NDK中相应的可执行程序路径。示例中使用的NDK版本为r16b，其他版本也一样，请根据自己的实际情况设置。

注意：虽然Cygwin也可以在Windows下作为CLion的编译工具链，但这里不能选择，因为它会导致编译android_toolchain.cmake脚本中操作系统环境（MinGW下为“Windows”，而Cygwin下为“Cygwin”）判断出错，从而某些文件的路径设置出错，编译不起来。

3.配置cmake

![cmake配置](https://img-blog.csdnimg.cn/20200109181019734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0lDQU5PRkNTVQ==,size_16,color_FFFFFF,t_70)

在新建一个CMake配置，Name随便，Toolchain选择上一步创建的工具链，CMake options中输入以下主要宏定义：

-DCMAKE_TOOLCHAIN_FILE="xxx\ndk_home\build\cmake\android_toolchain.cmake"

-DANDROID_TOOLCHAIN=gcc

-DANDROID_NATIVE_API_LEVEL=android-14

-DANDROID_ABI=armeabi-v7a

-DANDROID_CPP_FEATURES="rrti exceptions"

-DANDROID_ARM_NEON=TRUE

-DANDROID_STL=gnustl_static

其中CMAKE_TOOLCHAIN_FILE最重要，用于指定android_toolchain.cmake（位于NDK主目录\build\cmake\下）路径，其他以ANDROID开头的宏的含义和用途在android_toolchain.cmake的开头有说明。如下：

```bash
# Configurable variables.
# Modeled after the ndk-build system.
# For any variables defined in:
#         https://developer.android.com/ndk/guides/android_mk.html
#         https://developer.android.com/ndk/guides/application_mk.html
# if it makes sense for CMake, then replace LOCAL, APP, or NDK with ANDROID, and
# we have that variable below.
# The exception is ANDROID_TOOLCHAIN vs NDK_TOOLCHAIN_VERSION.
# Since we only have one version of each gcc and clang, specifying a version
# doesn't make much sense.
#
# ANDROID_TOOLCHAIN
# ANDROID_ABI
# ANDROID_PLATFORM
# ANDROID_STL
# ANDROID_PIE
# ANDROID_CPP_FEATURES
# ANDROID_ALLOW_UNDEFINED_SYMBOLS
# ANDROID_ARM_MODE
# ANDROID_ARM_NEON
# ANDROID_DISABLE_NO_EXECUTE
# ANDROID_DISABLE_RELRO
# ANDROID_DISABLE_FORMAT_STRING_CHECKS
# ANDROID_CCACHE
```

4.编译打包

![编译打包](https://img-blog.csdnimg.cn/20200109183111105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0lDQU5PRkNTVQ==,size_16,color_FFFFFF,t_70)

打开CMake工程并加载，选择之前的配置，点击小锤子即开始NDK编译过程，编译产物位于debug目录下。

## WINDOWS下使用CLION进行NDK编程(CMAKE)

​    众所周知，使用ndk编译代码有三种使用方式，分别是基于 Make 的 ndk-build、CMake以及独立工具链。以前进行ndk编程都是使用ndk-build进行的，新建jni目录，编写Android.mk和Application.mk，使用Notepad++进行代码编写，然后执行ndk-build进行编译。不过随着项目越来越大，这种基于记事本的编辑简直无法忍受了，于是想到了使用CLion IDE进行代码的编写和编译。CLion使用的是CMake进行项目构建的。

​    在ndk r19版本以前，要在CLion上编译ndk只能采用独立工具链的方式，创建独立工具链供CLion调用，官方文档如下：

[独立工具链-官方文档](https://developer.android.com/ndk/guides/standalone_toolchain)

不过r19之后，ndk的默认工具链都是独立工具链了，不需要再创建了。截止至2020.7.1，最新的NDK版本为21.0.6113669，本文将基于该ndk版本使用CLion进行编程。

# 1.工具链配置

首先安装CLion和ndk工具包，然后新建工程，打开菜单File->Settings->Build,Execution,Deployment->Toolchains，点击加号选择“MinGW”新建一个工具链，如果系统没有MinGW的话，可以点击菜单右边的“[Download](https://mingw-w64.org/doku.php/download/mingw-builds)”，在新打开的网页中点击“[Sourceforge](https://sourceforge.net/projects/mingw-w64/files/Toolchains targetting Win32/Personal Builds/mingw-builds/installer/mingw-w64-install.exe/download)”进行下载安装，安装完成后还需要添加环境变量，网上教程很多就不再赘述了。这里我装在C盘，工具链配置如下图所示，主要是Make、C Compiler、C++ Compiler以及Debugger这四个路径的配置。

![img](https://www.freesion.com/images/932/5a338648ac01f47f592e41904c91b104.png)

其中"\ndk\21.0.6113669\"为你的ndk根目录，根目录结构如下图所示。

![img](https://www.freesion.com/images/376/b7f56bd70031bc44edd5ce57deac2660.png)

# 2.CMAKE配置

打开菜单File->Settings->Build,Execution,Deployment->CMake，点击加号添加新的CMake项目配置，Name名字随便起。Toolchains选择刚才的工具链配置。 CMake options点击右边的展开，输入以下配置项。

```
-DCMAKE_TOOLCHAIN_FILE="D:\AndroidStudioSDK\ndk\21.0.6113669\build\cmake\android.toolchain.cmake"
-DCMAKE_SYSTEM_NAME=Android
-DANDROID_ABI=arm64-v8a
-DCMAKE_ANDROID_NDK="D:\AndroidStudioSDK\ndk\21.0.6113669"
-DCMAKE_SYSTEM_VERSION=24
-DCMAKE_C_FLAGS=""
-DCMAKE_CXX_FLAGS=""
-DCMAKE_ANDROID_NDK_TOOLCHAIN_VERSION=clang
```

CMake配置如下图所示，下面的三项可以留空。。

![img](https://www.freesion.com/images/174/a8275eedc4e7c6716c703f94d3ef4486.png)



其中最重要的是CMAKE_TOOLCHAIN_FILE、CMAKE_SYSTEM_NAME以及ANDROID_ABI这三项的配置，第一个指定ndk的CMake配置文件，它位于<NDK>/build/cmake/android.toolchain.cmake，第二个是目标系统平台，这里为Android，第三个是架构，这里手机CPU是arm64-v8a架构的，所以用arm64-v8a。具体配置参考项可以访问[官方文档](https://cmake.org/cmake/help/v3.16/manual/cmake-toolchains.7.html#cross-compiling-for-android-with-the-ndk)，左上角选择CMake版本即可查看。

# 3.编译下载

演示就用最简单的Hello World!吧，main.c代码如下。

```cpp
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

CMakeLists.txt配置如下。

```
cmake_minimum_required(VERSION 3.16)
project(Demo C)
set(CMAKE_C_STANDARD 11)
add_executable(Demo main.c)
```

点击左边的锤子图标即可编译，编译完成后文件生成在cmake-build-CMake配置名目录下。![img](https://www.freesion.com/images/805/013489d8bec5054cc881696f3b1ad305.png)![img](https://www.freesion.com/images/897/bf7ddb52a925013a32178a3783c9bc09.png)

使用adb下载到手机或模拟器，赋给可执行权限，然后执行即可看到效果，JNI编程则引入头文件jni.h并把项目类型改为Libray即可生成so库。

![img](https://www.freesion.com/images/66/19dce267a23b0a251f63b5911775d73a.png)

 