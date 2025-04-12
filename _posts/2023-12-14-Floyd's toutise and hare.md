---
title: Floyd's toutise and hare
layout: post
author:
name: 이민석
date: 2023-12-14 22:00:00 +0900
categories: ["CS","알고리즘"]
tag : ["연결 리스트"]
math: true
---

## 개요
해당 알고리즘은 어떠한 연결 리스트에서 사이클을 찾는 방법으로 여러 방면에서 두루 쓰이는 방법이다.

---

## 연결 리스트
단방향 연결 리스트 $L$에 대하여 $N$ 개의 노드가 존재할 때, 연결 관계는 다음과 같이 표현할 수 있다.

$$
x_1 = x,\space x_{i+1}=f(x_i)\quad (i \le N)
$$ 

만약 사이클이 존재한다면 $x_{N+1}=x_k\ (1 \le k \le N)$ 이다.\
이 때, $x_k$를 구할 수 있다면 우리는 많은 정보를 얻을 수 있을 것이다.

---

## 사이클 찾기
$f(0)=1, f(x)=x+1, f(n-1)=0 (1\le x \lt n-1)$ 이라고 하자. 두 시작 지점 $a,b (a<b)$ 에 대하여 속도가 $v_a, v_b$ 라고 한다면 시간 $t$ 가 흐른 뒤의 위치는 $(a+v_at) \bmod n$ , 
$(b+v_bt) \bmod n$ 가 된다. 만약 두 위치가 같다면 $(b-a+(v_b-v_a)t) \equiv 0 \pmod n$ 이다.\
따라서 속도차이가 1이라면 모든 $n$ 에 대하여 어느 지점에서 시작하든, 두 점은 반드시 만나게 된다.

이를 이용한다면 연결 리스트에서 사이클을 반드시 찾을 수 있다. 

---

### 동작 방식
$a,b$를 연결리스트의 시작지점$s$으로 하자. 이 때, $v_a = 1,\space v_b=2$ 라고 하자.\
그렇다면 $t$ 만큼 시간이 흐른 뒤, $a,b$ 의 위치는 $f^t(a),f^{2t}(a)$ 가 된다.\
이 때, $f^t(a)=x_k$ 라면 $b$ 는 사이클 내부에 위치해 있음은 자명하다.\
따라서 이를 반복하면서 $a=b$ 가 된다면 그 지점은 사이클 내부의 점이 되며 이 때의 위치 $x_p$ 라 한다면 다음의 성질을 만족한다.

$$
f^t(x_p)=f^t(s)
$$

----

#### 증명
먼저 $x_k$부터 사이클이 발생하니 이 사이클을 $f(0)=1, f(x)=x+1, f(n-1)=0 (1\le x \lt n-1)$ 에 대응하자. 그렇다면 $(b-a+(v_b-v_a)t) \equiv 0 \pmod n$ 에 대하여 $t \equiv (a-b) \pmod n$ 이다.\
이 때, $f^m(a)=0$ 이면 $b$ 는 $m \bmod n$ 이므로 $t+m \equiv (a-b+b) \equiv 0 \pmod n$ 이다. 

따라서 $f^t(x_p)=f^t(s)$ 이다. 

---

따라서 $x_k$ 를 찾는 방법은 종합하면 다음과 같다.
```
x ← head
y ← (x → next)
while x != y:
    x ← (x → next)
    y ← (y → next → next)
end
x ← head
while x != y
    x ← (x → next)
    y ← (y → next)
end
```

## 응용
이렇게 $x_k$ 를 구한다면 사이클의 길이도 역시 구할 수 있다.  

```
x ← (start point of cycle)
length ← 0
do:
    x ← (x → next)
    length ← (length + 1)
while x != (start point of cycle)
```