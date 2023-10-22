## 자바에서 HashMap에 대해서 설명해주세요

Map은 key - value 쌍 형태로 이루어진 자료구조입니다.

그 중 HashMap은 key를 자바의 `hashCode()` 메서드를 활용하여 key 값을 해싱하여 저장하는 자료구조입니다.

내부적으로 배열을 가지고 있고 key의 해시 값을 배열 인덱스로 하여 값을 저장합니다.

이러한 원리로 O(1)의 시간 복잡도로 데이터를 가져올 수 있습니다.

Hash 함수의 특징으로 인해 key 값에 해시 충돌이 발생할 수 있는데 이를 방지하기 위한 체이닝 방법이 있습니다.

## 자바에 또 HashTable과 HashMap의 차이가 어떻게 되나요?

HashTable은 HashMap이 등장하기 전 사용하던 자료구조로 HashMap과의 차이점으로는 HashMap에는 보조 해시 함수가 존재하여 해시 충돌을 줄이지만 HashTable은 보조 해시 함수가 존재하지 않기 때문에 해시 충돌이 발생할 가능성이 HashMap에 비해 큽니다.

다만 HashTable의 경우 동기화 키워드인 `synchronized` 가 메서드에 포함되어 있어 동기화가 되는 장점이 있습니다.

하지만 프로그래밍할 때 동시성을 고려할 때 `HashTable`보다는 `ConcurrentHashMap` 을 더 자주 쓰게되는데 그 이유는 HashTable은 메서드 전체에 `synchronized` 가 되기 때문에 성능이 떨어지는 단점이 존재하여 엔트리 단위로 `synchronized` 하여 동기화하는 `ConcurrentHashMap` 을 더 자주 사용하게 됩니다.

## HashSet의 내부 구현이 어떻게 되나요?

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/88527476/63df875a-66b6-4edb-a79f-9f479b120201)

HashSet은 내부적으로 HashMap을 가지고 있고, HashMap의 Key 값을 활용하여 중복 데이터를 검사할 수 있게 됩니다.

HashMap의 value 값 까지 확인하지 않아도 되므로 value에는 `PRESENT`라는 의미없는 상수를 집어넣어 데이터를 관리합니다.

## Java의 GC에 대해서 설명해주세요

java는 개발자가 직접 메모리를 관리하지 않기 때문에 가비지 컬렉터가 존재합니다.

자바가 컴파일되는 환경인 JVM에 존재하는데, 더 이상 사용이 되지 않는 할당 영역, 즉 garbage를 회수하는 역할을 한다. 

이 수행 방식을 더 자세히 말씀을 드리면 이 동적할당은 기본적으로 heap 영역에서 수행이 되는데, 이 heap 부분을 young과 old 영역으로 나누고 young은 다시 eden, survivor1, survivor2로 나눈다. 

이 상태 에서 동적할당이 일어나면 그 할당 영역은 young 영역 속 eden에 최초로 위치한다. 

그러다가 eden이 가득 차게 되면, 가비지 컬렉터가 실행되어 더 이상 참조되지 않는 것을 삭제하고 남은 것은 survivor1으로 옮긴다. 

young에서 이러한 작업이 반복되다가 참조가 오랫 동안 지속된 영역이 생기면 그것은 old 영역으로 간다. 그러다가 old 영역까지 가득차면 거기서도 mark and sweep 동작이 일어나는 방식이다.

그리고 GC 작업은 현재 프로그램의 모든 작업이 멈추는 stop the world가 발생하기 때문에 이러한 GC 작업의 속도를 빠르게 하는 것이 중요합니다.

## 자바의 메모리 영역에 대해서 설명해주세요

Method Area, Heap Area, Stack Area, PC register, Native method stack으로 구분됩니다.

Method Area는 모든 쓰레드가 공유하는 메모리 영역으로 클래스, 인터페이스, 메서드, 필드, static 변수 등의 바이트 코드를 보관합니다.

Heap Area는 모든 쓰레드가 공유하는 공간으로 new로 생성된 객체들을 모아두는 공간입니다. 동적으로 데이터가 할당되기 때문에 GC가 메모리 관리 하는 영역입니다.

Stack Area는 메서드 작업에 필요한 메모리 공간을 제공합니다. 메서드 호출마다 메서드 만을 위한 공간을 생성하고 호출된 메서드의 파라미터, 지역 변수, 리턴 값 등 메서드 작업에 필요한 값들을 임시로 저장합니다.

PC register 영역은 쓰레드가 어떤 부분을 어떤 명령으로 실행해야할 지에 대한 기록을 하는 부분으로 현재 수행중인 JVM 명령의 주소를 가집니다.

Native Method Stack은 자바 외 언어로 작성된 네이티브 코드를 위한 메모리 영역입니다.
