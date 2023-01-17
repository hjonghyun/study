# Chapter 4

## 리포지터리와 모델 구현



## JPA를 이용한 리포지터리 구현

객체 기반의 도메인 모델과 관계형 모델 간의 매핑을 처리하는 기술이 ORM 이고 자바의 ORM 표준이 JPA이다

리포지터리 인터페이스는 애그리거트와 같이 도메인 영역에 속하고 리포지터리를 구현한 클래스는 인프라스트럭처 영역에 속한다

4.1.2 리포지터리 기본 기능 구현

- ID로 애그리거트 조회하기
- 애그리거트 저장하기

인터페이스는 애그리거트 루트를 기준으로 작성한다.

스프링 데이터 JPA를 이용하면 구현 객체를 JPA가 알아서 해준다.

애그리거트를 수정한 결과를 저장소에 반영하는 메서드를 추가할 필요는 없다. JPA가 자동으로 변경 데이터를 반영한다.



스프링 데이터 JPA를 이용한 OrderRepository 인터페이스

```java
public interface OrderRepository extends Repository<Order, OrderNo> {
	Optional<Order> findById(OrderNo id);
	
	void save(Order order);
}
```

- Org.springframework.data.repository.Repository<T,ID> 인터페이스 상속
- T는 엔티티 타입, ID는 식별자 타입

 OrderRepository 기준

엔티티 저장 메서드는 다음중 하나를 사용

- Order save(Order entity)
- Void save(Order entity)



식별자 사용 엔티티 조회시 findById()

- Order findById(OrderNo id) - 없을 때 null 리턴
- Optional<Order> findById



특정 프로퍼티 이용 엔티티 조회시 findBy프로퍼티이름(프로퍼티 값) 

- list<Order> findByOrderer(Orderer orderer)

중첩 프로퍼티의 경우 아래 메서드는 Orderer 객체의 memberID 프로퍼티가 파라미터와 같은 Order목록을 조회한다

- List<Order> findByOrdererMemberId(MemberId memberId) 



엔티티 삭제는

- void delete(Order order)
- void deleteByID(OrderNo id)



## 엔티티티와 밸류 매핑

- 애그리거트 루트는 엔티티이므로 @Entity로 매핍
- 밸류는 @Embeddable 
- 애그리거트 루트의 엔티티에 속하는 밸류 타입 프로퍼티는 @Embedded로 매핑



4.3.2 기본 생성자

엔티티와 밸류의 생성자는 객체를 생성할 때 필요한 것을 전달받는다

원래라면 밸류 타입은 기본생성자가 필요 없지만 JPA가 DB에서 데이터를 읽어와 매핑된 객체를 생성 하려면 기본 생성자를 추가 해야한다

JPA 프로바이더가 객체를 생성할 때만 사용할 수 있게 protected로 선언한다



4.3.3 필드 접근 방식 사용

JPA는 필드와 메서드 두 가지 방식으로 매핑을 처리할 수 있다. 

메서드 방식

- 프로퍼티를 위한 get/set 메서드 구현필요
- class에 @Access(AccessType.PROPERTY) 명시
- 객체가 제공할 기능 중심으로 엔티티를 구현하게끔 유도하려면 JPA 매핑 처리를 프로퍼티 방식이 아닌 필드 방식으로 선택해서 불필요한 get/set 메서드를 구현하지 않는게 필요

필드 방식

- class에 @Access(AccessType.FIELD) 명시



4.3.4 AttributeConverter를 이용한 밸류 매핑 처리

다수의 밸류 타입의 프로퍼티를 한개 컬럼에 매핍해야 하는 경우 AttributeConverter 인터페이스 구현

- @Converter 애너테이션 적용



4.3.5 밸류 컬렉션: 별도 테이블 매핑



## 애그리거트 로딩 전략과 영속성 전파

## 식별자 생성 기능

