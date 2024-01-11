# ****ㄷㄷㄷ: Domain Driven Design과 적용 사례공유****

https://youtu.be/4QHvTeeTsj0?si=Q9haoaFjrYeXH4bv

## Domain Driven Design?

Monolithic 구조의 레거시 프로젝트를 MSA 프로젝트로 전환하면서 Domain을 중심으로 MSA를 설계

DDD의 정의

기능적인 문제의 영역을 정의하는 도메인과 그 도메인을 사용하는 비즈니스 로직을 중심으로 설계하는 방식

DDD의 특징

- Data 중심이 아닌 도메인의 모델과 로직에 집중
- 유비쿼터스 언어 → 보편화 된 언어를 사용 → 업무 용어를 통일하여 커뮤니케이션 (전체적인 팀에서 업무 용어 통일)
- Software Entity와 Domain간 개념의 일치
    - 도메인 모델부터 코드까지 함께 움직이는 모델을 지향

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/e46d0673-201c-4253-9853-67e27213b4c7)

파트너 사이트 → 큰 틀과 검증된 플로우 존재 → 필요한 기능들을 구별하고 필터링하는 과정이 핵심이라 DDD 선택

DDD 적용에 필요한 것들

- Bounded Context
    - 범위를 구분해놓은 하위 도메인 개념
- Context Map
    - 바운디드 컨텍스트 간의 관계를 보여줌
- Aggregate
    - 데이터 변경 단위 (라이프 사이클이 같은 도메인들의 집합)

Bounded Context의 구성

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/43b0d856-038c-4b69-a5fb-07b7c4709cbf)

Context Map의 구성

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/755680e1-bc05-4fcc-8948-d84ce93bb427)

Aggregate의 구성

서비스들이 가진 객체들을 그대로 사용하는건 의미가 없어 각 도메인 영역을 대표하는 Aggregate 설정

Aggregate의 특징

- 데이터의 변경 단위
- Aggregate의 접근은 Root Entity를 통해서만 가능하다.
- Aggregate를 통해 객체들 사이의 관계를 넓은 시야로 바라볼 수 있게 해준다.
- 제약 사항들을 하나의 맥락으로 관리하고, 비즈니스 로직에 집중할 수 있다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/76be5203-8691-4465-a489-5f7c93aa83d0)

## 아키텍처

### DDD의 대표적인 아키텍처

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/ce995280-de71-4eb5-a44a-72442f473e88)

레이어드 아키텍처

- User Interface (UI)
- Application
- Domain
- Infrastructure

클린 아키텍처

- External Interface
- Interface Adapter
- Use Case
- Entity

헥사고날 아키텍처

- 클린 아키텍처와 비슷하지만 port 개념이 존재
- Port And Adpter Pattern

### 헥사고날 아키텍처 적용

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/472e2ea5-918e-4c96-93ec-81378ca07c41)

### 단점

1. MSA에서 오는 단점 그대로 가져옴
2. 아키텍처 구현에 생성되는 코드가 상당히 많다
3. 각 도메인에 대한 높은 이해도 필요

### 장점

1. 새로운 기능 및 요구사항에서 유연함
2. Aggregate로 인한 캡슐화
3. Loose coupling, High cohesion (낮은 결합도, 높은 응집도)
4. 도메인 로직 분리로 비즈니스 로직에 집중 가능
5. 코드 가독성이 좋아짐
