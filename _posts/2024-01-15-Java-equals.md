---
layout: single
title: "Java equals() 메소드를 Overriding할 때"
categories: [Java]
tag: [java, object, api, overriding]
toc: true
author_profile: false
---

## `equals()` 메소드를 Overriding할 때
`equals()` 메소드를 Overriding할 때는 반드시 다음 다섯 가지의 조건을 만족시켜야 한다.

1. **재귀(reflexive):** `null`이 아닌 `x`라는 객체의 `x.equals(x)` 결과는 항상 `true`여야 한다.
2. **대칭(transitive):** `null`이 아닌 `x`와 `y` 객체가 있을 때, `y.equals(x)`가 `true`를 리턴했다면, `x.equals(y)`도 반드시 `true`를 리턴해야만 한다.
3. **타동적(transitive):** `null`이 아닌 `x`, `y`, `z`가 있을 대, `x.equals(y)`가 `true`를 리턴하고, `y.equals(z)`가 `true`를 리턴하면, `x.equals(z)`는 반드시 `true`를 리턴해야만 한다.
4. **일관(consistent):** `null`이 아닌 `x`와 `y`가 있을 때, 객체가 변경되지 않은 상황에서는 몇 번을 호출하더라도, `x.equals(y)`의 결과는 항상 `true`이거나, 항상 `false`여야만 한다.
5. **`null`과의 비교:** `null`이 아닌 `x`라는 객체의 `x.equals(null)` 결과는 항상 `false`여야만 한다.
   

<br><br>

> 자바의 신 VOL.1 기초 문법편, 12장.