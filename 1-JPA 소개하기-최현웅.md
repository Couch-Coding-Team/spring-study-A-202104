# 1.JPA 소개

## Why? JPA

지금 시대는 객체를 RDB에 관리.

→JPA와 같은 ORM을 사용하지 않으면 SQL과 JDBC API를 직접 사용

→ 비즈니스 로직보다 SQL과 JDBC API를 작성하는데 많은 시간 소모

→그렇게 나온게 IBatis와 스프링의 JDBC템블릿과 같은 SQL 매퍼 사용

→ 하지만 여전히 CRUD용 SQL을 반복 작성

→SQL에 의존적인 개발을 피할 수 없음.

- [ ]  객체와 관계형 데이터 베이스의 차이(패러다임 불일치)

1.상속

2.연관관계

3.데이터 타입

4.데이터 식별 방법



![image](https://user-images.githubusercontent.com/76509935/118667437-fc8c6d80-b82e-11eb-9c69-1380842edf49.png)




## Album을 저장할려면?

1.객체를 분해한다.

2.아이템에 관한 Insert문 직접 작성

3.앨범에 관한 Insert문 직접 작성

## Album을 조회할려면?

1.아이템과 앨범 테이블을 조회할 JOIN SQL 문 직접 작성

2.ALBUM 객체 생성

→이러한 과정들이 패러다임의 불일치를 해결하려고 사용하는 소모 비용.

하지만 위의 두과정을 자바 컬렉션으로 바꿔보면 다음과 같다.

## JPA와 상속

JPA는 이러한 상속과 관련된 패러다임의 불일치 문제를 개발자 대신 해결해준다.

## Album을 저장할려면?

```java
list.add(album);
```

## Album을 조회할려면?

```java
Album album = list.get(albumId);

Item item = list.get(albumId); //다형성을 활용한 경우
```

# 연관관계

-객체는 연관관계를 참조를 통하여 사용.

-테이블은 외래키를 이용하여 연관관계를 풀어나감.

## 객체를 테이블에 맞추어 모델링

```java
class Member {
 String id;
 Long teamId;
 String username;
}
```

```java
class Team {
 Long id;
 String name; 
}
```

→저장과 조회할때는 편리하다.

→ teamId의 외래 키의 값을 그대로 보관하는 teamId 필드에 문제가 있다.

→RDB는 조인 기능이 있으므로 외래키의 값을 그대로 보관해도 된다. 하지만 객체는 연관된 객체의 참조를 보관해야 연관된 객체를 찾을 수 있다.

→즉 Member객체와 연관된 Team객체를 참조를 통해서 조회할 수 없다.

## 객체다운 모델링

```java
class Member {
 String id;
 Team team;
 String username;
 
 Team getTeam() {
 return team;
 }
}
class Team {
 Long id; 
 String name;
}
```

→객체를 테이블에 저장하거나 조회하기가 불편함

→객체 모델은 외래 키가 필요없고 단지 참조만 있으면 되는 반면 테이블은 참조가 필요없고 외래 키만 있으면 된다.

→ 즉 개발자가 중간에서 변환 역할을 해야함.

## JPA와 연관관계

Jpa는 이러한 연관관계와 관련된 패러다임의 불일치 문제를 해결해 준다.

```java
member.setTeam(team); //회원과 팀 연관관계 설정
jpa.persist(member);  //회원과 연관관계 함께 저장.
```

1.개발자가 회원과 팀의 관계를 설정하고 회원 객체 저장

2.JPA가 team의 참조를 외래 키로 변환해서 적절한 INSERT SQL을 DB에 전달

3.CRUD관련된 명령 JPA가 처리.

## 연관관계와 관련해서 극복하기 어려운 패러다임의 불일치

### 객체 그래프 탐색

:객체에서 회원이 소속된 팀을 조회할 때 참조를 사용해서 연관된 팀을 찾으면 되는데, 이것을 객체 그래프 탐색이라고 한다.

![image](https://user-images.githubusercontent.com/76509935/118667343-e7174380-b82e-11eb-9d54-f295637b9af9.png)

```java
//객체 그래프 탐색 코드

Member.getOrder().getOrderItem()...  //자유로운 객체 그래프 탐색
```

만약 sql을 실행해서 회원과 팀에 대한 데이터만 조회했다면 member.getTeam()은 성공하지만 member.getOrder(); 의 경우에는 null 값이므로 탐색이 불가능하다.

SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다.

→객체지향 개발자에게 너무 큰 제약.

→비즈니스 로직에 따라 사용하는 객체 그래프가 다른데 언제 끊어질지 모를 객체 그래프를 함부로 탐색할 수 없기 때문이다.

→직접 DAO를 열어서 SQL문을 확인해야함.

→SQL문에 의존적인 개발.

### JPA와 객체 그래프 탐색

JPA는 자유롭게 객체 그래프 탐색을 하면 연관된 객체를 사용하는 시점에 적절한 SELECT SQL문 실행.

→지연로딩 : 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다.

```java
//지연 로딩 사용

Member member =jpa.find(Mmember.class,memberId);//처음 조회 시점 Member select문 발생

Order order =member.getOrder();
order.getOrderDate(); //실제 order를 사용하는 시점에 select Order sql문 발생
```

## 비교

DB 에서는 기본 키의 값으로 각 로우(row)를 구분한다. 반면에 객체는 동일성비교와 동등성 비교라는 두가지 비교 방법이 있다.

- 동일성비교는 ==비교다. 객체 인스터스의 주소 값을 비교한다.
- 동등성 비교는 equals메소드를 사용해서 객체 내부의 값을 비교한다.

→객체와 DB의 패러다임 불일치

```java
String memberId="100";
Member member1 =memberDAO.getMember(memberId);
Member member2 =memberDAO.getMember(memberId);

member1 ==member2  //다르다
```

위의 코드를 보면 두개가 같을 거 같은데 동일성(==) 비교하면 false가 반환된다. 

DB 입장에서는 두개는 같은 로우이지만 객체 측면에서 보면 둘은 다른 인스턴스 이다.

(MemberDao.getMember()를 호출 할때마다 new Member()로 새로 생성)

## JPA와 비교

JPA에서는 같은 트랜잭션일때 같은 객체가 조회되는 것을 보장한다. (1차 캐시)
