# 【极速下载】gradle各版本快速下载地址大全Android Studio中通过CMake使用NDK并编译自定义库和添加预编译库

> Note：这篇文章是基于Android Studio 2.3版本的，对于很多功能2.2开始就已经支持，但是存在一些bug，例如CMake中的add_custom_command用echo打印调试信息再2.2版本中无法查看，直到2.3才修复。其他的更新暂时没有体会。

当然在进行下面的步骤前，需要先在Android Studio中的SDK Manager安装好LLDB、CMake还有NDK。

## 从没有勾选C++ Support的空项目开始

在新建项目的时候，是可以勾选添加C++ Support选项的，它会自动产生CMakeLists.txt以及在build.gradle中添加对CMakeLists.txt的引用。但是为了理解Android Studio中gradle结合CMake是如何构建NDK项目的，我们还是从零开始。

当最终项目完成之后，目录树应该是下面的样子（去除了与构建结构无直接关联的目录及文件，带**星号**的是我们即将添加的部分）：

```
.
├── app
│   ├── build.gradle
│   ├── CMakeLists.txt
│   └── src
│       └── main
│           ├── AndroidManifest.xml
│           ├──*cpp
│           │   ├── native-math.cpp
│           │   └── native-opencv.cpp
│           └── java
│               └── com/huang/opencvtest
│                   └── MainActivity.java
├──*distribution
│   ├── include
│   └── libs
├──*mathlib
│   ├── build.gradle
│   ├── CMakeLists.txt
│   └── src
│       └── main
│           ├── AndroidManifest.xml
│           └── cpp
│               ├── add.cpp
│               └── add.h
├──*openCVLibrary320
│   ├── build.gradle
│   └── src
│       └── main
│           ├── AndroidManifest.xml
│           └── java/org/opencv
├── gradle.properties
├── local.properties
├── build.gradle
└── settings.gradle
```

我们要添加自己定义的一个简单的数学C++库，以及OpenCV4Android预编译库。

首先将视图切换到Project，今后操作一直在Project视图中进行。

## 构建的结构

顶级的`build.gradle`在我们一般的使用下并不需要去设置，顶级的构建文件中我们仅仅需要改`settings.gradle`。`settings.gradle`描述了这一个项目在构建的时候需要包含哪一些模块。最开始只有`app`模块，因此里面只有一句话`include ':app'`。

系统在构建的时候，就会根据这句话，去寻找模块名为`app`的目录下面的`build.gradle`文件，并将其纳入构建结构树中。最终形成一课完整的构建结构树，将所有的部分联系在一起，编译成最终的Android应用。

每个模块中的我们称之为局部`build.gradle`，在这里面，定义了该模块为application或者library或其他，一般我们考虑这两个选项。这里面定义了许多构建这个模块时要用到的参数，在后续我们添加NDK支持的时候需要往里面添加一些参数，其中对CMakeLists.txt的路径就是在这里指定的。

## 添加自定义的C++库mathlib

### 创建源文件

1. 我的项目名称为OpenCVTest，所以右键这个项目点击`New->Module`，然后选`Android Library`，输入库的名称`MathLib`，然后`Finish`，系统就会生成对应的模块，并构建好初始的目录树。系统将库命名为`MathLib`，但是目录树中还是小写的`mathlib`。这个时候系统会自动在顶级`settings.gradle`添加对于这个新模块的`include`语句。并且在模块目录下构建好了初始的`build.gradle`。
2. 现在我们开始创建自己的C++库，首先右键`mathlib`目录下的`src/main`，然后选择`New->Directory`，输入`cpp`并确定。这个目录就是我们要创建的库的源文件的位置。
3. 右键`add`，点击`New->C/C++ Source File`，输入`add.cpp`，并选中`Create an associated header`。
4. 在`.cpp`文件中定义好一个简单的加法函数，并在`.h`文件中添加好对应声明。

库本身的定义就到此为止。

### 将源文件关联到构建系统中

我们用CMake来构建C++库，然后CMake又要和gradle结合，在Android Studio里面协作管理C++和Java的代码。

我们在模块`mathlib`的根目录下创建一个名为`CMakeLists.txt`的文件，写入

```
cmake_minimum_required(VERSION 3.4.1)
add_library(add SHARED
            src/main/cpp/add.cpp)
set(distribution_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../distribution)
set_target_properties(add PROPERTIES
                      LIBRARY_OUTPUT_DIRECTORY
                      ${distribution_DIR}/libs/${ANDROID_ABI})
target_include_directories(add
                           PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp)
#add_custom_command(TARGET add POST_BUILD
#                   COMMAND ${CMAKE_COMMAND} -E
#                   copy ${distribution_DIR}/libs/${ANDROID_ABI}/libadd.so
#                   ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/libadd.so
#                   COMMAND ${CMAKE_COMMAND} -E
#                   echo "output libadd.so to ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}"
#                   COMMENT "Copying add to output directory")
```

上面的CMake指令就是将源文件添加，并构件为一个Library即库，然后将创建后的`libadd.so`复制到项目根目录的`distribution`中，并最终被主模块`app`引用。

> *Note*：在这里需要注意的是，`target_include_directories`，它对创建的库设置`include`路径，针对目标来设置，可以避免与其他库的冲突，并且此时对自定义的库设置好了此路径后，后续导入这个库就不需要再次设置了。但对于预构建的库，就需要设置，稍后会有详细讲解。

> *Note*：这里有个问题。被注释的部分，是将`libadd.so`复制到`${CMAKE_LIBRARY_OUTPUT_DIRECTORY}`中，被复制到这个目录的`.so`文件会被自动添加到`.apk`文件中去。但是在稍后为OpenCV的库添加的`android.sourceSets.jniLibs.srcDirs`会让这个特性失效，因此这里还是注释掉，稍后统一为库设置路径，让系统复制到`.apk`文件中。

这个时候，`CMakeLists.txt`还是独立的，并没有与Android Studio的构建系统联系起来。

接下来我们在模块`mathlib`的`build.gradle`中的`defaultConfig{}`中添加如下语句：

```
externalNativeBuild {
    cmake {
        arguments '-DANDROID_PLATFORM=android-19',
                  '-DANDROID_TOOLCHAIN=clang', '-DANDROID_STL=gnustl_static'
        targets 'add'
    }
}
```

这里`arguments`是编译参数，而`targets`则是相比于`add_subdirectory`更高权限的方法。一般来说可以把它删去，即默认构建所有目标。

> *Note*：之所以在`defaultConfig{}`中设置，是因为在Android中，有许多不同的所谓风味，即免费版和付费版等，`defaultConfig{}`的设置可以在不同风味中覆盖，即它是缺省设置。

然后在`android{}`最后添加如下语句，将`CMakeLists.txt`关联起来。

```
externalNativeBuild {
    cmake {
        path 'CMakeLists.txt'
    }
}
```

这样，我们的自定义C++库就添加完成了。

## 在主模块中使用自定义C++库

C++库已经创建好了，接下来就要在主模块中使用它了。

为了使用自定义C++库，我们需要一个中间人，它从Android本身的Java程序中获取请求，然后使用我们的C++库中的函数计算得到结果，并将数据传回Android本身的Java程序中。

扮演这个角色的，也是一个C++源文件。

1. 我们在主模块`app`的根目录下创建一个`CMakeLists.txt`文件，再在`src/main`下创建一个目录`cpp`，其中再创建`native-math.cpp`。`CMakeLists.txt`用来将主模块中包括`native-math.cpp`在内的主模块中的库作为一个顶级库。

2. 我们先将`native-math.cpp`空着，先写`CMakeLists.txt`。

3. 在`CMakeLists.txt`中写入

   ```
   cmake_minimum_required(VERSION 3.4.1)
   
   set(distribution_DIR ${CMAKE_SOURCE_DIR}/../distribution)
   
   # set add lib
   add_library(lib_add SHARED IMPORTED)
   set_target_properties(lib_add PROPERTIES IMPORTED_LOCATION
                         ${distribution_DIR}/libs/${ANDROID_ABI}/libadd.so)
   include_directories(lib_add
                       ${distribution_DIR}/include)
   
   add_library(native-math SHARED
               src/main/cpp/native-math.cpp)
   set_target_properties(native-math PROPERTIES
                         LIBRARY_OUTPUT_DIRECTORY ${distribution_DIR}/libs/${ANDROID_ABI})
   target_link_libraries(native-math
                         android
                         lib_add
                         log)
   ```

   我们将之前创建的`libadd.so`使用导入，指定其路径，并在这里命名为`lib_add`。

   然后使用`target_link_libraries`将包括默认的`android`在内的`lib_add`链接到`native-math`中。

4. 在模块`app`的局部`build.gradle`中，像之前一样添加好对应的语句：

   `defaultConfig{}`中：

   ```
   externalNativeBuild {
       cmake 
       arguments '-DANDROID_PLATFORM=android-19',
                 '-DANDROID_TOOLCHAIN=clang', '-DANDROID_STL=gnustl_static'
       }
   }
   ndk {
       abiFilters 'armeabi-v7a'
   }
   ```

   其中`abiFilters`的作用是，在生成`.so`库时，只生成对应结构的文件，以至于最后的`.apk`文件也只有对应架构的`.so`库。

   然后在`android{}`中：

   ```
   externalNativeBuild {
       cmake {
           path 'CMakeLists.txt'
       }
   }
   sourceSets {
       main {
           jniLibs.srcDirs = ['../distribution/libs']
       }
   }
   ```

   其中的`sourceSets.main.jniLibs.srcDirs`就是指定`jni`库的了路径，让系统自动把`.so`文件复制到`.apk`文件中。

5. 在`native-math.cpp`中已经可以使用`jni`接口了。

6. 单独构建好`mathlib`之后，在`native-math.cpp`中把接口按照规定的格式写好即可。

   ```
   #include <jni.h>
   #include <add.h>
   
   extern "C" {
   jint Java_com_huang_opencvtest_MainActivity_addFromCpp(JNIEnv *env, jobject thiz, jint a, jint b){
       return add(a,b);
   }
   }
   ```

   函数的名称按照与Android中对应的`.java`路径的格式来写。

7. 接着在`src/main/java/*/MainActivity.java`中的`MainActivity`类下面，加载库，以及设置好对应的方法声明：

   ```
   static {
       System.loadLibrary("native-math");
   }
   private native int addFromCpp(int a, int b);
   ```

8. 然后就可以在`onCreate`方法中使用这个C++库定义的函数，在Java中对应的函数了。

9. 最后别忘了在项目中添加模块的依赖关系才可以正常运行这个Android App。右键项目`OpenCVTest`，选择`Open Module Settings`。选择`app->Dependencies`，添加`Module dependency`，选择`mathlib`，确定即可。

然后到此完成，可以运行App了。

## 添加OpenCV库的支持

### 导入OpenCV进项目

1. 从OpenCV的官网将OpenCV4Android 3.2下载下来，解压到某个目录。
2. 点击Android Studio的`File->New->Import Module`，然后选择路径为`OpenCV-android-sdk/sdk/java`，确定。并在导入之后，修改`build.gradle`中的SDK版本。
3. 在`Open Module Settings`中添加模块的依赖关系，使`app`依赖`openCVLibrary320`。

现在已经可以在`.java`文件中看得到OpenCV的自动补全了。

### 配置OpenCV的C++预构建库

1. 把包含文件夹`OpenCV-android-sdk/sdk/native/jni/include`和预构建库文件夹`OpenCV-android-sdk/sdk/native/libs`也复制到项目的`distribution`中。

2. 由于之前已经在添加C++库时修改了`app`的`build.gradle`，所以这个步骤现在不需要再执行了。

3. 由于OpenCV是预构建库，所以没有编译的过程，因此模块`openCVLibrary320`中不需要添加`CMakeLists.txt`等。我们直接在`app`模块中根目录下的`CMakeLists.txt`导入OpenCV的库即可。

   ```
   add_library(lib_opencv_java3 SHARED IMPORTED)
   set_target_properties(lib_opencv_java3 PROPERTIES
                         IMPORTED_LOCATION ${distribution_DIR}/libs/${ANDROID_ABI}/libopencv_java3.so)
   include_directories(lib_opencv_java3
                       ${distribution_DIR}/include)
   ```

   需要注意的是`.so`使用`SHARED`，`.a`使用`STATIC`。

   > *Note*：与上面自定义的库相对，这里是导入预构建的库，因此需要设置一下`include`目录，对预构建的库不能用`target_include_directories`，因此在这里我们使用`include_directories`来设置`include`目录。同样地，在这里设置好了以后，后续链接这个库的时候，就不需要再次设置这个目录了。

4. 然后先像之前C++库一样，在`src/main/cpp`下新建文件`native-opencv.cpp`，先空着。

5. 然后在文件最后再写上以下部分：

   ```
   add_library(native-opencv SHARED
               src/main/cpp/native-opencv.cpp)
   set_target_properties(native-opencv PROPERTIES
                         LIBRARY_OUTPUT_DIRECTORY ${distribution_DIR}/libs/${ANDROID_ABI})
   target_link_libraries(native-opencv
                         android
                         lib_opencv_java3
                         log)
   ```

6. 由于我们统一将`include`目录设置为`distribution/include`，`libs`目录设置为`distribution/libs`，因此在`app`的`build.gradle`中我们不需要再做出改动。

7. 现在就已经可以在`native-opencv.cpp`中使用OpenCV的函数了。

   ```
   #include <jni.h>
   #include <opencv2/opencv.hpp>
   #include <vector>
   
   using namespace cv;
   using namespace std;
   
   extern "C" {
   void Java_com_huang_opencvtest_MainActivity_nativeProcessFrame(JNIEnv *env, jobject thiz, jlong addrGray, jlong addrRGBA){
       Mat& gray = *(Mat *) addrGray;
       Mat& rgba = *(Mat *) addrRGBA;
       vector<KeyPoint> v;
   
       Ptr<ORB> orb = ORB::create();
       orb->detect(gray, v, cv::Mat());
   
       for (int i = 0; i < v.size(); ++i) {
           const KeyPoint& kp = v[i];
           circle(rgba, Point(kp.pt.x, kp.pt.y), 10, Scalar(255,0,0,255));
       }
   }
   }
   ```

8. 现在就可以在`src/main/java/*/MainActivity.java`中按照同样的方法，载入库，写上方法声明。最后，如下所示。

   ```
   static {
       System.loadLibrary("native-opencv");
       System.loadLibrary("native-math");
   }
   private native int addFromCpp(int a, int b);
   private native void nativeProcessFrame(long addrGray, long addrRGBA);
   ```

9. 然后就可以继续完成整个`app`了。

## 用手机检测并画出ORB特征点

具体的代码在我的[GitHub仓库](https://github.com/soaph/OpenCVTest)中。

## 值得注意的问题

在这两天自己对这篇文章所述的进行探索和尝试的过程中，也慢慢对Android Studio的结构有了更深的理解。期间也碰到了不少问题，参考了不少资料才得以解决，有些方案不是最好的，但是还是解决了问题。

1. 自己编译的`libadd.so`库自动加入`.apk`文件的问题。一开始只是按照官方的Sample，把创建好的文件放到项目根目录下的`distribution`中，并且在最开始接触的时候，觉得`build.gradle`中的`sourceSets`那个设置很烦，而且想纯用CMake解决这个问题，所以搜索了很多资料。后来参考了[Google Code论坛](https://code.google.com/p/android/issues/detail?id=214664)，发现可以在`CMakeLists.txt`中，添加一个自定义命令，将编译后的`.so`复制到`${CMAKE_LIBRARY_OUTPUT_DIRECTORY}`中，这样系统就会自动添加这些`.so`库了。但是这样就会引发下一个问题。
2. 预构建库（如OpenCV）的`.so`库文件载入`.apk`文件的问题。对于预构建库，不存在编译过程，那么之前那个编译后的自定义命令就无效了。然后在参考[Martin的文章](http://blog.csdn.net/martin20150405/article/details/53284442)的时候，遇到一个问题，就是OpenCV的`.so`库一直不会被复制到`.apk`中，但是修改成一样的配置文件，仍然不行，但是将这个博主的项目下载下来构建后发现`.apk`中有`lib_opencv_java3.so`文件。这就很奇怪，于是我仔细对比了两个项目，发现在这个博主的项目中，在`Android`视图看项目后，存放OpenCV库的文件夹被认为是某种`Assets Folder`，但是项目中没有找到任何配置文件描述这一特性。在参考了[Stack Overflow问答](http://stackoverflow.com/questions/38625690/how-to-create-jnilibs-folder-on-android-studio)，并重复实验多次后，我发现：在`src/main`下新建一个`JNI Folder`，并且命名为`jniLibs`，在`build.gradle`中会自动生成一段`sourceSets { main { jni.srcDirs = ['src/main/jniLibs', 'src/main/jniLibs/'] } }`，即使把这段删除，然后在其中放入OpenCV的`.so`库，不需要任何配置，系统就会自动复制其中的`.so`进入`.apk`，很奇怪。换成别的名字都不行。但是对于其他名字，不删除这一段，就可以。结论就是`jniLibs`这个名字本身有某种特殊性，系统会自动将其中的`.so`文件作为库，复制到`.apk`中。但是如[这篇文章](http://www.cumulations.com/blogs/9/Importing-library-having-native-code-in-Android-studio)所述，这样会使本应将`${CMAKE_LIBRARY_OUTPUT_DIRECTORY}`中`.so`库复制到`.apk`中的过程失效。在试过很多中办法之后，最后妥协，干脆把所有自定义或者预构建的库都使用`sourceSets.main.jniLibs.srcDirs`的方法来处理（注意这里`jni.srcDirs`和`jniLibs.srcDirs`使用起来目前没有发现有区别，两个都行）。
3. OpenCV Manager的问题。虽然还不清楚这个具体是怎么弄的，但是在直接使用OpenCV的时候可能会出现提示找不到OpenCV Manager的问题，然后闪退。参考了[Stack Overflow问答](http://stackoverflow.com/questions/23541938/opencv-android-without-opencv-manager)之后，就可以把这个问题解决了。
4. 尝试在`.cpp`使用ORB特征检测器检测ORB特征的时候，编译时提示`abstract class`等错误提示，一开始以为是库没有正确加载，搞了半天，结果参考了[OpenCV问答](http://answers.opencv.org/question/55247/image-registration-opencv-300/)之后，发现是自己使用方法错误。应该是接口变了，不能直接这样定义`ORB orb;`，而应该使用`Ptr<ORB> orb = ORB::create()`来定义。
5. 之前在Android Studio 2.2中，在`CMakeLists.txt`中想要用`echo`命令输出某些信息时，发现输出不了，但是刚好在2017/03/04我更新了最新版的Android Studio 2.3后，貌似这个Bug被修复了，可以在Gradle Console中看到输出结果了。
6. 应用对摄像头权限的问题。由于初涉Android，不懂权限的申请，但是现在还是要手动在`Settings->Apps`中对应用添加摄像头权限才可以显示图像。
7. OpenCV中图像旋转了90°的问题。图像永远与摄像头实际方向相差90°，经过搜索，这应该是OpenCV本身的问题。网上有不完美修复的办法，例如[OpenCV问答](http://answers.opencv.org/question/20325/how-can-i-change-orientation-without-ruin-camera-settings/)中的通过旋转和镜像来处理，最终的图像虽然方向对了，但是是被压扁了的，这肯定不行。但是后来在[Martin的文章](http://blog.csdn.net/martin20150405/article/details/53284442)中提供的项目中找到了解决办法，就是在`app`的`AndroidManifest.xml`中强制手机横屏，即可。
8. 正如前面构建步骤中所述，自定义C++库中，`include`目录是需要用`target_include_directories`来设置的，并且这个设置是一次性的，即自定义C++库构建之后，被其他模块链接的时候不需要再次设置这个目录。但是预构建库则不同，我们需要用`include_directories`来设置`include`目录。

还有一些不是遇到的问题，但是是在我寻找上述问题答案的过程中，找到的可以作为参考的资料：

1. [Why there are two versions of OpenCV 3.x and 2.4.xx ? - OpenCV Q&A Forum](http://answers.opencv.org/question/92583/why-there-are-two-versions-of-opencv-3x-and-24xx/)
2. [OpenCV 3.0 | OpenCV](http://opencv.org/opencv-3-0.html)

## 参考资料

1. [Issue 214664 - android - We need to package the standard shared libraries in to the APK - Android Open Source Project - Issue Tracker - Google Project Hosting](https://code.google.com/p/android/issues/detail?id=214664)
2. [使用Android Studio 2.2和Cmake （CMakeLists）让OpenCV 飞起来 - Martin的博客 - 博客频道 - CSDN.NET](http://blog.csdn.net/martin20150405/article/details/53284442)
3. [How to create jniLibs folder on Android Studio? - Stack Overflow](http://stackoverflow.com/questions/38625690/how-to-create-jnilibs-folder-on-android-studio)
4. [Importing Library Having Native Code in Android studio | Cumulations](http://www.cumulations.com/blogs/9/Importing-library-having-native-code-in-Android-studio)
5. [OpenCV Android without OpenCV Manager - Stack Overflow](http://stackoverflow.com/questions/23541938/opencv-android-without-opencv-manager)
6. [Image Registration (OpenCV 3.0.0) - OpenCV Q&A Forum](http://answers.opencv.org/question/55247/image-registration-opencv-300/)
7. [How can i change orientation without ruin camera settings? - OpenCV Q&A Forum](http://answers.opencv.org/question/20325/how-can-i-change-orientation-without-ruin-camera-settings/)
8. [Why there are two versions of OpenCV 3.x and 2.4.xx ? - OpenCV Q&A Forum](http://answers.opencv.org/question/92583/why-there-are-two-versions-of-opencv-3x-and-24xx/)
9. [OpenCV 3.0 | OpenCV](http://opencv.org/opencv-3-0.html)
10. [soaph/OpenCVTest: OpenCV 3.2, Android Studio 2.3, with CMake](https://github.com/soaph/OpenCVTest)