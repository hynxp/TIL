# 변경감지와 병합

JPA에서 데이터를 변경할때의 기본 메커니즘은 변경 감지(Dirty Checking)이다.

## 💡 더티 체킹(Dirty Checking) a.k.a 변경 감지란?

```java
@Transactional
public void updateItem(Long itemId, String name, int price, int stockQuantity) {
    Item findItem = itemRepository.findOne(itemId);
    findItem.setName(name);
    findItem.setPrice(price);
    findItem.setStockQuantity(stockQuantity);
}
```

위는 pk인 itemId를 조건으로 Item을 조회하고, 조회한 Item 데이터를 수정하는 코드다.

코드만 보면 별도로 데이터베이스에 `UPDATE`해주지 않는다.

늘 하던대로라면 `itemRepository.update(findItem);` 가 존재해야한다.

## 💡 어떻게 가능할까?

JPA에서는 엔티티를 조회하면 해당 엔티티의 조회 상태 그대로 스냅샷을 만들어 놓는다.

그리고 트랜잭션이 끝나는 시점에 이 스냅샷과 비교해서 다른점이 있다면 변경을 감지해 Update Query를 데이터베이스로 전달한다.

이 변경 감지의 대상이 되지 않는 엔티티가 있다.

- 준영속 엔티티 (detach된 엔티티)
    - `em.detach(item);`
- 비영속 엔티티

준영속 엔티티란 영속성 컨텍스트가 더는 관리하지 않는 엔티티를 말한다.

```java
Member member = createMember("userA", "서울","1","11111");
em.persist(member);
```

객체를 처음 생성하고 persist() 하는 순간 영속상태가 된다.

```java
@Transactional
public void updateItem(Long itemId, String name, int price, int stockQuantity) {
    Item findItem = itemRepository.findOne(itemId);
    findItem.setName(name);
    findItem.setPrice(price);
    findItem.setStockQuantity(stockQuantity);
}
```

위 코드처럼 이미 itemId가 세팅이 되어있다는 건 이미 DB에 저장되어있다는걸 의미한다.

이 식별자(itemId)가 DB에 이미 존재하면 준영속 엔티티다.

## 💡 준영속 엔티티를 수정하는 2가지 방법

1. 변경 감지 기능 사용
2. 병합(merge) 사용

### 1. **변경 감지**

```java
@Transactional
void update(Item itemParam) { //itemParam : 준영속 상태의 엔티티
	 Item findItem = em.find(Item.class, itemParam.getId()); //조회
	 findItem.setPrice(itemParam.getPrice());

}
```

트랜잭션 안에서 엔티티를 다시 조회하면 영속 상태로 변경한다.

후에 데이터를 수정하면 변경 감지가 동작하기때문에 자동으로 UPDATE가 된다.

### 2. **병합(merge) 사용**

```java
@Transactional
void update(Item itemParam) { //itemParam: 준영속 상태의 엔티티
	 Item mergeItem = em.merge(item);
}
```

merge()를 수행하여 준영속 상태의 엔티티를 영속 상태로 변경한다.

![동작 방식](https://github.com/kyunghyun-Park/TIL/assets/50633008/cb3544ac-ab84-47bc-bed2-726f65ce4e0b)

**동작 방식**

1. merge()를 실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
    1. 2-1. 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고, 1차 캐시에 저장한다.
3. 조회한 영속 엔티티( mergeMember )에 member 엔티티의 값을 채워 넣는다. (member 엔티티의 모든 값 을 mergeMember에 밀어 넣는다. 이때 mergeMember의 “회원1”이라는 이름이 “회원명변경”으로 바뀐다.)
4. 영속 상태인 mergeMember를 반환한다.

간단하게 말하면 영속 엔티티(mergeItem)의 값을 파라미터로 들어온 준영속 엔티티(itemParam)값으로 모두 교체한다.

**병합시 조심해야할 점**

병합을 사용하면 엔티티의 모든 필드를 다 바꿔버리기 때문에 준영속 상태의 엔티티에 값을 세팅해주지 않은 필드는 null로 바뀔 수도 있다.

**가급적이면 merge를 쓰지말자.**

**엔티티를 변경할 때는 항상 변경 감지를 사용하자.**
