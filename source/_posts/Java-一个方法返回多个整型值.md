---
title: Java 一个方法返回多个整型值
date: 2018-6-23 12:51:00
categories: "二进制"
tags:
     - Java
     - Android
     - 二进制
---



### 前言

在 Android 开发中会经常用到 RecyclerView , 当 RecyclerView 的 item 类型有多种时，我们需要重写 getItemViewType 方法。

```Java
    @Override
    public int getItemViewType(int i) {
        return 0;
    }
```

getItemViewType 方法指定返回一个 int 值，如果我们需要返回多个值怎么办？

### 场景

当我们做 IM 的聊天消息展示时，可能会有 state 代表消息状态（接收的已读消息、接收的未读消息、发送成功的消息、发送失败的消息），type 代表消息类型（文字、图片、视频、音频）。此时，我们可能需要 getItemViewType 返回 state 和 type 两个值。

```Java
    @Override
    public int getItemViewType(int i) {
        int state = mMessageList.get(i).state;
        int type = mMessageList.get(i).type;

        return 0;
    }
```

### 解决方法

一个 int 类型有 32 位，我们用左边的高 16 位存储 state 的值，用右边的低 16 位存储 type 的值

```Java
    @Override
    public int getItemViewType(int i) {
        int state = mMessageList.get(i).state;
        int type = mMessageList.get(i).type;

        int result = (state & 0x7fff) << 16;
        result |= (type & 0x7fff);

        return result;
    }
```

在 onCreateViewHolder 中通过 viewType 解析出 state 和 type .

```Java
   @Override
    public AbstractChatHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        ViewHolder viewHolder;

        int type = viewType & 0x7fff;
        int state = (viewType >> 16) & 0x7fff;

        // 根据 state 和 type 创建各种 ViewHolder ......

        // 省略其它代码 ......

        return viewHolder;
    }
```

### 注意点

- state 和 type 的取值范围都是 [0,32767]，即最小值为 0，最大值为 32767（十六进制的 0x7fff），因为它们实际各占 15 位

