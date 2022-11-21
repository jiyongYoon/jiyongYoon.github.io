---
title: "JPA Repository 기본 메서드"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true

---

# 1. 조회

### - findById()

- `T findById()` : T 반환. 없으면 null 반환
- `Optional<T> findById()` : Optional 반환. 없으면 Optional empty 반환

### 

# 2. 삭제

### - delete()

- `void delete(T)` : 엔티티를 삭제함.
- `void deleteById()` : 내부적으로 findById()를 한 후 엔티티를 조회하여 deletd()로 삭제함.
- *주의* - 삭제할 대상이 존재하지 않는 경우 익셉션 발생



# 3. 저장

### - save()

- `void save(T)` 
- `T save(T)` 

### save() 호출 시 실행 쿼리

- select 쿼리 후 insert 쿼리

  -> 왜? `**Repository** `구조 때문.

```java
public <S extends T> S save(S entity) {
	Assert.notNull(entity, "Entity must not be null.");
	if (this.entityInformation.isNew(entity)) {
		this.em.persist(entity);
		return entity;
	} else {
		return this.em.merge(entity);
	}
}
```

`isNew` - 즉, 새 엔티티인지 아닌지! 확인하기 위해서 `select`쿼리가 한번 나감.

**그러면 뭐가 새 엔티티인가?**

- Persistable을 구현한 엔티티 (JPA 인터페이스 종류)
- `@Version` 속성이 있는 경우, 버전 값이 null이면 새 엔티티로 판단
- 식별자가 참조 타입이면(Integer, String 등등) null이면 새 엔티티로 판단
- 식별자가 숫자 타입이면(int, long, double 등) 0이면 새 엔티티로 판단



# 4. 특정 조건으로 찾기

### - findBy프로퍼티(값)

- 프로퍼티가 특정 값인 대상
  - ex) List<User> <u>findBy</u>Name(String name)
  - ex) List<User> <u>findBy</u>Name<u>And</u>Grade(String name, Grade g)
- 프로퍼티가 조건 비교인 경우 (Like, After, Between, LessThan, IsNull 등...)
  - ex) List<User> <u>findBy</u>Name<u>Like</u>(String keyword)
- 모두 조회
  - ex) findAll()

# 5. 기타

- findBy 메서드는 단순한 경우 사용할 것을 추천.
- 검색 조건이 단순하지 않으면 `@Query`, `SQL`, `QueryDSL`등을 사용할 것을 추천



---

- 참고영상: 유튜브 채널 최범균 - [Spring Data JPA 02 리포지토리 메서드 작성 규칙](https://youtu.be/qTiHaxVc6GY)

---

> 마지막 수정일시: 2022-11-21 22:30