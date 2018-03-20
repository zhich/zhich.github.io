---
title: Mock 测试
date: 2018-3-20
categories: "测试"
tags:
     - 测试
     - mock
---

https://github.com/dreamhead/moco/


![](/images/14.png)

**config.json**

```json
[{
  "request": {
    "uri": "/hello",
    "method": "get"
  },

  "response": {
    "file": "hello.json"
  }
}, {
  "request": {
    "uri": "/checkAppUpgrade",
    "method": "get"
  },

  "response": {
    "file": "checkAppUpgrade.json"
  }
}]
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

**命令**

```bash
java -jar moco.jar http -p 8089 -c config.json
```