---
title: Wall Making Game
layout: post
author:
name: 이민석
date: 2023-12-23 23:00:00 +0900
categories: ["PS"]
tag : ["게임이론", "스프라그-그런디 정리"]
math: true
---

[BOJ Wall Making Game](https://www.acmicpc.net/problem/11717)

## 풀이
현재 보드에서 고를 수 있는 셀 $(x,y)$ 을 고르면 보드는 그 셀을 기준으로 다시 4개의 똑같은 게임으로 나누어 진행하게 된다. 

따라서 현재 보드의 범위 $(x_1, y_1) \sim (x_2, y_2)$ 에 대한 게임을 하나의 그런디 수로 표현할 수 있다. (이를 $G(x_1, x_2, y_1, y_2)$ 라 하자.)

이는 고를 수 있는 셀들 $(x,y)$에 대하여 


$\begin{align}
G(x_1, x_2, y_1, y_2)&=\ast (mex(\\{m \| \ast m = G(x_1, x-1,y_1,y-1) + \newline
&G(x + 1, x_2,y_1,y-1) + G(x_1, x-1,y+1,y_2) + G(x+1, x_2,y+1,y_2)\\}))
\end{align}$

과 같이 정의할 수 있다.

그리고 각 범위에 그런디 수를 `메모이제이션` 을 이용하여 중복된 연산을 피할 수 있어, $O(W^3H^3)$ 에 해결할 수 있다. 
