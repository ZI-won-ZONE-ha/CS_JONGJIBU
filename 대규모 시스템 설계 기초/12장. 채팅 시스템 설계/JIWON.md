# 12장. 채팅 시스템 설계

---

## 문제 이해 및 설계 범위 확정

---

원하는 채팅 앱이 어떤 형식인지 잘 파악해야 한다.

- 카톡과 같은 1:1, 슬랙과 같은 업무용, 디스코드 같은 음성 채팅
- 모바일 앱, 웹 앱
- 트래픽 규모 (DAU)
- 그룹 채팅의 경우 인원 제한이 존재하는지
- 중요 기능 (파일 첨부, 그룹 채팅, 접속 상태 등)
- 메시지 길이 제한
- end to end 암호화 하는지
- 채팅 이력의 저장 기간

페이스북 메신저와 유사한 앱을 만든다고 가정해보자

- 응답 지연이 낮은 일대일 채팅
- 최대 100명까지 참여 가능한 그룹 채팅
- 사용자의 접속 상태 확인 가능
- 다양한 단말 지원
- 푸시 알림

## 개략적 설계안 제시 및 동의 구하기

---

클라이언트: 모바일 앱 또는 웹 애플리케이션, 클라이언트끼리 통신하지 않음 → 채팅 서비스와 통신함

채팅 서비스가 제공하는 기능

- 클라이언트들로부터 메시지 수신
- 메시지 수신자 결정 및 전달
- 수신자가 접속 상태가 아닌 경우 접속할 때까지 메시지 저장

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/ebd27712-7ba6-444c-8bc0-e75e0de38975)

클라이언트는 네트워크 통신 프로토콜을 사용하여 서비스에 접속

→ 채팅 서비스의 경우 어떤 통신 프로토콜을 사용할지 결정해야 된다.

HTTP 프로토콜을 사용한다고 하자

**이 때 HTTP의 연결 지속을 위한 keep-alive 헤더를 사용하면 좋다**

→ 클라이언트와 서버 사이의 연결을 끊지 않고 유지할 수 있기 때문에

메시지 수신 시나리오는 복잡함

→ HTTP는 클라이언트 to 서버 통신이기 때문에 서버 to 클라이언트로 사용할 수 없음

→ 폴링, 롱 폴링, 웹 소캣 등을 활용하여 이러한 문제 해결

### 폴링

폴링은 클라이언트가 주기적으로 서버에게 새 메시지가 있냐고 물어보는 방법

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/af313225-ec10-4132-918f-1b0ff80edbae)

**폴링을 자주하게 되면 비용이 증가하게 된다. + 메시지가 없는 경우 불필요한 서버 자원 소모**

### 롱 폴링

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/101f7d51-7eef-48ce-8d9d-098700d160ef)

폴링의 단점을 보완하기 위해 등장함

클라이언트가 새 메시지를 받거나 또는 타임아웃이 될 때까지 연결을 유지하는 방법이다.

메시지를 받거나 타임아웃되면 연결이 종료되고 서버에게 새로운 요청을 보내 다시 통신을 시작함

단점

- 메시지를 보내는 클라이언트와 수신하는 클라이언트가 다른 채팅 서버에 있을 수 있다.
- 서버 입장에서 클라이언트가 연결을 해제했는지 아닌지 알 방법이 딱히 없다.
- 여전히 비효율적이다.

### 웹소캣

서버가 클라이언트에게 비동기 메시지를 보낼 때 널리 사용하는 기술

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/bcd9e32b-8b53-436d-ae68-3ab106669d59)

웹 소캣 연결은 클라이언트가 시작한다. 한 번 맺어진 연결은 항구적이며 양방향이다.

처음에는 HTTP 연결이지만 특정 핸드셰이크 절차를 거치면 웹소캣 연결로 업그레이드 된다.

웹소캣 연결이 되면 서버는 클라이언트에게 비동기적으로 메시지를 전송할 수 있다.

웹소캣을 활용하게 되면 메시지를 주고 받을 때 동일한 프로토콜을 사용할 수 있어 설계 구현이 쉽고 직관적이다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/db01a401-e76c-4c3d-bf58-ce988bb996f8)

### 개략적 설계안

클라이언트 - 서버 사이의 주 통신 프로토콜은 웹소캣

다른 로그인, 회원가입과 같은 부분은 HTTP로 구현해도 무방

**채팅 서비스는 무상태 서비스, 상태유지 서비스, 제3자 서비스 세 부분으로 나누어 볼 수 있다.**

**무상태 서비스**

로그인, 회원가입, 사용자 프로필 등을 처리하는 전통적인 요청/응답 서비스

로드밸런서 뒤에 위치한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/ee50f38b-07a8-4d9e-8b1e-ce590ae52d69)

서비스 디스커버리 서비스는 클라이언트가 접속할 채팅 서버의 DNS 호스트 명을 클라이언트에게 알려주는 역할 수행

**상태 유지 서비스**

채팅 서비스의 경우 상태 유지가 필요함

클라이언트 - 채팅 서버 사이의 독립적인 네트워크 연결을 유지해야 된다.

클라이언트는 보통 서버가 살아 있는 한 다른 서버로 연결을 변경하지 않는다.

**제 3자 서비스 연동**

채팅 앱에서 가장 중요한 제 3자 서비스는 푸시 알림이다. 푸시 알림의 경우 3장 참고

**규모 확장성**

단일 서버로 구현하려면 할 수 있지만 SPOF가 될 수 있기 때문에 규모 확장성에 대해 고려해야 한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/6de7045c-8df4-48ab-9af4-29cdfb3197a2)

클라이언트 - 채팅 서버의 웹 소캣 연결은 끊지 않고 유지된다는 점만 유의!

- 채팅 서버는 클라이언트 사이의 메시지를 중계하는 역할
- 접속상태 서버는 사용자의 접속 여부를 관리한다.
- API 서버는 로그인, 회원가입, 프로필 변경 등 나머지 API를 처리한다.
- 알림 서버는 푸시 알림을 보낸다.
- 키-값 저장소에는 채팅 이력을 보관한다. → 채팅 이력을 빠르게 불러올 수 있음

**저장소**

RDB를 쓸 것인가 NoSQL을 쓸 것인가 판단하기 위해 데이터 유형과 읽기/쓰기 연산의 패턴을 알아야 한다.

채팅 시스템이 다루는 데이터

- 사용자 프로필, 설정, 친구 목록과 같은 일반적인 데이터: RDB에 안전하게 보관하는게 좋다.
- 채팅 이력과 같은 채팅 시스템의 고유한 데이터: NoSQL 중 키-값 저장소 사용하는게 좋다.
    - 채팅 이력 데이터는 엄청난 양이 존재한다.
    - 빈번하게 사용되는 것은 최근에 주고 받은 메시지
    - 특정 사용자가 언급된 메시지를 보거나, 특정 메시지로 점프하는 경우 무작위적인 데이터 접근 가능
    - 읽기 쓰기 비율이 1:1
- 키 - 값 저장를 사용하는 이유
    - 수평적 규모 확장이 쉽다.
    - 데이터 접근 지연시간이 낮다.
    - RDB는 롱 테일에 해당하는 데이터를 잘 처리하지 못하는 경우가 많음 + 인덱스가 커지면 데이터 무작위 접근을 처리하는 비용 증가
    - 여러 시스템들이 이미 키-값 저장소를 바탕으로 채팅 시스템 잘 구현함

### 데이터 모델

메시지 데이터를 어떻게 보관할 것인가

**1:1 채팅을 위한 메시지 테이블**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/d6109dd7-8c45-44fb-9cb6-0b5abdb10af6)

**그룹 채팅을 위한 메시지 테이블**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/df3d69df-e0f2-4fc1-aae7-0a4e72f7f247)

복합키 사용, 채널 id를 파티션 키로 사용하여 파티션 가능

**메시지 ID**

메시지 ID는 다음과 같은 속성을 만족해야 함

- 유니크
- 정렬 가능 및 정렬 시 시간 순서와 일치해야 한다.

스노플레이크 같은 순차 번호 생성기를 활용하여 메시지 ID 생성 가능하다.

7장 참고

지역적 순서 번호 생성기 활용해도 된다. (그룹끼리만 순차적인 번호 가지도록 설계)

## 상세 설계

---

### 서비스 탐색

클라이언트에게 가장 적합한 채팅 서버를 추천해주는 역할을 담당한다.

사용되는 기준은 클라이언트의 위치, 서버의 용량 등이 있다.

**주로 쓰이는 기술은 아파치 주키퍼를 주로 사용한다.**

→ 사용 가능한 채팅 서버를 모두 등록하고 클라이언트가 접속할 시 사전에 정한 기준에 따라 채팅 서비스를 골라준다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/f503c838-fc80-48a2-a69f-fa6998e9fa09)

1. 사용자가 시스템에 로그인
2. 로드 밸런서가 로그인 요청을 API 서버로 보낸다.
3. API가 사용자 인증을 처리하고 나면 서비스 탐색 기능이 동작하여 최적의 채팅 서버를 찾는다.
4. 사용자가 매핑된 채팅 서버와 웹소캣 연결을 맺는다.

### 메시지 흐름

**1:1 채팅 메시지 처리 흐름**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/9ef43cac-db71-4cf3-a215-7c02a3a6f2cb)

1. 사용자가 채팅 서버로 메시지 전송
2. 채팅 서버는 ID 생성기를 사용하여 메시지 ID 결정
3. 채팅 서버는 해당 메시지를 메시지 동기화 큐로 전송
4. 메시지가 키-값 저장소에 저장된다.
5. 메시지를 받는 사용자
    1. 접속 중인 경우: 메시지가 사용자가 접속 중인 채팅 서버로 전송된다.
    2. 접속 중이 아닌 경우: 푸시 알림 서비스를 이용하여 푸시 알림을 보낸다.
6. 채팅 서버는 메시지를 메시지를 받는 사용자에게 전송한다.

**여러 단말 사이의 메시지 동기화**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/a574cb26-da5d-4148-9862-f4ac2976bd63)

각 단말은 채팅 서버와 웹소캣 연결을 유지한다.

각 단말은 `cur_max_message_id` 변수를 들고 있는다.

→ 조회한 가장 최신의 메시지의 ID를 추적하는 용도

새 메시지 판단 기준

- 수신자 ID가 현재 로그인한 사용자 ID와 같음
- 키-값 저장소에 보관된 메시지로서, 그 ID가 `cur_max_message_id` 보다 큰 값이다.

**소규모 그룹 채팅에서의 메시지 흐름**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/2eed2014-51b7-48fc-a895-f4d2589a4000)

A가 메시지를 보내면 B와 C의 메시지 동기화 큐에 복사된다.

소규모 그룹 채팅의 경우 좋은 설계이다.

- 새로운 메시지 확인은 자신의 큐만 확인하면 된다.
- 그룹이 크지 않으므로 메시지를 수신자별로 복사해서 큐에 넣는 작업의 비용이 문제가 되지 않는다.

**다만 많은 사용자를 지원하는 그룹 채팅의 경우 이러한 설계 방식은 좋지 않다.**

한 수신자는 여러 사용자로부터 오는 메시지를 수신할 수 있어야 한다.

그렇게 되면 아래와 같은 설계가 나온다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/f29ada91-b798-4869-a052-c371625fd264)

### 접속 상태 표시

접속상태 서버는 웹소캣으로 통신하는 실시간 서비스의 일부이다.

**사용자 로그인**

클라이언트와 실시간 서비스 사이에 웹 소캣이 연결이 맺어지면 접속 상태 서버는 사용자의 상태와 `last_active_at` 타임 스탬프 값을 키-값 저장소에 저장한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/7e4c3f3c-7719-4770-8d80-eacf02deb20a)

**로그아웃**

키-값 저장소에 보관된 사용자의 상태가 online에서 offline으로 바뀌게 된다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/28592871-4bb4-4f3a-a16a-3d50fa9336e5)

**접속장애**

사용자의 인터넷이 끊어지게 되면 클라이언트와 서버 사이에 맺어진 웹소캣 연결도 끊어진다.

→ **간단한 방법은 사용자를 오프라인으로 바꾸고 다시 연결되면 온라인으로 바꾸는 것**

하지만 잠깐 끊겼다고 오프라인으로 바꾸면 사용자에게 별로 좋은 UX를 제공하지 못할것이다.

그렇기에 **박동 검사를 통해 문제를 해결하는게 좋다!**

주기적으로 박동 이벤트를 보내 사용자의 접속 상태를 체크하는게 더 바람직하다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/0a8d844d-5001-40d8-9f46-254cdf8ba4bd)

**상태 정보의 전송**

상태 정보 서버는 발행-구독 모델을 사용한다.

→ 각각의 친구 관계마다 채널을 하나씩 두는 방법

이 방법은 그룹의 크기가 작을 때 효과적이다.

→ 그룹의 크기가 커진다면 이벤트 발행이 너무 많아지는 문제가 있다.

이럴 때를 대비하여 **사용자가 그룹 채팅에 입장할 때만 상태 정보를 읽거나, 친구 리스트에 있는 사용자가 접속 상태를 수동으로 갱신하도록 설계 가능하다.**

## 마무리

---

추가적으로 논의할 내용

- 채팅 앱에서 사진, 비디오와 같은 미디어 파일을 지원하는 방법: 압축, 클라우드 저장소, 섬네일 등
- 종단 간 암호화: 메시지 전송에 있어 암호화를 하여 주고받는 사람만 메시지를 알도록 설계
- 캐시: 이미 읽은 메시지를 캐싱해두면 서버와 주고받는 데이터 양을 줄일 수 있다.
- 로딩 속도 개선
- 오류 처리
    - 채팅 서버 오류: 서비스 탐색 기능이 동작하여 다른 서버로 보내줘야 한다.
    - 메시지 재전송: 재시도나 큐는 메시지 안정성을 보장해준다.
