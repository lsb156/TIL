# Spring Data JPA
Spring에서 Spring Data JPA 사용하는 이유
- 구현체 교체의 용이성 : Hibernate 외에 다른 구현체로 쉽게 교체하기 위함
- 저장소 교체의 용이성 : RDB 외에 다른 저장소로 쉽게 교체하기 위함

SpringDataJpa 사용중에 SpringDataMongoDB로 변경하여도 인터페이스가 동일하여 문제가 없음

``` java
@Getter
@NoArgsConstructor
@Entity
public class posts {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public posts(Long id, String title, String content, String author) {
        this.id = id;
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```
1. @Entity
	- 테이블과 링크될 클래스
	- 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍으로 테이블 이름을 매칭
	(SalesManger.java -> sales_manager table)
2. @Id
	- 해당 테이블의 PK 필드
3. @GeneratedValue
	- 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 됨
	- [스프링 부트 2.0 버전과 1.5버전의 차이](https://jojoldu.tistory.com/295)

> 웬만하면 Entity의 PK는 Long 타입의 Auto_increment를 추천
> 1. FK키를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야 하는 상황이 발생
> 2. 인덱스에 좋은 영향을 끼치지 못함
> 3. 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생
>
> 주민등록번호, 복합키 등은 유니크 키로 별도로 추가하는것을 추천


## Entity 클래스에는 Setter 금지
Entity 클래스에서는 절대 Setter 메소드를 만들지 않으며 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드를 추가해야함.
``` java
// ========= 안좋은 사용의 예 =========
public class Order {
	public void setStatus(boolean status) {
		this.status = status;
	}
}
public void 주문서비스의_취소이벤트() {
	order.setStatus(false);
}


// ========= 올바른 사용 예 =========
public class Order {
	public void cancelOrder() {
		this.status = false;
	}
}
public void 주문서비스의_취소이벤트() {
	order.cancelOrder(false);
}
```
기본의 Setter를 안씀으로써 자동으로 디비에 반영이 안되지만 기본적인 구조는 생성자를 통해 최종값을 채운 후 DB에 삽입하는 방식으로 변경하며 해당 이벤트에 맞는 public 메소드를 호출 하여 변경하는 것을 전제로한다.

Entity의 값이 변경되면 JPA의 영속성 컨텍스트에서 더티 체킹을하여 자동으로 update 쿼리를 날려준다.
[더티 체킹이란?](https://jojoldu.tistory.com/415)


## CRUD API

### Spring Layer
- Web Layer
	- `컨트롤러(@Controller)`, `JSP/Freemarker` 등의 뷰 템플릿 영역
	- 이외에도 `필터(@Filter)`, `인터셉터`, `컨트롤러 어드바이스(@ControllerAdvice)` 등 외부 요청과 응답에 대한 전반적인 영역
- Service Layer
	- @Service에 사용되는 영역
	- Controller, Dao 중간영역
	- @Transactional이 사용되어야 하는 영역
- Repository Layer
	- Database와 같이 데이터 저장소에 접근하는 영역 == DAO (Data Access Object)
- Dtos
	- Dto(Data Transfer Object)는 계층간에 데이터 교환을 위한 객체를 이야기하며 Dtos는 이들의 영역
- Domain Model
	- 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화 시킨 것
	- Entity가 사용된 영역이 도메인 모델
	- 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아님 (VO처럼 값 객체들도 이 영역에 해당)

비지니스 모델을 처리해야 하는 영역은 **Domain**이다.
``` java
// 서비스 클래스에서 비즈니스 로직을 처리한 안좋은 예
@Transactional
public Order cancelOrder(int orderId) {
	OrdersDto order = ordersDao.selectOrders(orderId);
	BillingDto billing = billingDao.selectBillint(orderId);
	DeliveryDto delivery = deliveryDao.selectDelivery(orderId);

	String deliveryStatus = delivery.getStatus();
	
	if ("IN_PROGRESS".equals(deliveryStatus)) {
		delivery.setStatus("CANCEL");
		deliveryDao.update(delivery);
	}

	order.setStatus("CANCEL");
	orderDao.update(order);

	billing.setStatus("CANCEL");
	deliveryDao.update(billing);

	return order;
}
```
위 코드는 모든 로직이 서비스 클래스 내부에서 처리된다. 그러다보니 서비스 계층이 무의미하며, 객체란 단순히 덩어리 역할만 하게됨. 반면 도메인 모델에서 처리할 경우 다음과 같은 코드로 변환이 된다.
``` java
@Transactional
public Order cancelOrder(int orderId) {
	Orders order = orderRepository.findById(orderId);
	Billing billing = billingRepository.findByOrderId(orderId);
	Delivery delivery = deliveryRepository.findByOrderId(orderId);

	delivery.cancel();
	order.cancel();
	billing.cancel();
	
	return order;
}
```


## 화면에 넘기는 Data에 DTO를 사용하는 이유
Entity 클래스는 Database와 맞닿아있는 핵심 클래스이다.
Entity 클래스를 기준으로 테이블이 생성되고, 스키마가 변경된다.
화면 변경은 아주 잦고 사소한 변경들이 많아 Entity를 계속 수정하는것은 문제가된다.

## Querydsl
1. 타입 안정성이 보장
	- 단순한 문자열로 쿼리를 생성하는 것이 아니라, 메소드를 기반으로 쿼리를 생성
	- 오타나 존재하지 않는 컬럼명을 명시할 경우 IDE에서 자동으로 검출
	- 이 장점은 jooq에서도 지원하는 장점이지만 MyBatis에서는 지원하지 않음
2. 국내 많은 회사에서 사용 중
	- 쿠팡, 배민 등 JPA를 적극적으로 사용하는 회사에서는 Querydsl를 적극적으로 사용
3. 많은 레퍼런스
	- 많은 회사와 개발자들이 사용하다보니 그만큼 국내 자료가 많음

