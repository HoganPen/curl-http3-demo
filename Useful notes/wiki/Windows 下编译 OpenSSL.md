# Windows 下编译 openssl-1.0.2m

## 简述

OpenSSL 是一个开源的第三方库，它实现了 SSL（Secure SocketLayer）和 TLS（Transport Layer Security）协议，被广泛企业应用所采用。对于一般的开发人员而言，在 [Win32 OpenSSL](http://slproweb.com/products/Win32OpenSSL.html) 上下载已经编译好的 OpenSSL 库是省力省事的好办法。对于高级的开发用户，可能需要适当的修改或者裁剪 OpenSSL，那么编译它就成为了一个关键问题。

下面，主要讲述如何在 Windows 上编译 OpenSSL 库。

**|** 版权声明：一去、二三里，未经博主允许不得转载。

## 环境准备

1. 下载并安装 Visual Studio（以 VS 2015 为例）。

2. 下载并安装 ActivePerl。
   下载地址：http://www.activestate.com/activeperl/downloads
   我下载的是：[ActivePerl-5.26.0.2600-MSWin32-x64-403866.exe](https://www.activestate.com/activeperl/downloads/thank-you?dl=http://downloads.activestate.com/ActivePerl/releases/5.26.0.2600/ActivePerl-5.26.0.2600-MSWin32-x64-403866.exe)

   打开命令提示符，定位到 `D:\Program Files\Perl\eg` 目录，执行 `perl example.pl`，若提示 `Hello from ActivePerl!` 则说明 Perl 安装成功：

   ![这里写图片描述](https://img-blog.csdn.net/20171121152529542?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3. 下载并安装 Nasm 汇编器，并将 `D:\Program Files\NASM` 添加到系统环境变量 Path 中。
   下载地址：http://www.nasm.us/
   我下载的是：[nasm-2.13.01-installer-x64.exe](http://www.nasm.us/pub/nasm/releasebuilds/2.13.01/win64/nasm-2.13.01-installer-x64.exe)

4. 下载并安装 OpenSSL
   下载地址：http://www.openssl.org/
   我下载的是：[openssl-1.0.2m.tar.gz](https://www.openssl.org/source/openssl-1.0.2m.tar.gz)

   完成上述所有步骤，将 OpenSSL 包解压至 `E:\openssl-1.0.2m`，便可以进行编译了。

   **注意：** 解压后的目录中有两个文件 - INSTALL.W32、INSTALL.W64，包含了 OpenSSL 的各个编译步骤。

## 编译步骤

1. 打开命令提示符，定位至 `E:\openssl-1.0.2m`：

   ![这里写图片描述](https://img-blog.csdn.net/20171121162554675?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2. 输入 `perl Configure VC-WIN32 --prefix=E:\OpenSSL`（将其安装到 `E:\OpenSSL`）：

   ![这里写图片描述](https://img-blog.csdn.net/20171121162604606?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3. 输入 `ms\do_nasm`：

   ![这里写图片描述](https://img-blog.csdn.net/20171121162615119?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4. 将命令提示符定位至 `D:\Program Files\Microsoft Visual Studio 14.0\VC\bin`， 然后输入`vcvars32.bat`：

   ![这里写图片描述](https://img-blog.csdn.net/20171121162847746?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

   如果没有这一步，会提示 nmake 不是内部或外部命令等一系列错误。

5. 再次将命令提示符定位至 `E:\openssl-1.0.2m`，然后输入 `nmake -f ms\ntdll.mak`：

   ![这里写图片描述](https://img-blog.csdn.net/20171121163647938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

   完成之后，会在 `openssl-1.0.2m` 目录下生成一个名为 out32dll 的文件夹，里面包含了一些动态库和 exe 文件：

   ![这里写图片描述](https://img-blog.csdn.net/20171121163824108?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

6. 输入 `nmake -f ms\ntdll.mak test`，若最终显示 `passed all tests` 则说明生成的库正确：

   ![这里写图片描述](https://img-blog.csdn.net/20171121164414118?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

7. 输入 `nmake -f ms\ntdll.mak install`：

   ![这里写图片描述](https://img-blog.csdn.net/20171121164029652?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

   完成之后，会在 `E:\OpenSSL` 目录下生成 bin、include、lib、ssl 四个文件夹：

   ![这里写图片描述](https://img-blog.csdn.net/20171121164237892?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbmcxOTg5MDgyMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**注意：**

- 以上编译的是 release 版本，若要编译 debug 版，将上述第 2 步中的 `VC-WIN32` 改成 `debug-VC-WIN32`即可。
- 若要编译静态库，则用 `ms\nt.mak` 替换掉上面用到的 `ms\ntdll.mak` 即可。
- 若要生成不带汇编支持的库，则需将上述第 2、3 步用 `perl Configure VC-WIN32 no-asm --prefix=E:\OpenSSL` 和 `ms\do_ms` 替换。
- 在 `E:\openssl-1.0.2m\tmp32dll` 文件夹下包含相应的汇编文件。

# Windows 编译 openssl-1.1.1d

1，下载 openssl 源码包：https://www.openssl.org/source/openssl-1.1.1d.tar.gz

2，下载并安装 NASM：https://www.nasm.us/pub/nasm/releasebuilds/2.13.01/win64/nasm-2.13.01-installer-x64.exe

记得将 BIN 加入 PATH环境变量

3，下载并安装 Perl：https://pre-platform-installers.s3.amazonaws.com/ActivePerl-5.28.1.2801-MSWin32-x64-24563874.exe?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJAFTYUXEZJ3HWLEQ%2F20200306%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200306T035849Z&X-Amz-Expires=21600&X-Amz-SignedHeaders=host&X-Amz-Signature=82981147f8d5f593e61d4dde4630507e6fc1f2cb630f5bce832053933a110c4e



安装完 perl 之后，记得将 perl.exe 加入PATH 环境变量

然后，打开控制台 ，输入：cpan

这里需要安装 dmake

在cpan控制台下

cpan> install dmake

然后，将 dmake.exe 加入 PATH 环境变量。

好了。准备工作终于完成了。



最关键 的来了，在开始菜单找到 x64_x86 Cross Tools Command Prompt for VS 2017，

然后打开，再切换到你的 openssl 解压的目录下面。类似这样的：

[![image](https://img2018.cnblogs.com/blog/802855/202003/802855-20200306162030093-87341855.png)](https://img2018.cnblogs.com/blog/802855/202003/802855-20200306162027543-566008376.png)

 

第一步： perl configure VC-WIN32 no-shared --prefix=D:\OpenSSL\Win32

第二步：nmake

第三步：nmake install

第四步：找到 D:\OpenSSL\Win32，这里就是你需要的静态库。

如果你需要动态库。去掉 no-shared 试试，我懒得去试了，嗯。我人比较懒。

如里你要编译 64 位的，好吧，反正我是没有编译成功的，要不，你试试吧，将WIN32 改为 WIN64A，即可。

# you may need to install the Win32::Console module

ActivePerl-5.28.1.XXXX.msi安装后，命令行执行：

```
D:\work\scm\cfet\openssl>perl Configure VC-WIN32 no-asm no-shared enable-tls1_3
--prefix="D:\work\scm\cfet\openssl\lib\x86"
```

会出现如下提示而无法继续。

Can't locate Win32/Console.pm in @INC (you may need to install the Win32::Console module) (@INC contains: C:\Perl64\site\lib C:\Perl64\lib) at C:\Perl64\lib/ActivePerl/Config.pm line 400.

解决办法，修改C:\Perl64\lib\ActivePerl\Config.pm，大约在400行左右：

```perl
# Prevent calling Win32::Console::DESTROY on a STDOUT handle
my $console;
sub _warn {
    # my($msg) = @_;
    # unless (-t STDOUT) {
	# print "\n$msg\n";
	# return;
    # }
    # require Win32::Console;
    # unless ($console) {
	# $console = Win32::Console->new(Win32::Console::STD_OUTPUT_HANDLE());
    # }
    # my($col,undef) = $console->Size;
    # print "\n";
    # my $attr = $console->Attr;
    # $console->Attr($Win32::Console::FG_RED | $Win32::Console::BG_WHITE);
    # for (split(/\n/, "$msg")) {
	# $_ .= " " while length() < $col-1;
	# print "$_\n";
    # }
    # $console->Attr($attr);
    # print "\n";
}
```