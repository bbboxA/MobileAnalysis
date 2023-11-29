# LIBSPECTOR: Context-Aware Large-Scale Network Trafﬁc Analysis of Android Applications

1. 不要去分析复杂的商业软件， Frida，xposed这些插桩工具无法搞定
2. 不需要修改Android的内核

#### Workflow
1. 插桩论文中提到的```socket()和```connect()```函数，
   两个函数都定义在在AOSP的```bionic/libc/include/sys/socket.h```中
   
   可见https://cs.android.com/android/platform/superproject/main/+/main:bionic/libc/include/sys/socket.h

   **看起来这两个函数像在libc中**

   
2. 插桩这两个函数，可用Frida，xposed
   可以参考教程https://kevinspider.github.io/frida/frida-hook-so/
   以及https://kuizuo.cn/docs/frida-so-hook
   中hook **libc**的部分

   ![image](https://github.com/bbboxA/MobileAnalysis/assets/55539482/f332b0f2-7033-4803-99e0-cfb607a4c790)


3. 在hook point中打印```socket()和```connect()```函数的参数中的 源 和目标 IP地址
4.  在hook point通过JAVA API getStackTrace打印调用栈
