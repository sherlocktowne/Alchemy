## VLC Android版编译笔记
VLC是开源的跨平台视频播放器，但是由于依赖众多、开发语言是C语言，上手并不容易。本文主要记录了VLC Android版的整个编译过程，以备后用。

在开始学习以及编译VLC之前，有必要对VLC的整体框架建立基本的认识。我们通常说的VLC，其实包括了多个部分，包括：libVLCCore、modules以及vlc应用程序等，详细的可以参考这篇博文[The architecture of VLC media framework](https://web.archive.org/web/20141204234622/http://www.enjoythearchitecture.com/vlc-architecture.html)。

好了，废话不多说了，回到正题。VLC的Android编译有对应的[官方文档](https://wiki.videolan.org/AndroidCompile/)，但是不幸的是，直接按照官方文档是编译不过的。下面说说具体的编译步骤吧。

### 准备Ubuntu编译环境
理论上Mac OSX也是可以编译通过的，不过为了省事避免遇到各种奇奇怪怪的问题，还是先基于官方推荐的Ubuntu系统吧。编译文档并没有对Ubuntu系统版本作特别的说明，本人使用的版本是16.04LTS，估计14.04LTS应该也是OK的。

#### 安装必要的编译工具：

	$ sudo apt-get install automake ant autopoint cmake build-essential libtool \
		patch pkg-config protobuf-compiler ragel subversion unzip git

### 安装Protobuf3依赖库
VLC依赖protobuf，当前的最新代码必须安装protobuf3，否则编译的时候会报”Unrecognized syntax identifier "proto3””的错误。

首先需要下载[Protobuf源码](https://github.com/google/protobuf/releases/download/v3.0.0/protobuf-cpp-3.0.0.zip)。下载完成后解压zip文件：

	$ unzip protobuf-cpp-3.0.0.zip

安装编译工具：
	$ sudo apt-get install autoconf automake libtool curl make g++ unzip

开始编译&安装：

	$ ./configure
	$ make
	$ make check
	$ sudo make install
	$ sudo ldconfig # refresh shared library cache.

#### 安装Android编译环境
##### 下载安装SDK、NDK
下载安装Android SDK，只需要下载SDK [command line tools](https://dl.google.com/android/repository/tools_r25.2.3-linux.zip)，不必下载包含Android Studio的完整版本。下载完成后执行以下命令完成解压以及SDK的安装：

	$ unzip tools_r25.2.3-linux.zip
	$ cd tools_r25.2.3-linux
	$ android list sdk —all
	$ android update sdk -u --all --filter 1,2,3,5,… #数字对应需要安装的package

下载安装[Android NDK](https://dl.google.com/android/repository/android-ndk-r13b-linux-x86_64.zip)，注意VLC当前的最新代码需要安装r13以上版本。下载完成后解压到适当的目录，这里不再赘述。

##### 配置环境变量
虽说bshell可以通过设置~/.bash_profile来设置持久化的环境变量，但是我设置过后依然无法被vlc的编译脚本识别，不知何故。后来只能曲线救国，通过设置/etc/profile后才肯好好工作。具体如下：

	$ vim /etc/profile

在profile文件中加入下面的环境变量（注意SDK、NDK需要根据实际的路径进行设置）：

	export ANDROID_SDK=~/Android/SDK
	export ANDROID_NDK=~/Andorid/NDK/ndk
	export PATH=$PATH:$ANDROID_SDK/platform-tools
	export PATH=$PATH:$ANDROID_SDK/tools

编辑完成后保存、退出。执行下面命令让环境变量立即生效：

	$ source /etc/profile

#### 准备编译
首先，我们需要下载VLC Android源代码：

	$ git clone https://code.videolan.org/videolan/vlc-android.git

下载完成后，进入源代码根目录，开始编译：
	
	$ ./compile.sh

一切顺利的话，一杯咖啡的功夫就可以编译完成了。

#### 运行、调试Android程序
由于我的Ubuntu系统没有安装Android Studio，无法进行Android的调试，因此在编译完成后，我把vlc-android目录拷回了Mac OSX系统，以便用Android Studio进行调试。其实可以尝试的更好的方法是，源代码放在Mac OSX，将源代码mount到Ubuntu系统，然后在Ubuntu系统直接编译就可以同步在Mac OSX看到最终的编译结果了。记得之前在Windows通过编译服务器编译Webkit的时候使用的就是这种方法，在局域网内速度是可以接受的。

用Android Studio导入vlc-android工程（对应vlc-android根目录）后，直接编译会发现编译错误，大致如下:
	
	* What went wrong:
	Execution failed for task ':libvlc:buildDebugARMv7'.
	> Process 'command './compile-libvlc.sh'' finished with non-zero exit value 1


原因是gradle脚本调用了编译Native Library的shell脚本，但是我们并没有在Mac OSX进行对应的配置。因为我们这里只是想把vlc-android的工程运行起来，解决起来就很简单了：打开vlc-android-vlc-android目录下的build.gradle文件，注释掉以下代码就可以了。

	tasks.whenTaskAdded { task ->     
		if (task.name.startsWith('assemble')) {         
			/*         
			if (task.name.endsWith('ARMv7Debug'))             
				task.dependsOn(":libvlc:buildDebugARMv7")         
			else if (task.name.endsWith('ARMv8Debug'))             
				task.dependsOn(":libvlc:buildDebugARM64")       
			else if (task.name.endsWith('X86Debug'))             
				task.dependsOn(":libvlc:buildDebugx86")         
			else if (task.name.endsWith('X86_64Debug'))             
				task.dependsOn(":libvlc:buildDebugx86_64")         
			else if (task.name.endsWith('MIPSDebug'))             
				task.dependsOn(":libvlc:buildDebugMIPS")         
			else if (task.name.endsWith('MIPS64Debug'))             
				task.dependsOn(":libvlc:buildDebugMIPS64")             
			*/     
		}
	 }

好了，到此就可以顺利的调试和运行VLC Android代码了。

熟悉开源代码的第一步，就是先让它跑起来，看看它能干什么，现在就只是完成了这第一步，真正要熟悉VLC的代码，路还很长。。。有空再写写吧。
















