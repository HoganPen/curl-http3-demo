# Gradle自己定义插件

在Gradle中创建自己定义插件，Gradle提供了三种方式：

- 在build.gradle脚本中直接使用
- 在buildSrc中使用
- 在独立Module中使用

开发Gradle插件能够在IDEA中进行开发。也能够在Android Studio中进行开发，它们唯一的不同，就是IDEA提供了Gradle开发的插件，比較方便创建文件和文件夹，而Android Studio中，开发人员须要手动创建（但实际上。这些文件夹并不多，也不复杂，全然能够手动创建）。

## 在项目中使用

在Android Studio中创建一个标准的Android项目，整个文件夹结构例如以下所看到的：

```
├── app
│   ├── build.gradle
│   ├── libs
│   └── src
│       ├── androidTest
│       │   └── java
│       ├── main
│       │   ├── AndroidManifest.xml
│       │   ├── java
│       │   └── res
│       └── test
├── build.gradle
├── buildSrc
│   ├── build.gradle            ---1
│   └── src
│       └── main
│           ├── groovy          ---2
│           └── resources       ---3
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
├── local.properties
└── settings.gradle
```

当中。除了buildSrc文件夹以外，都是标准的Android文件夹，而buildSrc就是Gradle提供的在项目中配置自己定义插件的默认文件夹。开发Gradle要创建的文件夹，也就是RootProject/src/main/groovy和RootProject/src/main/resources两个文件夹。

> 在配置完毕后，假设配置正确，相应的文件夹将被IDE所识别，成为相应类别的文件夹。

### 创建buildSrc/build.gradle—1

首先，先来配置buildSrc文件夹下的build.gradle文件。这个配置比較固定。脚本例如以下所看到的：

```
apply plugin: 'groovy'

dependencies {
    compile gradleApi()
    compile localGroovy()
}
```

### 创建Groovy脚本—2

接下来，在groovy文件夹下，创建一个Groovy类（与Java相似，能够带包名，但Groovy类以.grovvy结尾）。如图所看到的：![这里写图片描写叙述](http://img.blog.csdn.net/20160422172800794)

在脚本中通过实现gradle的Plugin接口，实现apply方法就可以，脚本例如以下所看到的：

```
package com.xys

import org.gradle.api.Plugin
import org.gradle.api.Project

public class MainPluginForBuildSrc implements Plugin<Project> {

    @Override
    void apply(Project project) {
        project.task('testPlugin') << {
            println "Hello gradle plugin in src"
        }
    }
}
```

在如上所看到的的脚本的apply方法中。笔者简单的实现了一个task，命名为testPlugin。运行该Task。会输出一行日志。

#### 创建Groovy脚本的Extension

所谓Groovy脚本的Extension。实际上就是相似于Gradle的配置信息，在主项目使用自己定义的Gradle插件时。能够在主项目的build.gradle脚本中通过Extension来传递一些配置、參数。

创建一个Extension，仅仅须要创建一个Groovy类就可以。如图所看到的：![这里写图片描写叙述](http://img.blog.csdn.net/20160422172832770)

如上所看到的。笔者命名了一个叫MyExtension的groovy类，其脚本例如以下所看到的：

```
package com.xys;

class MyExtension {
    String message
}
```

MyExtension代码很easy，就是定义了要配置的參数变量。后面笔者将详细演示怎样使用。

#### 在Groovy脚本中使用Extension

在创建了Extension之后，须要改动下之前创建的Groovy类来载入Extension。改动后的脚本例如以下所看到的：

```
package com.xys

import org.gradle.api.Plugin
import org.gradle.api.Project

public class MainPluginForBuildSrc implements Plugin<Project> {

    @Override
    void apply(Project project) {

        project.extensions.create('pluginsrc', MyExtension)

        project.task('testPlugin') << {
            println project.pluginsrc.message
        }
    }
}
```

通过project.extensions.create方法，来将一个Extension配置给Gradle就可以。

### 创建resources—3

resources文件夹是标识整个插件的文件夹。其文件夹下的结构例如以下所看到的：

└── resources
└── META-INF
└── gradle-plugins

该文件夹结构与buildSrc一样，是Gradle插件的默认文件夹，不能有不论什么改动。创建好这些文件夹后，在gradle-plugins文件夹下创建——插件名.properties文件，如图所看到的：![这里写图片描写叙述](http://img.blog.csdn.net/20160422173000764)

如上所看到的。这里笔者命名为pluginsrc.properties。在该文件里。代码例如以下所看到的：

```
implementation-class=com.xys.MainPluginForBuildSrc
```

通过上面的代码指定最開始创建的Groovy类就可以。

### 在主项目中使用插件

在主项目的build.gradle文件里，通过apply指令来载入自己定义的插件。脚本例如以下所看到的：

```
apply plugin: 'pluginsrc'
```

当中plugin的名字。就是前面创建pluginsrc.properties中的名字——pluginsrc，通过这样的方式。就载入了自己定义的插件。



#### 配置Extension

在主项目的build.gradle文件里，通过例如以下所看到的的代码来载入Extension：

```
pluginsrc{
    message = 'hello gradle plugin'
}
```

相同，领域名为插件名，配置的參数就是在Extension中定义的參数名。

配置完毕后，就能够在主项目中使用自己定义的插件了，在终端运行gradle testPlugin指令，结果例如以下所看到的：

```
:app:testPlugin
hello gradle plugin
```

## 在本地Repo中使用

在buildSrc中创建自己定义Gradle插件仅仅能在当前项目中使用，因此，对于具有普遍性的插件来说。一般是建立一个独立的Module来创建自己定义Gradle插件。



### 创建Android Library Module

首先。在主项目的工程中，创建一个普通的Android Library Module，并删除其默认创建的文件夹。改动为Gradle插件所须要的文件夹，即在buildSrc文件夹中的全部文件夹，如图所看到的：![这里写图片描写叙述](http://img.blog.csdn.net/20160422173038873)

如上图所看到的。创建的文件与在buildSrc文件夹中创建的文件都是一模一样的，仅仅是这里在一个自己定义的Module中创建插件而不是在默认的buildSrc文件夹中创建。



### 部署到本地Repo

由于是通过自己定义Module来创建插件的。因此。不能让Gradle来自己主动完毕插件的载入。须要手动进行部署，所以，须要在插件的build.gradle脚本中添加Maven的配置，脚本例如以下所看到的：

```
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    compile gradleApi()
    compile localGroovy()
}

repositories {
    mavenCentral()
}

group='com.xys.plugin'
version='2.0.0'
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: uri('../repo'))
        }
    }
}
```

相比buildSrc中的build.gradle脚本，这里添加了Maven的支持和uploadArchives这样一个Task，这个Task的作用就是将该Module部署到本地的repo文件夹下。

在终端中运行gradle uploadArchives指令，将插件部署到repo文件夹下，如图所看到的：![这里写图片描写叙述](http://img.blog.csdn.net/20160422173056476)

当插件部署到本地后，就能够在主项目中引用插件了。

当插件正式公布后，能够把插件像其他module一样公布到中央库，这样就能够像使用中央库的库项目一样来使用插件了。

### 引用插件

在buildSrc中。系统自己主动帮开发人员自己定义的插件提供了引用支持。但自己定义Module的插件中。开发人员就须要自己来加入自己定义插件的引用支持。

在主项目的build.gradle文件里。加入例如以下所看到的的脚本：

```
apply plugin: 'com.xys.plugin'

buildscript {
    repositories {
        maven {
            url uri('../repo')
        }
    }
    dependencies {
        classpath 'com.xys.plugin:plugin:2.0.0'
    }
}
```

当中。classpath指定的路径，就是相似compile引用的方式。即——插件名:group:version
配置完毕后，就能够在主项目中使用自己定义的插件了，在终端运行gradle testPlugin指令，结果例如以下所看到的：

```
:app:testPlugin
Hello gradle plugin
```

> 假设不使用本地Maven Repo来部署，也能够拿到生成的插件jar文件。拷贝到libs文件夹下，通过例如以下所看到的的代码来引用：
>
> ```
> classpath fileTree(dir: 'libs', include: '\*.jar') // 使用jar
> ```

參考：https://docs.gradle.org/current/userguide/custom_plugins.html