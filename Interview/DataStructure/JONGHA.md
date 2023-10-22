# 정렬 알고리즘

- 버블 소트
    - 인접한 원소끼리 정렬시킴 → 공간 O(1), 시간 O(N^2)
- selection 소트
    - n 개의 원소를 가진 배열을 정렬할 때, 계속해서 바꾸는 것이 아니라 비교하고 있는 값의 index를 저장해둔다. 그리고 최종적으로 한 번만 바꿔준다. 하지만 여러 번 비교를 하는 것은 마찬가지이다. → 공간 O(1), 시간 O(N^2)
- Insertion Sort
    - n개의 원소를 가진 배열을 정렬할 때, i번째를 정렬할 순서라고 가정하면, 0 부터 i-1 까지의 원소들은 정렬되어있다는 가정하에, i 번째 원소와 i-1 번째 원소부터 0 번째 원소까지 비교하면서 i 번째 원소가 비교하는 원소보다 클 경우 서로의 위치를 바꾸고, 작을 경우 위치를 바꾸지 않고 다음 순서의 원소와 비교하면서 정렬해준다. 이 과정을 정렬하려는 배열의 마지막 원소까지 반복해준다. → 공간 O(1), 시간 O(N^2)
- Merge Sort
    - 반씩 분할하다가, 더이상 나누어 지지 않는 경우, 자기 자신 , 즉 원소 하나를 반환한다. 반환한 값끼리 combine 될 때, 비교가 이뤄진다. → 공간 O(1), 시간 O(Nlogn)
- 힙 소트
    - Binary Heap
    - 정렬 대상을 힙에 넣었다가, Sorting
    - Heapify 를 거쳐 정렬
    - 공간 O(1), O(nlogn) 시간 복잡도
- Quick Sort
    - Sorting 기법 중 가장 빠르다고 해서 quick 이라는 이름이 붙여졌다. **하지만 Worst Case 에서는 시간복잡도가 O(n^2)가 나올 수도 있다.**
    - → 공간 O(1), 시간 O(NlogN)
    - 최악일 때, pivot을 선택된 배열 내에 가장 큰 값 또는 작은 값으로하면, 정렬 과정이 n, n-1, ,,,처럼 이루어지므로 O(N^2) 이다 따라서 balanced하게 설정하는 것이 중요.
    - 퀵 정렬 개선 방법
        1. 난수 분할 : Pivot을 매번 Random한 위치에서 잡는 방법
        2. 배열의 가장 첫 인덱스, 끝 인덱스, 중간 인덱스에 해당하는 값들을 정렬하여 중간값을 Pivot으로 사용하는 방법
        3. 삽입 정렬과 함께 사용 : Array의 길이가 특정 수 이하라면 삽입 정렬을 사용하고 크다면 퀵 정렬을 사용하는 방법
- Non-comparison 알고리즘인 Radix Sort
    - 범위가 제한적 → 데이터 길에 의존
    - → 공간 O(N), 시간 O(N)
    - LSD는 덜 중요한 숫자부터 정렬하는 방식으로 예를 들어 숫자를 정렬한다고 했을 때, 일의 자리부터 정렬하는 방식
    - LSD 는 중간에 정렬 결과를 볼 수 없다. 무조건 일의 자리부터 시작해서 백의 자리까지 모두 정렬이 끝나야 결과를 확인할 수 있고, 그 때서야 결과가 나온다.

# Array vs LinkedList

## Array

- 선형 자료 구조
- 논리적 저장 순서와 물리적 저장 순서가 일치한다. 따라서 `인덱스`(index)로 해당 원소(element)에 접근할 수 있다.
- 그렇기 때문에 찾고자 하는 원소의 인덱스 값을 알고 있으면 `Big-O(1)`에 해당 원소로 접근할 수 있다.
- 즉 **random access**가 가능하다는 장점이 있는 것이다.
- 삽입과 삭제시 삭제 시, 제한 원소보다 큰 인덱스를 갖는 원소들을 `shift`해줘야 하는 비용(cost)이 발생하고 이 경우의 시간 복잡도는 O(n)

## LinkedList

- 선형 자료 구조
- Array 와는 달리 논리적 저장 순서와 물리적 저장 순서가 일치하지 않기 때문이다
- 삭제 또는 추가 할 때, 그 원소를 찾기 위해서 O(N)의 시가닝 걸림. 그리고 막상 그 원소를 추가할 떄는 O(1)이 걸림.

# Stack And Queue

## Stack

- 선형 자료 구조
- LIFO  : 나중에 들어간 원소가 먼저 나온다.

## Queue

- 선형 자료 구조
- FIFO : 먼저 들어간 원소가 먼저 나온다.
- Java Collection에서 Queue는 인터페이스이다. → 이를 구현하기 있는 Priority Queue등을 사용할 수 있다.
- `대신 큐에 정의 된 인터페이스 메소드 만 사용 가능!!!`

# Tree

- 비선형 자료 구조
- Node (노드) : 트리를 구성하고 있는 각각의 요소를 의미한다.
- Edge (간선) : 트리를 구성하기 위해 노드와 노드를 연결하는 선을 의미한다.
- Root Node (루트 노드) : 트리 구조에서 최상위에 있는 노드를 의미한다.
- Terminal Node ( = leaf Node, 단말 노드) : 하위에 다른 노드가 연결되어 있지 않은 노드를 의미한다.
- Internal Node (내부노드, 비단말 노드) : 단말 노드를 제외한 모든 노드로 루트 노드를 포함한다.

## Binary Tree (이진 트리)

- 루트 노드를 중심으로, 두 개의 서브 트리
- 노드가 하나인 것도, 공집합인 것도 , 이진트리 정의이다.
- Perfect Binary Tree : 위 아래, 왼쪽 오른쪽 다 채워진 트리
- Complete Binary Tree : 순서대로 채워진 트리
- Full Binary Tree: 모든 노드가 0개 혹은 2개의 자식 노드
- full Binay Tree의 internal 노드가 n개라고 가정할 때, leaf 노드에는 몇개가 있을 까?  (넥슨 문제)
    - L = I + 1 - Full Binary Tree 에서 모든 leaf 노드의 개수는 internal node 의 개수 + 1 이다.
    
    

## BinarySearchTree

- 규칙 1. 이진 탐색 트리의 노드에 저장된 키는 유일하다.
- 규칙 2. 부모의 키가 왼쪽 자식 노드의 키보다 크다.
- 규칙 3. 부모의 키가 오른쪽 자식 노드의 키보다 작다.
- 규칙 4. 왼쪽과 오른쪽 서브트리도 이진 탐색 트리이다.
- 이진 탐색 트리의 탐색 연산은 O(log n)의 시간 복잡도를 갖는다. 사실 정확히 말하면 O(h)라고 표현
- skew되는 현상을 막기 위해 균형을 잡기 위한 트리 구조의 재조정을 `Rebalancing`

## Binary Heap

- Complete Binary Heap 이다.
- 1번이 루트 노드
- `힙(Heap)`에는 `최대힙(max heap)`, `최소힙(min heap)` 두 종류가 있다.
- `Max Heap`이란, 각 노드의 값이 해당 children 의 값보다 **크거나 같은** `complete binary tree`를 말한다. ( Min Heap 은 그 반대이다.)
- 최대값을 찾는데 소요되는 연산의 time complexity 이 O(1)이다. 그리고 `complete binary tree`이기 때문에 배열을 사용하여 효율적으로 관리할 수 있다. (즉, random access 가 가능하다.)
- 제거된 루트 노드를 대체할 다른 노드가 필요하다. 여기서 heap 은 맨 마지막 노드를 루트 노드로 대체시킨 후, 다시 heapify 과정을 거쳐 heap 구조를 유지한다. 이런 경우에는 결국 O(log n)의 시간복잡도로 최대값 또는 최소값에 접근할 수 있게 된다.

## Red-Black-Tree

- Java Collection 에서 TreeMap도 내부적으로 RBT 로 이루어져 있고, HashMap 에서의 `Separate Chaining`에서도 사용된다. 그만큼 효율이 좋고 중요한 자료구조이다.

# Hash Table

- `hash`는 내부적으로 `배열`을 사용하여 데이터를 저장하기 때문에 빠른 검색 속도를 갖는다.
- 특정한 값을 Search 하는데 데이터 고유의 `인덱스`로 접근하게 되므로 average case 에 대하여 Time Complexity 가 O(1)이 되는 것이다.
- 항상 O(1)이 아니고 average case 에 대해서 O(1)인 것은 collision 때문이다. 하지만 문제는 이 인덱스로 저장되는 `key`값이 불규칙하다는 것이다.
- **특별한 알고리즘을 이용하여** 저장할 데이터와 연관된 **고유한 숫자를 만들어 낸 뒤** 이를 인덱스로 사용한다. 특정 데이터가 저장되는 인덱스는 그 데이터만의 고유한 위치이기 때문에, 삽입 연산 시 다른 데이터의 사이에 끼어들거나, 삭제 시 다른 데이터로 채울 필요가 없으므로 연산에서 추가적인 비용이 없도록 만들어진 구조이다.

### Hash Function

- `hash method` 또는 `해시 함수(hash function)`라고 하고 이 메소드에 의해 반환된 데이터의 고유 숫자 값을 `hashcode`라고 한다. 저장되는 값들의 key 값을 `hash function`을 통해서 **작은 범위의 값들로** 바꿔준다.
- `hash function`을 통해서 key 값들을 결정한다면 동일한 값이 도출될 수가 있다. 이렇게 되면 동일한 key 값에 복수 개의 데이터가 하나의 테이블에 존재할 수 있게 되는 것인데 이를 `Collision` 이라고 한다.
- 일반적으로 좋은 `hash function`는 키의 일부분을 참조하여 해쉬 값을 만들지 않고 키 전체를 참조하여 해쉬 값을 만들어 낸다. 하지만 좋은 해쉬 함수는 키가 어떤 특성을 가지고 있느냐에 따라 달라지게 된다.
- Collision 이 많아질 수록 Search 에 필요한 Time Complexity 가 O(1)에서 O(n)에 가까워진다.

### Resolve Conflict

### 1. Open Address 방식 (개방주소법)

- 해시 충돌이 발생하면, (즉 삽입하려는 해시 버킷이 이미 사용 중인 경우) **다른 해시 버킷에 해당 자료를 삽입하는 방식.**
- 버킷이란 바구니와 같은 개념으로 데이터를 저장하기 위한 공간이라고 생각하면 된다.
- Collision이 발생하면 데이터를 저장할 장소를 찾아 헤맨다. Worst Case 의 경우 비어있는 버킷을 찾지 못하고 탐색을 시작한 위치까지 되돌아 올 수 있다.
    1. Linear Probing 순차적으로 탐색하며 비어있는 버킷을 찾을 때까지 계속 진행된다.
    2. Quadratic probing 2 차 함수를 이용해 탐색할 위치를 찾는다.
    3. Double hashing probing 하나의 해쉬 함수에서 충돌이 발생하면 2 차 해쉬 함수를 이용해 새로운 주소를 할당한다. 위 두 가지 방법에 비해 많은 연산량을 요구하게 된다.

### 2. Separate Chaining 방식

- 일반적으로 Open Addressing은 Separate Chaining 보다 느리다. Open Addressing 의 경우 해시 버킷을 채운 밀도가 높아질수록 Worst Case 발생 빈도가 더 높아지기 때문이다.
- 반면 Separate Chaining 방식의 경우 해시 충돌이 잘 발생하지 않도록 보조 해시 함수를 통해 조정할 수 있다면 Worst Case 에 가까워 지는 빈도를 줄일 수 있다. Java 7 에서부터 Separate Chaining 방식을 사용하여 HashMap을 구현하고 있다.
- **연결 리스트를 사용하는 방식(Linked List)**
    - 각각의 버킷(bucket)들을 연결리스트(Linked List)로 만들어 Collision이 발생하면 해당 bucket 의 list 에 추가하는 방식이다. 연결 리스트의 특징을 그대로 이어받아 삭제 또는 삽입이 간단하다.
    - 하지만 단점도 그대로 물려받아 작은 데이터들을 저장할 때 연결 리스트 자체의 오버헤드가 부담이 된다. 또 다른 특징으로는, 버킷을 계속해서 사용하는 Open Address 방식에 비해 테이블의 확장을 늦출 수 있다.
- **Tree 를 사용하는 방식 (Red-Black Tree)**
    - 기본적인 알고리즘은 Separate Chaining 방식과 동일하며 연결 리스트 대신 트리를 사용하는 방식
    - 연결 리스트를 사용할 것인가와 트리를 사용할 것인가에 대한 기준은 하나의 해시 버킷에 할당된 key-value 쌍의 개수이다.
    - 데이터의 개수가 적다면 링크드 리스트를 사용하는 것이 맞다. 트리는 기본적으로 메모리 사용량이 많기 때문이다.
    - 데이터 개수가 적을 때 Worst Case 를 살펴보면 트리와 링크드 리스트의 성능 상 차이가 거의 없다. 따라서 메모리 측면을 봤을 때 데이터 개수가 적을 때는 링크드 리스트를 사용한다.

### Open Address vs Separate Chaining

- 일단 두 방식 모두 Worst Case에서 O(M)이다.
- 하지만 Open Address방식은 연속된 공간에 데이터를 저장하기 때문에 Separate Chaining에 비해 캐시 효율이 높다. 데이터의 개수가 충분히 적다면 Open Address방식이 Separate Chaining보다 더 성능 좋다.
- Separate Chaining방식에 비해 Open Address방식은 버킷을 계속해서 사용한다. 따라서 Separate Chaining 방식은 테이블의 확장을 보다 늦출 수 있다.

### 보조 해시 함수

- 보조 해시 함수의 목적은 key의 해시 값을 변형하여 해시 충돌 가능성을 줄임
- Separate Chaining 방식을 사용할 때 함께 사용되며 보조 해시 함수로 Worst Case에 가까워지는 경우를 줄일 수 있다.

### 해시 버킷 동적 확장(Resize)

- 해시 버킷의 개수가 적다면 메모리 사용을 아낄 수 있지만 해시 충돌로 인해 성능 상 손실이 발생한다.
- HashMap 은 key-value 쌍 데이터 개수가 일정 개수 이상이 되면 해시 버킷의 개수를 두 배.
- 해시 충돌로 인한 성능 손실 문제를 어느 정도 해결할 수 있다. 해시 버킷 크기를 두 배로 확장하는 임계점은 현재 데이터 개수가 해시 버킷의 개수의 75%가 될 때이다. `0.75`라는 숫자는 load factor 라고 불린다.

# Graph

### 정점과 간선의 집합, Graph

*cf) 트리 또한 그래프이며, 그 중 사이클이 허용되지 않는 그래프*

## 그래프 구현

- 인접 리스트 : vertex 의 adjacent list를 확인해봐야 하므로 vertex 간 연결되어있는지 확인하는데 오래 걸린다. Space Complexity 는 O(E + V) 이다. Sparse graph 를 표현하는데 적당한 방법이다.
- 인접 행렬 : O(1) 으로 파악할 수 있다. Edge 개수와는 무관하게 V^2 의 Space Complexity 를 갖는다. Dense graph 를 표현할 때 적절할 방법이다.
- 깊이 우선 탐색 (Depth First Search: DFS)
    - Stack 이다. **Time Complexity : O(V+E)**
- 너비 우선 탐색 (Breadth First Search: BFS)
    - BFS 에서는 자료구조로 Queue 를 사용한다. 연락을 취할 정점의 순서를 기록하기 위한 것이다. 우선, 탐색을 시작하는 정점을 Queue 에 넣는다.
    - **Time Complexity : O(V+E)**
    - 모든 간선에 가중치가 존재하지않거나 모든 간선의 가중치가 같은 경우, BFS 로 구한 경로는 최단 경로

### Minimum Spanning Tree (MST)

- 그래프 G 의 spanning tree 중 edge weight 의 합이 최소인 spanning tree
- spanning tree란 그래프 G의 모든 vertex가 cycle이 없이 연결된 형태.

### Kruskal Algorithm

- 초기화 작업으로 **edge없이** vertex들만으로 그래프를 구성.
- Edge Set 을 non-decreasing 으로 sorting 해야 한다. 그리고 가장 작은 weight 에 해당하는 edge 를 추가하는데 추가할 때 그래프에 cycle 이 생기지 않는 경우에만 추가
- 연결할 때 마다 값이 동일한 `set-id` 개수가 많은 `set-id` 값으로 통일시킨다
- Edge 의 weight 를 기준으로 sorting - O(E log E)
- cycle 생성 여부를 검사하고 set-id 를 통일 - O(E + V log V) => 전체 시간 복잡도 : O(E log E)
- 일반적으로 유니언 파인드를 할 때, 트리의 높이만큼 탐색하는 logn이지만, skew에서는 O(n).
- Weighted union rule, Path compression 중 하나만 사용하면 amotizedO(logN)
- 둘 모두를 사용하면 amortized O(a(n))

[Path Comprssion DSU의 amortized O(logN) 증명](https://kim1109123.tistory.com/84)

### Prim Algorithm

- 초기화 과정에서 한 개의 vertex 로 이루어진 초기 그래프 A 를 구성한다.
- 그래프 A 내부에 있는 vertex 로부터 외부에 있는 vertex 사이의 edge 를 연결하는데 그 중 가장 작은 weight 의 edge 를 통해 연결되는 vertex 를 추가.
- 연결된 vertex 는 그래프 A 에 포함된다. 위 과정을 반복하고 모든 vertex 들이 연결되면 종료한다.
- => 전체 시간 복잡도 : O(E log V)
