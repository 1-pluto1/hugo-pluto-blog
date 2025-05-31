---
title: STL学习 06
date: 2023-03-23T11:30:03+00:00
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
### 十一、array数组

#### 11.1 介绍

头文件：

```c++
#include<array>
```

`array`是C++11新增的容器，效率与普通数据相差无几，比`vector`效率要高，自身添加了一些成员函数。

和其它容器不同，`array` 容器的大小是**固定**的，无法动态的扩展或收缩，**只允许访问或者替换存储的元素。**

**注意：**`array`的使用要在`std`命名空间里

#### 11.2 简实使用

##### 11.2.1 声明和初始化

**基础数据类型**

声明一个大小位100的int型数组，元素的值不确定

```c++
array<int, 100> a;
```

声明一个大小为100的int型数组，初始值均为0（初始值与默认元素类型等效）、

```c++
array<int, 100> a{};
```

声明一个大小为100的`int`型数组，初始化部分值，其余全部为`0`

```c++
array<int, 100> a{1, 2, 3};
```

或者可以用等号

```c++
array<int, 100> a = {1, 2, 3};
```

**高级数据类型**

不同于数组的是对元素类型不做要求，可以套结构体

```c++
array<string, 2> s = {"ha", string("haha")};
array<node, 2> a;
```

##### 11.2.2. 取存元素值

**修改元素**

```c++
array<int, 4> a = {1, 2, 3, 4};
a[0] = 4;
```

**访问元素**

下标访问:

```c++
array<int, 4> a = {1, 2, 3, 4};
for(int i = 0; i < 4; i++) 
    cout << a[i] << " \n"[i == 3];
```

利用`auto`访问:

```c++
for(auto i : a)
    cout << i << " ";
```

迭代器访问:

```c++
auto it = a.begin();
for(; it != a.end(); it++) 
    cout << *it << " ";
```

`at()`函数访问：

下标为`1`的元素加上下标为`2`的元素，答案为`5`

```c++
array<int, 4> a = {1, 2, 3, 4};
int res = a.at(1) + a.at(2);
cout << res << "\n";
```

`get`方法访问:

将`a`数组下标为`1`位置处的值改为`x`.获取的下标只能写数字，不能填变量

```c++
get<1>(a) = x;
```

#### 11.3 成员函数

| 成员函数            | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| begin()             | 返回容器中第一个元素的访问迭代器（地址）                     |
| end()               | 返回容器最后一个元素之后一个位置的访问迭代器（地址）         |
| rbegin()            | 返回最后一个元素的访问迭代器（地址）                         |
| rend()              | 返回第一个元素之前一个位置的访问迭代器（地址）               |
| size()              | 返回容器中元素的数量，其值等于初始化 array 类的第二个模板参数`N` |
| max_size()          | 返回容器可容纳元素的最大数量，其值始终等于初始化 array 类的第二个模板参数 N |
| empty()             | 判断容器是否为空                                             |
| at(n)               | 返回容器中 n 位置处元素的引用，函数会自动检查 n 是否在有效的范围内，如果不是则抛出 out_of_range 异常 |
| front()             | 返回容器中第一个元素的直接引用，函数不适用于空的 array 容器  |
| back()              | 返回容器中最后一个元素的直接引用，函数不适用于空的 array 容器。 |
| data()              | 返回一个指向容器首个元素的指针。利用该指针，可实现复制容器中所有元素等类似功能 |
| fill(x)             | 将 `x` 这个值赋值给容器中的每个元素,相当于初始化             |
| array1.swap(array2) | 交换 array1 和 array2 容器中的所有元素，但前提是它们具有相同的长度和类型 |
|                     |                                                              |





#### 11.4 部分用法示例

##### data()

指向底层元素存储的指针。对于非空容器，返回的指针与首元素地址比较相等。

##### at()

下标为`1`的元素加上下标为`2`的元素，答案为`5`

```c++
array<int, 4> a = {1, 2, 3, 4};
int res = a.at(1) + a.at(2);
cout << res << "\n";
```

##### fill()

array的`fill()`函数，将`a`数组全部元素值变为`x`

```c++
a.fill(x);
```

另外还有其它的fill()函数:将a数组[begin,end]全部值变为x
```c++
fill(a.begin(), a.end(), x);
```

##### get方法获取元素值

将`a`数组下标为`1`位置处的值改为`x`

⭐️注意⭐️获取的下标只能写数字，不能填变量

```c++
get<1>(a) = x;
```

##### 排序

```c++
sort(a.begin(), a.end());
```

### 十二、 tuple

#### 12.1 介绍

tuple模板是pair的泛化，可以封装不同类型任意数量的对象。

可以把tuple理解为pair的扩展，tuple可以声明二元组，也可以声明三元组。

tuple可以等价为**结构体**使用

❤️ 头文件

```cpp
#include<tuple>
```

#### 12.2 基础用法

##### 12.2.1 声明及初始化

 声明一个空的`tuple`三元组

```cpp
tuple<int, int, string> t1;
```

赋值

```cpp
t1 = make_tuple(1, 1, "hahaha");
```

创建的同时初始化

```cpp
tuple<int, int, int, int> t2(1, 2, 3, 4);
```

可以使用pair对象构造tuple对象，但tuple对象必须是两个元素

```cpp
auto p = make_pair("wang", 1);
tuple<string, int> t3 {p}; //将pair对象赋给tuple对象
```

##### 12.2.2 元素操作

获取tuple对象`t`的第一个元素

```cpp
int first = get<0>(t);
```

修改tuple对象`t`的第一个元素

```cpp
get<0>(t) = 1;
```

#### 12.3 函数操作

- 获取元素个数

  ```c++
  tuple<int, int, int> t(1, 2, 3);
  cout << tuple_size<decltype(t)>::value << "\n"; // 3
  ```

- 获取对应元素的值

  通过`get<n>(obj)`方法获取,`n`必须为数字不能是变量

  ```c++
  tuple<int, int, int> t(1, 2, 3);
  cout << get<0>(t) << '\n'; // 1
  cout << get<1>(t) << '\n'; // 2
  cout << get<2>(t) << '\n'; // 3
  ```

- 通过`tie`解包 获取元素值

  `tie`可以让tuple变量中的三个值依次赋到tie中的三个变量中

  ```c++
  int one, three;
  string two; 
  tuple<int, string, int> t(1, "hahaha", 3);
  tie(one, two, three) = t;
  cout << one << two << three << "\n"; // 1hahaha3
  ```

  



