# Android Debugging Guideline

#### 非官方出品，无责任保证 ####
#### Unofficial, no guarantee ####
#### 欢迎贡献 ####


不管你是超级小白兔，还是过弯不减速的资深老司机，开发过程中可能会遇到各种各样的问题。这里主要聚集的是实时通讯或者多媒体相关的问题。

需要注意的是，如果你不能借助外力看到一些必须的信息的话，一定要学会，否则成不了 Senior/Staff/Principal Android Developer。



##### 0. 万一有什么重要的就放在这里

##### 1. 程序崩溃/闪退怎么办？堆栈如何看？
崩溃了想办法弄到调用栈，当然去找 logcat 或者 bugreport 或者 tombstone 或者 Bugly 或者 Crashlytics

* adb logcat > logcat.txt

这是一个文本文件，直接拿出来就看

* adb bugreport > bugreport.txt

这也是一个文本文件，可以拿出来直接看或者通过 chkbugreport 工具来分析 http://developer.sonymobile.com/knowledge-base/tools/analyse-your-bugreports-with-our-open-source-tool/

* /data/tombstones/ 对于一些 root 权限的的机器，我们也可以把它的墓志铭弄出来分析

* 其他 Bugly 或者  crashlytics  如果不是太懂可以去互联网搜寻

* 堆栈当中会有一些 **关键字**，比如 AndroidRuntime 或者 Fatal/fatal 或者 signal/SIG

##### 2. Android 当中崩溃一般分哪些？
Android 应用程序运行的时候一般包括位于虚拟机代码部分和本地代码部分，两者崩溃是不一样的。

虚拟机上执行的代码崩溃比较好查，通常我们能够看到行数，直接对着源码去看就好了。

本地代码稍微难看点，但是也可以借助 addr2line 工具

`$PATH_TO_NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-addr2line -f -e armeabi-v7a/libagora-rtc-sdk.so 0x0029b6f4`

so 文件是带符号的，后面一串 16 进制的就是 PC 地址，这样也可以反解出代码行数

##### 3. 调试的时候看本地代码崩溃行数

`adb logcat -v threadtime | ndk-stack -sym PATH_TO_YOUR_NATIVE_PROJECT/obj/local/armeabi-v7a`

##### 4. SurfaceView/GLSurfaceView 为什么设置 setVisibility 为 GONE/INVISIBLE 对它不起作用？
通常来说是因为 SurfaceView 的原理和其他不一样，具体的参见 https://developer.android.com/reference/android/view/SurfaceView 或者各路网络上搜索。

跟这个症状还有关的是它的层级。通常来说可以通过 setZOrderOnTop/setZOrderMediaOverlay 来去解决，这两个是不一样的。

##### 5. SurfaceView 为什么会出现遮挡/看不见？
跟 4.  差不多 

##### 6. 如何看某些视图被遮挡，布局是否有写对？
早期有工具叫做 Hierarchy Viewer 或者 uiautomatorviewer，如今 Android Studio 提供更方便的工具 Tools -> Layout Inspector，打开看看就知道了。

##### 7. 如何反编译 App？
很多工具，但是我一般用 [http://ibotpeaches.github.io/Apktool/](http://ibotpeaches.github.io/Apktool/)

反编译 apk

`apktool d your_apk_name.apk`

重新打包 apk

`apktool b your_apk_name`

在 your_apk_name/dist 下面即可找到打包好的 apk

⚠️ 重新打包好之后需要签名才能安装到手机当中运行

可以把要签名的 apk 拷贝到 `~/.android` 当中，然后在该目录下执行

`jarsigner -verbose -keystore resigned.keystore -signedjar baba_signed.apk baba.apk resigned.keystore`

根据提示输入 resigned.keystore 的密码即可开始签名

直到看到类似于

jar signed.

这类的输出，就表示签名成功

##### 8. 如何替换市场 App 当中的 so 文件？
A) 通过上面的步骤可以实施，只需要找到位于 lib 文件夹下的 so 文件，替换之后再打包即可

B) 通过 [https://github.com/asLody/VirtualApp](https://github.com/asLody/VirtualApp) 来完成
过程很简单，先设备上安装 VirtualApp，再 VirtualApp 里面安装需要替换的应用(不管是克隆还是全新安装都可以)

`adb shell`
`run-as io.virtualapp`
`cd /data/data/io.virtualapp/virtual/data/app/`

想法把 so 拷贝到对应的地方，比如 `/data/data/io.virtualapp/virtual/data/app/xyz.xxx/lib`
重启 App 运行，不行就重启 VirtualApp 和你想换的 app

##### 9. 自己采用 OpenGL ES 渲染的视频是黑的
看看代码没有逻辑错误的话，看看 Shader 是否正确，把所有 OpenGL 的 error 都弄出来看下 glError()

##### 10. 使用 Agora 设置的 setLocalVideo/setupRemotVideo 视频是黑的
目前这两个 API 不支持自己 new SurfaceView 或者  xml 配置的 SurfaceView，需要找到 SDK 对应的创建渲染视图的方法。






#### 贡献者名单




##### 看完了整个页面，在网络上搜寻了也还是搞不定的话，找到写这个的人，超贵计时服务，300 RMB/小时，不足一小时按一小时算。咨询聊天服务也如此 ^_^
