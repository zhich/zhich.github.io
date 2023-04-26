---
title: localStorage 实现历史记录功能
date: 2018-09-29
categories: "前端"
tags:
     - localStorage
---

<meta name="referrer" content="no-referrer">




我们知道 **localStorage 的生命周期是永久的**。除非用户在浏览器上手动删除 localStorage 信息，否则这些信息将永久存在。

**sessionStorage 的生命周期为当前窗口或者标签页**。用户一旦关闭了窗口或者标签页，那么通过 sessionStorage 存储的数据也将被清空。

> 不同浏览器间无法共享 localStorage 或 sessionStorage 中的数据，相同浏览器的不同页面间可共享相同的 localStorage（页面属于相同域名和端口），但不同页面或标签间无法共享 sessionStorage 的数据。


如此看来，localStorage 更适合用来做历史输入记录的存储。

> 思路：存储的历史记录用 historyItems 表示，historyItems 以 '|' 分隔符存储各项记录，当某项记录 a 在 historyItems 中存在，那么把原来的 a 去掉，把新的记录 a 放到最前面。

**存储数据**

```
setHistoryItem(keyword) {
    keyword = keyword.trim();
    let { historyItems } = localStorage;
    if (historyItems === undefined) {
        localStorage.historyItems = keyword;
    } else {
        const onlyItem = historyItems.split('|').filter(e => e != keyword);
        if (onlyItem.length > 0) {
            historyItems = keyword + '|' + onlyItem.join('|');
        }
        localStorage.historyItems = historyItems;
    }
}
```

**获取所有数据**

```
getHistoryItems() {
    if (localStorage.historyItems === undefined) {
        return [];
    }
    return localStorage.historyItems.split('|');
}
```

**根据关键字获取数据**

```
getHistoryItemsByKeyword(keyword) {
    if (localStorage.historyItems === undefined) {
        return [];
    }
    keyword = keyword.trim();
    let seletedHistoryItems = localStorage.historyItems.split('|').filter(e => e.indexOf(keyword) != -1);
    return seletedHistoryItems;
}
```

**根据关键字删除数据**

```
deleteHistoryItemByKeyword(keyword) {
    if (localStorage.historyItems === undefined) {
        return;
    }
    let historyItems = localStorage.historyItems.split('|');
    let index = historyItems.indexOf(keyword);
    if (index < 0) {
        return;
    }
    historyItems.splice(index, 1);
    localStorage.historyItems = historyItems.join('|');
}
```

**清空数据**

```
clearHistory() {
    localStorage.removeItem('historyItems');
}
```