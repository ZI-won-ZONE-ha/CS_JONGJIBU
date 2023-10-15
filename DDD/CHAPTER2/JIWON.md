# Chapter 2. 아키텍처 개요

---

## 4개의 영역

---

- 표현 영역: 사용자 요청을 받아 응용 영역에 전달하고 응용 영역의 처리 결과를 다시 사용자에게 보여주는 역할을 수행
- 응용 영역: 시스템에 사용자에게 제공해야 할 기능을 구현. 기능 구현을 위해 도메인 모델을 사용
- 도메인 영역: 도메인 모델을 구현한다.
- 인프라스트럭처 영역: 구현 기술에 대한 것을 다룬다. (RDB 연동, 메시징 큐, 몽고 DB 등) → 실제 구현을 다룸

## 계층 구조 아키텍처

---

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/792d154e-db8c-44df-8933-3bf1f6d25cd0)

표현 영역과 응용 영역은 도메인 영역을 사용하고 도메인 영역은 인프라스트럭처 영역을 사용한다.

상위 계층에서 하위 계층으로의 의존만 존재하고 하위 계층은 상위 계층에 의존하지 않는다.

구현의 편리함을 위해 계층 구조를 유연하게 적용하기도 한다.

→ 응용 계층은 도메인 계층에 의존하지만 외부 시스템과의 연동을 위해 더 아래 계층인 인프라 계층에 의존하기도 한다.

→ 다만 이런 설계를 할 경우 응용 계층과 도메인 계층이 인프라 계층에 종속적이게 되는 문제가 있다.

이렇게 **코드가 인프라스트럭처에 의존하게 된다면 테스트가 어려워지고 기능 확장이 어려워진다.**

## DIP

---

고수준 모듈: 의미 있는 단일 기능을 제공하는 모듈 ex) 가격 할인 계산

저수준 모듈: 하위 기능을 실제로 구현한 것 ex) 회원 정보 가져오기, 주문 정보와 회원 정보를 활용해 룰 실행하기

고수준 모듈이 제대로 동작하려면 저수준 모듈을 사용해야 한다.

→ **고수준 모듈이 저수준 모듈을 사용하면 구현 변경과 테스트가 어렵다는 문제가 발생!!**

**DIP는 이 문제를 해결하기 위해 저수준 모듈이 고수준 모듈에 의존하도록 변경**

→ 추상화인 **인터페이스**를 활용하여 저수준 모듈이 고수준 모듈에 의존하도록 변경한다.

```java
// 인터페이스
public interface PaymentClient {
	Mono<Void> validatePayment(String paymentKey, String orderToken, Money totalPrice);
}

// 구현체
public class TossPaymentClient implements PaymentClient {

	private final Duration timeout;
	private final int maxRetry;
	private final WebClient webClient;
	private final String paymentSecretKey;
	private final String url;

	public TossPaymentClient(
		final WebClient webClient,
		@Value("${payment.toss.secret-key}") final String paymentSecretKey,
		@Value("${payment.toss.confirm-url}") final String url,
		@Value("${payment.toss.timeout}") final int timeout,
		@Value("${payment.toss.max-retry}") final int maxRetry
	) {
		this.webClient = webClient;
		this.paymentSecretKey = paymentSecretKey;
		this.url = url;
		this.timeout = Duration.ofMillis(timeout);
		this.maxRetry = maxRetry;
	}

	public Mono<Void> validatePayment(final String paymentKey, final String orderToken,
		final Money totalPrice) {
		// 로직 생략
	}
}
```

그 다음 해당 기능을 사용하는 클래스가 인터페이스에 의존하도록 설계하면 해당 클래스는 추상화에 의존하지 구현에 의존하지 않을 수 있다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/7ea02661-15a0-4701-b12e-59ca0ce067a6)

이런 구조로 바뀌게 된다.

`PaymentClient`는 고수준 모듈에 속하고, `TossPaymentClient`는 저수준 모듈에 속한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/ccc28e1e-b1f2-4ceb-a9f6-2be09928c4bc)

고수준 모듈이 저수준 모듈을 사용하려면 고수준 모듈이 저수준 모듈에 의존해야 하는데 이렇게 저수준 모듈이 고수준 모듈에 의존하게 하는 것을 DIP(의존 관계 역전 원칙)라고 한다.

이렇게 DIP를 활용하게 되면 구현 교체가 어렵다는 점과 테스트가 어렵다는 점을 해결할 수 있다.

- 구현 기술 교체

```java
public class KakaoPayClient implements PaymentClient {

	public Mono<Void> validatePayment(final String paymentKey, final String orderToken,
		final Money totalPrice) {
		// 로직 생략
	}
}
```

단순히 구현체를 바꿔주면 된다. → Spring의 DI를 활용하면 쉽게 해결 가능!

- 테스트

```java
@Test
void test1() {
		PaymentClient paymentClient = mock(PaymentClient.class);
		// 뒤 로직 생략
}
```

Mock과 같은 대역 객체를 만드는 프레임워크 또는 사용자가 직접 대역 객체를 생성해서 테스트를 진행할 수 있다.

→ 실제 구현체 없이 간단하게 테스트가 가능하다.

### DIP 주의 사항

인터페이스를 적용할 때 인프라 영역에 인터페이스를 생성하면 도메인 영역이 인프라 영역에 의존하는 문제는 여전히 해결되지 않는다.

→ 해결하려는 문제의 핵심인 고수준 모듈이 저수준 모듈에 의존한다는 문제를 해결하지 못한다.

그래서 아래와 같은 패키지 구조로 설계해야 DIP를 활용하여 문제를 해결했다고 할 수 있다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/a2645f80-5619-43de-a56c-103d6012c66f)

## DIP와 아키텍처

---

DIP를 적용하게 되면 인프라 영역이 응용 영역과 도메인 영역에 의존하는 구조가 만들어진다.

즉, 저수준 모듈이 고수준 모듈에 의존하는 형태를 만들 수 있다.

**인프라 영역의 코드의 변경이 도메인과 응용 영역에 영향을 주지 않거나 최소화하는 방향으로 설계 가능**

> DIP를 항상 적용하지 않아도 된다.
구현 기술에 따라 구현 기술에 의존적인 코드를 도메인에 일부 포함하는 경우가 나은 경우가 있다!
ex) `@Transactioanl`
> 

## 도메인 영역의 주요 구성요소

---

| 요소 | 설명 |
| --- | --- |
| 엔티티 | 고유의 식별자를 가지는 객체, 자신의 라이프 사이클을 가진다. 도메인의 고유한 개념을 포함. 도메인 모델의 데이터를 포함하며 해당 데이터와 관련된 기능을 함께 제공한다. |
| 밸류 | 고유의 식별자를 가지지 않는 객체. 개념적으로 하나인 값을 표현할 때 사용한다. 엔티티의 속성 또는 다른 밸류의 속성으로 활용 가능 |
| 애그리거트 | 연관된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것. |
| 리포지토리 | 도메인 모델의 영속성을 처리한다. |
| 도메인 서비스 | 특정 엔티티에 속하지 않는 도메인 로직을 제공한다. 도메인 로직이 여러 엔티티와 밸류를 필요로 하면 도메인 서비스에서 로직을 구현한다. |

### 엔티티와 밸류

도메인 모델의 엔티티와 RDB의 엔티티는 다르다.

**도메인 모델의 엔티티는 데이터와 함께 도메인 기능을 함께 제공한다!**

**도메인 모델의 엔티티는 두 개 이상의 데이터가 개념적으로 하나인 경우 밸류 타입을 이용해 표현할 수 있다!**

→ RDB에서 이러한 관계를 표현하려면 별도의 테이블을 분리하여 표현해야 된다. (표현이 어려움)

### 애그리거트

도메인 모델도 개별 객체뿐 아니라 상위 수준에서 모델을 이해해야 한다.

→ 전체 모델의 관계와 개별 모델을 이해하는 데 도움이 된다.

도메인 모델에서 전체 구조를 이해하는 데 도움을 주는 것이 애그리거트이다.

애그리거트는 **관련 객체를 하나로 묶은 군집**이다.

ex) 주문 → 주문은 주문, 배송지 정보, 주문자, 주문 목록, 총 결제 금액의 하위 모델로 구성된다.

**애그리거트를 사용하면 개별 객체가 아닌 관련 객체를 묶어서 객체 군집 단위로 모델을 바라볼 수 있게 된다!**

→ 큰 틀에서 도메인 모델을 관리할 수 있다.

루트 엔티티: 군집에 속한 객체를 관리하는 엔티티

**루트 엔티티는 애그리거트에 속한 엔티티와 밸류를 이용하여 애그리거트가 구현해야 할 기능을 제공**

루트를 통해서 간접적으로 애그리거트 내의 엔티티나 밸류 객체에 접근

→ 애그리거트 단위로 구현을 캡슐화 할 수 있다.

### 리포지토리

물리적인 저장소에 도메인 객체를 보관하게 해주는 도메인 모델

**애그리거트 단위로 도메인 객체를 저장하고 조회하는 기능을 정의한다.**

리포지토리 인터페이스는 도메인 모델 영역, 이를 구현하는 클래스는 인프라 영역에 속한다.

응용 서비스와 밀접한 연관이 있다

- 필요한 도메인 객체를 구하거나 저장할 때 리포지토리 사용
- 트랜잭션 관리하는데, 트랜잭션 처리는 리포지토리 구현 기술의 영향을 받음

## 요청 처리 흐름

---

표현 영역에서 사용자의 요청을 받아 처리한다.

→ 사용자가 전송한 데이터 형식이 올바른지 검사하고 응용 영역이 원하는 형태로 데이터 전달

응용 서비스는 도메인 모델을 이용하여 기능을 구현.

→ 도메인의 상태를 변경할 때는 트랜잭션을 사용해야 한다. (물리 저장소에 올바르게 반영되도록)

## 인프라스트럭처 개요

---

표현, 응용, 도메인 영역을 지원한다.

다른 영역에서 필요로 하는 프레임워크, 구현체, 보조 기능을 지원한다.

DIP를 활용하여 인프라스트럭처에 구현체를 두는 방법으로 유연하게 설계 가능

그러나 무조건 의존을 없앨 필요는 없다.

→ 없애는 것이 더 큰 cost를 발생시키면 의존을 두는게 나을 수 있음

## 모듈 구성

---

- 도메인이 크면 하위 도메인마다 별도의 패키지 구성
- 도메인 모듈은 도메인에 속한 애그리거트를 기준으로 다시 패키지 구성

한 패키지에 너무 많은 타입이 몰려서 코드를 못 찾을 정도만 아니면 된다!
