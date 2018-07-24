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

![Mou icon](http://pcckwdbix.bkt.clouddn.com/3.png)

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

  判断当前网站的安全性，查看有效证书等。

- Audits

  对当前网页进行网络利用情况、网页性能方面的诊断，并给出一些优化建议。比如列出所有没有用到的 CSS 文件等。

### Elements

可以查看、修改页面上的元素，包括 DOM 标签、CSS 样式，还有相关盒模型的图形信息。

- 双击 DOM 树视图里面的节点，可以实时编辑标签属性，修改的效果会立刻反应在浏览器里面。

- 点击右侧的 Styles 标签页，可以实时修改 CSS 的属性值，所有的 Name 和 Value 值都是可以编辑的。在每个属性后面单击可以添加新的样式。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/4.png)

- 点击 Computed 标签页，可以编辑左侧选中的盒子模型参数，所有值都是可以修改的。点击不同的位置（top、bottom、left、right）就可以修改元素的 padding、border、margin 属性值。

> 注意：**以上对页面上的修改并不会作用到源码上**，它仅用于调试，一刷新这些修改就会复原。我们可以一次性在浏览器中做了修改后，再把改动复制到源码中。

### Console

#### 在控制台输出日志
**在 JS 代码或者命令行中，可通过以下 API 将日志信息打印到控制台中**

- console.log

  打印一般的基础日志信息，当要打印的基础日志太多时可使用 `console.group` 将相关的日志进行分组。

- console.warn

  打印带有黄色小图标的警告信息。

- console.error

  打印带有红色小图标的错误信息。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/5.png)

- console.assert

  当第一个参数为 false 时，才会打印第二个参数的值。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/6.png)

**可以根据 JS 条件判断输出不同的日志信息**

> 当需要换到下一行而不是回车的时候，请按 `Shift + Enter`

![Mou icon](http://pcckwdbix.bkt.clouddn.com/7.png)

#### 与控制台交互

- JS 表达式计算

  可以在控制台中输入 JS 表达式，然后点击 `Enter` 键得出结果。当在控制台输入命令时，会弹出相应的智能提示框，可按 `Tab` 键补全命令。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/8.png)

- 选择元素

  **$(selector)**: 列出与 selector 匹配的所有元素。

  **$$(selector)**: 列出与 selector 匹配的所有元素组成了数组。

  **$x(path)**：返回的是一个数组，数组中即为与 xpath 匹配的所有元素。 

另外还有两种方法与上面类似：

**document.querySelector("img")**：会返回 DOM 中匹配的第一个元素（只返回一个元素）。

**document.querySelectorAll("img")**：等同于 $$(selector) 。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/9.png)

> 点击返回的每个元素，则会定位到页面中的 img 元素及 html 中的具体位置。

### Sources

#### 格式化代码

![Mou icon](http://pcckwdbix.bkt.clouddn.com/10.png)

![Mou icon](http://pcckwdbix.bkt.clouddn.com/11.png)

以上图片分别为源代码格式化前和格式化后的状态。通过点击源代码底部的 `{}` 即可将源代码格式化。

#### Sinppets 代码片段

我们可以在 Sources 标签页左侧的 Sinppets 按钮中创建代码片段用于测试。

以下创建一个 sum.js 的代码片段。在代码区内写完代码后，可以格式化、打断点，右键选择 `Save` 保存代码。在 sum.js 文件上右键选择 `Run` 即可运行代码，代码的运行结果会在底部的 Console 输出。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/12.png)

此外，**如果是在你自己的项目环境内，Sinppets 代码片段可执行项目内的方法。**

比如，可以新建一个 app.js 代码片段，用于调用我们项目环境中 common.js 的某些方法或字段。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/13.png)

#### 断点调试

#### 在代码中打断点

![Mou icon](http://pcckwdbix.bkt.clouddn.com/18.png)

在源代码左侧处点击即可打上一个断点。 当代码运行到断点处，右侧功能区域会展示出调试的相关信息。右侧最上面一排分别是：暂停/继续、单步执行、单步跳入此执行块、单步跳出此执行块、禁用/启用所有断点。

> 在调试过程中，把鼠标停留在字段上即可实时看到该字段的值。

#### 快速进入调试的方法

当我们的代码执行到某个程序块方法处，可能你并没有在该方法上设置相关的断点，此时你可以 `F11` 进入该方法中，但是我们的项目往往都是经过很多源代码封装好的方法，有时候进入后，会走很多底层的封装方法，需要很多步骤才能真正进入这个方法内，此时将鼠标放在此方法上，会出现相关提示，会告诉你在该文件的哪一行代码处，点击即可直接看到这个方法，然后临时打上断点，按 `F10` 或者点击右上角的第二个按钮即可直接进入此方法的断点处。

##### 调试的功能区域介绍

调试的功能区域在调试页面的**右侧**。下图为在某方法处打了一个断点。

![Mou icon](http://pcckwdbix.bkt.clouddn.com/19.png)

- watch

  可以用来实时监视变量的值。

- Call Stack

  断点执行到程序块停下来后，Call Stack 会显示断点所在处的方法调用栈。

  如果想重新从 Call Stack 调用栈中调用某个方法，可以在 Call Stack 调用栈中的该方法处右键点击 **Restart Frame** ，那么断点就会跳转到该方法的开头处重新执行。 

- Scope

  可以查看此时局部变量和全局变量的值。
 
- Breakpoints

  展示当前所有 js 断点，通过点击按钮可以去掉或加上断点，点击下方的代码表达式可跳转到该断点的程序代码处。

- XHR Breakpoints

  点击右侧 + 号，可以添加请求的 URL ，当 XHR 调用触发时就会在 request.send() 处中断。

- DOM Breakpoints

  当给 DOM 元素设置断点（来查看元素的变化情况）时，该 DOM 断点就会出现在 DOM Breakpoints 中。

- Event listener Breakpoints 

  此处列出了各种可能的事件类型。勾选对应的事件类型，当触发该事件类型的 js 代码时就会自动中断。








参考资料：

[Chrome开发者工具详解(1)-Elements、Console、Sources面板](http://www.cnblogs.com/charliechu/p/5948448.html)

[超完整的 Chrome 浏览器客户端调试大全](http://web.jobbole.com/89344/)