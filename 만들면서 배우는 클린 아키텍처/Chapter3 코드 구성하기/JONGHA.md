# 계층으로 구성하기

```jsx
buckapl
|--- domain
|    |----- Account
|    |----- Activity
|    |----- AccountRepository
|    |----- AccountService
|--- persistence
|    |----- AccountRepositoryImpl
|--- web
|    |----- AccountController
```

기능을 구분짓는 패키지 경계가 없다. 

애플리케이션이 어떤 유스 케이스를 제공하는지 파악하기 어렵

패키지 구조를 통해서는 우리가 목표로 하는 아키텍처를 파악할 수 없다

# 기능으로 구성하기

```jsx
buckapl
|--- account
|    |----- Account
|    |----- Activity
|    |----- AccountRepository
|    |----- SendMoneyService
|    |----- AccountRepositoryImpl
|    |----- AccountController
```

pakage-private 접근 수준과 결합하여 각 기능 사이의 불필요한 의존성을 방지할 수 있다. 

<aside>
💡 소리치는 아키텍처 : 애플리케이션의 기능을 코드를 통해 볼 수 있게 만드는 것

</aside>

문제점

- 패키지명이 없고, 인커밍, 아웃고잉 포트를 확인할 수 없다.
- 도메인 코드가 실수로 영속성 코드에 의존하는 것을 막을 수 없다.

# 아키텍쳐 적으로 표현력 있는 구조

```jsx
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

핵심적인 요소는 엔티티, 유스케이스, 인커밍/아웃고잉 포트, 인커밍/아웃고잉 어댑터이다. 

domain : 도메인 모델이 속함

application : 도메인 모델을 둘러싼 서비스 계층을 포함

- LoadAccountPort, UpdateAccountStatePort : 아웃고잉 포트 인터 페이스.

adapter : 인커밍 포트를 호출하는 인커밍 어댑터와 애플리케이션 계층의 아웃고잉 포트에 대한 구현을 제공하는 아웃고잉 어댑터 포함. 

- adapter : **package-private 접근 수준으로 놔도 됨**. 애플리케이션에서 어댑터 클래스로 향하는 의존성이 없다.
- application, domain : 도메인 클래스는 public. 서비스는 인커밍 포트 인터페이스 뒤에 숨겨질 수 있기 떄문에 public 일 필요가 없다.
- 어댑터를 다른 구현으로 바꿀때 교체가 빠르다.
- DDD 개념을 직접적으로 대응시킬 수 있다. 상위레벨 패키지(account)는 다른 바운디드 컨텍스트와 통신할 전용진입점이자, 출구를 포함하는 바운디드 컨텍스트

# 의존성 주입

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2f5d3563-032e-4afd-b6b2-5dd0febd696d)

- 인커밍과 아웃 고잉 어댑터에 의존성을 갖지 않게 한다.
    - 인커밍은 제어흐름 방향이 어댑터와 도메인 코드간의 의존성 방향과 같아서 하기 쉽다. 즉, 어댑터가 애플리케이션 계층의 서비스를 호출하는 것.
    - 아웃 고잉은 의존성 역전 원칙을 이용해야 한다. 애플리케이션에 인터페이스를 두고, 어댑터 계층해서 이를 구현한다.
    
    # 유지보수 가능한 소프트웨어 만드는데 어떻게 도움?
    
    육각형 아키텍처 패키지 구조를 통해, 의사소통, 개발, 유지보수가 편해진다.
