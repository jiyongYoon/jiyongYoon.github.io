---
title: "코딩테스트 준비"
layout: single
categories: coding_test
typora-root-url: ..\..\images\
toc: true
tag: [coding_test]
---

이번 본문은 유튜브 영상을 보며 필요한 내용을 정리한 글입니다.

자료출처: [개발자 장고](https://youtu.be/pvufY7rK7VA)

------

## 1. 코딩 테스트 필수 알고리즘

- BFS
- DFS
- 백트래킹
- 시뮬레이션
- 투포인터
- 이진탐색
- Greedy
- DP
- MST
- 다익스트라
- 플로이드

## 2. 코딩 테스트 푸는 방법 **중요*

### 1) 풀기전에 최대한 구체적으로 계획을 세우기

### 2) 다음 세가지를 주석으로 써보고 문제 풀기

#### 1 - 아이디어

: 문제를 어떻게 풀 것인지 설계하고 진행.

#### 2 - 시간복잡도

: 빅 O를 통해 시간초과가 나지 않는지 확인. 보통 컴퓨터는 1초당 2억개의 연산정도를 한다고 하니, 연산횟수가 2억개를 넘지 않는지(1초라면) 확인할 것.

// 혹자는 1억으로 간주하고 준비하라고 하기도 한다.

- O(N) 기준: 2억 (or 1억)
- O(N^2) 기준: 1만

#### 3 - 자료구조

: 숫자의 경우 최대자리에 따라 데이터 타입 예상.(int는 20억까지 소화 가능함)



## 3. 연습 예제

[백준 2559 수열](https://www.acmicpc.net/problem/2559)

```
1. 아이디어
for문으로 인덱스를 돌면서 각 위치에서 K개의 숫자를 더하자.
그리고 그 때마다 최대값을 갱신하자.

2. 시간복잡도
- N의 개수의 범위: 2 ~ 100,000
- N의 숫자의 범위: -100 ~ 100
- K의 숫자의 범위: 1 ~ N
for문: O(N)
1) 각 위치에서 K개를 더하자: O(K)
2) 전체 시간복잡도: O(KN) 
   => 100,000 X 100,000 = e^5 * e^5 = e^10 > 2억 초과
   => 시간초과 예상
   => for문을 다 돌면 안되겠구나

---- 아이디어 다시 ----

1. 새로운 아이디어
처음에 0번 인덱스부터 K개의 값을 더하자.
그리고 for문으로 인덱스를 하나씩 뒤로 밀어가며 추가된 값은 더하고 빠지는 값은 빼자.
그리고 그 때마다 최대값을 갱신하자.

2. 시간복잡도
- N의 개수의 범위: 2 ~ 100,000
- N의 숫자의 범위: -100 ~ 100
- K의 숫자의 범위: 1 ~ N
1) for문: O(N)
2) 인덱스 밀어가며 추가, 제거 작업: 각 작업당 X2
3) 전체 시간복잡도: O(2N) == O(N)
   => 가능하겠구나

3. 자료구조
- N의 개수의 범위: 2 ~ 100,000
- N의 숫자의 범위: -100 ~ 100
- K의 숫자의 범위: 1 ~ N
1) 전체 수 배열: int배열 가능 
   => -100 ~ 100을 커버하면 된다
2) K개를 더했을때 최대 값: int 가능 
   => 최대 수인 100을 최대 개수인 100,000개만큼 더한다면 
      e^2 * e^5 = e^7 < 20억 
   => int 가능
   
---- 코딩 시작 ----
```

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Practice087 {
    static int n, k;
    static int[] arr;
    public static void solution() {
        int left = 0;
        int right = k - 1;
        int maxTemp = Integer.MIN_VALUE;
        int sum = 0;
        for (int i = 0; i < k; i++) {
            sum += arr[i];
        }
        maxTemp = Math.max(maxTemp, sum);

        while(right < arr.length - 1) {
            sum += arr[++right];
            sum -= arr[left++];
            maxTemp = Math.max(maxTemp, sum);
        }
        System.out.println(maxTemp);
    }


    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String[] s = br.readLine().split(" ");
        n = Integer.parseInt(s[0]);
        k = Integer.parseInt(s[1]);
        arr = new int[n];
        String[] s2 = br.readLine().split(" ");
        for (int i = 0; i < arr.length; i++) {
            arr[i] = Integer.parseInt(s2[i]);
        }
        solution();
    }
}
```



------

> 마지막 수정일시: 2022-06-29 13:14