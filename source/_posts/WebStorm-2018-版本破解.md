---
title: WebStorm 2018 版本破解
date: 2018-08-15
categories: "工具"
tags:
     - WebStorm
     - 工具
---

<meta name="referrer" content="no-referrer">




本文讲述的是 Windows 的 WebStorm 2018 版本的破解。步骤如下：

1. 下载 WebStorm . 笔者下载的版本为 WebStorm 2018.1.2

2. 下载破解补丁。[下载链接](https://pan.baidu.com/s/1ExVU0878pvHTuMQ8myKdCg)，密码：yxb5

3. 拷贝补丁到 WebStorm 安装目录的 bin 目录下

4. 同时修改 bin 目录下的 WebStorm.exe.vmoptions 和 WebStorm64.exe.vmoptions 文件，在它们的最上面添加以下格式的代码：

   **-javaagent:webstorm安装路径/bin/破解补丁名字.jar**

   如笔者要添加的一行代码为：

   ```Java
   -javaagent:C:/Program Files/JetBrains/WebStorm 2018.1.2/bin/JetbrainsCrack-2.8-release-enc.jar
   ```

   > 注意斜杠的方向

5. 保存文件。启动 WebStorm , 选择 activation code , 并将上面的那一行代码作为激活码拷贝进入即可破解成功。