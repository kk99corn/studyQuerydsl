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