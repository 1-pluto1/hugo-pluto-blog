---
title: STL学习 03
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
### 三、queue

#### 3.1 介绍 

队列是一种先进先出的数据结构。

```c++
//头文件
#include<queue>
//定义初始化
queue<int> q;
```

#### 3.2 方法函数

| 代码            | 含义                                         |
| --------------- | -------------------------------------------- |
| q.front()       | 返回队首元素O(1)                             |
| q.back()        | 返回队尾元素，O(1)                           |
| q.push(element) | 尾部添加一个元素element进队O(1)              |
| q.pop()         | 删除第一个元素 出队O(1)                      |
| q.size()        | 返回队列中元素个数，返回值unsigned int，O(1) |
| q.empty()       | 判断是否为空，队列为空，返回true,O(1)        |

#### 3.3 队列模拟

使用**q[]**数组模拟队列

**hh**表示队首元素的下标，初始值为0

**tt**表示队尾元素的下标，初始值为-1，**表示刚开始队列为空**

一般来说单调栈和单调队列写法均可使用额外变量**tt**或**hh**来进行模拟

```c++
#include<bits/stdc++.h>
using namespace std;
const int N = 1e5 +5;
int q[N];
int main(){
    int hh = 0,tt = -1;
    //入队
    q[++tt] = 1;
    q[++tt] = 2;
    //将所有元素出队
    while(hh <= tt){
        int t = q[hh++];
        printf("%d",t);
    }
    return 0;
}
```

#### 3.4 队列模拟例题

题目链接： https://atcoder.jp/contests/abc247/tasks/abc247_d

```c++
#include<bits/stdc++.h>
using namespace std;

using ll = long long ;
using pii = pair<int, int>;
const int N = 250005, mod = 998244353;

int q[N], num[N];

void solve() {
	int qq, hh = 0, tt = -1;
	cin >> qq;
	while(qq--) {
		int op;
		cin >> op;
		int x, c;
		if(op == 1) {
			cin >> x >> c;
			q[++tt] = c;
			num[tt] = x;
		} else {
			cin >> c;
			ll res = 0;
			while(c) {
				if(q[hh] > c) {
					q[hh] -= c;
					res += 1ll * c * num[hh];
					break;
				} else {
					res += 1ll * q[hh] * num[hh];
					c -= q[hh++];
				}
			}
			cout << res << "\n";
		}
	}
}

int main() {
	ios::sync_with_stdio(false);
	cin.tie(0);

	int t;
	// cin >> t;
	t = 1;
	while(t--)
		solve();
	return 0;
}
```

### 四、deque

#### 4.1 介绍

首尾都可以插入和删除的队列为双端队列。

```c++
//添加头文件
#include<deque>
//初始化定义
deque<int> dq;
```

#### 4.2 方法函数

```
注意双端队列的常数比较大。
```

| 代码                                | 含义                               |
| ----------------------------------- | ---------------------------------- |
| push_back(x)/push_front(x)          | 将x插入队尾后/队首O(1)             |
| back()/front()                      | 返回队尾/队首元素O(1)              |
| pop_back()/pop_front()              | 删除队尾/队首元素 O(1)             |
| erase(iterator it)                  | 删除双端队列中的某一个元素         |
| erase(iterator first,iterator last) | 删除双端队列中[first,last)中的元素 |
| empty()                             | 判断deque是否为空,O(1)             |
| size()                              | 返回deque的元素数量O(1)            |
| clear()                             | 清空deque                          |

#### 4.3 注意点

deque可以进行排序

双端队列排序一般不用，感觉毫无用处，使用其他STL仍然可以实现相似的功能

```c++
//从小到大
sort(q.begin(), q.end())
//从大到小排序
sort(q.begin(), q.end(), greater<int>());//deque里面的类型需要是int型
sort(q.begin(), q.end(), greater());//高版本C++才可以用
```

### 五、priority_queue

#### 5.1 介绍

优先队列是在正常队列的基础上加了优先级，保证每次的队首元素都是优先级最大的。

可以实现每次从优先队列中取出的元素都是队列中**优先级最大**的一个。

它的底层是通过堆来实现的。

```c++
//头文件
#include<queue>
//初始化定义
priority_queue<int> q;
```

#### 5.2 函数方法

| 代码                                                  | 含义                        |      |
| ----------------------------------------------------- | --------------------------- | ---- |
| q.top()                                               | 访问队首元素O(1)            |      |
| q.push()                                              | 入队，O(logN)               |      |
| q.pop()                                               | 堆顶（队首）元素出队O(logN) |      |
| q.size()                                              | 队列元素个数O(1)            |      |
| q.empty()                                             | 是否为空O(1)                |      |
| 注意：没有clear()!                                    | 不提供该方法                |      |
| 优先队列只能通过top()访问队首元素（优先级最高的元素） |                             |      |
|                                                       |                             |      |

#### 5.3 设置优先级

##### 5.3.1 基本数据类型的优先级

```c++
priority_queue<int> pq;//默认大根堆，即每次取出的元素是队列中的最大值
priority_queue<int,vector<int>,greater<int>> q;//小根堆，每次取出的元素是队列中的最小值
```

**参数解释**：

- 第一个参数：就是优先队列中存储的数据类型

- 第二个参数：

  vector<int>是用来承载底层数据结构堆的容器，若优先队列中存放的是**double**型数据，就要填**vector<double>**

  总之存的是什么类型的数据，就相应的填写对应的类型，同时也要改动第三个参数里面的对应类型。

- 第三个参数：

  less<int>表示数字大的优先级大，堆顶为最大的数字

  greater<int>表示数字小的优先级大，堆顶为最小的数字

  int代表的是数据类型，也要填优先队列中存储的数据类型。

---

下面介绍基础数据类型优先级设置的写法：

1. 基础写法（常用）

   ```c++
   priority_queue<int> q1; // 默认大根堆, 即每次取出的元素是队列中的最大值
   priority_queue<int, vector<int>, less<int> > q2; // 大根堆, 每次取出的元素是队列中的最大值，同第一行
   
   priority_queue<int, vector<int>, greater<int> > q3; // 小根堆, 每次取出的元素是队列中的最小值
   ```

2. 自定义排序（不常见，主要是书写麻烦）

   ```c++
   struct cmp1 {
   	bool operator()(int x, int y) {
   		return x > y;
   	}
   };
   struct cmp2 {
   	bool operator()(const int x, const int y) {
   		return x < y;
   	}
   };
   priority_queue<int, vector<int>, cmp1> q1; // 小根堆
   priority_queue<int, vector<int>, cmp2> q2; // 大根堆
   ```

   

##### 5.3.2 高级数据类型（结构体）优先级

即优先队列中存储结构体类型，必须要设置优先级，即结构体的比较运算（因为优先队列的堆中要比较大小，才能将对应最大或者最小元素移到堆顶）

优先级设计可以定义在**结构体内**进行小小于号重载，也可以定义在**结构体外**

```c++
//要排序的结构体（存储在优先队列里面的）
struct Point{
int x,y;
};
```

**版本一：自定义全局比较规则**

```c++
//定义的比较结构体
//注意：cmp是个结构体
struct cmp{//自定义堆的排序规则
    bool operator()(const Point& a,const Point& B=b){
        return a.x < b.x;
    }
};
//初始化定义
priority_queue<Point,vector<Point>,cmp> q;//x大的在堆顶
```

**版本二：直接在结构体里面写**

因为是在结构体内部自定义的规则，一但需要比较结构体，自动调用结构体内部重载运算符规则

结构体内部有两种方式：

**方式一**：

```c++
struct node{
    int x, y;
    friend bool operator<(Point a, Point b){//为两个结构体参数，结构体调用一定要写上friend
        return a.x < b.x;//按x从小到大排，x大的在堆顶
    }
}
```

**方法二**：（推荐）

```c++
struct node {
    int x, y;
    bool operator < (const Point &a) const {//直接传入一个参数，不必要写friend
        return x < a.x;//按x升序排列，x大的在堆顶
    }
};
```

优先队列的定义：

```c++
priority_queue<Point> q;
```

注意：优先级队列自定义排序规则和**sort**()函数定义**cmp**函数很相似，但是最后返回的情况是**相反的**。即相同的符号，最后定义的排序顺序是完全相反的。

所以只要记住**sort**的排序规则和优先队列的排序规则是相反的就可以了。

当理解了堆的原理就会发现，堆调整时比较顺序是孩子和父亲节点进行比较，如果是 `>` ，那么孩子节点要大于父亲节点，堆顶自然是最小值。

#### 5.4 存储特殊类型的优先级

##### 5.4.1 存储pair类型

- 排序规则：

  默认对**pair**的**first**进行降序排序，然后对**second**降序排序

  对**first**先排序，大的排在前面，如果**first**元素相同，再对**second**元素排序，保持大的在前面。

  ```c++
  #include<bits/stdc++.h>
  using namespace std;
  int main() {
      priority_queue<pair<int, int> >q;
  	q.push({7, 8});
  	q.push({7, 9});
  	q.push(make_pair(8, 7));
      while(!q.empty()) {
          cout << q.top().first << " " << q.top().second << "\n";
          q.pop();
      }
      return 0;
  }
  结果：
  8 7
  7 9
  7 8
  ```

  



