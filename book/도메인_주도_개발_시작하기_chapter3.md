# Chapter 3. 애그리거트

## 3.1 애그리거트

개별 객체 수준에서 모델을 바라보면 상위 수준에서 관계를 파악하기 어려움

복잡한 도메인을 이해하고 관리하기 쉬운 단위로 만들려면 상위 수준에서 모델을 조망할 수 있는 방법이 애그리거트

관련되 객체를 하나의 군으로 묶어 모델을 이해하고 일관성을 관리하는 기준을 만든다



### 한 애그리거트에 속한 객체는 유사하거나 동일한 라이프 사이클을 갖는다 

도메인 규칙에 따라 최초 시점에 일부 객체를 만들 필요가 없는 경우도 있지만 애그리거트에 속한 구성요소는 대부분 함께 생성하고 함께 제거한다



### 애그리거트는 경계를 갖는다

한 애그리거트에 속한 객체는 다른 애그리거트에 속하지 않는다. 애그리거트는 독립된 객체 군이며 각 애그리거트는 자기 자신을 관리할 뿐 다른 애그리거트를 관리하지 않는다. 경계 설정의 기본이 되는 것은 도메인 규칙과 요구사항이다

애그리거트가 한개의 엔티티 객체만 갖는 경우가 많으며 두개 이상의 엔티티로 구성되는 경우는 드물다



## 3.2 애그리거트 루트와 역할

에거리거트에 속한 모든 객체가 일관된 상태를 유지하려면 애그리거트 전체를 관리할 주체가 필요한데, 이 책임을 지는 대표 엔티티가 애그리거트의 루트 엔티티. 에

## 3.2.1 도메인 규칙과 일관성

애그리거트가 제공해야 할 도메인 기능을 구현 

## 3.2.2 애그리거트 루트의 기능 구현

애그리거트 루트는 내부의 다른 객체를 조합해서 기능을 완성한다

## 3.2.3 트랜잭션 범위

한 트랜잭션에서 한 애그리거트만 수정 -> 애그리거트에서 다른 애그리거트를 변경하지 않는다.

애그리거트가 자신의 책임 범위를 넘어 다른 애그리거트의 상태까지 관리하는 꼴이 되며 결합도가 높아져 수정 비용이 높아진다. 부득이 하게 한 트랜잭션으로 처리해야 할 경우 응용서비스에서 수정하도록 구현한다.



## 3.3 리포지터리와 애그리거트

애그리거트는 개념상 완전한 한 개의 도메인 모델을 표현하므로 객체의 영속성을 처리하는 리포지터리는 애그리거트 단위로 존재. 물리적으로 가가 별도의 DB테이블에 저장한다고 해서 리포지터리를 각각 만들지 않느다. 



## 3.4 ID를 이용한 애그리거트 참조

애그리거트 간 참조는 애그리거트의 루트를 참조 한다는 것.

내부 필드를 통한 다른 애그리거트 직접 참조(JPA의 경우 @ManyToOne, @OneToOne과 같은 애너테이션 사용으로 연관된 객체 로딩)는 구현은 편하지만  다른 애그리거트 상태를 변경할 수 있는 위험, 로딩시 성능을 고민해야 하는점 (참조객체의 lazy, eager 로딩 ), 추후 확장성 문제(MSA로 확장등)가 있음.

따라서 ID를 이용해서 다른 애그리거트를 간접 참조한다. ID 참조를 사용하면 모든 객체가 참조로 연결되지 않고 한 애그리거트에 속한 객체들만 참조로 연결된다. 애그리거트의 경계를 명확히 하고 애그리거트 간 물리적인 연결을 제거하기 때문에 모델의 복잡도를 낮춰준다. 또한 애그리거트 간의 의존을 제거하므로 응집도를 높여주는 효과도 있다.

구현 복잡도도 낮아짐. 참조하는 애그리거트가 필요하면 응용 서비스에서 ID를 이용해서 로딩하면 된다.(애글거트 수준에서 지연 로딩을 하는 것과 동일한 결과를 만듦)



## 3.4.1 ID를 이용한 참조와 조회 성능

조인을 이용하면 한번에 모든 데이터 조회가 가능하지만 ID참조는 N+1 조회 문제가 생기며 속도 저하의 원인이 된다. 해결책으로 데이터 조회를 위한 별도 DAO를 만들고 DAO의 조회 메서드에서 조인을 이용해 한 번의 쿼리로 필요한 데이터를 로딩한다. 애그리거트마다 서로 다른 저장소를 사용하면 캐시를 적용하거나 조회 전용 저장소를 따로 구성한다. 코드가 복잡해지는 단점이 있지만 시스템의 처리량을 높일 수 있다



## 3.5 애그리거트 간 집합 연관

- 1-N, N-1 

  카테고리와 상품간 관계가 대표적. 카테고리 별 상품을 검색하게 되면 보통 페이징을 해서 보여주는데 카테고리를 중심으로 상품을 검색하면 카테고리에 속한 모든 상품을 조회한후 결과를 보내게 되고 상품 갯수가 많으면 실행 속도가 급격히 느려진다. 상품 입장에서 자신이 속한 카테고리를 N-1로 연관 지어 페이징 처리 하면서 조회하면 된다.

  *개념적으로 존재하는 애그리거트 간의 1-N 연관을 실제 구현에 반영하는 것이 요구사항을 충족하는 것과는 상관없을 때가 있다.

- M-N

  상품이 여러 카테고리에 속할 수 있다고 가정했을 때 실제 요구사항을 고려하여 M-N연관을 구현에 포함 시킬지를 결정해야 한다.제품이 속한 모든 카테고리가 필요한 화면은 상품 상세 화면이므로  상품에서 카테고리로의 집합 연관만 존재하면 된다. 즉 개념적으로는 상품과 카테고리의 양방향 M-N 연관이 존재하지만 실제 구현에서는 상품에서 카테고리로의 단방향 M-N 연관만 적용하면 됨.  실제 구현은 JPA 어노테이션 + JPQL로 구현 



## 3.6 애그리거트를 팩토리로 사용하기

에그리거트가 갖고 있는 데이터를 이용해서 다른 애그리거트 생성해야 한다면 애그리거트에 팩토리 메서드를 구현하는 것을 고려해야 함. 예를 들어 상품을 생성할 때 상점의 오픈여부가 필요하면 상점의 애그리거트 내부에 상품 애그리거트 생성 메소드를 만듦. 이때 많은 정보를 알아야 한다면 상점 애그리거트에서 상품을 직접 생성하지 않고 다른 팩토리에 위임하는 방법도 있다 예를 들어 ProductFactory 같은걸 만들어 내부 create 매서드에서 다양한 정보를 얻은 뒤 상품을 생성한다.