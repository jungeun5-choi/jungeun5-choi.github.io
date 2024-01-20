---
layout: single
title: "동시성 제어; 비관적 락을 통해 요청에 대한 데이터 정확성 확보"
categories: [MySQL]
tag: [db, concurrency control, pessimistic lock]
---

<div class="notice">
    <h4> 💡 참고: </h4>
    <ul>
        <li>
            <a href="https://medium.com/pocs/%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%A0%9C%EC%96%B4-%EA%B8%B0%EB%B2%95-%EC%9E%A0%EA%B8%88-locking-%EA%B8%B0%EB%B2%95-319bd0e6a68a"> 동시성 제어 기법 — 잠금(Locking) 기법 </a>
        </li>
        <li>
            <a href="https://velog.io/@p4rksh/RDB%EC%9D%98-%EB%8F%99%EC%8B%9C%EC%84%B1%EA%B3%BC-%EC%9D%BC%EA%B4%80%EC%84%B1-%EC%9D%B4%EC%8A%88%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B2%98%EB%A6%AC%ED%95%A0-%EA%B2%83%EC%9D%B8%EA%B0%80"> RDB의 동시성과 일관성 이슈를 어떻게 처리할 것인가? </a>
        </li>
        <li>
            <a href="https://incheol-jung.gitbook.io/docs/q-and-a/spring/feat.-tmi"> 동시성 해결하기(feat. TMI 주의) </a>
        </li>
    </ul>
</div>


## 동시성 제어(Concurrency Control)란?
다중 사용자 환경을 지원하는 데이터베이스 시스템에서는, 여러 트랜잭션[^1]들이 동시에 실행되기 마련이다. 이 때, 트랜잭션 간의 간섭으로 인한 문제가 발생하지 않도록, 트랜잭션의 실행 순서를 제어하는 것을 말한다.

[^1]: **트랜잭션:** Transaction; 데이터베이스의 상태를 변화시키기 해서 수행하는 작업의 단위.


## 문제 상황
여러 사용자가 동시에 구매 요청을 보냈을 때, **`구매 처리된 상품 개수`와 `남은 상품 개수` 간 데이터 불일치 문제가 발생**했다.

#### 테스트 조건: 사용자 1,000명이 100개의 상품에 대해 동시에 구매 요청
#####  1. 구매 처리된 상품 개수
![구매item수량]({{site.url}}/images/2023-09-18-pessimistic-lock/구매item수량.png){: width="45%" height="45%"}
```
expected: <100> but was: <936>
Expected : 100
Actual : 936
```
DB에 있던 전체 상품 수량은 `100`개였다. 하지만, 

#####  2. 남은 상품 개수
![남은item수량]({{site.url}}/images/2023-09-18-pessimistic-lock/남은item수량.png){: width="45%" height="45%"}
```
expected: <0> but was: <6>
Expected : 0
Actual : 6
```

<br>

테스트 결과, `구매 처리된 상품 개수`와 `현재 남은 상품 개수`의 데이터의 정확성이 확보가 되지 않고 있음을 알 수 있다. 이러한 문제를 해결하기 위해, 동시성 제어 기법을 도입하기로 결정했다.


## 비관적 락(Pessimistic Lock)이란?

~비관적 락 이야기~

### 비관적 락 도입의 이유
당시 도입하려던 동시성 제어 기법 후보에는 동기화(Synchronized), 비관적 락(Pessimistic Lock), 낙관적 락(Optimistic Lock), 네임드 락(Named Lock)이 있었다.

~동기화 이야기~
~네임드 락 이야기~
~낙관적 락 이야기~

세션과 서버가 여러 개로 나뉘었고, 이러한 환경으로 인해 Synchronized와 네임드 락을 도입하기 어려움이 있었다.

### 비관적 락 적용

###### ItemRepository.java
```java
import org.springframework.data.jpa.repository.Lock;
import jakarta.persistence.LockModeType;

@Repository
public interface ItemRepository extends JpaRepository<Item, Long>, ItemRepositoryCustom {
    @Lock(LockModeType.PESSIMISTIC_WRITE)           // ←
    @Query("select s from Item s where s.id=:id")   // ←
    Optional<Item> findByIdWithPessimisticLock(Long id);
```

###### ItemServiceImpl.java
```java
public Item findItemPessimisticLock(Long id) {
    return  itemRepository.findByIdWithPessimisticLock(id).orElseThrow(() ->
        new CustomException(CustomErrorCode.ITEM_NOT_FOUND, null)
    );
}
```

###### PurchaseServiceImpl.java
```java
@Override
@Transactional
public void purchaseItem(Long itemId, PurchaseRequestDto requestDto) {
    User user = userService.findUser(requestDto.getEmail());
    Item item = itemService.findItemPessimisticLock(itemId); // ←
    checkStock(item);
    Purchase purchase = Purchase.builder().item(item).user(user).build();
    purchaseRepository.save(purchase);
    item.sellOne();
}
```


#### 테스트 조건 1: 사용자 1,000명이 100개의 상품에 대해 동시에 구매 요청

#### 테스트 조건 2: 사용자 5,000명이 100개의 상품에 대해 동시에 구매 요청

## 한계
~~~에서는 ~~~한 한계가 있었다. <br>
(nGrinder 그래프)

### 해결 방안 (발전 가능성)

1. **대기열 생성:** <br>
    대기열을 생성하여 의도적으로 요청 처리 속도를 늦추는 방식이다. 현재도 단 시간에 사용자가 다량 몰리는 티켓팅 사이트 등에서 많이 사용하고 있다.

2. **인 메모리 데이터베이스 사용:** <br>
    Redis 등의 인 메모리 데이터베이스는 처리 속도가 디스크 보다 현저하게 빠르기 때문에, 지연 현상이 거의 일어나지 않을 것이다. 실제로 Redis 사용을 진작부터 고려하고 있었지만 일부러 사용하지 않았다(시간 문제도 있었지만). 왜냐하면, 이런 상황에서 Redis는 일종의 치트키다. 프로젝트를 본격적으로 진행하기에 앞서서, 다량의 요청을 처리하는 방법에 대해 학습했었는데, 학습한 것들을 실제로 적용해보고 싶었다. 그런 상황에서 Redis를 덜컥 도입하는 것은 이전 학습을 의미없게 만든다고 생각했다.



---