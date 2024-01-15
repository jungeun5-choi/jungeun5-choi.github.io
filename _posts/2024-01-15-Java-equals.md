---
layout: single
title: "Java equals() Overriding, hashCode()"
categories: [Java]
tag: [java, object, api, overriding]
toc: true
author_profile: false
---

## `equals()` 메소드를 Overriding할 때

> 참고: 자바의 신 VOL.1 기초 문법편, 12장.


`equals()` 메소드를 Overriding할 때는 반드시 다음 다섯 가지의 조건을 만족시켜야 한다.

1. **재귀(reflexive):** `null`이 아닌 `x`라는 객체의 `x.equals(x)` 결과는 항상 `true`여야 한다.

2. **대칭(transitive):** `null`이 아닌 `x`와 `y` 객체가 있을 때, `y.equals(x)`가 `true`를 리턴했다면, `x.equals(y)`도 반드시 `true`를 리턴해야만 한다.
   
3. **타동적(transitive):** `null`이 아닌 `x`, `y`, `z`가 있을 대, `x.equals(y)`가 `true`를 리턴하고, `y.equals(z)`가 `true`를 리턴하면, `x.equals(z)`는 반드시 `true`를 리턴해야만 한다.

4. **일관(consistent):** `null`이 아닌 `x`와 `y`가 있을 때, 객체가 변경되지 않은 상황에서는 몇 번을 호출하더라도, `x.equals(y)`의 결과는 항상 `true`이거나, 항상 `false`여야만 한다.

5. **`null`과의 비교:** `null`이 아닌 `x`라는 객체의 `x.equals(null)` 결과는 항상 `false`여야만 한다.


### `equals()` Overriding 예제

```java
public class MemberDto {
    public boolean equals(Object obj) {
        if (this == obj) return true;       // x.equals(x)는 항상 true
        if (obj == null) return false;      // null이 아닌 x에 대해 x.equals(null)는 항상 false
        // 클래스의 종류가 다르므로 false
        // null이 아닌 x, y에 대해 x.equals(y)는 항상 ture이거나 false
        if(getClass() != obj.getClass()) return false;
        
        MemberDto other = (MemberDto) obj;  // 같은 클래스이므로 형 변환 실행

        // name이 null일 때,
        if (name == null) {            
            if (other.name != null) return false;   // 비교 대상의 name이 null이 아니면 false
                                                    // null이 아닌 x에 대해 x.equals(null)는 항상 false
        } else if (!name.equals(other.name)) return false;  // 두 개의 name 값이 다르면 false
                                                            // null이 아닌 x, y에 대해 x.equals(y)는 항상 ture이거나 false

        // 위의 name과 같은 비교 수행
        if (email == null) {
            if (other.email != null) return false;
        } else if (!email.equals(other.email)) return false;

        if (phone == null) {
            if (other.phone != null) return false;
        } else if (!phone.equals(other.phone)) return false;

        // 위의 false 조건에 부합하지 않은 객체는 true로 간주한다.
        return true;
    }
}

```
   

<br>

### `hashCode()`도 함께 Overriding해야 한다.
`equals()` 메소드를 Overriding 할 때는 `hashCode()` 메소드도 같이 Overriding해야만 한다. 왜냐하면, `equals()` 메소드를 Overriding해서 객체가 서로 같다고 이야기할 수는 있겠지만, 그 두 객체의 주소 값이 같다고는 볼 수 없기 때문이다.

즉, `equals()` 메소드만 Overriding한다면, `equals()` 메소드의 결과가 `true`이더라도 두 객체에 대한`hashCode()`를 호출한 값은 동일하지 않을 수도 있다. (물론 `equals()` 메소드를 반드시 Overriding해야 하는 것은 아니다. DTO를 만들 경우에는 객체 비교를 위해 필요해진다.)

<br>

### `hashCode()` 메소드를 Overriding할 때
1. Java 애플리케이션이 수행되는 동안, 어떤 객체에 대해 이 메소드가 호출될 때에는 항상 동일한 `int` 값을 리턴해 주어야 한다. 하지만, Java 애플리케이션 매 실행 시마다 같은 값이어야 할 필요는 전혀 없다.

2. 어떤 두 개의 객체에 대해 `equals()` 메소드를 사용하여 비교한 결과가 `true`일 경우, 두 객체의 `hashCode()` 메소드를 호출하면 동일한 `int` 값을 리턴해야만 한다.

3. 두 객체를 `equals()` 메소드를 사용하여 비교한 결과 `false`를 리턴했다고 해서, `hashCode()` 메소드를 호출한 `int` 값이 무조건 달라야 할 필요는 없다. 하지만, 이 경우에 서로 다른 `int` 값을 제공하면, 해시테이블(hashtable)의 성능을 향상시키는 데 도움이 된다.

<br>

(이러한 제약 때문에, 직접 `equals()` 메소드나 `hashCode()` 메소드를 작성하는 것은 권장하지 않는다. 요즘에 나오는 개발 툴에서는 두 개의 메소드를 자동으로 생성해주는 기능이 있으므로, 그 기능을 활용할 것을 권장한다.)
