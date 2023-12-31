## File

---

프로그램과 데이터 등의 정보의 모음

파일의 형태: text file, source file, executable file, image file 등 있다.

OS 입장에서 file은 메인 메모리가 아닌 **2차 저장소(HDD, SSD)에 저장된 데이터/프로그램**을 다루는 기본 단위로 볼 수 있다.

유저 입장에서 file은 논리적으로 저장되는 기본 단위로 볼 수 있다.

유저 입장에서 논리적인 파일을 실제 저장장치에 매핑해주는걸 OS가 담당한다.

파일은 파일 → 블록 → 레코드 → 필드 순서로 세분화 된다.

> 레코드 : 파일을 구성하는 요소
필드 : 의미 있는 데이터의 가장 작은 단위
> 

## File Naming

---

파일에는 Directory의 root에서 시작하는 Path를 포함되어 있다.

특정 파일의 경로명은 Directory 이름과 파일 이름으로 구성된다.

다만 모든 파일에 대해서 root 부터 현재 파일 이름까지 일일히 다 적으면서 접근하는게 불편하여 상대 경로라는 개념도 제공해준다.

1. 절대 경로: 모든 경로 포함 (C:\Program Files\Study\OS)
2. 상대 경로: 현재 위치에서 타겟까지 상대적 위치만 명시 (./../src/study/os/hello.java)

## File attribute

---

시스템에서 파일을 관리하는 데 필요한 정보, 메타 데이터

FCB는 디스크에 저장하여 데이터가 날라가지 않게 둔다.

파일 속성은 파일 시스템에 따라 구성이 다르다.

## File type

---

파일의 type으로 파일 내부 구조 형태를 짐작할 수 있다.

OS는 파일 시스템이 지원 가능한 파일 구조를 정의하고 해당 파일 구조에 맞는 연산을 제공한다.

- 일반 파일
    - 우리가 흔히 보는 파일들, text나 binary 형태로 데이터 표현
- 디렉토리 파일
    - 모든 유형의 파일에 접근할 수 있는 정보를 포함하지만, 실제 파일 데이터는 아니다.
- 특수 파일
    - 시스템 장치를 정의하거나 프로세스로 생성한 임시 파일
    - 파이프, 블록, 문자 등이 이에 해당

## File Descriptor

---

파일에 엑세스하는 동안 OS에 필요한 정보를 모아 놓은 자료구조

하나의 파일에 하나의 File Descriptor가 생성된다.

파일 시스템이 관리하여 사용자가 직접 참조 불가하다.

디스크에 저장되어 있르며 파일을 열면 메모리에 복사된다.

파일을 닫거나 프로세스가 종료되면 폐기된다.

## File Access

---

### Sequential Access

파일에 있는 정보는 일반적으로 레코드 단위의 순서로 처리

읽기 동작: 파일의 다음 부분을 읽고 파일 포인터 증가시킨다.

쓰기 동작: 파일 끝에 내용을 추가하고 포인터를 파일 끝으로 이동

### Random Access

모든 블록을 직접 읽거나 쓸 수 있다.

읽기나 쓰기의 순서가 없음 → 장점

### Index Access

레코드를 찾기 위해 Index를 찾고 대응되는 포인터를 얻는다. 

크기가 큰 파일에서 유용하다.

## File System

---

논리적인 저장 단위인 파일을 정의하고 메모리에 매핑시키는 기능 제공

파일과 디렉토리로 구성된다.

디스크 저장소(HDD, SSD)를 저장 공간 할당하는 목적으로 사용한다.

User가 파일을 생성, 수정, 삭제할 수 있도록 지원

파일을 각 Application에서 가장 적합한 구조로 사용할 수 있게 지원

## File System 기능

---

- 파일 구성
- 파일 관리
- 보조 메모리 관리: 다양한 형태의 저장 장치에 대한 입출력 지원
- 파일 무결성 보장
- 장치 독립성 유지
- 백업, 리커버리
- 파일 보호
- 파일 보안
- 정보 전송

## File System의 구조

---

### 논리 파일 시스템

디렉토리 정보 보호 및 보안 담당

### 파일 - 구성 모듈

파일의 논리 블록 주소를 물리 블록 주소로 변환해준다.

디스크의 가용 공간 파악

### 기본 파일 시스템

적절한 장치 드라이버 호출

### 입출력 제어기

주변 장치, 장치 제어기 등과 직접적으로 통신하여 입출력 수행

## Directory

---

파일들의 이름과 위치 정보를 담은 파일

사용자 데이터를 저장하지 않는다.

파일 이름, 파일 인덱스의 내용을 포함하는 파일로 구현한다.

계층적으로 구성된다.

디렉토리의 공간 할당, 관리 방법은 파일 시스템의 효율성과 신뢰성에 큰 영향을 준다.

## Directory 구현 방법

---

### 선형 리스트

디렉토리에 파일 이름, 포인터들의 선형적 리스트 구성하여 파일의 생성과 삭제를 진행

선형 탐색을 해야 파일을 찾을 수 있다.

→ 탐색 비용 크다. 오버헤드 증가 및 시스템 성능 저하

리스트를 정렬하여 이진 탐색 방법도 고려할 수 있다.

→ 정렬 상태를 유지해야 되는 단점 존재, 이진 연결 트리로 해결 가능

### Hash Table을 이용한 디렉토리 구현

key-value로 찾는 방법

탐색 시간을 확실하게 줄일 수 있다.

Hash Table 크기가 고정되어 있어서 파일 수가 한도를 넘어가면 새로운 Hash Function을 만들어줘야한다.

## Directory 구조

---

### Single Level Directory

가장 간단한 구조

모든 파일이 동일한 디렉토리에 존재한다.

→ 모든 파일이 유니크한 이름을 가져야한다. (하나의 디렉토리에 모든 파일이 있기 때문에)

### Two-Level Directory

Root 아래에 Sub Directory를 두는 구조

Root는 MFD(Master File Directory)

Sub는 UFD(User File Directory)

사용자 이름과 파일 이름은 root에서 leaf까지 path로 정의한다.

**모든 파일의 경로명이 유일하면 된다!**

### Tree Directory

모든 디렉토리가 sub 디렉토리를 생성할 수 있는 구조

모든 파일의 경로명이 유일해야된다.

파일이나 디렉토리의 공유를 금지한다.

### Acyclic Graph Directory

트리 구조를 확장

파일이나 디렉토리 공유를 허용하지만 사이클이 생기면 안된다.

링크로 연결한다.

복잡해지는 단점이 있다.

### General Graph Directory

위의 Acyclic Graph Directory에서 순환이 허용되는 디렉토리

대신 순환으로 발생하는 이슈들 관리가 중요하다.

## 파일 디스크 할당 방법

---

### 연속 할당

파일을 디스크의 연속적인 block에 할당하는 방법

Sequential access에 효과적, Direct access가 가능하다.

HDD 특징을 생각할때 디스크 탐색이 최소화되는 장점이 있다.

단, SSD에서는 별 의미가 없다.

외부 단편화 발생 가능

### Linked 할당

파일을 디스크 블록들의 리스트에 연결

디스크 블록들은 연속적이지 않아도 된다. → LinkedList 특징을 생각하면 됨

외부 단편화가 없다. 대신에 내부 단편화 발생 가능

블록 크기를 잘 설정해야 된다. (탐색 시간과 내부 단편화 사이의 trade-off)

block 내부에 포인터 공간이 있어야 된다.

탐색 시간이 길다.

순차 access에 적합

직접 access는 시간 복잡도가 좋지 않음 (LinkedList에서 get() 함수 시간 복잡도 생각하면 됨)

### Indexed 할당

모든 포인터를 index block에서 관리하여 직접 access를 지원한다.

Paging 방식과 유사하다.

직접 access 성능이 좋다 (index block 1번, 파일 1번으로 총 2번으로 모든 block access 가능)

구현이 쉽고 외부 단편화 없다는 장점

이 방식도 block 크기에 따른 trade-off 존재
