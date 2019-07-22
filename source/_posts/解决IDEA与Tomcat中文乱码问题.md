---
title: 解决IDEA与Tomcat中文乱码问题
date: 2019-07-22 17:52:54
tags: java
---

#### Tomcat控制台乱码原因：

​		Windows 的控制台默认是GBK编码的，而Toncat安装目录下的conf/logging.properties文件中的输出日志编码是UTF-8；

<img src="解决IDEA与Tomcat中文乱码问题/1563789900223.png" width="500">

![1563789714940](解决IDEA与Tomcat中文乱码问题/1563789714940.png)

**解决方法：**

1. 将Tom安装目录下的conf/logging.properties文件中的输出日志编码改为**GBK**;

   ![1563809191066](解决IDEA与Tomcat中文乱码问题/1563809191066.png)

2. 只将Tomcat控制台的编码改为UTF-8

   第一步：Windows+R打开运行，输入regedit进入注册表编辑器

   第二步：在HKEY_CURRENT_USER→Console→Tomcat中修改CodePage为十进制的65001

   注意：如果没有Tomcat或者CodePage，直接新建一个，如下图所示![1563790913473](解决IDEA与Tomcat中文乱码问题/1563790913473.png)

   也可以直接复制下面的代码，保存为.bat文件后，直接运行，即可修改成UTF-8。

   ```shell
   set rr="HKCU\Console\Tomcat"
   reg add %rr% /v "CodePage" /t REG_DWORD /d 0x0000fde9 /f>nul
   ```

#### IDEA控制台乱码解决

乱码情况：

![1563791599285](解决IDEA与Tomcat中文乱码问题/1563791599285.png)

**解决方法：**

​		进入Help→Edit custom VM options 打开idea64.exe.vmoptions文件追加

```
-Dfile.encoding=UTF-8
```

重启IDEA

![1563792253002](解决IDEA与Tomcat中文乱码问题/1563792253002.png)