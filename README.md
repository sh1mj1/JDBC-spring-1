# JDBC-spring-1
# 1. 프로젝트 생성 & H2 데이터베이스 설정

### 프로젝트 생성

프로젝트를 생성해줍니다. 그 후 해당 프로젝트의 build.gradle 을 open as project 합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1702a43-3dae-46e8-8a7a-1aad9ec5f8ac/Untitled.png)

build.gradle 에 아래 코드를 추가해줍니다.

```java
// 테스트에서 lombok 사용
testCompileOnly 'org.projectlombok:lombok'
testAnnotationProcessor 'org.projectlombok:lombok'
```

서버 실행 후 로그

```java
Started JdbcApplication in 0.493 seconds (process running for 0.69)
```

위 로그가 확인되면 정상 동작하는 것입니다.

### H2 데이터베이스 설정

H2 데이터베이스는 개발이나 테스트 용도로 사용하기 좋은 가볍고 편리한 DB 입니다. 

그리고 SQL 을 실행할 수 있는 앱 화면을 제공합니다.

https://www.h2database.com

다운로드 및 설치

h2 데이터베이스 버전은 스프링 부트 버전에 맞춥니다.  우리 스프링 프로젝트의  External Libraries 에서 2.1.214 로 버전을 확인할 수 있네요.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e08f5e67-5c02-46c4-a720-973bbf3158bb/Untitled.png)

아래 링크에서 다양한 H2 다운로드 버전을 확인할 수 있습니다. (https://www.h2database.com/html/download-archive.html)

MAC, 리눅스 사용자는 터미널 명령어로 

`chmod 755 [h2.sh](http://h2.sh)` 로 권한을 준 후 `./h2.sh` 로 실행하면 됩니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d576b8e9-8380-456d-8d88-bc1973a05cd9/Untitled.png)

윈도우 사용자는 `h2.bat` 으로 실행하면 됩니다.

실행된 모습

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/434fbd7a-627a-4431-a460-d2b3033227a2/Untitled.png)

- 데이터베이스 파일 생성 방법
    - 사용자명은 `sa` 입력
    - JDBC URL 에 다음 입력. `jdbc:h2:~/test` (최초 한번)
    - `~/test.mv.db` 파일 생성 확인
    - 이후부터는 `jdbc:h2:tcp://localhost/~/test` 이렇게 접속하면 됩니다.

참고 - H2 데이터베이스가 정상 생성되지 않을 때

JDBC URL 에 `jdbc:h2:~/test` 을 입력해도 오류가 나오면 H2 데이터베이스가 생성되지 않는 경우도 있음.

해결방안은 아래와 같습니다.

.1. H2 데이터베이스를 종료하고 다시 시작

.2. 웹 브라우저가 자동 실행되면 주소창의 port의 바로 앞부분(숫자로 되어있다면) 을 [`localhost`](http://localhost) 로 변경하고 Enter 입력한다. 아래 그림처럼 말이죠.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/62a510fe-10e3-4df5-95ab-924ac9252b8e/Untitled.png)

.3. 이제 JDBC URL 에 `jdbc:h2:~/test` 을 입력하면 데이터베이스가 정상 생성됩니다.

.4. 이후에는 `jdbc:h2:tcp://localhost/~/test` 로 접속합니다.

h2 데이터베이스 접속에 성공하면 간단히 `member` 라는 테이블을 만들어봅시다. 아래처럼 만들면 됩니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ada07e16-85ea-429f-b8c6-358a2bc67f70/Untitled.png)

```java
drop table member if exists cascade;
create table member (
	member_id varchar(10),
	money integer not null default 0,
	primary key (member_id)
);

insert into member(member_id, money) values ('hi1', 10000);
insert into member(member_id, money) values ('hi2', 20000);
```

결과를 보면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e85a001-cd70-4de8-be63-a135f746df3b/Untitled.png)

이제 환경 설정은 끝이 났습니다.

# 2. JDBC 이해

### **JDBC 등장 이유**

우리는 애플리케이션을 개발할 때 중요한 데이터는 대부분 데이터베이스에 보관합니다.

**클라이언트, 애플리케이션 서버, DB**

![https://user-images.githubusercontent.com/52024566/183674051-a513c144-6714-4f76-8cfe-2c8de17b4215.png](https://user-images.githubusercontent.com/52024566/183674051-a513c144-6714-4f76-8cfe-2c8de17b4215.png)

클라이언트가 애플리케이션 서버를 통해 데이터를 저장하거나 조회하면, 애플리케이션 서버는 다음 과정을 통해서 데이터베이스를 사용합니다.

**애플리케이션 서버와 DB - 일반적인 사용법**

![https://user-images.githubusercontent.com/52024566/183674245-6c16576c-ab7b-449e-b22f-6158c5badf9d.png](https://user-images.githubusercontent.com/52024566/183674245-6c16576c-ab7b-449e-b22f-6158c5badf9d.png)

.1. 커넥션 연결: 주로 TCP/IP를 사용해서 커넥션을 연결

.2. SQL 전달: 애플리케이션 서버는 DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달

.3. 결과 응답: DB는 전달된 SQL을 수행하고 그 결과를 응답. 애플리케이션 서버는 응답 결과를 활용

**애플리케이션 서버와 DB - DB 변경**

![https://user-images.githubusercontent.com/52024566/183674250-3812e2cb-bb18-4365-a283-b4aea0d2c0b8.png](https://user-images.githubusercontent.com/52024566/183674250-3812e2cb-bb18-4365-a283-b4aea0d2c0b8.png)

그런데 각각의 데이터베이스마다 커넥션을 연결하는 방법, SQL을 전달하는 방법, 그리고 결과를 응답 받는 방법이 모두 다릅니다! 

참고로 관계형 데이터베이스는 수십 개가 존재합니다. 

여기에는 2가지 큰 문제가 있습니다.

.1. 데이터베이스를 다른 종류의 데이터베이스로 변경하면 애플리케이션 서버에 개발된 데이터베이스 사용 코드도 함께 변경해야 함.

.2. 개발자가 각각의 데이터베이스마다 커넥션 연결, SQL 전달, 그리고 그 결과를 응답 받는 방법을 새로 학습해야 함

이런 문제를 해결하기 위해 JDBC라는 자바 표준이 등장했습니다.

### **JDBC 표준 인터페이스**

> **JDBC(Java Database Connectivity)는 자바에서 데이터베이스에 접속할 수 있도록 하는 자바 API** 입니다. JDBC 는 데이터베이스에서 자료를 쿼리하거나 업데이트하는 방법을 제공합니다. - 위키백과
> 

**JDBC 표준 인터페이스**

![https://user-images.githubusercontent.com/52024566/183674251-9c6d9e42-beee-4eea-ba1d-bcc9e16cfa98.png](https://user-images.githubusercontent.com/52024566/183674251-9c6d9e42-beee-4eea-ba1d-bcc9e16cfa98.png)

대표적으로 다음 3가지 기능을 표준 인터페이스로 정의해서 제공합니다.

- `java.sql.Connection` - 연결
- `java.sql.Statement` - SQL을 담은 내용
- `java.sql.ResultSet` - SQL 요청 응답

자바는 이렇게 표준 인터페이스를 정의했으므로 이제부터 개발자는 이 표준 인터페이스만 사용해서 개발하면 됩니다.

그런데 인터페이스만 있다고해서 기능이 동작하지는 않습니다. 이 JDBC 인터페이스를 각각의 DB 벤더 (회사)에서 자신의 DB에 맞도록 구현해서 라이브러리로 제공하는데, 이것을 JDBC 드라이버라 합니다.

예를 들어서 MySQL DB에 접근할 수 있는 것은 MySQL JDBC 드라이버라 하고, Oracle DB에 접근할 수 있는 것은 Oracle JDBC 드라이버라고 합니다.

**MySQL 드라이버 사용**

![https://user-images.githubusercontent.com/52024566/183675469-ec3e616d-2815-47b1-a655-0a0bada71531.png](https://user-images.githubusercontent.com/52024566/183675469-ec3e616d-2815-47b1-a655-0a0bada71531.png)

**Oracle 드라이버 사용**

![https://user-images.githubusercontent.com/52024566/183674257-fbd99797-4cd2-48cc-aad9-835afb28ead5.png](https://user-images.githubusercontent.com/52024566/183674257-fbd99797-4cd2-48cc-aad9-835afb28ead5.png)

### **정리**

JDBC의 등장으로 다음 2가지 문제가 해결되었습니다.

.1. 데이터베이스를 다른 종류의 데이터베이스로 변경하면 애플리케이션 서버의 데이터베이스 사용 코드도 함께 변경해야 하는 문제.

        .-. 애플리케이션 로직은 이제 JDBC 표준 인터페이스에만 의존. 따라서 데이터베이스를 다른 종류의 데이터베이스로 변경하고 싶으면 JDBC 구현 라이브러리만 변경하면 됨. 따라서 다른 종류의 데이터베이스로 변경해도 애플리케이션 서버의 사용 코드를 그대로 유지.

.2. 개발자가 각각의 데이터베이스마다 커넥션 연결, SQL 전달, 그리고 그 결과를 응답 받는 방법을 새로 학습해야하는 문제.

        .-. 개발자는 JDBC 표준 인터페이스 사용법만 학습. 한번 배워두면 수십 개의 데이터베이스에 모두 동일하게 적용 가능

**참고 - 표준화의 한계**

> JDBC의 등장으로 많은 것이 편리해졌지만, 각각의 데이터베이스마다 SQL, 데이터타입 등의 일부 사용법이 다릅니다.
ANSI SQL이라는 표준이 있기는 하지만 일반적인 부분만 공통화했기 때문에 한계가 있습니다. 
대표적으로 실무에서 기본으로 사용하는 페이징 SQL은 각각의 데이터베이스마다 사용법이 다릅니다. 결국 데이터베이스를 변경하면 JDBC 코드는 변경하지 않아도 되지만 SQL은 해당 데이터베이스에 맞도록 변경해야 합니다. 
참고로 **JPA(Java Persistence API)를 사용하면 이렇게 각각의 데이터베이스마다 다른 SQL을 정의해야 하는 문제도 많은 부분 해결**할 수 있습니다.
>

# 3. JDBC 와 최신 데이터 접근 기술

- JDBC는 1997년에 출시될 정도로 오래된 기술이고, 사용하는 방법도 복잡합니다.
- 그래서 최근에는 JDBC를 직접 사용하기 보다는 JDBC를 편리하게 사용하는 다양한 기술이 존재합니다.
- 대표적으로 SQL Mapper와 ORM 기술로 나눌 수 있습니다.

### **JDBC 직접 사용**

![https://user-images.githubusercontent.com/52024566/183923003-97532bb6-b8e8-4de5-8a19-2a00e46930e5.png](https://user-images.githubusercontent.com/52024566/183923003-97532bb6-b8e8-4de5-8a19-2a00e46930e5.png)

### **SQL Mapper**

![https://user-images.githubusercontent.com/52024566/183923011-7b33209f-bb73-4b3c-ac09-a3f447642c89.png](https://user-images.githubusercontent.com/52024566/183923011-7b33209f-bb73-4b3c-ac09-a3f447642c89.png)

SQL Mapper 의 장,단점은 아래와 같습니다.

- 장점: JDBC를 편리하게 사용하도록 도와줍니다.
    - SQL 응답 결과를 객체로 편리하게 변환할 수 있습니다.
    - JDBC의 반복 코드를 제거합니다.

- 단점: 개발자가 SQL을 직접 작성해야 합니다.
- 대표 기술: 스프링 JdbcTemplate, MyBatis

### **ORM 기술**

![https://user-images.githubusercontent.com/52024566/183923012-b8da7061-c8d2-45ae-aabf-5f5e2fadd76e.png](https://user-images.githubusercontent.com/52024566/183923012-b8da7061-c8d2-45ae-aabf-5f5e2fadd76e.png)

ORM 기술의 특징입니다.

- ORM은 객체를 관계형 데이터베이스 테이블과 매핑해주는 기술입니다.
- 이 기술 덕분에 개발자는 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신에 SQL을 동적으로 만들어 실행할 수 있습니다. 추가로 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결됩니다.
- 대표 기술: JPA, 하이버네이트, 이클립스링크
- JPA는 자바 진영의 ORM 표준 인터페이스이고, 이것을 구현한 것으로 하이버네이트와 이클립스 링크 등의 구현 기술이 있습니다.

### **SQL Mapper vs ORM 기술**

**SQL Mapper** 와 **ORM** 기술 둘다 각각 장단점이 있음 쉽게 설명하자면 SQL Mapper는 SQL만 직접 작성하면 나머지 번거로운 일은 SQL Mapper가 대신 해결해줍니다. 즉, SQL Mapper는 SQL만 작성할 줄 알면 금방 배워서 사용할 수 있습니다. 

ORM기술은 SQL 자체를 작성하지 않아도 되어서 개발 생산성이 매우 높아집니다. 물론 편리한 반면에 쉬운 기술은 아니므로 실무에서 사용하려면 깊이있게 학습해야 합니다.

> 중요 이런 기술들도 **내부에서는 모두 JDBC를 사용합니다**. 따라서 JDBC를 직접 사용하지는 않더라도, JDBC가 어떻게 동작하는지 기본 원리를 알아두어야 합니다. 그래야 해당 기술들을 더 깊이있게 이해할 수 있고, 무엇보다 문제가 발생했을 때 근본적인 문제를 찾아서 해결할 수 있습니다. 
**JDBC는 자바 개발자라면 꼭 알아두어야 하는 필수 기본 기술입니다.**
>
이전 글에서 jdbc 커넥션을 얻어서 h2 데이터베이스에 연결했습니다.

이제 JDBC 을 사용하여 `DriverManager` 로 직접 커넥션을 얻어와서 DB 에 쿼리를 날려보는 로직을 만들어볼 것입니다.

# 5. JDBC 개발 - 등록

`schema.sql`

```sql
drop table member if exists cascade;
create table member (
    member_id varchar(10),
    money integer not null default 0,
    primary key (member_id)
);
```

위처럼 `member` 테이블을 만들어줍니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/228bcb99-91e5-4531-9022-1222d4037af9/Untitled.png)

`Member`

```java
package hello.jdbc.domain;

import lombok.Data;

@Data
public class Member {
    private String memberId;
    private int money;

    public Member() {
    }

    public Member(String memberId, int money) {
        this.memberId = memberId;
        this.money = money;
    }
}
```

회원의 ID와 해당 회원이 소지한 금액을 표현하는 단순한 클래스입니다. 

앞서 만들어둔 `member` 테이블에 데이터를 저장하고 조회할 때 사용할 것입니다.

`MemberRepositoryV0` - 회원 등록

```java
package hello.jdbc.repository;

import hello.jdbc.connection.DBConnectionUtil;
import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;

import java.sql.*;

@Slf4j
public class MemberRepositoryV0 {
    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values (?, ?)";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());
            pstmt.setInt(2, member.getMoney());
            pstmt.executeUpdate();
            return member;
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }

        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
        if (con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }

    }

    private Connection getConnection() {
        return DBConnectionUtil.getConnection();
    }
}
```

커넥션을 얻어 쿼리를 날리는 Repository 입니다.

로직을 요약하면 아래와 같습니다.

커넥션 획득

- `getConnection()`: 이전에 만들어둔 `DBConnectionUtil`를 통해서 데이터베이스 커넥션을 획득

`save()` - SQL 전달

- `sql`: 데이터베이스에 전달할 SQL을 정의함. 여기서는 데이터를 등록해야 하므로 insert sql 을 준비함.
- `con.prepareStatement(sql)` : 데이터베이스에 전달할 SQL과 파라미터로 전달할 데이터들을 준비
    - `sql`: `insert into member(member_id, money) values(?, ?)"`
    - `pstmt.setString(1, member.getMemberId())`: SQL의 첫번째 패러미터인 `?` 에 값을 지정함. 문자이므로 `setString`을 사용함.
    - `pstmt.setInt(2, member.getMoney())`: SQL의 두번째 패러미터인 `?` 에 값을 지정한다. `Int`형 숫자이므로 `setInt`를 지정함.
    - `pstmt.executeUpdate()`: `Statement`를 통해 준비된 SQL을 커넥션을 통해 실제 데이터베이스에 전달함. 
    참고로 `executeUpdate()`은 `int`를 반환하는데 영향받은 DB row 수를 반환함. 여기서는 하나의 row를 등록했으므로 1을 반환

### executeUpdate()

`int executeUpdate() throws SQLException;`

리소스 정리 쿼리를 실행하고 나면 리소스를 정리해야 합니다. 

여기서는 `Connection`, `reparedStatement`를 사용했습니다. 리소스를 정리할 때는 항상 역순으로 해야 합니다. 

`Connection`을 먼저 획득하고 `Connection`을 통해 `PreparedStatement`를 만들었기 때문에 리소스를 반환할 때는 `PreparedStatement`를 먼저 종료하고, 그 다음에 `Connection`을 종료해야 합니다.

> 주의 - 리소스 정리는 꼭! 해주어야 합니다.
> 

따라서 예외가 발생하든, 하지 않든 항상 수행되어야 하므로 `finally` 구문에 주의해서 작성해야 합니다. 

만약 이 부분을 놓치게 되면 커넥션이 끊어지지 않고 계속 유지되는 문제가 발생할 수 있습니다. 이런 것을 리소스 누수라고 하는데, 결과적으로 커넥션 부족으로 장애가 발생할 수 있습니다.

> 참고 - PreparedStatement 는 Statement 의 자식 타입인데, ? 를 통한 파라미터 바인딩을 가능하게 해줍니다.
> 

참고로 SQL Injection 공격을 예방하려면 `PreparedStatement` 를 통한 파라미터 바인딩 방식을 사용해야 한다.

`MemberRepositoryV0Test` - 회원 등록

```java
package hello.jdbc.repository;

import hello.jdbc.domain.Member;
import org.junit.jupiter.api.Test;

import java.sql.SQLException;

import static org.junit.jupiter.api.Assertions.*;

class MemberRepositoryV0Test {
    MemberRepositoryV0 repository = new MemberRepositoryV0();

    @Test
    void crud() throws SQLException {
        // save
        Member member = new Member("memberV0", 10000);
        repository.save(member);
    }
}
```

실행 결과 데이터베이스에서 `select * from member` 쿼리를 실행하면 데이터가 저장된 것을 확인할 수 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f28a395a-4ab9-4c6b-adc9-2e9f7a332726/Untitled.png)

참고로 이 테스트는 2번 실행하면 PK 중복 오류가 발생하므로 `delete from member` 쿼리로 데이터를 삭제한 후에 다시 실행해야 합니다.

PK 중복 오류

`org.h2.jdbc.JdbcSQLIntegrityConstraintViolationException: Unique index`


# 6. JDBC 개발 - 조회

이제 조회를 해봅시다.

`MemberRepositoryV0` - 회원 조회 메서드 추가

```java
...
public Member findById(String memberId) throws SQLException {
    String sql = "select * from member where member_id = ?";

    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try {
        con = getConnection();
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, memberId);

        rs = pstmt.executeQuery();

        if (rs.next()) {
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        } else {
            throw new NoSuchElementException("member not found memberId=" + memberId);
        }
    } catch (SQLException e) {
        log.error("db error", e);
        throw e;
    } finally {
        close(con, pstmt, rs);
    }
}
...
```

`findById()` - 쿼리 실행

- `sql`: 데이터 조회를 위한 select SQL을 준비합니다.
- `rs = pstmt.executeQuery()` 데이터를 변경할 때는 `executeUpdate()`를 사용하지만, 데이터를 조회할 때는 `executeQuery()`를 사용합니다. `executeQuery()`는 결과를 `ResultSet`에 담아서 반환합니다.

`executeQuery()`

```java
ResultSet executeQuery() throws SQLException;
```

`ResultSet`

![https://user-images.githubusercontent.com/52024566/184147645-b7244af0-0692-47d9-8039-7504cbe98358.png](https://user-images.githubusercontent.com/52024566/184147645-b7244af0-0692-47d9-8039-7504cbe98358.png)

- `ResultSet`은 위처럼 생긴 데이터 구조입니다. 보통 select 쿼리의 결과가 순서대로 들어갑니다.
    - 예를 들어서 `select member_id, money`라고 지정하면 `member_id`, `money`라는 이름으로 데이터가 저장됩니다.
    - 참고로 `select *`을 사용하면 테이블의 모든 컬럼을 다 지정하는 것입니다.

- `ResultSet` 내부에 있는 커서(`cursor`)를 이동해서 다음 데이터를 조회합니다.
- `rs.next()`: 이것을 호출하면 커서가 다음으로 이동합니다. 
참고로 최초의 커서는 데이터를 가리키고 있지 않기 때문에 `rs.next()`를 최초 한번은 호출해야 데이터를 조회할 수 있습니다.
    - `rs.next()`의 결과가 `true`면 커서의 이동 결과 데이터가 있음.
    - `rs.next()`의 결과가 `false`면 더 이상 커서가 가리키는 데이터가 없음.

- `rs.getString("member_id")`: 현재 커서가 가리키고 있는 위치의 `member_id` 데이터를 `String` 타입으로 반환합니다.
- `rs.getInt("money")`: 현재 커서가 가리키고 있는 위치의 `money` 데이터를 `int` 타입으로 반환합니다.

**ResultSet 결과 예시**

![https://user-images.githubusercontent.com/52024566/184147645-b7244af0-0692-47d9-8039-7504cbe98358.png](https://user-images.githubusercontent.com/52024566/184147645-b7244af0-0692-47d9-8039-7504cbe98358.png)

참고로 이 ResultSet 의 결과 예시는 회원이 2명 조회되는 경우입니다.

- `1-1`에서 `rs.next()`를 호출
- `1-2`의 결과로 `cursor`가 다음으로 이동. 이 경우 `cursor`가 가리키는 데이터가 있으므로 `true`를 반환
- `2-1`에서 `rs.next()`를 호출
- `2-2`의 결과로 `cursor`가 다음으로 이동. 이 경우 `cursor`가 가리키는 데이터가 있으므로 `true`를 반환
- `3-1`에서 `rs.next()`를 호출
- `3-2`의 결과로 `cursor`가 다음으로 이동. 이 경우 `cursor`가 가리키는 데이터가 없으므로 `false`를 반환

`findById()`에서는 회원 하나를 조회하는 것이 목적입니다. 

따라서 조회 결과가 항상 1건이므로 `while` 대신에 `if`를 사용합니다. 

다음 SQL을 보면 PK인 `member_id`를 항상 지정하는 것을 알 수 있습니다.

```sql
select * from member where member_id = ?
```

`MemberRepositoryV0Test` - 회원 조회 추가

```java
package hello.jdbc.repository;

import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

import java.sql.SQLException;

import static org.assertj.core.api.Assertions.assertThat;

@Slf4j
class MemberRepositoryV0Test {
    MemberRepositoryV0 repository = new MemberRepositoryV0();

    @Test
    void crud() throws SQLException {
        // save
        Member member = new Member("memberV0", 10000);
        repository.save(member);

        // findById
        Member findMember = repository.findById(member.getMemberId());
        log.info("findMember={}", findMember);
        assertThat(findMember).isEqualTo(member);
    }
}
```

**실행 결과**

```java
DBConnectionUtil - get connection=conn0: 
	url=jdbc:h2:tcp://localhost/~/test user=SA, 
	class=class org.h2.jdbc.JdbcConnection
DBConnectionUtil - get connection=conn1: 
	url=jdbc:h2:tcp://localhost/~/test user=SA, 
	class=class org.h2.jdbc.JdbcConnection
MemberRepositoryV0Test - findMember=Member(memberId=memberV0, money=10000)
```

- 회원을 등록하고 그 결과를 바로 조회해서 확인합니다.
- 참고로 실행 결과에 `member` 객체의 참조 값이 아니라 실제 데이터가 보이는 이유는 롬복의 `@Data`가 `toString()`을 적절히 오버라이딩 해서 보여주기 때문입니다.
- `isEqualTo()`: `findMember.equals(member)`를 비교합니다. 결과가 참인 이유는 롬복의 `@Data`는 해당 객체의 모든 필드를 사용하도록 `equals()`를 오버라이딩 하기 때문입니다

> **참고 -** 이 테스트는 2번 실행하면 PK 중복 오류가 발생하므로 `delete from member` 쿼리로 데이터를 삭제한 다음에 다시 실행해야 합니다.
>

