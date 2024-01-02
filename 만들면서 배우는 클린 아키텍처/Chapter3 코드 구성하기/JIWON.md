# 코드 구성하기

패키지 구성 방법에 대해 알아보자

## 계층으로 구성하기

```
buckapl
|-- domain
|    |----- Account
|    |----- Activity
|    |----- AccountRepository
|    |----- AccountService
|-- persistence
|    |----- AccountRepositoryImpl
|-- web
     |----- AccountController
```

최적의 패키지 구조가 아닌 이유

1. Application의 기능 조각이나 특성을 구분 짓는 패키지 경계가 없음
2. Application이 어떤 유스케이스들을 제공하는지 파악할 수 없다.
3. 패키지 구조를 통해 우리가 목표로하는 아키텍처를 파악할 수 없다.

## 기능으로 구성하기

```
buckpal
|-- account
    |---- Account
    |---- AccountController
    |---- AccountRepository
    |---- AccountRepositoryImpl
    |---- SendMoneyService
```

기능별로 패키지를 구성하여 패키지 간 경계를 강화할 수 있다.

하지만 아키텍처 가시성이 상당히 안좋아지는 문제점이 있다.

## 아키텍처적으로 표현력 있는 패키지 구조

```
buckpal
|-- account
    |---- adapter
    |   |---- in
    |   |   |---- web
    |   |       |---- AccountController
    |   |---- out
    |       |---- persistence
    |           |---- AccountPersistenceAdapter
    |           |---- SpringDataAccountRepository
    |---- domain
    |   |---- Account
    |   |---- Activity
    |---- application
        |---- SendMoneyService
        |---- port
            |---- in
            |   |---- SendMoneyUseCase
            |---- out
                |---- LoadAccountPort
                |---- UpdateAccountStatePort
```

헥사고날 아키텍처에서 구조적으로 핵심적인 요소는 **엔티티, 유스케이스, 인커밍/아웃고잉 포트, 인커밍/아웃고잉 어댑터**이다.

해당 요소들을 바탕으로 application의 아키텍처를 표현하는 패키지 구조로 구성할 수 있다.

domain: 도메인 모델이 속한 패키지

application: 도메인 모델을 둘러싼 서비스 계층을 포함한다.

`SendMoneyService`는 `SendMoneyUseCase`를 구현하고, 영속성 어댑터에 의해 구현된 `LoadAccountPort` , `UpdateAccountStatePort` 를 사용한다.

adapter: application 계층의 인커밍 포트를 호출하는 인커밍 어댑터와 아웃고잉 포트에 대한 구현을 제공하는 아웃고잉 어댑터를 포함한다.

**이러한 패키지 구조는 아키텍처-코드 갭을 효과적으로 다룰 수 있는 강력한 요소이다.**

## 의존성 주입의 역할

클린 아키텍처의 본질적인 요건: **Application 계층이 인커밍/아웃고잉 어댑터에 의존성을 가지면 안된다.**

웹 어댑터(Controller)와 같이 인커밍 어댑터에 대해서는 적용이 쉽다.

→ 제어의 흐름 방향이 어댑터와 도메인 코드 간의 의존성 방향과 같은 방향이기 때문에

영속성 어댑터(JPA)와 같이 **아웃고잉 어댑터에 대해서는 제어 흐름의 반대 방향으로 의존성을 돌리기 위해 의존성 역전 원칙을 사용해야 한다.**

→ Application 계층에 인터페이스를 만들고(LoadAccountPort) 어댑터에 해당 인터페이스를 구현한 클래스(AccountPersistenceAdapter)를 두면 된다.

실제 객체를 주입하기 위해 의존성 주입이 필요함 → 중립적인 컴포넌트를 도입하여 주입 (또는 Spring level로 해결)

→ 해당 컴포넌트는 아키텍처를 구성하는 대부분의 클래스를 초기화하는 역할을 수행

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/7b4c309b-0af4-47b8-ba93-e5b854b81d8a)

이미지 출처: [https://velog.io/@yhlee9753/만들면서-배우는-클린-아키텍처-3.-코드-구성하기](https://velog.io/@yhlee9753/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%EB%B0%B0%EC%9A%B0%EB%8A%94-%ED%81%B4%EB%A6%B0-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-3.-%EC%BD%94%EB%93%9C-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0)
