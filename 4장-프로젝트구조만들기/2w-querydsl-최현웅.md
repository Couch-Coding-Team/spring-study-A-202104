# 0428 발표자료

[엔티티 클래스와 querydsl 설정](https://www.notion.so/querydsl-dad14f6d53cf48d495ac6f4a114d0469)

![Untitled](https://user-images.githubusercontent.com/76509935/116230052-26072b80-a792-11eb-97ed-26e9b2d17730.png)
![Untitled 1](https://user-images.githubusercontent.com/76509935/116230037-23a4d180-a792-11eb-9fb5-a273802420ae.png)
![Untitled 2](https://user-images.githubusercontent.com/76509935/116230039-243d6800-a792-11eb-9203-6195bb78d2db.png)

Query dsl을 사용하기 위해서는 위와 같이 build.gradle에서 다음과 같은 설정들이 필요하다.

- plugins 항목에 querydsl 설정 추가.
- dependencies 항목에 필요한 라이브러리 추가.
- Gradle에서 사용할 추가적인 task를 추가.

![Untitled 3](https://user-images.githubusercontent.com/76509935/116230041-24d5fe80-a792-11eb-8f78-733ce3f4461f.png)

옵션들이 잘 들어갔다면 task-other 항목에 compilequerydsl이 추가 된 것을 볼 수 있다.

compilejquerydsl을 실행하면 기존에 있던 엔티티들의 이름에 Q가 붙은파일들이 자동적으로 build-generated폴더안에 생성이 되며 클래스 내부를 살펴보면 기존에 있던 필드들이 변수 처리 된 것을 확인할 수 있다.

![Untitled 4](https://user-images.githubusercontent.com/76509935/116230044-24d5fe80-a792-11eb-8d78-bd0e42e7f07d.png)
![Untitled 5](https://user-images.githubusercontent.com/76509935/116230047-256e9500-a792-11eb-9f53-e4e7ee5efc08.png)


위와 같은 Q클래스들은 개발자가 직접 건드리지 않고 gradle 의 task를 통해서 자동으로 생성되는 방법만을 사용한다. 

querydsl을 사용하기 위해서는 Repository 인터페이스에 추가적으로  QuerydslPredicateExecutor을 상속 받아야 한다.

QueryDsl의 사용법은 다음과 같습니다.

- BooleanBuilder를 생성합니다
- 조건에 맞는 구문은 Querydsl에서 사용하는 Predicate 타입의 함수를 생성합니다.
- BooleanBuilder에 작성된 Predicate를 추가하고 실행합니다.

-단일 항목 검색 테스트

![Untitled 6](https://user-images.githubusercontent.com/76509935/116230049-256e9500-a792-11eb-87ed-badc3e3b1a78.png)

1. 먼저 Q클래스 도메인 클래스를 가져온다. Q도메인 클래스를 이용하면 엔티티 내에 있는 필드들을 변수로 사용할 수 있다.
2. BooleanBuilder는 where문에 들어가는 조건들을 넣어주는 컨테이너로 생각하면 됩니다.
3. 원하는 조건은 필드 값과 결합해서 생성하여야 한다. BooleanBuilder 내부의 값은 com.querydsl.core.types.Predicate 타입이어야 합니다.
4. 만들어진 조건은 and나 or같은 키워드로 결합 시킨다.
5. BooleanBuillder는 QueryDslPredicateExcutor 인터페이스의 findAll()을 사용 할 수 있습니다.

위와 같은 코드를 사용하면 페이지 처리와 동시에 검색처리가 가능합니다.

-다중 항목 검색 테스트

복합 조건은 여러 조건이 결합된 형태이다. querydsl의 경우 BooleanBuilder는 and, or의 파라미터로 

BooleanBuilder를 전달할 수 있어서 복합적인 쿼리를 생성할 수 있다.

![Untitled 7](https://user-images.githubusercontent.com/76509935/116230051-26072b80-a792-11eb-9c1f-2728f831c04b.png)

다중 조건도 이전 소스와 유사하다.

중간에 exName과 exAge라는 BooleanExpression을 결합하는 부분과 BooleanBuilder에 추가되었다.

이렇게 조건이 추가될경우 BooleanExpression을 간단히 추가하여 복잡한 조건 검색을 간단히 할 수 있다.
