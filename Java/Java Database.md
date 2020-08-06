
# Java Database
여러 DB에서 공통적으로 사용할 수 있도록 자바에서는 JDBC라는 공통된 스펙을 제공

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [SQL DDL, DML, DCL?](#sql-ddl-dml-dcl)
- [ORM](#orm)
  - [JPA](#jpa)
    - [JpaRepository Class Diagram](#jparepository-class-diagram)
    - [데이터베이스와 객체 맵핑](#데이터베이스와-객체-맵핑)
      - [Entity](#entity)
      - [N:1](#n1)
      - [1:N](#1n)
  - [QueryDSL](#querydsl)
- [MyBatis](#mybatis)
- [Connection pool](#connection-pool)

## SQL DDL, DML, DCL?
- **데이터 정의 언어** - ( **DDL** : Data Definition Language )
	테이블이나 관계의 구조를 생성
	- CREATE - 새로운 데이터베이스 관계 (테이블) View, 인덱스 , 저장 프로시저 만들기.
	- DROP - 이미 존재하는 데이터베이스 관계 ( 테이블 ) , 뷰 , 인덱스 , 저장 프로시저를 삭제한다.
	- ALTER - 이미 존재하는 데이터베이스 개체에 대한 변경 , RENAME의 역할을 한다.
	- TRUNCATE - 관계 ( 테이블 )에서 데이터를 제거한다. ( 한번 삭제시 돌이킬 수 없음.)
- **데이터 조작 언어** - ( **DML** : Data Manipulation Language )
	테이블에 데이터 검색, 삽입, 수정, 삭제하는 데 사용
	- SELECT - 검색(질의)
	- INSERT - 삽입(등록)
	- UPDATE - 업데이트(수정)
	- DELETE - 삭제
- **데이터 제어 언어** - ( **DCL** : Data Control Language)
	 데이터의 사용 권한을 관리하는 데 사용하며 GRANT, REVOKE 문 등이 있다.
	- GRANT - 특정 데이터베이스 사용자에게 특정 작업에 대한 수행 권한을 부여한다.
	- REVOKE - 특정 데이터베이스 사용자에게 특정 작업에 대한 수행 권한을 박탈 or 회수 한다.
- 데이터베이스 사용자에게 GRANT 및 REVOKE로 설정 할 수 있는 권한.
	- SELECT - 데이터베이스에서 데이터를 검색할 수 있는 권한
	- INSERT - 데이터베이스에서 데이터를 등록(삽입)할 수 있는 권한
	- UPDATE - 데이터베이스의 데이터를 업데이트 할 수 있는 권한
	- DELETE - 데이터베이스의 데이터를 삭제할 수 있는 권한.
	- USAGE - 스키마 또는 함수와 같은 데이터베이스 개체를 사용할 수 있는 권한

> **DELETE, TRUNCATE, DROP의 차이**
> -  DELETE : 데이터는 지워지지만 테이블 용량은 줄어 들지 않는다. 원하는 데이터만 지울 수 있다. 잘못 삭제 한 경우 삭제한것을 되돌릴 수 있다.
> - TRUNCATE : 삭제후 용량이 줄어들고 인덱스 등도 모두 삭제된다. 테이블이 삭제 되지는 않으나 데이터만 삭제한다. 선택해서 지울 수 없다. 삭제 후 절대 되돌릴 수 없다
> - DROP : 테이블 전체를 삭제, 공간, 객체를 삭제한다, 삭제 후 절대로 되돌릴 수 없다.
## ORM
Object와 Relation 간의 불일치 문제를 해결하기 위한 도구
코드의 반복을 줄이고 객체 중심의 설계에 집중해서 개발을 진행할 수 있다.
### JPA
자바 진영에서 자주 쓰이는 ORM은 JPA와 Hibernate가 있다.
JPA를 만들당시 NOSQL이 나오지 않아서 RDBMS에만 한정적으로 사용이 가능
#### JpaRepository Class Diagram


```graphBT
JpaRepository--extends-->PagingAndSortingRepository
PagingAndSortingRepository--extends-->CrudRepository
CrudRepository--extends-->Repository
```
|Class                     |Methods|
|--------------------------|-|
|Repository                ||
|CrudRepository            |save<br>saveAll<br>findById<br>existsById<br>findAll<br>findAllById<br>delete<br>deleteAll|
|PagingAndSortingRepository|findAll(Sort var1)<br>findAll(Pageable var1)|
|JpaRepository             |flush()<br>saveAndFlush()<br>deleteInBatch()<br>getOne()|
#### 데이터베이스와 객체 맵핑
##### Entity
>자바 제너릭에서 Type 표기 약어의 의미
- E - Element (요소를 의미)
- K - Key (키를 의미)
- N - Number (숫자를 의미)
- T - Type (타입을 의미)
- V - Value (값을 의미)

##### N:1
Association 관계는 하나 이상의 객체가 연결되어 있는 상태를 나타낸다.
이떄 연결은 하나의 객체가 다른 객체를 소유하거나 참조하는 상태가 된다.

```java
Entity
public class School {
	@id @Column("SCHOOL_ID")
	private Long id;
	private String name;
	private String address;
	private String telnumber;
}

@Entity
public class Student {
	@Id @Column("STUDENT_ID")
	private Long id;
	private String userName;
	private String grade;
	@ManyToOne(fetch = FetchType.LAZY)
	@JoinCoulmn(name = "SCHOOL_ID")
	private School school
}
```
> `FetchType.LAZY` 로딩을 사용하면 JPA는 한 번에 모든 데이터를 다 로딩하지 않고 proxy를 이용해서 처리한다.
두 개 이상의 Entity들을 제어하는 메서드들을 사용할 때는 해당 메서드들을 `SchoolService`의 `findStudentInfo` 메서드와 같이 `@Transactional` 어노테이션을 사용해서 하나의 트랜잭션 안에서 실행되도록 해야 한다.

##### 1:N
```java
@Entity
public class School {
	@id @GeneratedValue
	@Column("SCHOOL_ID")
	private Long id;
	private String name;
	private String address;
	private String telnumber;
	
	@OneToMany(mappedBy = "school")
	private Set<Student> students;
	
	public void registerStudent(Student s){
		if (students == null) {
			students = new HashMap<Student>();
		}
		students.add(s);
	}
}

@Entity
public class Student {
	@Id @Column("STUDENT_ID")
	private Long id;
	private String userName;
	private String grade;
	@ManyToOne(fetch = FetchType.LAZY)
	@JoinCoulmn(name = "SCHOOL_ID")
	private School school
}
```

### QueryDSL
`QueryDSL`은 일종의 표현식으로 기존 언어의 메서드를 사용하여 데이터베이스에 질의를 할 수 있다.
maven, gradle 설정으로 `QClass` 를 Entity로부터 자동으로 생성해준다.

Spring Data JPA 에서는 queryDSL을 같이 사용할 수 있는 기반 클래스를 제공 하는데 그 클래스가 `QueryDslRepositorySupport` 이다.
JPA 사용할떄의 기존 Repository에 기능을 추가하는 형태로 개발 진행이 가능하다.
`QueryDslRepositorySupport`는 추상 클래스이므로 구현체를 만들어야 하는데 절차가 필요하다.
1. `QClass` 생성 (Build Tool 설정)
2. `Custom Repository Interface` 생성 (JPA에서 지원하지 않는 쿼리에 대한 Interface 생성)
3. `QueryDslRepositorySupport` 구현체 작성
4. `Custom Repository` 생성

```java
public interface UserRepositoryCustom {
	List findAllLike(String keyword);
}

public class UserRepositoryImpl extends QuerydslRepositorySupport implements UserRepositoryCustom {
    public UserRepositoryImpl() {
        super(UserEntity.class);
    }
    @Override
    @Autowired
    public void setEntityManager(EntityManager entityManager) {
        super.setEntityManager(entityManager);
    }
    @Override
    public List findAllLike(String keyword) {
        QUserEntity qUserEntity = QUserEntity.userEntity;
        JPQLQuery<UserEntity> query = from(qUserEntity);
        List<UserEntity> resultList = 
		        query.where(qUserEntity.username.like(keyword)).fetch();
        return resultList;
    }
}
```
> `QueryDslRepositorySupport`는 Entity Class에 대한 의존성이 강해서 생성시에 전달받지 못하면 인스턴스가 생성되어도 사용할 수 없으며 다른방법으로 Entity를 주입 할 수 없게 되어있다. (setEntity 없음)

## MyBatis
MyBatis에서는 쿼리 매핑 구문을 실행하기 위해 sqlSession	객체를 사용하고 `sqlSession` 객체를 생성하기 위해서 `SqlSessionFactory`를 사용한다. 스프링과 같이 사용할 때는 `SqlSessionTemplate`을 쓴다.
 그 이유는 스프링 사용시에 sqlSession을 대체하여주고 예외 처리를 Mybatis가 아니라 Spring의 `DataAccessException`으로 치환 시켜준다.

Mybatis에서는 Root Resource 폴더에 DDL, DML 파일을 두고 실행되게 할 수 있다.
application.properties 파일에 지정 가능한 옵션은 5가지이댜
- **Spring.database.platform** : 데이터 베이스 유형
- **Spring.datasource.schema** : 스키마를 정의하는 SQL파일
- **Spring.datasource.data** : 데이터를 입력하는 SQL 파일
- **Spring.datasource.separator** : SQL 구분자
- **Spring.datasource.sql-script-encoding** : SQL 인코딩 옵션


## Connection pool
Tomcat 7 버전까지는 DBCP1 - 싱글 스레드로 서버에 부하가 많이 몰릴 경우에 대기열이 생기는 문제가 많음
