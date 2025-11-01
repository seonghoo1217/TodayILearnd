## 개요

기존의 Spring Batch를 통해 **음료 데이터를 초기화**하기 위해서 **AWS Lambda**에 Python 코드(Crawling + OCR)를 활용하여 수동 트리거 로직을 구현했었다.

그러던 중 **클라우드 비용을 최소화하고** **짧은 피드백 루프(**개선속도를 높이기 위해**)를 위해** 로컬 Python 서버에서 크롤링과 OCR 로직을 수정하며 사용하고 있었다.

하지만, 개선작업이 끝난 이후 한 가지 실수를 하게되었는데 Spring Batch에 사용되는 URL을 [localhost](http://localhost) 주소로 기입하여 Batch가 **`Connection refused: localhost:8000`** 에러를 발생시키게 되었다.

이러한 경험을 계기로 Spring Batch의 기본 동작 흐름에 대해 톺아보려고한다.

---

## Batch의 동작흐름

Spring Batch는 단순 API 처럼 한번 실행되는 애플리케이션이 아닌 **데이터 처리 파이프라인**이다.

이때 흐름은 아래와 같은 순서로 처리된다.

```sql
JOB → STEP → (Reader -> Processor -> Writer)
```

### SpringBoot 실행 시점에서 보면

1. **Spring Context를 초기화**
    - 해당 단계에서 Bean생성과 더불어, @Componet, @Service, @Configuration 등 컴포넌트 관련 클래스들을 Spring IOC Container에 등록
        - tmi.)IOC Container 등록순서
            - 클래스 스캔 → BeanDefinition 등록 → Bean 인스턴스 생성 → 의존성 주입 → 초기화 콜백 → 컨테이너 등록 완료
            - 에러가 발생한건 **초기화 콜백시점(@PostConstruct)** 실행 함으로서 localhost:8000 연결 시도
    - **`BatchConfig`** 같은 배치 설정 클래스들이 이때 등록
    - Job, Step, Reader, Processor, Writer Bean들이 **이 시점에 생성**
2. **JobLauncher 실행**
    - 애플리케이션 기동 이후 JobLauncher 탐색(아맞당! 기준으로는 수동 trigger 구조)
    - `reader.read()`, `processor.process()`, `writer.write()` 순으로 실행
3. **각 단계에서 필요한 외부 리소스 접근**
    - Reader에서 DB/파일/REST API 호출 또느 Processor에서 외부 서버 호출 등의 작업

---

## 에러지점

에러는 위에도 적어놨듯이 Bean 초기화 단계에서 발생했다.

요약하자면 아래와 같다.

1. 현재 데이터를 제공하는 Fast API Server의 URI에 대해 WebClient로 HTTP 통신을 수행하는 구조
2. 데이터를 받기위해 WebClient 초기화
3. 하지만 [Https://amatdang.cloud](Https://amatdang.cloud의) 의 8000 포트는 Listen중이아님
4. 곧 바로 **`Connection Refused` Exception**

즉 Batch에 의한 에러라기 보단 Spring 에러에 가깝다. 하지만 Batch에서 또한 문제는 발생할 수 있기에 트러블 슈팅은 진행했어도 Batch에 대해서 좀 더 알아보자.

---

## Batch 핵심 요소 및 코드 동작

Batch는 아까도 말했듯 데이터를 **읽고(reader) → 가공하고(processor) → 처리(writer)**하는 데이터 파이프라인이다.

이는 하나의 **`Step`** 이란 단위로 동작한다.

---

### ItemReader

- DB, CSV, API,Json 등의 데이터들을 한 건(Generic)씩 읽어온다.
    - 여기서 정의된 한건은 비즈니스적으로 한 번에 처리하고자 코드적으로 정의된 단위
- 데이터의 **물리적 구조(DB row, 파일 라인)**보다 **논리적 처리 단위(Item)**에 초점을 두어 개발자가 유연한 설계를 할 수 있도록

**한건에 대한 정의**

```sql
public interface ItemReader<T> { 
    T read() throws Exception;//Signature 메서드
}
```

- **`read()`**가 호출될 때마다 **하나의 `T` 객체**를 반환한다.
- 더 이상 읽을 데이터가 없다면 **`null`** 반환 → Step Over

아맞당에서는 아래와 같은 구조로 데이터 파이프라인을 구성해놓았다.

```java
 @Bean
    public ItemReader<LambdaBeverageDto> beverageReader() {
        List<LambdaBeverageDto> list = cafeBeverageBatchService.fetchAll();
        return new ListItemReader<>(list != null ? list : List.of());
    }
```

```java
public record LambdaBeverageDto(
        String brand,
        String name,
        String image,
        String beverageType,
        String beverageTemperature,
        List<BeverageNutritionDto> beverageNutritions) {

    public CafeBeverage toEntity(CafeStore cafeStore) {
        BeverageType type =
                Optional.ofNullable(beverageType)
                        .map(String::toUpperCase)
                        .map(BeverageType::valueOf)
                        .orElse(BeverageType.ANY);

        Map<String, BeverageNutritionDto> nutritionsMap =
                beverageNutritions.stream()
                        .collect(
                                Collectors.toMap(
                                        BeverageNutritionDto::size,
                                        Function.identity(),
                                        (existing, replacement) -> existing));

        return CafeBeverage.of(
                name,
                UUID.randomUUID(),
                cafeStore,
                image,
                type,
                beverageTemperature,
                nutritionsMap);
    }
}
```

### ItemProcessor

- Reader가 읽은 데이터를 가공, 검증, 변환하는 단계

```java
O process(I item) throws Exception; //**메서드 시그니처**
```

- 입력 `I`(Reader의 출력)를 받아 가공 후 `O`를 반환
- `null`을 반환하면 Writer로 전달되지 않고 **필터링됨**

**아맞당 사용 예시**

```java
    @Bean
    public ItemProcessor<LambdaBeverageDto, CafeBeverage> beverageProcessor() {
        return cafeBeverageBatchService::toEntity;
    }
```

Reader에서 사용한 Dto 클래스를 **`from-to`** 패턴 을 사용하여 Entity로 변환시켜 영속화 대상으로 만든다.

### ItemWriter

- Processor의 결과 데이터를 **배치로 저장(쓰기)** 하는 단계
- Write의 대상은 DB영속화, API, Docs 등 다양하다.

```java
void write(List<? extends T> items) throws Exception; //**메서드 시그니처**
```

- 여러 건을 한 번에 받아 처리 (성능을 위해 batchSize 단위)
- 보통 JDBC batch insert, API bulk 요청, 파일 출력 등에 사용

**아맞당 사용예시**

```java
@Bean
    public ItemWriter<CafeBeverage> beverageWriter() {
        JpaItemWriter<CafeBeverage> writer =
                new JpaItemWriterBuilder<CafeBeverage>()
                        .entityManagerFactory(entityManagerFactory)
                        .build();

        return items -> {
            log.info("[DEBUG] Writing {} items to DB.", items.size());
            /*for (CafeBeverage item : items) {
                log.info(
                        "[DEBUG]   - Writing beverage: '{}', with sizes: {}",
                        item.getName(),
                        item.getSizes().stream()
                                .map(s -> s.getSizeType().name())
                                .collect(Collectors.toList()));
            }*/ // -> 로그용 Loop 없어도 코드는 똑같이 동작
            writer.write(items);
        };
    }
```

- **`JPAItemWriter`**가지정해놓은 Chunk(배치 단위)로 데이터를 저장
- Spring Batch가 **`chunk(n)`** 만큼 모아서 **`items`** 리스트로 넘기면

  `JpaItemWriter`가 그 리스트를 **엔티티 매니저에 차례로 persist/merge** 하고 **청크 경계에서 flush/clear**

- 즉, **트랜잭션/커밋 단위 = 청크 크기** (아맞당은 10)
    - **`StepBuilder`** (step 설정)에서 지정하는 값에 의해 결정

### 근데 데이터가 엄청나게 많다면요?

- 현재 위의 코드는 Spring Batch가 **`writer.write(items)`**로 청크 단위의 리스트를 넘겨주면 **`JPAItemWriter`**가 내부에서 **그 리스트를 루프 돌면서 각각 persist/merge** 하는 구조
- 현재 아맞당에서는 많아봤자 1000건의 데이터만을 Batch로 처리한다.
- 그렇다면 만약 데이터가 100만건이라면?
    - 실제로 Batch Excute를 진행하기위해선 **JPA Writer → JDBC 기반 Writer** WorkFlow를 구현해야한다.

이를 해결하기 위해선 아래와 같은 방법이 필요하다.

1. **JDBC Batch 이용**

```yaml
spring:
  jpa:
    properties:
      hibernate.jdbc.batch_size: 100          # JDBC 배치 ON
      hibernate.order_inserts: true
      hibernate.order_updates: true
      hibernate.generate_statistics: false
```

- 위 처럼 단순히 JDBC의 Batch를 이용할 수 있다.
- 하지만 PK 전략이 **IDENTITY 전략이면** insert 배치가 깨지거나 제한된다.
- Hibernate가 Batch 작업 주도
1. **JdbcBatchItemWriter로 전환**

```java
@Bean
public JdbcBatchItemWriter<CafeBeverage> beverageJdbcWriter(DataSource ds) {
    return new JdbcBatchItemWriterBuilder<CafeBeverage>()
        .dataSource(ds)
        .sql("""
             INSERT INTO cafe_beverage (name, brand, sugar, created_at)
             VALUES (:name, :brand, :sugar, :createdAt)
             """)
        .beanMapped() // CafeBeverage의 프로퍼티를 :파라미터에 매핑
        .assertUpdates(false) // (선택) trigger 등으로 갯수 불일치 허용
        .build();
}
```

- **Spring Batch Writer가 직접 `addBatch` 와 `executeBatch`** 를 보장하기 때문에 Exception Handling이 비교적 간단하다.
- 하지만, 직접 연관관계와 Insert 쿼리를 작성해야하기에 불편함은 존재한다.

### 🤔 근데 아맞당에서는 왜 Chunk Size를 10으로 했어요? 너무작은디

- 가장 주요한 이유는 데이터 정합성 때문이었다.
- 음료 목록의 타겟이되는 **Starbucks, MegaCoffee**등의 데이터들은 API로 제공되는 것이아니라 별도의 FastAPI 서버에서 **`Crawling`, `OCR`**로 수집하고 있기에 유효성 검사를 통해서 **데이터가 부정확하다면 빠르게 RollBack하기 위해 ChunkSize를 작게 가져갔다.**

---

## Spring Batch는 이 셋을 어떻게 돌리는가?

```java
while(true) {
    item = reader.read();         // 1건 읽기
    if(item == null) break;       // 데이터 끝
    result = processor.process(item);
    if(result != null) buffer.add(result);

    if(buffer.size() == chunkSize) {
        writer.write(buffer);     // 일정 단위(chunkSize)마다 기록
        buffer.clear();
    }
}
```

- ItemReader를 통해 Item을 읽어들이고 이를 Buffer에 가공한 상태로 저장한다.
- Buffer가 가득찬다면 Item에 대해 Writer 작업을 수행한다.

## Spring Batch의 엔진 레벨

좀 더 거대하게 보자면 아래와 같은 구조이다.

```java
JobLauncher.run()
   ↓
Job.execute()
   ↓
Step.execute()
   ↓
TaskletStep.execute()
   ↓
ChunkOrientedTasklet.execute()
   ↓
  ├─ ChunkProvider.provide() → reader.read()
  ├─ ChunkProcessor.process() → processor.process()
  └─ ChunkProcessor.write()   → writer.write()
```

- **reader.read() → processor.process() → writer.write()는 단순한 메서드 호출**이 아니라
  Spring Batch 엔진이 **청크 단위로 반복 호출하면서 트랜잭션 내에서 관리**하는 구조

Reader부터 다시 살펴보자

### Reader(`reader.read()` )

```java
T item = reader.read();  // 데이터 한 건 읽음
chunk.add(item);         // 청크 버퍼에 적재
```

1. Spring Batch가 현재 Step을 시작하면, 내부적으로 `ChunkProvider`가 작동
2. `reader.read()`가 반복 호출되어 **하나씩 데이터를 읽어온다.**
3. 이는 내부 버퍼(`Chunk`)에 임시 저장

아까 우리는 ChunkSize를 10으로 저장했다고했다. 그럼 Reader가 총 10건을 읽고나서 다음 단계인 **Processo**로 흐름이 넘어가게된다.

### Processor 단계(`processor.process()`)

1. Reader가 채운 Chunk(예: 10건)를 Processor가 하나씩 꺼내 변환
2. `process(item)`을 호출하여 비즈니스 로직 수행
    - 데이터 가공
    - 필터링 (`null` 반환 → writer로 전달 안 함)
    - 유효성 검증(아맞당에선 당류가 20% 이하일때 저당 Tag)
3. 결과물을 List에 담는다.

### Writer 단계 (`writer.write()`)

1. Processor가 반환한 결과물 리스트를 `writer.write()`에 전달
2. Writer는 일반적으로 **DB batch insert / update / 파일 쓰기 / API 전송**을 수행
3. 모든 write가 성공하면 트랜잭션 커밋, 실패하면 롤백.

총 플로우를 요약하자면 **“10건 읽고 → 10건 가공 → 10건 저장 → 1회 커밋”**의 파이프라인을 가진다.

위의 총 3단계가 트랜잭션에 감싸져서 트랜잭션이 커밋될때까지 한번의 Loop를 수행하며, 더이상 Reader에서 읽을 데이터가 없다면(null) 해당작업이 종료된다.

```java
while (true) {
    TransactionStatus tx = txManager.begin();
    chunk = provider.provide();           
    processor.process(chunk);             
    writer.write(chunk);                  
    txManager.commit();                  
}
```

- 위 코드에서 예외가 발생한다면 내부적으로 `txManager.rollback()` 을 수행하고 `retry` ,`skip` 등도 마찬가지

## 후기

Spring Batch의 완전한 개념을 통합해서 다뤄보고자 했지만 ItemReader, Processor등 내부 코드와 동작원리가 상당히 복잡하고 어려운 구조로 되어있다.

심지어 중요한 개념중 하나인 Job이나 Step 내부구조에 대해선 다루지 못했다.

이를 시리즈화 해서 앞으로는

Job과 Step의 생명주기 및 실행 컨텍스트 관리, Partitioning을 활용한 대규모 데이터 병렬 처리, 실패 복구와 재시도 메커니즘 구현등을 다뤄보고자한다.