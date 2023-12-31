# Chapter 9. 도메인 모델과 바운디드 컨텍스트

---

## 도메인 모델과 경계

---

도메인을 완벽하게 표현하는 단일 모델을 만드려고 하면 안 된다.

→ 하나의 도메인은 다시 여러 하위 도메인으로 구분되기 때문에

하위 도메인에 따라 다른 용어를 사용하는 경우도 있다.

**하위 도메인마다 같은 용어라도 의미가 다르고 같은 대상이라도 지칭하는 용어가 다를 수 있다.**

**올바른 도메인 모델을 개발하려면 하위 도메인마다 모델을 만들어야 한다!**

모델은 특정한 컨텍스트 하에서 완전한 의미를 가진다.

→ 바운디드 컨텍스트

## 바운디드 컨텍스트

---

바운디드 컨텍스트는 모델의 경계를 결정한다.

한 개의 바운디드 컨텍스트에는 논리적으로 한 개의 모델을 가진다.

용어를 기준으로 구분함

**이상적으로는 하위 도메인과 바운디드 컨텍스트가 일대일 관계를 가지면 좋지만 그렇지 못 한 경우가 발생한다.**

→ 개발 상황에 따라 지킬 수 없는 경우가 있다.

**여러 하위 도메인을 하나의 바운디드 컨텍스트에서 개발할 때 주의할 점은 하위 도메인의 모델이 섞이지 않도록 해야 한다!**

→ 하위 도메인별로 기능을 확장하기 어려워진다.

바운디드 컨텍스트는 도메인 모델을 구분하는 경계가 된다.

→ 구현하는 하위 도메인에 맞는 알맞은 모델 포함

## 바운디드 컨텍스트 구현

---

바운디드 컨텍스트는 도메인 모델만 포함하지 않고 도메인 기능을 제공하는데 필요한 표현, 응용 서비스, 인프라 영역을 모두 포함한다.

도메인 모델의 데이터 구조가 바뀌면 DB 스키마도 바뀌기 때문에 DB Table도 바운디드 컨텍스트에 포함

표현 영역은 사용자에게 HTML 페이지를 줄 수 있고 다른 바운디드 컨텍스트를 위해 REST API를 제공할 수도 있다.

모든 바운디드 컨텍스트를 DDD 방식으로 개발할 필요는 없다.

→ 로직이 간단하다면 CRUD 방식으로 개발해도 된다. (유지보수가 쉽기 때문에)

한 바운디드 컨텍스트에서 두 방식을 혼합해도 된다.

→ CQRS 패턴 (명령 기능, 조회 쿼리 기능을 위한 모델을 구분하는 패턴)

## 바운디드 컨텍스트 간 통합

---

관련된 바운디드 컨텍스트를 개발하면 자연스럽게 바운디드 컨텍스트 간 통합이 발생한다.

→ 통합이 필요한 기능이 생기게 됨

카탈로그 시스템은 추천 시스템으로 부터 추천 데이터를 받아온다.

→ 이 때 카탈로그 컨텍스트와 추천 컨텍스트의 도메인 모델은 서로 다르다.

추천 시스템: 상품의 정보는 포함하지 않고 아이템 ID로 상품 식별하고 추천 연산을 진행

카탈로그 시스템: 제품을 중심으로 도메인 모델을 구현

결론적으로 **카탈로그 시스템은 추천의 도메인 모델을 사용하지 않고 카탈로그 도메인 모델을 통해 추천 상품을 표현한다.**

즉 아래과 같은 형태가 나와야 한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/9bc0127c-3d3a-4c51-8129-a419fa75e05c)

**추천 시스템의 추천 상품은 카탈로그 도메인과 다른 형태의 모델이기 때문에 이를 카탈로그 도메인 모델 형태로 맞춰주는 작업이 필요하다!**

→ 변환 과정 필요! 이 때 변환 과정이 복잡하면 별도의 클래스로 분리해도 된다. (Mapper, Translator)

직접 통합하는 방법: REST API

간접적으로 통합하는 방법: 메시지 큐 (Kafka, RabbitMQ)

**메시지 큐를 활용한다면 카탈로그 시스템은 유저의 활동 이력을 메시지 큐에 보내고 추천 시스템은 이 메시지를 읽으면 된다.**

→ **메시지의 형식을 두 시스템이 맞춰야 한다. (큐를 누가 제공하느냐에 따라 형식을 맞춰주면 된다.)**

## 바운디드 컨텍스트 간 관계

---

바운디드 컨텍스트는 어떤 식으로든 연결되기 때문에 두 바운디드 컨텍스트는 다양한 방식으로 관계를 맺는다.

가장 흔한 관계는 REST API 형식이다.

이 관계에서 **REST API를 사용하는 바운디드 컨텍스트(하류)는 REST API를 제공하는 바운디드 컨텍스트(상류)에 의존하게 된다.**

→ 제공되는 REST API의 스펙이 변경되면 이를 사용하는 쪽에도 변경이 일어난다.

→ 서로 일정에 차질이 없도록 이 부분에 대한 합의가 잘 되야 한다.

**만약 하류 팀이 다수 존재한다면 상류 팀은 여러 하류 팀의 요구사항을 수용할 수 있는 API를 만들고 이를 제공할 수 있다. (공개 호스트 서비스)**

**상류 컴포넌트의 서비스는 상류 바운디드 컨텍스트의 도메인 모델을 따르기 때문에 하류 컴포넌트는 상류 서비스의 모델이 자신의 도메인 모델에 영향을 주지 않도록 보호해 주는 완충 지대를 만들어야 한다!**

→ infra 영역과 인터페이스를 활용하면 된다.

독립 방식은 서로 통합하지 않는 방식이다. → 통합을 하기 위해서는 수동으로 해야 됨

수동 통합은 한계가 있다.

## 컨텍스트 맵

---

개발 바운디드 컨텍스트에 매몰되면 전체를 보지 못할 수 있다.

이 때 필요한 것이 컨텍스트 맵이다.

**컨텍스트 맵은 바운디드 컨텍스트 간 관계를 표현해준다.**

→ 시스템 전반을 이해하는데 큰 도움을 준다.
