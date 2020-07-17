
<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [@Entity](#entity)
- [@Table](#table)
- [@Id](#id)
- [@Column](#column)
- [데이터베이스 스키마 자동 생성](#데이터베이스-스키마-자동-생성)
- [기타 맵핑 어노테이션 정리](#기타-맵핑-어노테이션-정리)

### @Entity
JPA에서 관리하는 클래스를 지정하는 어노테이션
JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity` 필수

주의
- 기본 생성자 필수 (파라메터가 없는 public, protected 생성자)
- 다음 클래스들은 생성이 안된다 : final Class, enum, interface, inner Class
- 저장할 필드에 final을 사용할 수 없다

### @Table
|속성|기능|기본값|
|:-|:-|:-|
|name|매핑할 테이블 이름|엔티티 이름을 사용|
|catalog|데이터베이스 catalog 매핑||
|schema|데이터베이스 schema 매핑||
|uniqueConstaints(DDL)|DDL 생성 시에 유니크 제약 조건 생성||

### @Id
기본 키 매핑 방법
- 직접 할당: `@Id` 사용
- 자동생성(`@GeneratedValue`)
	- IDENTITY : 데이터베이스에 위임, MYSQL
	- SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용, ORACLE
		- `@SequenceGenerator` 필요
	- TABLE : 키 생성용 테이블 사용, 모든 DB에서 사용
		- `@TableGenerator` 필요
	- AUTO : 방언에 따라 자동 지정, Default

> JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL을 실행한다.
> 하지만 IDENTITY 전략은 데이터베이스에 INSERT SQL을 실행한 이후에 ID값을 알 수 있는 구조로 되어있어 `em.persist()` 시점에 즉시 INSERT_SQL를 실행한뒤 1차캐시에 저장해놓는다.
> 그래서 IDENTITY 전략에서는 버퍼에 쌓아놓고 한번에 보내는 방식은 불가능하다.
> (비약적인 성능상 향상이 있는게 아니라 특별한 케이스가 아니라면 고려 안해도 될듯)


> 식별자 키를 따로 만들 경우에는 LONG + 대체키 + 키 생성전략을 사용
> AUTO_INCREMENT, Sequence Object, UUID을 권장
> 비지니스 값들을 유니크 키로 대체하면 큰 사고가 날 수 있음
> (추후 정책 변경에 대한 비용 증가)


### @Column
|속성|설명|기본값|
|:-|:-|:-|
|name|필드와 매핑할 테이블의 컬럼 이름|객체의 필드명|
|insertable, updatable|등록, 변경 가능 여부|TRUE|
|nullable (DDL)|null 값의 허용여부를 설정한다. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다||
|unique (DDL)|`@Table`의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 떄 사용한다.||
|columnDefinition (DDL)|데이터베이스 컬럼 정보를 직접 줄 수 있다 <br/> ex) varchar(100) default 'EMPTY'|필드의 자바 타입과 방언 정보를 사용|
|length (DDL)|문자 길이 제약 조건, String 타입에만 사용|255|
|precision, scale (DDL)|BigDecimal 타입에서 사용한다 (BigInteger에서도 사용) precision은 소수점을 포함한 전체 자릿수를, scale은 소수의 자릿수다. 참고로 double, float 타입에는 적용되지 않는다. 정밀한 소수를 다루어야 할 떄만 사용한다.|precisition=19 <br/> scale=|

### 데이터베이스 스키마 자동 생성
- DDL을 어플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언(Direct)을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 생성된 DDL은 `개발 장비에서만 사용`
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후에 사용된다.

hibernate.hbm2ddl.auto
|옵션|설명|사용용도}
|-|-|-|
|create|기존 테이블 삭제 후 다시 생성 (DROP -> CREATE)|개발초기|
|create-drop|create와 같으나 프로그램 종료 시점에 테이블 DROP|개발초기|
|update|변경분만 반영 (운영 DB에는 사용하면 안됨)|개발초기, 테스트서버|
|validate|엔티티와 테이블이 정상 매핑되었는지만 확인|테스트서버, 스테이징, 운영|
|none|사용하지 않음|스테이징, 운영|

> 운영 장비에서는 절대 create, create-dtop, update 사용하면 안됨 (DB 계정분리를 이용하여 못하도록 막아야한다.)
> update는 alter 컬럼이 나가면서 데이터베이스가 lock 상태가 되면서 서비스가 중단된다.
>개발초기 - create, update
>테스트 서버 - update, validate
>스테이징, 운영 - validate, none

### 기타 맵핑 어노테이션 정리

|어노테이션|설명|
|-|-|
|@Column|커럼 맵핑|
|@Temproral|날자 타입 매핑 (DATE, TIME, DATETIME), LocalDate나 LocalDateTime에서는 생략가능 (최신하이버네이트 지원))|
|@Enumerated|enum타입 매핑|
|@Lob|BLOB, CLOB|
|@Transient|특정 필드를 컬럼에서 제외|

