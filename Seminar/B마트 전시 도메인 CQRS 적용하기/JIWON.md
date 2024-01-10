# B마트 전시 도메인 CQRS 적용하기

https://youtu.be/fg5xbs59Lro?si=x8MNVEsOX84Xl8Ow

## 데이터 구조 살펴보기

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/da0cc303-0c75-429b-844a-ec781439f446)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/0a04d785-3733-4c1f-82c9-2972d35f2fe1)

데이터 스키마는 정규화된 형태

전시 도메인은 데이터를 노출하기 위해 비정규화된 형태

## CQRS?

---

### Command and Query Responsibility Segregation

명령과 조회의 책임을 분리!

### 초기 아키텍처

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/3d3c19c9-af92-4811-bb9e-9ba96742abb1)

초기에는 단순한 아키텍처로 구성되어 명령과 조회가 분리되지 않음

**모델이 명령과 조회 모두 수행**

### 복잡해진 모델

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/751191cc-8072-4526-9ab8-17391738ce9c)

**명령과 관련된 로직과 조회와 관련된 로직이 같은 모델을 바라보게 된다.**

→ 서로 의존성을 가지고 영향을 미치게 됨

### CQRS!

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/f64f4c71-afaf-4246-9833-bf20c2946ee4)

큰 도메인 안에서 명령과 조회의 영역을 나누어 문제를 해결한다.

**CQRS 활용**

## CQRS 적용하기

---

### 모델 분리하기

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/92a8a9a8-32e5-49ad-8c22-ae5d6ba7e343)

교집합 지점인 모델을 분리해야 한다.

**명령 모델은 기존 도메인 모델, 조회 모델은 비정규화된 데이터를 재정의한 모델**

### 조회 모델 Dto 활용하기 (추천!)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/87925dfa-3279-456b-a22d-ef38fcf726ef)

**조회 모델의 데이터는 엔티티의 일부인 경우가 많으므로 Dto를 만드는 것을 추천!**

### 캐시 활용

조회 모델은 비정규화 과정이 많다.

→ 성능상 부담이 가게 됨

이런 이유로 중간 지점에 캐시를 활용하기도 함

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/b5d01699-8918-4e12-b730-19978152ef50)

조회 모델 생성 시점과 이용하는 시점이 서로 호출하는 구조

→ 매 번 생성한다면 성능 상 이슈 (Heap size, 트래픽 등)

### 조회 모델의 생성, 조회 시점 나누기

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/b6442460-839c-46d6-982a-cf4498c7f0c5)

명령 모델이 데이터를 변경시킬 때 비정규화 된 조회 모델을 생성하고 저장한다.

**CQRS는 DB로 데이터를 가져오는 과정에서 join과 같은 연산 작업을 극히 제한해야 한다.**

→ DB에서 바로 데이터를 가져와서 이용할 수 있게 제공해야 한다.

→ **조회를 위한 비정규화 된 테이블을 만들어줘야 한다.**

**json 포맷이 주로 사용되는 이유로 NoSQL을 쓰는 경우가 많다.**

캐시 이용이 아닌 근본적인 해결 가능!

**다만, 명령 모델 변경 시 조회 모델을 생성하고 저장하므로 적은 명령 많은 조회에서 큰 이점을 가져감**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/b21c8d35-8ff7-476e-bde0-959ea430011d)

생성 조회 시점을 분리할 수 있게 됨

1. 명령이 수행되는 시점에 필요한 데이터를 생성
2. 조회에서는 이미 만들어진 결과물을 가져다 쓰면 된다.

UX 변경이 잦고 비즈니스가 자주 변화하는 환경에서 이렇게 분리시키면 실수를 덜 할 수 있게 된다.

### 성능 더 높이기

데이터가 분리되어 존재

→ **서로 다른 데이터 저장소 사용 가능!**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/7059e035-4760-45fe-8c55-a65a738a144a)

Redis를 활용하고 DB에도 콜백 상황을 대비하여 저장

**두 저장소 간의 데이터 정합성에 신경써야 하는 단점이 있음**

### 책임과 성능에 대한 부담

조회 모델은 개선되었지만 명령 모델이 조회 모델을 생성하는 책임을 가지게되어 부담이 더 커졌다.

### 영역을 한 번 더 분리!

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/274c3ad9-1d43-459a-95e8-48d23c82f728)

**영역을 한 번 더 분리하고 이벤트 소싱을 활용하여 문제를 해결함!**

명령 모델이 이벤트를 던지고 조회 모델 생성 및 저장하는 부분에서 이벤트를 구독

이벤트를 통해 두 작업을 분리!

### 최종 결과

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/189571d5-9ed4-413f-8aa1-ce35b01937d1)

### 그래서 언제 CQRS를 도입할까?

- UX와 비즈니스 요구사항이 복잡할 때
- 조회 성능을 보다 높이고 싶을 때
- 데이터를 관리하는 영역과 이를 뷰로 전달하는 영역의 책임이 나뉘어져야 할 때
- 시스템 확장성을 높이고 싶을 때

## B마트가 어떻게 CQRS를 적용했나

---

### 고객에게 카탈로그와 상품을 전시한다

전시 도메인에 CQRS를 적용함

필요한 일부 도메인에만 CQRS 도입 가능

### 메뉴 화면

가장 복잡한 비정규화 로직을 가지는 부분

요구 사항

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/fa83fe5e-5fee-4fc0-b158-ea35e3b6eab0)

요구사항을 바탕으로 모든 카탈로그에 대한 전체 트리 로직이 생기게 되었다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/d0c9555e-7ce0-41ed-bb27-dd18c7ffcbfa)

일종의 거대한 조회 모델

### 발전하는 서비스

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/8c7c8adb-ef05-4744-a79c-9d6eda2eef7b)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/7ef1cb46-966b-48d8-afa7-7f48903851e0)

전체 트리 모델이 거대해짐

경계를 넘나드는 코드가 생겨 스파게티 코드가 발생함

전체 트리 모델의 개수는 지점 수 * 카탈로그 수 * 상품 수에 비례하여 점점 커졌고 이로 인해 레디스에 부하가 점점증가함

### 나뉘게 된 팀

팀이 조회랑 명령으로 나뉘게 됨

→ CQRS를 도입하게 될 타이밍!

## 조회 모델 설계하기!

---

전체 트리 모델을 박살내고 조회 모델을 설계를 시작함

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/46a73e0f-6343-40fe-8f69-179e2de98985)

전체 트리 모델을 분리하면서 필요한 데이터 목록을 찾게 됨

→ 핵심 도메인 모델

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/b05452e1-8945-4fe5-aa10-008e1f04b79c)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/a2bb6014-02c6-461f-8475-d582e5cce5df)

## 감시 대상과 이벤트

---

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/0fbf8217-e577-4530-a4af-ad17fb8236f5)

이벤트 발행 방법

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/a987c219-ccd5-44b1-80d2-c7120d89522f)

id만 존재하면 Database에서 최신 데이터를 꺼내올 수 있기 때문에 entity의 id만 전달하는 형태로 설계

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/dbd32829-688e-44e7-a76c-3bc768082c58)

## 데이터 변경 감지하기

---

변경 감지 방법

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/fc2b64b7-c725-4bc2-af4d-f859e7875f40)

### JPA EntityListeners

spring jpa domain event를 활용하는 방법

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/5ea4b9e4-36f1-405e-a1d4-9707bb3f5616)

가장 추상화되어있고 가장 쓰기 간편함

엔티티가 어떤 값에서 어떤 값으로 변경되었는지 추적하기는 어려움

### 하이버네이트 EventListener

하이버네이트에서 제공하는 EventListener

Spring Boot Starter JPA의 구현체가 하이버네이트라 바로 사용 가능

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/8c953dea-08b2-4cba-b739-ab4bf8c25e97)

### 하이버네이트 인터셉터

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/3386732b-faa0-4c09-b80a-26c77d49ff62)

### Spring AOP

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/2adbf58c-facc-41d5-81b0-d3597ac850e0)

최종적으로 하이버네이트 이벤트리스너, Spring AOP 활용

엔티티 관리 쿼리 → 하이버네이트 이벤트리스너

생쿼리 → Spring AOP

## 이벤트 발행

---

Amazon SNS, SQS 활용
