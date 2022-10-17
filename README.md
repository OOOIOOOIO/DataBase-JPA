# DataBase-JPA

# JPA란

# JPA Maven 설정

## 1. Maven에 dependency(라이브러리) 추가
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>jpa-basic</groupId>
    <artifactId>ex-hello-jpa</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- JPA 하이버네이트 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.3.10.Final</version>
        </dependency>
        <!-- H2 데이터베이스 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.1.214</version>
        </dependency>
        <!-- Java11 사용할 때 추가-->
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.3.0</version>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

</project>
```

<br>

## 2. resource에META-INF/persistence.xml 파일 생성(resource/webapp/)

![image](https://user-images.githubusercontent.com/74396651/196167230-3eddbacd-5269-4764-89bd-5780aa9b376f.png)


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
version : JAP 버전
persistence-unit : 보통 DB 당 1개씩 생성
dialect : SQL 방언

-->
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션
                show : 쿼리 보이기
                format : 예쁘게
                comments : 주석
             -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```

## 3. Entity 클래스
```java
package hellojpa;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity // JPA를 사용하는 것 인식
@Table(name = "MEMBER") // 테이블 명시
public class Member {
    /*
        java 변수명은 table의 변수명과 같게 하는게 편하다. 마바처럼
     */
    @Id // PK 명시
    private Long id;

    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

## 4. 메인 클래스
```java
package hellojpa;

import javax.persistence.*;
import java.util.List;

/*
JPA는 트랜잭션이 중요하다. 모든 데이터의 변경은 트랜잭션 단위로 수행해야 한다.
 */
public class JpaMain {
    public static void main(String[] args) {
        // persistence.xml에 설정한 DB 불러오기. name = hello
        // 하나만 생성해서 애플리케이션 전체에서 공유한다.
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        
        // DB 연결, Java Connection이랑 비슷하다고 생각하면 된다.
        // EntityManager는 쓰레드 간에 절대 공유하면 안된다. 사용하고 버려야 한다.
        EntityManager em = emf.createEntityManager();

        // Transaction 얻기
        EntityTransaction tx = em.getTransaction();

        // Transaction 시작
        tx.begin();

        // code
        Member member = null;
        try {
            // insert
            member = new Member();
            member.setId(1L);
            member.setName("helloA");
            em.persist(member);

            // select
            Member findMember = em.find(Member.class, 1L);
            System.out.println("findMember = " + findMember.getId());
            System.out.println("findMember = " + findMember.getName());

            // Query로 select, JPQL 객체지향 쿼리 언어, SQL 문법 대부분 지원
            // JPQL은 엔티티 객체를 대상으로
            // SQL은 데이터베이스 테이블을 대상으로 쿼리를 짠다.
            List<Member> query = em.createQuery("select m from Memer as m", Member.class)
                    .setFirstResult(0) // offset
                    .setMaxResults(5) // limit
                    .getResultList();


            // delete
            em.remove(findMember);

            // update, 값이 바뀌면 알아서 해줌
            findMember.setName("bye1");

            // 커밋
            tx.commit();
        } catch (Exception e) {
            // 실패시 롤백
            tx.rollback();
            throw new RuntimeException(e);
        }finally {
            // DB 커넥션 닫기
            em.close();
        }

        emf.close();
    }

}

```


<br>

# JAP Gradle 설정

# JPQL
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화하기에 특정 데이터베이스 SQL에 의존하지 않는다.
- 한마디로 JPQL은 객체지향 SQL이다.
