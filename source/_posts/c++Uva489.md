---
layout:     post                    # 使用的布局（不需要改）
title:      Uva- 489 - Hangman Judge         # 标题
subtitle:                          #副标题
date:       2018-6-8 18:00              # 时间
author:     Ming.J                      # 作者
header-img: img/blog-header1.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - c++语言
    - ACM
    - Uva
---

> ## 刽子手游戏（Hangman Judge, UVa 489）
> 原题地址：[Uva489](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=430)

刽子手游戏其实是一款猜单词游戏，如图4-1所示。游戏规则是这样的：计算机想一个单词让你猜，你每次可以猜一个字母。如果单词里有那个字母，所有该字母会显示出来；如果没有那个字母，则计算机会在一幅“刽子手”画上填一笔。这幅画一共需要7笔就能完成，因此你最多只能错6次。注意，猜一个已经猜过的字母也算错。

在本题中，你的任务是编写一个“裁判”程序，输入单词和玩家的猜测，判断玩家赢了（You win.）、输了（You lose.）还是放弃了（You chickened out.）。每组数据包含3行，第1行是游戏编号（-1为输入结束标记），第2行是计算机想的单词，第3行是玩家的猜测。后两行保证只含小写字母。

样例输入：
```
1
cheese
chese
2
cheese
abcdefg
3
cheese
abcdefgij
-1
```

样例输出:

```
Round 1
You win.
Round 2
You chickened out.
Round 3
You lose.
```

> ## 代码实现

```c++
/**
 * UVa489
 */
#include <iostream>
#include <string>
#include <set>
#include <algorithm>

using namespace std;

const int ERROR_TORRENT = 7;
/**
 *
 * @param s
 * @param guess
 * @return  1 -> success  2 -> fail  0 -> give up
 */
int doGuess(string &s, string &guess){
    int errorTime = 0;
    set<char> errorSet;
    bool isMatch;
    bool isFinish;
    for(auto g : guess){
        isMatch = false;
        isFinish = true;
        if(errorSet.find(g) != errorSet.end()) //猜错一次后就不再猜了
            continue;
        errorSet.insert(g);
        for(auto &c : s){
            if(c == ' ')
                continue;
            isFinish = false;
            if(g == c) {
                c = ' ';
                isMatch = true;
            }
        }
        if(isFinish)
            return 1;
        if(!isMatch) {
            ++errorTime;
        }
        if(errorTime >= ERROR_TORRENT)
            return 2;
    }
    for(auto &g : s){
        if(g != ' ')
            return 0;
    }
    return 1;
}

int main(){
    int round;
    string target;
    string guess;
    while(cin >> round){
        if(round == -1)
            break;
        cin >> target >> guess;
        cout << "Round " << round << endl;
        switch (doGuess(target, guess)){
            case 0:
                cout << "You chickened out." << endl;
                break;
            case 1:
                cout << "You win." << endl;
                break;
            case 2:
                cout << "You lose." << endl;
                break;
        }
    }
    return 0;
}
```
