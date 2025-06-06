---
title: 保研机试day2
date: 2025-06-02T11:30:03+00:00
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

随手写了道代码随想录：
https://kamacoder.com/problempage.php?pid=1046

```c++

#include<bits/stdc++.h>

using namespace std;

  

int main(){

int n, bagweight;

cin >> n >> bagweight;

vector<int> weights(n, 0);

vector<int> values(n, 0);

  

for (int i = 0; i < n; i++) cin >> weights[i];

for (int i = 0; i < n; i++) cin >> values[i];

  

vector<vector<int>> dp(weights.size(), vector<int>(bagweight + 1, 0));

  

for (int j = weights[0]; j < bagweight + 1; j++) dp[0][j] = values[0];

  

for (int i = 1; i < n; i ++){

for (int j = 0; j <= bagweight; j++){

if (j < weights[i]) dp[i][j] = dp[i - 1][j];

if (j >= weights[i]) dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weights[i]] + values[i]);

}

}

cout << dp[n - 1][bagweight] << endl;

return 0;
}
```

### 洛谷-P1048

```c++
//经典的01背包问题

#include<bits/stdc++.h>

using namespace std;

  
int main(){

int T, M;

cin >> T >> M;

vector<int> values(M, 0);

vector<int> times(M, 0);

  

vector<vector<int>> dp(M, vector<int>(T + 1, 0));

  

for (int i = 0; i < M; i++){

cin >> times[i] >> values[i];

}

  

for (int j = times[0]; j < T + 1; j++) dp[0][j] = values[0];

  

for (int i = 1; i < M; i++){

for (int j = 0; j <= T; j++){

if (j < times[i]) dp[i][j] = dp[i - 1][j];

if (j >= times[i]) dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - times[i]] + values[i]);

}

}

cout << dp[M - 1][T];

return 0;

}

```


### 洛谷-U291802

```c++
//也是01背包都是比较基础的题目
#include<bits/stdc++.h>

using namespace std;

int dp[30][1010];

int nums[30];

int main(){

int n, t;

cin >> n >> t;

for (int i = 1; i <= n; i++){

cin >> nums[i];

dp[i][0] = 1;

}

dp[0][0] = 1;

for (int i = 1; i <= n; i++){

for (int j = 1; j <= t; j++){

if (j >= nums[i]){

dp[i][j] = dp[i - 1][j] + dp[i - 1][j - nums[i]];

}else {

dp[i][j] = dp[i - 1][j];

}

}

}

cout << dp[n][t];

return 0;

}

```


### 洛谷-P1616

```c++
//完全背包问题

#include<iostream>

#include<algorithm>

using namespace std;

int t, m;

const int M = 1e4 + 10, T = 1e7 + 10;

long long dp[T], values[M], times[M];

  

int main(){

cin >> t >> m;

for (int i = 1; i < m + 1; i++) {

cin >> times[i] >> values[i];

}

  

for (int i = 1; i < m + 1; i ++){

for (int j = times[i]; j < t + 1; j++){

dp[j] = max(dp[j], dp[j - times[i]] + values[i]);

}

}

  

cout << dp[t];

return 0;

}
```