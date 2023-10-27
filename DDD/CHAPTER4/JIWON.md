## JPA를 이용한 레포지토리 구현

---

### 모듈 위치

레포지토리 인터페이스는 도메인 영역에 속하고, 레포지토리를 구현한 구현체는 인프라 영역에 속한다.

가능하면 구현체를 인프라 영역에 두어 인프라 영역에 대한 의존을 낮춰야한다.

### 레포지토리 기본 기능 구현

제공하는 기본 기능은 2가지 이다.

- ID로 애그리거트 조회하기
- 애그리거트 저장하기

**인터페이스는 애그리거트 루트를 기준으로 작성**

그리고 해당 인터페이스를 JPA로 구현해주면 된다.

> Spring Data JPA를 사용하면 레포지토리 구현 객체를 Spring Data JPA가 자동으로 만들어준다.
실질적으로 레포지토리 인터페이스를 직접 구현할 일은 거의 없다.
> 

변경은 변경 감지를 활용하면 된다.

ID외의 다른 조건으로 애그리거트를 조회할 때는 Criteria나 JPQL을 활용할 수 있다.

> 삭제 요구사항이 있더라도 데이터를 실제로 삭제하지 않고 soft delete를 하는 방식으로 삭제한다.
JPA에서 soft delete 사용하는 방법: https://velog.io/@max9106/JPA-soft-delete
> 

## Spring Data JPA를 이용한 레포지토리 구현

---

Spring Data JPA는 다음 규칙에 따라 작성한 인터페이스를 찾아서 인터페이스를 구현한 스프링 빈 객체를 자동으로 등록한다.

- `Repository<T, ID>` 인터페이스 상속
- T: Entity Type, ID: PK Type

Spring Data JPA는 지정한 규칙에 맞게 메서드를 작성해야 한다.

→ 규칙에 맞게 안만들면 쿼리 생성 제대로 안되는 이슈 발생할 수 있음

## 매핑 구현

---

### 엔티티와 밸류 기본 매핑 구현

애그리거트 루트는 엔티티이므로 `@Entity` 로 매핑 설정한다.

밸류는 `@Embeddable`로 매핑 설정한다.

밸류 타입 프로퍼티는 `@Embedded`로 매핑 설정한다.

책 140p~141p 예제

### 기본 생성자

JPA에서 `@Entity`와 `@Embeddable` 로 클래스를 매핑하려면 기본 생성자가 필요하다.

→ 프록시 객체를 만들기 때문에

기본 생성자는 `protected`로 접근 제어자를 막아두자

### 필드 접근 방식 사용

메서드 방식을 사용하면 `getter`, `setter`를 추가해줘야 되니 필드 접근 방식을 사용하자

→ 객체가 제공할 기능 중심으로 엔티티를 구현하게끔 유도하자

### AttributeConverter를 이용한 밸류 매핑 처리

밸류 타입의 프로퍼티를 한 개의 칼럼에 매핑해야 할 때가 있다.

이 때는 `@Embeddable` 을 사용할 수 없고 AttributeConverter를 활용해야한다.

```java
@Getter
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Money {

    public static final Money ZERO = new Money(0L);

    private final Long value;

    public static Money from(final Long value) {
        return new Money(value);
    }

    public Money multiply(final Long value) {
        return new Money(this.value * value);
    }

    public Money add(final Money money) {
        return new Money(this.value + money.value);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof Money)) {
            return false;
        }
        Money money = (Money) o;
        return Objects.equals(value, money.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }

    @Override
    public String toString() {
        return "Money{" +
            "value=" + value +
            '}';
    }
}

----------------------------------------------------------------------

@Converter
public class MoneyConverter implements AttributeConverter<Money, Long> {

    @Override
    public Long convertToDatabaseColumn(final Money attribute) {
        return attribute.getValue();
    }

    @Override
    public Money convertToEntityAttribute(final Long dbData) {
        return Money.from(dbData);
    }
}

-----------------------------------------------------------------------

@Convert(converter = MoneyConverter.class)
@Column(name = "total_price")
private Money totalPrice;
```

### 밸류 컬렉션: 별도 테이블 매핑

밸류 컬렉션을 별도 테이블로 매핑할 수 있다.

`@ElementCollection`과 `@CollectionTable`을 함께 사용하여 매핑한다.

`@OrderColumn` 어노테이션을 통해 **지정한 칼럼에 리스트의 인덱스 값을 저장**할 수 있다.

### 밸류 컬렉션: 한 개 칼럼 매핑

별도 테이블이 아닌 한 개 칼럼에 저장해야 할 때가 있다.

이 때 AttributeConverter를 사용하면 밸류 컬렉션을 한 개의 칼럼에 쉽게 매핑할 수 있다.

단, AttributeConverter를 사용하기 위한 **밸류 컬렉션을 포함하는 새로운 밸류 타입(일급 컬렉션)**을 추가해야 한다.

### 밸류를 통한 ID 매핑

`@EmbeddedId` 어노테이션으로 밸류 타입을 식별자로 매핑할 수 있다.

식별자 타입은 `Serializable` 인터페이스를 상속 받아야 한다.

### 별도 테이블에 저장하는 밸류 매핑

애그리거트에서 루트 엔티티를 뺀 나머지 구성요소는 대부분 밸류이다.

→ 단지 **별도 테이블에 데이터를 저장한다고 해서 엔티티인 것은 아니다.**

루트 엔티티 외의 엔티티가 있다면 진짜 엔티티인지 의심해 봐야 한다.

책 152p~154p 예시들 살펴보기

### 밸류 컬렉션을 @Entity로 매핑하기

밸류이지만 구현 기술의 한계나 팀 표준 때문에 `@Entity`를 사용해야 할 때도 있다.

상속 구조를 가지는 밸류 타입을 사용하려면 `@Entity` 를 활용하여 상속 매핑으로 처리해야 한다!

→ 책 156p~157p

최상위 클래스를 추상 클래스로 만들고 아래의 설정을 적용한다.

- `@Inheritance` 적용
- `strategy` 값으로 `SINGLE_TABLE`사용 → 하나의 테이블에서 관리되도록
- `@DiscriminatorColumn` 을 이용하여 타입 구분용으로 사용할 칼럼 지정

상속 받은 클래스는 `@Discriminator` 를 사용하여 매핑을 설정한다.

책 159p 다시 읽어보기

### ID 참조와 조인 테이블을 이용한 단방향 M-N 매핑

애그리거트 간 집합 연관은 성능 상의 이유로 피해야 한다.

만약 집합 연관을 사용하는 것이 유리하다면 ID 참조를 이용한 단방향 집합 연관을 적용할 수 있다.

```java
@ElementCollection
@CollectionTable(세팅 생략)
private Set<MemberId> memberIds;
```

이러한 형태로 적용할 수 있다.

## 애그리거트 로딩 전략

---

애그리거트에 속한 객체가 모두 모여야 완전한 하나가 된다.

→ 애그리거트 루트를 로딩하면 루트에 속한 모든 객체가 완전한 상태여야 한다.

즉시 로딩을 하면 조회 시점에 애그리거트를 완전한 상태로 만들 수 있지만 조회 성능에 이슈가 발생한다.

→ 조인하는 과정에서 카티시안 곱이 발생하기 때문에

**애그리거트를 즉시 로딩으로 두지 않고 지연 로딩으로 두어 변경이 필요한 시점에 로딩하는 것이 좋다!**

다만 지연 로딩은 보내는 쿼리가 많아지는 단점이 있기 때문에 **즉시 로딩과 지연 로딩은 상황에 맞게 사용하는게 좋다!**

## 애그리거트의 영속성 전파

---

애그리거트가 완전한 상태 → 저장하고 삭제할 때도 하나로 처리되어야 한다.

- 저장 메서드는 애그리거트 루트만 저장하면 안되고 애그리거트에 속한 모든 객체를 저장해야 한다.
- 삭제 메서드는 애그리거트 루트뿐만 아니라 애그리거트에 속한 모든 객체를 삭제해야 한다.

**cascade 옵션**을 줘서 함께 저장, 삭제되도록 설정할 수 있다.

## 식별자 생성 기능

---

식별자를 만드는 과정은 다음 중 하나이다.

- 직접 생성
- 도메인 로직으로 생성
- DBMS에서 제공하는 방법 사용

식별자 생성 규칙이 있다면 식별자를 생성하는 기능을 따로 분리해야 한다.

→ 식별자 생성 규칙은 도메인 규칙이므로 도메인 영역에 식별자 생성 기능을 위치해야 한다.

도메인 서비스, 레포지토리에 이러한 로직을 만들 수 있다.

## 도메인 구현과 DIP

---

JPA를 통해 구현한 레포지토리는 DIP를 위반한다.

구현 클래스를 인프라에 옮길 수 있지만 JPA 기술은 거의 바뀌지 않기 때문에 개발 편의성과 실용성을 가져가는 것이 좋다.
