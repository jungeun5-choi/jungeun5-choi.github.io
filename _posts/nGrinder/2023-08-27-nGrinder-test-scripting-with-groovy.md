---
layout: single
title: "nGrinder; groovy 테스트 스크립트 작성 (POST 요청)"
categories: [nGrinder]
tag: [ngrinder, stress test, groovy]
---

## 👩‍💻 테스트 스크립트 작성
<div class="notice">
    작성했던 스크립트 전문은 <a href="https://github.com/Tour-Ranger/Tour-Ranger-Test-Runner"><b>여기</b></a>에서 확인 가능
</div>

### 💁‍♀️ 기본 @annotation 정보

스타일은 **JUnit**, 언어는 **Groovy**를 사용

- `@BeforeProcess`
    - Java의 `@BeforeClass`와 거의 동일하다
    - 프로세스 당 한 번의 테스트 셋업을 실행(설정)
    - `static` 메서드이다 - 클래스 단에서 실행되기 때문에
- `@BeforeThread`
    - 해당 객체가 만들어진 후에 쓰레드가 실행될 때 이 어노테이션이 붙은 메서드를 실행
- `@Test` → 반복 실행되는 부분

### 💣 HTTP 요청 연결 시간 제한 vs 연결 시간 초과

```groovy
@BeforeProcess
public static void beforeProcess() {
    HTTPRequestControl.setConnectionTimeout(300000)
    // ...
```

기본 스크립트에는 **위**⬆️와 같은 명령어가 써져있는데, 이에 대해서 찾아보면 **아래**⬇️와 같은 명령어도 함께 등장한다.

```groovy
@BeforeProcess
public static void beforeProcess() {
    HTTPPluginControl.getConnectionDefaults().timeout = 6000
    // ...
```

그래서 둘의 차이점을 찾아보았다.

#### 1. HTTPRequestControl.setConnectionTimeout(${connectionTimeout})**

|분류|내용|
|--:|:--|
|**목적:**|HTTP 요청에 대한 **connection timeout**을 설정|
|**범위:**|단일 HTTP 요청에만 적용|
|**입력 시간 단위:**|ms(밀리초)|
|**유연성:**|요청마다 제한이 다를 수 있음|
|**설정값 재정의:**|해당 없음|
|**사용 사례:**|다양한 요청에 다양한 시간 제한 요구 사항이 있는 경우|

###### 예제)
```groovy
    // 신속하게 응답해야 하는 endpoint 요청
    HTTPRequestControl.setConnectionTimeout(10000) // 10000 ms = 10 sec
    get("https://example.com/api/fast")
    
    // 응답하는데 더 오래 걸릴 수 있는 endpoint 요청
    HTTPRequestControl.setConnectionTimeout(60000) // 60000 ms = 60 sec
    get("https://example.com/api/slow")
```

#### 2. HTTPPluginControl.getConnectionDefaults().timeout = ${timeout}

|분류|내용|
|--:|:--|
|**목적:**|HTTP 요청에 대한 **connection timeout**을 설정|
|**범위:**|모든 HTTP 요청에 적용|
|**입력 시간 단위:**|ms(밀리초)|
|**유연성:**|모든 요청에 대해 동일한 시간 제한을 설정|
|**설정값 재정의:**|필요한 경우, 특정 요청에 대해 재정의가 될 수 있음|
|**사용 사례:**|일관된 시간 제한 설정이 필요한 경우|

###### 예제)        
```groovy
    HTTPPluginControl.getConnectionDefaults().timeout = 6000 // 6000 ms = 6 sec

    // 이후의 모든 요청은 기본 시간 제한인 6초를 사용한다.
    get("https://example.com/api/endpoint1")
    post("https://example.com/api/endpoint2")
```
        

### 📬 `POST` 요청 테스트 스크립트

<div class="notice">
    상품 주문 API (PurchaseItem): <a href="https://github.com/Tour-Ranger/Tour-Ranger-Test-Runner/blob/d027732b28b767b81eb51ad235ee81f1d2fff017/src/test/groovy/purchase/PurchaseItem.groovy">groovy/purchase/PurchaseItem.groovy</a>
</div>

**`GET` 요청과는 다른 점**

1. `POST` 요청의 경우, `@RequestBody`로 입력 정보를 받아오기 때문에, POST 요청과 함께 `body` 값도 전달해주어야 한다.
2. `application/json` 타입의 값을 변수로 지정해준다
    
    ```groovy
    class PurchaseItem {
        // ...
        public static String **body** = "{\n\"email\":\"email1@email.com\"}"
    ```
    
3. **`POST` 요청 시, `body`도 함께 전달해줘야한다** - `body.getBytes()` 형태로…
    
    ```groovy
    @Test
    public void test() {
        HTTPResponse response = request.POST("http://${NGRINDER_HOSTNAME}:1010/tour-ranger/purchases/1", body.getBytes())
    ```
    - 민감한 정보는 환경 변수로 처리해두는게 좋다.

4. POST 요청 결과 (성공)
    
    ![로컬DB에 POST요청]({{site.url}}/images/2023-08-27-nGrinder-test-scripting-with-groovy/로컬DB에-POST요청.png){: width="70%" height="70%"}
    
    
    
#### 참고) 없는 데이터를 건드리는 경우
아래 같은 에러가 발생한다
```
2023-08-27 01:07:11,509 ERROR 
Expected: is <200>
     got: <400>

java.lang.AssertionError: 
Expected: is <200>
     got: <400>

	at org.hamcrest.MatcherAssert.assertThat(MatcherAssert.java:21)
	at GetItem.test(GetItem.groovy:66)
	at jdk.internal.reflect.GeneratedMethodAccessor3.invoke(Unknown Source)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at net.grinder.scriptengine.groovy.junit.GrinderRunner.run(GrinderRunner.java:164)
	at net.grinder.scriptengine.groovy.GroovyScriptEngine$GroovyWorkerRunnable.run(GroovyScriptEngine.java:147)
	at net.grinder.engine.process.GrinderThread.run(GrinderThread.java:118)
2023-08-27 01:07:11,511 INFO  before. init headers
2023-08-27 01:07:11,516 INFO  http://{hostname}:1010/tour-ranger/items/1 -> 400 , 73 bytes
2023-08-27 01:07:11,516 ERROR
```