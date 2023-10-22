- CQRS
    - 명령 모델, 조회 모델을 분리하는 패턴
    - 상태 변경 기능과 데이터 조회 기능
    - 도메인 모델 : 명령 모델로 주로 사용
    - 정렬, 페이징, 검색조건 지정 : 조회 기능에 사용

# 검색을 위한 스펙

- 스펙
    - 검색 조건을 다양하게 조합해야할 때 사용
    - 애그리거트가 특정 조건을 충족하는지 검사할 때 사용
    - 모든 애그리 거트 객체 메모리에 보관하기 어렵(OOM),. 다 보관하더라도 조회 성능에 심각한 이슈 발생

```java
 public inteface Specification<T> {
	public boolean isSatisfiedBy(T agg); 
}
```

- agg는 검사 대상이 되는 객체를 의미.
- 스펙을 리포지터리에 사용하면 agg는 애그리거트 루트가 되고, 스펙을 DAO에 사용하면, agg는 검색 결과를 리턴할 데이터 객체가 된다.

<aside>
💡 리포지터리와 DAO의 차이?

</aside>

- 레포지터리나 DAO는 검색 대상을 걸러내는 용도로 스펙 사용

# 스프링 데이터 JPA를 이용한 스펙 구현

# Specification

- 스프링 데이터 JPA 가 제공하는 스펙 인터페이스

```java
package org.springframework.data.jpa.domain;

public interface Specification<T> extends Serializable {
	@Nullable
	Predicate toPredicate(Root<T> root,
    				CriteriaQuery<?> query,
    				CriteriaBuilder cb);
}

public class OrdererIdSpec implements Specification<OrderSummary> {
    private String ordererId;

    public OrdererIdSpec(String ordererId) {
        this.ordererId = ordererId;
    }

    @Override
    public Predicate toPredicate(Root<OrderSummary> root,
					    CriteriaQuery<?> query,
    					CriteriaBuilder cb) {
        return cb.equal(root.get(OrderSummary_.ordererId), ordererId);
    }
}
```

- 제네릭 타입 파라미터 T : JPA 의 엔티티 타입
- toPredicate() : JPA 크라테리아 API에서 조건을 표현하는 Predicate을 생성.
- 스펙 인터페이스는 함수형 인터페이스로, 람다식을 이용하여 객체 생성

## JPA 정적 메타 모델

- @StaticMetamodel 애너테이션 이용하여 관련 모델 지정
- 메타 모델 클래스는 모델 클래스 이름 뒤에 ‘_’를 붙인 이름 갖음
- 대상 모델의 각 프로퍼티와 동일한 이름을 갖는 정적 필드 정의
- Criteria를 사용할 때 정적 메타 모델 클래스 사용이 코드 안정성이나 생산성 측면에서 유리
- 하이버네이트와 같은 JPA프로바이더는 정적 메타 모델을 생성하는 도구 제공

# 스펙 조합

- 스프링 데이터 JPA는 스펙 조합 메서드 제공 → and(), or()
- and() 와 or() 메서드는 스펙을 조합할 수 있다
    - 두 메서드는 기본 구현을 가진 디폴트 메서드이다
    - and() 메서드는 두 스펙을 모두 충족하는 조건을 표현하는 스펙을 생성한다
    - or() 메서드는 두 스펙 중 하나 이상을 충족하는 조건을 표현하는 스펙을 생성한다
- not()은 정적 메서드로 조건을 반대로 적용할 때 사용한다
- where() 메서드는 스펙 인터페이스의 정적 메서드로 null을 전달하면 아무 조건도 생성하지 않는 스펙 객체를 리턴하고 null이 아니면 인자로 받은 스펙 객체 자체를 그대로 리턴한다.
- 

```java
Specification<OrderSummary>spec = Specification
.where(createNullableSpec())
.and(createOtherSpec());
```

# 정렬 지정

- 스프링 데이터 JPA는 두가지 방법 사용하여 정렬 지정
    - OrderBy를 사용하여 정렬 기준 지정
        - 정렬 기준 프로퍼티 두개 이상이면 메서드 이름 길어진다.
        - 메서드 이름으로 정렬 순서 정해져 정렬 순서 변경할 수 없다.
    - Sort를 인자로 전달

```java
Sort sort = Sort.by("number")
.ascending()
.and(Sort.by("orderDate")
.descending());
```

# 페이징 처리

- 스프링 데이터 JPA는 페이징 처리를 위해 Pageable 타입 사용
- Pageable 타입은 인터페이스로 구현체는 PageRequest 클래스를 사용
- Page 타입은 데이터 목록 뿐만 아니라, 조건에 해당하는 전체 개수 구할 수 있다.
- 스펙을 사용하는 findAll 메서드도 Pageable 사용 가능
- Pageable 메서드 리턴 타입이 Page일 경우, Count 쿼리도 실행하여 조건에 해당하는 데이터 개수 구함
    - findBy프로퍼티 형식은 List 리턴 타입일 때는 count 쿼리 x
    - 스펙 사용하는 findAll 메서드에 Pageable사용하면 리턴 타입 List 여도 count 쿼리 나감. → 이를 방지하려면 커스텀 리포지터리 직접 구현
- findFirstN 형식 : 처음 세개 가져온다. (Top 도 가능)

→ List로 받아라 가능하면 (Spring Data JPA) 

# 스펙 조합을 위한 스펙 빌더 클래스

- 스펙 빌더
    - if, 스펙 조합 코드는 복잡하고, 실수증가.
    - 조건을 표현, 메서드 호출 체인으로 연속된 변수 할당 줄여 코드 가독성 높이고 구조 단순
    - and(), ifHasText(), ifTrue() 메서드
    

# 동적 인스턴스

- 쿼리 결과에서 임의의 객체 동적 생성 기능
    - **JPQL 의 new 인스턴스 생성자를 통해. new A(a, b, c)**
- 동적 인스턴스 장점
    - JPQL 을 그대로 사용 → 객체 기준으로 쿼리 작성, 동시에 지연, 즉시 로딩같은 고민 없이 데이터 조회 가능.
    - 

## 하이버 네이트 @Subselect 사용

- Query결과를 @Entity로 매핑하는 유용한 기능
- @Immutable, @Subselect, @Synchronize는 하이버네이트 전용 애너테이션
- @Subselect는 조회 쿼리를 값으로 가져서 매핑할 테이블로 사용
- @Entity 역시 수정할 수 없다. → immutable을 통해, 하이버 네이트는 해당 엥닡티의 매핑 필드/프로퍼티 변경되도 DB에 반영하지 않고 무시
- SubSelect는 쿼리 전에 flush를 안해주기 때문에,만약 그전에 연관되어있는 테이블의 수정이 일어난다면, 이전의 값들이 들어갈수 있다.
    - JPQL 쿼리는 동작 전에 flush를 해준다.
        
        [영속성 관리 - 내부 동작 방식](https://www.notion.so/453f3d786655466f98314bd2f2b0cf5e?pvs=21)
        
- Synchronize를 통해 하이버 네이트가 엔티티를 로딩하기 전에 지정한 테이블과 관련된 변경 발생하면 플러시 한다→ 이를 통해 로딩전에 변경내역 반영
- 일반 entity와 같기 때문에, find, JPQL, Criteria 조회 가능
- @Subselect는 이름처럼 @Subselect의 값으로 지정한 쿼리를 from절의 서브 쿼리로 사용한다.

# 참고자료

- SubSelect와 관련된 N+1 문제

[JPA N+1 쿼리 이슈](https://yearnlune.github.io/java/java-jpa-n1/#)
