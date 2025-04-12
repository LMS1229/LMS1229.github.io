---
title: TV Show Game
layout: post
author:
name: 이민석
date: 2023-12-05 23:00:00 +0900
categories: ["PS"]
tag : ["강한 연결 요소"]
math: true
---

[BOJ TV Show Game](https://www.acmicpc.net/problem/16367)

## 풀이
$i$ 번 램프가 빨간색인 경우 $x_i$, 파란색인 경우 $\neg x_i$ 라 하자.\
입력으로 `3 R 5 R 6 B` 이 들어왔다면 $(x_3 \land x_5) \lor (x_3 \land \neg x_6) \lor (x_5 \land \neg x_6)$ 가 참이여야 한다.\
이 때, $(x_3 \land x_5) \lor (x_3 \land \neg x_6) \lor (x_5 \land \neg x_6) = $ $(x_3 \lor x_5) \land (x_3 \lor \neg x_6) \land (x_5 \lor \neg x_6)$ 이다.\
따라서 이는 2-sat 문제로 볼 수 있고 이는 강한 연결 요소를 이용해 풀 수 있다. 이는 추후에 포스팅하겠다. 