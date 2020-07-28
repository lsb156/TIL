<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [Proxy](#proxy)
  - [영속성 전이 (Cascade)](#영속성-전이-cascade)
  - [고아객체](#고아객체)
  - [영속성 전이 + 고아객체, 생명주기](#영속성-전이--고아객체-생명주기)

## Proxy
entityMenager에 존재하는 getReference 함수가 호출될때는 실제 디비에 쿼리를 하지 않고
프록시 객체를 만들게 되고 프록시객체의 필드 정보가 getter에 의해서 참조가될때 쿼리가 실행되어 실제 데이터를 디비에서 읽어온다.

MemberProxy Class에 target 이름의 필드에 Member라는 멤버변수가 존재

프록시의 특징
- 프록시 객체는 사용할 때 한번만 초기화하며 영속성컨텍스트에 찾는 엔티티가 이미 있으면 실제 엔티티를 반환하게된다.
- 프록시 객체는 원본 엔티티를 상속받음으로 타입 체크시 주의해야함 ( `==` 아닌 `instanceOf` 사용)
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때 프록시를 초기화하면 Exception발생 (LazyInitializationException)

> em.find로 먼저 가져온뒤에 em.reference로 다시 가져오면 미리 가져왔던 Member Class를 반환해준다.
> 하지만 em.reference 한뒤에 em.find 혹은 em.reference를 하면 MemberProxy Class를 반환한다.
> JPA는 항상 같은 객체의 엔티티에 대해서는 == 비교에 true를 해줘야하기 때문이다.

각 프록시 관련 기능 확인
``` java
// 프록시 인스턴스 초기화 여부
PersistenceUnitUtil.isLoaded(Object entity);

// 프록시 클래스 확인 방법
entity.getClass().getName();

// 프록시 강제 초기화
org.hibernate.Hibernate.initialize(entity);
```


프록시와 즉시로딩 주의
- 가급적 실무에서는 지연로딩만 사용하여야한다. (실무에서는 즉시 로딩을 사용하지 말아야한다)
- 즉시로딩이 필요할경우에는 JPQL fetch Join이나 엔티티그래프 기능을 사용해야한다. (즉시로딩은 상상을 할 수 없는 쿼리가 나감)
- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
- `@ManyToOne`, `@OneToOne`  두개의 default가 FetchType.EAGER로 되어있어 LAZY로 선언해줘야한다.

### 영속성 전이 (Cascade)
특정 엔티티를 영속 상태로 만들 떄 연관된 엔티티도 함께 영속성 상태로 만들고 싶을때.
부모 엔티티를 저장할 때 자식 엔티티도 함께 저장됨.
``` java
@OneToMany(mappedBy="parent", cascade=CascadeType.PERSIST)
```

영속성 전이는 연관관계와 아무 관련이 없음
단순히 편의성만 제공해줄뿐

|종류|설명|
|-|-|
|**ALL**|모두 적용|
|**PERSIST**|영속|
|**REMOVE**|삭제|
|MERGE|병합|
|REFRESH|REFRESH|
|DETACH|DETACH|

### 고아객체
``` java
@OneToMany(mappedBy="parent", orphanRemoval=true)
```
orphanRemoval 옵션으로 부모객체의 컬렉션에서 빠지게 될 경우 자동으로 디비에서 삭제가 된다.
특정 엔티티가 개인이 소유할때만 사용하고 다른곳에서 참조를 한다면 절대 사용해선 안된다. (파일 첨부 같은곳에서 사용)
`CascadeType.REMOVE`	 처럼 부모 객체가 삭제가 되어도 자식들도 같이 삭제가된다.

### 영속성 전이 + 고아객체, 생명주기
```
@OneToMany(mappedBy="parent", cascade=CascadeType.ALL, orphanRemoval=true)
```
cascade, orphanRemoval 두개의 옵션을 위와같이 활성화하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있다.
도메인 주도설계 DDD의 Aggregate Root 개념을 구현할 때 유용하다.
