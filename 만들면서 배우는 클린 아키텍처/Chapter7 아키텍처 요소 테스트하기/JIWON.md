# 아키텍처 요소 테스트하기

헥사고날 아키텍처의 테스트 전략에 대해서 알아보자

## 테스트 피라미드

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/380b12f9-dfd8-41d5-9cd7-5408798e587a)

비용이 많이 드는 테스트는 지양하고 비용이 적게 드는 테스트를 많이 만들어야 한다.

**테스트의 기본 전제: 만드는 비용이 적고, 유지보수가 쉽고, 빠르게 실행되고, 안정적인 작은 크기**

→ 단위 테스트

여러 단위가 합쳐지고 아키텍처의 경계, 시스템 경계를 결합하는 테스트는 만드는 비용이 비싸다.

→ 테스트 커버리지 목표를 작게 잡아야 한다.

**헥사고날 아키텍처를 테스트하기 위해 필요하다고 생각하는 계층: 단위, 통합, 시스템**

- 단위 테스트
    - 테스트 피라미드의 토대
    - 일반적으로 하나의 클래스를 인스턴스화하고 해당 클래스의 인터페이스를 통해 기능들을 테스트
    - 다른 클래스에 의존하는 클래스라면 다른 클래스들은 Mocking
- 통합 테스트
    - 연결된 여러 유닛을 인스턴스화하고 시작점이 되는 클래스의 인터페이스로 데이터를 보내 여러 유닛들이 잘 흐르는지 확인하는 테스트
    - 책에서 정의한 통합 테스트의 경우 두 계층 간의 경계를 걸쳐서 테스트할 수 있어 목을 대상으로 수행한다.
- 시스템 테스트
    - 모든 객체 네트워크를 가동하여 특정 유스케이스가 전 계층에서 잘 동작하는지 검증

## 단위 테스트로 도메인 엔티티 테스트하기

헥사고날 아키텍처의 중심인 도메인 엔티티를 단위 테스트 해보자

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/7849d321-71fc-4c0d-b22c-28c145f19e65)

출금 성공에 대한 단위 테스트이다.

**만들고 이해하기 쉽고 빠르게 실행되는 특징이 있다.**

→ 단위 테스트가 도메인 엔티티에 녹아있는 비즈니스 규칙을 검증하기에 가장 적절한 방법이다.

→ 도메인 엔티티가 다른 클래스에 의존하지 않기 때문에 다른 테스트는 불필요하다.

## 단위 테스트로 유스케이스 테스트하기

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/aa9ad0f4-5d93-4cc7-b624-b3ca5d2e924d)

Mockito 라이브러리를 활용하여 Mock 객체를 만들어 테스트를 진행하여 협력 객체와의 관계를 끊고 테스트하고자 하는 클래스만을 검증할 수 있다.

> Mockito 사용법
[https://jiwonchoi-study.notion.site/Mock-268ba2461f564253b4912bba3664d31d?pvs=4](https://www.notion.so/268ba2461f564253b4912bba3664d31d?pvs=21)
> 

## 통합 테스트로 웹 어댑터 테스트하기

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/a4769919-92d6-47fc-9e7e-56267373adcf)

웹 어댑터는 HTTP를 통해 입력을 받고, 입력에 대한 유효성 검증, 유스케이스에 맞는 데이터 포맷으로 매핑 등 작업을 수행하고 데이터를 유스케이스에 전달한다.

웹 어댑터 테스트의 경우 위 과정이 원활하게 이루어지는지 테스트해야 한다.

MockMvc를 통해 웹 어댑터를 테스트할 수 있다.

→ 웹 컨트롤러가 프레임워크에 강하게 묶여있어 프레임워크와 통합된 상태로 테스트

**MockMvc 테스트 방식 + MockMvc를 활용한 Spring Rest Docs 찾아보기**

## 통합 테스트로 영속성 어댑터 테스트하기

영속성 어댑터도 위의 웹 어댑터와 마찬가지로 특정 기술에 묶여있기 때문에 통합 테스트를 진행하는 것이 합리적이다.

→ **검증 대상: 어댑터의 로직 + 데이터베이스 매핑**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/743c8723-f00a-41f4-88e3-c9da0bac4a69)

`@DataJpaTest` 를 활용하여 Spring Data JPA에 필요한 의존성들만 가볍게 가져와서 테스트할 수 있다.

추가적으로 필요한 객체들은 `@Import` 를 통해 가져오면 된다.

`@Sql` 은 해당 테스트 실행 전 SQL을 실행시켜 데이터베이스를 특정 상황으로 만들어준다.

테스트에 특정 설정을 하지 않았다면 기본적으로 In-Memory H2 Database에 매핑이 된다.

→ 실제 DB, TestContainer 등 환경 변경 가능 (실제 개발환경에 맞는 테스트를 진행할 수 있다.)

## 시스템 테스트

피라미드의 최상단에 위치한 시스템 테스트는 전체 Application을 띄우고 API를 통해 요청을 보내고, 모든 계층이 조화롭게 잘 동작하는지 검증한다.

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class SendMoneySystemTest {

	@Autowired
	private TestRestTemplate restTemplate;

	@Autowired
	private LoadAccountPort loadAccountPort;

	@Test
	@Sql("SendMoneySystemTest.sql")
	void sendMoney() {

		Money initialSourceBalance = sourceAccount().calculateBalance();
		Money initialTargetBalance = targetAccount().calculateBalance();

		ResponseEntity response = whenSendMoney(
				sourceAccountId(),
				targetAccountId(),
				transferredAmount());

		then(response.getStatusCode())
				.isEqualTo(HttpStatus.OK);

		then(sourceAccount().calculateBalance())
				.isEqualTo(initialSourceBalance.minus(transferredAmount()));

		then(targetAccount().calculateBalance())
				.isEqualTo(initialTargetBalance.plus(transferredAmount()));

	}

	private Account sourceAccount() {
		return loadAccount(sourceAccountId());
	}

	private Account targetAccount() {
		return loadAccount(targetAccountId());
	}

	private Account loadAccount(AccountId accountId) {
		return loadAccountPort.loadAccount(
				accountId,
				LocalDateTime.now());
	}

	private ResponseEntity whenSendMoney(
			AccountId sourceAccountId,
			AccountId targetAccountId,
			Money amount) {
		HttpHeaders headers = new HttpHeaders();
		headers.add("Content-Type", "application/json");
		HttpEntity<Void> request = new HttpEntity<>(null, headers);

		return restTemplate.exchange(
				"/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
				HttpMethod.POST,
				request,
				Object.class,
				sourceAccountId.getValue(),
				targetAccountId.getValue(),
				amount.getAmount());
	}

	private Money transferredAmount() {
		return Money.of(500L);
	}

	private Money balanceOf(AccountId accountId) {
		Account account = loadAccountPort.loadAccount(accountId, LocalDateTime.now());
		return account.calculateBalance();
	}

	private AccountId sourceAccountId() {
		return new AccountId(1L);
	}

	private AccountId targetAccountId() {
		return new AccountId(2L);
	}

}
```

`@SpringBootTest` 는 스프링이 Application을 구성하는 모든 빈들을 띄운다.

메서드에서는 요청을 생성하여 Application에 보내고 응답 상태와 여러 필드를 검증

TestRestTemplate을 사용하여 실제 http 통신을 진행

**만약 서드파티 시스템이 필요한 경우 헥사고날 아키텍처를 잘 지켰다면 Mock 객체로 만들 수 있다.**

**테스트 가독성을 높이기 위해 Helper 메서드를 만들 수 있다.**

단위, 통합 테스트에서는 미처 발견하지 못하는 버그를 찾을 수 있는 테스트

→ **특히 여러 유스케이스가 결합된 시나리오에서 테스트 효과가 좋다**

## 적당한 테스트 범위

테스트 커버리지는 100%말고는 의미가 없고 100%라도 다른 부분에서 버그가 발생할 수 있다.

헥사고날 아키텍처의 테스트 전략

- 도메인 엔티티를 구현할 때는 단위 테스트로 커버하기
- 유스케이스를 구현할 때는 단위 테스트로 커버하기
- 어댑터를 구현할 때는 통합 테스트로 커버하기
- 사용자가 취하는 중요 애플리케이션 경로는 시스템 테스트로 커버하기
