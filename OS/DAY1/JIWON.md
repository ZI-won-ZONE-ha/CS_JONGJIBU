## 운영체제?

컴퓨터 하드웨어 바로 위에 설치되어 사용자 및 다른 모든 소프트웨어와 하드웨어를 연결하는 소프트웨어 계층

## 운영체제의 목적

자원을 효율적으로 관리하기 위한 목적으로 사용된다.

- 형평성있는 자원 분배
- 주어진 자원에서 최대한 효율을 낼 수 있도록
- 사용자가 컴퓨터 자원을 편리하게 사용할 수 있도록

## 운영체제 분류

- 동시 작업 가능 유무
    - 단일 작업
    - 동시 작업 → 요즘 운영체제
- 사용자의 수
    - 단일 사용자
    - 다중 사용자
- 처리 방식
    - 일괄 처리 (batch)
    - 시분할 방식 → 각 작업마다 일정한 시간 할당 → interactive
    - 실시간 → 정해진 시간 내로 어떤 작업이 완료되어야 하는걸 보장해야 할 때

## CPU, Memory, Cache

- CPU
    - clock 기반으로 동작하는 Digital Logic 회로이다. (clock : 0과 1이 반복되는 pulse)
    - 컴퓨터의 모든 장치의 동작을 제어하고 수행하는 역할을 한다.
- Memory
    - 컴퓨터의 모든 장치의 동작을 제어하고 수행하는 역할을 한다.
- Cache
    - CPU와 Memory 사이의 속도 차이를 보완하기 위해 사용하는 고속 버퍼
    - 메인 메모리까지 가지 않아도 되기 때문에 속도를 보완할 수 있다.

## 프로세스

실행 중인 프로그램을 말한다. → 메모리에 프로그램이 load 된 상태를 말한다.

- 프로세스의 context
    - CPU 수행 상태를 나타내는 하드웨어 문맥
        - Program Counter: 현재 프로세스가 어디까지 실행 되었나
        - register에 어떤 값을 넣어 뒀는가
    - 프로세스 주소 공간
        - code, data, stack에 어떤 데이터가 들어가 있는가
    - 커널 주소 공간
        - PCB
        - 커널 stack

## PCB (Process Control Block)

OS에서 **프로세스를 제어할 때 필요한 프로세스 상태 정보를 저장**한다.

OS 커널 메모리 영역에 생성된다.

프로세스 생성 시 PCB가 생성되고, 프로세스 종료 시 PCB 제거 됨

## 프로세스 메모리

프로세스가 점유한 메모리 공간은 사용 목적에 따라 나누어진다.

- Code 영역
    - 컴파일된 코드 내용을 저장하는 공간
- Data 영역
    - 전역 변수와 정적 변수를 저장하는 공간
- Stack 영역
    - 함수의 리턴 주소, 함수 파라미터, 함수 지역 변수 등의 데이터를 저장하는 공간
- Heap 영역
    - 동적으로 할당된 데이터를 저장하는 공간

## 프로세스 상태

- 생성
    - 프로세스가 생성된 상태
- 준비
    - CPU를 할당받기 위해 기다리는 상태
- 대기
    - 이벤트, 입출력 대기 신호 발생을 기다리는 상태
- 실행
    - CPU를 점유하여 명령어가 실행되는 상태
- 종료

## Context Switching

CPU에서 현재 작업 중인 프로세스를 바꾸는 과정

PCB에서 교체 당하는 프로세스의 상태를 저장해줘야 한다.

인터럽트, system call 등으로 발생한다.

## Thread

프로세스의 자원과 제어에서 제어만 분리한 실행 단위를 말한다.

하나의 프로세스에 하나 이상의 thread를 가질 수 있다.

thread끼리 code, data, heap 영역은 공유하고 stack 영역은 독자적으로 가진다.

- multi thread 프로그래밍 시 stack 영역을 제외한 영역으로 인한 동시성 문제를 고려해야 된다.

## Multi Thread Model

Thread의 종류

- Kernel Thread: 커널에서 직접 생성하고 관리하는 thread
    - kernel thread가 실질적으로 cpu에서 실행된다.
- User Thread: User가 라이브러리를 사용해 직접 구현한 스레드
    - User Thread를 kernel Thread에 연결해서 사용한다.

### User Level Thread

사용자의 여러 thread가 하나의 kernel Thread에 연결되는 형태 (1 to N)

kernel Thread 하나가 CPU를 계속 점유하기 때문에 Context Switching이 필요없다.

kernel Thread가 대기 상태에 들어가면 모든 user Thread 대기

보안에 취약하다 → 하나의 kernel thread만 털면 되기 때문에

### Kernel Level Thread

하나의 사용자 thread에 하나의 kernel thread 연결 (1 to 1)

특정 kernel Thread가 대기 상태에 들어가도 다른 kernel Thread로 작업 가능

Context Switching에 따른 오버헤드가 발생한다.

Multi CPU 사용 가능

보안에 강하다.

kernel Thread가 생성 제한이 있기 때문에 사용자 Thread도 같이 생성 제한이 걸린다.

### Multi-level Thread

앞선 두 방식을 합친 방식이다. (M to N Model)

하나의 커널 스레드가 대기 상태에 들어가면 다른 커널 스레드가 대신 작업 → 유연한 작업 가능

→ User Level보다 더 유연하게 작업 처리가 가능하다.

커널 레벨 스레드를 같이 사용하므로 Context switching이 발생하지만 Kernel level보다는 적게 발생한다.

→ User Level보다 빠르지 않다.

두 방식의 장점을 가져가면서 단점도 가져가는 형태

### Two-level Model

빠르게 동작해야되는 스레드는 사용자 레벨로 보낸다.

안정적으로 움직여야하는 스레드는 커널 레벨 스레드로 보낸다.

## 멀티 프로세스, 멀티 스레드

### 멀티 프로세스

- 여러 프로세스가 돌아가면서 CPU를 점유한다. → Context Switching 비용이 크다.
- 프로세스가 서로 독립적인 공간을 가지므로 데이터 공유가 어렵다. → IPC 필요
- 대신 서로 독립적이기 때문에 하나의 프로세스에 장애가 발생하더라도 다른 프로세스에 영향을 안준다.

### 멀티 스레드

- 프로세스 내의 작업을 여러 스레드로 분할 → 작업의 크기가 작아진다.
- 스레드 간 메모리 공유 → 서로 데이터 공유가 쉽다. + Context Switching 비용이 작다.
- 특정 thread에 이슈가 발생하면 같은 영역에 있는 thread들에 영향을 준다.

## CPU 스케줄링

어떤 프로세스를 어떤 시점에 CPU에 할당할지 결정해야 된다.

→ 시스템 작업 처리를 원할하게 해줘야된다.

스케줄링의 목적

| 공평성 | 자원을 공평하게 배정 |
| --- | --- |
| 효율성 | 자원을 유후시간이 없는 최대한의 효율로 사용 → 유후 자원을 사용하려는 프로세스에게 우선권 |
| 안정성 | 우선순위로 중요 프로세스가 먼저 작동하도록 배정 → 시스템을 파괴하려는 프로세스로부터 보호 (why? 보호하는 프로세스의 우선순위가 높아서 즉시 보호할 수 있기 때문에) |
| 확장성 | 프로세스가 증가하여도 시스템이 안정적으로 작동하도록 조치. 자원이 늘어났을 때 이를 시스템에 반영해준다. |
| 반응 시간 보장 | 시스템이 적절한 시간 안에 프로세스의 요구에 반응하게 해준다. |
| 무한 연기 방지 | 기아(starvation) 방지 |

## 스케줄링의 단계

- Long-term (High-level) 스케줄러

현재 동시에 실행되고 있는 프로세스의 숫자를 조절하여 **시스템의 전체 작업 수를 조절한다.** 

주로 created-ready, run-terminated 상태 전환 결정 

- short-term (Low-level) 스케줄러

어떤 프로세스에 CPU를 할당할지, 어떤 프로세스를 대기 상태로 보낼지 결정한다.

ready-run-waiting 상태 전환을 담당한다.

- Mid-term (Medium-level) 스케줄러

suspend 와 resume으로 활성화된 프로세스의 수를 조절하여 과부하를 막는다.

일부 프로세스를 suspend 상태로 보내서 나머지 프로세스 완만하게 동작하도록 돕는다.

short-term 스케줄링이 잘 이루어지도록 완충하는 역할을 수행한다.

suspend 상태가 존재하는 운영체제에만 있는 스케줄러이다.

## Preemtive, Non-Preemtive 스케줄링

### Preemtive

프로세스 하나가 장시간 CPU 점유하는 것을 막는다. (독점 방지)

우선 순위가 높은 프로세스 긴급처리가 가능하다.

우선 순위가 낮아 기아가 된 프로세스에 대한 처리도 가능하다.

Context Switching이 많아지는 단점이 있다.

우선 순위를 정말 잘 줘야 의미 있게 활용할 수 있다.

### Non-Preemtive

프로세스가 스스로 CPU를 나갈때 까지 점유하는 방식

비효율적일 수 있다.

응답 시간 예측이 쉽다.

Context Switching이 적게 발생하여 오버헤드가 감소한다.

## 스케줄링 알고리즘

### FCFS(First Come First Served)

큐와 비슷한 방식 (오는 순서대로 처리)

Non-Preemtive 방식으로 가장 단순한 알고리즘 (Preemtive로 사용 가능하긴 함)

Batch 시스템에서 효율적

프로세스 도착 순서에 따라 성능이 바뀐다.

### SJF(Shortest Job First)

Non-Preemtive 방식, Preemtive 방식 모두 사용 가능

Ready 상태에 존재하는 프로세스 중 실행시간이 가장 짧은 프로세스 먼저 할당하는 방식

→ 다른 프로세스의 대기 시간을 최소화하는게 목적이다.

기아 발생 가능, 실행 시간 예측 어려워 사용하기 힘든 단점 존재

### SRTF(Shortest Remaining Time First)

SJF를 Preemtive 방식으로 변형한 알고리즘

일단 먼저 도착한 프로세스를 실행하는데 중간에 실행시간이 더 짧은 프로세스가 Ready queue에 들어오면 해당 프로세스로 작업을 교체하는 알고리즘이다.

### Priority scheduling

프로세스에 우선 순위를 두어 우선 순위가 높은 프로세스 먼저 처리하는 방식

Non-preemtive, preemtive 둘 다 사용가능하다.

실시간 시스템에서 빠른 응답이 가능해서 사용하기 좋다.

Starvation이 발생할 수 있다.

→ aging을 이용하여 오래 기다린 프로세스의 우선순위를 높혀주는 방법으로 기아 해결 가능

단, 에이징을 도입하면 오버헤드 발생 가능

### Round-robin

시분할 시스템을 위해 설계했다. → Preemtive에서만 사용 가능하다.

time quantum 또는 time slice를 정의해야된다.

Ready queue를 circular queue로 설계하여 스케줄러가 ready queue를 돌면서 CPU를 할당한다.

모든 프로세스가 공평한 시간동안 CPU 사용 가능

time quantum이 작을수록 context switching이 자주 발생한다. 

→ context switching에 소요하는 시간보다 time quantum을 충분히 크게 하는게 좋다.

→ 적절한 time quantum을 설정해야된다.

프로세스가 CPU를 동일하게 점유하여 제한된 waiting time을 가지고 기아가 발생하지 않는다.

### HRN(Highest Response-ratio Next)

SJF의 약점인 불평등을 보완하여 나온 알고리즘

Non-preemtive 스케줄링이며 priorty 스케줄링의 다른 예시이다.

FCFS, SJF의 약점을 해결하기 위해 제안되었다.

**우선순위 = 서비스를 받을 시간 + 대기한 시간 / 서비스를 받을 시간**

기아를 완화한다.

오버헤드가 있고, **여전히 불공정**하여 자주 사용되지는 않는다는 단점이 있다.
