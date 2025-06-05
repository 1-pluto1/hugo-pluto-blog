---
title: 保研机试day1
date: 2025-06-01T11:30:03+00:00
tags:
  - 每日一题
  - Cpp
  - STL
categories:
  - 保研复习
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


题单主要是鼠群群友书写的感觉还不错，就先拿这个练练手吧。链接：https://vjudge.net/article/8781



### 洛谷-P5734 文字处理软件

代码：

```c++

#include<bits/stdc++.h>

using namespace std;

  

int q, opt;

string str1;

int main(){

cin >> q >> str1;

while (q--)

{

int m, n;

string str2;

cin >> opt;

if (opt == 1){

cin >> str2;

str1 += str2;

cout << str1 << endl;

}

if (opt == 2){

cin >> m >> n;

str1 = str1.substr(m, n);

cout << str1 << endl;

}

if (opt == 3){

cin >> m >> str2;

str1.insert(m ,str2);

cout << str1 << endl;

}

if (opt == 4){

cin >> str2;

if (str1.find(str2) < str1.size()){

cout << str1.find(str2) << endl;

} else{

cout << -1 << endl;

}

}

}

return 0;

}

```



### 洛谷-P1603

```c++
#include<bits/stdc++.h>

using namespace std;

  

map<string, int> q;

  

string s;

int nums[10];

int cnt;

int main(){

q["one"]=1;q["two"]=2;q["three"]=3;q["four"]=4;q["five"]=5;q["six"]=6;q["seven"]=7;q["eight"]=8;q["nine"]=9;q["ten"]=10;

q["eleven"]=11;q["twelve"]=12;q["thirteen"]=13;q["fourteen"]=14;q["fifteen"]=15;q["sixteen"]=16;q["seventeen"]=17;q["eighteen"]=18;q["nineteen"]=19;q["twenty"]=20;

q["a"]=1;q["both"]=2;q["another"]=1;q["first"]=1;q["second"]=2;q["third"]=3;

  

for (int i = 0; i < 6; i++){

cin >> s;

if (q[s]){

int value = q[s] * q[s] % 100;

if (value == 0) continue;

nums[cnt++] = value;

}

}

  

sort(nums, nums + cnt);

cout << nums[0];

for (int i = 1; i < cnt; i++){

if (nums[i] < 10) cout << 0;

cout << nums[i];

}

return 0;

  
}
```


### 洛谷-P1308

```c++

#include<bits/stdc++.h>

using namespace std;

  

int ans;

string article;

string target;

int cur = -1;

string temp;

  

int main(){

cin >> target;

for (char& c : target) c = tolower(c);

getchar();

  

getline(cin, article);

article += ' ';

for (char& c : article) c = tolower(c);

for (int i = 0; i < article.size(); i ++){

if (article[i] != ' '){

temp += article[i];

}else{

if (temp == target){

ans ++;

if (cur == -1) cur = i - temp.size();

}

temp = "";

}

}

if (ans == 0){

cout << -1;

} else{

cout << ans << " " << cur;

}

  

return 0;

}
```


