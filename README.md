# App分析的工具
## Android App

#### 静态：
1. flowdroid

    https://github.com/secure-software-engineering/FlowDroid

   使用方法：
```
java -jar soot-infoflow-cmd/target/soot-infoflow-cmd-jar-with-dependencies.jar \
    -a <APK File> \
    -p <Android JAR folder> \
    -s <SourcesSinks file>
```
   在SourcesSinks file中声明source和sink即可，具体写法可以参考： https://github.com/secure-software-engineering/FlowDroid/blob/develop/soot-infoflow-android/SourcesAndSinks.txt
  

2. appshark： 字节跳动的工具，比flowDroid看起来更好用一些

   和FlowDroid用起来差不多

   使用指南：https://github.com/bytedance/appshark/blob/main/doc/zh/how_to_find_compliance_problem_use_appshark.md


4. Jeb, JD-GUI：JAVA/Android的反编译查看源码的工具

   Jeb: https://www.pnfsoftware.com/
   JD-GUI: https://java-decompiler.github.io/
   使用方法：把APK拖入这两个工具即可

6. androguard: https://github.com/WuFengXue/android-reverse

   像是一个Android各种逆向工具的collection，提供了Python API用于查看apk的各种信息
   例如
   使用``a.get_permissions()``查看APP所用的全部权限


5. 其它常用的Android分析工具
   
   ADB
```
adb shell: ssh连上手机、模拟器
adb install: 安装app
adb logcat: 查看android系统和应用的log
adb push: 向手机、模拟器 的文件系统中上传一个文件 类似于scp 
adb pull: 从手机、模拟器 的文件系统中下载一个文件
```

#### 动态测试：
1. Appium: 一款UI测试工具

   https://appium.io/docs/en/2.1/

   可以参考这个教程安装和测试：https://zhuanlan.zhihu.com/p/144737398

3. uiautomator2:  一款UI测试工具

   https://github.com/openatx/uiautomator2

   同Appium, 但比Appium更简单易用，官方（Git仓库）中的教程简单易读
   可以参考这个教程使用：https://blog.csdn.net/d1240673769/article/details/113809889


4. Frida： 好用的插桩软件
   
   Frida的安装比较复杂，总的来说，电脑上需要先安装一个frida-tools， 然后再通过adb push传一个frida server到手机上，然后通过adb shell在手机中运行这个server。 server可以接收来自电脑上frida-tools的命令，做后续的操作

   官网的教程省略了很多重要信息
   可以看： https://medium.com/@briskinfosec/getting-started-with-frida-de44d932ae7
   或者这个视频：https://www.youtube.com/watch?v=Ew3VfY07Pxk

   具体的用法：写一个Python脚本
```
console.log("Script loaded successfully ");

setTimeout(function(){
    //判断是否加载了Java VM，如果没有加载则不运行下面的代码
    if(Java.available) {
      
        Java.perform(function(){
        
        // 先通过Java.use函数得到Ordinary_Class类
        // 即这个例子中我们要插桩 android.bluetooth.BluetoothGattCharacteristic这个类中的方法
        var Ordinary_Class = Java.use("android.bluetooth.BluetoothGattCharacteristic");
        if(Ordinary_Class != undefined) {

            /** 
            这里 setValue表示，android.bluetooth.BluetoothGattCharacteristic这个类中有一个方法叫setValue
            该函数的原型如下:
            public boolean setValue(byte[] value) {
                  mValue = value;
                  return true;
            }

            Ordinary_Class.setValue.overload('[B')
            这里的'[B'中  [ 表示数组，B 表示字节（byte）。所以，'[B' 就是表示字节数组。
            
            因为android.bluetooth.BluetoothGattCharacteristic这个类中setValue有多种参数形式，例如下面这个setValue也是该类中的一个同名函数
            public boolean setValue(int value, int formatType, int offset)

            这种情况下，只写setValue的话 Frida不知道我们要hook哪个函数，因此还需要用overload指定参数类型。 其它参数类型怎么指定可以询问chatGPT或者搜索一些教程

            function (a){} 这里表示当setValue这个函数被触发时我们要做的事情， a是该函数的入参，即为byte[] value
            我们可以在function (a){}的函数体重打印，修改它

            **/
            Ordinary_Class.setValue.overload('[B').implementation = function (a){

               var ByteString = Java.use("com.android.okhttp.okio.ByteString");
                var s = ByteString.of(a).hex();

                var res =this.setValue(a);
                console.log("setValue获得值:"+ByteString.of(a).hex());
                console.log("\n");
                return res;
            }

    }

    console.log("hook end");
  });    
 }
});
```
   Frida还可以修改被插桩函数的返回值等。


4. Xpose：强大的插桩软件

   在真机上安装比较复杂，建议在模拟器（夜神，雷电模拟器等）上安装使用
   https://support.yeshen.com/zh-CN/qt/xp

   具体的用法和Frida差不多


5. tcpdump: 流量抓包

   用adb push向手机上上传一个tcpdump的binary，然后用adb shell 在手机上运行
   教程：https://blog.csdn.net/iamcxl369/article/details/77720857

7. Charles，mitmproxy：中间人抓包

   使用Charles，mitmproxy这一类的工具作为Android手机的proxy，抓包

   对于HTTPs, 在收集和电脑上同时下载证书：
   https://juejin.cn/post/6874903020677791758

   不过中间人抓包目前对于大部分的商业APP都不可行，原因是大部分APP不会trust用户自己安装的证书


7. justTrustMe忽略证书效验

   该工具似乎是通过hook一些通用的证书效验的API来绕过6中所提到的“大部分APP不会trust用户自己安装的证书”

   这套抓包的使用方法较为复杂，可以参考下面这个教程
   https://crifan.github.io/app_capture_package_tool_charles/website/how_capture_app/complex_https/https_ssl_pinning/android/xposed_justtrustme.html



## iOS App

#### 静态：
1. iblessing: 较好用的静态分析工具

   https://github.com/Soulghost/iblessing
   可以看到iOS binary中所有的class以及call-graph

3. IDA-pro

   iOS app binary主要是汇编语言，通过IDA-pro的反编译（F5 汇编--》源码） 可以大致了解源码的逻辑

#### 动态：
1. Appium: UI测试工具

   用法同Android Appium，但是安装比较复杂，需要先在手机上装一个WebDriverAgent, 然后在电脑上使用Appium和手机上的WebDriverAgent通讯
   安装可以参考：
   https://blog.csdn.net/liuage_/article/details/124508920
   https://juejin.cn/post/7011778694511329310

***************

# App论文相关

## Android App
#### UI Testing

1. Guided, stochastic model-based GUI testing of Android apps
经典的UI测试论文，基于APP页面跳转图做测试


2. Practical GUI testing of Android applications via model abstraction and refinement
对于《Guided, stochastic model-based GUI testing of Android apps》 中APP页面跳转图的进一步抽象


3. Reinforcement learning based curiosity-driven testing of Android applications
经典的基于强化学习的UI测试



#### 隐私相关
1. Ppchecker: Towards accessing the trustworthiness of android apps' privacy policies
经典的APP隐私政策论文


#### 第三方库相关
1.  Research on third-party libraries in android apps: A taxonomy and systematic literature review
值得一看的综述，涵盖第三方库有关的所有研究

2. Automated third-party library detection for android applications: Are we there yet?
值得一看的第三方库检测的综述


#### 广告相关
1. MadDroid: Characterising and Detecting Devious Ad Content for Android Apps

2. Frauddroid: Automated ad fraud detection for android apps



#### fingerprinting

1. FLOWPRINT: Semi-Supervised Mobile-App Fingerprinting on Encrypted Network Traffic

2. Appscanner: Automatic fingerprinting of smartphone apps from encrypted network traffic

3. FOAP: Fine-Grained Open-World Android App Fingerprinting


#### 其它经典的论文
1. Reflection-Aware Static Analysis of Android Apps
讨论了Android APP静态分析中反射的问题

2. Iccta: Detecting inter-component privacy leaks in android apps
讨论了Android中通过Intent泄露的隐私

3. 




*******
## 小程序
WeMinT: Tainting Sensitive Data Leaks in WeChat Mini-Programs

Don't Leak Your Keys: Understanding, Measuring, and Exploiting the AppSecret Leaks in Mini-Programs
小程序的流量解密+隐私分析







## iOS App

#### 隐私相关

1. Pios: Detecting privacy leaks in ios applications.
静态分析工具，taint analysis找隐私泄露

2.xxxxx

3. Lalaine: Measuring and Characterizing Non-Compliance of Apple Privacy Labels
检查app store 隐私标签

4. 

Understanding {iOS-based} Crowdturfing Through Hidden {UI} Analysis
基于app 页面跳转图 的 众包软件检测



iOS APP分析好用的工具

动态：
Appium: https://appium.io/docs/en/2.1/


越狱：爱思助手



## 其它和程序分析/测试相关有趣的paper

1. Wuji: Automatic online combat game testing using evolutionary deep reinforcement learning
大型游戏的测试，基于演化算法
2. 



TODO:
https://github.com/JonathanSalwan/Triton
https://github.com/trailofbits/polytracker


