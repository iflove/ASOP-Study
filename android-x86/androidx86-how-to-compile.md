---
title: 基于 android-x86-4.4-r5 分支编译构建live cdrom iso映像
tags: Android x86
---



## 目录

[TOC]


## 在Android-x86树上选择分支

这里我选择 `android-x86-4.4-r5` 分支

## 建立一个Linux构建环境

> Android x86-4.4-r5版本在最近版本的Ubuntu LTS（16.04）64位上进行测试，具有所需的构建工具。
> 在下载和构建Android-x86源代码之前，请确保您的系统满足以下要求：

- 至少100GB的可用磁盘空间
- Python 2.6 - 2.7，你可以从python.org下载。
- GNU Make 3.81 - 3.82，你可以从gnu.org下载，
- 在Android开源项目（AOSP）中构建Android的4.4分支需要JDK 6
- Git 1.7或更新。你可以在git-scm.com找到它。


**请注意 :** 对于**kitkat-x86**或更高版本，64位环境是**必需**的。

#### 安装JDK

1. 从[oracle jdk-6u31](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase6-419409.html#jdk-6u31-oth-JPR)下载Java JDK 。
2. 点击**接受许可协议**
3. 点击**jdk-6u31-linux-x64.bin**
4. 用您的Oracle帐户登录到Oracle.com
5. 将JDK下载到您的**〜/ Downloads**目录
6. 下载后，打开一个终端，然后输入以下命令。

```shell
cd ~/Downloads
chmod +x jdk-6u31-linux-x64.bin
./jdk-6u31-linux-x64.bin
```



根据生成jdk 目录配置环境变量, 执行命令: `vi ~/.bashrc`

```bash
export JAVA_HOME=/home/jfn/androidDevelop/jdk1.6.0_31 #jdk的主目录
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$ANDROID_SDK/platform-tools:$PATH
```

使jdk环境变量生效, 执行 `source ~/.bashrc`

查看jdk版本, 执行 `java -version、javac –version`

#### 安装make-3.81

1.下载并解压make-3.81.tar.bz2，进入make-3.81目录并执行 `./configure`

2.执行完后make-3.81目录会多出一个build.sh文件，执行`./build.sh` 即可得到make文件

3.验证编译出来的make是不是我们想要的3.81版本。`make -v`

4.替换系统原有的make。（记得备份原有文件。）

```bash
cd /usr/bin/
sudo mv ./make ./make.backup
mv ~/make-3.81/make ./
```



#### **安装所需的软件包**（Ubuntu 16.04）

您将需要一个64位版本的Ubuntu。推荐Ubuntu 16.04。

```bash
sudo apt-get install git-core gnupg flex bison gperf build-essential \
   zip curl zlib1g-dev gcc-multilib g ++  -  multilib libc6-dev-i386 \
   lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
   libgl1-mesa-dev libxml2-utils xsltproc unzip squashfs-tools python-mako
```

你还需要安装OpenSSL相应软件包。OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。

```
sudo apt-get install apache2      ##安装Apache
sudo apt-get install openssl      ##安装openssl
sudo apt-get install libssl-dev    ##安装openssl开发库
sudo apt-get install bless      ##编辑器使用 bless 十六进制编辑器,需预先安装
```



## 获取 Android 源码

[Android x86 开源项目](http://www.android-x86.org/) 网站提供了，获取方法如下:

​	http://www.android-x86.org/getsourcecode

下面介绍了 `repo` 工具的安装,以及基本用法。

### Installing Repo

1.确保您的主目录中有一个bin /目录，并将其包含在您的路径中：

```shell
mkdir ~/bin
PATH=~/bin:$PATH
```

2.下载Repo工具并确保它是可执行的：

```shell
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

3.通过 `repo init` 将当前目录初始化为工作区,

```shell
mkdir android-x86
cd android-x86
repo init -u git://git.osdn.net/gitroot/android-x86/manifest -b android-x86-4.4-r5
```

NOTE:由于网络原因国内的访问不了google的仓库，需要把 REPO_URL = '[https://gerrit.googlesource.com/git-repo' 改为](https://gerrit.googlesource.com/git-repo'%E6%94%B9%E4%B8%BA) REPO_URL = 'git://git.omapzoom.org/git-repo.git' 。另外还需科学上网download 源码，这里推荐

`Shadowsocks-Qt5` , 配置好网络代理后，还需要设置git 的代理

```bash
git config --global https.proxy https://127.0.0.1:1080
```

4.使用身份验证

注意: 默认情况下，访问 Android 源代码均为匿名操作。为了防止服务器被过度使用，每个 IP 地址都有一个相关联的配额。正常来说是下载源码不成功的，请参考官方ASOP  [下载源代码](https://source.android.com/setup/downloading)

5.repo init 完成

成功初始化后,会在当前工作目录下创建 `.repo` 文件夹（隐藏的）,并从 `-u` 参数所指定的repository 下载一个 `mainffest.xml` 文件到`.repo` 文件夹。清单文件定义了Android源代码中所有的git项目的清单。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>

  <remote  name="aosp"
           fetch=".." />
  <default revision="refs/tags/android-7.1.1_r28"
           remote="aosp"
           sync-j="4" />

  <project path="build" name="platform/build" groups="pdk,tradefed" >
    <copyfile src="core/root.mk" dest="Makefile" />
  </project>
  ...
</manifest>
```

显而易见，其中每个 project 项都代表一个 git 项目，`name` 属性指定了git 项目名称，`path`属性指定了git 项目将被下载到哪个文件中。`revision` 指定了那个分支的代码。通过 `repo sync` 命令便可以下载代码。`repo sync` 可以接受 -j 参数进行多线程的代码下载以提高下载速度。例：`repo sync -j4` 。

```
repo sync platform/frameworks/base
```

repo sync 也可以接受 git 项目名称作为参数单独下载这个项目的代码。

6.从Android 源代码树默认清单中指定的代码库下载到工作目录

```bash
repo sync
```



## 编译Android源码

### 直接构建一个live cdrom iso映像

```shell
make iso_img TARGET_PRODUCT = android_x86
```

注意: android-x86-4.4-r5 分支 请必须直接按上构建iso 映像

### 编译要求

下载和编译 Android 源代码之前，请先确保您的系统符合官方要求。请参阅[requirements](https://source.android.com/source/requirements)

### 编译错误

编译Android 源码总不会一帆风顺的。根据错误信息提示，少了什么就安装什么。比如：`/bin/bash: xmllint: 未找到命令`   就执行`sudo apt-get install libxml2` 即可，c++ 头文件、Python module 也是如此。另外多查查资料。

#### hybrid-v35-nodebug-pcoem-6_30_223_271.tar.gz  文件缺失错误

错误日志

```bash
gzip: stdin: unexpected end of file
tar: Child returned status 1
tar: Error is not recoverable: exiting now
make: *** [kernel/drivers/net/wireless/wl/Makefile] Error 2
make: *** 正在等待未完成的任务....
make[2]: Entering directory `/home/jfn/androidDevelop/out/target/product/x86/obj/kernel'
  HOSTCC  scripts/basic/fixdep
  GEN     ./Makefile
  HOSTCC  scripts/kconfig/conf.o
  SHIPPED scripts/kconfig/zconf.tab.c
  SHIPPED scripts/kconfig/zconf.lex.c
  SHIPPED scripts/kconfig/zconf.hash.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
scripts/kconfig/conf --olddefconfig Kconfig
.config:6553:warning: override: reassigning to symbol ANDROID_LOGGER
.config:6554:warning: override: reassigning to symbol ANDROID_INTF_ALARM_DEV
#
# configuration written to .config
#
make[2]: Leaving directory `/home/jfn/androidDevelop/out/target/product/x86/obj/kernel'
make[1]: Leaving directory `/home/jfn/androidDevelop/kernel'

```

由 tar: Error ,而且 /kernel/drivers/net/wireless/wl 下的hybrid-v35-nodebug-pcoem-6_30_223_271.tar.gz 文件长度为0. 打开build.mk 发现了通过 http://www.broadcom.com/docs/linux_sta/ 网站去下载的，而这个链接过期了

下载 [hybrid-v35-nodebug-pcoem-6_30_223_271.tar.gz ](https://pkgs.rpmfusion.org/repo/pkgs/nonfree/broadcom-wl/hybrid-v35-nodebug-pcoem-6_30_223_271.tar.gz/4e75f4cb7d87f690f9659ffc478495f0/) 文件放进去再编译就好了。



## Testing

生成系统镜像在下面目录

```
out/target/product/x86/android_x86.iso
```



## 在 Android Studio中导入 Android 源码

在 `/development/ide` 下分别有eclipse/emacs/intellij 这3个ide 的配置文件。因为源码文件十分多,强烈建议as增加jvm 内存大小。



#### 生成导入AS所需配置文件(*.ipr)

- 编译源码(为了确保生成了.java文件，如R.java；如果编译过，则无需再次编译)

- 检查out/host/linux-x86/framework/目录下是否有idegen.jar


如果idegen.jar不存在, 执行:

```bash
mmm development/tools/idegen/
development/tools/idegen/idegen.sh
```

等待出现类似下面的结果:

```shell
Read excludes: 5ms
Traversed tree: 44078ms
```

这时会在源码的根目录下生成android.ipr和android.iml两个IntelliJ IDEA(AS是基于IntelliJ IDEA社区版开发的)的配置文件.

Tips：AS在导入代码时比较慢，建议先修改android.iml，将自己用不到的代码exclude出去.可以仿照过滤.repo文件夹的语法,如:

```
<excludeFolder url="file://$MODULE_DIR$/.repo" />
<excludeFolder url="file://$MODULE_DIR$/abi" />
<excludeFolder url="file://$MODULE_DIR$/art" />
...
```

还可以通过as 的module settings - dependents 删除掉所有不需要的module-library项

这样在导入时就会跳过abi和art文件夹.过滤的越多，AS的处理速度就会越快.最后在AS中打开源码根目录下新生成的android.ipr



#### 解决源码中跳转错误问题

要为当前工程设置正确的SDK和JDK，需要Android API 19 Platform，jdk 则无所谓，因为libcore 目录下包括了jdk 的源码。添加一个jdk 接着把classpath 所有项remove 掉，接着Android API 19 Platform 选择空的jdk.如下图
![步骤1](http://img.blog.csdn.net/20180207145631611?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ1NETm5v/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

点击Project
![步骤2](http://img.blog.csdn.net/20180207145747921?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ1NETm5v/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

点击Modules, 选择相应的代码模块，比如frameworks、packages 等源码目录，且把它们移动到最前优先从这两个文件夹下查找，而不是在Android.jar中查找
![这里写图片描述](http://img.blog.csdn.net/20180207145838877?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ1NETm5v/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
NOTE: Empty Library 不要紧的。

#### DEBUG源码
在'Modules'中添加'Android Framework'来让AS将它作为一个Android工程，从而方便我们调试代码.
![这里写图片描述](http://img.blog.csdn.net/20180207150331651?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ1NETm5v/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


在代码中加断点，然后选择'Run'->'Attach debugger to Android process'或者直接点击ToolBar 上的Attach debugger图标。在弹出的选择进程(Choose Process)对话框中，勾选显示所有进程，选择要DEBUG的代码所在的进程，点击OK即可.


