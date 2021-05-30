---
title: "[Spring] SpringStudy-8일차"
layout: post
subtitle: Spring
date: "2021-05-11-23:58:51 +0900"

categories: study
tags: Spring
# layout: post
# title:  WebFrontEnd
# subtitle:   "시작하기"
# categories: study
# tags: java
comments: true
---

### 메이븐과 사용 라이브러리

- Maven:

아파치 앤트의 대안으로 만들어짐.
아파치 라이센스로 배포되는 오픈 소스 소프트웨어
=> 프로젝트를 진행하면서 사용하는 수 많은 라이브러리들을 관리해주는 도구.

- Gradle:

기본적으로 빌드 배포 도구.
안드로이드 앱 공식 ㅁㅈㅇ빌드 시스템.
빌드 속도가 maven에 비해 10~100배 가량 빠름.
JAVA, C/C++, Python등을 지원.
빌드툴인 Ant Builder와 그루비 스크립트를 기반으로 구축되어 기존 Ant의 역할과 배포 스크립트의 기능을 모두 사용 가능.
=> 라이브러리 관리, 프로젝트 관리, 단위 테스트 시 의존성 관리.



#####

maven은 프로젝트가 커질수록 빌드 스크립트의 내용이 길어지고 가독성이 떨어짐.
gradle은 훨씬 적은 양의 스크립트로 짧고 간결하게 작성.
maven의 경우 멀티 프로젝트에서 특정 설정을 다른 모듈에서 사용하려면 상속을 받아야함.
gradle은 설정 주입 방식을 사용 => 멀티 프로젝트에 매우 적합!

- https://velog.io/@woo00oo/%EB%A9%94%EC%9D%B4%EB%B8%90Maven%EA%B3%BC-%EA%B7%B8%EB%9E%98%EB%93%A4Gradle


![20210519_182400](/assets/20210519_182400.png)

pom.xml에서 POM ( Project Object Model )은 프로젝트의 구조와 내용을 설명하고 있으며 pom.xml 파일에 프로젝트 관리 및 빌드에 필요한 환경 설정, 의존성 관리 등의 정보들을 기술.

이걸 그래이들에선 application.

<dependecies>에 사용할 라이브러리 지정.

그럼 자동으로 메이븐 공식 저장소에서 라이브러리에 자동으로 추가.

JPA에 하이버네이트 구현체를 사용하려면 많은 라이브러리가 필요하지만 핵심 라이브러리는  2개

- hibernate-core.jar
- hibernate-jpa-2.1-api.jar


### 객체 매핑

```
create table member(
  id varchar(255) not null, --아이디(기본 키)
  name varchar(255) , -- 이름
  age integer, --나이
  primary key(id)
  )
```


![20210519_191058](/assets/20210519_191058.png)



![20210519_191604](/assets/20210519_191604.png)


```
package jpabook.start;

import javax.persistence.*;
import java.util.List;

/**
 * @author holyeye
 */
public class JpaMain {

    public static void main(String[] args) {

        //엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
        EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성

        EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

        try {


            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료
    }

    public static void logic(EntityManager em) {

        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("지한");
        member.setAge(2);

        //등록
        em.persist(member);

        //수정
        member.setAge(20);

        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        //목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());

        //삭제
        em.remove(member);

    }
}

```

----

여기서 @Entity, @Table, @Column 이 매핑 정보. JPA는 매핑 어노테이션을 분석해서 어떤 객체가 어떤 테이블과 관계가 있는 지 알아냄.

@Entity
- 이 클래스를 테이블과 매핑한다고 JPA에 알려줌. 이렇게 사용된 클래스를 엔티티 클래스라고 함

@table

- 엔티티 클래스에 매핑할 테이블 정보를 알려줌. 여기서는 name 속성을 사용해서 member 엔티티를 member테이블에 매핑. 이 어노테이션을 생략하면 클래스 이름을 테이블 이름으로 매핑함.

@id

- 엔티티 클래스의 필드를 테이블의 기본키에 매핑한다. 여기서는 엔티티의 id 필드를 테이블의 id 기본 키 컬럼에 매핑했다.

이렇게 @id가 사용된 필드를 식별자 필드라 한다.

@Column
필드를 컬럼에 매핑한다. 여기서는 name속성으로 Member 엔티티의 username 필드를 member테이블의 name컬럼에 매핑했다.

- 매핑 정보가 없는 필드

age 필드에는 매핑 어노테이션이 없다. 이렇게 매핑 어노테이션을 생략하면 필드명을 사용해서 컬럼명으로 매핑한다. 여기서는 필드명이 age이므로 age컬럼으로 매핑한다.

-----

### persistence.xml

JPA는 persistence.xml사용해서 필요한 설정 정보들 관리. 이 설정 파일이 클래스 패스 경로에 있으면 JPA사용 가능.

```
<!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />

```


property name="javax.persistence.jdbc.driver" : JDBC 드라이버
property name="javax.persistence.jdbc.user : 디비 접속 아이디
property name="javax.persistence.jdbc.password" : 디비 접속 패스워드
property name="javax.persistence.jdbc.url": 디비 접속 url

옵션처럼 hibernate로 시작하는 속성은 하이버네이트에서만 사용 가능

#### 데이터 베이스 방언

JPA는 특정 데이터 베이스에 종속적이지 않은 기술. 따라서 다른 디비로 쉽게 교체 가능. 각 디비가 제공하는 SQL 문법과 함수가 다르다는 점이 있음.

mysql 에서의 varchar가 oracle에서의 varchar2라던ㄷ지

이걸 데이터베이스 방언이 처리해준다.

H2: 에서는 org.hibernate.dialect.H2Dialect가 처리해준다.


![20210519_193537](/assets/20210519_193537.png)


----


```

//엔티티 매니저 팩토리 생성
       EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
       EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성

       EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득
        try {


            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료
    }


```

코드는 3부분으로 설정.


- 엔티티 매니저 설정
- 트랜잭션 관리
- 비즈니스 로직


##### 엔티티매니저는 영속 컨텍스트에 접근하여 엔티티에 대한 DB 작업을 제공.


![20210519_194127](/assets/20210519_194127.png)


persistence.xml의 설정정보로 엔티티 매니저 팩토리 생성.

이떄 Persistence 클래스를 이용하는데 이 클래스로 엔티티 매니저 팩토리 생성해서 jpa사용할 수 있게 한다.


이렇게 하면 persistence.xml에서 이름이 jpabook인 영속성 유닛을 찾아서 엔티티 매니저 팩토리 생성.

이때 persisten.xml 의 설정정보를 읽어서 jpa를 동작시키기 위한 기반 객체를 만듬.

jpa 구현체에 따라 커넥션 풀도 생성하므로 엔티티 매니저 팩토리를 생성하는 비용은 아주 크다.

엔티티 매니저는 애플리케이션 전체에서 한번만 생성하고 공유해서 사용해야 한다.(싱글톤)

##### 엔티티 매니저 생성

엔티티 매니저 팩토리에서 엔티티 매니저 생성.
JPA 대부분의 기능은 엔티티 매니저가 제공.
엔티티 매니저로 엔티티를 디비에 CRUD 가능.
엔티티 매니저는 내부에 데이터 소스를 유지하면서(디비 커넥션)
디비와 통신.
엔티티 매니저는 디비오 ㅏ밀접한 관계가 있으므로 스레드 간 공유하거나 재사용하면 안 됨.


----

##### 트랜잭션 관리

JPA 사용하려면 항상 트랜잭션 안에서 데이터 변경.
트랜잭션 없이 데이터 변경시 예외 발생
트랜잭션 시작하려면 엔티티 매니저에서 트랜잭션 API 받아옴.
