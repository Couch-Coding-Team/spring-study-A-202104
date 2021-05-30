---
title: "[Spring] SpringStudy-6일차"
layout: post
subtitle: Spring
date: "2021-05-11-23:48:51 +0900"

categories: study
tags: Spring
# layout: post
# title:  WebFrontEnd
# subtitle:   "시작하기"
# categories: study
# tags: java
comments: true
---

### JPQL을 이용할 떄의 검색

- 단일 엔티티가 아니라 QueryDSL에서 필요한 쿼리를 직접 실행할 구조가 필요

- Repository를 확장해서 JPQLQuery를 이용해 직접 JPQL을 생성해서 처리.

Repository에 search 패키지 추가하고

인터페이스와 Impl파일 추가.

```
test {
    useJUnitPlatform()
}

def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main.java.srcDir querydslDir
}

configurations {
    querydsl.extendsFrom compileClasspath
}


```

쿼리dsl을 사용하기 위해 build.gradle에 설정.

@Repository를 확장하기 위해

- 쿼리 메서드나 @Query로 처리 할 수 없는 기능은 별도의 인터페이스로 설계

- 별도의 인터페이스에 대한 구현클래스를 작성. 이때 QueryRepositorySupport라는 클래스를 부모로 사용.
- 구현 클래스에 Q도메인 클래스와 JPQLQuery를이용.

QueryRepositorySupport 참고 :
https://awse2050.tistory.com/26

이 클래스느 ㄴJPA에 포함된 클래스로 Querydsl라이브러리로 직접 뭔가 구현할때 사용.

repository에 search 패키지 작성.

![20210512_013612](/assets/20210512_013612_tbltfkxdy.png)

추가된 search 패키지에 확장하고 싶은 기능을 인터페이스로 선언후 Board 타입 객체 반환하는 메서드 선언

```
public interface SearchBoardRepository {

    Board search1();

    Page<Object[]> searchPage(String type, String keyword, Pageable pageable);

}

```

이 인터페이스를 구현하기 위한 SearchBoardRepositoryImpl에서 QuerydslRepositorySupport클래스를 상속해야함. 이 클래스는 생성자가 존재해서 super()를 이용해서 호출해야하고 이때 도메인 클래스를 지정하는 데 null값을 넣을 수는 없음.

정상동작은 BoardRepositoryTest로 확인

```
2021-05-12 01:50:48.121  INFO 22632 --- [    Test worker] o.z.b.r.s.SearchBoardRepositoryImpl      : search1........................
```

search 로그가 찍힘.

위와같이 로그가 기록되는게 확인되면 실제 JPQL을 작성하고 실행해보는 단계로 감.

이 과정에서 Querydsl의 라이브러리 내 JPQLQuery라는 인터페이스 활용.

활용하기 위해 코드 설정

```
QBoard board = QBoard.board;
       QReply reply = QReply.reply;
       QMember member = QMember.member;

       JPQLQuery<Board> jpqlQuery = from(board);
       jpqlQuery.leftJoin(member).on(board.writer.eq(member));
       jpqlQuery.leftJoin(reply).on(reply.board.eq(board));

       JPQLQuery<Tuple> tuple = jpqlQuery.select(board, member.email, reply.count());
       tuple.groupBy(board);

       log.info("---------------------------");
       log.info(tuple);
       log.info("---------------------------");

       List<Tuple> result = tuple.fetch();

       log.info(result);

       return null;

```

가장 중요한게 from이용

select 했을 때 객체 자체가 쿼리 자체 넣는거.(JPQL자체를 구문으로 만들어냄.

실제 밑에서 fetch로 결과를 가져옴.

```
select board, member1.email, count(reply)
from Board board
  left join Member member1 with board.writer = member1
  left join Reply reply with reply.board = board
group by board

```

이와 같이 바꿔줌.
그리고 Board형 객체 대신 각각의 데이터를 추출하는 경우 Tuple을 이용해서 수정하는게 좋음. (위 코드가 엔티티 객체인 Board 대신 Tuple사용한 코드)

select() 결과를 JPQLQuery <tuple>을 이용해서 처리하도록 변경하고 result변수의 타입도 List<Tuple> 타입으로 변경.

-> 다 가져오는게 아니라 부분적으로 가져오고 싶을때 튜플 사용한다 생각 하면 됨.

---

목록 만들고 페이지 처리 해줘야함

파라미터 Pageable을 전송하고 Page<Object[]>를 반환

SelectBoardRepository에 파라미터와 리턴타입을 반영하는 searchList()설계

```
public interface SearchBoardRepository {

    Board search1();

    Page<Object[]> searchPage(String type, String keyword, Pageable pageable);

}


```

searchPage()에서 검색타입, 키워드 페이지 정보를 파라미터로 받아왔는데, pageRequestDTO자체를 파라미터로 처리하지 않는 이유는DTO를 가능하면 Repository영역에서 다루지 않기 때문

```
public Page<Object[]> searchPage(String type, String keyword, Pageable pageable) {

        log.info("searchPage.............................");

        QBoard board = QBoard.board;
        QReply reply = QReply.reply;
        QMember member = QMember.member;

        JPQLQuery<Board> jpqlQuery = from(board);
        jpqlQuery.leftJoin(member).on(board.writer.eq(member));
        jpqlQuery.leftJoin(reply).on(reply.board.eq(board));

        //SELECT b, w, count(r) FROM Board b
        //LEFT JOIN b.writer w LEFT JOIN Reply r ON r.board = b
        JPQLQuery<Tuple> tuple = jpqlQuery.select(board, member, reply.count());

        BooleanBuilder booleanBuilder = new BooleanBuilder();
        BooleanExpression expression = board.bno.gt(0L);

        booleanBuilder.and(expression);

        if(type != null){
            String[] typeArr = type.split("");
            //검색 조건을 작성하기
            BooleanBuilder conditionBuilder = new BooleanBuilder();

            for (String t:typeArr) {
                switch (t){
                    case "t":
                        conditionBuilder.or(board.title.contains(keyword));
                        break;
                    case "w":
                        conditionBuilder.or(member.email.contains(keyword));
                        break;
                    case "c":
                        conditionBuilder.or(board.content.contains(keyword));
                        break;
                }
            }
            booleanBuilder.and(conditionBuilder);
        }

        tuple.where(booleanBuilder);

```

여기 까지 하면 목록 생성한 게 확인 됨

```
Hibernate:
    select
        board0_.bno as col_0_0_,
        member1_.email as col_1_0_,
        count(reply2_.rno) as col_2_0_,
        board0_.bno as bno1_0_0_,
        ...
    from
        board board0_
    left outer join
        tbl_member member1_
            on (
                board0_.writer_email=member1_.email
            )
    left outer join
        reply reply2_
            on (
                reply2_.board_bno=board0_.bno
            )
    where
        board0_.bno>?
        and (
            board0_.title like ? escape '!' //제목으로 검색되는 조건이 추가되었음
        )
    group by
        board0_.bno


```

---

pageable의 sort객체는 jpqlQuery의 order파라미터로 전달되어야 하지만 jpql에선 sort를 지원하지 않아서 OrderSpecifier<T extends COmparable> 을 파라미터로 처리

OrderSpecifier에서 order,Expression은 각각 com.querydsl.core타입이고

org.springframework.data.domain.Sort는 내부적으로 여러 sort객체 연결할 수 있어 foreach()로 처리.

OrderSpecifier엔 정렬이 필요해서 Sort객체의 정렬 관련 정보를 Order타입으로 처리하고 Sort객체에 속성(bno, title)등은 PathBuilder로 처리

```
//order by
       Sort sort = pageable.getSort();

       //tuple.orderBy(board.bno.desc());

       sort.stream().forEach(order -> {
           Order direction = order.isAscending()? Order.ASC: Order.DESC;
           String prop = order.getProperty();

           PathBuilder orderByExpression = new PathBuilder(Board.class, "board");
           tuple.orderBy(new OrderSpecifier(direction, orderByExpression.get(prop)));

       });
       tuple.groupBy(board);

       //page 처리
       tuple.offset(pageable.getOffset());
       tuple.limit(pageable.getPageSize());

       List<Tuple> result = tuple.fetch();

       log.info(result);

       long count = tuple.fetchCount();

       log.info("COUNT: " +count);

       return new PageImpl<Object[]>(
               result.stream().map(t -> t.toArray()).collect(Collectors.toList()),
               pageable,
               count);
   }

```

JPQLQUery를 이용해서 동적으로 검색 조건을 처리해보면 복잡하지만 한번의 개발로 count쿼리도 같이 처리 가능(다른 쿼리도 같이 처리)

---

검색처리는
PageRequestDTO로 이전 예제에서 검색에 쓰던 action 속성을 board/list로 변경하는거 제외하면 별 다른 작업 필요하지 않음.
컨트롤러 역시 동일함.

```

//        Page<Object[]> result = repository.getBoardWithReplyCount(
//                pageRequestDTO.getPageable(Sort.by("bno").descending())  );
        Page<Object[]> result = repository.searchPage(
                pageRequestDTO.getType(),
                pageRequestDTO.getKeyword(),
                pageRequestDTO.getPageable(Sort.by("bno").descending())  );

```

BoardServiceImpl은 주석부분을 위와같이 바꿔주면 됨.
