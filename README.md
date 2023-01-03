# studyQeurydsl
## 스프링 Qeurydsl 스터디
- https://www.inflearn.com/course/querydsl-%EC%8B%A4%EC%A0%84
- 기간: 2022.12.28 ~
- 프로젝트 구성
	- Java 17
	- SpringBoot 3.0.1
  - Dependencies
    - Spring Web
    - Spring Data JPA
    - H2 Database
    - Lombok
---------------
## Study Notes
- SpringBoot 3.0 이상 + Querydsl Gradle 설정
  - https://velog.io/@juhyeon1114/Spring-QueryDsl-gradle-%EC%84%A4%EC%A0%95-Spring-boot-3.0-%EC%9D%B4%EC%83%81
  - https://www.inflearn.com/chats/669477 (테스트 실패하는 경우, Q{entity} 저장위치 변경)
- complieQuerydsl을 통해 생성되는 Q{entity} 파일의 경우, 저장소에 안올리는걸 추천(ignored)


- Querydsl 라이브러리 살펴보기
  - querydsl-apt: Querydsl 관련 코드 생성 기능 제공
  - querydsl-jpa: Querydsl 라이브러리
- 스프링 부트 라이브러리 살펴보기
  - spring-boot-starter-web
    - spring-boot-starter-tomcat: 톰캣(웹서버)
    - spring-webmvc: 스프링 웹 MVC
  - spring-boot-starter-data-jpa
    - spring-boot-starter-aop
    - spring-boot-starter-jdbc
      - HikariCP 커넥션 풀(부트 2.0 기본)
    - hibernate + JPA: 하이버네이트 + JPA
    - spring-data-jpa: 스프링 데이터 JPA
  - spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot
      - spring-core
    - spring-boot-starter-logging
      - logback, slf4j
- 테스트 라이브러리 살펴보기
  - spring-boot-starter-test
    - junit: 테스트 프레임워크, 스프링 부트 2.2부터 junit5 사용
    - mockito: 목 라이브러리
    - assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    - spring-test: 스프링 통합 테스트 지원


- JPQL vs Querydsl
  - 오류 시점
    - JPQL: 쿼리가 문자(String)기 때문에 쿼리에 오타가 있는 경우, 실행 시점(Runtime) 오류 발생
    - Querydsl: 쿼리가 Java코드로 작성되기 때문에 쿼리에 오타가 있는 경우, 컴파일 시점(Compile) 오류 발생
      - Querydsl이 JPQL보다 이런면에서 안정성이 높음
  - 파라미터 바인딩
    - JPQL: 파라미티 직접 바인딩
    - Querydsl: 파라미타 자동 바인딩 처리


- JPAQueryFactory를 필드로 제공하면 동시성 문제는 어떻게될까?
  - 동시성문제는 JPAQueryFactory를 생성할때 제공하는 EntityManager(em)에달려있다. 스프링프레임워크는 여러 쓰레드에서 동시에 같은 EntityManager에 접근해도, 트랜잭션마다 별도의 영속성컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도된다


- 기본 Q-Type 활용
  - QClass 인스턴스 사용하는 2가지 방법
    ```java
    QMember qMember = newQMember("m"); // JPQL alias(별칭) 직접 지정
    QMember qMember = QMember.member; // 기본 인스턴스 사용(QMember에 생성되는 public static final 변수)
    ```
  - 다음 설정을 추가하면 실행되는 JPQL을 확인 할 수 있다(in application.yml or properties)
    ```
    spring.jpa.properties.hibernate.use_sql_comments: true
    ```
    

- 검색 조건 쿼리
  - 기본 검색 쿼리
    ```java
    Member findMember = queryFactory
      .selectFrom(member)
      // and 조건
      .where(member.username.eq("member1")
              .and(member.age.eq(10)))
      .fetchOne();
    ```
  - 검색조건은 .and(), .or()등 메서드를 체이닝 가능하다.
  - 참고: select(), from()은 selectFrom()으로 합칠 수 있다.
  - JPQL이 제공하는 모든 검색 조건 제공
    ```java
    member.username.eq("member1") // username = 'member1'
    member.username.ne("member1") // username != 'member1'
    member.username.eq("member1").not() // username != 'member1'
    
    member.username.isNotNull() // 이름이 is not null
    
    member.age.in(10,20) // age in (10,20)
    member.age.notIn(10,20) // age not in (10, 20)
    member.age.between(10,30) //between 10, 30
    
    member.age.goe(30) // age >= 30
    member.age.gt(30) // age > 30
    member.age.loe(30) // age <= 30
    member.age.lt(30) // age < 30
    
    member.username.like("member%") // like 검색
    member.username.contains("member") // like ‘%member%’ 검색
    member.username.startsWith("member") //like ‘member%’ 검색
    ...
    ```
  - AND 조건을 파라미터로 처리
    ```java
    // 기존 체이닝 방식
    Member findMember = queryFactory
      .selectFrom(member)
      // AND 조건 체이닝
      .where(member.username.eq("member1")
              .and(member.age.eq(10)))
      .fetchOne(); 

    // 파라미터 방식
    
    Member findMember = queryFactory
      .selectFrom(member)
      // AND 조건 파라미터
      .where(
            member.username.eq("member1"),
            member.age.eq(10))
      .fetchOne(); 
    ```
    - where()에 파라미터로 검색조건을 추가하면, AND 조건으로 동작한다.
    - 이 경우 null 값은 무시 -> 메서드 추출을 활용해서 동적 쿼리를 깔끔하게 작성 가능


- 결과 조회
  - fetch(): 리시트 조회, 데이터 없으면 빈 리스트 반환
  - fetchOne(): 단 건 조회
    - 결과가 없으면, null
    - 결과가 둘 이상이면, com.querdsl.core.NonUniqueResultException
  - fetchFirst(): limit(1).fetchOne()
  - @Deprecated fetchResults(): 페이징 정보 포함. total count쿼리 추가 실행
  - @Deprecated fetchCount(): count쿼리로 변경해서 count 수 조회
    - https://velog.io/@ok0/QueryDSL-FetchResults-is-deprecated


- 정렬
  ```java
  /**
   * 회원 정렬 순서
   * 1. 회원 나이 내림차순(desc)
   * 2. 회원 이름 오름차순(asc)
   * 단 2에서 회원 이름이 없으면, 마지막에 출력(nulls last)
   */
  @Test
  public void sort() {
      QMember member = QMember.member;

      em.persist(new Member(null, 100));
      em.persist(new Member("member5", 100));
      em.persist(new Member("member6", 100));

      List<Member> fetch = queryFactory
              .selectFrom(member)
              .where(member.age.eq(100))
              .orderBy(member.age.desc(), member.username.asc().nullsLast())
              .fetch();

      Member member5 = fetch.get(0);
      Member member6 = fetch.get(1);
      Member memberNull = fetch.get(2);

      assertThat(member5.getUsername()).isEqualTo("member5");
      assertThat(member6.getUsername()).isEqualTo("member6");
      assertThat(memberNull.getUsername()).isNull();
  }
  ```
  - desc(), asc(): 일반 정렬
  - nullsLast(), nullsFirst(): null 데이터 순서 여부


- 페이징
  ```java
  @Test
  public void paging1() {
      QMember member = QMember.member;
      List<Member> fetch = queryFactory
              .selectFrom(member)
              .orderBy(member.username.desc())
              .offset(1)
              .limit(2)
              .fetch();
      assertThat(fetch.size()).isEqualTo(2);
  }
  
  @Test
  public void paging2() {
      QMember member = QMember.member;
      QueryResults<Member> memberQueryResults = queryFactory
              .selectFrom(member)
              .orderBy(member.username.desc())
              .offset(1)
              .limit(2)
              .fetchResults();
      assertThat(memberQueryResults.getTotal()).isEqualTo(4);
      assertThat(memberQueryResults.getLimit()).isEqualTo(2);
      assertThat(memberQueryResults.getOffset()).isEqualTo(1);
      assertThat(memberQueryResults.getResults().size()).isEqualTo(2);
  }
  ```
  - 주의: fetchResults()는 count쿼리가 같이 실행되니 성능상 주의(모든 조건이 중복 실행된다.)
  - 참고: 실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 하지만, count쿼리는 조인이 필요 없는 경우도 있다. 그런데 이렇게 자동화된 count쿼리는 원본 쿼리와 같이 모두 조인을 해버리기 때문에 성능이 안나올 수 있다. count쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야한다.


- 집합
  - 집합 함수
    - JPQL이 제공하는 모든 집합 함수를 제공
    - Tuple은 프로젝션과 결과반환 강의에서 추후 설명
    - 종류
      - count(): 수
      - sum(): 합
      - avg(): 평균
      - max(): 최대
      - min(): 최소


- 조인
  - 기본 조인
    - 조인의 기본 문법은 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭(alias)으로 사용할 Q타입을 지정하면 된다.
      ```
      join(조인 대상, 별칭으로 사용할 Q타입)
      ```
    - join(), innerJoin(): 내부 조인(inner join)
    - leftJoin(): left 외부 조인(left outer join)
    - rightJoin(): right 외부 조인(right outer join)
    - JPQL의 on과 성능 최적화를 위한 fetch 조인 제공
  - 세타 조인
    - 연관 관계가 없는 필드로 조인
    - from 절에 여러 엔티티를 선택해서 세타 조인(theta join)
    - 외부 조인 불가능