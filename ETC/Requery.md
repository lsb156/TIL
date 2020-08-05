# Requery

<!--[TOC]: # "## Table of Contents"-->

- **No reflection** (apt code generation) - Fast instancing, (vs Hibernate Proxy = No LazyInitialize)
- Typed Query Language (vs Hibernate Criteria)
- Fast startup & performance
- Entity is stateless (vs Hibernate stateful)
- Thread에 제한 받지 않음 (vs JPA EntityManager)
- Schema generation
- **Blocking / Non-blocking API (Reactive with RxJava)**
- Support RxJava, Async Operations, Java8
- Support partial object / refresh / **upsert**
- Custom type converter like JPA
- Compile time entity/query validation (vs Hibernate)
- Support almost JPA annotaion

## Table of Contents
- [Why Requery](#why-requery)
- [EntityDataStore](#entitydatastore)
- [spring-data-requery](#spring-data-requery)

## Why Requery
- Provide benefit of ORM
	- Entity Mapping
	- Schema Generation
	- Compile time error datecting
- Performance
	- When bulk job, max 100x then JPA
	- REST API - 2~10x throughput
	- Support Upsert, Lazy loading ...

|define          |JPA             |requery     |
|:---------------|:---------------|:-----------|
|Entity Class    |@Entity         |@Entity     |
|Identifier      |@Id             |@Key        |
|Relationships   |@OnetoMany      |@OnetoMany  |
|Auto Generated  |@GeneratedValue |@Generated  |
|Converter       |                |@Converter  |
|Listeners       |                |@PostLoad   |

``` java
	// Query by Fluent API
	Result<Person> query = data
		.select(Person.class)
		.where(Person.NAME.lower().like("b%"))
		.and(Person.AGE.gt(20))
		.orderBy(Person.AGE.desc())
		.limit(5)
		.get();

	// Reactive Programming Cold Observable Non Blocking
	Observable<Person> observable = data
		.select(Person.class)
		.orderBy(Person.AGE.desc())
		.get()
		.observable();	
```

``` kotlin
// kotlin
@Entity(model = "tree")
interface TreeNode {
	@get:Key
	@get:Generated
	val id: Long
	
	@get:Column
	var name: String
	
	@get:ManyToOne(cascade = [DELETE])
	var parent: TreeNode?

	@get:OneToMany(mappedBy = "parent", cascade = [SAVE, DELETE])
	val children: MutableSet<TreeNode>
}
```

Requery는 Blob 같은 대용량 처리가 미비함
``` kotlin
// requery - Blob/Clob usage
class ByteArrayBlobConverter : Converter<ByteArray, Blob> {

	override fun getPersistedSize(): Int? = null
	
	override fun getPersistedType(): Class<Blob> = Blob::class.java
	
	override fun getMappedtype(): Class<ByteArray> = ByteArray::class.java
	
	override fun convertToMapped(type: Class<out ByteArray>?, value: Blob?): ByteArray? {
		return value?.binaryStream?.readBytes()
	}
	
	override fun convertToPersisted(value: ByteArray?): Blob? {
		return value?.let { SerialBlob(it) }
	}
}
```
## EntityDataStore
- `findByKey`
- `select` / `insert` / `update` / `update` / `delete`
- `where` / `eq`, `lte`, `lt`, `gt`, `gte`, `like`, `in`, `not` ...
- `groupBy` / `having` / `limit` / `offset`
- support SQL Functions
	- `count`, `sum`, `avg`, `upper`, `lower` ...
- raw query (Plain SQL)

## spring-data-requery
- RequeryOperations
	- Wrap EntityDataStore
- RequeryTransactionManager for TransactionManager
	- Support Spring @Transactional
- Better performance than spring-data-jpa
	- when exists, paging, not load all entities

> 출처 : https://www.youtube.com/watch?v=2zQdmC0vnFU
