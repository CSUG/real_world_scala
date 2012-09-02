# 使用SBT构建Scala应用

## SBT简介

SBT是Simple Build Tool的简称，如果读者使用过Maven，那么可以简单将SBT看做是Scala世界的Maven，虽然二者各有优劣，但完成的工作基本是类似的。

虽然Maven同样可以管理Scala项目的依赖并进行构建， 但SBT的某些特性却让人如此着迷，比如：

* 使用Scala作为DSL来定义build文件（one language rules them all）;
* 通过触发执行(trigger execution)特性支持持续的编译与测试；
* 增量编译；
* 可以混合构建Java和Scala项目；
* 并行的任务执行；
* 可以重用Maven或者ivy的repository进行依赖管理；

等等这些，都是SBT得以在Scala的世界里广受欢迎的印记。

SBT的发展可以分为两个阶段， 即SBT_0.7.x时代以及SBT_0.10.x以后的时代。

目前来讲， SBT_0.7.x已经很少使用， 大部分公司和项目都已经迁移到0.10.x以后的版本上来，最新的是0.12版本。 0.10.x之后的版本build定义采用了新的Settings系统，与最初0.7.x版本采用纯Scala代码来定义build文件大相径庭，虽然笔者在迁移之前很抵触（因为0.7.x中采用Scala定义build文件的做法可以体现很好的统一性），但还是升级并接纳了0.10.x以后的版本，并且也逐渐意识到， 虽然新的版本初看起来很复杂，但一旦了解了其设计和实现的哲学跟思路，就会明白这种设计可以更便捷的定义build文件。而且可选的build文件方式也同样运行采用Scala代码来定义，即并未放弃统一性的思想。

以上是SBT的简单介绍，如果读者已经急于开始我们的SBT之旅，那么让我们先从SBT的安装和配置开始吧！

## SBT安装和配置

SBT的安装和配置可以采用两种方式，一种是所有平台都通用的安装配置方式，另一种是跟平台相关的安装和配置方式，下面我们分别对两种方式进行详细介绍。

### 所有平台通用的安装配置方式
所有平台通用的安装和配置方式只需要两步：

1. 下载sbt boot launcher
	- 本书采用最新的sbt0.12，其下载地址为<http://typesafe.artifactoryonline.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/0.12.0/sbt-launch.jar>；  
2. 创建sbt启动脚本（启动脚本是平台相关的）
	- 如果是Linux/Unit系统，创建名称为sbt的脚本，并赋予其执行权限，并将其加到PATH路径中； sbt脚本内容类似于
``java -Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=384M -jar `dirname $0`/sbt-launch.jar "$@"``， 可以根据情况调整合适的java进程启动参数；
	- 如果是Windows系统，则创建sbt.bat命令行脚本，同样将其添加到PATH路径中。 脚本内容类似于``set SCRIPT_DIR=%~dp0 \n
java -Xmx512M -jar "%SCRIPT_DIR%sbt-launch.jar" %*``

以上两步即可完成sbt的安装和配置。

### 平台相关的安装配置方式
笔者使用的是Mac系统，安装sbt只需要执行``brew install sbt``即可（因为我已经安装有homebrew这个包管理器），使用macport同样可以很简单的安装sbt - ``sudo port install sbt``;

如果读者使用的是Linux系统，那么这些系统通常都会有相应的包管理器可用，比如yum或者apt，安装和配置sbt也同样轻松，只要简单的运行``yum install sbt`` 或者 ``apt-get install sbt``命令就能搞定(当然，通常需要先将有sbt的repository添加到包管理器的列表中)；

Windows的用户也可以偷懒，只要下载MSI文件直接安装，MSI文件下载地址为<http://scalasbt.artifactoryonline.com/scalasbt/sbt-native-packages/org/scala-sbt/sbt/0.12.0/sbt.msi>。

以上方式基本上囊括三大主流操作系统特定的安装和配置方式，其它特殊情况读者可以酌情处理 ^_^

## SBT基础篇

既然我们已经安装和配置好了SBT，那就让我们先尝试构建一个简单的Scala项目吧！

### Hello, SBT

在SBT的眼里， 一个最简单的Scala项目可以极简到项目目录下只有一个.scala文件，比如HelloWorld.scala:

```scala
object HelloWorld{
  def main(args: Array[String]) {
    println("Hello, SBT")
  }
}
```

假设我们HelloWorld.scala放到hello目录下，那么可以尝试在该目录下执行：
<pre>
$ sbt   
> run  
[info] Running HelloWorld   
Hello, SBT   
[success] Total time: 2 s, completed Sep 2, 2012 7:54:58 PM
</pre>
怎么样，是不是很简单那？ （画外音： 这岂止是简单，简直就是个玩具嘛，有啥用嘛？！ 来点儿实在的行不？）

好吧， 笔者也承认这太小儿科了，所以，我们还是来点儿"干货"吧！

> NOTE: 以上实例简单归简单，但可不要小看它哦，你可知道笔者开始就因为忽略了如此简单的小细节而"光阴虚度"？ 该实例的项目目录下，没有定义任何的build文件，却依然可以正确的执行sbt命令， 实际上， 即使在一个空目录下执行sbt命令也是可以成功进入sbt的console的。 所以，只要了解了sbt构建的这个最低条件，那么，当你无意间在非项目的根目录下执行了相应sbt命令而出错的时候，除了检查build文件的定义，另外要注意的就是，你是否在预想的项目根目录下面执行的sbt命令！

### SBT项目工程结构详解

一般意义上讲，SBT工程项目的目录结构跟Maven的很像， 如果读者接触过Maven，那么可以很容易的理解如下内容。

一个典型的SBT项目工程结构如下图所示：

![SBT项目工程结构图](images/sbt_project_structure.png)

#### src目录详解

Maven用户对src目录的结构应该不会感到陌生，下面简单介绍各个子目录的作用。

* src/main/java目录存放Java源代码文件
* src/main/resources目录存放相应的资源文件
* src/main/scala目录存放Scala源代码文件
* src/test/java目录存放Java语言书写的测试代码文件
* src/test/resources目录存放测试起见使用到的资源文件
* src/test/scala目录存放scala语言书写的测试代码文件

#### build.sbt详解

读者可以简单的将build.sbt文件理解为Maven项目的pom.xml文件，它是build定义文件。 SBT运行使用两种形式的build定义文件，一种就是放在项目的根目录下，即build.sbt， 是一种简化形式的build定义； 另一种放在project目录下，采用纯Scala语言编写，形式更加复杂，当然，也更完备，更有表现力。

我们暂时先介绍build.sbt的定义格式，基于scala的build定义格式我们稍后再细说。

一个简单的build.sbt文件内容如下：

```scala
name := "hello"      // 项目名称

organization := "xxx.xxx.xxx"  // 组织名称

version := "0.0.1-SNAPSHOT"  // 版本号

scalaVersion := "2.9.2"   // 使用的Scala版本号


// 其它build定义
```

其中， name和version的定义是必须的，因为如果想生成jar包的话，这两个属性的值将作为jar包名称的一部分。 

build.sbt的内容其实很好理解，可以简单理解为一行代表一个键值对(Key-Value Pair)，各行之间以空行相分割。

当然，实际情况要比这复杂，需要理解SBT的Settings引擎才可以完全领会， 以上原则只是为了便于读者理解build.sbt的内容。

除了定义以上项目相关信息，我们还可以在build.sbt中添加项目依赖：

```scala
// 添加源代码编译或者运行期间使用的依赖
libraryDependencies += "ch.qos.logback" % "logback-core" % "1.0.0"

libraryDependencies += "ch.qos.logback" % "logback-classic" % "1.0.0"

// 或者

libraryDependencies ++= Seq(
                            "ch.qos.logback" % "logback-core" % "1.0.0",
                            "ch.qos.logback" % "logback-classic" % "1.0.0",
                            ...
                            )

// 添加测试代码编译或者运行期间使用的依赖
libraryDependencies ++= Seq("org.scalatest" %% "scalatest" % "1.8" % "test") 
```

甚至于直接使用ivy的xml定义格式:

```scala
ivyXML :=
  <dependencies>
    <dependency org="org.eclipse.jetty.orbit" name="javax.servlet" rev="3.0.0.v201112011016">
        <artifact name="javax.servlet" type="orbit" ext="jar"/>
    </dependency>
    <exclude module="junit"/>
    <exclude module="activation"/>
    <exclude module="jmxri"/>
    <exclude module="jmxtools"/>
    <exclude module="jms"/>
    <exclude module="mail"/>
  </dependencies>
```

在这里，我们排除了某些不必要的依赖，并且声明了某个定制过的依赖声明。

当然， build.sbt文件中还可以定义很多东西，比如添加插件，声明额外的repository，声明各种编译参数等等，我们这里就不在一一赘述了。

#### project目录即相关文件介绍

project目录下的几个文件实际上都是非必须存在的，可以根据情况添加。

__build.properties__文件声明使用的要使用哪个版本的SBT来编译当前项目， 最新的sbt boot launcher可以能够兼容编译所有0.10.x版本的SBT构建项目，比如如果我使用的是0.12版本的sbt，但却想用0.11.3版本的sbt来编译当前项目，则可以在build.properties文件中添加`sbt.version=0.11.3`来指定。 默认情况下，当前项目的构建采用使用的sbt boot launcher对应的版本。

__plugins.sbt__文件用来声明当前项目希望使用哪些插件来增强当前项目使用的sbt的功能，比如像assembly功能，清理ivy local cache功能，都有相应的sbt插件供使用， 要使用这些插件只需要在plugins.sbt中声明即可，不用自己去再造轮子：

```scala
resolvers += Resolver.url("git://github.com/jrudolph/sbt-dependency-graph.git")

resolvers += "sbt-idea-repo" at "http://mpeltonen.github.com/maven/"

addSbtPlugin("com.github.mpeltonen" % "sbt-idea" % "1.1.0")

addSbtPlugin("net.virtual-void" % "sbt-dependency-graph" % "0.6.0")
```

在笔者的项目中， 使用sbt-idea来生成IDEA IDE对应的meta目录和文件，以便能够使用IDEA来编写项目代码； 使用sbt-dependency-graph来发现项目使用的各个依赖之间的关系；

为了能够成功加载这些sbt插件，我们将他们的查找位置添加到resolovers当中。有关resolvers的内容，我们后面将会详细介绍，这里注意一个比较有趣的地方就是，sbt支持直接将相应的github项目作为依赖或者插件依赖，而不用非得先将相应的依赖或者插件发布到maven或者ivy的repository当中才可以使用。

#### 其它

以上目录和文件通常是在创建项目的时候需要我们创建的，实际上， SBT还会在编译或者运行期间自动生成某些相应的目录和文件，比如SBT会在项目的根目录下和project目录下自动生成相应的target目录，并将编译结果或者某些缓存的信息置于其中， 一般情况下，我们不希望将这些目录和文件记录到版本控制系统中，所以，通常会将这些目录和文件排除在版本管理之外。 

比如， 如果我们使用git来做版本控制，那么就可以在.gitignore中添加一行`"target/"`来排除项目根目录下和project目录下的target目录及其相关文件。


> TIPS
> 
> 在sbt0.7.x时代， 我们只要创建项目目录，然后在项目目录下敲入sbt，则应该创建哪些需要的目录和文件就会由sbt自动为我们生成， 而sbt0.10之后，这项福利就没有了。 所以，刚开始，我们可能会认为要很苦逼的执行一长串命令来生成相应的目录和文件：
> <pre>
	$ touch build.sbt
	$ mkdir src
	$ mkdir src/main
	$ mkdir src/main/java
	$ mkdir src/main/resources
	$ mkdir src/main/scala
	$ mkdir src/test
	$ mkdir src/test/java
	$ mkdir src/test/resources
	$ mkdir src/test/scala
	$ mkdir project
	$ ...
</pre>
> 如果是Maven的用户，是不是很想念Maven的Archetype特性那？！ 
> 其实， SBT为我们关了一扇窗，却开了另一道门， 我们可以使用giter8来自动化以上步骤。
> giter8可以自动从github上抓取.g8项目模板，并自动在本地生成相应的项目结构， 比如笔者在github上创建了xsbt.g8项目模板，则直接执行`"g8 fujohnwang/xsbt"`就可以在本地自动生成一个sbt的项目。 有关giter8的更多信息可参考<https://github.com//giter8>.

### SBT的使用

### SBT的依赖管理





## SBT进阶篇

### .scala形式的build定义
### 	自定义SBT Task
### 	SBT插件
### 	多模块工程管理(Multi-Module Project)





## SBT扩展篇 - 使用SBT创建可自动补全的命令行应用程序
	sbt boot launcher + sbt0.12 commands framework





