# 一. Android Killer 环境配置
1. 安装java环境
2. 在android killer 工具的配置中Java->SDK-安装路径中配置Java的jdk的bin路径
3. 从 https://ibotpeaches.github.io/Apktool/ 中下载最新的apktool，再将apktool放到android killer的bin\apktool\apktool目录中
4. 然后修改AndroidKiller根目录下的bin\apktool下的apktool.bat和apktool.ini文件

    ```
    将apktool.bat文件中的：
    java -jar "%~dp0\apktool\ShakaApktool.jar" %1 %2 %3 %4 %5 %6 %7 %8 %9
    中的
    ShakaApktool.jar改成刚刚拷贝过去的apktool的名称，比如：
    java -jar "%~dp0\apktool\apktool_2.5.0.jar" %1 %2 %3 %4 %5 %6 %7 %8 %9
    ```
    ```
    将apktool.ini文件中的：
    path0=ShakaApktool.jar
    也改成刚刚拷贝过去的apktool的名称，比如
    path0=apktool_2.5.0.jar
    ```

# 2. Android Studio调试smali
1. 安装Android Studio，需要安装3.1版本，最好是3.1.4版本的Andrid Studio。因为目前高版本的Android Studio对smalidea的支持不太好。
2. 下载smalidea插件 https://bitbucket.org/JesusFreke/smali/downloads/smalidea-0.05.zip，目前版本的0.05。
3. Andriod Studio中，File->settings->Plugins->install plugin from disk 安装下载好的zip文件，选择整个zip文件安装。
4. 先使用Android Killer 进行反编译，得到smali文件。
5. 使用Android Studio 打开Android Killer反编译得到的工程目录，一般在Android Killer的安装目录下的Project目录中（如：E:\AndroidTools\AndroidKiller_v1.3.1\projects\小米有品\Project）。
6. 配置工程调试：Android Studio中，Run->Edit Configrations->点击加号添加配置->Remote->名字随便取，端口填8700->确定，配置完成。
7. 使用ADB命令以调试模式启动应用程序 
    ```
    adb shell am start -D -n PACKAGE_NAME/ACTIVITY_NAME
    - 其中adb shell表示执行adb的命令
    - am：使用此命令可以从控制台发送启动activity、service或者发送broadca等功能
    - am start：启动一个activity，参数 -D：使用调试。 -n：不清楚
    - PACKAGE_NAME和ACTIVITY_NAME都是从apk打包中的 AndroidManifest.xml中获取。PACKAGE_NAME在第一标签的package="com.xiaomi.youpin"属性中。ACTIVITY_NAME在启动标签中，可以查找 action android:name="android.intent.action.MAIN 找到。在该标签的Activity父标签中的android:name="com.xiaomi.youpin.activity.SplashActivity"属性中。其中.activity.SplashActivity就是ACTIVITY_NAME
    - 如启动小米有品的方法：
    adb shell am start -D -n com.xiaomi.youpin/.activity.SplashActivity
    其中/后面要加. 才能找到对应的activity
    ```
    > 在启动时，可能报错  no devices/emulators found。表示模拟器或者真机没有链接上电脑。
    - 确定模拟器或者真机是否打开USB调试
    - 模拟器需要connect命令进行连接：adb connect 127.0.0.1:21503 其中21503是逍遥模拟器的连接端口，不同的模拟器连接端口不同。
8. 查看进程ID，用以端口转发：
    ```
    adb shell "ps | grep xiaomi"
    可以列出小米有品在手机中的进程信息：
    u0_a4     2164  99    1216096 62216 futex_wait b76f7a3c S com.xiaomi.youpin
    获取到小米有品的pid 2164
    ```
    > 在获取进程信息时，如果出现 " 'grep' 不是内部或外部命令，也不是可运行的程序或批处理文件。"这个提示，在adb shell后面的命令都加双引号就行。

9. 端口转发：adb forward tcp:8700 jdwp:2164
设置端口转发，这条命令的含义可以认为是在本地8700端口与手机2164进程之间建立一条通道，当开始调试时，AS连接本地的8700端口，通过这条通道控制程序的运行。其中8700端口是在上面的AS中的工程调试配置中配置的。

    > adb forward 的一些基本操作:
    - adb forward --list查看刚才的执行结果。
        ```
        E:\Environment\ADB>adb forward --list
        127.0.0.1:21503 tcp:8700 jdwp:2164
        ```
    - adb forward --remove tcp:8700 删除建立的转发。
        ```
        E:\Environment\ADB>adb forward --remove tcp:8700
        ```

10. 开始调试：点击Android Sutdio的Run->Debug 'smali' 开启调试。如果出现 “Connected to the target VM, address: 'localhost:8700', transport: 'socket'” 说明配置没问题，如果有问题，很大原因是端口占用问题。

# 三、IDA配置和认识
## 1. IDA简介：
- 交互式反汇编器专业版（Interactive Disassembler Professional）常称其为IDA Pro，或简称为IDA,是总部位于比利时列日市（Liège）的Hex-Rayd公司的一款产品。
- IDA分为标准版(Stdandard)和高级版(Advance)。标准版(Standard)支持二十多种处理器。高级版(Advanced)支持50多种处理器。
- 安装好的IDA分别由64位和32位启动程序。

## 2. IDA安装
    ```
    链接：https://pan.baidu.com/s/1z4yV88h_WMAYYhLAGdyl-Q 
    提取码：1234 
    ```
1. 以安装IDA7.0为例，x64_idapronw_hexarm64w_hexarmw_hexx64w_hexx86w_170914_e723c5648dc3f2f588ab8339ccf62ec0.exe 该程序为安装程序，按照普通安装方式安装即可。
2. 输入password，在ReadMe.txt中放有对应的passwword：qY2jts9hEJGy
3. 安装python2.7，在输入password之后，会让选择安装python2.7。需要勾选安装，安装路径是默认的IDA安装目录下，并且不会自动设置python2.7的环境变量，也就是说不会和已经安装的python产生冲突。
4. 完成安装，会看到32位启动程序和64位启动程序。64位启动程序图标会有64的标志。

## 3. IDA使用
1. 打开IDA，为进入IDA界面提供三种选项，分别是New（新建），Go（运行），Previous（上一个）。
2. 拖入或打开想要逆向的可执行文件，会显示一个Load a new file的界面。

3. 
4. 退出IDA时，会进行文件保存确认，如果需要继续进行分析，将IDA中间数据库打包，下次继续打开就可以进行分析；如果不需要继续分析，选择不要打包，不要存储数据库。
> 注：IDA打开应用程序时，会为其创建一个数据库，后缀为IDB。IDB由4个文件组成，后缀为id0的二叉树形式的数据库，后缀为id1的程序字节标识，后缀为nam的Named窗口的索引信息，后缀为til的给定数据库的本地类型定义的相关信息。一旦IDA为某个可执行程序创建数据库，它本身就不再需要访问这个可执行文件，除非使用IDA的Debug功能。

## 4.IDA调试so
### 1. 附加进程调试
1. 使用adb命令连接上真机或虚拟机：
- 连接模拟器和AS调试Smail中是一样的
- 连接真机：
    - 如果不是虚拟机里，直接插上数据线，打开USB调试。
    - 如果是虚拟机，需要确保宿主机中 VMware USB Arbitration Service 服务运行中。并且在虚拟机设置中的硬件->USB控制器->所有选项勾上。
    - 打开CMD窗口，输入adb shell (要确保配置了adb.exe的环境变量)。等待连接成功，进入adb shell 命令状态：
        ```
        C:\Users\linigtao>adb shell

        dipper:/ $
        ```
    - 此时需要给adb shell 一个root权限：
        ```
        dipper:/ $ su

        dipper:/ #
        ```
2. 安卓中运行android_server
- 找到IDA安装目录下的dbgsrv文件夹下的android_server文件：
    ```
    E:\AndroidTools\IDA\Installed\IDAPro7.0\dbgsrv\android_server
    ```
- 将android_server 通过adb 的 push命令，推送到安卓真机或模拟器中：
    ```
    新打开一个cmd窗口，输入如下命令，将android_server推送到安卓系统中的/data/local/tmp路径下：
    C:\Users\linigtao>adb push E:\AndroidTools\IDA\Installed\IDAPro7.0\dbgsrv\android_server /data/local/tmp

    E:\AndroidTools\IDA\Installed\IDAPro7.0\dbgsrv\android_server: 1 file pushed. 10.8 MB/s (589588 bytes in 0.052s)
    ```
- 修改android_server权限：
    ```
    dipper:/data/local/tmp # chmod 777 android_server

    chmod 777 android_server
    ```
- 直接运行 android_server 文件
    ```
    127|dipper:/data/local/tmp # ./android_server

    ./android_server
    IDA Android 32-bit remote debug server(ST) v1.22. Hex-Rays (c) 2004-2017
    Listening on 0.0.0.0:23946...
    ```
    - 如果出现 bind:Address already in use 说明端口被占用。
    - 启动android_server默认使用23946端口。
    - 可以在运行android_server时指定端口号：
        ```
        1|dipper:/data/local/tmp $ ./android_server -p31928

        IDA Android 32-bit remote debug server(ST) v1.22. Hex-Rays (c) 2004-2017
        Listening on 0.0.0.0:31928...
        ```
    - 解决端口占用除了启动android_server时指定端口，也可以杀死占用这个端口的android_server进程
        - 先 ps | grep android 查看下android_server的pid，注意：如果此命令查看不到android_server相关进程，尝试在ps 后加个 -a 参数：ps -a | grep android
        - kill -9 android_server的pid
3. 端口转发
    ```
    重新打开个cmd
    C:\Users\linigtao>adb forward tcp:31928 tcp:31928
    ```

4. 附加进程调试：

- 启动调试：
    - 如果安卓机上没有安装要调试的apk，则可以使用 adb install 命令安装，也可以手动安装
        ```
        C:\Users\linigtao>adb install E:\data\jisuanqi1\bin\jisuanqi1.apk

        Success
        ```
    - 启动要调试的安卓程序

- 打开IDA，选择Debugger->Attach->Remote ARMLinux/Android debugger 打开远程调试选项框
    - Hostname：目标主机，填127.0.0.1
    - Port：目标端口，如果启动 android_server 时配置了端口，就填配置的端口，如果没有，则用默认的23946
    - Password：为空，不填
    - 点击确定后，会出现一个进程列表，选择要调试的进程进行附加，可以搜索进程关键字，关键字一般是apk名称

> 注意：使用模拟器
### 2. 调试模式启动调试
1. 按照附加进程调试的1、2、3部完成部署
2. 按照：Android Studio调试smali中第7部 使用ADB命令以调试模式启动应用程序中使用adb shell am start 命令以调试模式启动目标程序。
3. 使用IDA附加要调试的进程。打开IDA，选择Debugger->Attach->Remote ARMLinux/Android debugger
    >注意：如果打开远程调试选项框，输入hostname和prot后，进入选择进程列表对话框时，出现了 Bogus or irresponsive remote server错误。首先检查输入的hostname和port是不是正确的。如果是正确的，再确定是否正确转发端口。如果还是正确，再退出运行android_server程序，重新进入adb shell命令，给root权限，也就是在命令行中输入su命令，看到$变成了#即可。再重新运行android_server程序，再尝试。
4. 进入IDA调试界面后，选择Debugger->Debugger Options中,勾选三个选项，确保在必要时能够断下来：
    - Suspend on process entry point    程序入点处中断
    - Suspend on thread start/exit      进程开始/退出时中断
    - Suspend on library load/unload    库加载/卸载时中断
5. 获取目标程序端口，用于执行jdb命令，这里用ddms获取，也有其他方式获取：
    - CMD命令窗口输入DDMS，打开DDMS
    - DDMS中有红色小虫的程序就表示正在被挂起的进程，进程后就是端口
4. 开始运行挂起程序，执行下面命令，会让程序退出Waitting for debugger状态
    ```
    C:\Users\linigtao>jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=8603

    设置未捕获的java.lang.Throwable
    设置延迟的未捕获的java.lang.Throwable
    正在初始化jdb...
    >
    ```
    - 此时，如果端口号错误，会出现报错：无法附加到目标 VM。
    - 运行完命令后，查看ddms，原来红色虫子变为绿色虫子

> 注意：安卓真机大部分是使用arm架构cpu，而市面上的模拟器，如逍遥模拟器，多采用x86架构。所以在调试模拟器时，要注意的是兼容问题。在adb shell 启动android_server步骤时，android_server需要用IDA安装目录中dbgsrv文件夹中的android_x86_server替换。并且在IDA附加进程调试时，需要选择Debugger->Attach->Remote Linux debugger。

# 四、frida环境配置

## 1. frida介绍
- Frida是一款基于python + javascript 的hook框架，适用于android/ios/linux/win/osx等平台。
- Frida的动态代码执行功能，主要是在它的核心引擎Gum中用C语言来实现的。
- Frida运行模式：
    - 注入模式：
        - 大部分情况下，我们都是附加到一个已经运行到进程，或者是在程序启动到时候进行劫持，然后再在目标进程中运行我们的代码逻辑。这种方式就是Frida最常用的使用方式。
        - 注入模式的大致实现思路是这样的，带有GumJS的Frida核心引擎被打包成一个动态连接库，然后把这个动态连接库注入到目标进程中，同时提供了一个双向通信通道，这样你的控制端就可以和注入的模块进行通信了，在不需要的时候，还可以在目标进程中把这个注入的模块给卸载掉。
        - 除了上述功能，Frida还提供了枚举已经安装的App列表，运行的进程列表已经已经连接的设备列表，这里所说的设备列表通常就是frida-server 所在的设备。frida-server 是一个守护进程，通过TCP和Frida核心引擎通信，默认的监听端口是27042。
    - 嵌入模式：
        - 在实际使用的过程中，你会发现在没有 root 过的iOS Android设备上你是没有办法对进程进行注入的，也就是说第一种注入模式失效了，这个时候嵌入模式就派上用场了。
        - Frida提供了一个动态连接库组件 frida-gadget， 你可以把这个动态库集成到你程序里面来使用Frida的动态执行功能。一旦你集成了gadget，你就可以和你的程序使用Frida进行交互，并且使用 frida-trace 这样的功能，同时也支持从文件自动加载Js文件执行JS逻辑。
        - 关于 Gadget 更多功能，请参考原文链接（https://www.frida.re/docs/modes）
    - 预加载模式：自动加载Js文件。
## 2. 安装python环境
- windows的frida目前只支持python2.7和python3.7
- 安装python的IDE工具Pycharm
## 3. 安装frida相关软件
- 这里需要三个相关的软件：
    - frida：是一个python库，里面提供了python的接口，用以使用python开发相应的hook插件
    - frida-tools:frida命令行工具，可以通过命令行进行frida注入等操作
    - frida-server:运行在目标端(android)的服务，必须要有这个服务才能实现frida的hook，并且需要进行端口转发
- 安装frida：
    - cmd中执行：pip install frida
    - 如果慢的话，可以使用国内源安装：pip install frida -i https://pypi.tuna.tsinghua.edu.cn/simple/
    - 安装指定版本的frida：pip install frida==12.9.4
    - 具体的版本可以在github上查看：https://github.com/frida/frida/releases
    - 如果之前装了frida，先要卸载之前的版本：pip uninstall frida
- 安装frida-tools:
    - cmd中执行：pip install frida-tools
    - 如果慢的话，可以使用国内源安装：pip install frida-tools -i https://pypi.tuna.tsinghua.edu.cn/simple/
    - 安装指定版本的frida-tools: pip install frida-tools==7.2.2
    - 具体的版本可以在github上查看: https://github.com/frida/frida-tools/releases
    - 如果之前装了frida-tools，先要卸载之前的版本:pip uninstall frida-tools
    - 查看frida版本：frida --version
    - 目前测试frida==12.9.4+frida-tools==7.2.2能够比较好的运行
- 安卓下安装frida:
    - 先查看手机处理器型号，以便下载对应的frida-server：adb shell getprop ro.product.cpu.abi
        - 一般模拟器是x86架构，真机是arm64架构
    - 下载对应的frida-server版本：
        - https://github.com/frida/frida/releases
        - 比如我使用模拟器，cpu是x86的，所以就下载frida-server-[version]-android-x86_64.xz文件，如果使用真机，cpu是arm64，就下载frida-server-[vserion]-android-arm64.xz，比如：frida-server-12.9.4-android-arm64.xz
        - frida-server的版本尽量和frida的版本一致：
            - 查看安装的frida版本：cmd中输入 frida --v 查看当前frida版本
            - 然后根据这个frida版本去github上下载对应版本的frida-server
    - 解压下载的*.xz文件，将其中的frida-server-[version]-android-cputype 文件放到android的目录下，比如放在/data/local/tmp下
    - 在adb shell命令中进入到/data/local/tmp中，给frida-server程序相应的启动权限:chmod 777 frida-server-12.9.4-android-arm64，再启动frida
    - 转发端口：adb forward tcp:27042 tcp:27042
- 在Pycharm中新建一个python工程，用来运行frida：
    - File->New Project->Pure Python
    - 选择工程存放路径，选择python版本，如果安装了多个python版本的话，可以在base interpreter中选择，这里选择python3.7版本的就行了
    - 一定要勾选Inherit global site-packages和Make available to all projects
    - 点击creak就创建成功
- 编写frida hook代码：
    - 在工程中新建一个.py文件


# 五、android Xposed和EdXposed介绍，以及常用模块
## 1. Xposed介绍

- 首先，Xposed是一个很牛逼的框架，可以在不修改APK的情况下影响程序的运行。

- 有一个叫做“Zygote”的过程。从它的名字（中文含义——受精卵）这是Android运行时的核心。每个应用程序都作为它的副本（“fork”）启动。/init.rc手机启动时，脚本会启动此过程。进程开始完成/system/bin/app_process，加载所需的类并调用初始化方法。

- 这就是Xposed发挥作用的地方。安装框架时，会将扩展的app_process可执行文件复制到/system/bin。这个扩展的app_process就会把XposedBridge.jar加载到运行时环境，这样我们就可以在虚拟机启动之前，甚至是在Zygote的main方法被执行之前做一些操作（其实就是加载插件）。此时我们的插件被执行，就是Zygote进程的一部分，所以可以直接获取到应用的上下文Context，然后做很多超出想象的事情——对于任何一个app ，我们都可以hook或者替换掉其中的类或方法或对象。再加上已经root。我们可以为所欲为了。

- jar位于，/data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar其源代码可在此处找到。查看类XposedBridge，您可以看到该main方法。这就是我上面写的内容，这个过程在一开始就被调用了。在那里进行了一些初始化，并且还加载了模块。

- 只需要在Android手机中安装Xposed框架后，再通过Xposed框架提供的API进行HOOK我们需要hook的方法等，就可以在程序正常运行过程中，做一些其他自己想要的操作。比如修改方法的参数、返回值，使得修改程序的运行逻辑。比如可以通过修改检查程序签名的验证方法的返回值来直接跳过签名验证等。

- Xposed不仅仅是一个插件加载功能，而是它从根上Hook了Android Java虚拟机，所以它需要root，所以每次为它启用新插件APP都需要重新启动。
## 2. Xposed安装

- Xposed包含如下几个工程：
    - XposedInstaller：这是Xposed的插件和模块管理（通过Xposed提供的接口，自己或他人开发出来进行HOOK特定功能的APK），也就是说Xposed整体管控功能就是由这个APP来完成的，它包括启用Xposed插件功能，下载和启用指定插件APP，还可以禁用Xposed插件功能等。注意，这个app要正常无误得运行必须能拿到root权限。
    - Xposed：这个项目属于Xposed框架，其实它就是单独搞了一套xposed版的zygote。这个zygote会替换系统原生的zygote。所以，它需要由XposedInstaller在root之后放到/system/bin下。
    - XposedBridge：这个项目也是Xposed框架，它属于Xposed框架的Java部分，编译出来是一个XposedBridge.jar包。
    - XposedTools：Xposed和XposedBridge编译依赖于Android源码，而且还有一些定制化的东西。所以XposedTools就是用来帮助我们编译Xposed和XposedBridge的。

- 首先手机需要先root。现在由于Android新版本的普及，现在root手机比较麻烦，可以淘宝咸鱼找个人帮忙root下
    - 是否容易root，取决于手机厂商希不希望用户可以root。绝大部分厂商是不希望的。
    - 之前的手机容易root是因为系统有漏洞。哪怕厂商不愿意，第三方root工具仍然可以利用漏洞来root。而在Android 6.0之后，由于大多数系统都开了selinux（linux下的一个安全策略，用于管理权限）和secureboot（系统启动时的一个安全策略），很难找到可以用来root的漏洞了。
    - 现在各大手机厂商对自己系统生态的限制，手机出厂都会添加一个BL锁（BootLoader的简称）。
        - Bootloader是在Android系统开始运行前的一个小程序，它是运行的第一个程序。Bootloader是OEM厂商（华为、小米和三星等）或者运营商加锁和限制的地方，一旦Bootloader锁上，你就无法在手机上安装其它操作系统，
        - 世上有两种类型的安卓手机：一些可以让你解锁Bootloader，另一些就不能。安卓手机能不能解锁Bootloader取决于手机制造商、手机型号和运营商。谷歌的Nexus手机全都是可以解锁的，一加、小米、三星、摩托罗拉和HTC的很多手机也能让你解锁Bootloader。
        - 上锁的Bootloader安卓手机就像iPhone，手机厂商完全控制了你的手机。只要它们给你推送一次更新（可以强制做到，除非你不联网），就能把你的手机变慢（请参考苹果降速门），甚至让手机不断重启和死机，这些在技术上都是可以做到的。
- 下载「Xposed Installer」软件并安装
    - Xposed 官方网址：
    ```
        https://repo.xposed.info/module/de.robv.android.xposed.installer
    ```
    - Android 5.0 及以上版本的下载地址：
    ```
        http://forum.xda-developers.com/showthread.php?t=3034811
    ```
    - 安装好xposed installer后，打开，然后在安装/更新处下载安装xposed就可以使用了
## 3. EdXposed介绍
-  有些系统和机型，可能是由于系统版本或其他原因，xposed installer安装不上，或者安装上了之后会闪退，或者不闪退，安装不上xposed框架，此时可能就需要安装EdXposed了。

- 由于现在Android新版本的普及，新上市的手机基本都是8.0以上了。所以以前的Xposed框架已经不适用。EdXposed团队成为了后续版本研发的继任者，从Xpsoed的wikipedia中可以查到交接历史：

    - 2017年7月，abforce在GitHub上发布了支持Nougat的xposed，不过此发布需在编译ROM前集成在源码中而不是以前直接卡刷的形式。
    - 2017年10月，Xposed框架开始支持。[3]
    - 2019年1月，ElderDrivers完成了EdXposed的开发。EdXp是一个Magisk模块，依赖于riru框架，成功将Xposed移植到了Android Pie上，成为最接近原版Xposed的框架。
    -2019年9月，EdXposed正式支持Android Q。
    - 2020年1月，EdXposed与Xposed原开发团队达成共识，成为Xposed停止更新后的官方接任者。

## 4. EdXposed安装
- TWRP Recovery、Magisk、EdXposed
    - 由于我的测试机是Android 8.1，所以要刷EdXposed。大致思路很简单，也是最容易成功的步骤：先刷三方Rec（这里是TWRP），然后装Magisk，最后通过Magisk装EdXposed。
    - 此处的Magisk，应该不是taichi.Magisk，而是台湾一个作者@topjohnwu开发的开源hook框架。Magisk的github地址：
    ```
        https://github.com/topjohnwu/Magisk
    ```

    - 安装的教程可以参考这个文章：
    ```
        https://blog.csdn.net/ysy950803/article/details/107118656/
    ```

    - 我安装的方式是通过在手机中安装Official TWRP APP（从Google Play中下载的），APK放在：
    ```
        链接：https://pan.baidu.com/s/1QcYLbBmQ7iVlDLiAFOLsHw  提取码：mpzc 
    ```
    - 通过Official TWRP APP中来实现下载和刷入TWRP。在刷入TWRP后，需要通过Official TWRP APP来重启到Recover模式中，并且这种方式刷TWRP是一次性的，下次需要进入TWRP的话，就需要再通过该APP来刷入TWRP。

    - 首次进入TWRP，可能会进入输入密码解锁Data分区的界面，此时直接点Cancel就可以回到主界面。关于Data分区，可以参考这个文章：
    ```
        http://www.manosp.com/archives/264
    ```

    - 刷入Magisk时，再TWRP中如果出现目录和文件名称全是十六进制乱码的情况，重新进入系统，把锁屏密码去掉就行了。
    

# 六.抓包工具的配置
## 1.Fiddler
- 基础配置
- 远程手机抓包配置
- 高版本Android手机抓HTTPS包配置
    - 先从fiddler中导出证书，Tools->HTTPS->Actions->Export Root Ceritficate to Desktop
    该操作会直接把证书导出到桌面。
    - 导出的证书格式是cer格式的，需要通过openssl转换成pem格式
        ```
        openssl x509 -in FiddlerRoot.cer -inform DER -out fiddler.pem -outform pem
        ```
        此时会在cer同目录下生成一个fiddler.pem文件
    - 提取该pem文件的hash值，用以给该文件命名：
        ```
        openssl x509 -inform PEM -subject_hash -in fiddler.pem
        035f9290
        -----BEGIN CERTIFICATE-----
        MIIDsjCCApqgAwIBAgIQQU7LwqQPS7NCt2luvNJx0DANBgkqhkiG9w0BAQsFADBn
        MSswKQYDVQQLDCJDcmVhdGVkIGJ5IGh0dHA6Ly93d3cuZmlkZGxlcjIuY29tMRUw
        ...
        ```
        035f9290就是该pem的hash值。
    - 修改pem文件的文件名为035f9290.0（文件名是Hash值加數字後綴。後綴名的數字是為了防止文件名衝突的，比如如果兩個證書算出的Hash值是一樣的話，那麼一個證書的後綴名數字可以設置成0，而另一個證書的後綴名數字可以設置成1）
    - 拷贝035f9290.0到Android证书目录下：
        ```
        adb push 035f9290.0 /system/etc/security/cacerts/
        ```
        如果出现：
        ```
        adb: error: failed to copy '035f9290.0' to '/system/etc/security/cacerts/035f9290.0': remote couldn't create file: Read-only file system

        执行：
        adb remount
        后再push一次
        ```
        - 并修改权限为664，重启手机
## 2.Charles
- 高版本Android抓HTTPS包配置
   ``` 
   charles-proxy-ssl-proxying-certificate.pem.crt

    安全證書在 /system/etc/security/cacerts/目錄下, 證書文件，文件名是Hash值加數字後綴。後綴名的數字是為了防止文件名衝突的，比如如果兩個證書算出的Hash值是一樣的話，那麼一個證書的後綴名數字可以設置成0，而另一個證書的後綴名數字可以設置成1

    用“openssl x509 -subject_hash_old -in 證書名加後綴”計算出hash值，並將後綴設為0

    openssl x509 -subject_hash_old -in charles-proxy-ssl-proxying-certificate.pem.crt

    修改後的文件為<hash>.0
    3cfb33fd.0

    複製的時候，要使用 adb root，以root權限進入到adb中，才能有權限進行拷貝到/system目錄下

    然後復製到/system/etc/security/cacerts/目錄下

    然後修改權限
    chmod 644 /system/etc/security/cacerts/3cfb33fd.0
    ```

# 五、 js调试环境
## 1. WebView调试环境
- 采用混合开发（hybird开发）的APP，可以在APP中嵌入网页，Android中，采用WebView控件来加载嵌入网页。高一点的Android版本，内置的WebView的内核是采用Chrome的内核，实际WebView就是一个内嵌浏览器，所以使用chrome能很好的调试。
- Android中发布出来的APP，一般都不会开启WebView的调试开关（要调试某个APP的WebView加载的网页，需要APP的代码中打开WebView的调试开关），所以这里需要采用一个Xposed模块来强制打开WebView的调试开关，该模块叫：WebViewDebugHook.apk。其GitHub地址是：https://github.com/feix760/WebViewDebugHook
- 连接手机，打开开发者调试
- 在谷歌浏览器的导航栏中输入 chrome://inspect/ ，等待连接设备
- 在手机中的APP中，打开要调试的网页，等待chrome://inspect/ 页面中出现要调试的页面
- 点击inspect，就可以打开devtools进行调试


## 2. 普通js文件调试
- 但是采用上述方式调试，在进入devtools开始调试时，整个页面是已经加载完成了，如果要在js刚开始加载的时候进行调试，可以采用node+devtools的方式
- 安装chrome扩展 NIM-Node.js 调试管理工具（https://chrome.google.com/webstore/detail/nodejs-v8-inspector-manag/gnhhdgbaldcilmgcpfddgdbkhjohddkj
）（需要翻墙，其实可以不用安装，看需求）
- 打开chrome://inspect
- 点击 Open dedicated DevTools for Node后，会弹出空白的devtools页面
- 执行你的js文件
    ```
    node --inspect=9222 file.js（如：node --inspect=9222 subscribeBuy.e22df20c.js）
    ```
- 执行上一步命令是直接开始运行整个js文件，如果要在刚刚加载js文件时就断下来，可以执行命令：
    ```
    node --inspect-brk=9222 file.js（如：node --inspect-brk=9222 subscribeBuy.e22df20c.js）
    ```
- 然后打断点，开始调试
- 参考：https://segmentfault.com/a/1190000020205396

## 3. 从服务器中下载下来的js
- 从服务器上下载下来的js，如果使用node运行的话，会有一些只有浏览器才有的对象在js中使用了，而node运行时是没有的。如window对象。
- 此时，可以考虑新建一个html文档，然后用 \<srcipt>标签来引用对应的js文件：
```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <script type="text/javascript"  src="subscribeBuy.e22df20c.js"></script>
        <script type="text/javascript"  src="chunk-vendors.153482d5.js"></script>
        <script type="text/javascript"  src="chunk-common.521dbf39.js"></script>
    </head>
    <body>
    <div>xiaomiyoupin</div>
    </body>
    </html>

```
- 如果可以，可以将调试获得的html页面整个拷贝到本地，或者通过postman等来请求html页面的http请求获得对应的html文件，然后修改html中对js文件的引用路径，来达到可以在本地从头加载js，然后从头调试的效果。但是这种并不是很好的调试方法，只能帮助分析逻辑
