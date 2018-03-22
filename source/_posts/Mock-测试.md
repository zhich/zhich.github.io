---
title: Mock 测试
date: 2018-3-20
categories: "测试"
tags:
     - 测试
     - mock
---

在开发过程中，前端和后端的工作通常都是并行的，要想有效地提高工作效率，后端的接口文档就显得特别重要。

接口文档代表着一份请求/响应的契约书，简单地讲就是前端需要带什么样的数据过去？后端返回什么样的数据？

有了接口文档，我们就清楚了与后端交互的**数据结构**，然后可以通过 Mock 模拟请求/响应的数据。这样可以在前后端互不干扰的情况下完成各自的工作，大大地提高了开发效率。

以下简单介绍 Github 上的一个 Mock 开源库的使用 - [moco](https://github.com/dreamhead/moco/)。

**step 1:**

下载如图所示的 jar 包，并简单命名为 moco .

![](/images/15.png)

**step 2:**

把上面的 moco.jar 放到一个文件夹里面，并在该文件夹中创建配置文件 config.json 。config.json 中配置了两个请求，一个输出 Hello World 的请求，一个检查 App 升级的请求，他们分别输出 hello.json 文件和 checkAppUpgrade.json 文件中的内容。

![](/images/14.png)

**config.json**

```json
[
  {
    "request": {
      "uri": "/hello",
      "method": "get"
    },
    "response": {
      "file": "hello.json"
    }
  },
  {
    "request": {
      "uri": "/checkAppUpgrade",
      "method": "get"
    },
    "response": {
      "file": "checkAppUpgrade.json"
    }
  }
]
```

**hello.json**

```json
{
  "code": 0,
  "msg": "请求成功",
  "data": {
    "desc": "Hello World !"
  }
}
```

**checkAppUpgrade.json**

```json
{
  "code": 0,
  "msg": "请求成功",
  "data": {
    "versonName": "1.2",
    "versonCode": 3,
    "downloadUrl": "http://www.baidu.com/v1.2.apk",
    "desc": "v1.2 版本修复了重大 bug .",
    "isForceUpdate": true
  }
}
```

**step 3:**

在当前文件夹 cmd 运行以下命令即可启动 moco 服务器。

```bash
java -jar moco.jar http -p 8089 -c config.json
```

**step 4:**

在浏览器输入 config.json 配置文件中配置的 uri 即可输出对应的 file 文件中指定的 json 数据。

![](/images/16.png)

![](/images/17.png)

> 把 localhost 换成自己电脑的 ip 就可以在手机上访问了。
