---
title: "순열"
layout: single
categoies: math&algorithm
---

# 순열











# 순열

## 1) 팩토리얼 (Factorial)

$$
n! = n(n-1)(n-2)...1(n은 자연수)
$$

```java
for(int i = n; i >= 1; i--) {
	result *= i;
}
```



## 2) 순열 (Permutation)

서로 다른 n개 중 r 개를 선택하여 나열하는 경우의 수 (순서O, 중복X)
$$
nPr = n! / (n-r)! = n(n-1)(n-2)...(n-r+1)
(단, 0<r<=n)
$$

```java
for(int i = n; i >= n - r + 1; i--) {
    result *= i;
}
```



## 2-1) 중복순열

서로 다른 n개 중 r 개를 선택하는 경우의 수

순열과 같지만, 중복이 있는 경우
$$
n^r
$$

```java
for(int i = 1; i <= r; i++) {
	result *= n;
}
// Math 함수 활용
result = Math.pow(n, r);
```



## 2-2) 원 순열

원으로 나열하는 경우의 수
$$
n! / n = (n-1)!
$$

```java
for(int i = n-1; i>=1; i--) {
    result *= i;
}
```

---

마지막 수정일시: 2022-05-21 23:45

