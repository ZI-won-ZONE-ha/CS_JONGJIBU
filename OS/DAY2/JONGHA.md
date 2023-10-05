# thread란 무엇일까?

- **flow of control with in Process**
- CPU 스케줄링의 basic unit이다. 즉 프로세스 안에 있는 Thread 단위로 스케줄링이 일어난다.
- 동일한 프로세스 내에서 code, address space, operating resouces를 공유할 수 있다.
    - Thread의 상태를 저장하기 위해서, `TCB` (thread Control block) 이 필요하다.
    - creation과 switching 비용은 비싸지 않다.
- switching 과정
    - T1의 레지스터 state를 저장한다.
    - T2의 레지스터 state를 복구한다.
    - address space를 동일하게 남겨둔다.
    
<img width="684" alt="스크린샷 2023-10-04 오후 4 53 57" src="https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/f3e9b87e-a7e8-441d-a4fa-9607f105d0a6">
    

## Single Thread vs MultiThread Processes

<img width="682" alt="스크린샷 2023-10-04 오후 4 38 54" src="https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c424aecc-b898-4ffd-b480-fd1405c564eb">

다음과 같이 Multi-thread Procees는 thread마다 별도의 stack을 갖고 있다. 또한 공유하는 공간을 가지고 있다. 

## 왜 Thread를 쓸까?

`**Parallelism` 과 `Concurrency`를 지키기 위함이다.** 

### Parallelism

쉽게 말해서, Number of CPUS (cores) 를 나타낸다. 

<img width="587" alt="스크린샷 2023-10-04 오후 4 40 23" src="https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/de48a36c-843e-48bb-acf8-5aea2fb7b8c7">

멀티 코어 상에서 동시 다발적으로 실행할 수 있다. 

### Concurrency

<img width="535" alt="스크린샷 2023-10-04 오후 4 41 00" src="https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c2b520e2-c753-47ec-b110-bd3271a029e0">

싱글 Process의 경우에도 다음과 같이 `concurrent`하게 돌려 `Parallelism`을 수행하는 것과 같은 착각을 일으키게 한다.

(실질적으로는 time-sharing을 한다고 보면 된다.)

<aside>
💡 싱글 프로세스 내에서 IO bound thread가 block 될 때, 커널이 다른 thread로 switch하게 된다면, computation 속도를 높일 수 있다.

즉, Computation과 IO의 overlap 이 가능하다.

</aside>

cf) 참고로 thread API 는 `thread_create`, thread_join 등이 있다. 이에 대한 내용은 작성하지 않을 것이다. 

# Atomicity

atomicity를 시키기 위해선 여러가지 방법이 있다. lock, semaphore, condition variable, zemaphore… 등등..

## Lock

- avaliable (**unlocked** or **free**)
- acquired (**locked** or **held**)

- **Fine-grained approach** : **Critical Section** 을 더 작게 쪼갤 수록 성능이 좋다(Concurrency increase)

## 척도

- Mutual exclusion : 다른 뜨레드들이 critical section에 진입 못하게 막을 수 있을 까?
- Fairness : 각각 뜨레드 들이 starvation 없이 공평한 기회를 가질까?
- Performance : 락을 이용하여 얻는 time 오버헤드

### Interrupt를 막는 방법으로 lock을 이용한다면

- `Single-processor` 에 한정된 방법이다. (CPU당 interrupt가 disable되기 때문에, 다른 CPU가 접근할 수 있다.)
- 시스템의 모든 것들을 다 block 시킨다. (timer, scheduler…)
- 너무 application에 많은 신뢰를 준다. thread에게 interrupt on/off하는 `privileged` 한 Operation을 주면 안된다.

### Hardware 방법 이용 (Test-And-Set)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/bcf98da8-7421-4fe9-879e-9205446f42ca)

수도 코드일 뿐, 이는 하드웨어의 도움으로 atomic하게 일어난다고 가정한다. 즉 이것이 실행되거나 안 실행되거나 둘중 하나의 작업만이 일어난다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/b67813da-a2e3-4ab6-afb7-43dd53900ec7)

→ 이는 `spin-lock`이라고 일컫는다. `Mutual Exclusion`은 지켜주지만, `Fairness`는 지켜주지 않는다. 

### Spin lock

- CorrectNess : Yes
- Fairness : No
- Performance
    - Single CPU에서는 성능이 좋지 않다.
    - Multi CPU에서는 성능이 좋은 편이다. (Critical section이 작다면, lock이 available해질 것이므로)

강의 자료에 `Compare-And-Swap`, `Load-Linked and Store-Conditional` 등 방법이 있는데 이는, 시간이 남을 때 보충하도록 하겠다…

- 참고..
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/34c50f47-b613-4933-be3b-81cd13eb779e)
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/781ec32c-f530-45e3-9beb-fe4912b1d666)
    

### Fetch-And-Add and Ticket-Lock

하드웨어의 지원을 통해 lock을 구현하는 마지막 방법이다.

```c
int FetchAndAdd(int *ptr) {
	int old = *ptr;
	*ptr = old + 1;
	return old;
}
```

위의 코드는 아래와 같이 작동한다.

> ptr이 가리키는 값을 읽어와서,여기에 1을 더해 갱신해주고 이전에 읽어온 값을 반환한다.
> 

```c
typedef struct __lock_t {
  int ticket;
  int turn;
} lock_t;

void lock_init(lock_t *lock) {
  lock->ticket = 0;
  lock->turn = 0;
}

void lock(lock_t *lock) {
  int myturn = FetchAndAdd(&lock->ticket);
  while (lock->turn != myturn)
 	 ; // spin
}

void unlock(lock_t *lock) {
	lock->turn = lock->turn + 1;
}
```

`mutual exclusion`은 하드웨어의 도움으로 만족할 것이고, `fairness` 또한 turn을 주기적으로 증가시켜주며 해당 thread만 동작한다는 점에서 만족한다. 다만 spin lock을 돌고 있기 때문에 `performance`는 ..  

<aside>
💡 Spinning과 Non-spinning의 성능 측면에서 가장 결정적인 차이를 만드는 요소는 무엇일까? → **CPU를 점유하고 있느냐, 그렇지 않느냐 정도가 될 것 같다.** 할 수 있는 것도 없으면서 스케줄러의 선택을 받아 무의미한 busy waiting을 하고 있다.

</aside>

### SpinLock을 해결할 수 있는 가장 간단한 방법? yield

간단히 말해서, thread가 만약 실행할 수 없을 때 `ready`상태로 돌리는 `yield` (running state → ready state) 라는 시스템콜을 호출하는 방법이다. 

이게 과연 최선의 방법일까? 뜨레드가 너무 많아진다면, thread를 계속 `context switch` 하는데에서도 비용이 크게 발생할 것이다. 

---

# 자료 구조에서 thread-safe를 지키는 방법

- Lock을 사용한다.
    - 성능이 안좋다면 Sloppy Counter 를 이용할 수 있다. Sloppiness 값이 높으면, 성능이 좋은 대신, global count가 지연된다.

## Linked List에서 Lock

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/bf9f4ad0-d2fe-43f1-8911-5a931f28e83f)

## Queue에서 Lock

- 더미노드를 생성하면, 새로운 것 집어넣을 때, head pointer가 바뀌지 않는다.
  
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/edaa2b69-c8a7-43f6-bc6e-ce0280d73683)

## HashTable에서 Lock

- 자료구조 자체가 각각의 bucket들이 서로 영향을 끼칠 수 없도록 설계되어 있기 때문에, lock을 각각의 bucket에 나누어 준다

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2aeb4ccf-d241-4028-88a6-f413b0ad7938)

# Condition Variable

- **`일종의 대기열`**이라고 생각하면 된다.

예를 들어, 자식의 일이 다 끝날 때까지 부모 쓰레드가 실행되는 데에는 두가지 방법이 있다. 

- 부모 뜨레드가 자식 뜨레드 일이 다 끝날 때까지 대기 (while , spin lock)

→ 이는 CPU time 을 낭비한다. 

- 부모 뜨레드가 잠자다가 깨운다 (sleep 후 다시 ready)

→ 이 방법을 Condition variable로 할 수 있다. 

## 구현 방법

```c
pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);
pthread_cond_signal(pthread_cond_t *c)
```

### Wait

해당 함수를 호출한 스레드가 **특정 조건이 만족**되어 신호를 받을 때까지 스스로 잠들고자 할때

→ mutex를 이용해 critical section이 보호 받는다. 

### Signal

condVar에 잠들어 있는 스레드 하나를 깨우고자 할 때(스케줄링 되기를 기다리기 위한 **Ready queue**로 옮기고자 할 때) 사용한다.

### 가장 중요한 문장

<aside>
💡 wait()가 호출될 때 mutex는 잠겨있었다고 가정한다. wait()의 역할은 락을 해제하고 호출한 쓰레드를 재우는 것이다. 어떤 다른 쓰레드가 시그널을 보내어 쓰레드가 깨어나면, wait()에서 리턴하기 전에 락을 재획득해야 한다.

</aside>

→ **wait과 signal이 항상 critical section 안에서 호출됨을 전제로 한다. 깨어나면, 다시 락을 재획득해야한다.** 

### Mesa semantics (↔ Hoarse semantics)

wait하는 thread는 실제 run할 때, 컨디션에 충족이 안될 수 있다. 따라서 state 자체가 안바뀐다는 것을 보장할 수 없다. signal and continue 즉, 시그널 주고 자기꺼 한다. 성능상의 이슈로 mesa를 더 많이 쓴다. 

반면에 Hoarse semantics는 signal and wait. 

### 예시 T1, T2

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/bfe6b06b-fb8a-4b5b-bf37-ae6bd184db4b)

- 해당 방식은 wait하려면 mutex 앞뒤로 걸고, state를 확인한다. 이는 state를 보호하기 위함이다.
- 하지만 wait를 할 때는 lock을 풀어주므로 다른 thread가 들어오는게 상관이 없다. 내부적으로 자동적으로 lock이 풀린다.
- signal을 주고 깨어날 때 mutex를 다시 잡고 수행한다.


![스크린샷 2023-10-04 오후 11 37 13](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/6a87e026-aad0-4f7b-aac0-c387e25b1fee)

다음과 같이 t1, t2 가 있다고 해보자. 

- t1이 wait하고 mutex를 풀어주는데, t2가 잡고 들어가서 signal 보내면 wake할 때 다시 t1이 들어갈 수 있다. → t1, t2 동시에 critical section에 들어갈 수 있다.
- 이때 무엇이 먼저 실행 되는 것이 좋을까?
- T2먼저 돌고 T1도는 것이 성능상 좋다. (mesa) 왜냐하면 context switch가 한번만 일어나기 때문이다. 단점은 t1의 state가 바뀔 수 있어 만족하지 않을 수 있다.
- 따라서 이를 방지하기 위해, if 대신 while loop 를 쓰는 것이다.

### Broadcast

```jsx
pthread_cond_broadcast(pthread_cond_t *cv); 
```

이는 모든 wait하는 thread를 다 깨워 준다. 

## 실제 parent-child using condition variable

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/170c5e79-b066-4ede-bc5c-0f993d5ddad2)

- 중요한 것
- while : 부모 thread가 깨어나고, 실제로 만족하는 지 확인해야 함. 깨워준 시점과 도는 시점에 상태가 다를 수 있기 때문이다.
- 잘못된 예시
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/54048aaf-6a76-4188-bf9a-b60aa9898d83)
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/b31b97f0-b9c3-40b6-96f1-301786108cb3)
    

### Producer / Consumer Problem

- 버퍼는 일정 크기를 가지고 있다. 이는 shared resource이므로 synchronization을 잘 해줘야 함.
- Producer : 버퍼가 Full인지 체크해서Full 이면 기다렸다가 집어넣음
- 컨슈머:  버퍼가 엠티면 일 안하고, 엠티 아니면 일을 한다.

→ 이를 Condition variable 로 잘 풀 수 있다.

- 문제 코드
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2b9eb44a-4856-4c49-b184-8d1c8262d82a)
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/5d99fe40-0a1c-42f7-9466-925e13f33311)

    
 → `mesa` 방법을 쓰기 때문에, state 보장이 안된다. 따라서 while 루프를 써야 한다. 
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/17f5804e-c239-46a7-b690-2ea2b7812d3e)
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/e7412d4e-28b0-43ad-a752-28bfe68ab51f)
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/465de62b-35dc-4590-ae76-840b0d32c4de)

    
 → 해결책 : condition variable이 같기 때문에, 깨어나는 프로세스가 누구인지 몰라서 생기는 문제이다. 따라서, consumer, producer condition variable을 따로 두어야 함. 
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c81bf0ac-50cc-4fa0-a546-c9824655ca8d)

    
다음과 같이 condition variable을 이용해 해결할 수 있다.!!!
    
    
- 해결 코드
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/f7296007-40f1-4253-bd91-d8ee78cf511b)

    
buffer slot을 넓혀 더 효율적이게 짤수 있다. 
    
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/24743d57-0a85-4268-87a7-be6b40216ddf)

    
최종 코드 이다.





# Semaphore란?


- 컨디션 , 뮤텍스 : thread 의 싱크로 나이즈 primitive
- Semaphore : 프로세스에서의 싱크로 나이즈 → thread 에서도 사용이 가능.

```c
#include <semaphore.h>
sem_t s;
sem_init(&s, 0, 1)

여기서 0은 thread에서 semaphore가 공유된다는 뜻이다. 
만약 0이 아닌 다른 값이면, 프로세스에서 공유된다는 뜻임. 
마지막 인자는 위에서 얘기한 세마포어의 상태값을 의미한다.
```

# Semaphore API

```c
int sem_wait(sem_t *s) {
  decrement the value of semaphore s by one
  wait if value of semaphore s is negative
}

int sem_post(sem_t *s) {
  increment the value of semaphore s by one
  if there are one or more threads waiting, wake one
}
```

- `sem_wait`은 세마포어의 값을 1 내린 다음 그 값이 음수인 경우 wait 상태에 들어가며, `sem_post`는 세마포어의 값을 1 올려준 다음 대기중인 스레드가 있으면 그중 하나를 깨워주는 역할을 한다.
- 1로 주면 락이 되고, 0으로 초기화 하면 order를 지켜주는 방식(condition variable), n이면 counting 할 때 쓰인다.
- 음수 값의 의미 : - 뺀 나머지 → thread 웨이트 개수 (ex -2 => 2개의 thread가 기다리고 있다.)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/0f4a742d-4727-46f5-9f3f-59b5e930b403)

# Semaphore 용도

## Lock으로 Semaphore 예시

## Condition variable로써의 Semaphore 예시

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/7a67f568-9151-4218-9172-f23fd8826658)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/d379c193-8f3b-4de1-9742-9a5fbde62c36)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/af7c9099-8bb1-4d4a-b974-3b67aef034b0)

다음과 같이 child가 먼저해도, parent가 먼저 실행되어도, Order를 보장한다. 

## Producer / Consumer Problem (bounded-buffer)

- 문제의 코드
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2e0fc4c6-73fd-4e68-ad1a-ca0863e8b4b2)
    
문제의 코드이다. producer에서 sem_wait → put 하는 과정에서 mutual exclusive가 보장되지 않는다. 따라서 wait, post하는 곳에 각각 lock을 걸어준다. 
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/f6da2eec-e03e-4c8c-8791-ea6d1ee9e72b)
    
하지만 이또한 문제가 있다. 왜냐하면 `consumer`가 c0, c1을 거쳐서 기다리게 되고, `mutex`를 놓고 있지 않으므로 `producer`가 계속 대기 하고 있는 `deadlock`이 걸린다. 
    

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/ca7446af-1a9e-4df2-8bac-6f686cbaf0fc)

따라서 다음과 같이 순서를 바꾸면, deadlock을 면할 수 있다. 

## Reader Writer Locks

락에 의해 성능이 떨어지는 것을 최소화 시키는 것이 중요하다. 따라서 read할 때는 read를 가능하게 한다. 성능향상을 위해 reade와 writer를 분리한다. 

두가지 방법이 있다. 

- Reader 우선권 : read할 때, read Lock을 얻을 수 있다. writer가 starve
- Writer 우선권 : write할 때, read를 하지 못한다. reader가 starve

```c
typedef struct _rwlock_t {
  sem_t lock; // binary semaphore (basic lock)
  sem_t writelock; // allow ONE writer/MANY readers
  int readers; // #readers in critical section
} rwlock_t;

void rwlock_init(rwlock_t *rw) {
  rw->readers = 0;
  sem_init(&rw->lock, 0, 1);
  sem_init(&rw->writelock, 0, 1);
}

void rwlock_acquire_readlock(rwlock_t *rw) {
  sem_wait(&rw->lock);
  rw->readers++;
  if (rw->readers == 1) // first reader gets writelock
  	sem_wait(&rw->writelock);
  sem_post(&rw->lock);
}

void rwlock_release_readlock(rwlock_t *rw) {
  sem_wait(&rw->lock);
  rw->readers--;
  if (rw->readers == 0) // last reader lets it go
  	sem_post(&rw->writelock);
  sem_post(&rw->lock);
}

void rwlock_acquire_writelock(rwlock_t *rw) {
	sem_wait(&rw->writelock);
}

void rwlock_release_writelock(rwlock_t *rw) {
 	sem_post(&rw->writelock);
}
```

예시) 

- W1 R1 R2
    - R1 → write lock 에서 기다린다.
- R1 R2 R3 W1
    - R1 R2 R3 W1으로 실행이 된다.
- R1 R2 W1 R3 W2 W3 R4 W4
    - R1, R2 같이 수행
    - W1 기다림
    - R3 같이 수행
    - W2 , W3 기다림
    - R4 같이 수행
    - R1, R2, R3, R4, W1, W2, W3
    

## Dining PhiloSopher

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/fbd18769-da40-4860-9aca-dd552d9f0628)

```c
while (1) {
  think();
  get_forks(p);
  eat();
  put_forks(p);
}
```

철학자들은 다음과 같은 과정을 거친다. 

```c
void get_forks(int p) {
  sem_wait(&forks[left(p)]);
  sem_wait(&forks[right(p)]);
}

void put_forks(int p) {
  sem_post(&forks[left(p)]);
  sem_post(&forks[right(p)]);
}
```

포크를 쥔다는 것은 자기 말고 다른 누구도 그 포크를 사용할 수 없음을 의미하므로, 그 자체로 lock을 걸어 놓는 것과 같다고 생각할 수 있다. 

### 문제점

- DeadLock

만약 철학자들이 모두 왼쪽의 포크를 집는다면, 오른쪽 포크를 못 얻으므로 Dead Lock에 걸린다. 이는 `Hold And Wait`

### 해결법

이는 **의존성 순환** 때문에 발생하는 문제이다. 따라서 의존성을 끊어준다. 

```c
void get_forks(int p) {
  if (p == 4) {
    sem_wait(&forks[right(p)]);
    sem_wait(&forks[left(p)]);
  } else {
    sem_wait(&forks[left(p)]);
    sem_wait(&forks[right(p)]);
  }
}
```

Dead loc k이 걸리진 않지만, 본인 왼쪽 오른쪽 사람이 계속 밥 먹으면 `starvation` 생겨서 죽을 수도 있다.

---

## Zemaphore (Semaphore using Condition variable)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/e26ac00b-4b4d-49d5-9f1c-b21201a163f9)

마이너스가 될 수 없는 세마포어라고 생각하면 된다. 

<aside>
💡 정리하자면! Condition variable은 semaphore를 구현할 수 있다. Semaphore 또한 Condition Variable을 구현할 수 있다.

</aside>

---

# DeadLock

cf ) 실무에서 Deadlock이 왜 일어날까? 

- 컴포넌트 간의 복잡성 때문에 (vm은 file system을 접근하고 싶은데, 파일 시스템은 disk에서 메모리에 얹히기 위해 virtual memory에 접근해야 함)
- encapsulation  때문에 (우리가 만들지 않은 collection, library를 통한 문제가 생길 수도 있다. )

## Condition for DeadLock

1. **Mutual exclusion :** 각 스레드는 스레드가 얻으려는 자원에 대한 제어권을 한 순간에 혼자만 사용하기를 요청한다.(lock을 얻는다던지,,)
2. **Hold-and-wait:** 각 스레드는 자신에게 이미 할당된 자원(이미 acquire한 lock 등)을 가지고 있는 상태로 또다른 자원(이어서 얻기를 원하는 lock)을 얻기 위해 대기할 수 있다.
3. **No preemption:** (lock 등의) 자원을 쥐고 있는 스레드로부터 해당 자원을 강제로 빼앗을 수 없다.
4. **Circular wait :** 각 스레드는 체인에 소속된 다음 스레드가 요청한 하나 이상의 자원(lock 등)을 가지고 있는 circular chain에 소속되어 있다.

4가지 조건을 하나라도 만족하지 않으면 DeadLock은 발생하지 않는다. 

그렇다면 어떻게 `Prevent` Or `Avoid` Or `Detect/Recover` 할까 ? 

## Prevention : Circular Wait

`Parital Ordering`을 이용하여 끊는다. 

L1을 L2보다 항상 먼저 얻게 함으로써 아까와 같이 lock을 얻는 순서가 꼬이는 일을 방지하는 것이라 할 수 있겠다.

L1, l2, .. , L100까지 넘어가면 연상하기가 힘들어질 것 같다. 그럴 때에는 다시 Dining philosophers 문제를 떠올리면 좋을 것 같다. 포크를 쥐는 순서를 정해서 철학자들에게 강제한다면 당연히 해당 포크를 집을 철학자가 먼저 식사를 할 수 있겠지? 이렇게 모든 lock acquire order를 저장하는 방법을 **total ordering**이라 한다.

예를 들어 여기서는 L1->L2, L2->L3, ... 와 같이 ordering의 일부만 저장해놓는 **partial ordering**을 사용할 수도 있다.

## Prevention : Hold - And - Wait

```c
pthread_mutex_lock(prevention); // begin acquisition
pthread_mutex_lock(L1);
pthread_mutex_lock(L2);
...
pthread_mutex_unlock(prevention); // end
```


Lock 이 언제 사용될지 모를 때 미리 잡아 놓는다. 미리 어떤 락이 필요한지 다 알고 있어야 한다. 

이는, dead lock 은 안걸리지만  Concurrency가 줄어들고 락 잡는 크기 최소한으로 줄여야 한다.

## Prevention : No Preemption

```c
top:
pthread_mutex_lock(L1);
if (pthread_mutex_trylock(L2) != 0) {
  pthread_mutex_unlock(L1);

  goto top;
}
```

위의 코드는 아래와 같이 작동한다.

1. 일단 L1을 얻는다. 여기서 실패하면? 다른 스레드가 L1을 먼저 쥔 것이다.

2. 다음으로 L2를 얻으려 시도하는데, 이때 성공하면 0이 아닌 값을 반환한다.

3. L2를 얻는데 성공하면 L1과 L2를 온전히 얻은 것이므로 그냥 반환하면 된다.

4. 만약 실패했다면, 쥐고 있던 L1을 다시 내려놓고 `top`으로 돌아가 단계 1부터 다시 실행한다.

보면 알겠지만 위 코드의 각 명령어가 atomic하게 실행되는 것이 보장되지는 않기 때문에 두 스레드가 저 각각의 lock acquire 코드로 L1과 L2를 얻으려 하면 서로 각자 L1과 L2를 쥐었다가, 놓았다가만 영원히 반복하는 **livelock** 문제가 발생할 수도 있다.

## Prevention : HardWare Exclusion

Lock 을 잡지 않고, prevent 하는 것 찾아 보자.

```c
int CompareAndSwap(int *address, int expected, int new) {
  if (*address == expected) {
    *address = new;
    return 1; // success
  }
  return 0; // failure
}
```

```c
void AtomicIncrement(int *value, int amount) {
  do {
  	int old = *value;
  } while (CompareAndSwap(value, old, old + amount) == 0);
}
```

---

## **Deadlock Avoidance via Scheduling**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/f97397aa-7156-4b9a-9c62-ddde2ba75461)

다음과 같이 스케줄링 방식을 통해 피할 수 있다. 

## Detect And Recover

돌려보다가 deadlock 발생 할 때 detection 해서 방식을 취한다. 사이클 깨주는 액션(Leave, or 종료)

