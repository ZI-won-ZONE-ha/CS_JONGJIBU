# 8장 인덱스

# 디스크 읽기 방식

---

데이터 저장 매체는 컴퓨터에서 가장 느린 부분으로, 데이터 베이스 성능 튜닝은 어떻게 디스크 I/O 를 줄이느냐가 관건이다.

- CPU , 메모리 → 전자식 장치
- 하드 디스크 드라이브 → 기계식 장치

## HDD, SSD

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/2331b2e2-1715-4227-90e8-c2d9aa50b814)

- HDD
    - 기계식 하드 디스크
    - 자기 디스크 원판에 의존
- SSD(Solid State Drive)
    - 전자식 저장 매체
    - 원판을 제거하고, 플래시 메모리 장착 → 아주 빨리 데이터 읽을 수 있다. 전원이 공급되지 않아도, 데이터가 삭제되지 않는다. 메모리(D-ram)보다는 느리지만, 기계식 하드 디스크 보다는 훨씬 빠르다.
    - 하드 디스크 보다 용량이 적으며, 가격이 비싸지만, 처리속도는 하드 디스크보다 훨씬 빠르다.
    - 순차 I/o에서는 조금 빠르거나, 거의 비슷한 성능 ↔ 랜덤 I/O 에서는 훨씬 빠르다. → 이는 **DB 같은 경우 랜덤 I/O를 많이 하므로, SSD가 DBMS에 최적이다.**

## 랜덤 I/O와 순차 I/O

- 랜덤 I/O는 읽어야하는 데이터가 물리적으로 **불연속적**으로 있기 때문에 **디스크 헤더를 이동** 시킨 다음 데이터를 읽는 것을 의미한다. 이때 디스크 헤드를 이동시키는 시간을 Seek time이라고 한다.
- 순차 I/O는 읽어야하는 **데이터가 연속적**으로 있어 쭉 읽기만 하는 경우를 의미한다.
- 랜덤과 순차의 차이
    - 시스템 콜 요청, 디스크 헤드 위치 차이: 랜덤이 더 많은 페이지 access 하므로 시스템 콜 더 많이 요청 → 그만큼 디스크 헤드 위치 이동
    - 부하 : 여러번 이동하므로, 랜덤이 작업 부하가 크다.
    - **디스크 성능을 결정하는 요인 : 디스크 헤더의 위치 이동 없이 얼마나 많은 데이터 한번에 기록하느냐.**

**→ 쿼리를 튜닝한다는 것은 순차를 랜덤으로 바꾸는게 아닌, 랜덤 I/O를 최대한 줄이는 것을 말한다.**

cf ) 큰 테이블의 레코드를 대부분 읽을 때는 인덱스를 사용하는 레인지 스캔보다, 풀 스캔을 때린다. → 즉, 랜덤 I/o보다는 순차 I/O를 쓴다. 이는 훨씬 빨리 많은 레코드를 읽어올 수 있기 때문이다.

# 인덱스란

---

## 정렬

- DBMS의 인덱스는 `SortedList` 자료구조로, 저장되는 칼럼의 값을 이용해 항상 정렬된 상태
- 데이터 파일은 `ArrayList`와 같이, 저장된 순서대로, 정렬 없이 저장.
- 장단점이 존재한다.
    - 인덱스는 저장과정이 복잡하고, 느리지만 정렬되어 있어 원하는 값을 빠르게 찾아옴. → 데이터의 저장 성능을 희생하고, 읽기 속도를 높인다.
- 인덱스를 추가할 때 고려해야 할점.
    - 데이터의 저장 속도를 어디 까지 희생? (INSERT, UPDATE, DELETE)
    - 읽기 속도 얼마나 빠르게 만들어야 해? (READ)

## 인덱스의 종류

- 역할
    - 프라이 머리 키 : 식별자, Null 허용 X, 중복 허용 X
    - 세컨 더리 인덱스 : 전체 - 프라이머리 키
- 알고리즘
    - B-Tree 인덱스 : 일반적으로 많이 사용함. 컬럼의 값을 변형하지 않고, 원래의 값을 이용해 인덱싱 한다.
    - Hash 인덱스 : 칼럼의 값으로 해쉬값을 계산하여 인덱싱 한다. 값을 변형하여 인덱싱 →  **전방 일치 같이 값의 일부만 검색 또는 범위를 검색할 때는 해시 인덱스 사용 불가능**
- 중복 허용 여부
    - 유니크 인덱스 : 단순히 같은 값이 하나만 존재하는지
    - 유니크 하지 않은 인덱스 : 1개 이상 존재할 수 있는지
    - **이는 옵티마이저에게 겁나 중요하다!**
- 그외 기능별..(전문 검색용, 공간 검색용)

# B-Tree 인덱스

---

- Balanced Tree, 가장 일반적으로 사용한다.
- 원래 칼럼 값을 변형 하지 않고, 인덱스 구조체 내에서 항상 정렬된 상태 유지한다.

## 구조 및 특성

- 루트 노드 , 브랜치 노드, 리프 노드로 이루어져 있다.
- 인덱스와 실제 저장된 데이터는 따로 관리 → 리프 노드가 항상 실제 데이터 레코드를 찾아가기 위한 주소 값을 가지고 있다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/a2884f44-6444-4e0b-b84f-19fa5d25e724)

- `DBMS` 에서 삭제된 공간 재활용하여 사용→ 항상 데이터 파일이 Insert된 순서대로 저장되는 것이 아님.
- `InnoDB` 테이블에서 레코드는 클러스터 되어 디스크에 저장되므로 기본적으로 프라이머리 키 순서로 정렬되어 저장된다.
- 클러스터링 : 비슷한 값을 최대한 모아서 저장하는 방식
- InnoDB에서는 프라이머리 키를 주소처럼 사용하기 때문에 논리적인 주소, MyISAM 테이블은 세컨더리 인덱스가 물리적인 주소를 가진다. 따라서 InnoDB 에서는 데이터 파일을 바로 찾아가지 못한다. 프라이머리 키 인덱스를 한번 더 검색한 후, 리프 페이지에 저장되어 있는 레코드를 읽는다.
- → **즉, 모든 세컨더리 인덱스 검색에서 데이터 레코드를 읽기 위해, 반드시 프라이 머리 키를 저장하고 있는 B-tree 한번 더 검색하여야 한다.**
- **프라이 머리 키에서 리프 노드의 데이터는 실제 레코드의 칼럼 값들이며, 세컨더리 인덱스 페이지에서는 프라이머리 키값을 가진다.**

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/30c05fb3-736b-4fe5-9eff-beeb09686eca)

- 참고
    
    테이블의 프라이머리 키에 대해서만 적용된다. 즉 프라이머리 키값이 비슷한 레코드끼리 묶어서 저장하는 것을 클러스터링 인덱스라고 표현한다. 이는 프라이머리 키 값에 의해 물리적인 저장 위치가 바뀌어야 한다는 것을 의미한다. 따라서 인덱스 알고리즘이라기 보다 테이블 레코드의 저장 방식이라고 볼 수 있다. 클러스터링 인덱스는 프라이머리 키 기반의 검색이 매우 빠르며, 레코드 저장이나 프라이머리 키 변경이 상대적으로 느리다.
    

## 인덱스 키 추가

- B-tree 에 저장된 때 저장될 키 값을 이용해 B-Tree상의 적절한 위치를 검색하고, 결정되면 키 값과 대상 레코드의 주소 정보를 B-Tree 리프노드에 저장
- 만약 리프 노드에 저장 못한다면, 리프 노드를 분리하여야 한다. 상위 브랜치 노드까지 범위 넓어짐. → 쓰기 작업 코스트가 크다.
- 레코드 추가 비용 :1, 인덱스 키 추가 비용 : 1.5 라 가정 → 이 a때의 비용은 CPU, 메모리 처리 시간이 아닌, 디스크로부터 인덱스 페이지 읽고 쓰기 비용
- InnoDB에서는 인덱스 키 추가 작업을 지연시킬 수 있지만, 프라이 머리 키와 유니크 인덱스 경우에는 중복 체크를 해야 하기 때문에 즉시 추가 OR 삭제해야 한다.

## 인덱스 키 삭제

- 리프 노드에 삭제 마크를 한다.  (공간을 재활용하거나 방치)
- InnoDB에서 지연처리 가능 (마킹 작업은 디스크 쓰기 필요) → 체인지 버퍼 4절에 있다.

## 인덱스 키 변경

- B-Tree 키 값 변경은 키를 삭제 한 후, 다시 새로운 키 값 추가
- 이 또한 InnoDB에서는 체인지 버퍼를 이용해 지연 처리

## 인덱스 키 검색

- 트리 탐색 :  B-Tree 의 루트 노드 부터 시작하여, 브랜치 노드 거쳐 최종 리프 노드 까지 이동하며 비교 작업 수행
- 100% 일치, 앞부분 일치, 부등호, 비교조건 (<, >)
- 뒷부분 검색, 키 값 변형 등은 사용할 수 없다.
- 인덱스가 중요한 이유!
    - 레코드 잠금,넥스트 키락을 통한 인덱스 잠근 후, 테이블 잠그는 방식으로 구현되어 있음.
    - 인덱스가 없으면, Update, Delete 문장 실행될 때 불필요하게 많은 레코드를 잠근다.

<aside>
💡 페이지 (Page) 블록

</aside>

- 디스크의 데이터 저장 기본 단위 및 읽기 쓰기 작업의 최소 작업 단위
- InnoDB 스토리지 엔진의 버퍼 풀에서 데이터를 버퍼링하는 기본 단위
- 루트, 브랜치, 리프 노드 구분 기준

## B-Tree 인덱스 성능 요소

### 인덱스 키 값의 크기

- 인덱스의 페이지 크기와 키 값의 크기에 따라 결정
- 만약 인덱스의 키가 크다면, 한 페이지에 인덱스 키의 수가 줄어든다. →  레코드 500개 읽어와야 하는데, 만약 한 페이지가 그보다 작다면, 인덱스 페이지 최소 2번 읽어야 한다.  → 디스크 읽어야 하는 횟수 증가 → 느려진다.
- 인덱스 키 값 길다. → 인덱스의 크기가 커진다. → InnoDB 버퍼 풀, MyISAM 키 캐시 영역 제한적 → 메모리에 캐시해 둘 수 있는 레코드 줄어든다. → 메모리에 효율 떨어짐.

### B-Tree 깊이

- MySQL 에서 값을 검색할 때, 몇번이나 랜덤하게 디스크를 읽어와야 하는지의 문제
- 인덱스 키 값 커지면, 하나의 인덱스 페이지 담을 수 있는 인덱스 키 개수 적어짐 → 깊이가 깊어져, 디스크 읽기 더 많이 필요.
- 보통 5단계 이상 까지 깊어지는 경우 거의 없다.

### 선택도(기수성) (Cardinality, Selectivity)

- 모든 인덱스 키 값 가운데, 유니크 한 값의 수
- 중복 된 값 → **기수성** 낮, **선택도** 높아짐

<aside>
💡 기수성과 선택도의 차이점은 무엇일까?

1. **기수성 (Cardinality)**
    - 기수성은 특정 칼럼의 유니크한 값의 수를 의미합니다.
    - 예를 들어, 성별 칼럼이 '남자', '여자' 두 가지 값만 가진다면 기수성은 2입니다. 반면 사용자 ID와 같은 칼럼은 각 사용자마다 다른 값을 가지므로 높은 기수성을 갖습니다.
    - 일반적으로 높은 기수성을 가진 칼럼은 인덱스로 좋은 후보입니다. 이는 해당 칼럼에 대한 쿼리가 특정 값을 정확히 타겟으로 할 때, 인덱스를 사용하여 빠르게 데이터를 찾을 수 있기 때문입니다.
2. **선택도 (Selectivity)**
    - 선택도는 특정 값이나 조건을 만족하는 행의 비율을 의미합니다.
    - 선택도 = 조건을 만족하는 행의 수 / 전체 행의 수
    - 선택도가 높을수록 (즉, 조건에 맞는 행이 많을수록) 인덱스의 효과가 줄어들게 됩니다. 이는 인덱스를 사용하여도 많은 행을 스캔해야 하기 때문입니다.
    - 반대로, 선택도가 낮을수록 (즉, 조건에 맞는 행이 적을수록) 인덱스를 사용하여 데이터를 빠르게 검색할 수 있습니다.
</aside>

- ex) 데이터 10000개이고, 유니크 값 개수 각각 10개, 1000개 일때, 각각 조회되는 건수는 1000개, 10개이다. 전자의 경우, 999개가 불필요한 값이고, 후자의 경우 9개가 불필요하므로 후자의 경우가 효율적이다.

### 읽어야 하는 레코드 건수

- 비용 : 인덱스 통해 테이블의 레코드 읽는 비용 >> 바로 테이블의 레코드 비용
    - 거의 4~5배 정도 비싸다.
- 만약 인덱스를 통해 읽어야할 레코드 건수가 전체 레코드의 20~25 퍼센트를 넘어서면 모두 읽는 것이 효율적이다.

## B-Tree 인덱스를 이용한 데이터 읽기

### 인덱스 레인지 스캔

- 가장 대표적인 접근 방식
- 레코드를 하나, 한건 이상 읽는 경우 구분 → 10장에서

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/9c91e81b-0518-4964-a9dc-726d33631e07)

- 검색해야 할 인덱스의 범위가 결정되었을 때 사용한다.
- 인덱스 조건 만족하는 값 위치 찾는다. 루트 노드에서 비교를 시작하여, 브랜치,리프 노드까지 들어가서 레코드의 시작지점을 읽고 → 리프 노드 레코드를 순서대로 스캔
- 인덱스 탐색, 인덱스 스캔, 최종 레코드를 가져온다.
- 인덱스 칼럼의 정순 또는 역순으로 정렬되어 있다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/14c692ae-d70b-4f7a-b500-94d20041d59c)

- 인덱스의 리프 노드에서 검색 조건에 일치하는 건들은 데이터 파일에서 레코드를 읽어와야 한다. → 레코드 한건 단위로 랜덤IO가 한번 씩 일어난다.
- 커버링 인덱스 : 레코드 주소를 이용해 저장된 페이지 가져오고, 최종 레코드 읽어오는 과정 필요 없다. → 랜덤 읽기가 줄어들고 성능 빨라진다.
- 인덱스를 얼마나 탔는지 확인하는 방법

```sql
SHOW STATUS LIKE 'Handler_%';
```

- `Handler_read_key` : 인덱스에서 조건을 만족하는 값이 저장된 위치를 찾는 과정(인덱스 탐색)이 실행된 횟수
- `Handler_read_next` : 탐색 위치부터 필요한 만큼 인덱스를 읽은 레코드 건수(인덱스 스캔)

### 인덱스 풀 스캔

- 인덱스 레인지 스캔 보다 느림.
- 인덱스의 처음부터 끝까지 읽는다.
- 쿼리의 조건절에 사용된 칼럼이 인덱스의 첫 번째 칼럼이 아닌 경우
- 인덱스의 리프노드를 연결하는 링크드 리스트를 따라서 처음부터 끝까지 스캔하는 방식

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/df4af060-8369-499f-b8f9-afc87703cc78)

<aside>
💡 인덱스를 사용한다.

- 인덱스 레인지 스캔, 루스 인덱스 스캔

인덱스를 사용하지 못한다. 

- 인덱스 풀스캔 방식, 테이블 전체 읽기
</aside>

### 루스 인덱스 스캔

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/43a46828-542c-4dc3-95a4-6ddc4d644fbc)

- 인덱스를 듬성듬성 읽기
- GROUP BY 또는 집합 함수 가운데 min, max 함수로 최적화 할경우 쓴다.
- MySQL  8.0 버전 부터 인덱스 스킵 스캔 지원
- 인덱스 레인지와 인덱스 풀스캔을 tight 스캔으로 상반되어 말할 수 있음
- 제약조건은 10장 실행계획 참고

### 인덱스 스킵 스캔

- 인덱스 핵심은 값이 정렬되어 있으므로, 인덱스 구성하는 칼럼의 순서가 중요하다.
- 8.0 버전 부터, WHERE 조건절 검색 위해 사용가능하도록 용도 넓어짐
- 즉 인덱스가 (gender, birth_date) 와 같이 있다면, birth_date만으로 검색할 수 있도록 최적화 시켜줌

```sql
SELECT gender, birth_date
FROM employees
WHERE birth_date >= '1964-12-20'; 
```

- 인덱스 스킵 스캔 활성화 유무에 따라서 실행계획의 type 칼럼 값이 ‘index’ 에서 ‘range’로 표시

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c8b830a5-9a6c-44f4-8af8-9cecc8fe8ec7)

1. 루스 인덱스 스캔 방식으로 맨 첫번째 인덱스에 칼럼에 대해 모든 값을 추출한다.(중복 제외)
2. 추출한 값에 대해 인덱스 스킵 스캔을 실행한다.

의 방식으로 최적화를 한다. 

- **따라서, 인덱스 스킵 스캔을 하려면, 인덱스의 선행 칼럼이 가진 유니크한 값의 개수가 소량일 때만 적용이 가능하다.**
- 또한, SELECT 절에서, 모든 칼럼을 조회하도록 한다면, 나머지 칼럼도 필요로 하기 때문에, 인덱스 스킵 스캔 사용 못하고 풀 테이블 스캔한다.

### 다중 칼럼 인덱스

- 2개 이상의 칼럼을 포함하는 인덱스
- **인덱스의 두번째 칼럼은 첫 번째 칼럼에 의존해서 정렬됨. 즉, 첫번째 칼럼이 동일한 경우에만 두번째가 의미있다.**
- **따라서 각 칼럼의 위치(순서)가 상당히 중요하다!**

### B-Tree 인덱스의 정렬 및 스캔 방향

- 인덱스의 정렬
    - 8.0 버전 부터는 정렬 순서를 혼합한 인덱스를 생성할 수 있다.
- 인덱스 스캔 방향
    - 쿼리가 그 인덱스를 사용하는 시점에 인덱스를 읽는 방향에 따라 오름차순 또는 내림차순 정렬 효과
    
![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/96cd1f7d-4f55-46f0-9bbd-9ddda742486c)
    

```sql
SELECT * from employees
	ORDER BY first_name DESC LIMIT 5;
```

예를 들어 위 쿼리는 오름차순으로 모든 데이터를 읽고 뒤 5개의 데이터를 가져오는게 아니라, **뒤에서부터 인덱스를 역순으로 읽으면서 필요한 5개의 데이터만 레코드**만 가져오게 된다.

### 내림차순 인덱스

- 인덱스 역순 스캔이 인덱스 정순 스캔보다 조금 느리다.
- 기본 `B-Tree` 의 리프 노드에서, 하나의 페이지 내부서 레코드가 **단방향 오름차순 형태**로 만 연결이 되어 있기 때문이다.
- 만약, DESC  조회가 빈번하게 발생한다면, 오름차순 인덱스 보다는 내림차순 인덱스가 더 효율적일 수 있다.
- 만약, 뒷쪽만 집중적으로 읽어서 인덱스의 특정 페이지 잠금이 병목이 될 것으로 예상한다면, 쿼리에서 자주 사용하는 정렬 순서대로 인덱스를 생성하는 것 → 잠금 병목현상 안화.

### B-Tree 인덱스의 가용성과 효율성

- 비교 조건의 종류와 효율성
    - 다중칼럼 인덱스에서 칼럼의 순서와, 칼럼의 조건이 동등, 범위 조건인지에 따라 각 인덱스의 칼럼의 활용형태와 효율이 달라진다.
    - 작업 범위 결정 조건 : 작업의 범위를 결정하는 조건
    - 필터링 조건, 체크 조건 : 범위를 줄이지 못하고 단순히 거름 종이 역할만 하는 조건
    - 작업 범위 조건이 많아지면, 쿼리의 성능 속도가 높아지지만, 체크조건은 쿼리 실행을 오히려 더 느리게 할때가 많다.
- 인덱스의 가용성
    - B-Tree 인덱스는 왼쪽 값에 기준해서 (Left-most) 오른쪽 값이 정렬되어 있다.
        - 하나의 컬럼으로 검색해도, 왼쪽 부분 없으면, 인덱스 레인지 스캔 X
        - `SELECT * FROM employees WHERE first_name LIKE '%mer';`
        - 다중 컬럼 인덱스도 왼쪽 칼럼 모르면, 인덱스 레인지 스캔 X
        - `SELECT * FROM dept_emp WHERE emp_no >= 10114;` (`(dept_no, emp_no)` 칼럼 순서대로 생성되어 있다)
        - WHERE , GROUP BY, ORDER BY 똑같이 적용
- 작업 범위 결정 조건과 관련된 인덱스 판단 조건
- 단일 칼럼
    - `NOT-EQUAL` 조건으로 비교될 경우`<>`, `NOT IN`, `NOT BETWEEN`, `IS NOT NULL` 조건이 포함된다.
    - `LIKE '%??'` 형태로 문자열 패턴이 비교될 경우
    - 다른 연산자로 인덱스 칼럼이 변형된 후 비교된 경우
    - 데이터 타입이 서로 다른 비교
- 다중 칼럼
    - `INDEX ix_test (column1, column2, colum3 ... columnn)`
        - 작업 범위 결정 조건으로 인덱스를 사용하지 못하는 경우
            - column1에 대한 조건이 없는 경우
            - column1의 비교 조건이 인덱스 사용 불가 조건 중 하나인 경우
        - 작업 범위 결정 조건으로 인덱스를 사용할 수 있는 경우
            - column1~column(i-1) 까지 동등 비교 형태('=' 혹은 "IN")
            - column(i) 칼럼에 대해 다음 연산자 중 하나로 비교
                - 동등 비교
                - 크다 작다 형태(">", "<")
                - LIKE 좌측 일치 패턴(LIKE '승환 %')
                
                이렇게 되면 columni 까지는 작업 범위 결정 조건, column(i+1)부터 나머지 까지 조건은 체크 조건으로 사용된다.
                

# R-Tree 인덱스

---

- 공간 인덱스 : R-Tree 인덱스 알고리즘을 이용해 2차원의 데이터를 인덱싱하고 검색
- B-tree인덱스는 칼럼의 값이 스칼라 값, R-Tree 인덱스는 2차원의 공간 개념이다.
- MySQL 의 공간 확장을 이용하면 GPS에 기반한 SNS 서비스 간단한게 구현 가능

## 구조 및 특성

- Point, Line, PolyGon, Geometry
- MBR : Minimum Bounding Rectangle, 최소 크기의 사각형
- R-Tree 인덱스 : 이 사각형들의 포함 관계를 B-Tree 형태로 구현한 인덱스
- 최하위 레벨 : 각 도형 데이터의 MBR → 리프 노드
- 최상위 MBR : 루트 노드에 저장되는 정보
- 차상위 레벨 : 중간 크기의 MBR → 브랜치 노드

## 용도

- 위도, 경도 좌표 저장에 주로 사용
- ST_Contains(), ST_Within() 등과 같은 포함 관계를 비교하는 함수로 검색 수행해야지 인덱스로 이용 가능.
- 즉, 5km 반경 이내의 음식점을 검색하고 싶으면, 반경 5Km를 그리는 원을 포함하는 최소 사각형(MBR) 과 포함 관계를 비교한다.
- ST_Contains, ST_Within()은 동일한 비교 수행하지만 두 함수의 파라미터는 반대로 사용하여야 한다.

# 전문 검색 인덱스

---

- Full Text Search
- 기존의 인덱스 알고리즘은 좌측 일부 일치 검색이 가능하다.
- 문서의 내용 전체를 인덱스화 해서 특정 키워드가 포함된 문서 검색하는 Full Text 검색에는 B-Tree 인덱스를 사용할 수 없다.

## 인덱스 알고리즘

- 키워드를 분석하고, 빠른 검색용으로 사용할 수 있는 인덱스를 구축한다.
- 문서의 키워드를 인덱싱 하는 기법
    - 단어의 어근 분석
    - n-gram 분석 알고리즘

### 어근 분석 알고리즘

- 불용어 (Stop Word) 처리
    - 별 가치가 없는 단어를 모두 필터링 해서 제거한다.
    - 개수가 많지 않아, 알고리즘을 구현한 코드에 모두 상수로 정의한 경우가 많다.
- 어근 분석(Stemming)
    - 검색어로 선정된 단어의 뿌리인 원형을 찾는 작업
    - 플러그인 형태로 지원

### n-gram 알고리즘

- 무조건 몇글자씩 잘라서 인덱싱 한다.
- 형태소 분석보다는 알고리즘이 단순하다.
- 만들어진 인덱스의 크기가 상당히 크다.
- n은 인덱싱할 키워드의 최소 글자수를 의미하는데, 일반적으로 2글자 단위로 키워드를 쪼개서 인덱싱하는 2-gram 방식이 많이 사용된다.
- 2-gram 알고리즘에서는 10글자의 단어면 9개 토큰으로 이루어진다.
- mysql 서버에서 불용어를 제외하고 전문 검색인덱스에 등록한다.
- 더 빠르게 하기 위해 2단계 인덱싱, Merge-Tree 방식 또한 있다.
- 불용어 변경 및 삭제
    - MySQL 서버 내장 불용어가 존재한다.
    - MySQlL 서버의 모든 전문 검색 인덱스에 대해 제거하거나, 아니면 스토리지 엔진 별로 불용어 처리를 무시할 수 있다.
    - 사용자 정의 불용어를 설정할 수 있다.
        - 이는 , 불용어 목록을 파일로 저장하여 시스템 변수로 등록하거나, (InnoDB에서만) 불용어 목록을 테이블로 저장한다.

### 전문 검색 인덱스의 가용성

- 다음의 조건을 갖춰야 한다.
    - 쿼리 문장이 전문 검색을 위한 문법을 사용 ex) `Match(… ) AGAINST (…)`
    - 테이블이 전문 검색 대상 칼럼에 대해 전문 인덱스 보유

# 함수 기반 인덱스

---

- 칼럼의 값을 변형해서 만들어진 값에 대해 인덱스 구축해야 할 경우 사용
- MySQL 에서는 가상 칼럼, 또는 함수를 이용하여 구현. 이는 인덱싱할 값을 계산하는 과정 차이
- B-tree와 내부구조가 같다.
- 가상 칼럼과 직접 함수 기반 인덱스는 내부적인 구현이 같고 성능 차이가 발생하지 않는다.

## 가상 칼럼을 이용한 인덱스

• MySQL 8.0 버전 부터는 가상 칼럼을 추가하고 그 가상 칼럼에 인덱스를 생성할 수 있게 됐다.

```java
ALTER TABLE user
	ADD full_name VARCHAR(30) AS (CONCAT(first_name, ' ', last_name)) VIRTUAL,
	ADD INDEX ix_fullname (full_name);
```

## 함수를 이용한 인덱스

• MySQL 8.0 버전부터는 다음과 같이 테이블의 구조를 변경하지 않고, 함수를 직접 사용하는 인덱스를 생성할 수 있게 됐다.

- 함수 기반 인덱스를  활용하려면 반드시 조건절에 함수 기반 인덱스에 명시된 표현식이 그대로 사용돼야 한다.

```java
CREATE TABLE user (
	user_id BIGINT,
	first_name VARCHAR(10),
	last_name VARCHAR(10),
	PRIMARY KEY (user_id),
	INDEX ix_fullname **((CONCAT(first_name, ' ', last_name)))**
);

EXPLAIN SELECT * FROM user WHERE **CONCAT(first_name, ' ', last_name)** = 'harris lee';
```

# 멀티 밸류 인덱스

---

- 전문 검색 인덱스 제외한 모든 인덱스는 레코드 1건이 1개의 인덱스 키 값
- 멀티 밸류 인덱스는 하나의 데이터 레코드가 여러개의 키 값 가짐
- JSON의 배열 타입의 필드에 저장된 원소들에 대한 인덱스 요건으로 인해 최근 RDBMS에서 지원하기 시작했다.
- MySQL 8.0 버전 부터, Multi value index, 배열 형태에 대한 인덱스 생성.
- 멀티 밸류 인덱스를 활용하기 위해서는 반드시 다음 함수들을 이용해서 검색해야 옵티마이저가 인덱스를 활용한 실행 계획을 수립한다.
    - MEMBER OF()
    - JSON_CONTAINS()
    - JSON_OVERLAPS()

# 클러스터링 인덱스

---

- 클러스터링 : 여러개를 하나로 묶는다.
- MySQL에서 클러스터링 인덱스는 **InnoDB 스토리지 엔진에서만 지원!** 레코드를 프라이머리 키 기준으로 묶어서 저장.

## 클러스터링 인덱스

- 테이블의 프라이머리 키에 대해서만 적용
- 키 값이 비슷한 레코드 끼리 묶어서 저장하는 것을 클러스터링 인덱스
- **프라이머리 키 값에 의해 레코드의 저장위치가 결정,** 프라이머리 키 값이 변경되면, 레코드의 물리적인 저장 위치가 바뀐다. 따라서 인덱스 알고리즘 보다는 테이블 레코드의 저장 방식이다.

→ 프라이머리 키값으로 클러스터링된 테이블은 프라이머리 키 값 자체에 대한 의존도가 상당히 크기 때문에 프라이머리키 값을 신중히 결정해야한다**.**

- InnoDB와 같이 항상 클러스터링 인덱스로 저장되는 테이블은 프라이 머리 키 기반 검색 빠르다. 레코드의 저장이나 프라이머리 키 변경이 느림

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/278772ad-44f9-4543-a3ad-61985e121f78)

- **세컨 더리 인덱스를 위한 B-Tree의 리프노드와는 달리, 클러스터링 인덱스의 리프노드에는 레코드의 모든 칼럼이 같이 저장되어 있다.**
- 프라이 머리 키가 없을 떄는 InnoDB 스토리지 엔진이 우선순위 대로 프라이 머리 키를 대체할 칼럼을 선택한다.
    - 프라이머리 키가 있으면 기본적으로 프라이머리 키를 클러스터링 키로 선택
    - NOT NULL 옵션의 유니크 인덱스(UNIQUE INDEX) 중에서 첫 번째 인덱스를 클러스터링 키로 선택
    - 자동으로 유니크한 값을 가지도록 증가되는 칼럼을 내부적으로 추가한 후, 클러스터링 키로 선택

## 세컨더리 인덱스에 미치는 영향

- MYISAM, MEMORY 테이블 같이 클러스터링 되지 않은 테이블은 INSERT 될 때, 처음 저장된 공간에서 절대 이동하지 않고, 데이터 레코드가 저장된 주소는 내부적인 레코드 아이디 역할을 한다. 프라이 머리, 세컨더리 인덱스의 각 키는 그 주소를 이용해 실제 데이터 레코드를 찾아온다. 따라서 프라이 머리와 세컨더리 인덱스는 구조적으로 차이가 없다.
- MyISAM : ix_firstname 인덱스를 검색해서 레코드의 주소를 확인한다. 해당 레코드 주소로 최종 레코드를 가져온다.
- InnoDB 테이블 (클러스터링)의 모든 세컨더리 인덱스는  레코드의 저장된 주소가 아닌, 프라이 머리 키값을 저장되어 있다.
- InnoDB : ix_firstname 인덱스를 검색해서 레코드의 프라이머리 키 값을 확인한다. 해당 프라이머리 키 인덱스를 검색해서 최종 레코드를 가져온다

## 클러스터링 인덱스의 장점과 단점

- 장점
    - 프라이머리 키로 조회할 때 처리 성능이 매우 빠름
    - **테이블의 모든 세컨더리 인덱스가 프라이머리 키를 가지고 있기 때문에 인덱스만으로 처리 되는 경우가 많다(커버링 인덱스)**
- 단점
    - 모든 세컨더리 인덱스가 클러스터링 키를 갖기 때문에 키 값이 크면 전체적으로 인덱스의 크기가 커진다.
    - 세컨더리 인덱스를 통해 검색할 때 프라이머리 키로 한번 더 검색해야해서 성능이 느리다.
    - INSERT 할 때 프라이머리 키에 의해 레코드의 저장 위치가 결정되기 때문에 처리 성능이 느리다.
- 대부분 클러스터링 인덱스의 장점은 빠른 읽기이며, 단점은 느린 쓰기라는 것을 알 수 있다

## 클러스터링 테이블 사용 시 주의 사항

### 클러스터링 인덱스 키의 크기

- 클러스터링 테이블의 경우, 모든 세컨더리 인덱스가 프라이머리 키 값을 포함한다.
- 프라이 머리 키의 크기가 커지면 인덱스도 자동으로 크기가 커진다.
- **인덱스가 커질 수록 메모리가 더 필요해지므로, 프라이 머리 키는 신중하게 선택해야 한다.**

### 프라이 머리 키는 Auto-increment 보다는 업무적인 칼럼으로 생성(가능한 경우)

- 프라이 머리키는 레코드의 저장 방식. InnoDB에서는 엄청난 차이를 만들어 낸다.
- 업무적으로 해당 레코드를 대표할 수 있다면 해당 칼럼으로 설정한다.

### 프라이 머리 키는 반드시 명시할 것

- 가능하면 AUTO_INCREMENT 칼럼을 이용해서라도 프라이 머리 키는 생성한다.

### AUTO_INCREMENT 칼럼을 인조 식별자로 사용할 경우

- 프라이 머리 키의 크기가 길어도,  세컨더리 인덱스가 필요치 않다면 그대로 프라이머리 키를 사용하는 것이 좋다.
- 세컨더리 인덱스도 필요하고 프라이 머리 키의 크기도 길다면  AUTO_INCREMENT 칼럼 추가하여 PK 구성
- 인조 식별자 : 프라이 머리 키를 대체하기 위해 인위적으로 추가된 프라이머리 키
- 로그 테이블과 같이 조회보다 Insert 위주의 테이블 들을 AUTO_INCREMENT 를 이용한 인조 식별자가 성능향상에 도움이 된다.

# 유니크 인덱스

- 유니크는 제약조건에 가깝다. 같은 값이 2개 이상 저장될 수 없는 것.
- 유니크 인덱스에서 NULL 값은 특정한 값이 아니므로 2개 이상 저장될 수 있다.
- MYISAM 이나 MEMORY 에서는 프라이 머리 키는 NULL 이 허용되지 않는 유니크 인덱스와 같지만, InnoDB 에서는 PK 는 클러스터링 키 역할도 하므로 유니크와는 다르다.

## 유니크 인덱스와 일반 세컨더리 인덱스의 비교

- 유니크 인덱스와 유니크 하지 않은 일반 세컨더리 인덱스 성능 상 큰 차이가 없다.

### 인덱스 읽기

- 유니크 하지 않은 세컨더리 인덱스는 중복된 값이 허용되므로 읽어야 할 레코드가 많아 느린것이지 인덱스 자체 특성 때문에 느린게 아니다.
- 결국 디스크를 읽지 않는 이상, 레코드는  cpu 비교 작업 → 이는 그리 큰 코스트가 아니다.

### 인덱스 쓰기

- 새로운 레코드가 insert 되거나 인덱스 칼럼의 값이 변경되는 경우 인덱스 쓰기 작업이 필요
- 유니크 인덱스 키 값을 쓸 때는 기존의 값과 중복 있는지 검사해야하므로, 유니크 하지 않은 세컨 더리 인덱스 쓰기 보다 느리다.
- 중복 체크를 할때는 읽기 잠금, 쓰기를 할 때는 쓰기 잠금을 사용하므로 데드락이 빈번히 발생한다.
- InnoDB는 체인지 버퍼로 인덱스 키 지연 저장 가능한데,  유니크 인덱스는 반드시 중복 체크를 해야하므로, 작업 자체를 버퍼링 하지 못하여 느리다.

## 유니크 인덱스 사용 시 주의 사항

- MYSQL 에서  일반 인덱스와 유니크는 같은 영학을 하기 때문에, 중복 생성 X
- 동일한 칼럼에 대해 프라이 머리 키와 유니크 인덱스를 동일 생성도 중복이다.
- → 유니크 인덱스 쿼리의 실행계획, 파티션 부분 참고 (10, 13 장)
- 결론은 유일성 보장 칼럼은 유니크 인덱스 하되, 꼭 안필요하면, 유니크 하지 않은 세컨더리 인덱스 생성 고려

# 외래 키

- MySQL 에서 외래키는 InnoDB 스토리지 엔진에서만 생성
- 외래키 제약이 설정되면 자동으로 연관되는 테이블의 칼럼에 인덱스 생성
- 외래 키 관리의 두가지 특성
    - 테이블의 변경 (쓰기 잠금) 발생 하는 경우에만, 잠금 경합( 잠금 대기) 발생
    - 외래 키와 연관되지 않는 칼럼의 변경은 최대한 잠금 경합을 발생 시키지 않음.
- 예시

```sql
CREATE TABLE tb_parent (
	id INT NOT NULL,
    fd VARCHAR(100) NOT NULL,
    PRIMARY KEY(id)
    )
ENGINE = InnoDB;
```

```sql
CREATE TABEL tb_child (
	id INT NOT NULL,
    pid INT DEFAULT NULL,
    PRIMARY KEY (id),
    KEY ix_parentid(pid),
    CONSTRAINT child_ibfk_1 FOREIGN KEY (pid) REFERENCES tb_parent (id) ON DELETE CASCADE )
ENGINE = InnoDB:
```

```sql
INSERT INTO tb_parent
VALUES
(1, 'parent-1'),
(2, 'parent-2');
```

```sql
INSERT INTO tb_child
VALUES
(100,1, 'child-100');
```

## 자식 테이블의 변경이 대기하는 경우

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/9456ac09-2402-4091-8587-6e2f1c8637d6)

- 부모 id =2인 레코드가 쓰기 잠금을 갖고 있다.
- 자식은 외래키 칼럼인 pid 를 2로 변경하고 싶으므로, 부모 테이블 변경 작업이 끝날 때까지 기다린다.
- 부모가 끝나면(ROLLBACK OR COMMIT), 자식 작업이 처리 된다.

**→ 자식 테이블의 외래키 칼럼의 변경은 부모 테이블의 확인이 필요하고, 부모 테이블의 해당 레코드가 쓰기 잠금이 걸려있으면 해제 될때까지 기다린다. - INNODB이 1번째 특징**

**→ 자식 테이블의 외래키가 아닌 칼럼의 변경은 외래키로 인한 잠금 확장이 발생하지 않는다.  - INNODB의 두번째 특징**

## 부모 테이블의 변경 작업이 대기하는 경우

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/96b626cd-2160-421b-90c3-3fb8b5e3b3bb)

- 부모 키 1을 참조하는 레코드 변경 시,  자식테이블의 레코드에 쓰기 잠금이 걸린다.
- 부모 테이블에서 id = 1인 레코드를 삭제하는 경우, 자식 테이블의 레코드 쓰기 잠금이 해제될때 까지 기다린다.

**→ 자식테이블이 생성될 때 정의 된 외래키의 특성 (ON DELETE CASCADE) 때문에 부모 레코드가 삭제 되면 자식 레코드도 동시에 삭제되는 식으로 작동하기 때문**

<aside>
💡 외래 키의 특성 (on delete cascade) 가 없다면, 쓰기 잠금이 걸리지 않을까?

</aside>

- 물리적으로 외래키 생성 → 자식 테이블 레코드 추가 시, 참조키가 부모 테이블에 존재하는지 확인 한다. →  물리적인 외래키 고려 사항은 연관 테이블에 읽기 잠금을 건다. 잠금을 통해, 쿼리 동시 처리에 영향을 미친다.

# 참고 자료

Real MySQL 8장 인덱스 

[[MySQL] B-Tree 인덱스 구조와 인덱스 스캔](https://velog.io/@semi-cloud/MySQL-B-Tree-인덱스-구조와-인덱스-스캔)

[쿼리 튜닝 관점에서의 저장 매체와 랜덤 I/O, 순차 I/O](https://hudi.blog/storage-and-random-sequantial-io/)

[Real MySQL 8장 인덱스 #2 (8.6~)](https://harrislee.tistory.com/97)