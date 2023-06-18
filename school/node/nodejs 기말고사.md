## (10) 1. MongoDB문법: 소녀시대
- MongoDB 모듈 : MongoDB Driver
- MongoDB 설치
```javascript
npm install mongodb
```
- MongoDB 데이터 베이스 연결 셋팅
```javascript
var MongoClient = require('mongodb').MongoClient;
var url = 'mongo://localhost:27017/database';
```

- MongoDB 데이터베이스 연결
```javascript
var database;

MongoClient.connect(url,function (err,db)){
    console.log("DB Connect")
    database=db;
}
```
### 소녀시대 예문
```javascript
db.users.remove({name:/소녀/})
db.users.find().pretty()
db.users.insert(
    {
        id: "test01",
        name: "소녀시대",
        password: "123456"
    }
)
db.users.find().pretty()
```

```markdown
>> 결과값
{
    _id: 랜덤값
    id: "test01",
    name: "소녀시대",
    password: "123456"
}
```

## (10) 2. mongodb client: (사용자인증: 로그인)

### Express의 미들웨어 호출
```markdown
var bodyParser = require('body-parser')
, cookieParser = require('cookie-parser')
, static = require('serve-static')
, errorHandler = require('errorhandler');
```

### 에러 핸들러 모듈 호출
```javascript
var expressErrorHandler=require('express-error-handler');
```

### Session 미들 웨어 호출
```javascript
var expressSession=require('express-session');
```

### 익스프레스 객체 생성
```javascript
var app=express();
```
### 익스프레스 속성 설정
```javascript

// 기본 포트 설정
app.set('port',process.env.PORT || 3000);

// body-parser를 이용하여 application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({extended:false}))

// body-parser를 json으로 파싱
app.use(bodyParser.json())

// cookie-parser 설정
app.use(cookieParser());
```

### 로그인 메인 로직
```javascript
var router = express.Router();

router.route('/process/login').post(function (req,res){
    console.log("/process/login 호출됨");
    
    var paramId = req.body.id || req.query.id;
    var paramPassword = req.body.password || req.query.password;
    
    if (database){
        authUser(database,paramId,paramPassword,function (err,docs){
            if (err) {throw  err};
            //조회된 레코드 있으면 성공 응답
            if (docs){
                console.dir(docs);
                //로그인 성공시 로직~~~~~~~~
                res.writeHead('200',{'Content-Type':'text/html;charset=utf'});
                res.write("로그인 성공")
            } else {
                // 조회된 레코드 없음, 즉 회원가입 데이터 없음
                res.writeHead('400',{'Content-Type':'text/html;charset=utf'});
                res.write("로그인 실패")
            }
        })
    }else {
        // DB 연결 실패
    }
})
```
### 라우터 사용(로그인)
```javascript
app.use('/',router);

var authUser=function (database,id,password,callback) {
    var users=database.collection('users');
    
    users.find({"id":id,"passwor":password}).toArray(function (err,docs){
        if (err){
            callback(err,null);
            return;
        }
        
        if (docs.length>0){
            // 사용자 찾음
            callback(null,docs)
        }else {
            //조회하지 못함
            callback(null,null);
        }
    })
}
```

### 에러 핸들링
```javascript
var errorHandler=expressErrorHandler({
    static: {
        '404':'./public/404.html'
    }
});

app.use(expressErrorHandler.httpError(404));
app.use(errorHandler);
```

### 사용자 추가(로그인)
```javascript
var router=express.Router();
router.route('/process/addUser').post(function (req,res){
    var paramId = req.body.id || req.query.id;
    var paramPassword = req.body.password || req.query.password;
    var paramName = req.body.name || req.query.name;
    
    if (database){
        addUser(database,paramId,paramPassword,paramName,function (err,result){
            
        })
    }
})
```
### 회원가입 로직
```javascript
var addUser=function (database,id,password,name,callback){
    var users=database.collection('users');
    
    users.insertMany([{"id":id,"password":password,"name":name}],function (err,result){
        if (err) throw  err;
        
        if (result.insertedCount>0){
            //회원가입됨
        }else {
            //회원가입안됨
        }
        
        callback(null,result);
    })
}
```
## Express 모듈

### 생성 및 사용예시
```javascript
var http = require('http');
var express=require('express')
var app=express();

app.listen(3000,function (){
    
});

http.createServer(app).listen(3000,function (){
    
});
```

### express 메서드
- set(key,value) : 서버 설정을 위한 속성지정
- get(key) : 서버 설정 속성을 불러옴
- use([path,]function[,function ...]) : 미들웨어 함수사용
- get(path,callback) : GET 메서드의 요청처리
- post(path,callback) : POST 메서드의 요청처리
- all(path,callback) : 모든 요청처리

```javascript
app.set('port',process.env.PORT || 3000);
let port=app.get('port')
```

- env: 서버환경설정
- views: 뷰들이 들어있는 폴더를 설정
- view engine : 기본 뷰엔진 설정

### 요청 파라메터 사용 
- params
- query
- body

- get 요청 : query를 이용해서 데이터 전달
- post 요청 : body를 이용해서 데이터 전달
- params : url 경로에 존재하는 데이터

```javascript
app.get('/users/:name', (req, res) => {
  let name = req.params.name;

  let params = req.params;
  let query = req.query;
})
```

### 요청 응답
**요청**
- req.query : 쿼리 문자열
- req.path : 요청 URL 경로
- req.paramas : URL 파라미터
- req.cookie : 요청 메시지내에 쿠키(cookie-parser 필요)
- req.body : 요청 메시지 내에 바디 분석 (body-parser 필요)

## (10) 3. 미들웨어: next() 메서드 (myLogger 사용자 미들웨어)
### 미들웨어: myLogger 사용자 미들웨어
미들웨어는 위에서 아래로 실행되기 때문에 순서가 중요하다

각각의 미들웨어는 next() 메서드 호출하여
그 다음 미들웨어가 처리될 수 있도록 순서를 넘긴다

next() 는 다음 미들웨어로 넘기는 역할을 한다
```javascript
res.send();
next();

send() //메서드로 프로그램 종료되므로
//다음 미들웨어 실행하는
next()// 메서드 필요
```

## (10) 4. 미들웨어: static 정적/body-parser 미들웨어
- Express 미들웨어 종류
  - static : 특정 폴더의 파일을 특정 패스로 접근
  - morgan : 클라이언트 요청 로그를 확인하는 미들웨어
  - body-parser : body에 포함된 데이터를 JSON 형태로 파싱하는 미들웨어
  - cookie-parser : 헤더에 포함된 데이터를 JSON 형태로 파싱하는 미들웨어
  - multer : 파일 업로드를 위한 미들웨어
  - express.static : 정적파일 제공을 위한 미들웨어
  - passport : 사용자 인증을 위한 미들웨어

```javascript
// 미들웨어 설치
npm install serve-static
npm install body-parser
npm install cookie-parser
```
### static 미들웨어 예제
```javascript
var static=require('serve-static')
res.end("<img src='/images/house.png' width='50%'");

app.use('/public',static(path.join(_dirname,'public')));
```
특정 폴더의 파일을 특정패스로 접근
```javascript
http://localhost:3000/images/house.png
```

### body-parser 미들웨어
- Http POST방식으로 전송된 body 내부의 데이터를 파싱
- POST 요청했을때 요청 파라미터 확인
- get 방식: 주소 문자열에 요청 파라미터가 들어감
- post 방식: 본문 body 영역에 요청 파라미터가 들어가므로 요청파라미터를 파싱해야함

### 사용예제
```javascript
const bodyParser=require('body-parser');
app.use(bodyParser.urlencoded({extends:false}));
app.use(bodyParser.json({limit:5000000}))//5MB까지 제한 해제 //application json
```
- bodyParser를 제대로 사용하기 위해선 bodyParser.json() 형태로 호출해야한다.
- body에는 제한이 없지만,node.js에서는 10kb로 제한
- json() 옵션을 추가하여 제한 변경가능

```javascript
//JSON 데이터 파싱을 위한 미들웨어 등록
app.use(bodyParser.json());
```

```javascript
//URL-encoded 데이터 파싱을 위한 미들웨어 등록
app.use(bodyParser.urlencoded({extended:false}));
```
## (10) 5. MySQL: 과제3
### Mysql Connection Pool 만들기
- connectionLimit: 커넥션 풀에서 만들 수 있는 최대 연결개수
- host: 연결할 호스트 이름을 설정
- port: 데이터베이스가 사용하는 포트 번호
- user: 데이터베이스 사용자 아이디
- password: 사용자 비밀번호
- database: 데이터베이스 이름
- debug: 데이터베이스 처리과정을 로그로 남길지 결정

```javascript
// mysql 연결설정
var pool = mysql.createPool({
    connectionLimit :10,
    host: 'localhost',
    user: 'node',
    password: 'node123',
    database: 'nodeDB',
    debug: false
});
```

### DB 생성
```javascript
var mysql=require("mysql");

var conn_info={
    host:"localhost",
    port:"3306",
    user:"study",
    password:"study123",
    database:"nodedb"
}
var conn=mysql.createConnection(conn_info);

conn.connect(function (error){
    if (error) throw  error
    else {
        console.log("성공");
    }
})
```

### Query 직접쓰기
```javascript
var sql="insert into Testtable(ind_data,str_data) values (?,?)";

var input_data1=[100,"문자열1"];

conn.query(sql,input_data1,function (err){
    console.log("저장완료")
})
```

### DB Connection
```javascript

```

## (10) 6. mongoose: 과제4