## Mutual Exclustion (상호 배제)

병행 프로세스에서 **프로세스 하나가 공유 자원 사용시 다른 프로세스들이 해당 공유 자원에 접근하지 못하게 하는 방법**

> 읽기 연산: 모두가 하나의 자원에 대해 읽는다면 동시에 접근해도 무방
>
> 쓰기 연산: 정해진 순서대로 작업이 진행되어야 한다. → 모두가 동시에 들어가면 이를 지킬 수 없음

## Mutual Exclusion의 조건

1. 두 프로세스는 동시에 공유 자원에 진입할 수 없다.
2. 프로세서의 속도나 프로세서 수에 영향을 받지 않는다.
3. 공유 자원을 사용하는 프로세스만 접근이 가능하고 나머지는 차단
4. 프로세스가 공유 자원을 사용하려고 너무 오래 기다리면 안된다.

## Process Synchronization (프로세스 동기화)

프로세스들이 **공유 자원을 동시에 사용하지 못하게 실행을 제어하는 방법**

동기화는 순차적으로 재사용 가능한 자원을 공유하려고 상호작용하는 프로세스 사이에서 발생한다. 

**동기화로 상호배제를 보장**은 가능하나 **동기화 과정으로 인한 교착 상태와 기아 상태가 발생할 수 있다.**

## Critical Section (임계 영역)

다수의 프로세스가 접근 가능하지만, 어느 한 순간에서는 프로세스 하나만 이용 가능한 영역을 말한다.

### Critical Section을 활용한 Mutual Exclusion

Lock을 이용하여 간편하게 구현할 수 있다.

들어갈 때 key를 얻고 임계 영역을 Lock하고 들어간다.

나가면서 key를 반납한다.

### Critical Section의 조건

1. Mutual Exclusion: 누군가 임계 영역에 진입하면 다른 프로세스는 진입할 수 없다.
2. Progress: 프로세스 여러 개가 임계 영역에 진입하고자 할 때 누구를 먼저 들여보낼지 결정해야된다.
3. Bounded Wating: 다른 프로세스가 임계 영역을 무한 대기하는 상황을 막기 위해 임계 영역에 들어갔다 나온 프로세스는 다음 임계 영역에 다시 들어갈 때 제한을 둬야한다.

## Producer - Consumer Problem

비동기적으로 수행하는 모델에서 producer 프로세스가 생산한 정보를 consumer 프로세스가 소비하는 형태를 Producer - Consumer라고 한다.

이 때 Producer와 Consumer가 Buffer를 공유하게 되는데 **Buffer의 사이즈가 유한하기 때문에 Producer의 생산 속도와 Consumer의 소비 속도가 균형이 맞아야 한다.**

→ Buffer의 상태로부터 동작을 결정하는 과정이 필요하다. (꽉 찬 Buffer에 생산한 데이터를 넣으면 안되기 때문에)

그리고 두 프로세스가 Buffer의 생산, 소비 Pointer 정보를 교환하는 과정이 필요하고, 이 Pointer 정보가 공유되기 때문에 Pointer 정보를 Critical Section에 넣어둬서 동기화를 해줘야 한다.

만약 생산 Pointer와 소비 Pointer가 같은 위치에 존재하면 이는 Buffer가 가득 찼거나, 빈 상태를 의미하여 이 때 특정 로직이 필요하게 된다. (생산 프로세스는 데이터를 생산 안하게, 소비 프로세스는 데이터 소비 안하게 로직 설계 필요)

관련 링크: https://letsmakemyselfprogrammer.tistory.com/112

## Race Condition

여러 프로세스가 동시에 공유 데이터 접근 시, 접근 순서에 따라 실행 결과가 달라지는 상황

**공유 데이터의 결과를 예측할 수 없는 문제 발생**

예측이 안되는 문제는 발생시키면 안된다.

### Race Condition 예방

동기화: Critical Section으로 Mutual Exclusion 구현

→ 임계 영역에 하나의 프로세스만 접근할 수 있게 설계하여 Race Condition 자체가 발생하지 않게 설계해야된다.

## Mutual Exclusion 구현

여러 방법이 존재한다.

소프트웨어 레벨로 구현할 수 있고, 하드웨어 레벨로 구현할 수 있다.

## Mutex Lock

Critical Section에 진입할 때 Mutex Lock을 얻고 나갈 때 릴리즈 하는 방식

acquire()로 Mutex Lock을 얻고, release() 함수로 릴리즈

busy waiting 발생 → 프로세스가 Running에서 계속 기다려야 된다.

Mutex Lock을 얻은 프로세스만 이를 해제할 수 있다.

## Semaphore

Semaphore S라는 Integer 타입의 변수를 두는 방식

S의 값을 유동적으로 두어 여러 프로세스가 Critical Section에 진입하게 설계할 수 있다.

S 값을 감소시키면서 Critical Section에 진입하고, S 값을 증가시키면서 Critical Section에서 나간다.

Lock을 걸지 않은 프로세스도 Signal을 보내 S 값을 조정하여 Lock을 해제할 수 있다.

## Monitor

Semaphore에서 S 값의 연산이 일부 생략되는 경우 문제가 발생하는데 이를 해결하기 위해 등장한 방법이다.

프로그램 언어 레벨에서 제공하는 Mutual Exclusion 구현 방식

> Java의 Synchronized는 Monitor 기반이다.
> 

Semaphore에 비해 안정적인 장점이 있다.

→ 일단 프로세스가 공유 자원에 접근하는걸 막기 때문에 (모니터를 생성해야 되기 때문)

관련 링크: https://tecoble.techcourse.co.kr/post/2021-10-23-java-synchronize/

## Deadlock

두 프로세스가 서로 사용하는 자원을 사용하기 위해 기다리는 상태

→ 절대로 요청을 받아들일 수 없게 된다. (무한 대기)

자원 해제 요청을 받아들일 때까지 프로세스들은 작업 진행 불가

자원 해제 수신 때까지 현재 보유 자원도 해제 불가

OS가 교착 상태를 해결하지 못하면, 시스템 운영자나 사용자가 작업 교체 및 종료를 하는 외부 간섭으로 해결해야 된다.

### 프로세스의 자원 사용 순서

1. 자원 요청: 프로세스가 필요한 자원 요청, 해당 자원을 다른 프로세스가 사용 중이면 요청을 수락할 때 까지 대기
2. 자원 사용: 프로세스가 요청한 자원 획득하여 사용
3. 자원 해제: 프로세스가 자원 사용을 마친 후 해당 자원 되돌려(해제) 준다.

## Deadlock의 발생 조건

1. Mutual Exclusion: 자원에 하나의 프로세스만 들어갈 수 있음
2. Hold and Wait: 자원 점유한 상태에서 대기
3. No Preemtive: 선점 불가능
4. Circular wait: 서로 원하는 자원이 순환되는 구조

## Deadlock의 해결 방법

1. 예방: 위의 Deadlock 발생 조건들을 제거 → 자원 사용 효율이 떨어진다.
2. 회피: Deadlock이 발생하려고 할 때 회피 → Deadlock 발생 상황을 감지해야 한다. 이 과정에서 오버헤드 발생 가능
3. 감지 후 복구: Deadlock이 발생 했을 때 감지하고 복구 → 주기적으로 Deadlock 걸렸는지 확인해야 된다.

### Avoidance

prevention에 비해 덜 엄격한 조건으로 자원을 효율적으로 사용가능

prevention에 비해 병행성이 증가하는 장점

회피 방법 : deadlock 발생이 예상될 때 1. 프로세스의 시작 중단 2. 자원 할당 거부

각 자원의 할당 상태, 프로세스 최대 요청을 계속 분석해야 된다. → 오버헤드가 발생한다.

### Recovery

데드락이 발생할때까지 아무것도 안한다. → Avoidance에서 발생하는 오버헤드 감소

Deadlock을 감지하는 Deadlock Detection Algorithm 필요

Deadlock 회복 알고리즘 필요

데드락 감지는 주기적으로 하기 때문에 데드락 감지 알고리즘이 가벼워야 좋다. 

Deadlock 회복 알고리즘은 무겁더라도 확실하게 해야 좋다. 

데드락 감지 **알고리즘을 자주 실행하면 시스템 성능이 감소**하지만 더 빨리 감지가 가능해서 유휴 상태 방지 가능. 감지를 자주 안하면 반대 상황 발생. 

 필요한 정보를 유지하고 탐지 알고리즘을 실행시키는 비용뿐 아니라 데드락 회복에 필요한 부담까지 요청.

## Starvation

Deadlock 해결책들은 Deadlock은 해결해줘도 Starvation은 해결하지 못한다.

Starvation을 해결하기 위해 먼저 기다리는 작업을 파악하면서 작업들이 기다린 시간을 알아야한다.

그리고 Starvation이 감지되면 해당 작업을 실행할 수 있도록 조치해줘야 한다.
