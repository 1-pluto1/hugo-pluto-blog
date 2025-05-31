---
title: STL学习 04
date: 2023-03-22T11:30:03+00:00
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
### 六、map

#### 6.1 介绍 

映射类似于函数的对应关系，每一个**x**对应一个**y**,而**map**是每个键对应一个值。和python的字典非常相似。

```c++
//头文件
#include<map>
//初始化定义
map<string,string> mp;
map<string,int> mp;
map<int,node> mp;//node是结构体类型
```

**map特性：map会按照键的顺序从小到大自动排序，键的类型必须可以比较大小**

#### 6.2 函数方法

##### 6.2.1 函数方法

| 代码                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| mp.find(key)         | 返回值为key的映射的迭代器。O(logN)。注意：用find函数来定位数据出现的位置，它返回一个迭代器。当数据存在时，返回数据所在位置的迭代器，数据不存在时，返回map.end() |
| map.erase(it)        | 删除迭代器对应的键和值O(1)                                   |
| mp.erase(key)        | 根据映射的键删除键和值O(logN)                                |
| mp.erase(first,last) | 删除左闭右开区间迭代器对应的键和值O(last−first)              |
| mp.size()            | 返回映射的对数O(1)                                           |
| mp.clear()           | 清空map中的所有元素O ( N )                                   |
| mp.insert()          | 插入元素，插入时要构造键值对                                 |
| mp.empty()           | 如果map为空，返回true，否则返回false                         |
| mp.begin()           | 返回指向map第一个元素的迭代器（地址）                        |
| mp.end()             | 返回指向map尾部的迭代器（最后一个元素的**下一个**地址）      |
| mp.rbegin()          | 返回指向map最后一个元素的迭代器（地址）                      |
| mp.rend()            | 返回指向map第一个元素前面(上一个）的逆向迭代器（地址）       |
| mp.count(key)        | 查看元素是否存在，因为map中键是唯一的，所以存在返回1，不存在返回0 |
| mp.lower_bound()     | 返回一个迭代器，指向键值>= **key**的第一个元素               |
| mp.upper_bound()     | 返回一个迭代器，指向键值> key的第一个元素                    |
|                      |                                                              |

##### 6.2.2 注意点

**下面说明部分函数方法的注意点**

注意：
查找元素是否存在时，可以使用
①`mp.find()` ② `mp.count()` ③ `mp[key]`
但是第三种情况，如果不存在对应的`key`时，会自动创建一个键值对（产生一个额外的键值对空间）
所以为了不增加额外的空间负担，最好使用前两种方法

##### 6.2.3 迭代器进行正反向遍历

- `mp.begin()`和`mp.end()`用法：

  **用于正向遍历map**

  ```c++
  map<int,int> mp;
  mp[1] = 2;
  mp[2] = 3;
  mp[3] = 4;
  auto it = mp.begin();
  while(it != mp.end()) {
  	cout << it->first << " " << it->second << "\n";
  	it ++;
  }
  ```

  **结果：**

  ```c++
  1 2
  2 3
  3 4
  ```

- `mp.rbegin()`和`mp.rend()`

  **用于逆向遍历map**:

  ```c++
  map<int,int> mp;
  mp[1] = 2;
  mp[2] = 3;
  mp[3] = 4;
  auto it = mp.rbegin();
  while(it != mp.rend()) {
  	cout << it->first << " " << it->second << "\n";
  	it ++;
  }
  ```

  **结果：**

  ```c++
  3 4
  2 3
  1 2
  ```

##### 6.2.4 二分查找

二分查找`lower_bound() upper_bound()`

map的二分查找以第一个元素（即键为准），对**键**进行二分查找
返回值为map迭代器类型

```c++
#include<bits/stdc++.h>
using namespace std;

int main() {
	map<int, int> m{{1, 2}, {2, 2}, {1, 2}, {8, 2}, {6, 2}};//有序
	map<int, int>::iterator it1 = m.lower_bound(2);
	cout << it1->first << "\n";//it1->first=2
	map<int, int>::iterator it2 = m.upper_bound(2);
	cout << it2->first << "\n";//it2->first=6
	return 0;
}
```

#### 6.3 添加元素

```c++
//先声明
map<string, string> mp;
```

- **方式一：**

  ```c++
  mp["学习"] = "看书";
  mp["玩耍"] = "打游戏";
  ```

- **方式二：插入元素构造键值对**

  ```c++
  mp.insert(make_pair("vegetable","蔬菜"));
  ```

- **方式三：**

  ```c++
  mp.insert(pair<string,string>("fruit","水果"));
  ```

- **方式四:**

  ```c++
  mp.insert({"hahaha","wawawa"});
  ```

#### 6.4 访问元素

##### 6.4.1 下标访问

(大部分情况用于访问单个元素)

```c++
mp["菜哇菜"] = "强哇强";
cout << mp["菜哇菜"] << "\n";//只是简写的一个例子，程序并不完整
```

##### 6.4.2 遍历访问

- 方式一：迭代器访问

  ```c++
  map<string,string>::iterator it;
  for(it = mp.begin(); it != mp.end(); it++) {
  	//      键                 值 
  	// it是结构体指针访问所以要用 -> 访问
  	cout << it->first << " " << it->second << "\n";
  	//*it是结构体变量 访问要用 . 访问
  	//cout<<(*it).first<<" "<<(*it).second;
  }
  ```

- 方式二：智能指针访问

  ```c++
  for(auto i : mp)
  cout << i.first << " " << i.second << endl;//键，值
  ```

- 方式三：对指定单个元素访问

  ```c++
  map<char,int>::iterator it = mp.find('a');
  cout << it -> first << " " <<  it->second << "\n";
  ```

- 方式四：c++17特性才具有

  ```c++
  for(auto [x, y] : mp)
  	cout << x << " " << y << "\n";
  //x,y对应键和值
  ```



#### 6.5 与unordered_map的比较

这里就不单开一个大目录讲unordered_map了，直接在map里面讲了。

##### 6.5.1 内部实现原理

**map**：内部用**红黑树**实现，具有**自动排序**（按键从小到大）功能。

**unordered_map**：内部用**哈希表**实现，内部元素无序杂乱。

##### 6.5.2 效率比较

**map：**

- 优点：内部用红黑树实现，内部元素具有有序性，查询删除等操作复杂度为O ( l o g N ) O(logN)O(logN)
- 缺点：占用空间，红黑树里每个节点需要保存父子节点和红黑性质等信息，空间占用较大。

**unordered_map：**

- 优点：内部用哈希表实现，查找速度非常快（适用于大量的查询操作）。

- 缺点：建立哈希表比较耗时。

  

两者方法函数基本一样，差别不大。

**注意：**

- 随着内部元素越来越多，两种容器的插入删除查询操作的时间都会逐渐变大，效率逐渐变低。


- 使用[]查找元素时，如果元素不存在，两种容器都是创建一个空的元素；如果存在，会正常索引对应的值。所以如果查询过多的不存在的元素值，容器内部会创建大量的空的键值对，后续查询创建删除效率会大大降低。


- 查询容器内部元素的最优方法是：先判断存在与否，再索引对应值（适用于这两种容器）

  ```c++
  // 以 map 为例
  map<int, int> mp;
  int x = 999999999;
  if(mp.count(x)) // 此处判断是否存在x这个键
      cout << mp[x] << "\n";   // 只有存在才会索引对应的值，避免不存在x时多余空元素的创建
  ```

另外：

> 还有一种映射：`multimap`
>
> 键可以重复，即一个键对应多个值，如要了解，可以自行搜索。

### 七 、set

#### 7.1 介绍

set容器中的元素不会重复，当插入集合中已有的元素时，并不会插入进去，而且set容器里的元素自动从小到大排序。

即：set里面的元素**不重复且有序**

```c++
//头文件
#include<set>
//初始化定义
set<int> s;
```

#### 7.2 函数方法

| 代码                | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| s.begin()           | 返回set容器的第一个元素的地址（迭代器）O(1)                  |
| s.end()             | 返回set容器的最后一个元素的下一个地址（迭代器）O(1)          |
| s.rbegin()          | 返回逆序迭代器，指向容器元素的最后一个位置O(1)               |
| s.rend()            | 返回逆序迭代器吗，指向容器第一个元素前面的位置O(1)           |
| s.clear()           | 删除set容器中的所有的元素，返回unsigned int类型O(N)          |
| s.empty()           | 判断set容器是否为空O(1)                                      |
| s.insert()          | 插入一个元素                                                 |
| s.size()            | 返回当前set容器中的元素个数O(1)                              |
| erase(inerator)     | 删除定位器iterator指向的值                                   |
| erase(first,second) | 删除定位器first和second之间的值                              |
| erase(key_value)    | 删除键值key_value的值                                        |
| s.find(element)     | 查找set中的某一个元素，有则返回该元素对应的迭代器，无则返回结束迭代器 |
| s.count(element)    | 查找set中的元素出现的个数，由于set中元素唯一，此函数相当于直接询问element是否出现 |
| s.lower_bound(k)    | 返回大于等于k的第一个元素的迭代器O(logN)                     |
| s.upper_bound(k)    | 返回大于k的第一个元素的迭代器O(logN)                         |
|                     |                                                              |

#### 7.3 访问

- 迭代器访问

  ```c++
  for(set<int>::iterator it = s.begin();it!=s.end();it++)
      cout << *it << " ";
  ```

- 智能指针

  ```c++
  for (auto i : s)
      cout << i << endl;
  ```

- 访问最后一个元素

  ```c++
  //第一种
  cout << *s.rbegin() << endl;
  ```

  ```c++
  //第二种
  set<int>::iterator iter = s.end();
  iter--;
  cout << (*iter) << endl; //打印2;
  ```

  ```c++
  //第三种
  cout << *(--s.end()) << endl;
  ```



#### 7.4 重载<运算符

- 基础数据类型

  方式一：改变set排序规则，set中默认使用less比较器，即从小到大排序。（常用）

  ```c++
  set<int> s1; // 默认从小到大排序
  set<int, greater<int> > s2; // 从大到小排序
  ```

  方式二：重载运算符。（很麻烦，不太常用，没必要）

  ```c++
  //重载 < 运算符
  struct cmp {
      bool operator () (const int& u, const int& v) const {
         // return + 返回条件
         return u > v;
      }
  };
  set<int, cmp> s; 
  
  for(int i = 1; i <= 10; i++)
      s.insert(i);
  for(auto i : s)
      cout << i << " ";
  // 10 9 8 7 6 5 4 3 2 1
  ```

  方式三：初始化时使用匿名函数定义比较规则

  ```c++
  set<int, function<bool(int, int)>> s([&](int i, int j){
      return i > j; // 从大到小
  });
  for(int i = 1; i <= 10; i++)
      s.insert(i);
  for(auto x : s)
      cout << x << " ";
  ```

- 高级数据结构（结构体）

  直接重载结构体运算符即可，让结构体可以比较。

  ```c++
  struct Point {
  	int x, y;
  	bool operator < (const Point &p) const {
  		// 按照点的横坐标从小到大排序,如果横坐标相同,纵坐标从小到大
  		if(x == p.x)
  			return y < p.y;
  		return x < p.x;
  	}
  };
  
  set<Point> s;
  for(int i = 1; i <= 5; i++) {
      int x, y;
      cin >> x >> y;
      s.insert({x, y});
  }	
  /* 输入
  5 4
  5 2
  3 7
  3 5
  4 8
  */
  
  for(auto i : s)
      cout << i.x << " " << i.y << "\n";
  /* 输出
  3 5
  3 7
  4 8
  5 2
  5 4
  */
  
  ```

#### 7.5 其他set

`multiset`:元素可以重复，且元素有序

`unordered_set` ：元素无序且只能出现一次

`unordered_multiset` ： 元素无序可以出现多次

### 八、 pair

#### 8.1 介绍

pair只含有两个元素，可以看作是只有两个元素的结构体

**应用**：

- 代替二元结构体

- 作为map键值对进行插入（代码如下）

  ```c++
  map<string, int> mp;
  mp.insert(pair<string, int>("xingmaqi",1));
  // mp.insert(make_pair("xingmaqi", 1));
  // mp.insert({"xingmaqi", 1});
  ```

初始化和赋值操作：

```c++
//头文件
#include<utility>
//1.初始化定义
pair<string,int> p("cailiangwei",1);//带初始值的
pair<string,int> p ;//不带初始值的

//2.赋值
p = {"cai",18};
p = make_pair("wang", 18);
p = pair<string, int>("wang", 18);

```

#### 8.2 访问

```c++
//定义结构体数组
pair<int,int> p[20];
for(int i = 0;i < 20; i++){
    //和结构体相似，first代表第一个元素，second表示第二个元素
    cout << p[i].first << " " << p[i].second;
}
```



