# <p style="text-align:left;">安装Groovy</p>

### 1.下载
在这块下载区域,您能够下载发布版(二进制版和源码版)、窗口安装程序和Groovy文档。
 
 为了快速便捷的在Mac OSX,Linux或者Cygwin系统上面使用,您能够使用[SDKMAN!](https://sdkman.io/)(软件开发工具管理工具)下载和设置任何您选择的groovy版本。您能够在下面找到基本的<a href="#SDKMAN">说明</a>。
 
### 1.1.稳定版
 
* 下载zip:[二进制发布版](https://bintray.com/artifact/download/groovy/maven/apache-groovy-binary-2.5.2.zip)|[源码发布版](https://bintray.com/artifact/download/groovy/maven/apache-groovy-src-2.5.2.zip)
 
 * 下载文档: [JavaDoc和在线压缩文档](https://bintray.com/artifact/download/groovy/maven/apache-groovy-docs-2.5.2.zip)
 
 * 组合的二进制/源码/文档资源包:[发布的资源包](https://bintray.com/artifact/download/groovy/maven/apache-groovy-sdk-2.5.2.zip)
 
 您能在[发布发布笔记](http://groovy-lang.org/releasenotes/groovy-2.5.html)或者在[发布日志](http://groovy-lang.org/changelogs/changelog-2.5.2.html)里了解更多
 
 如果您打算使用动态执行支持,[请阅读这些说明](http://docs.groovy-lang.org/latest/html/documentation/invokedynamic-support.html)
 
### 1.2.快照版本
 
 对于那些想要测试最终版本的Groovy和想要挑战自己的人,你能够使用我们的[快照版本](https://oss.jfrog.org/oss-snapshot-local/org/codehaus/groovy)。一旦一次创建在我们的持续集成服务器上面成功了一个快照版本就被发布到 Artifactory的OSS快照仓库.。

### 1.3Groovy运行需求
 
Groovy2.5需要Java 6以上并且全面支持Java 8。当您使用Java 9快照版本的时候现阶段一些方面有一些问题。groovy-nio模块需要Java 7以上。使用Groovy的动态执行特点需要Java 7以上但是我们推荐使用Java 8。

为了确定Groovy不同的发行版需要的Java版本Groovy CI 服务器也是非常值得一看的。这些测试集(差不多将近有10000个)通过所有主要的Java版本运行在当前的被推荐使用的groovy上。

### 2.Maven仓库

如果您希望将groovy嵌入到您的引用中,您可能仅仅喜欢指向您最喜欢的maven仓库或者[jcenter maven 仓库](https://oss.jfrog.org/oss-release-local/org/codehaus/groovy)(我大天朝的网络并不推荐,使用大家都懂得除非你有vpn:scream:)

### 2.1稳定发布版

|     Gradle        |        Maven              |     说明              |
|:-----------------:|:--------------------:|:-----------------:|
|'org.codehaus.groovy:groovy:2.5.2'|<groupId>org.codehaus.groovy</groupId> <artifactId>groovy</artifactId> <version>2.5.2</version>|只是没有模块的核心groovy代码(详情见下面)|
|'org.codehaus.groovy:groovy-$module:2.5.2'|<groupId>org.codehaus.groovy</groupId> <artifactId>groovy-$module</artifactId> <version>2.5.2</version>|"$module"代表不同的可选的groovy模块:"ant","bsf","console","docgenerator","groovydoc","groovysh","jmx","json", "jsr223", "servlet", "sql", "swing", "test", "testng" and "xml"。比如: <artifactId>groovy-sql</artifactId>|
|'org.codehaus.groovy:groovy-all:2.5.2'|<groupId>org.codehaus.groovy</groupId> <artifactId>groovy-all</artifactId> <version>2.5.2</version>|核心代码加上所有的模块。可选的依赖被标记为可选的。为了使用Groovy的一些特性您可能需要引入一些可选的依赖。比如:AntBuilder, GroovyMBeans等等|

如果您想要使用[动态执行版本](http://docs.groovy-lang.org/latest/html/documentation/invokedynamic-support.html)(JDK 7字节码新增加的功能)的jar包只需要给gradle增加":indy",Maven增加"<classifier>indy</classifier>"。

### <a style="color:black;" name="SDKMAN"> 3.SDKMAN!(软件开发工具管理工具)</a>

这个工具使Groovy非常简单的在命令行平台(Mac OSX, Linux, Cygwin, Solaris 或者 FreeBSD)下安装。

简单的代开一个终端(命令行)和输入:

``` bash
$ curl -s get.sdkman.io | bash
```

跟着屏幕上的说明去完成安装。

打开一个新的终端(命令行)或者粘贴命令:

``` bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

安装最新的Groovy

``` bash
$ sdk install groovy
```

安装完成了之后它就会成为你的默认版本,使用下面的命令测试它


``` bash
$ groovy -version
``` 
这就是所有了!

### 4.其他获得Groovy的方法

#### 4.1.使用Mac OS X安装

##### 4.1.1.MacPorts

如果您的系统是MacOS并且安装了[MacPorts](http://www.macports.org/),您能够运行:

``` bash
sudo port install groovy
```

##### 4.1.2.Homebrew

如果您的系统是MacOS并且安装了[Homebrew](https://mxcl.github.com/homebrew),您能够运行:

``` bash
brew install groovy
```

#### 4.2.在Windows上安装

如果您的系统是Windows,您能够使用[NSIS Windows installer](http://docs.groovy-lang.org/latest/html/documentation/TODO-Windows+NSIS-Installer)。

#### 4.3.其他发行版

您能在[这个网站](https://bintray.com/groovy/maven)下载其他Groovy发行版。

#### 4.4.源代码

如果您喜欢挑战自己,你也能够从GitHub上面获取[源代码](https://github.com/apache/groovy)。

#### 4.5.IDE插件

如果您是一个IDE用户,您能够仅仅获取最新版的[IDE插件](http://docs.groovy-lang.org/latest/html/documentation/tools-ide.html)并阅读插件安装说明进行安装

### 5.安装二进制版

这些说明描述了怎样安装Groovy二进制发行版。

* 首先,下载一个Groovy二进制发行版并将他解压缩到您的本地文件系统的任意位置。

* 配置一个GROOVY_HOME环境变量并设置为你解压的发型版的目录。

* 增加GROOVY_HOME/bin(请根据自己的操作系统自行修改GROOVY_HOME,例如:windows为%GROOVY_HOME%,类Unix系统为$GROOVY_HOME)到您的PATH环境变量中。

 * 设置您的JAVA_HOME变量指向您的JDK。在OS X这是/Library/Java/Home,在其他unixes系统上他经常是/usr/java,这里不一一列举了。如果您已经安装了类似Ant或者Maven的工具您可能已经完成了这个步骤。
 
 您现在应该已经装好了Groovy。您能够在命令行shell里粘贴下面的命令测试
 
 ``` bash
 groovysh
 ```
 
 这是一个交互式的shell编程环境您能够直接在里面粘贴Groovy语句块。或者您也可以运行Swing交互式控制台:
 
  ``` bash
 groovyConsole
 ```
 
 运行指定的Groovy脚本:
 
   ``` bash
groovy SomeScript
 ```
