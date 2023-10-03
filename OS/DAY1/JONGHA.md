# Process란?

프로세스는 프로그램을 돌리는 것이다. 

프로그램은 실행가능한 파일이 메모리에 적재될 때 프로세스가 된다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/a698f0ee-eac9-4651-91b6-041d5b0af190)


프로세스는 code, data, stack, heap으로 이루어져 있다. 

- Process vs Program 의 차이
    
    Program은 디스크에 저장되어 있는 일련의 `instruction` 들로 이루어진 `passive Entity` 이다. 
    
    Process는 `Program Counter`과 함께 이루어진 `Active Entity`이다. 
    

# Process의 생성 과정

1. 프로그램 코드를 메모리에 로딩한다.
    1. 프로그램은 실행 가능한 형태로 처음에 디스크에 올려져 있다. 
    2. Modern OS는 `Lazily` 하게 로딩한다. 
    
    즉, 모든 프로그램 데이터를 로딩하는 것이 아닌, 필요한 데이터만 로딩한다 ( = `메모리 가상화`)
    
2. 프로그램 런타임 스택을 할당한다. 
    1. 런타임 환경에서 가장 중요한 것은 스택이다. 이는 함수의 로컬 변수, 함수 호출할 때 파라미터, return address를 넘긴다. 
3. 프로그램의 heap을 초기화 한다. 
4. 다른 initialization Task를 진행한다.
    1. IO initialization : 모든 프로세스는 default로 파일 디스크립터 3개가 생긴다. Stdin(0), stdout(1), stderr(2)
5. 프로그램을 start하고, main function을 실행 시킨다. 
    1. OS가 새롭게 생성된 Process에게 CPU 의 권한을 넘긴다. 

# Process States And Transition

모든 프로세스는 state를 갖고 있고, 이는 운영체제 마다 다르다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/4374931d-f018-4418-b786-c95443ee110b)


- Ready : 메모리에 올라가 있는데, 돌고 있지 않지만 대기하는 상태. 스케줄링 알고리즘을 통해, 대기하고 있는 여러 Ready Process에서 Running 상태로 바꿔준다.
- Running : 실제로 실행되는 프로세스. 1개 밖에 없음
- Blocked : IO 또는 event wait 작업을 진행할 때 Block 상태로 간다. 해당 작업이 끝나면 다시 `Ready` 상태로 간다.
- 모든 프로세스는 CPU 와 IO 작업을 반복하는 과정을 거친다. IO를 한 후, `Ready`로 바꾼다.  잠시 대기하고 있는 프로세스 중 하나를 골라주면 CPU 사용도가 높아진다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/0244fee1-e80b-4f60-9a32-ed4f2e22e44e)

- `Running` State에서 반드시 `Terminate`를 할 때 `Zombie` 상태를 거친다. Parent가 `Running` 하면서 Child Process를 회수해주어 프로세스를 제거한다. 즉, 모든 프로세스는 일시적으로 좀비 상태에 머무르며, 누군가에 의해 Reaping 당하면 `Terminate` 상태로 간다.

# Process 자료 구조

프로세스의 주소 공간은 `code, data, stack` 등으로 구성된다. 각 프로그램마다 이러한 주소 공간을 별도로 가지며, 이러한 주소 공간을 `Virtual Memory` 또는 `Logical Memory`라고 한다. 이는 실제 물리적 메모리의 주소와 독립적으로 각 프로그램마다 독자적인 주소 공간을 가지기 때문이다.


OS 또한 프로그램이므로 `Code`. `Data`, `Stack`의 주소 공간을 가지고 있다.

- 커널의 Code 영역에는
    - `CPU`, `Memory` 등 자원 관리를 위한 부분과
    - 사용자에게 편리한 인터페이스를 제공하기 위한 부분
    - System Call, 인터럽트 처리하는 부분
- 커널의 Data 영역에는 각종 자원을 관리하기 위한 자료구조가 저장된다.
    - CPU, Memory 와 같은 하드웨어 자원 관리
    - 현재 수행중인 프로그램(프로세스)을 관리. **`PCB`**
        - 각 프로세스의 상태, CPU 사용 정보, 메모리 사용 정보 등을 유지
- 커널의 Stack 영역은 함수 호출 시의 복귀 주소를 저장하기 위한 용도로 사용된다.
    - 유저 프로그램의 스택과 달리 현재 수행중인 프로세스마다 별도의 스택을 두어 관리한다.
    - 프로세스가 System Call 후 System Call 내부에서 다른 함수를 호출하는 경우
    - 그 복귀 주소는 커널 내의 주소가 되어 유저 프로그램의 스택과는 별도의 저장 공간이 필요하게 됨.
    - 커널은 일종의 공유 코드이므로 일관성 유지를 위해 각 프로세스마다 커널 내 별도의 스택이 필요함
    

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/d1459fb5-a59f-44f0-a72c-b1b00f097ed1)

## PCB (Process Control Block)

모든 프로세스가 생성이 되면, 해당 프로세스의 중요한 정보를 담고 있는 자료 구조가 생기는 데 이를 PCB라고 한다. Program counter, register, open file 에 대한 정보를 가지고 있다. 

## Process List (Queue)

시스템 상 프로세스는 하나가 돌고, 나머지는 기다리고 있다. 

- Ready processes → Ready Process 를 담고 있는 Queue
- Blocked Processed
- Current running Processes

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c966219f-b80f-49b1-a53f-717d4fc0ed99)

<aside>
💡 즉 정리하자면! 새로운 프로세스를 만드는 것은 프로그램을 메모리에 올리고, Stack, heap 그리고 다른 과정을 초기화 하는 것이다. 그리고 메모리에 올라오면 새로운 PCB가 malloc되고, 해당 프로세스가 나갈 때까지(`exit`) 할 때 까지 `PCB`를 유지한다.

</aside>

<aside>
💡 궁금증 : PCB는 커널의 어느 영역에 저장이 될까? → Data 영역

</aside>

# Process API

### fork

- 부모 프로세스는 fork를 이용하여 자식 프로세스를 생성한다. 부모의 address space를 copy한다.
- fork의 리턴 값이 달라지는데, parent는 자신이 생성한 자식 프로세스의 pid를 받게 된다. 반면에 child Process는 자신이 생성한 프로세스가 없으므로 0을 받게 된다.
- fork를 하면서 둘다 ready 상태로 바뀌는데, 누구를 먼저 선택할 지 는 OS 의 스케쥴러에 따라 달라진다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/098c87b6-591a-4d86-9c3a-f46767ad6729)

### Wait() or Waitpid

- 자식 프로세스가 `terminate`되면, `exit`을 통해서 `wait`하고 있는 부모 프로세스에게 알려주어 자식 프로세스의 `PCB`를 회수 할 수 있다.
- 만약, 부모 프로세스가 wait 하지 않는다면, child process는 zombie, 즉 시스템에서 불필요한 프로세스가 돌고 있다는 이야기이다. 이는 성능에 영향을 미친다.
- 자식은 exit 함수의 number를 통해서 부모 프로세스에게 정상, 또는 비정상적으로 끝났는지 알려준다.

### Zombie vs Orphan 프로세스

- zombie
    - Parent is executing while the child is dead
    - 프로세스 수행은 마쳤지만, Process Table에 entry가 존재하는 프로세스이다.
    - 모든 프로세스는 좀비 상태에 잠깐 머무르고 끝난다.
    - Parent가 wait 함수를 통해 정상적으로 회수할 수 있다.
- Orphan
    - Parent is dead, while the child is executing
    - Init 프로세스가 주기적으로 orphan 프로세스 리핑하는 절차 거침.

<aside>
💡 Orphan 이었다가 좀비로 가는 경우도 있을까?

</aside>

### exec()

- OS는 새로운 binary 이미지를 만들고, stack, heap을 새롭게 초기화 한다. 기존의 메모리를 재사용하며, 새로운 프로세스를 돌린다. 따라서 exec 밑에 구문들은 수행되지 않는다.
- exec는 return 값이 존재하지 않는다. 해당 이미지를 수행한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/591343a6-4244-4600-87ae-30a50af60765)

- Parent는 exec를 수행한 프로세스를 받아준다.

<aside>
💡 fork() 와 exec() 는 왜 분리하였을까? 
→ fork 를 한 다음 exec 부르기 전에 다양한 초기화를 진행할 수 있다. (ex) IO redirection, pipe 등 )

</aside>

### IO Redirection

File, command, program script 등의 `output` 을 받아서 `input` 으로 변경해주는 것

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/8121da93-eb0e-4207-9d4b-e3e8880ae288)

### File descriptor에서 fork

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/93d24917-5d11-49ea-8b4a-75095145318e)

fork를 하면 부모의 파일 디스크립터 파일을 카피한다. 반대로 exec할 때는 파일 디스크립터 그대로 가져오게 된다. 

# Process 통신 : Pipe

단방향 통신으로, 한프로세스에서 다른 프로세스로 전달된다. 

## Anonymous Pipe

- 존재를 알고 있는 프로세스끼리 통신이 가능하다. 주로 부모 자식간 통신에서 사용한다.
- 부모의 파일 디스크립터가 카피가 된다. fork 시스템 콜을 이용하여 자식 프로세스를 사용한다.
- 단방향 통신 (↔ Named Pipe)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c98a7642-3643-4cbe-84dc-27f69a0559a8)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/92dbee2d-2a4b-4617-aa3a-e83e4f7db208)

## Named Pipe

- 양방향 통신이다.
- 두 개의 프로세스가 관련이 없어도 데이터를 주고 받을 수 있다.
- 여러명의 writer 가 존재할 수 있다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/e2de7278-a10f-443b-b363-2da72eb8b9bb)

- 파일 생성 : mkfifo, mknod
- 중오햔 이슈: 동기화, 즉 writer가 오픈할 때, 해당 named pipe를 읽는 process가 준비되어 있지 않으면 문제가 생긴다. 따라서 open할 때, 상대방이 준비될 때까지 기다리는 옵션을 이용한다. 이는 `NON_BLOCK` 옵션을 통해서 가능하다.

## IPC (Inter-Process Communication)

- 프로세스는 자신만의 `private` 한 address space를 갖고 있다.
- 공유할 수 있는 메모리가 존재한다. 공유할 수 없으면 `message passing`을 한다.
- Signal, pipe, socket : 전통적인 IPC
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c33a5529-479d-4eff-80a3-d58650bfeeca)
    

# CPU Control

## Interrupt Handling

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/80ae5ec8-f6c2-4d81-8505-500415e76230)

interrupt는 interrupt Service Routine (interrupt handler)에게 권한을 넘겨준다. Interrupt Handler 는 운영체제의 한 파트이다. 

- CPU는 PC가 가리키는 곳에 있는 명령을 수행한다
- CPU가 다른 일을 수행하도록 하기 위해 인터럽트 라인을 세팅해 인터럽트 발생
- PC는 다음 명령을 수행하기 직전에 인터럽트 라인을 체크
- 수행중이던 프로세스를 멈추고 OS의 인터럽트 처리 루틴으로 이동하여 처리

특정 프로세스가 CPU를 독점하는 것을 막기 위해 OS는 Timer 인터럽트 사용

- OS는 CPU를 유저 프로세스에게 이양하기 전 타이머를 세팅
- 유저 프로세스가 해당 시간만큼 CPU를 점유한 뒤 발생하는 타이머 인터럽트 때 CPU를 회수하여 다른 프로세스에 이양

### 인터럽트 종류

- 하드웨어 인터럽트
- 소프트웨어 인터럽트 (Trap)
    - Exception
        - 프로그램이 허용되지 않은 연산을 수행하려 할 때
        - (segmentation fault(Trap), divide by 0)
    - System Call
        - 사용자 프로세스가 특권 명령을 수행하려 할 때

### Trap

- disk IO
- keyboard Interrupt
- System call
- Trap table (IDT)
    - os가 trap handler의 주소를 하드웨어에게 알려준다. (privileged instruction)

## Dual Mode

- System call 을 호출하면, 운영체제는 `유저 모드` 에서 `커널 모드`로 바뀌고, `Previleged instruction` 수행한다.
- 사용자 모드에서 interrupt 를 발생 시키면 운영체제 있는 벡터를 통해 핸들러가 동작한다.
- 외부에서 packet이 걸려서 interrupt가 걸리면 유저모드와 상관없이 커널 모드로 가야 하므로, 기존의 유저 코드를 save하고 커널 모드로 간다.
- ex) Open 함수를 동작 시켰을 때 interrupt → vector → handler -> 핸들러 종류마다 번호가 다르게 매겨지고, 번호에 따라서 기능 수행이 된다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/3b84d981-eb81-47d4-8932-32f14c1d1059)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2079b940-bb3b-47b6-b267-927a1664a5d7)

## System Call

- Parameter Passing
    - Register : 용량이 너무 작아서 어렵다.
    - Stack : user stack, kernel Stack 이 존재하는데, 유저의 스택에다 파라미터를 넣어주고, 커널에서 빼내서 수행한다.
    - Block or table : 저장한 블록의 주소를 register에 넣는다.
    
    → 이에 대한 방법은 운영체제 마다 다르다. 
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c326a5a5-0c1c-48e9-bccf-58f2f583cf72)

- stack에 size, buffer, fd, number 순으로 push 한다. 4개를 push 했으므로, 하나당 4바이트이고 +16 만큼 stack 의 point를 옮겨 주어 정상화한다.

## CPU Control Approach

- Cooperative (or non-preemptive) approach
    - 시스템 콜을 기다린다.
    - yield와 같은 시스템 콜을 통해 프로세스는 CPU를 줄 수 있다.
- Non-Cooperative Approach
    - 운영체제가 Timer interrrupt 을 이용해 주기적으로 interrupt를 건다.
- Context switch
    - PC, process state, memory management와 같은 정보들을 프로세스의 context라고 한다.
    - CPU 가 다른 프로세스로 switch 되는 과정에서 그 전의 프로세스의 state는 kernel stack에, 그리고 새로운 프로세스의 state를 커널 스택에서 가져온다.
- 동시화 이슈: 만약 interrupt 걸릴 때, 또 다른 interrupt가 걸린다면?
    - interrupt가 진행될 때 disable 시킨다.
    - lock 기법을 이용하여 동시접근을 막는다.

# CPU 스케줄링

## 성능적인 척도

- Turnaround Time : job이 arrive 하고 complete되는 차이
- Waiting Time : ready Queue 안에서 기다리는 시간.
- Fairness

→ Fairness와 Performance는 tradeoff 이다. 

## FIFO (First In First Out)

### First Come First Served

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/ced47053-addc-4375-8f68-fcb483b8ff05)

- 단점 : **Convoy Effect**
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/580f7d62-8e2c-4a42-b8dd-eed1f119f44a)
    

누가 먼저 수행되느냐에 따라서 Performance 매트릭의 차이가 있다. 

<aside>
💡 Convoy Effect?

</aside>

- CPU bound process  : CPU 많이 쓰므로, CPU burst 길이 길다.
- IO bound process : IO 많이 쓰고,  CPU 쪼금 쓴다. CPU burst 길이가 상대적으로 짧다.

ex) iO 프로세스가 CPU 점유→ 이는 CPU 점유 시간이 짧으므로 IO Queue에 빠르게 들어간다. CPU 프로세스는 CPU time을 길게 쓴다. **따라서 IO 프로세스들이 CPU 를 기다리고 있어 IO device가 idle 한다.** 

CPU 프로세스가 끝나면, IO Queue로 보내지는데, CPU 타임이 짧기 때문에 다시 IO Queue로 가게 된다. 하지만 **CPU bound Process는 여전히 IO를 쓰고 있기 때문에, CPU가 놀게 된다.** 

→ `FIFO` 스케줄러에서 turnaround time 과 waiting time이 매우 길 수 있다. 

## Shortest Job First (SJF)

- Non-preemptive
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/fc6cac57-6299-4973-911c-0469f10a275e)
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/790a977a-fdd5-4230-9257-07d3a6bb76ff)
    

→ 다음과 같이 도중에 들어오게 되면 긴 job을 끝낸 후에야 수행이 가능하다.

- 단점
    
    하지만 sjf 는 잡의 길이를 모를 경우에는 스케쥴링 하기가 어렵다.
    
    따라서 sjf 는 실제로 적용되기 어렵다. 실제로 하려면 들어온 process 의 cpu burst 를 예측하는 방법으로 한다.
    
    기존에 어떤 일이 있었는지 과거의 데이터를 갖고 예측을 한다.
    

## Shortest Time to Completion First (STCF)

- preemption을 넣는다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/eea96d6f-1f9b-4f5b-875c-97f6019c6776)

- 새로운 잡이 들어올 때 기존의 잡과 새로운 잡을 비교하여 스케줄링한다.


# 참고자료

서강대학교 박성용 교수님 운영체제 강의 자료

[운영체제(3)-인터럽트의 원리](https://getchan.github.io/cs/OS_3/)
