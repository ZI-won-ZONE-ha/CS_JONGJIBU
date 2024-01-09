규모가 커질 수록 아키텍처가 서서히 무너지게 된다. 계층 간의 경계가 약화되고, 코드는 테스트하기 어려워진다. 새로운 기능을 구현하는데 점점 더 많은 시간이 든다. 

# 경계와 의존성

**경계를 강제한다는 것은 의존성이 올바른 방향을 향하도록 강제하는 것이다.** 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/9eef1fcb-14c9-4a5e-a830-eacd317672fe)

계층 경계를 넘는 의존성은 항상 안쪽 방향으로 향해야 한다. 

따라서 이러한 의존성 규칙을 강제하는 방법을 알아보자!

# 방법 1 : 접근 제한자.

public, protected, private, default(package-private) 제한자가 존재한다. 

**여기서 default 제한자가 중요하다.** 

왜냐하면 자바 패키지를 통해, 클래스들을 응집적인 모듈로 만들어 주기 때문에, 모듈 내에 있는 클래스들은 서로 접근 가능 하지만, 패키지 바깥에서는 접근할 수 없다. 

다른 패키지가 접근하려면, 모듈의 진입점으로 활용될 클래스를 public으로 만들어 주면된다.!!!! 

기존의 코드 구성을 봐보자 

```java
buckapl
|--- account
|    |----- adapter
|    |      |----- in
|    |      |      |---- web
|    |      |      |     |---- AccountController
|    |      |----- out
|    |      |      |---- persistence
|    |      |      |     |---- AccountPersistenceAdapter
|    |      |      |     |---- SpringDataAccountRepository
|    |---- domain
|    |     |----- Account
|    |     |----- Activity
|    |---- application
|    |     |----- SendMoneyService
|    |     |----- port
|    |     |     |---- in
|    |     |     |     |---- SendMoneyUseCase
|    |     |     |---- out
|    |     |     |     |---- LoadAccountPort
|    |     |     |     |---- UpdateAccountStatePort
```

persistence 패키지는 외부에서 접근할 이유가 없기 때문에 default로 둬도 된다. 

또한 SendMoneyService 도 마찬가지로 외부에서 접근할 이유가 없기 때문에 default로 둔다. 

의존성 주입 메커니즘은 리플랙션으로 주입되기 때문에, default 여도 상관없다. 

하지만 이 방법은 클래스 패스 스캐닝을 이용할 때의 이야기 이고, 만약, 다른 방법(bean 주입) 이라면, public 제한자를 이용하여야 한다. 

나머지 클래스들 (domain, application 계층..) 은 public으로 정의되어 있어야 한다. 

하지만 단점으로, 규모가 커지면 혼란을 겪을 수 있다. 리팩터링을 하면서, 하위 패키지를 만들 수 있는데, 하위패키지는 다른 패키지로 취급되기 때문에 하위 패키지의 package-private  멤버에 접근할 수 없다. 이를 통해 public으로 바꾸어주면 또 의존성 규칙이 깨질 수 있다. 

# 방법 2: 컴파일 후 체크

코드가 컴파일 된 후에 런타임에 체크한다는 것이다. 이는, 지속적 통합 빌드환경에서 자동화된 테스트 과정에서 잘 작동된다. 

이에 대한 도구로 ArchUnit이 있다. 이는, 의존성 방향이 기대한 대로 잘 설정돼 있는지 체크할 수 있는 API이다. 의존성 방향을 위반할 시, 예외를 던지고, 테스트를 실패시킨다. 

이를 잘 이용하면, 육각형 아키텍쳐 내에서 관련된 모든 패키지를 명시할 수 있는 일종의 도메인 특화 언어를 만들 수 있고, 패키지 사이의 의존성 방향이 올바른지 자동으로 체크할 수 있다. 

```java
@Test
	void validateRegistrationContextArchitecture() {
		HexagonalArchitecture.boundedContext("io.reflectoring.buckpal.account")

				.withDomainLayer("domain")

				.withAdaptersLayer("adapter")
				.incoming("in.web")
				.outgoing("out.persistence")
				.and()

				.withApplicationLayer("application")
				.services("service")
				.incomingPorts("port.in")
				.outgoingPorts("port.out")
				.and()

				.withConfiguration("configuration")
				.check(new ClassFileImporter()
						.importPackages("io.reflectoring.buckpal.."));
	}
```

바운디드 컨텍스트의 부모 패키지를 지정한 후, 하위패키지를 지정하고, check 함수를 통해 패키지 의존성이 의존성 규칙을 따라 유효하게 설정됐는지 검증할 수 있다. 

하지만, 이에 대한 단점으로는 만약 타이핑 에러로 buckpal 이 아니라 bucpal 과 같이 작성되었다면, 테스트가 어떤 클래스도 찾지 못하여 테스트 자체가 무의미해진다. 따라서 리팩터링 할 시에 만약 패키지명이 바뀌면, 이에 대한 방안으로 클래스를 못찾았을 때 실패하는 테스트를 추가하여야 한다.

따라서 이 방법은 언제나 코드와 함께 유지 보수하여야 하는 대상이다. 

# 방법 3: 빌드 아티팩트

빌드 아티팩트는 빌드 프로세스의 결과물이다. 대표적으로 자바에서는 메이븐과 그레이들이 있다. 

빌드 도구의 주요 기능 중 하나는 의존성 해결이다. 이를 활용해서 모듈과 아키텍처의 계층 간의 의존성을 강제할 수 있다. 

**잘못된 의존성을 막기 위해 다음과 가이 아키텍처를 여러개의 빌드 아티팩트로 만들 수 있다.**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/09cbbb1e-d66b-40de-b91a-58b2f2f0214e)

- 모듈을 더 세분화 할 수록, 모듈 간 의존성을 더 잘 제어할 수 있다. 더 작게 분리할 수록, 모듈 간에 매핑을 더 많이 수행해야 한다.

### 패키지로 구분하는 것에 비해 빌드 모듈로 구분하는 것의 장점

- 빌드 도구가 순환 의존성을 허용하지 않는다.
- 다른 모듈을 고려하지 않고, 특정 모듈의 코드를 격리한 채로 변경할 수 있다. 여러개의 빌드 모듈은 각 모듈을 격리한 채로 변경할 수 있게 해준다.
- 모듈 간 의존성이 빌드 스크립트에 분명하게 선언돼 있기 때문에 새로 의존성을 추가하는 일은 우연이 아닌 의식적인 행동이 된다. 따라서 이 의존성이 필요한지 한번 더 고민하게 된다.

### 단점

단점으로는 빌드 스크립트를 유지보수하여야 한다. → 이는, 여러 개의 멀티 모듈로 나누기 전에 아키텍쳐가 안정되어야 한다.
