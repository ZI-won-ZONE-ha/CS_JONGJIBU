# 영속성 어댑터 구현하기

---

영속성 계층을 애플리케이션 계층의 플러그인으로 만드는 방법을 알아보자

## 의존성 역전

---

애플리케이션 서비스에 영속성 기능을 제공하는 영속성 어댑터에 대해 알아보자

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/ddc7b57a-832f-449f-b783-922daa54af16)

**Application 서비스에서 영속성 기능을 사용하기 위해 포트 인터페이스를 호출한다.**

포트가 실제로 영속성 작업을 수행하고 DB와 통신할 책임을 가진 영속성 어댑터 클래스에 의해 구현된다.

**영속성 어댑터는 아웃고잉 어댑터이다. (서비스 → 외부)**

→ Application에 의해 호출될 뿐, Application을 호출하지 않기 때문에

포트는 다른 포트와 마찬가지로 간접적인 계층이다.

→ 영속성 문제에 신경 쓰지 않고 도메인을 개발하기 위해 만든 계층 (**영속성에 의존하지 않게 하기 위함**)

의존성은 애플리케이션 코어 → 영속성 어댑터로 향한다.

## 영속성 어댑터의 책임

---

- 입력을 받는다.
    - 포트 인터페이스를 통해 입력을 받는다.
    - 입력 모델은 인터페이스가 지정한 도메인 엔티티 또는 특정 데이터베이스 연산 전용 객체
- 입력을 DB 포맷으로 매핑한다.
    - JPA의 경우는 도메인 엔티티 → JPA 엔티티 객체로 매핑
    - 8장 매핑하지 않는 전략도 있으니 참고
- 입력을 DB로 보낸다.
- DB 출력을 Application 포맷으로 매핑한다.
- 출력을 반환한다.

**입력 모델, 출력 모델이 영속성 어댑터가 아닌 애플리케이션 코어에 있는 것이 중요하다!**

→ 영속성 어댑터 내부를 변경해도 코어에 영향을 주지 않게 됨

## 포트 인터페이스 나누기

---

포트 인터페이스를 나누는 방법에 대해 알아보자

일반적으로 특정 엔티티가 필요로 하는 DB 연산을 하나의 리포지토리 인터페이스에 넣어두는 방법이 있다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/1c0b6679-63ac-4123-9dc6-2246f8aa8c50)

**해당 방식으로 구현하게 되면 넓은 포트 인터페이스에 의존을 가져 코드에 불필요한 의존성이 발생한다.**

→ 코드를 이해하고 테스트하기 어려워진다.

**인터페이스 분리 원칙을 적용하여 클라이언트가 자신이 필요로하는 메서드만 알면 되도록 인터페이스를 분리해야 한다!!!!**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/d992eca8-db11-49c4-a2b2-25da410ff0b7)

**각 서비스가 필요한 메서드에만 의존할 수 있게 설계해야 한다!**

## 영속성 어댑터 나누기

---

위의 그림은 영속성 어댑터를 1개로 뒀지만 여러개로 만들어도 상관없다.

영속성 연산이 필요한 도메인 클래스(또는 애그리거트) 하나당 하나의 어댑터를 생성해도 된다.

**이렇게 설계하면 영속성 어댑터들은 각 영속성 기능을 이용하는 도메인 경계를 따라 자동으로 나뉜다.**

JPA와 JDBC 포트를 각각만들고 각 어댑터를 생성해주는 것이 하나의 예시

**포트가 구현만 된다면 영속성 어댑터는 어떤 규칙을 사용해도 상관없다.**

→ 애그리거트 당 하나의 영속성 어탭터와 같은 규칙 적용 가능

책 70p account, billing 바운디드 컨텍스트에 따라 나뉜 영속성 어댑터 확인하기

## Spring Data JPA 예제

---

- AccountJpaEntity

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/754ef93c-47e6-4dfa-a381-cbef5b6baf9c)

Account Domain에 매핑되는 JPA Entity 클래스이다.

현재는 id 필드만 존재하지만 추후 user id와 같은 필드가 추가된다.

그리고 ActivityJpaEntity와는 연관 관계 매핑을 굳이 하지 않음 → 이슈 발생 고려

- ActivityJpaEntity

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/e010872a-7589-401b-b887-e9319c725289)

activity 테이블을 표현하기 위한 JPA Entity 클래스이다.

- AccountRepository

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/f014b0ac-7b53-4cda-80b8-1158f6c49ba9)

쿼리를 제공해준다.

- ActivityRepository

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/81ff84f1-7c12-4b42-b976-fd1965a4b0ec)

쿼리를 제공해준다.

- AccountPersistenceAdapter

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/f48fbbb8-74b4-4010-ae00-7d36bad62021)

영속성 어댑터는 Application에 필요한 포트를 구현한다.

책 77p 영속성 어댑터 로직 확인

## 데이터베이스 트랜잭션 처리 방법

---

1. Service 클래스에 Transactional Annotation 붙이기
    1. 구현이 간편한 대신 도메인 로직이 오염되는 단점 존재
2. AOP로 위빙하기
    1. 구현이 복잡한 대신 도메인 로직이 오염되지 않음

두 방법 중 선택해서 트랜잭션을 처리하면 된다.

## 유지보수 가능한 소프트웨어가 되는 이유

---

도메인 로직이 영속성과 관련된 것들로부터 분리되어 풍부한 도메인 모델을 만들 수 있다.

포트 인터페이스를 좁게 설정하여 다른 방식으로 구현할 수 있는 가능성을 만들어준다.
