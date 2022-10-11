---
layout: single
title: "Jekyll 블로그 빌드 중 Liquid Exception 해결방법"
categories: github_blog
toc: true
---



블로그 빌드 중 오류 발생으로 글이 publish가 안되었다.

```
// 에러코드
Liquid Exception: Liquid syntax error (line 37): Variable '{{1, 0}' was not properly terminated with regexp: /\}\}/ in /github/workspace/_posts/coding_test/programmers/2022-10-11-prog1844.md
```



지난 번에도 한번 해결했던 것 같은데, 역시 사람은 기록을 해야해.....

검색해보니 지킬에 나와 같은 문제에서 질문한 사람이 있었다.

```java
[수정 전]
/*
int[][] dir = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
*/
[수정 후]
int[][] dir = { {1, 0}, {0, 1}, {-1, 0}, {0, -1} };
```

{ { } } 이 중괄호 사이에 space가 있어야 되나보다.





참고링크: [https://github.com/jekyll/jekyll/issues/5458](https://github.com/jekyll/jekyll/issues/5458)

------

> 마지막 수정일시: 2022-10-11 13:43