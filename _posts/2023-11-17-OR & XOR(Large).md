---
title: "[BOJ] OR & XOR(Large)"
author:
name: 이민석
date: 2023-11-17 19:28:00 +0900
categories: ["PS"]
tags: ["다이나믹 프로그래밍","비트마스크"]
math: true
---


[BOJ OR And XOR(Large)](https://www.acmicpc.net/problem/30523)

## 풀이

먼저, $\oplus$ 연산을 $ \lor $ 연산으로 바꾸게 되면 $x \lor y = x\oplus y + x  \And y$ 가 된다. 따라서 $N^2$ 개의 $A_i  \And B_j (1 \le i,j \le n)$ 연산 중, 가장 큰 $p$ 개를 고르면 된다.


$A$ 에 존재하는 수 $A_i$  중, $A_i  \And x = x$ 의 개수를 $count(A,x)$, B에 존재하는 수 $B_i$ 중,  $B_i \And x = x$ 의 개수를 $count(B,x)$ 라 하자.

그렇다면 $A_i  \And B_j = Result_{i,j} (1 \le i,j \le n) $ 에 대하여 $Result_{i,j}  \And x = x$ 를 만족하는 $(i,j)$ 의 경우의 수$count(Result,x) = count(A,x)\times count(B,x)$ 가 된다. 

$count(A,x)$ 는 어떻게 구할 수 있을까?

### $count(A,x)$

$A_{cnt,y}[i] = A$ 에 존재하는 원소 $i$ 에 대하여 $(i \lor \sum_{k=0}^{y-1}(1 \ll k)) \And i = i$ 의 개수 하자. 
배열 $A$ 에 존재하는 $i$ 의 개수는 $A_{cnt,0}[i]$ 가 되며

$i \And (2^{y-1}) = i$ 라면 $A_{cnt,y}[i] = A_{cnt,y-1}[i]$ 이고\
$i \And (2^{y-1}) = 0$ 라면 $A_{cnt,y}[i] = A_{cnt,y-1}[i] + A_{cnt,y-1}[i \lor 2^{y-1}]$ 가 된다.

따라서 $count(A,x)=A_{cnt,y=17}[x]$ 가 된다.

위와 같은 과정으로 $count(B,x)$ 도 구할 수 있다. 


그러나 우리는 최종적으로 $Result_{i,j}=x$ 가 되는 경우를 구해야 한다. 이는 위의 $count(A,x)$ 를 구하는 과정과 유사하다. 

$Result_{cnt,y}[x] = x \lor \sum_{k=y}^{16}(1 \ll k) \And x= Result_{i,j}$ 를 만족하는 $(i,j)$ 의 개수 라 하자. 

$count(Result,x)=Result_{cnt,0}[x]$ 이다.

$x \And (2^{y}) = i$ 라면 $Result_{cnt,y+1}[x] = Result_{cnt,y}[x]$ 이고\
$x \And (2^{y}) = 0$ 라면 $Result_{cnt,y+1}[x] = Result_{cnt,y}[x] - Result_{cnt,y}[x \lor 2^{y}]$ 가 된다.

따라서 최종적으로 $Result_{i,j}=x$ 인 $(i,j)$ 의 개수는 $Result_{cnt,17}[x]$ 가 된다.

이를 이용해 모든 $0 \le x \lt 2^{u=17}$ 에 대하여 $Result_{cnt,17}$ 를 구한 후, $x$ 가 큰 순서대로 $p$ 개의 결과를 더하면 된다.

각 과정은 모두 배열에 (존재할 수 있는 최댓값)$\lt 2^u$ 에 $0 \sim 2^u-1$ 까지 모든 수에 대하여 $u$ 개의 비트만큼 반복하므로 $O(u2^u)$ 의 시간 복잡도를 가진다.

그리고 $\sum_{i=1}^{N} \sum_{j=1}^{N} {A_i \oplus B_j}$ 는 각 비트별로 개수를 세어 비트단위로 다음과 같이 계산한다.
$$
\sum_{i=0}^{16}((1 \ll i)((N-bitCount(A,i))bitCount(B,i)+bitCount(A,i)(N-bitCount(B,i))))
$$
$$
bitCount(A,i)=n(\{A_x | A_x \in A, A_x \And (1 \ll i) = 1 \ll i\})
$$