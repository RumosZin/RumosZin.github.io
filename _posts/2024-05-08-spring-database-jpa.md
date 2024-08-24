---
title: "[Spring/Database] Spring JPA와 Spring Data JPA"
author:
date: 2024-05-08 23:30:00 +0900
categories:
  [
    Google Developer Student Club,
    Fairy Tale Island / 2024 Google Solution Challenge
  ]
tags: [Spring, Database, ORM]
---

✅ [🏝️ Fairy-Tale Island 🏝️](https://github.com/GDSC-CAU/FTIsland-BE) 리팩토링!

## **Spring Database 접근 방식**

Spring에서 데이터베이스에 접근하는 방식들을 알아본다. ORM 이전의 접근 방법들과 단점을 알아보고, 이를 보완하기 위해 등장한 ORM의 개념과 Spring에서 사용하는 Java ORM 기술을 알아본다.

#1 [[Database] ORM 정의, 등장 배경, 장단점](https://rumoszin.github.io/posts/database-orm/)

#2 [[Spring/Database] JDBC와 JDBCTemplate](https://rumoszin.github.io/posts/spring-database-jdbc-template/)

#3 [NOW] [[Spring/Database] Spring JPA와 Spring Data JPA](https://rumoszin.github.io/posts/spring-database-jpa/)

#4 [전체 코드 저장소 : Various way of Spring Database Access](https://github.com/RumosZin/spring-various-db-access)

<br>

## **데이터베이스 접근 : Spring JPA**

JPA(Java Persistence API)는 **Java ORM 기술에 대한 API 표준 명세 인터페이스이다.** 정의에서 알 수 있듯이, 특정 기능을 하는 라이브러리가 아니라, **인터페이스**이다. 단순히 Java 애플리케이션에서 관계형 데이터베이스에 접근하는 방법을 정의한 것이기 때문에, 구현이 아니다. **Hibernate와 같은 ORM 프레임워크에서 이용 가능한 공통 API를 제공한다.**

Java에서 인터페이스 : 구현체 관계처럼, **JPA : Hibernate도 인터페이스와 구현체 관계**라고 할 수 있다. Java ORM 기술에 대한 명세를 했으므로, **ORM의 장점을 가지고 있다.** 즉, Hibernate가 프로그래밍 언어 Java에서 Entity의 attribute를 관계형 데이터베이스 Table의 Column과 매핑을 지원하기 때문에, **개발자가 일일이 매핑을 위해 SQL문을 작성할 필요가 없다.**

### **Spring JPA 사용 방법**

`build.gradle`에 의존성을 추가한다.

```gradle
dependencies {
    ...

    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    ...
}
```

<br>

**Spring JPA는 `EntityManager` 객체를 이용한다. `EntityManger`도 JPA에서 정의된 인터페이스의 일부이고, 명시적으로 Java에 등록된 객체를 데이터베이스에 저장된 데이터와 매핑한다.** [JpaMemberRepository](https://github.com/RumosZin/spring-various-db-access/commit/adbae3634b6c4810f118ce1be958d111cb798f10)에서 `EntityManager` 객체를 생성자에 주입 받아 사용할 수 있다.

명시적으로 객체를 등록하려면, 도메인에 `@Entity` 어노테이션을 달아준다. `EntityManager`는 `@Entity` 어노테이션을 가지고 있는 객체들을 관리하고, 데이터베이스 테이블과 매핑하여 데이터를 Create / Update / Save 하는 기능들을 수행한다.

(여기서는 데이터베이스에 접근하는 다양한 방법들을 소개하는데 중점을 두므로, `Spring JPA`의 `EntityManager`에 대해서는 다른 글에서 자세히 다루겠다.)

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    ...
}
```

앞선 순수 `JDBC`, `JDBCTemplate`에서는 코드가 너무 길어 메서드 하나, 두 개만 첨부하였다. 하지만 `Spring JPA` 사용하면, 네 개 메서드 코드를 전부 첨부할 수 있을 정도로 코드가 간결하다.

```java
public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    public Member save(Member member) {
        em.persist(member);
        return member;
    }
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class).getResultList();
    }

    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }
}
```

이전 `JDBCTemplate`에서는 반복적인 데이터베이스 연결 코드를 작성하지 않아도 되었지만, SQL문은 직접 작성해야 했다. `Spring JPA`에서는 SQL문도 자동으로 작성한다. 하지만 `findAll`, `findByName`을 보면 알 수 있듯이, **모든 것을 JPA로 처리할 수 있는 것은 아니다.**

**개발을 하다보면 JPA의 Query Method 만으로는 조회가 불가능한 경우가 존재한다. 이러한 경우 JPQL(Java Persistence Query Language)를 이용할 수 있다.**

(`EntityManager`의 `createQuery`를 이용하는 방법 외 다양한 방법이 존재하는데, 이는 다른 글에서 자세히 소개하도록 하겠다.)

<br>

## **데이터베이스 접근 : Spring Data JPA**

### **Spring Data JPA 사용 방법**

`Spring JPA`와 같은 환경설정이다. `build.gradle`을 그대로 사용한다.

앞선 `Spring JPA`만 사용해도 SQL문을 직접 작성해야 하는 경우가 매우 줄어들고, 데이터베이스 연결도 자동으로 처리하기 때문에 비즈니스 로직 개발에 집중할 수 있었다. 여기에 추가적으로 `Spring Data JPA`를 사용하면, 구현체 없이 인터페이스만으로 개발을 완료할 수 있다!

아래 코드가 정말 구현의 전부이다!!

```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    Optional<Member> findByName(String name);
}

```

`Spring Data JPA`는 어떻게 인터페이스만으로 개발을 완료할 수 있는 것일까? 아래 그림의 4가지 요소는 모두 인터페이스이다. `JpaRepository` 인터페이스는 스프링 데이터 JPA에서, `PagingAndSortingReposiroty` / `CrudRepository` / `Repository` 인터페이스는 스프링 데이터에서 제공한다. 가장 아래의 `JpaRepository`에서 제공하는 인터페이스가 존재하므로, 이를 이용해 짜야 할 코드를 확 줄일 수 있다.

![Untitled](/assets/img/240107-2.png){: width="60%"}

Spring으로 개발하면서 앞선 `Spring JPA`에서 등장한 `EntityManager`를 직접 다루지 않았었다. 분명 ORM으로 `JPA`를 사용하고 있었는데, 직접 다룰 일이 왜 없을까 생각해보니, 데이터베이스에 접근하는 상황에서 `Repository`를 정의해서 접근했기 떄문이다!

`Spring Data JPA`는 JPA에 추가적인 기능을 제공해 개발을 간편하게 만드는 라이브러리/프레임워크이다. **이는 `Repository` 인터페이스를 제공함으로써 이루어진다! `Spring Data Jpa`가 궁극적으로 제공하는, 가장 윗 단의 `Repository` 인터페이스를 이용함으로써 JPA 기반 애플리케이션 개발을 간편하게 할 수 있었던 것이다.**

개발하면서 `Repository` 인터페이스에 정의된 대로 코드를 짜면, Spring이 알아서 메서드 이름에 적합한 쿼리를 보내는 구현체를 만들어서 `Bean`으로 등록한다. 그림에서 `Repository` 인터페이스와 `JpaRepository` 인터페이스 사이에 있는 인터페이스들을 통해서 CRUD 연산, 페이징, 정렬을 쉽게 할 수 있고, 코드의 양을 획기적으로 줄일 수 있다.

### **이미지 출처**

- [https://www.geeksforgeeks.org/spring-boot-difference-between-crudrepository-and-jparepository/](https://www.geeksforgeeks.org/spring-boot-difference-between-crudrepository-and-jparepository/)
