---
title: STL学习 01
date: 2023-03-20T11:30:03+00:00
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
### 一、vector

#### 1.1 介绍

**vector**为可变长数组（动态数组），定义的**vector**数组可以随时添加数值和删除元素。

> **注意：在局部区域中（比如局部函数里面）开vector数组，是在堆空间里开的。**
>
> 在局部区域开数组是在栈空间开的，而栈空间比较小，如果开了非常长的数组就会发生爆栈。故局部数组**不可以**开大长度数组，但是可以开大长度**vector**.

- 头文件

```c++
#include <vector>
```

- 一维初始化

  ```c++
  vector<int> a;//定义了一个名为a的一维数组，数组存储int类型的数据
  vector<double> b;//定义了一个名叫b的一维数组，数组存储double类型的数据
  vector<node> c;//定义了一个名为c的一维数组，数组存储结构体类型数据，node是结构体类型
  ```

  指定**长度**和**初始值**的初始化：

  ```c++
  vector<int> v(n);//定义一个长度为n的数组，初始值默认为0，下标范围为[0,n-1]
  vector<int> v(n,1);// v[0] 到 v[n - 1]所有的元素初始值均为1
  //注意：指定数组长度之后（指定长度后的数组就相当于正常的数组了）
  ```

  初始化中有多个元素：

  ```c++
  vector<int> a{1,2,3,4,5};//数组a中有五个元素，数组长度为5
  ```

  拷贝初始化：

  ```c++
  vextor<int> a(n+1,0);
  vector<int> b(a);//两个数组中的类型必须相同，a和b都是长度为n+1,初始值为0的数组
  vector<int> c = a;//也是拷贝初始化，c和a是完全一样的数组。
  ```

- 二维初始化

  定义第一维固定长度为5，第二维可变化的二维数组：

  ```c++
  vector<int> v[5]//定义可变长二维数组
  //注意：行不可变（只有5行）, 而列可变,可以在指定行添加元素
  //第一维固定长度为5，第二维长度可以改变
  ```

  `vector<int> v[5]`可以这样理解：长度为5的v数组，数组中存储的是`vector<int> `数据类型，而该类型就是数组形式，故`v`为二维数组。其中每个数组元素均为空，因为没有指定长度，所以第二维可变长。可以进行下述操作：

  ```c++
  v[1].push_back(2);
  v[2].push_back(3);
  ```

  **行列均可变：**

  ```c++
  //初始化二维均可变长数组
  vector<vector<int>> v;//定义一个行和列均可变的二维数组
  ```

  应用：可以在v数组中装入多个数组：

  ```c++
  vector<int> t1{1, 2, 3, 4};
  vector<int> t2{2, 3, 4, 5};
  v.push_back(t1);
  v.push_back(t2);
  v.push_back({3, 4, 5, 6}) // {3, 4, 5, 6}可以作为vector的初始化,相当于一个无名vector
  ```

  行列长度均固定n+1行m+1列初始值为0：

  ```c++
  vector<vector<int>> a(n + 1, vector<int>(m + 1, 0));
  ```

  c++17或者c++20支持的形式（不常用），与上面相同的初始化

  ```c++
  vector a(n + 1, vector(m + 1, 0));
  ```

#### 1.2 方法函数

知道了如何定义初始化可变数组，下面就要知道如何添加、删除、修改数据。

**c指定为数组名称**，含义中会注明算法复杂度。

| 代码                             | 含义                                                 |
| -------------------------------- | ---------------------------------------------------- |
| c.front()                        | 返回第一个数据O(1)                                   |
| c.back                           | 返回最后一个数据O(1)                                 |
| c.pop_back()                     | 删除最后一个数据O(1)                                 |
| c.push_back(element)             | 在尾部添加一个数据O(1)                               |
| c.size()                         | 返回实际数据个数(unsigned类型)O(1)                   |
| c.clear()                        | 清除元素个数O(N),N为元素个数                         |
| c.resize(n,v)                    | 改变数组大小为n，n个空间数值赋为v，如果没有默认值为0 |
| c.insert(it,x)                   | 向任意迭代器it插入一个元素x，O(N)                    |
| 例：`c.insert(c.begin() + 2,-1)` | 将`-1`插入`c[2]`的位置                               |
| c.erase(first,last)              | 删除[first,last]的所有元素，O(N)                     |
| c.begin()                        | 返回首元素的迭代器(通俗的来说就是地址),O(N)          |
| c.end()                          | 返回最后一个元素后一个位置的迭代器,O(1)              |
| c.empty()                        | 判断是否为空，为空则返回真，反之则返回假,O(1)        |
|                                  |                                                      |

注意：

- end()返回的是最后一个元素的后一个位置的地址，不是最后一个元素的地址，**所有STL容器均是如此**

- 使用vi.resize(n,v)函数时，若vi之前指定过大小为pre

  - pre>n：即数组大小变小了，数组会保存前n个元素，前n个元素值为原来的值，不是都为v
  - pre<n：即数组大小变大了，数组会在后面插入n-pre个值为v的元素

  也就是说，这个初始值v只对新插入的元素有效。

  ```c++
  #include<bits/stdc++.h>
  using namespace std;
  void out(vector<int> &a) { for (auto &x: a) cout << x << " "; cout << "\n"; }
  int main() {
  	vector<int> a(5, 1);
  	out(a); // 1 1 1 1 1
  	a.resize(10, 2);
  	out(a); // 1 1 1 1 1 2 2 2 2 2
  	a.resize(3, 3);
  	out(a); // 1 1 1
  	return 0;
  }
  ```

  **排序**

  使用sort排序要：sort(c.begin(),c.end());

  对所有元素进行排序，如果要对指定区间进行排序，可以对sort()里面的参数进行加减改动。

  ```c++
  vector<int> a(n + 1);
  sort(a.begin() + 1, a.end()); // 对[1, n]区间进行从小到大排序
  ```

#### 1.3 访问

共三种方法：

- **下标法**：和普通数组一样

  注意：一维数组的下标是从0到v.size()-1，访问之外的数回出现越界错误

  

- **迭代器法**：类似指针一样的访问，首先需要声明迭代器变量，和声明指针变量一样，可以根据代码进行理解。

  ```c++
  vector<int> vi; //定义一个vi数组
  vector<int>::iterator it = vi.begin();//声明一个迭代器指向vi的初始位置
  
  ```

- **使用auto** ：非常简便，但是会访问数组的所有元素（特别注意0位置元素也会访问到）

##### 1.3.1 下标访问

直接和普通数组一样访问就行。

```c++
//添加元素
for(int i = 0; i < 5; i++)
	vi.push_back(i);
	
//下标访问 
for(int i = 0; i < 5; i++)
	cout << vi[i] << " ";
cout << "\n";
```

##### 1.3.2 迭代器访问

类似指针，迭代器就是充当指针的作用。

```c++
vector<int> vi{1, 2, 3, 4, 5};
//迭代器访问
vector<int>::iterator it;   
// 相当于声明了一个迭代器类型的变量it
// 通俗来说就是声明了一个指针变量
```

- 方式一：

  ```c++
  vector<int>::iterator it = vi.begin(); 
  for(int i = 0; i < 5; i++)
  	cout << *(it + i) << " ";
  cout << "\n";
  ```

- 方式二：

  ```c++
  vector<int>::iterator it;
  for(it = vi.begin(); it != vi.end();it ++)
  	cout << *it << " ";
  //vi.end()指向尾元素地址的下一个地址
  
  // 或者
  auto it = vi.begin();
  while (it != vi.end()) {
      cout << *it << "\n";
      it++;
  }
  ```

##### 1.3.3 智能指针

只能遍历完数组，如果要指定的内容进行遍历，需要另行选择方法。

**auto** 能够自动识别并获取类型。

```c++
// 1. 输入
vector<int> a(n);
for (auto &x: a) {
    cin >> x; // 可以进行输入，注意加引用
}
// 2. 输出
vector<int> v;
v.push_back(12);
v.push_back(241);
for(auto val : v) {
	cout << val << " "; // 12 241
}
```

`vector`注意：

- `vi[i]` 和 `*(vi.begin() + i)` 等价，与指针类似。
- `vector`和`string`的`STL`容器支持`*(it + i)`的元素访问，其它容器可能也可以支持这种方式访问，但用的不多，可自行尝试。



