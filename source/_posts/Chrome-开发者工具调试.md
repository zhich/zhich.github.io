---
title: Chrome 开发者工具调试
date: 2018-3-4
categories: "工具"
tags:
     - Chrome
     - 工具
     - 调试
---






### Chrome 开发者工具面板简介

![](/images/3.png)

- 箭头图标

  用于在页面选择一个元素来审查和查看它的相关信息，当我们在 Elements 标签页页面下点击某个 DOM 元素时，箭头图标会变成选择状态。

- 设备图标

  可以选择不同的终端设备、不同的尺寸比例进行模拟开发。

- Elements

  查找网页源代码 HTML 中的任一元素，手动修改任一元素的**属性**和**样式**且能实时在浏览器里面得到反馈。

- Console

  记录开发者开发过程中的日志信息，且可以作为与 JS 进行交互的命令行 Shell .

- Sources

  用于**查看**和**调试**当前页面所加载的脚本的源文件。

- Network

  用于查看 HTTP 请求的详细信息，如请求头、响应头及返回内容等。

- Performance

- Memory

- Application

  记录网站加载的所有资源信息，包括存储数据（Local Storage、Session Storage、IndexedDB、Web SQL、Cookies）、缓存数据、字体、图片、脚本、样式表等。

- Security

  判断当前网页是否安全。

- Audits

  对当前网页进行网络利用情况、网页性能方面的诊断，并给出一些优化建议。比如列出所有没有用到的 CSS 文件等。

### Elements

可以查看、修改页面上的元素，包括 DOM 标签、CSS 样式，还有相关盒模型的图形信息。

- 双击 DOM 树视图里面的节点，可以实时编辑标签属性，修改的效果会立刻反应在浏览器里面。

- 点击右侧的 Styles 标签页，可以实时修改 CSS 的属性值，所有的 Name 和 Value 值都是可以编辑的。在每个属性后面单击可以添加新的样式。

![](/images/3.png)

- 点击 Computed 标签页，可以编辑左侧选中的盒子模型参数，所有值都是可以修改的。点击不同的位置（top、bottom、left、right）就可以修改元素的 padding、border、margin 属性值。

> 注意：**以上对页面上的修改并不会作用到源码上**，它仅用于调试，一刷新这些修改就会复原。我们可以一次性在浏览器中做了修改后，再把改动复制到源码中。

### Console







