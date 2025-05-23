---
title: "[알고리즘] numpy의 다차원 배열간의 broadcast 를 1차원 배열간의 연산으로 변환"
author:
name: 이민석
date: 2022-03-09 23:30:00 +0900
categories: ["알고리즘"]
tag: ["구현", "알고리즘"]
math: true
---

아는 지인의 부탁으로 Numpy의 broadcast를 C로 구현해달란 부탁을 받았다...할 순 있으니 해봐야지...
# Numpy의 broadcasting
Numpy에서 배열의 연산은 보통 원소별로 짝을 맞추어 연산을 진행한다.

 ```python
a=numpy.array([1,2,3])
b=numpy.array([2,3,4])
a*b
array([ 2,   6, 12])
a+b
array([ 3,   5, 12])
 ```

그러나 특정 조건을 만족하면 더 작은 배열이 연산이 가능한 큰 배열로 확장한다. 이를 `broadcasting` 이라 한다. 하나의 예시로 array에 스칼라 값을 더하면 array에 있는 모든 원소에 스칼라 값을 더하게 된다.
 ```python
a=numpy.array([1,2,3])
a+1
array([2, 3, 4])
 ```

# 조건
두 array에 대해서 연산할 때, 둘의 `shape`에 있는 원소를 비교한다. 이 때 비교 순서는 둘의 `shape`를 가장 뒤(오른쪽)에서부터 앞(왼쪽)으로 이동하며 비교한다. 그러면서 각 비교가 모두 다음 조건 중 하나를 만족해야한다. 

```
1. 두 값이 같다.
2. 둘 중 하나가 1이다.
```

그러나 두 배열의 차원이 같을 필요는 없다. 만약 `256 x 256 x 16` 의 배열과 `16`개의 값을 갖는 배열에 대해서 연산을 할 수 있다.

`broadcast`의 규칙에 의하면 `shape`의 가장 오른쪽부터 비교하면 위의 조건에 만족함을 알 수 있다.

```
A       (3차원) : 256 x 256 x 16
B       (1차원) :             16
Result  (3차원) : 256 x 256 x 16

```

만약 같은 차원의 크기을 비교했을 때, 한 쪽이 1이라면 다른 값이 연산결과의 해당차원 값이 된다. 즉, 크기가 1인 차원은 비교한 차원의 크기와 같도록 늘어나거나 복사된다.

```
A       (4차원) : 12 x 4 x 1 x 5 
B       (3차원) :      1 x 5 x 5
Result  (4차원) : 12 x 4 x 5 x 5
```

# 예시
1차원 배열과 2차원 배열의 덧셈 연산을 예로 들면 다음과 같다.

 ![Desktop View](/assets/img/[구현]numpy의broadcast를C++로구현/broadcast예시.png)

2차원 배열과 1차원 배열의 차원의 크기를 비교하면 1차원에서 서로 3으로 같음을 알 수 있다. 따라서 `broadcast`를 진행해 1차원 배열을 2차원 배열의 각 행에 추가 되어 연산을 수행한다.

# 본격적인 구현

## 조건
1. 다차원 배열을 모두 1차원배열로 치환해야한다. 

## 관찰
먼저 `broadcast`가 가능한 두 배열 중, 더 높은 차원을 가진 배열을 $H$, 다른 하나를 $L$이라 하자. 둘의 배열이 $n$차원,$m$차원일때, 배열의 모양(`shape`)을 $ (h_1, h_2, h_3, ... , h_n) ,(l_1, l_2, l_3, ... , l_m) $ 이라 하자. (Numpy.Array의 shape튜플이라 생각하면된다.)
그렇다면 연산 결과 result 배열$R$은 다음과 같은 상태가 된다. 
- 모든 예시에서 연산자의 좌측에 있는 피연산자의 차원이 더 높다고 가정하겠다.

>1. n차원을 가진다.
>2. $R$의 shape $ (r_1,r_2,...,r_n) = (h_1, h_2, h_3, ...max(h_{n-m-1},l_1),\dots ,max(h_n,l_m)) $ 이다.

그리고 $ k(2 \le k \le  n)$ 까지의 연산 결과를 얻기 위해선 $ k-1 $ 차원까지의 연산결과를 가지고 $k$차원까지의 연산결과를 이용하면 된다.

## 재귀함수로 구현
이를 토대로 재귀함수형태로 시도할려 했으나, 지인이 재귀함수는 사용할 수 없다고 알려줬다...그래서 스택을 이용하기로 했다.

## 스택을 이용한 구현

### 연산 결과와 연산하는 두 배열의 관계

연산 결과 $Result$에 대해서 각 차원에 대한 인덱스를 이용해 원소의 관계를 표현하면 다음과 같다.

>$ R[0][0][0]...[0] = H[0][0][0]...[0]\left(operator\right)L[0][0]...[0]$이다. 

이때 만약 연산하는 두 배열의 모든 차원의 크기가 1이 아니라고 하자. 이 때, 마지막 인덱스를 1 늘리면 다음과 같다.

>$ R[0][0][0]...[1] = H[0][0][0]...[1]\left(operator\right)L[0][0]...[1] $

이 때, 세 배열의 차원별 인덱스 변화를 스택 배열의 형태로 표현하면 다음과 같다. 
(이를 `인덱스 스택`이라 부르겠다.)

>$R\\_IndexStack=[0,0,0,0,0,...,0] \rightarrow [0,0,0,0,0,...,1]$
>
>$H\\_IndexStack=[0,0,0,0,...,0] \rightarrow [0,0,0,0,...,1]$
>
>$L\\_IndexStack=[0,0,0,0,...,0] \rightarrow [0,0,0,0,...,1]$

즉, 마지막 원소가 모두 1씩 늘어나게 된다. 이는 3개의 스택이 동시에 동작함을 의미한다. 만약 인덱스 스택의 탑이 $r_n$ 이 되면 다음과 같이 진행하면 된다.

>1. 탑을 제거한다.
>2. 탑 원소 값을 1증가시킨다.
>3. 스택의 길이가 n이 되도록 0을 삽입한다.

이를 각 스택에 기본적인 수행 과정은 아래와 같다.

>1. 현재 스택의 길이를 k라 하자. 스택의 탑에 있는 값을 1 늘렸을 때 $r_k$ 가 된다면 스택에서 탑을 제거하고 1의 과정을 반복한다.
>2. 만약 스택이 비어진다면 연산을 종료한다.
>3. 스택이 비지 않았다면 스택의 탑에 있는 값을 1늘리고 스택의 길이가 $n$이 될 때 까지 0을 삽입한다.

하지만 $n \not =m $이면 필연적으로 $L \\_ IndexStack$은 비었지만 $H \\_ IndexStack$은 비어있지 않을 수 있다.

이 경우엔 $L\\_IndexStack$은 비어둔 채, $H\\_IndexStack$과 $R\\_IndexStack$에서만 위의 과정을 반복한다.
그리고 연산이 종료되지 않으면 $L\\_IndexStack$을 0으로 채우면 된다.

---

그리고 우리는 `broadcast`에서 두 배열의 같은 차원에서 차원의 크기가 한 쪽이 1이라면 다른 쪽의 크기에 맞추어 복사된 뒤, 연산을 진행 하는 것을 안다. 따라서 두 배열의 인덱스 스택에 대해서 다르게 처리해야한다.

>1. 두 인덱스 스택의 탑의 값이 1인 경우, 차원의 크기가 1이 아닌 경우와 똑같이 처리한다.
>2. 만약 한 쪽만 1인 경우, 다른 쪽 스택의 탑을 확인하여 그 원소를 제거할지를 정하여 제거해야 한다면 두 스택의 탑을 모두 제거하고 새로운 탑을 비교한다. 아니라면 다른 쪽 인덱스 스택의 탑을 1 증가 시킨다.
>3. 스택의 길이가 차원의 길이와 같도록 0을 삽입한다.

그렇다면 $R$의 인덱스 스택은 어떻게 될까? $R$의 각 차원의 크기들은 크기가 큰 쪽에 맞춰지기때문에 달라질 건 없다.

이런 과정을 거치며 $R$의 원소를 채우면 $R$의 모든 원소를 구할 수 있다.

### 증명

#### $R$ 에 있는 모든 원소에 접근할 수 있는가?

n차원 배열 $R$ 의 형태를 $(r_1,r_2,r_3,r_4,...,r_n)$ 이라 하자.
이는 Numpy.Array를 이용하여 만든 R의 shape와 같다.

이 때, $R$ 의 임의의 원소에 대응되는 인덱스 튜플을 $(i_1,i_2,i_3,i_4,...,i_n)$ 이라 하자.

이에 대해 다음과 같이 정의하겠다.

1. $R(i_1,i_2,i_3,...,i_k) = R[i_1][i_2][i_3]...[i_k] (1\le k\le n)$

그렇다면 다음 성질을 가진다.

$$\forall k\in[1,n] : 0\le i_k \le r_k-1 (k\in\mathbb{N})$$

이제 위의 알고리즘을 이용해 다음과 같이 표현할 수 있다.
1. $k=n$, 현재 인덱스 튜플을 $(i_1,i_2,i_3,i_4,...,i_n)$ 이라 하자. 
2. $k$ 가 0 이 아니고 $i_k=r_k-1$ 이면 k를 1 감소하고 다시 2 과정으로 간다. 아니라면 3번 과정으로 간다.
3. $k$가 0이면 종료한다. 
4. $i_k$를 1 늘린다.
5. $i_{k+1} \sim i_n $을 모두 0으로 바꾼다.
6. 1번과정을 다시 진행한다.

이제 위 알고리즘이 n차원 배열 R의 모든 원소를 접근할 수 있는지 알아보자. 
그 전에, 정리를 하나하자. 

>$ R_k= \\{0,1,2,3,...,r_k-1 \\} $ 이면 R의 서로 다른 모든 인덱스 튜플의 집합은 $\prod_{i=1}^n R_i$ 이다.

1. 임의의 인덱스 튜플 $T$에 대해서 $R$의 한 원소에 접근할 수 있을 때, 모든 $T$에 대한 집합을 $T_R$이라고 하자. 이 때, $T_R=\prod_{i=1}^n R_i$ 이다.
2. 먼저 $T=(0_1,0_2,0_3,0_4,...,0_n)$ 이라 하자. (모두 같은 0이지만 튜플의 길이를 구분하기 위해 아랫첨자를 썼다.)
3. 위의 알고리즘을 T에 적용했을 때, 2번 과정을 탈출했을 때, $k=n-1$ 되는 순간은 1~4의 과정을 $r_n$번 반복하는 경우이다. 이 때까지 거쳐간 튜플들은 $ (0\_1,0\_2,0\_3,0\_4,...,0\_{n-1}) \times R\_n $ 이다.
4. $k=n-2$ 가 되는 순간은 $k=n-1$이 되는 순간을 $r_{n-1}$ 반복했을 때이다. 왜냐하면 $i_{n-1}$이 1이 증가하기 위해선 $i_n$ 이 $r_n$ 증가해야하기 때문이다. 따라서 이때까지 방문한 튜플은 $ (0\_1,0\_2,0\_3,...,0\_{n-2})\times R\_{n-1}\times R\_n $ 이다.
5. 위 과정을 $k=0$ 일때 까지 반복하면 방문한 튜플들은 $R_1 \times R_2 \times R_3 \times...\times R_n$ 이 된다. 그러나 위에서 봤듯이  $T_R=\prod_{i=1}^n R_i=R_1 \times R_2 \times R_3 \times...\times R_n$ 이므로 위 알고리즘을 적용했을 때, 서로 다른 모든 인덱스 튜플에 접근할 수 있다.

#### $H\\_IndexStack$ 과 $L\\_IndexStack$ 에 저장된 인덱스로 구할 수 있는 원소들의 연산결과가 $R\\_IndexStack$ 에 저장된 인덱스로 구할 수 있는 $R$ 의 원소위치에 저장되는 것이 옳은가?

먼저 broadcast가 가능한 두 배열의 Shape $ (h_1, h_2, h_3, ... , h_n) ,(l_1, l_2, l_3, ... , l_m) $ 에서 $l_m=h_n, l_{m-1}=h_{n-1} , ... , l_1=h_{n-m+1}$ 일 때  $n=m$ 이면 위의 명제는 자명하게 참이다. 

그럼 이 상태에서 $n>m$ 일때를 보자.

1. 먼저 위에서 봤듯이 인덱스 스택이 $(0_1,0_2,0_3,...,0_m)$ 부터 시작된다면 $L$ 에 있는 모든 원소에 접근할 수 있다. 
2. $n=m+1$ 이라면 연산시 `broadcast`에 의해 $L$의 모양은 $n+1$ 로 확장되어 $(h_1,l_1, l_2, l_3, ... , l_m) $ 의 모양이 된다. 확장된 $L$ 을 $ExtendL$ 라 하면 $ExtendL[0]=ExtendL[1]=...=ExtendL[h_1-1]$ 이 된다. 따라서 $n=m+1$ 일 때, $ExtendL$ 의 모든 원소에 접근하기 위해선 $L$ 의 모든 원소에 접근하는 연산을 $H\\_IndexStack$ 에서 $h_1$ 에 대응하는 원소 값이 1 늘어날때마다 반복하면 된다. 따라서 현재 각 스택에 있는 값들을 $(ri\\_1,...,ri\\_n),(hi\\_1,...,hi\\_n)$ 에 대하여 다음이 성립한다.
>$$ R(ri\\_1)= H(hi\\_1)+ExtendL(hi\\_1)=H(hi\\_1)+L$$
3. 임의의 자연수$k$ 에 대하여 $n=m+k$ 일때 $L$의 모양은 $(h_1,h_2,...h_k,l_1,l_2,...,l_m)$ 으로 확장된다. 따라서 $ExtendL[0\_1][0\_2]\dots[0\_k]= \dots =ExtendL[h\_1-1][h\_2-1]\dots[h\_k-1]=L$ 이 된다. 따라서 $H$ 의 임의의 인덱스 튜플의 일부인 $(i_1,i_2,i_3,...,i_k) \in \prod_{i=1}^k H\_i$ 에 대해서 다음이 성립한다.
>$\\ \forall (i_1,i_2,i_3,...,i_k) \in \prod_{i=1}^k H_i:R(i_1,i_2,i_3,...,i_k)=H(i_1,i_2,i_3,...,i_k)+L$
4. 이 때, $n=m+k+1$ 에 대해서도 3의 과정을 적용하면 똑같이 성립한다.
5. 따라서 모든 $n\ge m$ 에 대해서 모두 성립한다. 

그렇다면 $m$보다 작거나 같은 자연수 $k$ 에 대해서 $l_k\not=h_{n-m+k}$ 를 만족하는 $k$ 가 1개 이상 존재한다고 해보자. 

1. 먼저 이를 만족하는 i개의 $k$ 에 대하여 내림차순으로 정렬한 결과를 $\{k_1,k_2,k_3,...,k_i\}$ 라 하자.
2. 이 때 $k_1$ 에 대해서 $k_1<j$ 를 만족하는 자연수 $j$ 에 대하여 $l_j=h_{n-m+j}$ 가 성립한다. 따라서 $l_{k_1}과 h_{n-m+k_1}$ 중 1인 쪽의 배열을 확장하면 된다. 이는 위에서 증명한 것을 토대로 다음을 만족한다. ($l\_{k\_1}=1$ 이라 하겠다.)
    - 둘의 shape에 대해서 $L_{sub_k}=\prod_{i=k+1}^m L_i$ ,  $H_{sub_k}=\prod_{i=n-m+k+1}^n H_i$ 라 하자. 
    - 이 때, $L_{sub_{k_1}} = H_{sub_{k_1}}$ 임은 자명하다.
    - $L_{sub_{k_1}+1}, H_{sub_{k_1}+1}$ 로 만든 배열을 $sub_{k_1}L, sub_{k_1}H$ 라 하자
    - $l_{k_1}=1$ 이므로 $sub_{k_1}L$ 은 확장되는데 `broadcast` 의 원리에 의해 $sub_{k_1}L[0]=sub_{k_1}L[1]=,,,=sub_{k_1}L[h_{n-m+{k_1}}-1]$ 이 성립되도록 확장되어 연산한다.
    - 따라서 $sub_{k_1}L, sub_{k_1}H$ 사이의 연산은 $sub_{k_1}H[i]\left(operator\right)sub_{k_1}L[0], i \in (sub_{k_1}H)_1$ 과 같이 처리할 수 있다.
3. 이는 모든 $k$ 에 대해서 똑같이 적용할 수 있다.

<h2 data-toc-skip>이 두 증명으로 인해 위의 알고리즘에는 모순이 없다. </h2>

### 그런데...

만약 python이였으면 
```python
R[0][0][0][0]...[0] = H[0][0][0][0][0]...[0](operator)L[0][0][0][0]...[0]
```
이런 식으로 각 차원에 해당하는 인덱스를 쓰면 됐다. 하지만 요구 사항은 C에서의 구현이였고 1차원배열로 해결해야한다고 했다...

모든 배열은 컴퓨터상에서 1차원배열의 형태니 불가능하진 않을 것 같다. 저 인덱스들을 이용하자.

### 어떻게?
먼저 C언어에서 3차원 배열 `int arr[4][3][2] `이 있다고 하자. 이는 numpy에서 `(4,3,2)`의 `shape`를 가진 array와 같다. 

1. `arr[0][0][0]` 와 `arr[0][0][1]`의 주소 차이는 얼마인가? 아는 int형 크기만큼의 차이이다. 이 차이를 1이라 하자. 
2. `arr[0][0][0]` 와 `arr[0][1][0]`의 주소 차이는 얼마일까? `arr[0][0][0]`의 주소에서 int형 크기만큼 주소를 늘려간다면 `arr[0][0][0]` $\rightarrow$ `arr[0][0][1]` $\rightarrow$ `arr[0][1][0]` 으로 변한다. 즉 2번째 인덱스 1 늘어나면 처음 주소와의 차이는 2라고 할 수 있다. 
3. `arr[0][0][0]` 와 `arr[1][0][0]`는 얼만큼 차이가 날까? `arr[0][0][0]`에서 2번째 인덱스가 3만큼 늘어나면 되므로 3\*2=6 만큼의 차이가 난다.

이를 토대로 `arr[0][0][0]`와 `arr[2][2][1]`과의 주소차이를 구해보자. 
1번째 인덱스가 2, 2번째 인덱스가 2, 1번째 인덱스가 1 늘어났으니 6\*2+2\*2+1\*1 =15 이다. 

이를 통해 n차원 배열에서 맨 앞의 인덱스(차원)를 1번째라 했을 때, k번째 차원의 크기를 $dim(k)$라 하자. 
(`int arr[4][3][2]` 에선 $dim(1)=4, dim(2)=3, dim(3)=2)$

k번째 인덱스가 1늘어날때마다 늘어나는 주소의 크기 $Jump(k)$ 은 다음과 같이 정의할 수 있다.

>$ Jump(n)=1 $
>
>$ Jump(k)=Jump(k+1)*dim(k+1) $

이를 이용해 $R,H,L$의 대해서 각각의 차원에 대해서 인덱스 값이 1 늘어날 때마다 늘어나는 주소를 모은 배열을 각각 $JumpArrayR,JumpArrayH, JumpArrayL$ 라 하자. 그렇다면 각각의 다차원 배열을 1차원 배열이라 생각해보자. 

>
>$R\left[\sum_{k=1}^n \left(R \\_ IndexStack[k]\times JumpArrayR[k]\right)\right]=H\left[\sum_{k=1}^n \left(H \\_ IndexStack[k]\times JumpArrayH[k]\right)\right]$
>
>$\left(operator\right)L\left[\sum_{k=1}^m \left(L \\_ IndexStack[k]\times JumpArrayL[k]\right)\right]$

로 표현할 수 있다.

- 만약 연산자의 좌측에 있는 배열의 차원이 더 낮다면 위의 식에서 피연산자의 순서만 바꾸면 된다.

참고자료: <https://numpy.org/doc/stable/user/basics.broadcasting.html>