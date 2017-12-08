---
title: 剑指 Offer
date: 2017-12-9
categories: "算法"
tags:
     - 算法
---






### [二维数组中的查找](https://www.nowcoder.com/practice/abc3fe2ce8e146608e868a70efebf62e?tpId=13&tqId=11154&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

时间限制：1 秒 空间限制：32768 K

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