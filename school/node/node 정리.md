# JS

# 2주차

## JavaScript 화살표 함수

http.createServer(function(req,res)) 해당 사항을 arrow function으로 변경하면 아래와같다.

http.createServer((req,res)⇒ 로 변경가능

var: 유효범위 함수

let, const: 유효범위 블록

const는 한번 선언하면 절대 바뀌지 않는 값

var와 let은 가변성이 존재(재할당 가능)

var는 타입을 붙여 재선언 해도 에러가 발생하지않지만(재선언, 재정의 가능), let의 경우 같은 이름의 변수를 재선언할경우 에러를 일으킨다.(재정의는 가능하나,재선언 불가능)

| Type | 재선언 | 재정의 |
| --- | --- | --- |
| var | o | o |
| let | x | o |
| const | x | x |

## 콜백함수

- 콜백함수는 익명함수이다.
- 다른 함수가 종료될 때 실행되도록 설정
- 함수 인자로 전달되는 함수

**장점**

- 다른 코드를 실행하기 전에 원래 함수의 실행종료를 기다릴 필요가 없다.

### 콜백의 동작순서

함수가 실행되고, callback을 요청하지 않는한 콜백의 내부 동작은 절대 실행되지 않는다.

결론적으로 콜백함수란, 특정시점에 자신이 호출하고 싶을 때 실행시키는 함수이다.

# 3주차

문자열을 병합하는 방법 : ‘+’를 사용하여 병합가능하다.

템플릿 리터럴을 사용하는 방법 : 벡틱을 사용하여 문자열과 값을 하나의 문자열로 나타낼 수 있다.

객체 : 속성,메서드

```java
let foo={
	a:1,
	b:2,
	c:()=>10
}
```

## Node.js

- 비동기 입출력 / 이벤트 기반 입출력 방식
- 모듈과 패키지를 사용하면서 서버 프로그램 구성
- 이미 만들어 놓은 모듈이 많다.

### 모듈

- 기능별로 분리된 별도의 파일
- 특정한 기능을 하는 함수나 변수의 집합
- 여러 프로그램에 해당 모듈을 재사용 가능
- 파일한개가 모듈하나

## 노드 : 모듈

- 코드를 모듈로 만들 수 있다는 점에서 브라우저의 자바스크립트와 다르다

하나의 파일에  모든 기능 넣는 것보단 분리할 경우 재사용성이 높아 분리한다.

### exports

- 모듈에서 사용하는 것. 해당 모듈의 함수,변수,전체일 수도 있다.
- 내보내기
- 객체처럼 속성명과 속성값 대입

### require

- 메인에서 사용하는 것. 모듈을 받아올 때 사용

### module.exports

- 객체 그대로 내보내기
- 어떤 값이나 대입가능하다.
- 하나의 변수나 함수 또는 객체를 직접 할당
- 객체를 그대로 할당
- 할당한 객체 안에 넣어 둔 변수나 함수를 메인파일에서 사용가능

```java
const odd="홀수"
const even="짝수"

module.exports={
odd,
even
};
```

위의 예시는 `module.exports` 를 사용하여 한번에 내보내는것

이경우 let {”odd”,”even”}=required(”./number”); 또는 number_check.odd,number_check.even 으로 받을 수 있다.

만약 따로 내보낸다면

```java
exports.odd="홀수";
exports.even="짝수";
```

## Node.js 특징

- 자바스크립트 언어(개발언어)
- 크롬 V8엔진
- npm 사용하여 외부 모듈을 자유롭게 사용
- 다양한 플랫폼에서 사용
- 기본적으로 싱글스레드
- 싱글 스레드
- 이벤트 기반
- 논 블록킹
- 비동기식 I/O

Node.js 장점

- I/O 처리 잘 하는 노드를 서버로 사용

Node.js 단점

- CPU 부하가 큰작업에는 부적합

**동기식**

- ******I/O 동작이 끝날때까지 대기******

비동기식

- I/O 동작이 끝날때까지 대기하진않는다

**비동기 방식**

- 싱글 스레드 방식으로 동작한다.
- 논블록킹 방식 : 값이 아니라 종료되면 함수를 반환한다
- 함수 : 함수가 종료되면 값을 반환
- 메인스레드가 존재하고 메인스레드는 실제 코드를 실행시킨다.
    1. 메인스레드는 실행 시간이 오래걸린다고 판단하면, 내부에 처리하는 다른 스레드로 해당작업을 전달
    2. 메인스레드에게 작업을 전달받은 다른 스레드는 해당 작업이 끝나면 콜백함수를 메인스레드에게 돌려준다
    3. 메인 스레드는 전달받은 콜백함수를 실행

**이벤트 기반 방식**

- 이벤트가 발생할 때 미리 지정해둔 작업수행

**이벤트 리스너**

- 이벤트가 발생하면 이벤트 리스너에 등록해둔 콜백함수를 호출
- 발생한 이벤트가 없거나 발생했던 이벤트 다 처리하면 다음이벤트 발생까지 대기

### 논블록킹 I/O

- 이벤트 루프 활용
    - 오래 걸리는 작업 효율적 처리
- 블록킹 :이전 작업이 끝나야만 다음작업실행
- 논블록킹 :이전작업 완료전까지 대기하지 않고 다음작업수행

### 비동기 단점

- 실행순서 보장 X

```java
function long() {
  console.log("2");
}

console.log(1);
long();
console.log(3);

function long2() {
  console.log("2");
}

console.log(1);
setTimeout(long2, 0);
console.log(3);
```

해당 방법을 사용하면, 콜백으로 인해 논블록킹 방식으로 동작

# 5주차

## NoSQL

- SNS 서비스가 활성화 됨에 따라 `데이터 패러다임`이 한정된 규모의 복잡성이 높은 데이터에서 단순한 대량의 데이터로 변경
- 기존 시스템의 변화가 필요하여 NoSQL을 사용
- 일반적인 RDB에선 하나의 고성능 DB에 데이터를 저장하는 용도
- 하지만, NOSQL에선 일반적인 서버 수십대를 연결하여 데이터를 저장 및 처리
- 여러대의 서버에 분산저장
- 데이터를 상호복제하여 특정 서버에 장애발생시에, 데이터 유실이 없는 형태
- RDB와 다르게 테이블의 스키마가 유동적이다

NoSQL은 RDB의 테이블과 비슷한 개념으로 `컬렉션(Collection)` 을 사용한다.

이외에도 여러 특성이 존재한다.

- Join 없다
- 성능 최우선
- 분산처리 용이
- 트랜잭션을 지원하지 않는다.

## MongoDB 용어

테이블 → 컬렉션(Collection)

레코드 → 문서객체(Document)

### Document

JSON 기반의 데이터로 객체형태를 지원한다.

## Collection 생성

```java
// 기존 SQL

CREATE TABLE people{
	id init Not NULL,
	user_id varchar(30),
  age int,
  status char(1),
  PRIMARY KEY(id)	
}
```

```java
//MongoDB
db.people.insertOne({
	user_id:"lee",
	age: 23,
	status:"A"
})
```

```java
//MongoDB collection 생성
db.createCollection("people"):
```

### Collection 구조변경

```java
//SQL
-- 추가
ALTER TABLE people ADD COLUMN join_date DATE;

-- 삭제
ALTER TABLE people DROP COLUMN join_date;
```

```java
// Mongo DB
-- 추가
db.people.updateMany({},{$set:{join_date:new Date()}});

-- 삭제
db.people.updateMany({},{$unset:{"join_date":""}});
```

Collection 구조에서 기본적으로 ALter는 필요없기에 추가 삭제만 존재

### DOCUMENT 입력

- insertOne() : 한개의 document 생성
- insertMany() : 여러개의 document 생성

```java
SELECT *
FROM people
```

```java
db.people.find();
```

```java
SELECT id,user_id,status
FROM people
```

```java
db.people.find({},{user_id:1,status:1})
```

```java
SELECT user_id,status
FROM people
```

```java
db.people.find({},{user_id:1,status:1,_id:0})
```

```java
SELECT *
FROM people
Where status= “A”
```

```java
db.people.find({status:"A"})
```

```java
SELECT *
From people
where status =”A” and age =50
```

```java
db.people.find({status:"A",age:50})
```

```java
SELECT *
from people
where status =”A” or age=50
```

```java
db.people.find({$or:[{status:"A"},{age:50}]})
```

```java
SELECT *
FROM people
Where age>25
```

```java
db.people.find({age:{$gt:25}})
```

```java
SELECT *
FROM people
Where age<25
```

```java
db.people.find({age:{$lt:25}})
```

```java
SELECT *
FROM people
Where age>25 AND age<=50
```

```java
db.people.find({age:{$gt:25,$lte:50}})
```

```java
SELECT *
FROM people
Where age=5 or age=15
```

```java
db.people.find({age:{$nin:[5,15]}})
```

```java
SELECT *
FROM people
Where user_id like "%bc%"
```

```java
db.people.find({user_id:/bc/})

db.people.find({user_id:{$regex:/bc/}})
```

```java
SELECT *
FROM people
Where user_id like "bc%"
```

```java
db.people.find({user_id:/^bc/})

db.people.find({user_id:{$regex:/^bc/}})
```

```java
SELECT *
FROM people
Where status="A"
ORDER BY user_id ASC
```

```java
db.people.find({status="A"}).sort({user_id:1})
```

```java
SELECT *
FROM people
Where status="A"
ORDER BY user_id DESC
```

```java
db.people.find({status="A"}).sort({user_id:-1})
```

```java
SELECT COUNT(*) FROM people
```

```java
db.people.count()
```

```java
SELECT COUNT(user_id)
FROM people
```

```java
db.people.count({user_id:{$exists:true}})
```

```java
SELECT COUNT(*)
FROM people
Where age>30
```

```java
db.people.count({age:{$gt:30}})
```

```java
SELECT DISTINCT(status)
FROM people
```

```java
db.people.distinct("status")
db.people.findOne()
```

```java
SELECT *
FROM people
Limit 1
```

```java
db.people.find().limit(1)
```

```java
COllection 생성
db.createCollection("people")
```

db.people.find() == db.people.find({ })

```java
SELECT user_id,status
FROM people
Where status="A" and user_id="kim
```

```java
db.people.find({status:"A",user_id="kim"},{_id:0,user_id:1,status:1})
```

# QUIZ

1-1. db.createCollections(”employees”)

1-2. db.employees.insertMany(

[

{user_id:”bcd001”,age:45,status:”A”}

]

);

1-2-1. db.find({user_id:”bcd002”},{user_id:1,age:1,status:1});

1-2-2. db.find({user_id:”bcd003”},{user_id:1,age:1,status:1,_id:0})

1-2-3. db.find({$or:[{user_id:”bcd004”},{age:”28”}]})

1-3.