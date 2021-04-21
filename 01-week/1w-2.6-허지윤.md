## 2.6 쿼리 메서드 기능과 @Query

### 1. 쿼리 메서드란?

- 메서드의 이름 자체가 쿼리의 구문으로 처리되는 기능
- docs: [link](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods)

### 2. 쿼리 메서드 종류
|                    | Sample                                                  | JPQL snippet                                                   |
|--------------------|---------------------------------------------------------|----------------------------------------------------------------|
| Distinct           | findDistinctByLastnameAndFirstname                      | select distinct … where x.lastname = ?1 and x.firstname = ?2   |
| And                | findByLastnameAndFirstname                              | … where x.lastname = ?1 and x.firstname = ?2                   |
| Or                 | findByLastnameOrFirstname                               | … where x.lastname = ?1 or x.firstname = ?2                    |
| Is, Equals         | findByFirstname,findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                       |
| Between            | findByStartDateBetween                                  | … where x.startDate between ?1 and ?2                          |
| LessThan           | findByAgeLessThan                                       | … where x.age < ?1                                             |
| LessThanEqual      | findByAgeLessThanEqual                                  | … where x.age <= ?1                                            |
| GreaterThan        | findByAgeGreaterThan                                    | … where x.age > ?1                                             |
| GreaterThanEqual   | findByAgeGreaterThanEqual                               | … where x.age >= ?1                                            |
| After              | findByStartDateAfter                                    | … where x.startDate > ?1                                       |
| Before             | findByStartDateBefore                                   | … where x.startDate < ?1                                       |
| IsNull, Null       | findByAge(Is)Null                                       | … where x.age is null                                          |
| IsNotNull, NotNull | findByAge(Is)NotNull                                    | … where x.age not null                                         |
| Like               | findByFirstnameLike                                     | … where x.firstname like ?1                                    |
| NotLike            | findByFirstnameNotLike                                  | … where x.firstname not like ?1                                |
| StartingWith       | findByFirstnameStartingWith                             | … where x.firstname like ?1 (parameter bound with appended %)  |
| EndingWith         | findByFirstnameEndingWith                               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing         | findByFirstnameContaining                               | … where x.firstname like ?1 (parameter bound wrapped in %)     |
| OrderBy            | findByAgeOrderByLastnameDesc                            | … where x.age = ?1 order by x.lastname desc                    |
| Not                | findByLastnameNot                                       | … where x.lastname <> ?1                                       |
| In                 | findByAgeIn(Collection<Age> ages)                       | … where x.age in ?1                                            |
| NotIn              | findByAgeNotIn(Collection<Age> ages)                    | … where x.age not in ?1                                        |
| True               | findByActiveTrue()                                      | … where x.active = true                                        |
| False              | findByActiveFalse()                                     | … where x.active = false                                       |
| IgnoreCase         | findByFirstnameIgnoreCase                               | … where UPPER(x.firstname) = UPPER(?1)                         |

### Ex 1) Between 을 사용한다고 했을 때

- **SQL 문 사용시**

```sql
SELECT * FROM spring.tbl_memo
WHERE mno >= 5 AND mno <= 9
ORDER BY mno DESC
```

- **Spring Data JPA 사용시**

👇 MemoRepository 인터페이스에 추가

```java
package org.zerock.ex2.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.zerock.ex2.entity.Memo;

import java.util.List;

public interface MemoRepository extends JpaRepository<Memo, Long>{
    List<Memo> findByMnoBetweenOrderByMnoDesc(Long from, Long to);
}
```

👇 MemoRepositoryTests 클래스에 추가

```java
package org.zerock.ex2.repository;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.zerock.ex2.entity.Memo;

import java.util.List;
import java.util.Optional;
import java.util.stream.IntStream;

@SpringBootTest
public class MemoRepositoryTests {
    @Autowired
    MemoRepository memoRepository;

    @Test
    public void testQueryMethods(){
        List<Memo> list = memoRepository.findByMnoBetweenOrderByMnoDesc(5L, 9L);
        for (Memo memo : list){
            System.out.println(memo);
        }
    }
}
```

- **결과**

<img width="573" alt="스크린샷 2021-04-20 오후 4 08 36" src="https://user-images.githubusercontent.com/60052127/115383831-41fa5280-a211-11eb-9810-23206c63d4ac.png">

### Ex 2) 쿼리 메서드 + Pageable

### 왜 필요한가?

- Ex 1) 에서와 같이 'OrderBy' 가 들어가면 메서드 이름이 길어져서 혼동하기가 쉬워짐. 그래서 Pageable 파라미터를 같이 결합해서 좀 더 간략한 메서드를 생성하기 위해서 필요하다.

👇 MemoRepository 인터페이스에 추가

```java
package org.zerock.ex2.repository;

import org.springframework.data.domain.Page;
import org.springframework.data.jpa.repository.JpaRepository;
import org.zerock.ex2.entity.Memo;

import java.awt.print.Pageable;
import java.util.List;

public interface MemoRepository extends JpaRepository<Memo, Long>{
    //    길고 헷갈림
    List<Memo> findByMnoBetweenOrderByMnoDesc(Long from, Long to);

    //    ⭐️ 짧음
    Page<Memo> findByMnoBetween(Long from, Long to, Pageable pageable);
}
```

👇 MemoRepositoryTests 클래스에 추가

```java
@Test
public void testQueryMethodWithPageable(){
    Pageable pageable = PageRequest.of(0, 4, Sort.by("mno").descending());
    Page<Memo> result = memoRepository.findByMnoBetween(1L, 7L, pageable);
    result.get().forEach(memo -> System.out.println(memo));
}
```

- **결과**

<img width="418" alt="스크린샷 2021-04-20 오후 4 38 50" src="https://user-images.githubusercontent.com/60052127/115384022-80900d00-a211-11eb-8f05-913a3c8d59a1.png">

### Ex 3) deleteBy로 시작하는 삭제처리

👇 MemoRepository 인터페이스에 추가

```java
import org.springframework.data.domain.Page;
import org.springframework.data.jpa.repository.JpaRepository;
import org.zerock.ex2.entity.Memo;

import org.springframework.data.domain.Pageable;
import java.util.List;

public interface MemoRepository extends JpaRepository<Memo, Long>{

    //    delete
    void deleteMemoByMnoLessThan(Long num);
}
```

👇 MemoRepositoryTests 클래스에 추가

```java
		@Commit
    @Transactional
    @Test
    public void testDeleteQueryMethods(){
        memoRepository.deleteMemoByMnoLessThan(2L);
    }
```

@Transactional 어노테이션 필요한 이유?

- deleteBy..의 경우 우선 'select'문으로 해당 엔티티 객체들을 가져오는 작업과 각 엔티티를 삭제하는 작업이 같이 이루어지기 때문.



@Commit 어노테이션 필요한 이유?

- 최종 결과를 커밋하기 위해서 사용. 이를 적용하지 않으면 deleteBy.. 가 롤백 처리되어서 결과가 반영되지 않음.

그런데 deleteBy 는 실제 개발에서 많이 사용되지 않는다고 함. 왜? SQL 처럼 한번에 삭제가 이루어지는 것이 아니라 각 엔티티 객체를 하나씩 삭제하기 때문!

### 3. @Query 란?

- SQL과 유사하게 엔티티 클래스의 정보를 이용해서 쿼리를 작성하는 기능
- 일반적인 경우 간단한 처리할 때만 쿼리 메서드 이용하고, @Query 이용하는 경우가 더 많다.
- @Query 의 value는 JPQL(Java Persistence Query Language) 로 작성하는데 흔히 객체지향 쿼리라고 불린다.

### 4. @Query 가 하는 일

- 필요한 데이터만 선별적으로 추출하는 기능
- 데이터베이스에 맞는 순수한 SQL(Native SQL) 사용 가능
- insert, delete, update 와 같은 select 가 아닌 DML 등을 처리하는 기능(@Modifying과 함께 사용)

👇 **DDL vs DML vs DCL 차이?**

- DDL(Data Definition Language) - 데이터 정의어 란? 데이터베이스를 정의하는 언어이며, 데이터리를 생성, 수정, 삭제하는 등의 데이터의 전체의 골격을 결정하는 역할을 하는 언어 입니다. CREATE, ALTER, DROP, TRUNCATE
- DML(Data Manipulation Language) - 데이터 조작어란? 정의된 데이터베이스에 입력된 레코드를 조회하거나 수정하거나 삭제하는 등의 역할을 하는 언어를 말합니다. SELECT, INSERT, DELETE, UPDATE
- DCL(Data Control Language) - 데이터 제어어란? 데이터베이스에 접근하거나 객체에 권한을 주는등의 역할을 하는 언어를 입니다. GRANT, REVOKE, COMMIT, ROLLBACK

👇 예시: mno의 역순으로 정렬하고 싶을 때

```java
@Query("select m from Memo m order by m.mno desc")
List<Memo> getListDesc();
```

### @Query의 파라미터 바인딩

- '?1, ?2'와 1부터 시작하는 파라미터의 순서를 이용하는 방식
- ':xxx' 와 같이 ':파라미터 이름'을 활용하는 방식

```java
@Transanctional
@Modifying
@Query("update Memo m set m.memoText = :memoText where m.mno = :mno ")
int updateMemoText(@Param("mno") Long mno, @Param("memoText") String memoText);
```

- ':#{ }'과 같이 자바 빈 스타일을 이용하는 방식

```java
@Transanctional
@Modifying
@Query("update Memo m set m.memoText = :#{#param.memoText} where m.mno = :#{#mno}")
int updateMemoText(@Param("param") Memo memo);
```

### Object[]리턴

- JOIN, GROUP BY 없이 원하는 데이터을 Object[] 형태로 쏙 뽑아올 수 있다.
- 예를들어 mno와 memoText 그리고 현재 시간을 얻어오고 싶다고 할 때, 현재 시간은 JPQL 로 CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP 로 현재 DB의 시간을 가져올 수 있다.

```java
@Query(value = "select m.mno, m.memoText, CURRENT_DATE from Memo m where m.mno > :mno",
countQuery = "select count(m) from Memo m where m.mno > :mno")
Page<Object[]> getListWithQueryObject(Long mno, Pageable pageable);
```

### Native SQL 처리

- 고유의 SQL 구문을 그대로 활용하는 방식이다.
- JPA 자체가 데이터베이스에 독립적으로 구현이 가능하다는 장점을 잃어버리기는 하지만 복잡한 JOIN 구문 등을 처리하기 위해 사용한다. nativeQuery 속성값을 true 로 지정하고 일반 SQL 사용하면 된다.

```java
@Query(value = "select * from memo where mno > 0", nativeQuery = true)
List<Object[]> getNativeResult();
```
