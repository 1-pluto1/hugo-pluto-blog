---
title: STL学习 02
date: 2023-03-21T11:30:03+00:00
tags:
  - STL
  - Cpp
categories:
  - Library
  - Programming Language
author: ZhaoYang
showToc: true
TocOpen: true
draft: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---
### 二、Stack

#### 2.1 介绍

栈作为数据结构中的一种，是STL实现的一个先进先出，后进后出的容器。

```c++
//头文件添加
#include<stack>

//声明
stack<int> s;
stack<string> s;
stack<node> s;//node是结构体类型
```

#### 2.2 方法函数

| 代码        | 含义                           |
| ----------- | ------------------------------ |
| s.push(ele) | 元素ele入列，增加元素O(1)      |
| s.pop()     | 移出栈顶元素O(1)               |
| s.top()     | 取得栈顶元素（但是不删除）O(1) |
| s.empty()   | 检测栈内是否为空，空为真O(1)   |
| s.size()    | 返回栈内元素个数O(1)           |
|             |                                |

#### 2.3 栈遍历

##### 2.3.1 栈遍历

栈只能对栈顶元素进行操作，如果想要进行遍历，只能将栈中元素一个个取出来存在数组中

```c++
stack<int> st;
for (int i = 0; i < 10; ++i) st.push(i);
while (!st.empty()) {
    int tp = st.top(); // 栈顶元素
    st.pop();
}
```

##### 2.3.2 数组模拟栈进行遍历

通过一个数组对栈进行模拟，一个存放下标的变量**top**模拟指向栈顶的指针。

一般来说单调栈和单调队列写法均可以使用额外变量**tt**或者**hh**来进行模拟

特点：比STL的stack速度更快，遍历元素方便

```c++
int s[100]; // 栈 从左至右为栈底到栈顶
int tt = -1; // tt 代表栈顶指针,初始栈内无元素，tt为-1

for(int i = 0; i <= 5; ++i) {
	//入栈 
	s[++tt] = i;
}
// 出栈
int top_element = s[tt--]; 

//入栈操作示意
//  0  1  2  3  4  5  
//                tt
//出栈后示意
//  0  1  2  3  4 
//              tt
```



