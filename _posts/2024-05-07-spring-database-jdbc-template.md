---
title: "[Spring/Database] JDBC와 JDBCTemplate"
author: 
date: 2024-05-07 22:19:00 +0900
categories: [Spring, DB]
tags: [Spring, Database, ORM]
---

✅ [🏝️ Fairy-Tale Island 🏝️](https://github.com/GDSC-CAU/FTIsland-BE) 리팩토링!

## **Spring Database 접근 방식**

Spring에서 데이터베이스에 접근하는 방식들을 알아본다. ORM 이전의 접근 방법들과 단점을 알아보고, 이를 보완하기 위해 등장한 ORM의 개념과 Spring에서 사용하는 Java ORM 기술을 알아본다.

#1 [[Database] ORM 정의, 등장 배경, 장단점](https://rumoszin.github.io/posts/database-orm/)

#2 [NOW] [[Spring/Database] JDBC와 JDBCTemplate](https://rumoszin.github.io/posts/spring-database-jdbc-template/)

#3 [[Spring/Database] Spring JPA와 Spring Data JPA](https://rumoszin.github.io/posts/spring-database-jpa/)

#4 [전체 코드 저장소 : Various way of Spring Database Access](https://github.com/RumosZin/spring-various-db-access)

<br>

## **데이터베이스 접근 : JDBC**

JDBC(Java Database Connectivity)는 Java 기반 애플리케이션의 데이터를 데이터베이스에 저장 및 업데이트하거나, 저장된 데이터를 Java에서 사용할 수 있도록 하는 자바 API이다.

Spring에서 순수 JDBC를 이용해 데이터베이스에 접근하는 방법을 알아보자.

### **Spring JDBC 사용 방법**

`build.gradle`에 의존성을 추가한다.
```gradle
dependencies {
    ...

    implementation 'org.springframework.boot:spring-boot-starter-jdbc'

    ...
}
```

<br>

이전에 개발을 시작할 당시에는 어떤 데이터베이스를 사용할 지 정해지지 않아서, 인터페이스 [MemberRepository](https://github.com/RumosZin/spring-various-db-access/commit/ff8981c6e439a1b3320848428f7e6a7a3b575310#diff-2b4a09be69a8dd2064d1d8d2df9b7689677c3287bc5f05987fcd162c8f1bca6e)를 구현해놓은 상태였다. 데이터베이스가 정해지고, 애플리케이션과 데이터베이스 사이의 연결 다리인 [JDBCMemberRepository](https://github.com/RumosZin/spring-various-db-access/commit/35c5f0ae873e76cd4a642994ecabd49060c36b5b#diff-83e66a80230931b6d66beac82938cc5be246408621dfffad8392a55d6202a166)를 구현하였다.

JDBC를 이용한 구현의 경우, 생성자에서 `DataSource` 주입을 필요로 한다.

```java
public class JDBCMemberRepository implements MemberRepository {

    private final DataSource dataSource;

    public JDBCMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";

        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
            pstmt.setString(1, member.getName());
            pstmt.executeUpdate();
            rs = pstmt.getGeneratedKeys();
            if (rs.next()) {
                member.setId(rs.getLong(1));
            } else {
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }

    { ... }
}
```


`String sql = "insert into member(name) values(?)";`처럼 SQL문을 직접 구현해야 하고, `try-catch`문과 같이 모든 메서드에 데이터베이스 연결 구문을 작성해야 했다.

스키마가 확장될 떄마다 SQL문을 수정해야 하는 점은 소프트웨어 개발 원칙에서 Open-Closed 원칙에도 어긋나고, 개발 비용을 증가시킨다. `try-catch` 안의 데이터베이스 연결 코드가 반복되는 점도 대표적인 불편한 점이다.

<br>

## **데이터베이스 접근 : JDBCTemplate**

위와 같은 순수 JDBC 사용의 문제점을 줄여주는 JDBCTemplate가 있다.

### **Spring JDBCTemplate 사용 방법**

순수 JDBC와 같은 환경설정이다. `build.gradle`을 그대로 사용한다.

JDBCMemberRepository와 마찬가지로, MemberRepository의 구현체인 [JDBCTemplateMemberRepository](https://github.com/RumosZin/spring-various-db-access/commit/5f1bd7468e52b4655861031d5b889b2a7bd8daeb#diff-e777de624a114e0580fb693da7c37748fd04b00be6b0986af52637c63234e194)를 작성한다.

```java
public class JDBCTemplateMemberRepository implements MemberRepository {
    private final JdbcTemplate jdbcTemplate;

    public JDBCTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());
        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }

    {...}
}
```

이전 순수 JDBC를 사용할 떄는 모든 메서드 구현마다 `try-catch`를 사용해서 데이터베이스 연결을 설정해야 했는데, JDBCTemplate에서는 그렇지 않다. 확연히 연결 설정와 관련된 중복 코드가 삭제되었다! 

하지만 `List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);`과 같이, SQL문은 직접 작성해야 한다.

<br>

## **Spring의 관리 방법**

위와 같이 순수 JDBC 방식으로 데이터베이스에 접근하다가, JDBCTemplate로 접근 방법을 변경하고자 한다면, 관련된 모든 코드들을 수정해야 할까? 답은 **전혀 아니다.**

앞서 데이터베이스가 정해지지 않아서, MemberRepository 인터페이스를 구현했다고 했다. 인터페이스로 먼저 구현해야 할 메서드들을 정의만 해두고, 구현체로 JDBCMemberRepository와 JDBCTemplateMemberRepository가 존재하는 것이다. **`SpringConfig` 설정만으로, 다른 코드들을 전혀 손대지 않고 설정 파일만으로 구현체를 바꿀 수 있다!** **SOLID 개발 원칙의 Open-Closed 원칙을 실현하는 것이다!** 어떻게 확장에는 열려 있고 수정에는 닫혀 있는지 확인하자.

```java
@Configuration
public class SpringConfig {
    private final DataSource dataSource;

    public SpringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        // JDBC
        return new JDBCMemberRepository(dataSource);

        // JDBCTemplate
        return new JDBCTemplateMemberRepository(dataSource);
    }
}
```

`public MemberRepository memberRepository()` 인터페이스로 구현한 MemberRepository의 **구현체를 무엇으로 할 건지 반환만 변경하면, 아래 그림과 같이 바로 구현체를 갈아끼울 수 있다! 스프링의 Dependency Injection(의존성 주입)의 큰 장점이다.**

<br>

![Untitled](/assets/img/240107-1.png){: width="100%"}