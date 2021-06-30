---
tags: 
    - java
    - jpa
    - spring
    - sqlite
    - datasource
    - cipher
title: sqlite cipher 적용하기
date: 2021/06/30
author: 김동환
description: sqlite-cihper를 spring의 DataSource로 주입하여 JPA를 활용하기
disabled: false
categories:
  - java
---

　
# 목표

sqlite-cihper를 spring의 DataSource로 주입하여 JPA를 활용하기

1. Gradle Dependency 설정
2. SQLite Config 설정
3. @Bean DataSource 주입
4. JPA 활용

　
# 테스트 환경

- spring-boot 2.5.2
- gradle 7.0.2

　
# Gradle

## build.gradle

sqlite-cipher를 활용하기 위해 `io.github.willena:sqlite-jdbc` 를 추가하며, dialect는 `com.github.gwenn:sqlite-dialect` 를 활용한다.

```groovy
dependencies {
    // ...
    
    /* SQLITE */
    implementation 'io.github.willena:sqlite-jdbc:3.35.5.3'
    implementation 'com.github.gwenn:sqlite-dialect:0.1.2'
    
    // ... 
}
```

　
# Spring DataSource

Spring에서 기본적으로 사용하는 DataSource를 사용하기엔 sqlite-cipher가 부적절(Password Schema 적용)하여 DataSource의 connection 정보를 따로 정의하고 이를 주입시켜 JPA에서 활용할 수 있도록 한다.

```java
@Slf4j
@Configuration
public class ApplicationDatabase {

    @Value("${spring.datasource.url:jdbc:sqlite:example.db}")
    private String url;
    
    @Value("${spring.datasource.password:7eff056138424802bbe296fe66022047}")
    private String password;
    
    private SQLiteMCConfig sqLiteConfig;
    
    private Connection connection;

    @PostConstruct
    protected void initSqliteConfig() {
        this.sqLiteConfig = SQLiteMCSqlCipherConfig
            .getV4Defaults()
            .withKey(password);
        this.sqLiteConfig.setJournalMode(SQLiteConfig.JournalMode.WAL);
    }

    @Bean
    public DataSource getDataSource() {
        try {
            this.connection = sqLiteConfig.createConnection(url);
        } catch (SQLException | IllegalStateException e ) {
            e.printStackTrace();
        }
    
        return new SingleConnectionDataSource(connection, true);
    }
}
```

### URL & PASSWORD

sqlite의 url과 password는 기존 application.yml에서 쉽게 사용할 수 있도록 `@Value`를 통해서 주입시킨다. 값이 없을 경우를 대비해 기본 값도 설정해 두었다.

```java
@Value("${spring.datasource.url:jdbc:sqlite:example.db}")
private String url;

@Value("${spring.datasource.password:7eff056138424802bbe296fe66022047}")
private String password;

// application.yml
spring
    datasource:
        url: jdbc:sqlite:example.db
        password: 7eff056138424802bbe296fe66022047
```

### SQLite Config

`@PostConstruct`를 활용하여 기본 템플릿을 제공해주는 `SQLiteMCConfig` 를 활용하여 원하는 sqlite config(Pragma)를 설정하였다. 아래는 `SQLCipher4`와 password를 설정하였으며. JournalMode를 설정하였다.

```java
private SQLiteMCConfig sqLiteConfig;

@PostConstruct
protected void initSqliteConfig() {
    this.sqLiteConfig = SQLiteMCSqlCipherConfig
        .getV4Defaults()
        .withKey(password);
    this.sqLiteConfig.setJournalMode(SQLiteConfig.JournalMode.WAL);
}
```

　
![sqlite-table](/assets/images/sqlite-cipher/sqlite-cipher.jpg)

　
![sqlite-table](/assets/images/sqlite-cipher/pragma.jpg)

### DataSource

위에서 적용한 SQLite config를 토대로 Connection을 생성한다. 생성한 Connection을 Spring에서 제공하는 `SingleConnectionDataSource` 를 활용하여 DataSource를 주입하였다. 좀 더 디테일한 Connection 처리가 필요한 경우 `AbstractDataSource`를 상속받아 구현해주면 된다.

```java
private Connection connection;

@Bean
public DataSource getDataSource() {
    try {
        this.connection = sqLiteConfig.createConnection(url);
    } catch (SQLException | IllegalStateException e ) {
        e.printStackTrace();
    }
    
    return new SingleConnectionDataSource(connection, true);
}
```

　
# JPA

SQLite에서 활용할 Entity와 Repository를 구현한다.

### Entity

```java
@Entity
@Table(name = "ot_config")
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Config {
    @Id
    private String key;
    
    @Column(columnDefinition = "TEXT")
    private String value;
}
```

```bash
Hibernate: create table ot_config (key varchar(255) not null, value TEXT, primary key (key))
```

　
![sqlite-table](/assets/images/sqlite-cipher/sqlite-table.png)

### Repository

```java
@Repository
public interface ConfigRepository extends JpaRepository<Config, String> {
    Optional<Config> findConfigByKey(String key);
}
```