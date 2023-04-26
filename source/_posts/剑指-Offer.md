---
title: 剑指 Offer
date: 2017-12-9
categories: "算法"
tags:
     - 算法
---

<meta name="referrer" content="no-referrer">






## [二维数组中的查找](https://www.nowcoder.com/practice/abc3fe2ce8e146608e868a70efebf62e?tpId=13&tqId=11154&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

**题目描述**

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

**思路**

从二维数组右上角（或左下角）开始遍历查找。下面两种解法都是从右上角开始查找的，左下角同理。

Solution 1 : (168 ms , 16792 K)

```Java
    public boolean Find(int target, int[][] array) {
        int rowCount = array.length;
        int columnCount = array[0].length;
        int tempJ = columnCount - 1;
        for (int i = 0; i < rowCount; i++) {
            for (int j = tempJ; j >= 0; j--) {
                if (target == array[i][j]) {
//                    System.out.println(i + " " + j);
                    return true;
                }
                if (target > array[i][j]) {
                    break;
                }
                tempJ = j - 1;
            }
        }
//        System.out.println("not exist .");
        return false;
    }
```

Solution 2 : (184 ms , 16708 K)

```Java
    public boolean Find2(int target, int[][] array) {
        int rowCount = array.length;
        int columnCount = array[0].length;
        int i = 0;
        int j = columnCount - 1;
        while (i < rowCount && i >= 0 && j < columnCount && j >= 0) {
            if (target == array[i][j]) {
//                System.out.println(i + " " + j);
                return true;
            }
            if (target > array[i][j]) {
                i++;
            } else {
                j--;
            }
        }
//        System.out.println("not exist .");
        return false;
    }
```

## [变态跳台阶](https://www.nowcoder.com/practice/22243d016f6b47f2a6928b4313c85387?tpId=13&tqId=11162&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

**题目描述**

一只青蛙一次可以跳上 1 级台阶，也可以跳上 2 级 …… 它也可以跳上 n 级。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

**思路 1：**

在 n 级台阶中，第一步可能会跳 1 级、2 级 …… 、n 级。

假设 n 级台阶有 f(n) 种跳法。

若第一步跳了 1 级，那么剩下的 n - 1 级会有 f(n - 1) 种跳法； 

若第一步跳了 2 级，那么剩下的 n - 2 级会有 f(n - 2) 种跳法；

……

若第一步跳了 n - 1 级，那么剩下的 1 级会有 f(1) 种跳法。

总结起来就是：

**f(n) = f(n - 1) + f(n - 2) + …… + f(1)**  &nbsp; &nbsp; &nbsp; ①

根据递推知识，不难想到

f(n - 1) = f(n - 2) + f(n - 3) + …… + f(1)  &nbsp; &nbsp; &nbsp; ②

将 ② 代入 ① 得：

**f(n) = 2 * f(n - 1)**

即 n 级台阶有 **2 ^ (n - 1)** 种跳法。

Solution 1 : (17 ms , 8624 K)

```Java
    int JumpFloorII(int target) {
        if (target <= 0) {
            return 0;
        }
        return 1 << (target - 1);
    }
```

**思路 2：**

n 级台阶中，每级台阶只有「跳」与「不跳」两种可能。但是第 n 级台阶必须要跳，否则不算到达终点。那么，最终的跳法为 **2^n / 2 = 2^(n - 1)** 种。


