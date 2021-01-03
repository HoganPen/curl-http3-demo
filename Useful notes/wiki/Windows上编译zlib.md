把zlib 1.2.8解压到zlib/zlib-1.2.8

在deflate.c文件中把deflate_copyright改成一个static变量。

在zlib目录底下创建并用Visual Studio 2008命令行运行如下bat即可生成Debug版本：

@Echo off

set LIBDIR=%CD%\..

:: create build directory
mkdir build
cd build

cmake -G "NMake Makefiles" ..\zlib-1.2.8 ^
-DCMAKE_INSTALL_PREFIX=%LIBDIR%\zlib\install ^
-DCMAKE_C_FLAGS_DEBUG="/D_DEBUG /MTd /Zi /Ob0 /Od /RTC1" ^
-DCMAKE_BUILD_TYPE=Debug

nmake
nmake install

cd ..

mkdir elvic\lib
mkdir elvic\include
copy install\lib\zlibd.lib elvic\lib\zlibd.lib
copy install\lib\zlibd.lib elvic\lib\libz_d.lib
copy install\lib\zlibstaticd.lib elvic\lib\libz_st_d.lib
copy install\bin\zlibd.dll elvic\lib\zlibd.dll
copy install\include\*.h elvic\include\

 

生成Release版本请用如下bat：

@Echo off

set LIBDIR=%CD%\..

:: create build directory
mkdir build
cd build

cmake -G "NMake Makefiles" ..\zlib-1.2.8 ^
-DCMAKE_INSTALL_PREFIX=%LIBDIR%\zlib\install ^
-DCMAKE_C_FLAGS_RELEASE="/MT /O2 /Ob2 /D NDEBUG" ^
-DCMAKE_BUILD_TYPE=Release

nmake
nmake install

cd ..

mkdir elvic\lib
mkdir elvic\include
copy install\lib\zlib.lib elvic\lib\zlib.lib
copy install\lib\zlib.lib elvic\lib\libz.lib
copy install\lib\zlibstatic.lib elvic\lib\libz_st.lib
copy install\bin\zlib.dll elvic\lib\zlib.dll
copy install\include\*.h elvic\include\

build会生成到zlib/elvic目录中。