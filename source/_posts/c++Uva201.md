---
layout:     post                    # 使用的布局（不需要改）
title:      Uva- 201 - Squares         # 标题
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

> ## 正方形（Squares, ACM/ICPC World Finals 1990, UVa201）
> 原题地址：[Uva201](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&category=&problem=137&mosmsg=Submission+received+with+ID+21445316)

有n行n列（2≤n≤9）的小黑点，还有m条线段连接其中的一些黑点。统计这些线段连成了多少个正方形（每种边长分别统计）。

行从上到下编号为1～n，列从左到右编号为1～n。边用H i j和V i j表示，分别代表边(i,j)-(i,j+1)和(i,j)-(i+1,j)。如图4-5所示最左边的线段用V 1 1表示。图中包含两个边长为1的正方形和一个边长为2的正方形。

{% img /img/c++/01.png %}

> ## 代码实现

```c++
#include <iostream>
#include <array>

using namespace std;
const int MAX_LEN = 10;


bool checkV(const array<array<int, 10>, 10> &arr, int i, int j) {
    return arr[i][j] >= 2;
}

bool checkH(const array<array<int, 10>, 10> &arr, int i, int j) {
    return arr[i][j] == 1 || arr[i][j] == 3;
}

/**
 * 检查以（i, j）为左上顶点是否能组成一个边长为len的正方形
 * @param arr
 * @param i
 * @param j
 * @param len
 */
bool checkBuildSquare(const array<array<int, 10>, 10> &arr, int i, int j, int len) {
    for (int s = i; s < i + len; s++) {
        if (!checkV(arr, s, j) || !checkV(arr, s, j + len))
            return false;
    }
    for (int s = j; s < j + len; s++) {
        if (!checkH(arr, i, s) || !checkH(arr, i + len, s))
            return false;
    }
    return true;
}

void check(const array<array<int, MAX_LEN>, MAX_LEN> &arr, int n) {
    static array<int, MAX_LEN> squares{};
    squares.fill(0);
    int maxLen;
    for (int i = 1; i < n; i++)
        for (int j = 1; j < n; j++) {
            if (arr[i][j] != 3)      //一个点→或者↓没有变则不会是某个正方形的左上端点
                continue;
            maxLen = min(n - i, n - j);
            for (int k = 1; k <= maxLen; k++) {
                if (checkBuildSquare(arr, i, j, k))
                    ++squares[k];
            }
        }
    bool hasSquare = false;
    for (int i = 1; i < MAX_LEN; i++) {
        if (squares[i] == 0)
            continue;
        hasSquare = true;
        cout << squares[i] << " square (s) of size " << i << endl;
    }
    if (!hasSquare) {
        cout << "No completed squares can be found." << endl;
    }
}


int main() {
    /**
     * 1 -> H
     * 2 -> V
     * 3 -> HV
     */
    array<array<int, MAX_LEN>, MAX_LEN> sides{};
    int n, sideNum;
    char c;
    unsigned int i, j;
    int tern = 0;
    bool first = true;
    while (cin >> n >> sideNum) {
        ++tern;
        if(first){
           first = false;
        } else {
            cout << endl << "**********************************" << endl << endl;
        }
        for (auto &s : sides)
            s.fill(0);
        for (int k = 0; k < sideNum; k++) {
            cin >> c >> i >> j;
            if (c == 'H')
                sides[i][j] += 1;
            else if (c == 'V')
                sides[j][i] += 2;
        }
        cout << "Problem #" << tern << endl << endl;

        check(sides, n);

    }
}
```
