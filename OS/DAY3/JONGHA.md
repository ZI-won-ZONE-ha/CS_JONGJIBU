cf) CPU 가상화 = **Time-sharing** = **multitasking** 

cf) 메모리 가상화 = **Multi-programming** 

# Prodcess Address Space

Os는 physical memory를 추상화한다.

`Runtime`때 `virtual memory` 가 `physical memory`에 매핑된다. 

현재 그림은 하나의 프로세스에 해당되는 32bit Process Address Space라고 보시면 됩니다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/b19ac17e-6e5a-4190-8919-ecc4da7d80e2)

- 커널 또한 virtual address mapping을 쓰지만, 커널 메모리는 physical address에 linearly 하게 저장이 됩니다.

## User Stack And Kernel Stack

일반적으로 프로그램을 돌릴 때 두가지 타입의 스택이 쓰입니다. 

- 유저 모드가 실행 될 때 `user stack`이 쓰인다.
- 시스템 콜을 통해 커널 모드가 실행되면 `kernel stack`이 쓰인다.

각각의 장소는 어디일까요? 

- `User Stack`은 Process address Space의 stack에 위치합니다.
- `Kernel Stack`은 실제 피지컬 메모리의 pcb 의 시작부터 4K 떨어져 있는 위치에 위치합니다.

# Physical Memory와 Virtual Memory의 매핑

process는 `4G` 즉, `32bit`라고 가정한다. 

만약 프로세스에서 코드가 돌다가 인터럽트가 걸리면, 커널 영역의 PCB에 가게 됩니다. 실제 피지컬 메모리의 PCB에 매핑되는 구조입니다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/73eda550-ae2e-4e9e-a372-2c8f5f70fa5c)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/f560f1e5-8baf-495a-acde-9e9b6da39466)

PCB의 4KB 떨어진 시점부터 커널 시점이 됩니다. 이는, PCB 위치에서 마지막 12 비트를 찾으면 바로 찾을 수 있습니다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/5175e9ff-6644-4e94-b273-0c5f5aa4b1d0)

<aside>
💡 일대일 때 커널 스레드가 한정되었다고 하는 것은 여기서 말하는 커널 space가 user space 보다 작어서 일까요?

</aside>

### Logical vs Physical Address Space

- Run time 에 virtual → physical로 바꾸어 주는 hardware를 MMU(memory management unit) 이라고 합니다.
    - MMU의 종류에 따라 Mapping Algorithm이 달라진다. **가장 널리 쓰이는 방법은 Paging입니다.**
- Logical : CPU에 의해서 만들어 집니다.
- Physical : memory unit에 의해 보여집니다.

## Virtual-Physical Memory의 매핑 방법

**32-Bit Linux Machine의 Maximum Memory Size는 4GB**이다. 즉, **사용자 입장에서 가장 크게 사용할 수 있는 크기는 1G는 Kernel(OS)것**이므로 **3G밖에 되지 않는 것**이다. **그래서 보통 Process의 Logical Memory를 Disk에도 맵핑시키는 것**이다.

- **Logical(Virtual)** Memory를 똑같은 크기의 Block들로 나눈다.
    - 이를 **Page**라고 부른다.
        - **2^n승 크기**로 설정하며, **일반적으로 512B에서 8192B**까지 나뉜다.
            - 0.5K ~ 2K Size
- **Physical** Memory도 고정 사이즈 Block들로 나눈다.
    - 이를 **(Page) Frame**이라 부른다.
- Page와 frame은 보통 동일한 사이즈 입니다. n Pages, M free frame이 있다고 쳤을 때, 모든 프로그램을 돌리려면 n=m 이어야 합니다. n> m 이라면 일부분의 virtual memory를 disk 에다 놓습니다.
- 각각의 프로세스는 logical → physical 변환을 위해 page table을 가지고 있습니다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/3eddeced-5f91-496e-8a82-fcb114c27338)

# Memory API

이는 간단히 하고 넘어가겠다. 

- Malloc : 동적으로 생성되는 메모리는 바로 physical memory를 가져오지 않고, virtual memory를 가져온다.
- BRK : malloc이 쓰는 system call, 어드레스 상에 힙 스페이스 확장하도록 요청한다.
- mmap:  힙과 stack 사이 비어있는 공간에 특정 영역을 할당하도록 os에게 요청하는 것이다.

## Free Space Management

메모리 관리에는 여러가지 메커니즘이 존재한다. Splitting, Coalescing, Traking Free Region,  Best Fit, Wort Fit, First Fit, Next Fit , Buddy Allocator 등이 존재한다. 

이에 관련해선 지금 당장 다뤄야 할 내용은 아닌 것 같으므로,  나중에 정리하도록 하겠다. 

# Address Translation

Memory Virtualization 

- 각각의 프로세스에게 메모리 스페이스를 제공하는 것과 같은 환상을 줌.
- 각각의 프로세스가 모든 메모리는 이용하는 것 같은 느낌을 줌
- 메모리 가상화에 여러 전제를 붙인다.
    - 프로세스는 연속적인 Physical 메모리 공간을 사용한다.
    - 프로세스가 필요한 메모리는 항상 실제 Physical 메모리보다 작다
    - 모든 프로세스는 같은 크기의 메모리를 사용한다.
    - → 이는 실제와는 거리가 멀지만, 쉽게 이해하기 위해 세운 가정입니다.

## Translation 방법

### Time Sharing

CPU 가상화 하는 것 처럼 time-sharing을 하는 것이 어떨까??? 

→ process가 돌지 않는것을 disk 에다가 올려 놓자.

→ 단점 : performance가 너무 안좋아.. 이는 특정 메모리 매번 내리고 올리는데, Overhead가 있다. 

→ 철회! 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/29a696be-8ce2-423f-acbc-ee48f90e7354)

### Static Relocation

쉽게 말해서 OS가 static하게 올라간 주소를 바꿔주는 것이다. 

→ 문제점 : 메모리가 load 되고, 수행 중에 프로텍션이 없는 것이 큰 문제이다. 한번 loading 된 다음 다른 곳으로 replace 불가능하다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2a31ce84-09f4-4ca1-be99-24e7086ed68f)

→ 따라서 동적으로 하는게 좋아보인다….

### Dynamic Relocation - MMU

드디어 나오는 `MMU` 방식! 이는 메모리 액세스 할 때마다 translation을 Hardware의 도움을 받는 것이다. MMU는 translation을 위한 `Base Register` 와, Protection을 위한 `BOUNDS Register`를 따로 둔다. (CPU 에 존재하는 하드웨어이다.) 

- Base Register : loading 하고자 하는 Process의 Location (smallest physical Address)
- Bounds Register : 프로세스의 virtual Address의 사이즈. 이는 에러를 체크해주기 위함이다.

ex) Base + Logical > Bounds ⇒ ERROR 

- Base + Bounds

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/868df37f-b797-4178-97f6-90ba324f88eb)

1. Kernel, User 모드가 구분되어야 합니다.
2. Base, limit 레지스터를 제공해야 합니다.
3. Base, limit 레지스터로 가상 주소를 변환하는 능력, 잘못된 가상 주소 공간을 체크하는 능력을 제공해야 합니다.
4. 오류가 발생했을 때 처리할 수 있는 능력이 있어야 합니다.
5. 오류가 발생했을 때 오류를 발생할 수 있어야 합니다.

→ 2, 3, 4, 5 → Kernel Mode에서 진행하여야 한다. 즉 , previleged mode로 진행하여야 한다. 

- 동작 과정
    - 각 프로세스의 PCB에 BASE, BOUNDS Register를 더한다. 즉, Context Switch 가 일어날 때 마다 BASE, BOUND의 값을 세팅해줘야 한다.
    - **실제 메모리 주소 (Physical Memory) = 가상 주소 (virtual Address) + base 레지스터**
    - 이때, 비교할 때 만약 **virtual > Bounds 이다면 에러**이다.
    - Hole : Block of Avaliable Memory : Holes of variaous size are scattered throughout Memory
    - Hole 에다가 memory를 배치하는 여러가지 방법? First-Fit, Best-Fit, Worst-Fit 등이 있다.
    - 50 Percent Rule : n개 블럭이 할당받아지면 절반은 lost된다.

### Fragmentation

하지만 중요한 용어는 따로 정리한다. 바로 `Fragmentation`

이 둘은 정말 한 천백번 교수님이 강조를 하셨다..

- `**External Fragmentation**` : Total memory space는 요청에 맞지만, 연속적이지 않다.

<aside>
💡 → `compaction` 으로 해결 가능하다. free memory를 하나의 큰 block 으로 만든다. 이는 dynamic relocation일때만 가능하고, 실행 시간 일 때만 가능하다. 
**가장 큰 문제는 i/o 버퍼를 특정 위치에 latch 시켜야 함. 버퍼를 움직이면, 디바이스가 알지 못한다.**

</aside>

- `**Internal Fragmentation`** : 할단된 메모리가 요청한 것에 비해 조금 크다.

### Dynamic Relocation 장점과 단점

- 장점
    - Protection 가능
    - MMU 와 few Register → inexpensive implementataion
    - Fast
- 단점
    - Process가 Physical Memory에   프로세스 별로 Continuous하게 올라가야 한다.
    
    → `external fragmentaion` 해결하기 어려움
    
    - address space를 share 하는 것 구현하지 못한다.

<aside>
💡 → 이를 어떻게 해결할 까?  : Segmentation

</aside>

# Segmentation

기존의 문제점 : 연속적으로 할당하다 보면 Memory fragmentation 이 생긴다. **메모리 관리 잘하려면 fragmentation 줄이는게 핵심이다.** 

→ **Segmentation은 OS 에게 segment (Code, Stack, Heap) 들을 Physical Memory의 각기 다른 Part에 올**려 unused한 virtual address space의 낭비를 줄인다. 

## Segmented Addressing

- Logical Address

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/7711ec4c-7c73-420e-9cbc-73b6c9ce8182)

- Segment # : Segment를 선택
- Offset : segment 의 Offset을 선택
- MMU 는 다음과 같은 Segment Table을 가지고 있다.
    - Process 당 PCB에 Segment table이 저장이 된다.
    - 문제점 : 테이블이 커진다면? Context Switch 할 때, OverHead가 크다.
    - → 테이블 메모리에다 넣고, 위치를 Register 에다가 저장해놓는다.
    - → Translation 할 때마다 테이블 한번, 실제 위치 한번 → 2번 Access 해야한다는 단점.

- 예시 )

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/adfafa51-d0f5-475d-9990-07570088aaa6)

- Segment # : `4 segment` = 2 Bit = Segment #
- Offset : 14 - 2 = 12 Bit  = 2^12 = `4K Offset`
- Logical Address : 2^14 = `16K` = 메모리의 크기

이를 만약 Segment 8개로 쪼갠다면? 

- Segment # = `8 Segment` = 3Bit
- Offset : 14 - 3 = 11 = 2^11= `2K` Offset

## Segmentation 장점과 단점

- 장점
    - 기존 Process 별로 Allocation을 Sparse 하게 Allocation 할 수 있다.
    - 서로 공유할 수 있다. (selected Segment)
- 단점
    - Segment가 커지면 External Fragmentation의 문제가 또 생긴다.
    
    <aside>
    💡 **→ 해결법? 페이징을 하자!!!!**
    
    </aside>
    

# Paging

- 정의 Page : 쉽게 말하면, logical address space를 똑같은 크기로 나누자.
- Segment
    
    logical 단위로 쪼개자 (code, stack, heap,…)
    
- 용어
    - Logical : `Page`
    - Logical Page Number = `VPN` = Virtual Page Number
    - Physical : `Frame`
    - Physical Frame Number = `PFN`
- 구현 방법
    - Logical 은 Page로 나누고, Physical은 Frame으로 나누고 테이블을 중간에 둬서 매핑해주자.
- **과정 : VPN → 테이블로 PFN 을 찾는다. → PFN + OFFSET = Physical Address**
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/b2e46995-ef76-4597-a2c3-cd0828134dc5)
    
- 장점 : 크기가 Fixed 되어 있기 때문에, Free Space Management가 쉽게 가능해진다. Flexible해진다. Internal Fragmentation 중간에 비어있는 것이 아닌, 마지막에만 생긴다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2a47b0f3-7672-4f9e-8bc8-09b8e3999db4)

### Page Table 이 어디에 저장될까?

- Page Table은 넓다 → 각각의 프로세스의 가상 memory에 저장된다. 그리고 page table의 주소를 PTBR이라는 Register 가 저장한다.
- 즉 실제 메모리를 액세스 할 때 총 두번의 memory access 과정 필요
    - VPN → PFN 으로 변환할 때 page table 실제로 뒤짐
    - 실제 메모리 access 할 때 뒤짐

### Page Table은 무엇을 가지고 있을까?

- 이는 `Virtual` 을 `Physical` 로 바꾸어주는 자료구조이다.
- ValidBit, Protection Bit, Present Bit, Dirty Bit, Reference Bit 등을 가지고 있다. 이에 대한 설명은 생략한다…

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/26e5f167-b5b0-4730-945d-746f4436aa15)

# TLB

기존의 문제점 : `Fragmentation`을 해결하기 위해 Paging을 도입하였지만, Memory Access 가 증대하였다. 이를 어떻게 해결할까? 

<aside>
💡 정답은 TLB이다.

</aside>

## 정의

MMU 안에 Translation look-aside Buffer라는 TLB라는 Fast-lookup hardware cache가 존재한다.

- TLB는 **full associative method**를 사용한다.

<aside>
💡 Full Associative Method란?

Full Associative Method(전역 연관 방식)은 캐시 메모리에서 데이터를 찾는 방법 중 하나입니다. 이 방식에서는 데이터가 어디에 위치해 있는지에 대한 탐색을 선형 검색으로 수행합니다. 따라서, 데이터를 찾는 데 걸리는 시간은 일정하지만, 검색하는데 필요한 비트 수가 많아집니다. 이 방식은 캐시 메모리의 성능을 향상시키기 위해 사용되지만, 하드웨어 비용이 높아질 수 있습니다.

</aside>

## Translation 과정

전체적인 흐름은 다음과 같다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c1cdef59-ffc9-41cb-b1b1-22df975abe17)

1. VPN을 가지고 TLB에 있는 지 찾는다. 
    1. 있으면 TLB hit
        1. pfn을 찾았으므로, Physical Memory = pft + offset
    2. 없으면 TLB miss이다. 
        1. PTBR을 통해 Page Table을 접근한다.
        2. Page Table에 있는 내용 TLB로 옮겨 놓는다. → 특정 위치 주변에 액세스 하는 확률이 높아진다.

## Cache Locality

- Temporal Locality

한번 액세스 한 거는 미래에 액세스 될 확률이 높다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/a724ba97-99d6-4b79-b376-0ebff91b9f21)

- Spatial Locality

액세스된 address x 주변에 곧 액세스 될 확률이 높다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/49107564-a11d-4e6f-8221-14133ff3c5d0)

## 각기 다른 프로세스마다 TLB를 어케 구분하지?

- 문제점
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/21d48512-1840-4e3b-8239-e5fc9eb746ef)
    

→ 이런 상황이 생길 수 있다. 

<aside>
💡 ASID 라는 BIT 을 두어 해결한다.

</aside>

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/cb42bdee-4730-40e0-91f6-aaacaad8e440)

## Problem : Page Sharing → Replacement Policy

다음과 같은 경우에 P1은 10번 페이지가 101번 피지컬 메모리에 매핑, P2는 50번 페이지가 101번에 매핑되어있어 서로 피지컬 메모리를 공유한다는 것을 알 수 있다. (공간 Sharing)

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/d2b95a6f-f8d0-470a-b548-972df343e9d8)

## 해결 방법 : ReplaceMent Policy

최대한 miss rate 를 줄여 performance를 증가시키는 방향으로 진행하여야 한다.

### LRU  (Least Recently Used)

- 가장 참조되지 않은 entry를 evict한다.
- locality를 이용한다.
- 단점은 젤 오래된 것을 계속 찾아야 한다. 일반적으로는 LRU가 좋다.

### Random

- random으로 evict한다.
- 코너 케이스 행동에 유용하다.

# Page Problem

## Big Table

캐쉬를 통해서 페이지 테이블 Access 하는 시간 줄여놨어도, 페이지 테이블이 너무 크다는 단점 해결해야 한다. 메모리를 너무 많이 먹는다. 

ex) 페이지 offset이 4KB Page가 있고 각 4바이트라고 가정. 그리고 32Bit address space라고 가정

> 페이지 테이블 사이즈는 어떻게 될까?
> 
> 
> 페이지 엔트리는 = (2^32)/(2^12) = 2^20
> 
> 페이지 테이블 사이즈는 2^20 * 4바이트 = `4MB` 바이트 
> 

→ 진짜 너무 크다 

→ `Offset`을 더 늘려서 페이지 테이블 더 작게 안될까?  페이지 당 크기를 `16KB`로 늘려보자

> 페이지 엔트리 = 2^32/ 2^14 = `2^18`
> 
> 
> 페이지 테이블 사이즈 2^18*4 = `1MB`
> 
> 페이지 테이블 사이즈는 1/4 줄고, 페이지 당 크기는 늘었다!
> 

→ 하지만 페이지 하나당 크기 키우면은 `Internal Fragmentation` 문제가 생길수가 있다.

## Unused Table

페이지 테이블에서 invalid entry 많을 수 있다. 

→ 따라서 Hybrid 방법으로 Paging과 Segment를 같이 쓰는 방식이 있다네요? 

→ 그런데도 여전히 Heap 영역 같은 경우 중간에 비는 것이 생겨서 waste가 많다고 합니다. 

<aside>
💡 해결법 : 페이지 테이블을 또 다시 페이지를 하자

→ 멀티 레벨 테이블 

</aside>

# Multi-level Table

레벨을 늘릴 수록 **페이지 테이블 크기가 작아지고, Memory Acess 수가 많아진다.** → tradeoff이다.

- Page Directory Entry : Page table을 관리하는 entry이다.
- 어느 테이블이 valid한지 관리한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/106529b2-8cbf-4e4c-be8c-d5e6f33d96ae)

- 장점
    - 쓰이는 페이지만 담아놓을 수 있다.
- 단점
    - 시간과 공간에 대한 trade-off 이다.
    - 복잡도

# Inverted Page Table

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/97bb2120-95a7-45e3-a0e0-8b3979789394)

- 역으로 되어 있다. 즉, 피지컬 메모리에 누가 저장되어 있는지 반대로 갖고 있다….

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c33e6c28-cced-4523-995c-ff57d9686f35)

- 테이블 매번 search 하면서, 몇 번째 프레임에 내 페이지 들어가있는지 찾아야 함.
- page table을 store하기는 쉽다. 각자 가지고 있는 거에 비해서는 사이즈가 줄었지만, 찾을 때 시간이 너무 오래 걸린다.

# Demand Paging

- 실행시간에 필요할 때, Page가 load된다.
- 어떤 페이지를 가져올 것인지가 성능에 영향을 미친다.
- 장점
    - less IO , less memory needed
    - 응답이 빠르다
    - 더 많은 유저 수용

→ Physical Memory가 작으니까, Disk에 일부 올려 놓는다.→ Swap Space

### Swap space and Present Bit

- OS는 page-size단위로 swap space를 지원한다.
- Present bit로 페이지가 Physical memory에 있는지 없는지 판단한다.
    - 1일때 physical memory에 있다.
    - 0일때 디스크에 있다.

# Page Fault

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/912f18aa-6e6d-4733-a581-f2831a6fc6c1)

## 동작과정

다음과 같이 페이지 폴트가 일어난다.

- TLB 확인한다.
- page table 확인한다.
    - present bit 확인
        - memory에 없다 →  trap 을 os에게 건다 = `page fault`
        - 이때 Page fault 가 발생하는 것임
    
    ---
    
- os는 empty page를 찾는다.
    - 만약 못찾으면 있는 것을 빼버린다. (replacement 알고리즘 사용)
- 페이지 fault handler가 페이지를 memory에 disk로 부터 가져온다
    - disk read 한다.
    - page table update 한다.
    - reset page table (set present bit)
- restart instruction

## Demand Paging의 성능

- Page Fault Rate가 성능에 엄청난 영향을 준다.
    - EAT (Effective Access Time)

## Page Replacement Algorithm

페이지 교체 알고리즘에는 여러가지가 있다. 

### FIFO

- 처음 Page부터 evict된다.
- Belady’s Anomaly : 프레임의 Count가 늘려졌음에도, Page Fault가 늘어나는 현상이다.

### Random

- 랜덤으로 evict된다.

### Optimal

- 미래 젤 나중에 access 하는 애 뺴냄
- 단점 : 미래 뭐가 나올지 알아야 함.

### LRU

- 오랫동안 이용하지 않았던 페이지 빼냄
- Counter Algorithm
- Stack Algorithm
- Reference Bit / Byte Algorithm
- Second Chance (Clock) Algorithm

### MFU

- 카운터가 많은 페이지를 victim으로 선정한다. 카운터가 적은 페이지는 access 될 가능성 크고, 그리고 큰 애들은 이미 많이 액세스 되어서 안될 것이다.

### Global vs Local Replacement

모든 frame 중에서 택하는 방식. 또는 자신에 할당된 프레임 중에서 선택하는 방식이 있다. 

<aside>
💡 교체 알고리즘 또한 중요할 뿐만 아니라, frame을 각 프로세스마다 얼마나 할당할 지도 중요하다. 이로 인해 Page Fault rate가 결정되기 때문이다. 할당을 결정하는 algorithm 또한 있지만 다루지 않을 예정이다.

</aside>

## Thrashing

프레임이 충분하지 않으면, 페이지 폴트가 일어난다. 연속적으로 일어나면 페이지 폴트 때문에 프로세스가 도는 시간보다, 디스크 스왑하는 일들이 훨씬 많아 질 수 있다. 이는 Cpu IO 작업이 많아질거고 CPU Utilization이 떨어진다. 

→ 이를 방지하기 위해서는 Page Daemon 같은 것을 해야함. Local Replacement가 이를 조금은 해결해 주는데, 완전히는 해결하지 못한다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/cc838f45-5a7d-4bee-82f0-b8f4b04c317b)

## Prefetching

OS가 access될 것 같은 page를 미리 access해오는 방식이다. 

## Clustering

메모리에서 디스크로 write할 때 한번에 write를 하는 방식이다. 같은 block 안에 있는 연속적인 Page만 가능하다. 

# 참고 자료

[OS - 2.1 (MV) (1) Memory Virtualization](https://velog.io/@junttang/OS-2.1-Memory-Virtualization-MV-1)

[[OS] Paging을 사용한 고정 크기 메모리 관리 및 추상화 - OS 공부 12](https://icksw.tistory.com/148)

[운영체제 10 - Free Space Management [Memory Virtualization]](https://koolreview.tistory.com/93)

[[OS] 메모리 가상화를 위한 주소 변환 (Address Translation) - OS 공부 9](https://icksw.tistory.com/143?category=878876)


