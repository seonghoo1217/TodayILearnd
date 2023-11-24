DB 컬럼에 primary key, unique 등을 사용하여 값의 중복을 방지할 수 있는데, 여러개의 컬럼을 동시에 체크하여 중복을 체크해야 하는 경우 여러개를 묶어서 unique 처리를 할 수 있다.

```sql
ALTER TABLE 테이블명 ADD UNIQUE (컬럼1, 컬럼2, 컬럼3);
```
동시에 관리할 컬럼들을 UNIQUE() 안에 넣어주면 된다.

## in Spring Boot

```java
@Entity
@Table(
	name="tableName",
    uniqueConstraints={
        @UniqueConstraint(
            name={"contstraintName"}
            columnNames={"col1", "col2"}
        )
    }
)
@Data
public class Entity{
    @Column(name="col1")
    int col1;
    
    @Column(name="col2")
    int col2;
}
```