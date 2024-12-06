JPA에서 제공하는 ID(PK)를 생성하는 방법은 두가지가 있다.

- 직접할당
- 자동할당

## 직접할당

**`@Id`** 어노테이션으로 ID 값을 직접적으로 할당하는 방식

## 자동할당

가장 흔하게 사용되는 방식으로 Id **`@GenratedValue`** 의 **`stretagy`** 옵션을 통해 설정하는 방식이다.

여기에는 총 4가지방식이 존재한다.

- IDENTITY
    - 기본 키 생성 방식을 DB의 DBMS에 생성 전략에 위임하는 방법이다. 해당 전략을 사용하면 엔티티를 생성할 때 **쓰기 지연**이 적용되지 않는다.
      그 이유로는 JPA에선 엔티티를 영속하기 위해, 고유 식별자가 필요한데, IDENTITY 전략에선 식별자가 DB에 저장되어야 할당되기때문이다.(엔티티 생성시 즉시 **INSERT 쿼리** 발생)
    - 이때 하이버네이트를 사용하는 경우에는 INSERT 쿼리의 결과를 다시 조회하지 않기 위해서 내부적으로 Statement.getGeneratedKeys를 사용할 수 있으며 INSERT 벌크 연산이 불가하다.
    - 대신, 생성되자마자 다른 트랜잭션에서 이를 참조할 수 있다.
- SEQUENCE
    - **SEQUENCE 전략**은 시퀀스 키 생성 전략을 지원하는 DB에서 사용할 수 있다.(지원안하는 DB가 존재)
    - 유일한 값을 자동으로 생성하는 전략으로 **`auto_increment`**와 달리 **초기 값과 한번에 증가할 크기를 설정**할 수 있다.
     어떤 시퀀스를 사용할 것인지를 `@SequenceGenerator` 로 설정할 수 있으며, 엔티티 매니저가 영속화를 시도할 때(`em.persist()` ) 먼저 데이터베이스 시퀀스를 이용하여 식별자를 조회한다.
      이후 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다.(영속화 할때 flush가 발생하는 Identity와 달리 Commit이후 저장)
- TABLE
    - 키 생성 전용 테이블을 만들어 시퀀스를 흉내내는 전략.
      어떤 테이블을 사용할 것인지를 `@TableGenerator`로 설정할 수 있다.

    ```jsx
    CREATE TABLE id_generator (
        id BIGINT NOT NULL,
        PRIMARY KEY (id)
    );
    
    -- 초기 값을 설정
    INSERT INTO id_generator (id) VALUES (0);
    ```

    ```jsx
    import javax.persistence.*;
    
    @Entity
    public class MyEntity {
    
        @Id
        @GeneratedValue(strategy = GenerationType.TABLE, generator = "id_gen")
        @TableGenerator(
            name = "id_gen",
            table = "id_generator",
            pkColumnName = "id",
            valueColumnName = "id",
            allocationSize = 1
        )
        private Long id;
    
        private String name;
    
        // Getters and Setters
    }
    
    ```

- AUTO
    - 데이터베이스 방언(종류)에 따라서 IDENTITY, SEQUENCE, TABLE 중 하나를 자동으로 선택

### 쓰기 지연?

데이터베이스에서 트랜잭션을 처리할 때, 실제로 데이터에 대한 디스크 I/O가 발생하기 전에 메모리나 다른 임시 저장소에 먼저 기록하고, 일정 조건이 만족되면 나중에 디스크에 저장하는 전략을 뜻한다.