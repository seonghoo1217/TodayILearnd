## 인덱스 종류
인덱스의 물리적 구성 방식에 따라 **BTree(Balanced Tree Index)**, **BITMAP**, **cluster**로 구분되고 
BTree 인덱스가 가장 보편적으로 사용된다.

## BTree 인덱스의 특징
- Balance 구조이기에 모든 Leaf 블록들이 같은 deapth(깊이)를 가지게 된다.
- Leaf 블록에는 키 데이터와 키 데이터를 포함한 레코드의 물리적 주소 정보를 키 데이터 정렬 순서에 맞춰 저장된다.
- 인덱스의 키 데이터가 테이블에서 Update가 되면, 인덱스에서는 Delete와 Insert를 하는 것으로 처리

## BTree Index Scan Type

### ⑴ Index Unique Scan
![IndexUniqueScan](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbXRVOf%2Fbtq6zry8315%2FpWdmM3nxpu0ehEJgQ89KuK%2Fimg.png)
- Index Unique Scan은 인덱스를 사용한 검색 방식 중 가장 빠른 방법 중 하나로 해당 방식을 사용하기 위해서는 기본 키 또는 Unique한 Index가 생성되어 있어야 하며, 인덱스를 구성하고 있는 모든 컬럼이 조건절에서 '='로 비교되어야 한다.
- - 조인되는 Inner Table 과의 조인 조건에도 Unique Index 또는 기본 키 컬림이 모두 조인에 참여했을 때에만 Index Unique Scan을 할 수 있다

### ⑵ Index Range Scan
![IndexRangeScan](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FczhTxr%2Fbtq6CN71DSM%2FKL3E5466ETIRC8Kxn9vGik%2Fimg.png)
- 인덱스를 사용해 인덱스가 생성된 컬럼에 대해 범위 검색을 하는 방법
- Unique Index를 사용하지 않거나, 비교연산자를 사용한 대다수의 경우가 이 방식으로 처리되는데, 비교연산자로는 <, <=, >, >=, between, like 등이 사용될 수 있다.
  - 현재 띱 프로젝트에 존재하는 진행중인 이벤트 필터링이 해당 예

### ⑶ Index Skip Scan
- Oracle 9i부터 적용이 가능한 방식으로, 결합 인덱스의 선행 컬럼에 대한 조건이 없고, 후행 컬럼에 대한 조건만 있는 경우에 적용.
- 이 방식은 선행 컬럼 값의 중복을 제거한 값의 종류가 적은 경우에 유리하다.

### ⑷ Index Full Scan
![IndexFullScan](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FLVLjI%2Fbtq6zWFplOu%2FkGyI9Q2lZTEsY1v9WkIhlk%2Fimg.png)
- 조건절에서 인덱스 컬럼 중 하나 이상을 사용한 경우 또는 SQL에서 사용한 컬럼들이 모두 하나의 인덱스에 존재할 경우 적용되는 방식이다.
- 이 가운데 SQL에서 사용한 컬림들이 모두 하나의 인덱스에 존재할 경우, 인덱스를 구성하는 컬럼 중 최소한 하나의 컬럼은 Not Null 
제약 조건을 충족해야 한다. 또, 이 방식을 병렬로 처리하는 것은 불가능하다.(단일 블록 I/O 블록 단위 검색)

### ⑸ Index Fast Full Scan
- Select 절과 조건절에 사용된 모든 컬럼이 인덱스 컬럼으로 구성되어 있어 테이블을 검색하지 않고 인덱스의 블록만을 스캔하여
원하는 데이터를 검색하는 방식이다. Index Full Scan 과 달리 병렬처리가 가능하나, 인덱스 키 데이터의 정렬은 보장되지 않는다.
-  또, 비트맵 인덱스에 대해서는 이 방식을 적용할 수 없다.

