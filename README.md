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



# 7. JDBC 개발 - 수정, 삭제

`MemberRepositoryV0` - 회원 수정 추가

```java
private void update(String memberId, int money) throws SQLException {
    String sql = "update member set money=? where member_id=?";
    Connection con = null;
    PreparedStatement pstmt = null;

    try {
        con = getConnection();
        pstmt = con.prepareStatement(sql);
        pstmt.setInt(1, money);
        pstmt.setString(1, memberId);
        int resultSize = pstmt.executeUpdate();
        log.info("resultSize={}", resultSize);
    } catch (SQLException e) {
        log.error("db error", e);
        throw e;
    } finally {
        close(con, pstmt, null);
    }
}
```

`executeUpdate()`는 쿼리를 실행하고 영향받은 row 수를 반환합니다. 여기서는 하나의 데이터만 변경하기 때문에 결과로 1이 반환됩니다. 

만약 회원이 100명이고, 모든 회원의 데이터를 한번에 수정하는 update sql을 실행하면 결과는 100 입니다.

`MemberRepositoryV0Test` - 회원 수정 추가

```java
@Test
void crud() throws SQLException {
    //save
    Member member = new Member("memberV0", 10000);
    repository.save(member);
  
    //findById
    Member findMember = repository.findById(member.getMemberId());
    assertThat(findMember).isEqualTo(member);
  
    //update: money: 10000 -> 20000
    repository.update(member.getMemberId(), 20000);
    Member updatedMember = repository.findById(member.getMemberId());
    assertThat(updatedMember.getMoney()).isEqualTo(20000);
}
```

회원 데이터의 `money`를 10000 → 20000으로 수정하고, DB에서 데이터를 다시 조회해서 20000으로 변경 되었는지 검증합니다.

**실행 로그**

```java
DBConnectionUtil - get connection=conn0: url=...
DBConnectionUtil - get connection=conn1: url=...

MemberRepositoryV0Test - findMember=Member(memberId=memberV0, money=10000)
DBConnectionUtil - get connection=conn2: url=...

MemberRepositoryV0 - resultSize=1
DBConnectionUtil - get connection=conn3: url=...
```

`MemberRepositoryV0 - resultSize=1`

`pstmt.executeUpdate()`의 결과가 1인 것을 확인할 수 있습니다. 

이것은 해당 SQL에 영향을 받은 로우 수가 1개라는 의미입니다.

데이터베이스에서 조회하면 `memberV0` 의 `money` 가 `20000`으로 변경된 것을 확인할 수 있습니다.

> 참고 - 이 테스트는 2번 실행하면 PK 중복 오류가 발생. 이 경우 `delete from member` 쿼리로 데이터를 삭제한 다음에 다시 실행
> 

`MemberRepositoryV0` - 회원 삭제 추가

```java
public void delete(String memberId) throws SQLException {
    String sql = "delete from member where member_id=?";

    Connection con = null;
    PreparedStatement pstmt = null;

    try {
        con = getConnection();
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, memberId);
        pstmt.executeUpdate();
    } catch (SQLException e) {
        log.error("db error", e);
        throw e;
    } finally {
        close(con, pstmt, null);
    }
}
```

`MemberRepositoryV0Test` - 회원 삭제 추가

```java
package hello.jdbc.repository;

import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

import java.sql.SQLException;
import java.util.NoSuchElementException;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

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

        // update(money: 10000 -> 20000)
        repository.update(member.getMemberId(), 20000);
        Member updateMember = repository.findById(member.getMemberId());
        assertThat(updateMember.getMoney()).isEqualTo(20000);

        // delete
        repository.delete(member.getMemberId());
        assertThatThrownBy(() -> repository.findById(member.getMemberId())).isInstanceOf(NoSuchElementException.class);
    }
}
```

회원을 삭제한 다음 `findById()` 를 통해서 조회합니다. 

회원이 없기 때문에 `NoSuchElementException`이 발생합니다. `assertThatThrownBy`는 해당 예외가 발생해야 검증에 성공합니다.

> 참고 - 마지막에 회원을 삭제하기 때문에 테스트가 정상 수행되면, 이제부터는 같은 테스트를 반복해서 실행할 수 있다.
> 

 물론 테스트 중간에 오류가 발생해서 삭제 로직을 수행할 수 없다면 테스트를 반복해서 실행할 수 없습니다.

**트랜잭션을** 활용하면 이 문제를 깔끔하게 해결할 수 있습니다.


# ===== 2. 커넥션 풀과 데이터소스 이해 ======
# 1. 커넥션 풀 이해

### 데이터베이스 커넥션을 매번 획득

![https://user-images.githubusercontent.com/52024566/189142260-b62c32ce-bca4-44cf-b59d-53c0ab18f700.png](https://user-images.githubusercontent.com/52024566/189142260-b62c32ce-bca4-44cf-b59d-53c0ab18f700.png)

데이터베이스 커넥션을 획득할 때는 아래와 같은 복잡한 과정을 거칩니다.

1. 애플리케이션 로직은 DB 드라이버를 통해 커넥션을 조회함.
2. DB 드라이버는 DB와 TCP/IP 커넥션을 연결함. 당연히 이 과정에서 3 way handshake 같은 TCP/IP 연결을 위한 네트워크 동작이 발생함.
3. DB 드라이버는 TCP/IP 커넥션이 연결되면 ID, PW와 기타 부가정보를 DB에 전달함.
4. DB는 ID, PW를 통해 내부 인증을 완료하고, 내부에 DB 세션을 생성함.
5. DB는 커넥션 생성이 완료되었다는 응답을 보냄.
6. DB 드라이버는 커넥션 객체를 생성해서 클라이언트에 반환함.

이렇게 커넥션을 새로 만드는 것은 과정도 복잡하고 시간도 많이 소모됩니다.

DB는 물론이고 애플리케이션 서버에서도 TCP/IP 커넥션을 새로 생성하기 위한 리소스를 매번 사용해야 합니다.

가장 큰 문제는 고객이 애플리케이션을 사용할 때, SQL을 실행하는 시간 뿐만 아니라 커넥션을 새로 만드는 시간이 추가되기 때문에 결과적으로 응답 속도에 영향을 줍니다. 

**이것은 나쁜 사용자 경험을 줄 수 있습니다.**

> 참고 - 데이터베이스마다 커넥션을 생성하는 시간은 다르다. 
시스템 상황마다 다르지만 MySQL 계열은 수 ms(밀리초) 정도로 매우 빨리 커넥션을 확보할 수 있다. 반면에 수십 밀리초 이상 걸리는 데이터베이스들도 있음
> 

이런 문제를 한번에 해결하는 아이디어가 바로 **커넥션을 미리 생성해두고 사용하는 커넥션 풀**이라는 방법입니다!!!

### 커넥션 풀 초기화

![https://user-images.githubusercontent.com/52024566/189142269-d286e671-06dc-45c9-929e-ff038f154494.png](https://user-images.githubusercontent.com/52024566/189142269-d286e671-06dc-45c9-929e-ff038f154494.png)

애플리케이션을 시작하는 시점에 커넥션 풀은 필요한 만큼 커넥션을 미리 확보해서 풀에 보관합니다. 

보통 얼마나 보관할 지는 서비스의 특징과 서버 스펙에 따라 다르지만 기본값은 보통 10개입니다.

### **커넥션 풀의 연결 상태**

![https://user-images.githubusercontent.com/52024566/189142272-2984801b-11f6-4d9c-aeed-fb089f976ee5.png](https://user-images.githubusercontent.com/52024566/189142272-2984801b-11f6-4d9c-aeed-fb089f976ee5.png)

커넥션 풀에 들어 있는 커넥션은 TCP/IP로 DB와 커넥션이 연결되어 있는 상태이기 때문에 언제든지 즉시 SQL을 DB에 전달할 수 있습니다.

### **커넥션 풀 사용1**

![https://user-images.githubusercontent.com/52024566/189142275-088c1b26-254f-45ad-981a-1822519625d1.png](https://user-images.githubusercontent.com/52024566/189142275-088c1b26-254f-45ad-981a-1822519625d1.png)

- 애플리케이션 로직에서 더 이상 DB 드라이버를 통해서 새로운 커넥션을 획득하지 않습니다.
- 이제는 커넥션 풀을 통해 이미 생성되어 있는 커넥션을 객체 참조로 그냥 가져다 쓰기만 하면 됩니다.
- 커넥션 풀에 커넥션을 요청하면 커넥션 풀은 자신이 가지고 있는 커넥션 중에 하나를 반환합니다.

### **커넥션 풀 사용2**

![https://user-images.githubusercontent.com/52024566/189142279-cbfeeee6-20a7-4b10-bef9-2d61b677e11c.png](https://user-images.githubusercontent.com/52024566/189142279-cbfeeee6-20a7-4b10-bef9-2d61b677e11c.png)

- 애플리케이션 로직은 커넥션 풀에서 받은 커넥션을 사용해서 SQL을 데이터베이스에 전달하고 그 결과를 받아서 처리합니다.
- 커넥션을 모두 사용하고 나면 이제는 커넥션을 종료하는 것이 아니라, 다음에 다시 사용할 수 있도록 해당 커넥션을 그대로 커넥션 풀에 반환합니다.
    - 여기서 주의할 점은 커넥션을 종료하는 것이 아니라 커넥션이 살아있는 상태로 커넥션 풀에 반환해야 한다는 것입니다!!

### 정리

- 적절한 커넥션 풀 숫자는 서비스의 특징과 애플리케이션 서버 스펙, DB 서버 스펙에 따라 다르기 때문에 성능 테스트를 통해서 정해야 합니다.
- 커넥션 풀은 서버당 최대 커넥션 수를 제한할 수 있습니다. 
따라서 DB에 무한정 연결이 생성되는 것을 막아주어서 DB를 보호하는 효과도 있습니다.
- 이런 커넥션 풀은 얻는 이점이 매우 크기 때문에 실무에서는 항상 기본으로 사용합니다.
- 커넥션 풀은 개념적으로 단순해서 직접 구현할 수도 있지만, 사용도 편리하고 성능도 뛰어난 오픈소스 커넥션 풀이 많기 때문에 오픈소스를 사용하는 것이 좋습니다.
- 대표적인 커넥션 풀 오픈소스는 `commons-dbcp2`, `tomcat-jdbc pool`, `HikariCP` 등이 있습니다.
- 성능과 사용의 편리성 측면에서 최근에는 `hikariCP`를 주로 사용합니다. 스프링 부트 2.0 부터는 기본 커넥션 풀로 `hikariCP`를 제공합니다.
    - 성능, 사용의 편리함, 안전성 측면에서 이미 검증이 되었기 때문에 커넥션 풀을 사용할 때는 고민할 것 없이 `hikariCP`를 사용하면 됩니다. 
    실무에서도 레거시 프로젝트가 아닌 이상 대부분 `hikariCP`를 사용합니다.


# 2. DataSource 이해

커넥션을 얻는 방법은 앞서 학습한 JDBC `DriverManager` 를 직접 사용하거나, 커넥션 풀을 사용하는 등 다양한 방법이 존재합니다.

### 커넥션을 획득하는 다양한 방법

![https://user-images.githubusercontent.com/52024566/189354727-ddaeb839-5291-4005-a3b9-e6f3c156f7ac.png](https://user-images.githubusercontent.com/52024566/189354727-ddaeb839-5291-4005-a3b9-e6f3c156f7ac.png)

`DriverManager`를 통해 커넥션 획득하는 방법

![https://user-images.githubusercontent.com/52024566/189354734-d281fa65-6263-4719-abab-92a3892778e2.png](https://user-images.githubusercontent.com/52024566/189354734-d281fa65-6263-4719-abab-92a3892778e2.png)

`DriverManager`를 통해 커넥션을 획득하다가 커넥션 풀로 변경시 문제점

![https://user-images.githubusercontent.com/52024566/189354736-7c8cb850-56b4-4d70-a564-d01cd156e8b6.png](https://user-images.githubusercontent.com/52024566/189354736-7c8cb850-56b4-4d70-a564-d01cd156e8b6.png)

애플리케이션 로직에서 `DriverManager`를 사용해서 커넥션을 획득하다가 `HikariCP` 같은 커넥션 풀을 사용하도록 변경하면 커넥션을 획득하는 애플리케이션 코드도 함께 변경해야 합니다. 

의존관계가 `DriverManager`에서 `HikariCP`로 변경되기 때문에 문제가 발생합니다. 

물론 둘의 사용법도 조금씩 다른 문제도 있죠.

커넥션을 획득하는 방법을 추상화

![https://user-images.githubusercontent.com/52024566/189354740-5ea79b18-45cf-49b6-a115-6a15fa255f0b.png](https://user-images.githubusercontent.com/52024566/189354740-5ea79b18-45cf-49b6-a115-6a15fa255f0b.png)

자바에서는 이런 문제를 해결하기 위해 `javax.sql.DataSource`라는 인터페이스를 제공합니다

`DataSource`는 **커넥션을 획득하는 방법을 추상화**하는 인터페이스입니다.

이 인터페이스의 핵심 기능은 단 하나, 커넥션 조회입니다. (다른 일부 기능도 있지만 크게 중요하지 않습니다.)

### DataSource 핵심 기능만 축약

`DataSource`

```java
public interface DataSource {
    Connection getConnection() throws SQLException;
}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/39335582-1d3e-4d5e-80f6-f98944cc45ed/Untitled.png)

### 정리

대부분의 커넥션 풀은 `DataSource` 인터페이스를 이미 구현하고 있습니다. 

따라서 개발자는 `DBCP2 커넥션 풀`, `HikariCP 커넥션 풀`의 코드를 직접 의존하는 것이 아니라 `DataSource` 인터페이스에만 의존하도록 애플리케이션 로직을 작성하면 됩니다.

만약 커넥션 풀 구현 기술을 변경하고 싶으면 해당 구현체로 갈아끼우기만 하면 됩니다.

`DriverManager`는 `DataSource` 인터페이스를 사용하지 않습니다. 따라서 `DriverManager`는 직접 사용해야 합니다. 따라서 `DriverManager`를 사용하다가 `DataSource` 기반의 커넥션 풀을 사용하도록 변경하면 관련 코드를 다 고쳐야 합니다. 

이런 문제를 해결하기 위해 스프링은 `DriverManager`도 `DataSource`를 통해서 사용할 수 있도록 `DriverManagerDataSource`라는 `DataSource`를 구현한 클래스를 제공하고 있습니다.

자바는 `DataSource`를 통해 커넥션을 획득하는 방법을 추상화하고 있습니다. 이제 애플리케이션 로직은 `DataSource` 인터페이스에만 의존하면 됩니다. 

덕분에 `DriverManagerDataSource`를 통해서 `DriverManager`를 사용하다가 커넥션 풀을 사용하도록 코드를 변경해도 애플리케이션 로직은 변경하지 않아도 됩니다!!!


# 3. DataSource 예제 1. DriverManager

`ConnectionTest` - 드라이버 매니저

```java
package hello.jdbc.connection;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

import static hello.jdbc.connection.ConnectionConst.*;

@Slf4j
public class ConnectionTest {
    @Test
    void driverManager() throws SQLException {
        Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        log.info("connection={}, class={}", con1, con1.getClass());
        log.info("connection={}, class={}", con2, con2.getClass());
    }
}
```

실행 결과

```java
connection=conn0: 
	url=jdbc:h2:tcp://localhost/~/test user=SA, 
	class=class org.h2.jdbc.JdbcConnection
connection=conn1: 
	url=jdbc:h2:tcp://localhost/~/test user=SA, 
	class=class org.h2.jdbc.JdbcConnection
```

이번에는 스프링이 제공하는 `DataSource`가 적용된 `DriverManager`인 `DriverManagerDataSource`를 사용해봅시다.

`ConnectionTest` - 데이터소스 드라이버 매니저 추가

```java
package hello.jdbc.connection;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

import static hello.jdbc.connection.ConnectionConst.*;

@Slf4j
public class ConnectionTest {
    @Test
    void driverManager() throws SQLException {
        Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        log.info("connection={}, class={}", con1, con1.getClass());
        log.info("connection={}, class={}", con2, con2.getClass());
    }
    @Test
    void dataSourceDriverManager() throws SQLException {
        // DriverManagerDataSource - 항상 새로운 커넥션 휙득
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        useDataSource(dataSource);
    }

    private void useDataSource(DataSource dataSource) throws SQLException {
        Connection con1 = dataSource.getConnection();
        Connection con2 = dataSource.getConnection();
        log.info("connection={}, class={}", con1, con1.getClass());
        log.info("connection={}, class={}", con2, con2.getClass());
    }
}
```

`dataSourceDriverManager()` - 실행 결과

```java
DriverManagerDataSource - Creating new JDBC DriverManager Connection to [...]
DriverManagerDataSource - Creating new JDBC DriverManager Connection to [...]
connection=conn0: 
	url=jdbc:h2:tcp://localhost/~/test user=SA, 
	class=class org.h2.jdbc.JdbcConnection
connection=conn1: 
	url=jdbc:h2:tcp://localhost/~/test user=SA, 
	class=class org.h2.jdbc.JdbcConnection
```

기존 코드와 비슷하지만 `DriverManagerDataSource`는 `DataSource`를 통해서 커넥션을 획득할 수 있습니다. 참고로 `DriverManagerDataSource`는 스프링이 제공하는 코드입니다.

### **파라미터 차이**

기존 `DriverManager`를 통해서 커넥션을 획득하는 방법과 `DataSource`를 통해서 커넥션을 획득하는 방법에는 큰 차이가 있습니다! 코드를 되짚어 보면서 확인해봅시다.

`DriverManager`

```java
DriverManager.getConnection(URL, USERNAME, PASSWORD)
DriverManager.getConnection(URL, USERNAME, PASSWORD)
```

`DataSource`

```java
void dataSourceDriverManager() throws SQLException {
    DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
    useDataSource(dataSource);
}

private void useDataSource(DataSource dataSource) throws SQLException {
    Connection con1 = dataSource.getConnection();
    Connection con2 = dataSource.getConnection();
    log.info("connection={}, class={}", con1, con1.getClass());
    log.info("connection={}, class={}", con2, con2.getClass());
}
```

`DriverManager`는 커넥션을 획득할 때마다 `URL`, `USERNAME`, `PASSWORD` 같은 파라미터를 계속 전달해야 합니다. 

반면에 `DataSource`를 사용하는 방식은 처음 객체를 생성할 때만 필요한 파리미터를 넘겨두고, 커넥션을 획득할 때는 단순히 `dataSource.getConnection()`만 호출하면 됩니다.

**설정과 사용의 분리**

**설정**: `DataSource`를 만들고 필요한 속성들을 사용해서 `URL`, `USERNAME`, `PASSWORD` 같은 부분을 입력하는 것을 의미합니다. 이렇게 설정과 관련된 속성들은 한 곳에 있는 것이 향후 변경에 더 유연하게 대처할 수 있습니다.

**사용**: 설정은 신경쓰지 않고 `DataSource`의 `getConnection()`만 호출해서 사용하면 됩니다.

이 부분이 작아보이지만 큰 차이를 만들어내는데, 필요한 데이터를 `DataSource`가 만들어지는 시점에 미리 다 넣어두게 되면, `DataSource`를 사용하는 곳에서는 `dataSource.getConnection()`만 호출하면 되므로, `URL`, `USERNAME`, `PASSWORD` 같은 속성들에 의존하지 않아도 됩니다. 

그냥 `DataSource`만 주입받아서 `getConnection()`만 호출하면 되는 것이지요

쉽게 이야기해서 리포지토리(Repository)는 `DataSource`만 의존하고, 이런 속성을 몰라도 됩니다.

애플리케이션을 개발해보면 보통 설정은 한 곳에서 하지만, 사용은 수많은 곳에서 하게 됩니다.

덕분에 객체를 설정하는 부분과 사용하는 부분을 좀 더 명확하게 분리할 수 있습니다.

# 4. DataSource 예제 2. 커넥션 풀

`ConnectionTest` - 데이터소스 커넥션 풀 추가

```java
import com.zaxxer.hikari.HikariDataSource;

@Test
void dataSourceConnectionPool() throws SQLException, InterruptedException {
    //커넥션 풀링: HikariProxyConnection(Proxy) -> JdbcConnection(Target)
    HikariDataSource dataSource = new HikariDataSource();
    dataSource.setJdbcUrl(URL);
    dataSource.setUsername(USERNAME);
    dataSource.setPassword(PASSWORD);
    dataSource.setMaximumPoolSize(10);
    dataSource.setPoolName("MyPool");
  
    useDataSource(dataSource);
    Thread.sleep(1000); //커넥션 풀에서 커넥션 생성 시간 대기
}
```

HikariCP 커넥션 풀을 사용합니다. `HikariDataSource`는 `DataSource` 인터페이스를 구현하였습니다.

커넥션 풀 최대 사이즈를 10으로 지정하고, 풀의 이름을 `MyPool` 이라고 지정했습니다.

커넥션 풀에서 커넥션을 생성하는 작업은 애플리케이션 실행 속도에 영향을 주지 않기 위해 별도의 쓰레드에서 작동합니다. 

별도의 쓰레드에서 동작하기 때문에 테스트가 먼저 종료되어 버리므로 예제처럼 `Thread.sleep`을 통해 대기 시간을 주어야 쓰레드 풀에 커넥션이 생성되는 로그를 확인할 수 있습니다.

**실행 결과** (로그 순서는 이해하기 쉽게 약간 수정)

```java
#커넥션 풀 초기화 정보 출력
HikariConfig - MyPool - configuration:
...
HikariConfig - maximumPoolSize................................10
HikariConfig - poolName................................"MyPool"
...

#커넥션 풀 전용 쓰레드가 커넥션 풀에 커넥션을 10개 채움
[MyPool connection adder] MyPool - Added connection conn0: url=...
	user=SA
[MyPool connection adder] MyPool - Added connection conn1: url=...
	user=SA
[MyPool connection adder] MyPool - Added connection conn2: url=...
	user=SA
[MyPool connection adder] MyPool - Added connection conn3: url=...
	user=SA
		...
[MyPool connection adder] MyPool - Added connection conn9: url=...
	user=SA

#커넥션 풀에서 커넥션 획득1
ConnectionTest - connection=HikariProxyConnection@446445803 wrapping conn0:
	url=... user=SA, 
	class=class com.zaxxer.hikari.pool.HikariProxyConnection
#커넥션 풀에서 커넥션 획득2
ConnectionTest - connection=HikariProxyConnection@832292933 wrapping conn1:
	url=... user=SA, 
	class=class com.zaxxer.hikari.pool.HikariProxyConnection

MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)
```

### **실행 결과 분석**

**HikariConfig** : HikariCP 관련 설정을 확인할 수 있습니다. 풀의 이름(`MyPool`)과 최대 풀 수(`10`)을 확인할 수 있습니다.

**MyPool connection adder** : 별도의 쓰레드 사용해서 커넥션 풀에 커넥션을 채우고 있는 것을 확인할 수 있습니다.

이 쓰레드는 커넥션 풀에 커넥션을 최대 풀 수(`10`)까지 채웁니다. 

> 별도의 쓰레드를 사용해서 커넥션 풀에 커넥션을 채우는 이유는?
> 

커넥션 풀에 커넥션을 채우는 것은 상대적으로 오래 걸리는 일입니다. 애플리케이션을 실행할 때 커넥션 풀을 채울 때 까지 마냥 대기하고 있다면 애플리케이션 실행 시간이 늦어지겠죠. 따라서 이렇게 별도의 쓰레드를 사용해서 커넥션 풀을 채워야 애플리케이션 실행 시간에 영향을 주지 않습니다.

**커넥션 풀에서 커넥션 획득 :** 커넥션 풀에서 커넥션을 획득하고 그 결과를 출력합니다. 

여기서는 커넥션 풀에서 커넥션을 2개를 획득하고 반환하지는 않습니다. 

따라서 풀에 있는 10개의 커넥션 중에 2개를 가지고 있는 상태입니다. 

그래서 마지막 로그를 보면 사용중인 커넥션 `active=2`, 풀에서 대기 상태인 커넥션 `idle=8`을 확인할 수 있습니다. 

```java
MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)
```

> 참고 HikariCP 커넥션 풀에 대한 더 자세한 내용은 다음 공식 사이트를 참고합시다.
https://github.com/brettwooldridge/HikariCP
>

# 5. DataSource 적용

`MemberRepositoryV1`

```java
package hello.jdbc.repository;

import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.support.JdbcUtils;

import javax.sql.DataSource;
import java.sql.*;
import java.util.NoSuchElementException;

/**
 * JDBC - DataSource 사용, JdbcUtils 사용
 */
@Slf4j
public class MemberRepositoryV1 {
    private final DataSource dataSource;

    public MemberRepositoryV1(DataSource dataSource) {
        this.dataSource = dataSource;
    }

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

    public void update(String memberId, int money) throws SQLException {
        String sql = "update member set money=? where member_id=?";
        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            int resultSize = pstmt.executeUpdate();
            log.info("resultSize={}", resultSize);
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    public void delete(String memberId) throws SQLException {
        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        JdbcUtils.closeConnection(con);
    }

    private Connection getConnection() throws SQLException {
        Connection con = dataSource.getConnection();
        log.info("get connection={}, class={}", con, con.getClass());
        return con;
    }
}
```

`save`, `findById`, `update`, `delete` 메서드는 이전에 `MemberRepositoryV0` 에서의 메서드와 같습니다.

`DataSource` 의존관계 주입

외부에서 `DataSource`를 주입 받아서 사용합니다.. 이제 직접 만든 `DBConnectionUtil`을 사용하지 않아도 됩니다.

`DataSource`는 표준 인터페이스 이기 때문에 `DriverManagerDataSource`에서 `HikariDataSource`로 변경되어도 해당 코드를 변경하지 않아도 됩니다. 그래서 우리는 4가지 메서드를 변경하지 않은 것입니다.

`JdbcUtils` 편의 메서드

스프링은 JDBC를 편리하게 다룰 수 있는 `JdbcUtils`라는 편의 메서드를 제공합니다.

`JdbcUtils`을 사용하면 커넥션을 좀 더 편리하게 닫을 수 있습니다.

`MemberRepositoryV1Test`

```java
package hello.jdbc.repository;

import com.zaxxer.hikari.HikariDataSource;
import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.sql.SQLException;
import java.util.NoSuchElementException;

import static hello.jdbc.connection.ConnectionConst.*;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.AssertionsForClassTypes.assertThatThrownBy;

@Slf4j
public class MemberRepositoryV1Test {
    MemberRepositoryV1 repository;

    @BeforeEach
    void beforeEach() throws Exception {
        // 기본 DriverManager - 항상 새로운 커넥션 휙득
        // DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        
        // 커넥션 풀링: HikariProxyConnection -> JdbcConnection
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(URL);
        dataSource.setUsername(USERNAME);
        dataSource.setPassword(PASSWORD);

        repository = new MemberRepositoryV1(dataSource);
    }

    @Test
    void crud() throws SQLException, InterruptedException {
        log.info("start");

        //save
        Member member = new Member("memberV0", 10000);
        repository.save(member);

        //findById
        Member memberById = repository.findById(member.getMemberId());
        assertThat(memberById).isNotNull();

        //update: money: 10000 -> 20000
        repository.update(member.getMemberId(), 20000);
        Member updatedMember = repository.findById(member.getMemberId());
        assertThat(updatedMember.getMoney()).isEqualTo(20000);

        //delete
        repository.delete(member.getMemberId());
        assertThatThrownBy(() -> repository.findById(member.getMemberId()))
                .isInstanceOf(NoSuchElementException.class);

    }

}
```

`MemberRepositoryV1`은 `DataSource` 의존관계 주입이 필요합니다.

### DriverManagerDataSource 사용

```java
get connection=conn0: url=jdbc:h2:.. user=SA class=class org.h2.jdbc.JdbcConnection
get connection=conn1: url=jdbc:h2:.. user=SA class=class org.h2.jdbc.JdbcConnection
get connection=conn2: url=jdbc:h2:.. user=SA class=class org.h2.jdbc.JdbcConnection
get connection=conn3: url=jdbc:h2:.. user=SA class=class org.h2.jdbc.JdbcConnection
get connection=conn4: url=jdbc:h2:.. user=SA class=class org.h2.jdbc.JdbcConnection
get connection=conn5: url=jdbc:h2:.. user=SA class=class org.h2.jdbc.JdbcConnection
...
```

`DriverManagerDataSource`를 사용하면 `conn0~5` 번호를 통해서 항상 새로운 커넥션이 생성되어서 사용되는 것을 확인할 수 있습니다. 

`DriverManagerDataSource` 을 사용하려면 아래처럼 코드를 주석 처리해야 함.

```java
// 기본 DriverManager - 항상 새로운 커넥션 휙득
DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);

// 커넥션 풀링: HikariProxyConnection -> JdbcConnection
// HikariDataSource dataSource = new HikariDataSource();
// dataSource.setJdbcUrl(URL);
// dataSource.setUsername(USERNAME);
// dataSource.setPassword(PASSWORD);
```

### **HikariDataSource 사용**

```java
get connection=HikariProxyConnection@xxxxxxxx1 wrapping conn0: url=jdbc:h2:...
user=SA
get connection=HikariProxyConnection@xxxxxxxx2 wrapping conn0: url=jdbc:h2:...
user=SA
get connection=HikariProxyConnection@xxxxxxxx3 wrapping conn0: url=jdbc:h2:...
user=SA
get connection=HikariProxyConnection@xxxxxxxx4 wrapping conn0: url=jdbc:h2:...
user=SA
get connection=HikariProxyConnection@xxxxxxxx5 wrapping conn0: url=jdbc:h2:...
user=SA
get connection=HikariProxyConnection@xxxxxxxx6 wrapping conn0: url=jdbc:h2:...
user=SA
```

커넥션 풀 사용시 `conn0` 커넥션이 재사용 된 것을 확인할 수 있습니다.

테스트는 순서대로 실행되기 때문에 커넥션을 사용하고 다시 돌려주는 것을 반복하므로 `conn0`만 사용됩니다.

웹 애플리케이션에 동시에 여러 요청이 들어오면 여러 쓰레드에서 커넥션 풀의 커넥션을 다양하게 가져가는 상황을 확인할 수 있습니다.

커넥션 풀 을 사용하려면 아래처럼 코드를 주석 처리해야 함.

```java
// 기본 DriverManager - 항상 새로운 커넥션 휙득
// DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);

// 커넥션 풀링: HikariProxyConnection -> JdbcConnection
HikariDataSource dataSource = new HikariDataSource();
dataSource.setJdbcUrl(URL);
dataSource.setUsername(USERNAME);
dataSource.setPassword(PASSWORD);
```

### DI

`DriverManagerDataSource` → `HikariDataSource`로 변경해도 `MemberRepositoryV1`의 코드는 전혀 변경하지 않아도 됩니다. 

`MemberRepositoryV1`는 `DataSource` 인터페이스에만 의존하기 때문입니다. 

이것이 `DataSource`를 사용하는 장점입니다!!! (**DI + OCP**)


# ==== 3. 트랜잭션 이해 ====
# 1. 트랜잭션 - 개념 이해

> 데이터를 저장할 때 단순히 파일에 저장해도 되는데, 데이터베이스에 저장하는 이유는 무엇일까요?
> 

여러가지 이유가 있지만, 가장 대표적인 이유는 바로 데이터베이스는 트랜잭션이라는 개념을 지원하기 때문입니다.

트랜잭션을 이름 그대로 번역하면 거래라는 뜻으로, 이것을 쉽게 풀어서 이야기하면, 데이터베이스에서 트랜잭션은 **하나의 거래를 안전하게 처리하도록 보장**해주는 것을 의미합니다. 

그런데 하나의 거래를 안전하게 처리하려면 생각보다 고려해야 할 점이 많습니다. 

예를 들어서 A 의 5000원을 B 에게 계좌이체한다고 가정하면 A 의 잔고를 5000원 감소시키고, B 의 잔고를 5000원 증가시켜야 합니다.

**5000원 계좌이체**

1. A의 잔고를 5000원 감소
2. B의 잔고를 5000원 증가

계좌이체라는 거래는 이렇게 2가지 작업이 합쳐져서 하나의 작업처럼 동작해야 합니다. 

만약 1번은 성공했는데 2번에서 시스템에 문제가 발생하면 계좌이체는 실패하고, A의 잔고만 5000원 감소하는 심각한 문제가 발생하게 됩니다.

데이터베이스가 제공하는 트랜잭션 기능을 사용하면 1,2 둘다 함께 성공해야 저장하고, 중간에 하나라도 실패하면 거래 전의 상태로 돌아갈 수 있습니다. 

만약 1번은 성공했는데 2번에서 시스템에 문제가 발생하면 계좌이체는 실패하고, 거래 전의 상태로 완전히 돌아갈 수 있습니다. 결과적으로 A의 잔고가 감소하지 않겠죠. 

모든 작업이 성공해서 데이터베이스에 정상 반영하는 것을 **커밋(Commit)**이라 하고, 작업 중 하나라도 실패해서 거래 이전으로 되돌리는 것을 **롤백(Rollback)**이라 합니다.

### **트랜잭션 ACID**

트랜잭션은 **ACID**(http://en.wikipedia.org/wiki/ACID)라 하는 **원자성(Atomicity)**, **일관성(Consistency)**, **격리성(Isolation)**, **지속성(Durability)**을 보장해야 합니다.

- 원자성(Atomicity): 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공 하거나 모두 실패해야 합니다.
- 일관성(Consistency): 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 합니다.
예를 들어 데이터베이스에서 정한 무결성 제약 조건을 항상 만족해야 합니다.
- 격리성(Isolation): 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리해야 합니다. 
예를 들어 동시에 같은 데이터를 수정하지 못하도록 해야 합니다. 
격리성은 동시성과 관련된 성능 이슈로 인해 트랜잭션 격리 수준(Isolation level)을 선택할 수 있습니다.
- 지속성(Durability): 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 합니다. 
중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 합니다.

트랜잭션은 원자성, 일관성, 지속성을 보장합니다. 

여기서 복잡한 문제는 격리성인데 트랜잭션 간에 격리성을 완벽히 보장하려면 트랜잭션을 거의 순서대로 실행해야 하므로 동시 처리 성능이 매우 나빠질 수 밖에 없습니다. 

이런 문제로 인해 ANSI 표준은 트랜잭션의 격리 수준(Isolation level)을 4단계로 나누어 정의합니다.

**트랜잭션 격리 수준 - Isolation level**

- READ UNCOMMITED(커밋되지 않은 읽기) - Dirty Read
- READ COMMITTED(커밋된 읽기) - 가장 많이 씀
- REPEATABLE READ(반복 가능한 읽기)
- SERIALIZABLE(직렬화 가능) - 거의 사용하지 않음

이 글에서는 일반적으로 많이 사용하는 READ COMMITTED(커밋된 읽기) 트랜잭션 격리 수준을 기준으로 설명합니다. 트랜잭션 격리 수준은 데이터베이스 자체에 관한 부분이므로 다른 글을 참고하세요! (앞으로 데이터베이스 관련 내용을 정리할 예정이나 조금 시간이 걸릴 것 같습니다.)

# 2. 데이터베이스 연결 구조와 DB 세션

### 데이터베이스 연결 구조1

![https://user-images.githubusercontent.com/52024566/190993959-2cf26cde-1241-4fe0-b837-747da0ac9de5.png](https://user-images.githubusercontent.com/52024566/190993959-2cf26cde-1241-4fe0-b837-747da0ac9de5.png)

1. 사용자는 웹 애플리케이션 서버(WAS)나 DB 접근 툴 같은 클라이언트를 사용해서 데이터베이스 서버에 접근할 수 있습니다. 
2. 클라이언트는 데이터베이스 서버에 연결을 요청하고 커넥션을 맺습니다. 
3. 이 때 데이터베이스 서버는 내부에 세션이라는 것을 생성합니다. 
4. 그리고 앞으로 해당 커넥션을 통한 모든 요청은 이 세션을 통해서 실행합니다.
쉽게 이야기해서 개발자가 클라이언트를 통해 SQL을 전달하면 현재 커넥션에 연결된 세션이 SQL을 실행하는 것이지요.
5. 세션은 트랜잭션을 시작하고, 커밋 또는 롤백을 통해 트랜잭션을 종료합니다. 그리고 이후에 새로운 트랜잭션을 다시 시작할 수 있음
6. 사용자가 커넥션을 닫거나, 또는 DBA(DB 관리자)가 세션을 강제로 종료하면 세션은 종료됩니다.

### 데이터베이스 연결 구조2

![https://user-images.githubusercontent.com/52024566/190993965-827d69c3-0189-4417-b6d4-d21eeb71b07a.png](https://user-images.githubusercontent.com/52024566/190993965-827d69c3-0189-4417-b6d4-d21eeb71b07a.png)

커넥션 풀이 10개의 커넥션을 생성하면, 세션도 10개 생성됩니다.

# 3. 트랜잭션 - DB 예제1 - 개념 이해

### 개념 이해

본 설명은 트랜잭션 개념의 이해를 돕기 위해 예시로 구체적인 실제 구현 방식은 데이터베이스마다 조금씩은 다릅니다.

**트랜잭션 사용법**

데이터 변경 쿼리를 실행하고 데이터베이스에 그 결과를 반영하려면 커밋 명령어인 `commit`을 호출하고, 결과를 반영하고 싶지 않으면 롤백 명령어인 `rollback`을 호출하면 됩니다.

커밋을 호출하기 전까지는 임시로 데이터를 저장합니다. 따라서 해당 트랜잭션을 시작한 세션(사용자)에게만 변경 데이터가 보이고 다른 세션(사용자)에게는 변경 데이터가 보이지 않습니다.

**기본 데이터**

![https://user-images.githubusercontent.com/52024566/191275557-2ba6e934-e960-4f1d-9a11-85a1a1bd94bb.png](https://user-images.githubusercontent.com/52024566/191275557-2ba6e934-e960-4f1d-9a11-85a1a1bd94bb.png)

- 세션1, 세션2 둘다 가운데 있는 기본 테이블을 조회하면 해당 데이터가 그대로 조회됩니다.

**세션1 신규 데이터 추가**

![https://user-images.githubusercontent.com/52024566/191275571-305f53c6-3851-4cd8-b1a1-6cf83384379c.png](https://user-images.githubusercontent.com/52024566/191275571-305f53c6-3851-4cd8-b1a1-6cf83384379c.png)

세션1은 트랜잭션을 시작하고 신규 회원1, 신규 회원2를 DB에 추가한 상태입니다. 하지만 아직 커밋은 하지 않은 상태이지요.

새로운 데이터는 임시 상태로 저장됩니다.

세션1은 `select` 쿼리를 실행해서 본인이 입력한 신규 회원1, 신규 회원2를 조회할 수 있습니다.

하지만 세션1이 아직 커밋을 하지 않았기 때문에 세션2는 `select` 쿼리를 실행해도 신규 회원들을 조회할 수 없습니다.

> 만약 위와 다르게 커밋하지 않은 데이터를 다른 곳(세션2)에서 조회할 수 있을 때 발생하는 문제는 무엇일까요?
> 

예를 들어서 커밋하지 않는 데이터가 보인다면, 세션2는 데이터를 조회했을 때 신규 회원1, 2가 보이게 됩니다. 

따라서 신규 회원1, 신규 회원2가 있다고 가정하고 어떠한 로직을 수행할 수 있습니다. 그런데 세션1이 롤백을 수행하면 신규 회원1, 신규 회원2의 데이터가 사라지게 되므로 데이터 정합성에 큰 문제가 발생하게 됩니다.

즉, 세션2에서 세션1이 아직 커밋하지 않은 변경 데이터가 보인다면 세션1이 롤백 했을 때 심각한 문제가 발생할 수 있으므로 커밋 전의 데이터는 다른 세션에서 보이지 않아야 합니다.

**세션1 신규 데이터 추가 후 commit**

![https://user-images.githubusercontent.com/52024566/191275576-0ca2b795-8257-4606-bc4f-fa82e197e5a9.png](https://user-images.githubusercontent.com/52024566/191275576-0ca2b795-8257-4606-bc4f-fa82e197e5a9.png)

이제 세션1이 신규 데이터를 추가한 후에 `commit`을 호출했습니다.

`commit`으로 새로운 데이터가 실제 데이터베이스에 반영되어 데이터의 상태도 임시 완료로 변경됩니다.

이제 다른 세션에서도 회원 테이블을 조회하면 신규 회원들을 확인할 수 있습니다.

**세션1 신규 데이터 추가 후 rollback**

![https://user-images.githubusercontent.com/52024566/191275580-94127149-3b3b-4908-955a-e5d25be5b54e.png](https://user-images.githubusercontent.com/52024566/191275580-94127149-3b3b-4908-955a-e5d25be5b54e.png)

세션1이 신규 데이터를 추가한 후에 `commit` 이 아닌 `rollback`을 호출했습니다.

세션1이 데이터베이스에 반영한 모든 데이터가 처음 상태로 복구됩니다.

수정하거나 삭제한 데이터도 `rollback`을 호출하면 모두 트랜잭션을 시작하기 직전의 상태로 복구됩니다.

# 4. 트랜잭션 - DB 예제2 - 자동 커밋, 수동 커밋

`member` -  예제 스키마

```sql
drop table member if exists;
create table member (
member_id varchar(10),
money integer not null default 0,
primary key (member_id)
);
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e37cdd7a-0f4a-4e33-8cb7-adbef7c09bf3/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/643375ab-6593-40b7-a921-dc37428e294e/Untitled.png)

### 자동 커밋

자동 커밋으로 설정하면 각각의 쿼리 실행 직후에 자동으로 커밋을 호출합니다. 따라서 커밋이나 롤백을 직접 호출하지 않아도 되어 편리합니다. 

하지만 쿼리를 하나하나 실행할 때 마다 자동으로 커밋이 되어버리기 때문에 우리가 원하는 트랜잭션 기능을 제대로 사용할 수 없습니다.

자동 커밋 설정은 아래처럼 작성하면 됩니다.

```sql
set autocommit true; //자동 커밋 모드 설정

insert into member(member_id, money) values ('data1',10000); //자동 커밋
insert into member(member_id, money) values ('data2',10000); //자동 커밋

select * from member;
```

따라서 `commit`, `rollback`을 직접 호출하면서 트랜잭션 기능을 제대로 수행하려면 자동 커밋을 끄고 수동 커밋을 사용해야 합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df6d0a2e-ac1c-4b67-888f-c823039f7d36/Untitled.png)

### 수동 커밋

```sql
set autocommit false; //수동 커밋 모드 설정

insert into member(member_id, money) values ('data3',10000);
insert into member(member_id, money) values ('data4',10000);
commit; //수동 커밋
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3b772632-038a-4a83-b9b4-e950c3f999d5/Untitled.png)

보통 자동 커밋 모드가 기본으로 설정된 경우가 많기 때문에, 수동 커밋 모드로 설정하는 것을 트랜잭션을 시작한다고 표현할 수 있습니다. 

수동 커밋 설정을 하면 이후에 꼭 `commit`, `rollback`을 호출해야 합니다. 

참고로 수동 커밋 모드나 자동 커밋 모드는 한번 설정하면 해당 세션에서는 계속 유지됩니다. 

물론, 중간에 변경하는 것도 가능합니다.

# 5. 트랜잭션 - DB 예제3 - 트랜잭션 실습

### 1. 기본 데이터 입력

우리는 세션을 하나 더 열기 위해서 웹 콘솔 창을 하나 더 열어서 h2 데이터베이스로 실행할 것입니다.

> 주의 - H2 데이터베이스 웹 콘솔 창을 2개 열때 기존 URL을 복사하면 안 됩니다.
> 

꼭 http://localhost:8082 를 직접 입력해서 완전히 새로운 세션에서 연결하도록 해야 합니다. 
URL을 복사하면 같은 세션( jsessionId )에서 실행되어서 원하는 결과가 나오지 않을 수 있습니다.

예: http://localhost:8082 에 접근했을 때 다음과 같이 jsessionid 값이 서로 달라야 함. jsessionid 값이 같으면 같은 세션에 접근하는 것임.

예) 1번 URL: [http://localhost:8082/login.do?jsessionid=744cb5cbdfeab7d972e93d08d731b005](http://localhost:8082/login.do?jsessionid=546c1b90ed1f1268e57d427b3731664c)

예) 2번 URL: [http://localhost:8082/login.do?jsessionid=5e297b3dbeaa2383acc1109942bd2a41](http://localhost:8082/login.do?jsessionid=3c18778065d2bc882a74097aeeb219f4)

### 기본 데이터

![https://user-images.githubusercontent.com/52024566/191769670-74ed6eed-50bf-44c1-bbea-441ba7c7c7c5.png](https://user-images.githubusercontent.com/52024566/191769670-74ed6eed-50bf-44c1-bbea-441ba7c7c7c5.png)

**데이터 초기화 SQL**

```sql
//데이터 초기화
set autocommit true;
delete from member;
insert into member(member_id, money) values ('oldId',10000);
```

자동 커밋 모드를 사용했기 때문에 별도로 커밋을 호출하지 않아도 됩니다.

> 주의 - 만약 잘 진행되지 않으면 이전에 실행한 특정 세션에서 락을 걸고 있을 수 있습니다. 
이때는 H2 데이터베이스 서버를 종료하고 다시 실행하면 됩니다.
> 

데이터를 초기화하고 세션1, 세션2에서 다음 쿼리를 실행해서 결과를 확인해봅시다.

`select * from member;`

세션 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/de5b03b6-c0b1-4f77-a39b-fb4ea26b8396/Untitled.png)

세션 2

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3b45fb45-fb42-456e-a78b-1442eec27822/Untitled.png)

세션 1 과 세션 2 모두 동일하게 조회됩니다.

**세션1 신규 데이터 추가**

![https://user-images.githubusercontent.com/52024566/191769677-64e7c31a-be92-4419-9488-c5a67966a14e.png](https://user-images.githubusercontent.com/52024566/191769677-64e7c31a-be92-4419-9488-c5a67966a14e.png)

**세션1 신규 데이터 추가 SQL**

```sql
//트랜잭션 시작
set autocommit false; //수동 커밋 모드
insert into member(member_id, money) values ('newId1',10000);
insert into member(member_id, money) values ('newId2',10000);

select * from member;
```

세션1, 세션2에서 다음 쿼리를 실행해서 결과를 확인해봅니다.

세션 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df0f2d2b-fb65-4dfb-80fd-c88654b87519/Untitled.png)

세션 2

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fedd3a11-a1ed-4b17-82e6-20b42cde6a7e/Untitled.png)

아직 세션1이 커밋을 하지 않은 상태이기 때문에 세션1에서는 입력한 데이터가 보이지만, 세션2에서는 입력한 데이터가 보이지 않는 것을 확인할 수 있습니다.

### 커밋 - commit

세션1에서 신규 데이터를 입력했는데 아직 커밋은 하지 않았습니다. 이제 커밋해서 데이터베이스에 결과를 반영해봅시다.

**세션1 신규 데이터 추가 후 commit**

![https://user-images.githubusercontent.com/52024566/191769681-6bece174-a580-4ff9-9840-b3758ec996da.png](https://user-images.githubusercontent.com/52024566/191769681-6bece174-a580-4ff9-9840-b3758ec996da.png)

세션1에서 커밋을 호출합시다.

```sql
commit; //데이터베이스에 반영
```

세션1, 세션2에서 다음 쿼리를 실행해서 결과를 확인해봅시다.

```sql
select * from member;
```

결과를 이미지와 비교하면 같은 결과입니다. 

세션 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aa1e5e99-e55e-4e3c-a333-40524ab4de44/Untitled.png)

세션 2

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/960caafc-e8df-4c7a-a13c-bc2c9b4eda2c/Untitled.png)

세션1이 트랜잭션을 커밋했기 때문에 데이터베이스에 실제 데이터가 반영되었습니다. 커밋 이후에는 모든 세션에서 데이터를 조회할 수 있습니다.

### 롤백 - rollback

**기본 데이터**

![https://user-images.githubusercontent.com/52024566/191769685-b46c07cc-c256-4e75-b566-8bafe7c6e650.png](https://user-images.githubusercontent.com/52024566/191769685-b46c07cc-c256-4e75-b566-8bafe7c6e650.png)

예제를 처음으로 돌리기 위해 데이터를 초기화합시다.

```sql
//데이터 초기화
set autocommit true;
delete from member;
insert into member(member_id, money) values ('oldId',10000);
```

**세션1 신규 데이터 추가 후**

![https://user-images.githubusercontent.com/52024566/191769688-f5c0d064-59dd-4adc-b4ef-d843f957baab.png](https://user-images.githubusercontent.com/52024566/191769688-f5c0d064-59dd-4adc-b4ef-d843f957baab.png)

세션1에서 트랜잭션을 시작 상태로 만든 다음에 데이터를 추가합니다.

```sql
//트랜잭션 시작
set autocommit false; //수동 커밋 모드
insert into member(member_id, money) values ('newId1',10000);
insert into member(member_id, money) values ('newId2',10000);
```

세션1, 세션2에서 다음 쿼리를 실행해서 결과를 확인해봅시다.

```sql
select * from member;
```

세션 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0967dbec-b85d-456f-8721-4ef5c146616e/Untitled.png)

세션 2

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b3cacdd5-983c-42e0-87d1-0046ad11efec/Untitled.png)

아직 세션1이 커밋을 하지 않은 상태이기 때문에 세션1에서는 입력한 데이터가 보이지만, 세션2에서는 입력한 데이터가 보이지 않는 것을 확인할 수 있습니다.

**세션1 신규 데이터 추가 후 rollback**

![https://user-images.githubusercontent.com/52024566/191769692-f84d9980-3e91-4afd-ba85-8f306b2d0c74.png](https://user-images.githubusercontent.com/52024566/191769692-f84d9980-3e91-4afd-ba85-8f306b2d0c74.png)

세션1에서 롤백을 호출합니다.

```sql
rollback; //롤백으로 데이터베이스에 변경 사항을 반영하지 않는다.
```

세션1, 세션2에서 다음 쿼리를 실행해서 결과를 확인합니다.

```sql
select * from member;
```

세션 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f26f5732-c140-4b32-9fdc-20acf8ebc563/Untitled.png)

세션 2

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/681105a6-235d-4e5e-995d-c88420205dc7/Untitled.png)

롤백으로 데이터가 DB에 반영되지 않은 것을 확인할 수 있습니다.

# 6. 트랜잭션 - DB 예제4- 계좌 이체

### 계좌이체가 정상적으로 실행되는 모습

**기본 데이터 입력**

![https://user-images.githubusercontent.com/52024566/191990545-1ab03f65-5bd2-4e69-838c-546f35fa3464.png](https://user-images.githubusercontent.com/52024566/191990545-1ab03f65-5bd2-4e69-838c-546f35fa3464.png)

**기본 데이터 입력 - SQL**

```sql
set autocommit true;
delete from member;
insert into member(member_id, money) values ('memberA',10000);
insert into member(member_id, money) values ('memberB',10000);
```

`memberA` 는 10000원 을 가지고 있고 `memberB` 도 10000원을 갖고 있는 상태가 되었습니다.

**계좌이체 실행**

![https://user-images.githubusercontent.com/52024566/191990553-66541409-c575-49ce-aeb0-b40838f32a46.png](https://user-images.githubusercontent.com/52024566/191990553-66541409-c575-49ce-aeb0-b40838f32a46.png)

`memberA`의 돈을 `memberB`에게 2000원 계좌이체하는 트랜잭션을 실행합니다. 

다음과 같은 2번의 `update`쿼리가 수행되어야 합니다.

`set autocommit false`로 설정합니다.

아직 커밋하지 않았으므로 다른 세션에는 기존 데이터가 조회됩니다. 아래 코드를 통해서 실행됩니다.

**계좌이체 실행 SQL - 성공**

```sql
set autocommit false;
update member set money=10000 - 2000 where member_id = 'memberA';
update member set money=10000 + 2000 where member_id = 'memberB';
```

**커밋**

![https://user-images.githubusercontent.com/52024566/191990557-50c144c4-9b03-405d-9bed-bcf9427f150a.png](https://user-images.githubusercontent.com/52024566/191990557-50c144c4-9b03-405d-9bed-bcf9427f150a.png)

- `commit` 명령어를 실행하면 데이터베이스에 결과가 반영됨
- 다른 세션에서도 `memberA`의 금액이 8000원으로 줄어들고, `memberB`의 금액이 12000원으로 증가한 것을 확인할 수 있음

**세션 1 에서 커밋**

```sql
commit;
```

**확인 쿼리**

```sql
select * from member;
```

이렇게 되면 정상적으로 실행된 것을 확인할 수 있습니다. 하지만 문제 상황이 일어날 수도 있습니다. 아래 예시처럼 말이죠.

### 계좌이체 문제 상황 - 커밋

**기본 데이터 입력**

![https://user-images.githubusercontent.com/52024566/191990560-d27d76fd-05a6-46be-90ad-eaaf5b4089d1.png](https://user-images.githubusercontent.com/52024566/191990560-d27d76fd-05a6-46be-90ad-eaaf5b4089d1.png)

**기본 데이터 입력 - SQL**

```sql
set autocommit true;
delete from member;
insert into member(member_id, money) values ('memberA',10000);
insert into member(member_id, money) values ('memberB',10000);
```

**계좌이체 실행**

![https://user-images.githubusercontent.com/52024566/191990565-ffd59750-da9c-4802-8011-e85ee2190ca9.png](https://user-images.githubusercontent.com/52024566/191990565-ffd59750-da9c-4802-8011-e85ee2190ca9.png)

계좌이체를 실행하는 도중에 SQL에 어떠한 문제가 발생한다고 합시다. 그렇게 되면 `memberA`의 돈을 2000원 줄이는 것에는 성공했지만 `memberB`의 돈을 2000원 증가시키는 것에 실패할 수 있습니다.

아래 예시 코드를 보면 두 번째 SQL 에서 `member_iddd`라는 필드에 오타가 있습니다. 두 번째 `update` 쿼리를 실행하면 SQL 오류가 발생하는 것을 확인할 수 있습니다.

**계좌이체 실행 SQL - 오류**

```sql
set autocommit false;
update member set money=10000 - 2000 where member_id = 'memberA'; //성공
update member set money=10000 + 2000 where member_iddd = 'memberB'; //쿼리 예외 발생
```

**두 번째 SQL 실행시 발생하는 오류 메시지**

```sql
Column "MEMBER_IDDD" not found; SQL statement:
update member set money=10000 + 2000 where member_iddd = 'memberB' [42122-200] 42S22/42122
```

여기서 문제는 `memberA`의 돈은 2000원 줄어들었지만, `memberB`의 돈은 2000원 증가하지 않았다는 점입니다. 결과적으로 계좌이체는 실패하고 `memberA`의 돈만 2000원 줄어든 상황이 됩니다.

**강제 커밋**

![https://user-images.githubusercontent.com/52024566/191990567-41d32d4d-c94e-478a-9744-670fcfdfc7ec.png](https://user-images.githubusercontent.com/52024566/191990567-41d32d4d-c94e-478a-9744-670fcfdfc7ec.png)

만약 이 상황에서 강제로 `commit`을 호출하면 어떻게 될까요?

당연히 정상적인 계좌이체는 실패하고 `memberA`의 돈만 2000원 줄어드는 아주 심각한 문제가 발생하게 됩니다.

직접 커밋을 해서 확인해보면 위 그림처럼 되는 것을 확인할 수 있습니다.

**세션1 커밋**

```sql
commit;
```

**확인 쿼리**

```sql
select * from member;
```

이렇게 중간에 문제가 발생했을 때는 커밋을 호출하면 안됩니다!!

롤백을 호출해서 데이터를 트랜잭션 시작 시점으로 원복해야 합니다.

### 계좌이체 문제 상황 - 롤백

**기본 데이터 입력**

![https://user-images.githubusercontent.com/52024566/191990572-f9acc226-bf9b-467e-ab42-0d31e1c7808c.png](https://user-images.githubusercontent.com/52024566/191990572-f9acc226-bf9b-467e-ab42-0d31e1c7808c.png)

**기본 데이터 입력 - SQL**

```sql
set autocommit true;
delete from member;
insert into member(member_id, money) values ('memberA',10000);
insert into member(member_id, money) values ('memberB',10000);
```

**계좌이체 실행**

![https://user-images.githubusercontent.com/52024566/191990576-0212e9a1-dced-4cec-b853-ce39272cd504.png](https://user-images.githubusercontent.com/52024566/191990576-0212e9a1-dced-4cec-b853-ce39272cd504.png)

계좌이체를 실행하는 도중에 SQL에 문제가 발생했습니다. 그래서 `memberA`의 돈을 2000원 줄이는 것에는 성공했지만, `memberB`의 돈을 2000원 증가시키는 것에는 실패한 것이지요. 

두 번째 `update` 쿼리를 실행하면 SQL 오류가 발생하는 것을 확인할 수 있습니다.

**계좌이체 실행 SQL - 오류**

```sql
set autocommit false;
update member set money=10000 - 2000 where member_id = 'memberA'; //성공
update member set money=10000 + 2000 where member_iddd = 'memberB'; //쿼리 예외 발생
```

**두 번째 SQL 실행시 발생하는 오류 메시지**

```sql
Column "MEMBER_IDDD" not found; SQL statement:
update member set money=10000 + 2000 where member_iddd = 'memberB' [42122-200] 42S22/42122
```

여기서 문제는 `memberA`의 돈은 2000원 줄어들었지만, `memberB`의 돈은 2000원 증가하지 않았다는 점입니다. 

결과적으로 계좌이체는 실패하고 `memberA`의 돈만 2000원 줄어든 상황이 되어 버렸습니다.

**롤백**

![https://user-images.githubusercontent.com/52024566/191990580-0bfc28d1-5ef8-4849-9ecd-86cfba23637f.png](https://user-images.githubusercontent.com/52024566/191990580-0bfc28d1-5ef8-4849-9ecd-86cfba23637f.png)

이럴 때는 롤백을 호출해서 트랜잭션을 시작하기 전 단계로 데이터를 복구해야 합니다. 

아래에 보이는 코드를 통해서 롤백을 사용한 덕분에 계좌이체를 실행하기 전 상태로 원복할 수 있습니다. 

`memberA`의 돈도 이전 상태인 10000원으로 돌아오고, `memberB`의 돈도 10000원으로 유지되는 것을 확인할 수 있습니다!

**세션1 - 롤백**

```sql
rollback;
```

**확인 쿼리**

```sql
select * from member;
```

### 정리

**원자성** : 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공하거나 모두 실패해야 합니다! 

트랜잭션의 원자성 덕분에 여러 SQL 명령어를 마치 하나의 작업인 것처럼 처리할 수 있습니다. 성공하면 한번에 반영하고, 중간에 실패해도 마치 하나의 작업을 되돌리는 것 처럼 간단히 되돌릴 수 있습니다.

**오토 커밋 :** 만약 오토 커밋 모드로 동작하는데, 계좌이체 중간에 실패하면 어떻게 할까요? 쿼리를 하나 실행할 때마다 바로바로 커밋이 되어버리기 때문에 `memberA`의 돈만 2000원 줄어드는 심각한 문제가 발생하게 됩니다.

**트랜잭션 시작:** 따라서 이런 종류의 작업은 꼭 수동 커밋 모드를 사용해서 수동으로 커밋과 롤백을 할 수 있도록 해야 합니다. 보통 이렇게 자동 커밋 모드에서 수동 커밋 모드로 전환 하는 것을 트랜잭션을 시작한다고 표현합니다

# 7. DB 락 - 개념 이해

### **개념 이해**

세션1이 트랜잭션을 시작하고 데이터를 수정하는 동안 아직 커밋을 수행하지 않았는데, 세션2에서 동시에 같은 데이터를 수정하게 되면 여러가지 문제가 발생합니다. 

트랜잭션의 원자성이 깨지고 여기에 더해서 세션1이 중간에 롤백을 하게 되면 세션2는 잘못된 데이터를 수정하는 문제가 발생합니다.

이런 문제를 방지하려면, 당연히 세션이 트랜잭션을 시작하고 데이터를 수정하는 동안에는 커밋이나 롤백 전까지 다른 세션에서 해당 데이터를 수정할 수 없게 막아야 합니다.

**락0**

![https://user-images.githubusercontent.com/52024566/192289770-403240a0-373c-4f6b-b7e9-f73886cfbcf6.png](https://user-images.githubusercontent.com/52024566/192289770-403240a0-373c-4f6b-b7e9-f73886cfbcf6.png)

- 현재 세션1은 `memberA`의 금액을 500원으로 변경하고 싶고, 세션2는 같은 `memberA`의 금액을 1000원으로 변경하고 싶습니다.
- 데이터베이스는 이런 문제를 해결하기 위해 락(Lock)이라는 개념을 제공합니다.

**락1**

![https://user-images.githubusercontent.com/52024566/192289777-02459a65-ab83-46e6-9932-8218eace04f5.png](https://user-images.githubusercontent.com/52024566/192289777-02459a65-ab83-46e6-9932-8218eace04f5.png)

 1. 세션1은 트랜잭션을 시작합니다.

 2. 그리고 세션1은 `memberA`의 `money`를 500으로 변경을 시도합니다. 이 때 해당 row의 락을 먼저 획득해야 합니다. 해당 row의 락이 남아 있으므로 세션1은 락을 획득합니다.. (세션1이 세션2보다 조금 더 빨리 요청한 상황입니다.)

 3. 세션1은 락을 획득했으므로 해당 로우에 update sql을 수행합니다.

**락2**

![https://user-images.githubusercontent.com/52024566/192289782-e2a114b6-1b88-4767-9f14-b885464c185a.png](https://user-images.githubusercontent.com/52024566/192289782-e2a114b6-1b88-4767-9f14-b885464c185a.png)

 4. 그리고 세션2는 트랜잭션을 시작합니다.

 5. 세션2도 `memberA`의 `money` 데이터를 변경하려고 시도합니다. 이 때도 해당 로우의 락을 먼저 획득해야 하는데 락이 없으므로 락이 돌아올 때 까지 대기합니다.

(참고로 세션2가 락을 무한정 대기하는 것은 아닙니다. 락 대기 시간을 넘어가면 락 타임아웃 오류가 발생하게 됩니다. 락 대기 시간은 설정할 수 있습니다.)

**락3**

![https://user-images.githubusercontent.com/52024566/192289785-befdabb5-e5d6-427e-a405-01ac4988fc79.png](https://user-images.githubusercontent.com/52024566/192289785-befdabb5-e5d6-427e-a405-01ac4988fc79.png)

 6. 세션1은 커밋을 수행합니다. 커밋으로 트랜잭션이 종료되었으므로 락도 반납합니다.

**락4**

![https://user-images.githubusercontent.com/52024566/192289789-32005637-61f9-48dc-a70a-bbdf64e9e565.png](https://user-images.githubusercontent.com/52024566/192289789-32005637-61f9-48dc-a70a-bbdf64e9e565.png)

 7. 락을 획득하기 위해 대기하던 세션2가 락을 확득합니다.

**락5**

![https://user-images.githubusercontent.com/52024566/192289792-fbed62a5-bffc-4752-a0e9-780c7e9b1620.png](https://user-images.githubusercontent.com/52024566/192289792-fbed62a5-bffc-4752-a0e9-780c7e9b1620.png)

 8. 세션2는 update sql을 수행합니다.

**락6**

![https://user-images.githubusercontent.com/52024566/192289794-3dc94ea0-7179-4753-a110-548d1ffd39e2.png](https://user-images.githubusercontent.com/52024566/192289794-3dc94ea0-7179-4753-a110-548d1ffd39e2.png)

 9. 세션2는 커밋을 수행하고 트랜잭션이 종료되었으므로 락을 반납합니다.

이렇게 예시를 통해서 락(Lock) 이 무엇인지 알아보았습니다.

# 9. DB 락 - 조회

이번에는 DB 조회에서의 락을 살펴봅시다.

일반적인 조회는 락을 사용하지 않습니다!

- 데이터베이스마다 다르지만, 보통 데이터를 조회할 때는 락을 획득하지 않고 바로 데이터를 조회할 수 있습니다.
- 예를 들어서 세션1이 락을 획득하고 데이터를 변경하고 있어도, 세션2에서 데이터를 조회는 할 수 있습니다.
- 물론 세션2에서 조회가 아니라 데이터를 변경하려면 락이 필요하기 때문에 락이 돌아올 때 까지 대기해야 합니다.

**조회와 락**

- 그런데 데이터를 조회할 때도 락을 획득하고 싶을 때가 있습니다. 이럴 때는 `select for update` 구문을 사용하면 됩니다.
- 이렇게 하면 세션 1이 조회 시점에 락을 가져가버리기 때문에 다른 세션에서 해당 데이터를 변경할 수 없습니다.
- 물론 이 경우도 트랜잭션을 커밋하면 락을 반납하게 됩니다.

그렇다면 **조회 시점에 락이 필요한 경우가 무엇이 있을까요??**

- 트랜잭션 종료 시점까지 해당 데이터를 다른 곳에서 변경하지 못하도록 강제로 막아야 할 때 사용할 수 있습니다.
- 예를 들어서 애플리케이션 로직에서 `memberA` 의 금액을 조회한 다음에 이 금액 정보로 애플리케이션에서 어떤 계산을 수행한다고 합시다. 그런데 이 계산이 돈과 관련된 매우 중요한 계산이어서 계산을 완료할 때까지 `memberA`의 금액을 다른곳에서 변경하면 안 되는 상황인 것이지요. 이럴 때 조회 시점에 락을 획득해야 합니다.

![https://user-images.githubusercontent.com/52024566/192801917-ff671813-8894-4e7b-aaf9-b24d96bca116.png](https://user-images.githubusercontent.com/52024566/192801917-ff671813-8894-4e7b-aaf9-b24d96bca116.png)

**기본 데이터 입력 - SQL**

```sql
set autocommit true;
delete from member;
insert into member(member_id, money) values ('memberA',10000);
```

**세션 1**

```sql
set autocommit false;
select * from member where member_id='memberA' for update;
```

- `select for update`구문을 사용하면 조회를 하면서 동시에 선택한 로우의 락도 획득할 수 있습니다. 물론 락이 없다면 락을 획득할 때 까지 대기해야 합니다.
- 세션 1은 트랜잭션을 종료할 때까지 `memberA`의 row의 락을 보유합니다.

**세션 2**

```sql
set autocommit false;
update member set money=500 where member_id = 'memberA';
```

- 세션2는 데이터를 변경하고 싶으므로 락이 필요합니다.
- 세션1이 `memberA` 로우의 락을 획득했기 때문에 세션 2는 락을 획득할 때 까지 대기해야 합니다.
- 이후에 세션 1이 커밋을 수행하면 세션 2가 락을 획득하고 데이터를 변경합니다. 
(물론 락 타임아웃 시간이 지나면 락 타임아웃 예외가 발생합니다)

**세션 1 커밋**

`commit;`

세션 2도 커밋해서 데이터를 반영합니다.

**세션 2 커밋**

`commit;`

트랜잭션과 락은 데이터베이스마다 실제 동작하는 방식이 조금씩 다르기 때문에, 해당 데이터베이스 메뉴얼을 확인해보고, 의도한대로 동작하는지 테스트한 이후에 사용하면 됩니다.


# 10. 트랜잭션 - 적용1

이제 직접 프로젝트에 로직을 구현하면서 트랜잭션, 락을 적용해봅시다.

### **비즈니스 로직 구현**

`MemberServiceV1`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV1;
import lombok.RequiredArgsConstructor;

import java.sql.SQLException;

@RequiredArgsConstructor
public class MemberServiceV1 {
    private final MemberRepositoryV1 memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체 중 예외 발생!!");
        }
    }
}
```

- `fromId`의 회원을 조회해서 `toId`의 회원에게 `money`만큼의 돈을 계좌이체 하는 로직입니다.
    - `fromId` 회원의 돈을 `money`만큼 감소하는 것은 `MemberRepositoryV1` 클래스에서 구현해 놓은 update 메서드를 통해 UPDATE SQL 실행
    - `toId` 회원의 돈을 `money`만큼 증가하는 것은 `MemberRepositoryV1` 클래스에서 구현해 놓은 update 메서드를 통해 UPDATE SQL 실행

- 예외 상황을 테스트해보기 위해 `toId`가 `"ex"`인 경우 예외가 발생하도록 만들어 두었습니다.

`MemberServiceV1Test`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV1;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

import java.sql.SQLException;

import static hello.jdbc.connection.ConnectionConst.*;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.AssertionsForClassTypes.assertThatThrownBy;
import static org.junit.jupiter.api.Assertions.*;

/**
 * 기본 동작, 트랜잭션이 없어서 문제 발생
 */
class MemberServiceV1Test {
    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    private MemberRepositoryV1 memberRepository;
    private MemberServiceV1 memberService;

    @BeforeEach
    void before() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        memberRepository = new MemberRepositoryV1(dataSource);
        memberService = new MemberServiceV1(memberRepository);
    }

    @AfterEach
    void after() throws SQLException {
        memberRepository.delete(MEMBER_A);
        memberRepository.delete(MEMBER_B);
        memberRepository.delete(MEMBER_EX);
    }

    @Test
    @DisplayName("정상 이체")
    void accountTransfer() throws SQLException {
        // given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberB = new Member(MEMBER_B, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberB);

        // when
				memberService.accountTransfer(memberA.getMemberId(), memberB.getMemberId(), 2000);

        // then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberB.getMemberId());

        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberB.getMoney()).isEqualTo(12000);
    }

    @Test
    @DisplayName("이체 중에 예외 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        //when
        assertThatThrownBy(() -> memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 2000)).isInstanceOf(IllegalStateException.class);

        // then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberEx = memberRepository.findById(memberEx.getMemberId());

        // memberA 의 돈만 2000 원 줄고 ex 의 돈은 그대로 10000 원인 예외 발생
        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberEx.getMoney()).isEqualTo(10000);
    }

}
```

**주의 -** 테스트를 수행하기 전에 아래 SLQ 문을 통해 데이터베이스의 데이터를 삭제해야 합니다.

```sql
delete from member;
```

정상이체 - `accountTransfer()`

- given: 다음 데이터를 저장해서 테스트를 준비
    - `memberA` 10000원
    - `memberB` 10000원

- when: 계좌이체 로직을 실행
    - `memberService.accountTransfer()`를 실행
    - `memberA` → `memberB`로 2000원 계좌이체
        - `memberA`의 금액이 2000원 감소
        - `memberB`의 금액이 2000원 증가

- then: 계좌이체가 정상 수행되었는지 검증
    - `memberA` 8000원 - 2000원 감소
    - `memberB` 12000원 - 2000원 증가

테스트 결과 정상이체 로직이 정상 수행되는 것을 확인할 수 있습니다.

**테스트 데이터 제거** 

테스트가 끝나면 다음 테스트에 영향을 주지 않기 위해 `@AfterEach`에서 테스트에 사용한 데이터를 모두 삭제하고 있습니다.

- `@BeforeEach`: 각각의 테스트가 수행되기 전에 실행
- `@AfterEach`: 각각의 테스트가 실행되고 난 이후에 실행

`@AfterEach`

```java
void after() throws SQLException {
memberRepository.delete(MEMBER_A);
memberRepository.delete(MEMBER_B);
memberRepository.delete(MEMBER_EX);
}
```

- 테스트 데이터를 제거하는 과정이 불편하지만, 다음 테스트에 영향을 주지 않으려면 테스트에서 사용한 데이터를 모두 제거해야 합니다. 그렇지 않으면 이번 테스트에서 사용한 데이터 때문에 다음 테스트에서 데이터 중복으로 오류가 발생할 수 있습니다
- 테스트에서 사용한 데이터를 제거하는 더 나은 방법으로는 트랜잭션을 활용할 수 있습니다. 테스트 전에 트랜잭션을 시작하고, 테스트 이후에 트랜잭션을 롤백해버리면 데이터가 처음 상태로 돌아오게 됩니다.

이체중 예외 발생 - `accountTransferEx()`

- given: 다음 데이터를 저장해서 테스트를 준비
    - `memberA` 10000원
    - `memberEx` 10000원

- when: 계좌이체 로직을 실행
    - `memberService.accountTransfer()`를 실행
    - `memberA` → `memberEx`로 2000원 계좌이체
        - `memberA`의 금액이 2000원 감소
        - `memberEx` 회원의 ID는 `ex`이므로 중간에 예외가 발생

- then: 계좌이체는 실패. memberA 의 돈만 2000원 감소
    - `memberA` 8000원 - 2000원 감소
    - `memberB` 10000원 - 중간에 실패로 로직이 수행되지 않았으므로 그대로 10000원으로 남아있음

### **정리**

이체중 예외가 발생하게 되면 `memberA`의 금액은 10000원 → 8000원으로 2000원 감소합니다. 

그런데 `memberB`의 돈은 그대로 10000원으로 그대로 유지됩니다. 결과적으로 `memberA`의 돈만 2000원 감소되는 심각한 오류(예외)가 발생합니다.

# 11. 트랜잭션 - 적용2

### **DB 트랜잭션 사용**

**비즈니스 로직과 트랜잭션**

![https://user-images.githubusercontent.com/52024566/193279833-0b93f926-7ea0-40b4-b787-9fc12625d7a8.png](https://user-images.githubusercontent.com/52024566/193279833-0b93f926-7ea0-40b4-b787-9fc12625d7a8.png)

트랜잭션은 비즈니스 로직이 있는 서비스 계층에서 시작해야 합니다. 비즈니스 로직이 잘못되면 해당 비즈니스 로직으로 인해 문제가 되는 부분을 함께 롤백해야 하기 때문입니다.

그런데 트랜잭션을 시작하려면 커넥션이 필요합니다. 결국 서비스 계층에서 커넥션을 만들고, 트랜잭션 커밋 이후에 커넥션을 종료해야 하는 것이지요.

애플리케이션에서 DB 트랜잭션을 사용하려면 트랜잭션을 사용하는 동안 같은 커넥션을 유지해야 합니다. 그래야 같은 세션을 사용할 수 있습니다.

### **커넥션과 세션**

![https://user-images.githubusercontent.com/52024566/193279842-02595a67-21a3-4763-a149-b4821acf755d.png](https://user-images.githubusercontent.com/52024566/193279842-02595a67-21a3-4763-a149-b4821acf755d.png)

애플리케이션에서 같은 커넥션을 유지하는 가장 단순한 방법은 **커넥션을 파라미터로 전달**해서 같은 커넥션이 사용되도록 유지하는 것입니다.

`MemberRepositoryV2`

```java
package hello.jdbc.repository;

import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.support.JdbcUtils;

import javax.sql.DataSource;
import java.sql.*;
import java.util.NoSuchElementException;

@Slf4j
/**
 * JDBC - ConnectionParam
 */
public class MemberRepositoryV2 {

    private final DataSource dataSource;

    public MemberRepositoryV2(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values(?, ?)";

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

    public Member findById(Connection con, String memberId) throws SQLException {
        String sql = "select * from member where member_id = ?";

        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
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
            //connection은 여기서 닫지 않는다.
            JdbcUtils.closeResultSet(rs);
            JdbcUtils.closeStatement(pstmt);
        }
    }

    public void update(String memberId, int money) throws SQLException {

        String sql = "update member set money=? where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    public void update(Connection con, String memberId, int money) throws SQLException {

        String sql = "update member set money=? where member_id=?";

        PreparedStatement pstmt = null;

        try {
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            //connection은 여기서 닫지 않는다.
            JdbcUtils.closeStatement(pstmt);
        }
    }

    public void delete(String memberId) throws SQLException {

        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        JdbcUtils.closeConnection(con);
    }

    private Connection getConnection() throws SQLException {
        Connection con = dataSource.getConnection();
        log.info("get connection={} class={}", con, con.getClass());
        return con;
    }
}
```

`MemberRepositoryV2`는 기존 코드와 같고 커넥션 유지가 필요한 아래 두 메서드만 추가되었습니다. 참고로 다음 두 메서드는 계좌이체 서비스 로직에서 호출하는 메서드입니다.

- `findById(Connection con, String memberId)`
- `update(Connection con, String memberId, int money)`

**주의**

1. 커넥션 유지가 필요한 두 메서드는 파라미터로 넘어온 커넥션을 사용해야 합니다. 따라서 `con = getConnection()` 코드가 있으면 안됩니다.
2. 커넥션 유지가 필요한 두 메서드는 리포지토리에서 커넥션을 닫으면 안 됩니다. 커넥션을 전달 받은 리포지토리 뿐만 아니라 이후에도 커넥션을 계속 이어서 사용하기 때문입니다. 
이후에 서비스 로직이 끝날 때 트랜잭션을 종료하고 닫아야 합니다

`MemberServiceV2`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV2;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

/**
 * 트랜잭션 - 파라미터 연동, 풀을 고려한 종료
 */
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV2 {
    private final DataSource dataSource;
    private final MemberRepositoryV2 memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Connection con = dataSource.getConnection();
        try {
            con.setAutoCommit(false); // 트랜잭션 시작
            // 비즈니스 로직
            bizLogic(con, fromId, toId, money);
            con.commit(); // 성공 시 커밋하기
        } catch (Exception e) {
            con.rollback(); // 실패 시 롤백
            throw new IllegalStateException(e);
        } finally {
            release(con);
        }
    }

    private void bizLogic(Connection con, String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(con, fromId);
        Member toMember = memberRepository.findById(con, toId);

        memberRepository.update(con, fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(con, toId, toMember.getMoney() + money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체 중 예외 발생");
        }
    }

    private void release(Connection con) {
        if (con != null) {
            try {
                con.setAutoCommit(true); // 커넥션 풀을 고려하기
                con.close();
            } catch (Exception e) {
                log.info("error", e);
            }
        }
    }
}
```

- `Connection con = dataSource.getConnection();`
    - 트랜잭션을 시작하려면 커넥션이 필요합니다.

- `con.setAutoCommit(false); //트랜잭션 시작`
    - 트랜잭션을 시작하려면 자동 커밋 모드를 꺼야 합니다. 
    이렇게 하면 커넥션을 통해 세션에 `set autocommit false`가 전달되고, 이후부터는 수동 커밋 모드로 동작합니다. 
    이렇게 자동 커밋 모드를 수동 커밋 모드로 변경하는 것을 트랜잭션을 시작한다고 합니다.

- `bizLogic(con, fromId, toId, money);`
    - 트랜잭션이 시작된 커넥션을 전달하면서 비즈니스 로직을 수행합니다.
    - 이렇게 분리한 이유는 트랜잭션을 관리하는 로직과 실제 비즈니스 로직을 구분하기 위함입니다.
    - `memberRepository.update(con..)`: 비즈니스 로직을 보면 리포지토리를 호출할 때 커넥션을 전달하는 것을 확인할 수 있습니다.

- `con.commit(); //성공시 커밋`
    - 비즈니스 로직이 정상 수행되면 트랜잭션을 커밋합니다.

- `con.rollback(); //실패시 롤백`
    - `catch(Ex){..}`를 사용해서 비즈니스 로직 수행 도중에 예외가 발생하면 트랜잭션을 롤백합니다.

- `release(con);`
    - `finally {..}`를 사용해서 커넥션을 모두 사용하고 나면 안전하게 종료하도록 합니다. 
    그런데 커넥션 풀을 사용하면 `con.close()`를 호출 했을 때 커넥션이 종료되는 것이 아니라 풀에 반납됩니다. 현재 수동 커밋 모드로 동작하기 때문에 풀에 돌려주기 전에 기본 값인 자동 커밋 모드로 변경하는 것이 안전합니다.

이제 테스트를 실행해봅시다. 아래 테스트 코드를 만듭니다.

`MemberServiceV2Test`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV2;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

import java.sql.SQLException;

import static hello.jdbc.connection.ConnectionConst.*;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.AssertionsForClassTypes.assertThatThrownBy;
import static org.junit.jupiter.api.Assertions.*;

/**
 * 트랜잭션 - 커넥션 파라미터 전달 방식 동기화
 */
class MemberServiceV2Test {
    private MemberRepositoryV2 memberRepository;
    private MemberServiceV2 memberService;

    @BeforeEach
    void before() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        memberRepository = new MemberRepositoryV2(dataSource);
        memberService = new MemberServiceV2(dataSource, memberRepository);
    }

    @AfterEach
    void after() throws SQLException {
        memberRepository.delete("memberA");
        memberRepository.delete("memberB");
        memberRepository.delete("ex");
    }

    @Test
    @DisplayName("정상 이해")
    void accountTransfer() throws SQLException {
        //given
        Member memberA = new Member("memberA", 10000);
        Member memberB = new Member("memberB", 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberB);

        //when
        memberService.accountTransfer(memberA.getMemberId(), memberB.getMemberId(), 2000);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberB.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberB.getMoney()).isEqualTo(12000);
    }

    @Test
    @DisplayName("이체중 예외 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member("memberA", 10000);
        Member memberEx = new Member("ex", 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        //when
        assertThatThrownBy(() -> memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 2000))
                .isInstanceOf(IllegalStateException.class);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberEx = memberRepository.findById(memberEx.getMemberId());

        //memberA의 돈이 롤백 되어야함
        assertThat(findMemberA.getMoney()).isEqualTo(10000);
        assertThat(findMemberEx.getMoney()).isEqualTo(10000);
    }

}
```

정상이체 - `accountTransfer()` 

기존 로직과 같음

이체중 예외 발생 - `accountTransferEx()`

- 다음 데이터를 저장해서 테스트를 준비
    - `memberA` 10000원
    - `memberEx` 10000원

- 계좌이체 로직을 실행
    - `memberService.accountTransfer()`를 실행
    - 커넥션을 생성하고 트랜잭션을 시작
    - `memberA` → `memberEx`로 2000원 계좌이체
        - `memberA`의 금액이 2000원 감소
        - `memberEx` 회원의 ID는 `ex`이므로 중간에 예외가 발생
    - 예외가 발생했으므로 트랜잭션을 롤백

- 계좌이체가 실패했으므로 롤백을 수행해서 `memberA`의 돈이 기존 10000원으로 복구
    - `memberA` 10000원 - 트랜잭션 롤백으로 복구
    - `memberB` 10000원 - 중간에 실패로 로직이 수행되지 않음. 따라서 그대로 10000원으로 남아있음

트랜잭션 덕분에 계좌이체가 실패할 때 롤백을 수행해서 모든 데이터를 정상적으로 초기화 할 수 있게 되었습니다.

롤백이 되면 계좌이체를 수행하기 직전으로 돌아갑니다!

**그런데도 남은 문제가 있습니다.**

애플리케이션에서 DB 트랜잭션을 적용하려면 서비스 계층이 매우 지저분해지고, 생각보다 매우 복잡한 코드를 요구합니다. 추가로 커넥션을 유지하도록 코드를 변경하는 것도 쉬운 일은 아닙니다.

늘 그랬듯이 다음 글에서 이 문제를 스프링을 통해서 해결하는 것을 알아봅시다.

# ==== 4. 스프링과 트랜잭션 문제 해결 ====


# 1. 트랜잭션 문제점

### **애플리케이션 구조**

여러가지 애플리케이션 구조가 있지만, 가장 단순하면서 많이 사용하는 방법은 역할에 따라 3가지 계층으로 나누는 것입니다.

![https://user-images.githubusercontent.com/52024566/193834311-85946dbd-7992-40b1-987c-046f50fff23b.png](https://user-images.githubusercontent.com/52024566/193834311-85946dbd-7992-40b1-987c-046f50fff23b.png)

- 프레젠테이션 계층
    - UI와 관련된 처리 담당
    - 웹 요청과 응답
    - 사용자 요청을 검증
    - 주 사용 기술: 서블릿과 HTTP 같은 웹 기술, 스프링 MVC

- 서비스 계층
    - 비즈니스 로직을 담당
    - 주 사용 기술: 가급적 특정 기술에 의존하지 않고, 순수 자바 코드로 작성

- 데이터 접근 계층
    - 실제 데이터베이스에 접근하는 코드
    - 주 사용 기술: JDBC, JPA, File, Redis, Mongo …

### **순수한 서비스 계층**

- 여기서 가장 중요한 곳은 바로 핵심 비즈니스 로직이 들어있는 서비스 계층입니다. 시간이 흘러서 UI(웹)와 관련된 부분이 변하고, 데이터 저장 기술을 다른 기술로 변경해도, 비즈니스 로직은 최대한 변경없이 유지되어야 합니다.
- 이렇게 하려면 서비스 계층을 특정 기술에 종속적이지 않게 개발해야 합니다.

애초에 이렇게 계층을 나눈 이유도 서비스 계층을 최대한 순수하게 유지하기 위한 목적이 대부분입니다. 기술에 종속적인 부분은 프레젠테이션 계층과 데이터 접근 계층에서 가지고 갑니다.

**프레젠테이션 계층은 클라이언트가 접근하는 UI와 관련된 기술인 웹, 서블릿, HTTP와 관련된 부분을 담당**합니다. 그래서 서비스 계층을 이런 UI와 관련된 기술로부터 보호하는 것입니다. 
예를 들어서 HTTP API를 사용하다가 GRPC 같은 기술로 변경해도 프레젠테이션 계층의 코드만 변경하고, 서비스 계층은 변경하지 않아도 됩니다.

**데이터 접근 계층은 데이터를 저장하고 관리하는 기술을 담당**합니다. 그래서 JDBC, JPA와 같은 구체적인 데이터 접근 기술로부터 서비스 계층을 보호합니다. 
예를 들어서 JDBC를 사용하다가 JPA로 변경해도 서비스 계층은 변경하지 않아도 됩니다. 물론 서비스 계층에서 데이터 접근 계층을 직접 접근하는 것이 아니라, 인터페이스를 제공하고 서비스 계층은 이 인터페이스에 의존하는 것이 좋습니다. 그래야 서비스 코드의 변경 없이 `JdbcRepository` 를 `JpaRepository` 로 변경할 수 있습니다.

서비스 계층이 특정 기술에 종속되지 않기 때문에 비즈니스 로직을 유지보수 하기도 쉽고, 테스트 하기도 쉬워집니다.

정리하자면 **서비스 계층은 가급적 비즈니스 로직만 구현**하고 **특정 구현 기술에 직접 의존해서는 안되는 것입니다**.

이렇게 하면 **향후 구현 기술이 변경될 때 변경의 영향 범위를 최소화**할 수 있습니다.

### **문제점들**

이전에 만들었던 `MemberServiceV1` 코드를 살펴봅시다.

`MemberServiceV1`

```java
import java.sql.SQLException;

@RequiredArgsConstructor
public class MemberServiceV1 {
  
    private final MemberRepositoryV1 memberRepository;
  
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);
        memberRepository.update(fromId, fromMember.getMoney() - money);
        memberRepository.update(toId, toMember.getMoney() + money);
    }
}
```

`MemberServiceV1`은 특정 기술에 종속적이지 않고, 순수한 비즈니스 로직만 존재합니다.

특정 기술과 관련된 코드가 거의 없어서 코드가 깔끔하고, 유지보수 하기 쉽습니다.

만약 향후에 비즈니스 로직의 변경이 필요하면 이 부분을 변경하면 됩니다.

**현재 남은 문제는 크게 두 가지 문제가 있습니다.**

현재는 `SQLException`이라는 JDBC 기술에 의존하고 있습니다. 이 부분은 애초에 `memberRepository`에서 올라오는 예외이기 때문에 `memberRepository`에서 해결해야 합니다.

`MemberRepositoryV1`이라는 구체 클래스에 직접 의존하고 있습니다. `MemberRepository` 인터페이스를 도입하면 향후 `MemberService`의 코드의 변경 없이 다른 Repository 구현 기술로 손쉽게 변경할 수 있습니다.

그렇다면 두번째 버전으로 만들었던 `MemberServiceV2` 코드를 살펴봅시다.

`MemberServiceV2`

```java
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@Slf4j
@RequiredArgsConstructor
public class MemberServiceV2 {
  
    private final DataSource dataSource;
    private final MemberRepositoryV2 memberRepository;
  
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Connection con = dataSource.getConnection();
        try {
            con.setAutoCommit(false); //트랜잭션 시작
            //비즈니스 로직
            bizLogic(con, fromId, toId, money);
            con.commit(); //성공시 커밋
        } catch (Exception e) {
            con.rollback(); //실패시 롤백
            throw new IllegalStateException(e);
        } finally {
            release(con);
        }
    }
  
    private void bizLogic(Connection con, String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(con, fromId);
        Member toMember = memberRepository.findById(con, toId);
      
        memberRepository.update(con, fromId, fromMember.getMoney() - money);
        memberRepository.update(con, toId, toMember.getMoney() + money);
    }
}
```

**트랜잭션은 비즈니스 로직이 있는 서비스 계층에서 시작하는 것이 좋습니다.**

그런데 문제는 트랜잭션을 사용하기 위해서 `javax.sql.DataSource`, `java.sql.Connection`, `java.sql.SQLException` 같은 JDBC 기술에 의존해야 한다는 점입니다.

트랜잭션을 사용하기 위해 JDBC 기술에 의존하기 때문에 결과적으로 비즈니스 로직보다 JDBC를 사용해서 트랜잭션을 처리하는 코드가 더 많습니다.

향후 JDBC에서 JPA 같은 다른 기술로 바꾸어 사용하게 되면 서비스 코드도 모두 함께 변경해야 합니다. 

(JPA는 트랜잭션을 사용하는 코드가 JDBC와 다름)

핵심 비즈니스 로직과 JDBC 기술이 섞여 있어서 유지보수 하기 어렵습니다.

### **문제 정리**

**즉, 지금까지 개발한 애플리케이션의 문제점은 크게 3가지가 있습니다.**

- 트랜잭션 문제
- 예외 누수 문제
- JDBC 반복 문제

### **트랜잭션 문제**

가장 큰 문제는 트랜잭션을 적용하면서 생긴 다음과 같은 문제들입니다.

**JDBC 구현 기술이 서비스 계층에 누수되는 문제**

- 트랜잭션을 적용하기 위해 JDBC 구현 기술이 서비스 계층에 누수되었습니다.
- 서비스 계층은 순수해야 합니다. 구현 기술을 변경해도 서비스 계층 코드는 최대한 유지할 수 있어야 합니다 (변화에 대응)
    - 그래서 데이터 접근 계층에 JDBC 코드를 다 몰아두어야 하는 것이지요
    - 물론 데이터 접근 계층의 구현 기술이 변경될 수도 있으니 데이터 접근 계층은 인터페이스를 제공하는 것이 좋습니다.
    - 서비스 계층은 특정 기술에 종속되지 않아야 합니다. 지금까지 그렇게 노력해서 데이터 접근 계층으로 JDBC 관련 코드를 모았는데, 트랜잭션을 적용하면서 결국 서비스 계층에 JDBC 구현 기술의 누수가 발생합니다.

- 트랜잭션 동기화 문제
    - 같은 트랜잭션을 유지하기 위해 커넥션을 파라미터로 넘겨야 합니다.
    - 이때 파생되는 문제들도 있습니다. 똑같은 기능도 트랜잭션용 기능과 트랜잭션을 유지하지 않아도 되는 기능으로 분리해야 합니다.

- 트랜잭션 적용 반복 문제
    - 트랜잭션 적용 코드를 보면 줄일 수 있는 반복이 많습니다.. `try`, `catch`, `finally` …

**예외 누수**

- 데이터 접근 계층의 JDBC 구현 기술 예외가 서비스 계층으로 전파되었습니다.
- `SQLException`은 체크 예외이기 때문에 데이터 접근 계층을 호출한 서비스 계층에서 해당 예외를 잡아서 처리하거나 명시적으로 `throws`를 통해서 다시 밖으로 던져야 합니다.
- `SQLException`은 JDBC 전용 기술입니다. 향후 JPA나 다른 데이터 접근 기술을 사용하면, 그에 맞는 다른 예외로 변경해야 하고, 결국 서비스 코드도 수정해야 하는 문제가 있습니다.

**JDBC 반복 문제**

- 지금까지 작성한 `MemberRepository` 코드는 순수한 JDBC를 사용합니다.
- 이 코드들은 유사한 코드의 반복이 너무 많습니다. 아래와 같이 말이죠
    - `try`, `catch`, `finally` …
    - 커넥션을 열고, `PreparedStatement`를 사용하고, 결과를 매핑하고… 실행하고, 커넥션과 리소스를 정리하는 코드

# 2. 트랜잭션 추상화

현재 서비스 계층은 트랜잭션을 사용하기 위해서 JDBC 기술에 의존하고 있습니다. 향후 JDBC에서 JPA 같은 다른 데이터 접근 기술로 변경하면, 서비스 계층의 트랜잭션 관련 코드도 모두 함께 수정해야 합니다.

### **구현 기술에 따른 트랜잭션 사용법**

- 트랜잭션은 원자적 단위의 비즈니스 로직을 처리하기 위해 사용합니다.
- 구현 기술마다 트랜잭션을 사용하는 방법이 다릅니다. 예를 들어서 아래처럼 말이죠.
    - JDBC : `con.setAutoCommit(false)`
    - JPA : `transaction.begin()`

**JDBC 트랜잭션 코드 예시**

```java
public void accountTransfer(String fromId, String toId, int money) throws SQLException {
    Connection con = dataSource.getConnection();
    try {
        con.setAutoCommit(false); //트랜잭션 시작
        //비즈니스 로직
        bizLogic(con, fromId, toId, money);
        con.commit(); //성공시 커밋
    } catch (Exception e) {
        con.rollback(); //실패시 롤백
        throw new IllegalStateException(e);
    } finally {
        release(con);
    }
}
```

**JPA 트랜잭션 코드 예시**

```java
public static void main(String[] args) {
    //엔티티 매니저 팩토리 생성
    EntityManagerFactory emf =     Persistence.createEntityManagerFactory("jpabook");
    EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성
    EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득
  
    try {
        tx.begin(); //트랜잭션 시작
        logic(em); //비즈니스 로직
        tx.commit();//트랜잭션 커밋
    } catch (Exception e) {
        tx.rollback(); //트랜잭션 롤백
    } finally {
        em.close(); //엔티티 매니저 종료
    }
    emf.close(); //엔티티 매니저 팩토리 종료
}
```

이렇게 트랜잭션을 사용하는 코드는 데이터 접근 기술마다 다릅니다. 만약 다음 그림과 같이 JDBC 기술을 사용하고, JDBC 트랜잭션에 의존하다가 JPA 기술로 변경하게 되면 서비스 계층의 트랜잭션을 처리하는 코드도 모두 함께 변경해야 합니다.

아래 그림을 보면 더 이해하기 쉽습니다.

### **JDBC 트랜잭션 의존**

![https://user-images.githubusercontent.com/52024566/194064238-fdfc9f24-17f8-431a-aee6-8397add1f7e9.png](https://user-images.githubusercontent.com/52024566/194064238-fdfc9f24-17f8-431a-aee6-8397add1f7e9.png)

### **JDBC 기술 → JPA 기술로 변경**

![https://user-images.githubusercontent.com/52024566/194064244-3e1ccaf9-ebbc-4633-bfd5-6406040c23e5.png](https://user-images.githubusercontent.com/52024566/194064244-3e1ccaf9-ebbc-4633-bfd5-6406040c23e5.png)

이렇게 JDBC 기술을 사용하다가 JPA 기술로 변경하게 되면 서비스 계층의 코드도 JPA 기술을 사용하도록 함께 수정해야 합니다.

### **트랜잭션 추상화**

이 문제를 해결하려면 트랜잭션 기능을 추상화하면 됩니다. 

아주 단순하게 생각하면 다음과 같은 인터페이스를 만들어서 사용하면 되겠죠

`TxManager` - 트랜잭션 추상화 인터페이스

```java
public interface TxManager {
    begin();
    commit();
    rollback();
}
```

트랜잭션은 크게 복잡한 기능 없이 단순합니다. 트랜잭션을 시작하고, 비즈니스 로직의 수행이 끝나면 커밋하거나 롤백하면 됩니다.

그리고 다음과 같이 `TxManager` 인터페이스를 기반으로 각각의 기술에 맞는 구현체를 만들면 됩니다.

- `JdbcTxManager`: JDBC 트랜잭션 기능을 제공하는 구현체
- `JpaTxManager`: JPA 트랜잭션 기능을 제공하는 구현체

### **트랜잭션 추상화와 의존관계**

![https://user-images.githubusercontent.com/52024566/194064252-9b8bbbec-f9d0-4427-b61b-b425a54ac296.png](https://user-images.githubusercontent.com/52024566/194064252-9b8bbbec-f9d0-4427-b61b-b425a54ac296.png)

이렇게 설계하면 서비스는 특정 트랜잭션 기술에 직접 의존하는 것이 아니라, `TxManager`라는 추상화된 인터페이스에 의존합니다. 이제 원하는 구현체를 DI를 통해서 주입하여 사용합니다. 

예를 들어서 JDBC 트랜잭션 기능이 필요하면 `JdbcTxManager`를 서비스에 주입하고, JPA 트랜잭션 기능으로 변경해야 하면 `JpaTxManager`를 주입하면 되는 것입니다.

클라이언트인 서비스는 인터페이스에 의존하고 DI를 사용한 덕분에 OCP 원칙을 지키게 됩니다. 이제 트랜잭션을 사용하는 서비스 코드를 전혀 변경하지 않고, 트랜잭션 기술을 마음껏 변경할 수 있습니다.

### **스프링의 트랜잭션 추상화**

그런데 개발자는 직접 인터페이스를 만들 필요없이 스프링이 제공하는 트랜잭션 추상화 기술을 사용하면 됩니다. 심지어 스프링에서는 데이터 접근 기술에 따른 트랜잭션 구현체도 대부분 만들어두어서 가져다 사용하기만 하면 됩니다.

![https://user-images.githubusercontent.com/52024566/194064254-2764c447-95b6-4991-a630-efdb8d2df11a.png](https://user-images.githubusercontent.com/52024566/194064254-2764c447-95b6-4991-a630-efdb8d2df11a.png)

스프링 트랜잭션 추상화의 핵심은 `PlatformTransactionManager` 인터페이스입니다.

- `org.springframework.transaction.PlatformTransactionManager`

> 참고 - 스프링 5.3부터는 JDBC 트랜잭션을 관리할 때 DataSourceTransactionManager를 상속받아서 약간의 기능을 확장한 JdbcTransactionManager를 제공한다. 둘의 기능 차이는 크지 않으므로 같은 것으로 이해하면 된다.
> 

`PlatformTransactionManager` 인터페이스

```java
public interface PlatformTransactionManager extends TransactionManager {
  
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
  
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```

`getTransaction()`: 트랜잭션을 시작

- 이름이 make 가 아니고 `getTransaction()` 인 이유는 기존에 이미 진행중인 트랜잭션이 있는 경우 해당 트랜잭션에 참여할 수 있기 때문임.

`commit()`: 트랜잭션을 커밋

`rollback()`: 트랜잭션을 롤백

# 3. 트랜잭션 동기화

스프링이 제공하는 트랜잭션 매니저의 역할은 두 가지입니다.

- 트랜잭션 추상화
- 리소스 동기화

### **리소스 동기화**

트랜잭션을 유지하려면 트랜잭션의 시작부터 끝까지 **같은 데이터베이스 커넥션을 유지해야** 합니다. 

결국 같은 커넥션을 동기화(맞추어 사용)하기 위해서 이전에는 파라미터로 커넥션을 전달하는 방법을 사용했지요.

파라미터로 커넥션을 전달하는 방법은 코드가 지저분해지는 것은 물론이고, 커넥션을 넘기는 메서드와 넘기지 않는 메서드를 중복해서 만들어야 하는 등 여러가지 단점들이 많습니다.

### **커넥션과 세션**

![https://user-images.githubusercontent.com/52024566/194273214-f2449bc1-1a79-441d-8343-f7e10fb23820.png](https://user-images.githubusercontent.com/52024566/194273214-f2449bc1-1a79-441d-8343-f7e10fb23820.png)

### **트랜잭션 매니저와 트랜잭션 동기화 매니저**

![https://user-images.githubusercontent.com/52024566/194273225-6787d41b-536f-4db7-8115-97d7049afde6.png](https://user-images.githubusercontent.com/52024566/194273225-6787d41b-536f-4db7-8115-97d7049afde6.png)

스프링은 **트랜잭션 동기화 매니저**를 제공합니다. 

이것은 쓰레드 로컬(`ThreadLocal`)을 사용해서 커넥션을 동기화합니다. 트랜잭션 매니저는 내부에서 이 트랜잭션 동기화 매니저를 사용합니다. (쓰레드 로컬은 해당 쓰레드만 접근할 수 있는 특별한 저장소)

트랜잭션 동기화 매니저는 쓰레드 로컬을 사용하기 때문에 멀티쓰레드 상황에 안전하게 커넥션을 동기화 할 수 있습니다. 따라서 커넥션이 필요하면 트랜잭션 동기화 매니저를 통해 커넥션을 획득하면 됩니다. 따라서 이전처럼 파라미터로 커넥션을 전달하지 않아도 됩니다.

### **동작 방식**

1. 트랜잭션을 시작하려면 커넥션이 필요합니다. 트랜잭션 매니저는 데이터소스를 통해 커넥션을 만들고 트랜잭션을 시작합니다.
2. 트랜잭션 매니저는 트랜잭션이 시작된 커넥션을 트랜잭션 동기화 매니저에 보관합니다.
3. 리포지토리는 트랜잭션 동기화 매니저에 보관된 커넥션을 꺼내서 사용합니다. 따라서 파라미터로 커넥션을 전달하지 않아도 됩니다.
4. 트랜잭션이 종료되면 트랜잭션 매니저는 트랜잭션 동기화 매니저에 보관된 커넥션을 통해 트랜잭션을 종료하고, 커넥션도 종료합니다.

**트랜잭션 동기화 매니저**

다음 트랜잭션 동기화 매니저 클래스를 열어보면 쓰레드 로컬을 사용하는 것을 확인할 수 있습니다. 

`org.springframework.transaction.support.TransactionSynchronizationManager`

> 참고 - 쓰레드 로컬을 사용하면 각각의 쓰레드마다 별도의 저장소가 부여됩니다. 따라서 해당 쓰레드만 해당 데이터에 접근할 수 있습니다
>

# 4. 트랜잭션 문제 해결 - 트랜잭션 매니저 1

### **트랜잭션 매니저**

애플리케이션 코드에 트랜잭션 매니저를 적용해 봅시다.

`MemberRepositoryV3`

```java
package hello.jdbc.repository;

import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.datasource.DataSourceUtils;
import org.springframework.jdbc.support.JdbcUtils;

import javax.sql.DataSource;
import java.sql.*;
import java.util.NoSuchElementException;

/**
 * 트랜잭션 - 트랜잭션 매니저
 * DataSourceUtils.getConnection()
 * DataSourceUtils.releaseConnection()
 */
@Slf4j
public class MemberRepositoryV3 {
    private final DataSource dataSource;

    public MemberRepositoryV3(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values(?,?)";

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

    public void update(String memberId, int money) throws SQLException {
        String sql = "update member set money=? where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    public void delete(String memberId) throws SQLException {

        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        // 주의 - 트랜잭션 동기화를 사용하려면 DataSourceUtils 을 사용해야 함.
        DataSourceUtils.releaseConnection(con, dataSource);
    }

    private Connection getConnection() throws SQLException {
        // 주의 - 트랜잭션 동기화를 사용하려면 DataSourceUtils 을 사용해야 함.
        Connection con = DataSourceUtils.getConnection(dataSource);
        log.info("get connection={} class={}", con, con.getClass());
        return con;
    }
}
```

커넥션을 파라미터로 전달하는 부분을 모두 제거했습니다.

**DataSourceUtils.getConnection()**

`getConnection()` 메서드에서 `DataSourceUtils.getConnection()`를 사용하도록 변경된 부분을 특히 주의하여 봅시다.

`DataSourceUtils.getConnection()`는 다음과 같이 동작합니다.

- 트랜잭션 동기화 매니저가 관리하는 커넥션이 있으면 해당 커넥션을 반환
- 트랜잭션 동기화 매니저가 관리하는 커넥션이 없는 경우 새로운 커넥션을 생성해서 반환

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3cc8519d-06c7-461a-83ac-0bc0dd479dfc/Untitled.png)

**DataSourceUtils.releaseConnection()**

`close()`에서 `DataSourceUtils.releaseConnection()`를 사용하도록 변경된 부분을 특히 주의해야 합니다. 커넥션을 `con.close()`를 사용해서 직접 닫아버리면 커넥션이 유지되지 않는 문제가 발생합니다. 

이 커넥션은 이후 로직은 물론이고, 트랜잭션을 종료(커밋, 롤백)할 때 까지 살아있어야 합니다.

`DataSourceUtils.releaseConnection()`을 사용하면 커넥션을 바로 닫지 않습니다.

- 트랜잭션을 사용하기 위해 동기화된 커넥션은 커넥션을 닫지 않고 그대로 유지
- 트랜잭션 동기화 매니저가 관리하는 커넥션이 없는 경우 해당 커넥션을 닫음

`MemberServiceV3_1`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV3;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import java.sql.SQLException;

@Slf4j
@RequiredArgsConstructor
public class MemberServiceV3_1 {
    private final PlatformTransactionManager transactionManager;
    private final MemberRepositoryV3 memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        // 트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            // 비즈니스 로직
            bizLogic(fromId, toId, money);
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw new IllegalStateException(e);
        }
    }

    private void bizLogic(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);
        memberRepository.update(fromId, fromMember.getMoney() - money);
        memberRepository.update(toId, toMember.getMoney() + money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체 중 예외 발생");
        }
    }
}
```

`private final PlatformTransactionManager transactionManager`

- 트랜잭션 매니저를 주입 받습니다. 지금은 JDBC 기술을 사용하기 때문에 `DataSourceTransactionManager` 구현체를 주입 받아야 합니다.
- 물론 JPA 같은 기술로 변경되면 `JpaTransactionManager`를 주입 받으면 됩니다.

`transactionManager.getTransaction()`

- 트랜잭션을 시작함.
- `TransactionStatus status`를 반환. 현재 트랜잭션의 상태 정보가 포함되어 있음. 이 트랜잭션 상태 정보는 이후에 트랜잭션을 커밋, 롤백할 때 필요함.

`new DefaultTransactionDefinition()`

- 트랜잭션과 관련된 옵션을 지정할 수 있음

`transactionManager.commit(status)`

- 트랜잭션이 성공하면 이 로직을 호출해서 커밋11

`transactionManager.rollback(status)`

- 문제가 발생하면 이 로직을 호출해서 트랜잭션을 롤백

`MemberServiceV3_1Test`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV3;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.transaction.PlatformTransactionManager;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

import java.sql.SQLException;

import static hello.jdbc.connection.ConnectionConst.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * 트랜잭션 - 트랜잭션 매니저
 */
class MemberServiceV3_1Test {
    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    private MemberRepositoryV3 memberRepository;
    private MemberServiceV3_1 memberService;

    @BeforeEach
    void setUp() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        memberRepository = new MemberRepositoryV3(dataSource);
        memberService = new MemberServiceV3_1(transactionManager, memberRepository);
    }

    @AfterEach
    void tearDown() throws SQLException {
        memberRepository.delete(MEMBER_A);
        memberRepository.delete(MEMBER_B);
        memberRepository.delete(MEMBER_EX);
    }

    @Test
    @DisplayName("정상 이해")
    void accountTransfer() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberB = new Member(MEMBER_B, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberB);

        //when
        memberService.accountTransfer(memberA.getMemberId(), memberB.getMemberId(), 2000);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberB.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberB.getMoney()).isEqualTo(12000);
    }

    @Test
    @DisplayName("이체 중 예외 발생")
    void accountTransferEx() throws SQLException {
        // given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        // when
        assertThatThrownBy(() -> memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 2000)).isInstanceOf(IllegalStateException.class);

        // then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberEx = memberRepository.findById(memberEx.getMemberId());

        // memberA 의 돈이 롤백되어야 함.
        assertThat(findMemberA.getMoney()).isEqualTo(10000);
        assertThat(findMemberEx.getMoney()).isEqualTo(10000);
    }
}
```

**초기화 코드 설명**

```java
@BeforeEach
void setUp() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
    PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
    memberRepository = new MemberRepositoryV3(dataSource);
    memberService = new MemberServiceV3_1(transactionManager, memberRepository);
}
```

`new DataSourceTransactionManager(dataSource)`

- JDBC 기술을 사용하므로, JDBC용 트랜잭션 매니저( `DataSourceTransactionManager`)를 선택해서 서비스에 주입함.
- 트랜잭션 매니저는 데이터소스를 통해 커넥션을 생성하므로 `DataSource`가 필요함.

# 5. 트랜잭션 문제 해결 - 트랜잭션 매니저 2

위에서 본 코드를 그림을 통해 정리해봅시다.

### **트랜잭션 매니저 1 - 트랜잭션 시작**

![https://user-images.githubusercontent.com/52024566/194569724-646726ce-e0e4-4f37-a9b7-e90b980f8e71.png](https://user-images.githubusercontent.com/52024566/194569724-646726ce-e0e4-4f37-a9b7-e90b980f8e71.png)

클라이언트의 요청으로 서비스 로직을 실행합니다.

 1. 서비스 계층에서 `transactionManager.getTransaction()`을 호출해서 트랜잭션을 시작합니다.

 2. 트랜잭션을 시작하려면 먼저 데이터베이스 커넥션이 필요합니다. 트랜잭션 매니저는 내부에서 데이터소스를 사용해서 커넥션을 생성합니다.

 3. 커넥션을 수동 커밋 모드로 변경해서 실제 데이터베이스 트랜잭션을 시작합니다.

 4. 커넥션을 트랜잭션 동기화 매니저에 보관합니다.

 5. 트랜잭션 동기화 매니저는 쓰레드 로컬에 커넥션을 보관합니다. 따라서 멀티 쓰레드 환경에 안전하게 커넥션을 보관할 수 있습니다.

### **트랜잭션 매니저2 - 로직 실행**

![https://user-images.githubusercontent.com/52024566/194569728-b55e1aae-267e-4a53-ad79-a1475f840901.png](https://user-images.githubusercontent.com/52024566/194569728-b55e1aae-267e-4a53-ad79-a1475f840901.png)

 6. 서비스는 비즈니스 로직을 실행하면서 리포지토리의 메서드들을 호출합니다. 이 때 커넥션을 파라미터로 전달하지 않습니다.

 7. 리포지토리 메서드들은 트랜잭션이 시작된 커넥션이 필요합니다. 리포지토리는 `DataSourceUtils.getConnection()`을 사용해서 트랜잭션 동기화 매니저에 보관된 커넥션을 꺼내서 사용합니다. 이 과정을 통해서 자연스럽게 같은 커넥션을 사용하고, 트랜잭션도 유지합니다.

 8. 획득한 커넥션을 사용해서 SQL을 데이터베이스에 전달해서 실행합니다.

### **트랜잭션 매니저3 - 트랜잭션 종료**

![https://user-images.githubusercontent.com/52024566/194569731-93a77cfd-6dd0-4370-a042-1f309f633f31.png](https://user-images.githubusercontent.com/52024566/194569731-93a77cfd-6dd0-4370-a042-1f309f633f31.png)

 9. 비즈니스 로직이 끝나고 트랜잭션을 종료합니다. 트랜잭션은 커밋하거나 롤백하면 종료됩니다.

 10. 트랜잭션을 종료하려면 동기화된 커넥션이 필요합니다. 트랜잭션 동기화 매니저를 통해 동기화된 커넥션을 획득할 수 있습니다.

 11. 획득한 커넥션을 통해 데이터베이스에 트랜잭션을 커밋하거나 롤백할 수 있습니다.

 12. 전체 리소스를 정리

- 트랜잭션 동기화 매니저를 정리합니다. 쓰레드 로컬은 사용후 꼭 정리해야 합니다.
- `con.setAutoCommit(true)`로 되돌립니다. 이는 커넥션 풀을 고려해야 한 조치입니다.
- `con.close()`를 호출해셔 커넥션을 종료합니다. 커넥션 풀을 사용하는 경우 `con.close()`를 호출하면 커넥션 풀에 반환됩니다.

### **정리**

트랜잭션 추상화 덕분에 서비스 코드는 이제 JDBC 기술에 의존하지 않습니다!

- 이후 JDBC에서 JPA로 변경해도 서비스 코드를 그대로 유지할 수 있습니다.
- 기술 변경시 의존관계 주입만 `DataSourceTransactionManager`에서 `JpaTransactionManager`로 변경해주면 됩니다.
- 트랜잭션 동기화 매니저 덕분에 커넥션을 파라미터로 넘기지 않아도 됩니다.

> 참고 -  여기서는 DataSourceTransactionManager의 동작 방식을 위주로 설명했다. 다른 트랜잭션 매니저는 해당 기술에 맞도록 변형되어서 동작한다.
>

# 6. 트랜잭션 문제 해결 - 트랜잭션 템플릿

트랜잭션을 사용하는 로직을 살펴보면 다음과 같은 패턴이 반복되는 것을 확인할 수 있습니다.

### **트랜잭션 사용 코드**

```java
//트랜잭션 시작
TransactionStatus status = transactionManager.getTransaction(new
DefaultTransactionDefinition());

try {
    //비즈니스 로직
    bizLogic(fromId, toId, money);
    transactionManager.commit(status); //성공시 커밋
} catch (Exception e) {
    transactionManager.rollback(status); //실패시 롤백
    throw new IllegalStateException(e);
}
```

- 트랜잭션을 시작하고, 비즈니스 로직을 실행하고, 성공하면 커밋하고, 예외가 발생해서 실패하면 롤백합니다.
- 다른 서비스에서 트랜잭션을 시작하려면 try , catch , finally 를 포함한 성공시 커밋, 실패시 롤백 코드가 반복됩니다.
- 이런 형태는 각각의 서비스에서 반복. 달라지는 부분은 비즈니스 로직 뿐입니다.
- 이럴 때 템플릿 콜백 패턴을 활용하면 이런 반복 문제를 깔끔하게 해결할 수 있습니다.

### **트랜잭션 템플릿**

템플릿 콜백 패턴을 적용하려면 템플릿을 제공하는 클래스를 작성해야 하는데, 스프링은 `TransactionTemplate`라는 템플릿 클래스를 제공합니다.

`TransactionTemplate`

```java
public class TransactionTemplate {
  
    private PlatformTransactionManager transactionManager;
  
    public <T> T execute(TransactionCallback<T> action){..}
    void executeWithoutResult(Consumer<TransactionStatus> action){..}
}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c50aa007-876f-4b1f-83f4-58ec313eba95/Untitled.png)

`execute()`: 응답 값이 있을 때 사용

`executeWithoutResult()`: 응답 값이 없을 때 사용

트랜잭션 템플릿을 사용해서 반복하는 부분을 제거하겠습니다.

`MemberServiceV3_2`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV3;
import lombok.extern.slf4j.Slf4j;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.support.TransactionTemplate;

import java.sql.SQLException;

/**
 * 트랜잭션 - 트랜잭션 템플릿
 */
@Slf4j
public class MemberServiceV3_2 {
    private final TransactionTemplate txTemplate;
    private final MemberRepositoryV3 memberRepository;

    public MemberServiceV3_2(PlatformTransactionManager transactionManager, MemberRepositoryV3 memberRepository) {
        this.txTemplate = new TransactionTemplate(transactionManager);
        this.memberRepository = memberRepository;
    }

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        txTemplate.executeWithoutResult((status) -> {
            try {
                // 비즈니스 로직
                bizLogic(fromId, toId, money);
            } catch (SQLException e) {
                throw new IllegalStateException(e);
            }
        });
    }

    private void bizLogic(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체 중 예외 발생");
        }
    }
}
```

`TransactionTemplate`을 사용하려면 `transactionManager`가 필요합니다. 

생성자에서 `transactionManager`를 주입 받으면서 `TransactionTemplate`을 생성하였습니다.

### **트랜잭션 템플릿 사용 로직**

```java
txTemplate.executeWithoutResult((status) -> {
    try {
        //비즈니스 로직
        bizLogic(fromId, toId, money);
    } catch (SQLException e) {
        throw new IllegalStateException(e);
    }
});
```

트랜잭션 템플릿 덕분에 트랜잭션을 시작하고, 커밋하거나 롤백하는 코드가 모두 제거되었습니다.

트랜잭션 템플릿의 기본 동작은 아래와 같습니다.

- 비즈니스 로직이 정상 수행되면 커밋
- 언체크 예외가 발생하면 롤백. 그 외의 경우 커밋

코드에서 예외를 처리하기 위해 `try~catch`가 들어갔는데, `bizLogic()` 메서드를 호출하면 `SQLException` 체크 예외를 넘겨줍니다. 해당 람다에서 체크 예외를 밖으로 던질 수 없기 때문에 언체크 예외로 바꾸어 던지도록 예외를 전환하였습니다.

`MemberServiceV3_2Test`

```java
package hello.jdbc.service;

import static org.junit.jupiter.api.Assertions.*;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV3;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.transaction.PlatformTransactionManager;

import java.sql.SQLException;

import static hello.jdbc.connection.ConnectionConst.*;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

/**
 * 트랜잭션 - 트랜잭션 템플릿
 */
class MemberServiceV3_2Test {

    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    private MemberRepositoryV3 memberRepository;
    private MemberServiceV3_2 memberService;

    @BeforeEach
    void setUp() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        memberRepository = new MemberRepositoryV3(dataSource);
        memberService = new MemberServiceV3_2(transactionManager, memberRepository);
    }

    @AfterEach
    void tearDown() throws SQLException {
        memberRepository.delete(MEMBER_A);
        memberRepository.delete(MEMBER_B);
        memberRepository.delete(MEMBER_EX);
    }

    @Test
    @DisplayName("정상 이체")
    void accountTransfer() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberB = new Member(MEMBER_B, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberB);

        //when
        memberService.accountTransfer(memberA.getMemberId(), memberB.getMemberId(), 2000);

        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberB.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberB.getMoney()).isEqualTo(12000);
    }

    @Test
    @DisplayName("이체중 예외 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        //when
        assertThatThrownBy(() -> memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 2000))
                .isInstanceOf(IllegalStateException.class);
        //then

        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberEx = memberRepository.findById(memberEx.getMemberId());

        //memberA의 돈이 롤백 되어야함
        assertThat(findMemberA.getMoney()).isEqualTo(10000);
        assertThat(findMemberEx.getMoney()).isEqualTo(10000);
    }
}
```

### **정리**

트랜잭션 템플릿 덕분에, 트랜잭션을 사용할 때 반복하는 코드를 제거할 수 있습니다.

하지만 이곳은 **서비스 로직인데 비즈니스 로직 뿐만 아니라 트랜잭션을 처리하는 기술 로직이 함께 포함되어 있습니다.**

애플리케이션을 구성하는 로직을 핵심 기능과 부가 기능으로 구분하자면 서비스 입장에서 비즈니스 로직은 핵심 기능이고, 트랜잭션은 부가 기능입니다.

이렇게 비즈니스 로직과 트랜잭션을 처리하는 기술 로직이 한 곳에 있으면 두 관심사를 하나의 클래스에서 처리하게 되어서 결과적으로 코드를 유지보수하기 어려워집니다.

그렇다면 위 문제도 해결해야겠죠. 우리는 이것을 AOP을 적용하여 해결할 수 있습니다. 

다음 글에서 AOP 기술에 대해 알아보며 코드를 변경해봅시다.

# 7. 트랜잭션 문제 해결 - 트랜잭션 AOP 이해

이전 글에서 트랜잭션을 편리하게 처리하기 위해서 트랜잭션 추상화도 도입하고, 추가로 반복적인 트랜잭션 로직을 해결하기 위해 트랜잭션 템플릿도 도입했습니다.

트랜잭션 템플릿 덕분에 트랜잭션을 처리하는 반복 코드는 해결할 수 있었으나 서비스 계층에 순수한 비즈니스 로직만 남긴다는 목표는 아직 달성하지 못했습니다.

**이럴 때 스프링 AOP를 통해 프록시를 도입하면 문제를 깔끔하게 해결할 수 있습니다.**

### **프록시를 통한 문제 해결**

**프록시 도입 전**

![https://user-images.githubusercontent.com/52024566/195048957-e9c88d2b-534b-4301-a348-5c72749ff91b.png](https://user-images.githubusercontent.com/52024566/195048957-e9c88d2b-534b-4301-a348-5c72749ff91b.png)

프록시를 도입하기 전에는 기존처럼 서비스의 로직에서 트랜잭션을 직접 시작했습니다.

**서비스 계층의 트랜잭션 사용 코드 예시**

```java
//트랜잭션 시작
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

try {
    //비즈니스 로직
    bizLogic(fromId, toId, money);
    transactionManager.commit(status); //성공시 커밋
} catch (Exception e) {
    transactionManager.rollback(status); //실패시 롤백
    throw new IllegalStateException(e);
}
```

**프록시 도입 후**

![https://user-images.githubusercontent.com/52024566/195048961-4142fff5-6032-4c09-8962-552a6cd7fd86.png](https://user-images.githubusercontent.com/52024566/195048961-4142fff5-6032-4c09-8962-552a6cd7fd86.png)

프록시를 사용하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있습니다.

**트랜잭션 프록시 코드 예시**

```java
public class TransactionProxy {
  
    private MemberService target;
  
    public void logic() {
        //트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(..);
        try {
            //실제 대상 호출
            target.logic();
            transactionManager.commit(status); //성공시 커밋
        } catch (Exception e) {
            transactionManager.rollback(status); //실패시 롤백
            throw new IllegalStateException(e);
        }
    }
}
```

**트랜잭션 프록시 적용 후 서비스 코드 예시**

```java
public class Service {
    public void logic() {
          //트랜잭션 관련 코드 제거, 순수 비즈니스 로직만 남음
        bizLogic(fromId, toId, money);
    }
}
```

프록시 도입 전: 서비스에 비즈니스 로직과 트랜잭션 처리 로직이 함께 섞여있었습니다.

프록시 도입 후: 트랜잭션 프록시가 트랜잭션 처리 로직을 모두 가져갑니다. 그리고 트랜잭션을 시작한 후에 실제 서비스를 대신 호출합니다. 트랜잭션 프록시 덕분에 서비스 계층에는 순수한 비즈니즈 로직만 남길 수 있습니다.

### **스프링이 제공하는 트랜잭션 AOP**

스프링이 제공하는 AOP 기능을 사용하면 프록시를 매우 편리하게 적용할 수 있습니다.

물론 스프링 AOP를 직접 사용해서 트랜잭션을 처리해도 되지만, 트랜잭션은 매우 중요한 기능이고, 전세계 누구나 다 사용하는 기능입니다. 그래서 스프링은 트랜잭션 AOP를 처리하기 위한 모든 기능을 제공합니다. 

스프링 부트를 사용하면 트랜잭션 AOP를 처리하기 위해 필요한 스프링 빈들도 자동으로 등록할 수 있습니다.

개발자가 트랜잭션 처리가 필요한 곳에 `@Transactional` 애노테이션만 붙여주면 스프링의 트랜잭션 AOP는 이 애노테이션을 인식해서 트랜잭션 프록시를 적용합니다!

**@Transactional** - `org.springframework.transaction.annotation.Transactional`

> 참고 - 스프링 AOP를 적용하려면 어드바이저, 포인트컷, 어드바이스가 필요합니다. 스프링은 트랜잭션 AOP 처리를 위해 다음 클래스를 제공합니다. 스프링 부트를 사용하면 해당 빈들은 스프링 컨테이너에 자동으로 등록더ㅣㅂ니다.
> 
> 
> 어드바이저: `BeanFactoryTransactionAttributeSourceAdvisor` 
> 포인트컷: `TransactionAttributeSourcePointcut` 
> 어드바이스: `TransactionInterceptor`
>


# 8. 트랜잭션 문제 해결 - 트랜잭션 AOP 적용

그렇다면 이제 코드를 통해서 실제로 AOP 을 적용해봅시다.

`MemberServiceV3_3`

```java
package hello.jdbc.service;

import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV3;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.transaction.annotation.Transactional;

import java.sql.SQLException;

@Slf4j
@RequiredArgsConstructor
public class MemberServiceV3_3 {
    private final MemberRepositoryV3 memberRepository;

    @Transactional
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        bizLogic(fromId, toId, money);
    }
    private void bizLogic(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);
        memberRepository.update(fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체 중 예외 발생");
        }
    }
}
```

순수한 비즈니스 로직만 남기고 트랜잭션 관련 코드는 모두 제거했습니다.

스프링이 제공하는 트랜잭션 AOP를 적용하기 위해 `@Transactional` 애노테이션을 추가하면 됩니다.

`@Transactional` 애노테이션은 메서드에 붙여도 되고, 클래스에 붙여도 됩니다. 클래스에 붙이면 외부에서 호출 가능한 `public` 메서드가 AOP 적용 대상이 됩니다.

`MemberServiceV3_3Test`

```java
import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV3;
import lombok.extern.slf4j.Slf4j;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.aop.support.AopUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;
import java.sql.SQLException;

import static hello.jdbc.connection.ConnectionConst.*;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

/**
* 트랜잭션 - @Transactional AOP
*/
@Slf4j
@SpringBootTest
class MemberServiceV3_3Test {
  
    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";
  
    @Autowired
    MemberRepositoryV3 memberRepository;
    @Autowired
    MemberServiceV3_3 memberService;
  
    @AfterEach
    void after() throws SQLException {
        memberRepository.delete(MEMBER_A);
        memberRepository.delete(MEMBER_B);
        memberRepository.delete(MEMBER_EX);
    }
  
    @TestConfiguration
    static class TestConfig {
        @Bean
        DataSource dataSource() {
            return new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        }
        @Bean
        PlatformTransactionManager transactionManager() {
            return new DataSourceTransactionManager(dataSource());
        }
        @Bean
        MemberRepositoryV3 memberRepositoryV3() {
            return new MemberRepositoryV3(dataSource());
        }
        @Bean
        MemberServiceV3_3 memberServiceV3_3() {
            return new MemberServiceV3_3(memberRepositoryV3());
        }
    }
  
    @Test
    void AopCheck() {
    log.info("memberService class={}", memberService.getClass());
    log.info("memberRepository class={}", memberRepository.getClass());
      Assertions.assertThat(AopUtils.isAopProxy(memberService)).isTrue();
      Assertions.assertThat(AopUtils.isAopProxy(memberRepository)).isFalse();
    }
  
    @Test
    @DisplayName("정상 이체")
    void accountTransfer() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberB = new Member(MEMBER_B, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberB);
      
        //when
        memberService.accountTransfer(memberA.getMemberId(), memberB.getMemberId(), 2000);
      
        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberB.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(8000);
        assertThat(findMemberB.getMoney()).isEqualTo(12000);
    }
  
    @Test
    @DisplayName("이체중 예외 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);
      
        //when
        assertThatThrownBy(() ->
        memberService.accountTransfer(memberA.getMemberId(), memberEx.getMemberId(), 2000))
          .isInstanceOf(IllegalStateException.class);
      
        //then
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberEx = memberRepository.findById(memberEx.getMemberId());
      
        //memberA의 돈이 롤백 되어야함
        assertThat(findMemberA.getMoney()).isEqualTo(10000);
        assertThat(findMemberEx.getMoney()).isEqualTo(10000);
    }
}
```

`@SpringBootTest`: 스프링 AOP를 적용하려면 스프링 컨테이너가 필요합니다. 이 애노테이션이 있으면 테스트시 스프링 부트를 통해 스프링 컨테이너를 생성합니다. 그리고 테스트에서 `@Autowired`등을 통해 스프링 컨테이너가 관리하는 빈들을 사용할 수 있습니다.

`@TestConfiguration`: 테스트 안에서 내부 설정 클래스를 만들어서 사용하면서 이 에노테이션을 붙이면 스프링 부트가 자동으로 만들어주는 빈들에 추가로 필요한 스프링 빈들을 등록하고 테스트를 수행할 수 있습니다.

`TestConfig` 클래스

- `DataSource`: 스프링에서 기본으로 사용할 데이터소스를 스프링 빈으로 등록합니다. 추가로 트랜잭션 매니저에서도 데이터소스를 사용합니다.
- `DataSourceTransactionManager`: 트랜잭션 매니저를 스프링 빈으로 등록합니다.
    - 스프링이 제공하는 트랜잭션 AOP는 스프링 빈에 등록된 트랜잭션 매니저를 찾아서 사용하기 때문에 트랜잭션 매니저를 스프링 빈으로 등록해두어야 합니다.

**AOP 프록시 적용 확인**

```java
@Test
void AopCheck() {
    log.info("memberService class={}", memberService.getClass());
    log.info("memberRepository class={}", memberRepository.getClass());
    Assertions.assertThat(AopUtils.isAopProxy(memberService)).isTrue();
    Assertions.assertThat(AopUtils.isAopProxy(memberRepository)).isFalse();
}
```

실행 결과 - `AopCheck()`

```java
memberService class=class hello.jdbc.service.MemberServiceV3_3$
$EnhancerBySpringCGLIB$$...
memberRepository class=class hello.jdbc.repository.MemberRepositoryV3
```

`AopCheck()`의 실행 결과를 보면 `memberService`에 `EnhancerBySpringCGLIB..`라는 부분을 통해 프록시(CGLIB)가 적용된 것을 확인할 수 있습니다. 

`memberRepository`에는 AOP를 적용하지 않았기 때문에 프록시가 적용되지 않은 것을 확인 할 수 있습니다.

나머지 테스트 코드들을 실행해보면 트랜잭션이 정상 수행되고, 실패시 정상 롤백된 것을 확인할 수 있습니다.

# 9. 트랜잭션 문제 해결 - 트랜잭션 AOP 정리

### **트랜잭션 AOP 적용 전체 흐름**

![https://user-images.githubusercontent.com/52024566/195048963-9a1a940d-b8cd-4672-a635-8e0dcd342172.png](https://user-images.githubusercontent.com/52024566/195048963-9a1a940d-b8cd-4672-a635-8e0dcd342172.png)

### **선언적 트랜잭션 관리 vs 프로그래밍 방식 트랜잭션 관리**

**선언적 트랜잭션 관리(Declarative Transaction Management)**

- `@Transactional` 애노테이션 하나만 선언해서 매우 편리하게 트랜잭션을 적용하는 것을 선언적 트랜잭션 관리라 합니다.
- 선언적 트랜잭션 관리는 과거에는 XML에 설정하기도 했습니다. 이름 그대로 “해당 로직에 트랜잭션을 적용하겠다” 라고 어딘가에 선언하기만 하면 트랜잭션이 적용되는 방식입니다.

**프로그래밍 방식의 트랜잭션 관리(programmatic transaction management)**

- 트랜잭션 매니저 또는 트랜잭션 템플릿 등을 사용해서 트랜잭션 관련 코드를 직접 작성하는 것을 프로그래밍 방식의 트랜잭션 관리라고 합니다.
- 선언적 트랜잭션 관리가 프로그래밍 방식에 비해서 훨씬 간편하고 실용적이기 때문에 실무에서는 대부분 선언적 트랜잭션 관리를 사용합니다.
- 프로그래밍 방식의 트랜잭션 관리는 스프링 컨테이너나 스프링 AOP 기술 없이 간단히 사용할 수 있지만 실무에서는 대부분 스프링 컨테이너와 스프링 AOP를 사용하기 때문에 거의 사용되지 않습니다.
- 프로그래밍 방식 트랜잭션 관리는 테스트 시에 가끔 사용될 때는 있습니다.

### **정리**

스프링이 제공하는 선언적 트랜잭션 관리 덕분에 드디어 트랜잭션 관련 코드를 순수한 비즈니스 로직에서 제거할 수 있습니다.

결론적으로 개발자는 트랜잭션이 필요한 곳에 `@Transactional` 애노테이션 하나만 추가하면 됩니다!! 나머지는 스프링 트랜잭션 AOP가 자동으로 처리해줍니다.

# 10. 스프링 부트의 자동 리소스 등록

### **데이터소스와 트랜잭션 매니저를 스프링 빈으로 직접 등록**

```java
@Bean
DataSource dataSource() {
    return new DriverManagerDataSource(URL, USERNAME, PASSWORD);
}

@Bean
PlatformTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSource());
}
```

우리는 위에 8, 9 챕터에서 이렇게 데이터소스와 트랜잭션 매니저를 직접 스프링 빈으로 등록해야 했으나 스프링 부트가 나오면서 여기서도 많은 부분이 자동화할 수 있게 되었습니다. (더 오래전에 스프링을 다루어왔다면 해당 부분을 주로 XML로 등록하고 관리)

### **데이터소스 - 자동 등록**

스프링 부트는 데이터소스(`DataSource`)를 스프링 빈에 자동으로 등록합니다.

- 자동으로 등록되는 스프링 빈 이름: `dataSource`

참고로 개발자가 직접 데이터소스를 빈으로 등록하면 스프링 부트는 데이터소스를 자동으로 등록하지 않습니다.

스프링 부트는 다음과 같이 `application.properties`에 있는 속성을 사용해서 `DataSource`를 생성하고 스프링 빈에 등록합니다.

`application.properties`

```java
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
spring.datasource.password=
```

스프링 부트가 기본으로 생성하는 데이터소스는 커넥션풀을 제공하는 `HikariDataSource` 커넥션풀과 관련된 설정도 `application.properties`를 통해서 지정할 수 있습니다.

만약 `spring.datasource.url` 속성이 없으면 스프링부트는 내장 데이터베이스(메모리 DB)를 생성을 시도합니다.

### **트랜잭션 매니저 - 자동 등록**

스프링 부트는 적절한 트랜잭션 매니저(`PlatformTransactionManager`)를 자동으로 스프링 빈에 등록합니다.

- 자동으로 등록되는 스프링 빈 이름: `transactionManager`

참고로 개발자가 직접 트랜잭션 매니저를 빈으로 등록하면 스프링 부트는 트랜잭션 매니저를 자동으로 등록하지 않습니다. (데이터소스 자동 등록과 같죠!)

어떤 트랜잭션 매니저를 선택할지는 현재 등록된 라이브러리를 보고 판단합니다. 

만약 JDBC를 사용하면 `DataSourceTransactionManager`를 빈으로 등록하고, JPA를 사용하면 `JpaTransactionManager`를 빈으로 등록하는 것이지요. 

만약에 둘다 사용하는 경우 `JpaTransactionManager`를 등록합니다. 

참고로 `JpaTransactionManager`는 `DataSourceTransactionManager`가 제공하는 기능도 대부분 지원합니다!

**데이터소스, 트랜잭션 매니저 직접 등록**

```java
@TestConfiguration
static class TestConfig {
    @Bean
    DataSource dataSource() {
        return new DriverManagerDataSource(URL, USERNAME, PASSWORD);
    }
  
    @Bean
    PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
  
    @Bean
    MemberRepositoryV3 memberRepositoryV3() {
        return new MemberRepositoryV3(dataSource());
    }
  
    @Bean
    MemberServiceV3_3 memberServiceV3_3() {
        return new MemberServiceV3_3(memberRepositoryV3());
    }
}
```

- 이렇게 데이터소스와 트랜잭션 매니저를 직접 등록하면 스프링 부트는 데이터소스와 트랜잭션 매니저를 자동으로 등록하지 않음

## **데이터소스와 트랜잭션 매니저 자동 등록**

`application.properties`

```java
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
spring.datasource.password=
```

`MemberServiceV3_4Test`

```java
/**
* 트랜잭션 - DataSource, transactionManager 자동 등록
*/
@Slf4j
@SpringBootTest
class MemberServiceV3_4Test {
		public static final String MEMBER_A = "memberA";
		...
  
    @TestConfiguration
    static class TestConfig {
      
        private final DataSource dataSource;
      
        public TestConfig(DataSource dataSource) {
            this.dataSource = dataSource;
        }
      
        @Bean
        MemberRepositoryV3 memberRepositoryV3() {
            return new MemberRepositoryV3(dataSource);
        }
      
        @Bean
        MemberServiceV3_3 memberServiceV3_3() {
            return new MemberServiceV3_3(memberRepositoryV3());
        }
    }
    ...
}
```

기존(`MemberServiceV3_3Test`)과 같은 코드이고 `TestConfig` 부분만 다릅니다.

데이터소스와 트랜잭션 매니저를 스프링 빈으로 등록하는 코드가 생략되어 있습니다. 따라서 스프링 부트가 `application.properties`에 지정된 속성을 참고해서 데이터소스와 트랜잭션 매니저를 자동으로 생성합니다

코드에서 보는 것 처럼 생성자를 통해서 스프링 부트가 만들어준 데이터소스 빈을 주입 받을 수도 있습니다.

### **정리**

데이터소스와 트랜잭션 매니저는 스프링 부트가 제공하는 자동 빈 등록 기능을 사용하는 것이 편리합니다.

추가로 `application.properties`를 통해 설정도 편리하게 할 수 있습니다.

> 스프링 부트의 데이터소스 자동 등록에 대한 더 자세한 내용은 다음 스프링 부트 공식 메뉴얼을 참고합시다.
https://docs.spring.io/spring-boot/docs/current/reference/html/data.html#data.sql.datasource.production
> 
> 
> 자세한 설정 속성은 다음을 참고합시다. 
> https://docs.spring.io/spring-boot/docs/current/reference/html/applicationproperties.html
>

# === 5. 자바 예외 이해 ===

# 1. 예외 계층

### **예외 계층 그림**

![https://user-images.githubusercontent.com/52024566/195851243-59d311da-a993-4a1c-a17f-f6a8adf8aff8.png](https://user-images.githubusercontent.com/52024566/195851243-59d311da-a993-4a1c-a17f-f6a8adf8aff8.png)

- `Object`: 예외도 객체입니다. 모든 객체의 최상위 부모는 `Object`이므로 예외의 최상위 부모도 `Object` 입니다.
- `Throwable`: 최상위 예외 타입입니다. 하위에 `Exception`과 `Error`가 있습니다.
- `Error`: 메모리 부족이나 심각한 시스템 오류와 같이 애플리케이션에서 복구 불가능한 시스템 예외입니다. 애플리케이션 개발자는 이 예외를 잡으려고 해서는 안 되고 잡을 수도 없을 것입니다.
    - 상위 예외를 `catch`로 잡으면 그 하위 예외까지 함께 잡습니다. 따라서 애플리케이션 로직에서는 `Throwable` 예외도 잡으면 안되는데, 앞서 이야기한 `Error` 예외도 함께 잡을 수 있기 때문입니다. 애플리케이션 로직은 이런 이유로 `Exception`부터 필요한 예외로 생각하고 잡으면 됩니다.
    - 참고로 `Error`도 언체크 예외입니다.

- `Exception`: 체크 예외
    - 애플리케이션 로직에서 사용할 수 있는 실질적인 최상위 예외입니다.
    - `Exception`과 그 하위 예외는 모두 컴파일러가 체크하는 체크 예외입니다. 단 `RuntimeException`은 컴파일러가 체크하는 예외가 아닙니다.

- `RuntimeException`: 언체크 예외, 런타임 예외
    - 컴파일러가 체크 하지 않는 언체크 예외입니다.
    - `RuntimeException`과 그 자식 예외는 모두 언체크 예외입니다.
    - `RuntimeException`의 이름을 따라서 `RuntimeException`과 그 하위 언체크 예외를 런타임 예외라고 합니다.


# 2. 예외 기본 규칙

예외는 폭탄 돌리기라고 생각하면 됩니다!! 잡아서 처리하거나, 처리할 수 없으면 밖으로 던져야 합니다.

### **예외 처리**

![https://user-images.githubusercontent.com/52024566/195851248-303e30e8-fc71-43f9-9264-ffad354d802b.png](https://user-images.githubusercontent.com/52024566/195851248-303e30e8-fc71-43f9-9264-ffad354d802b.png)

- 5번에서 예외를 처리하면 이후에는 애플리케이션 로직이 정상 흐름으로 동작합니다.

### **예외 던짐**

![https://user-images.githubusercontent.com/52024566/195851253-8668f9d1-8750-4965-818a-a8da4750120a.png](https://user-images.githubusercontent.com/52024566/195851253-8668f9d1-8750-4965-818a-a8da4750120a.png)

예외를 처리하지 못하면 호출한 곳으로 예외를 계속 던집니다.

**예외의 2가지 기본 규칙**

 1. 예외는 잡아서 처리하거나 던져야 합니다.

 2. 예외를 잡거나 던질 때 지정한 예외 뿐만 아니라 그 예외의 자식들도 함께 처리됩니다.

- 예를 들어서 `Exception`을 `catch`로 잡으면 그 하위 예외들도 모두 잡을 수 있음
- 예를 들어서 `Exception`을 `throws`로 던지면 그 하위 예외들도 모두 던질 수 있음

> **참고: 예외를 처리하지 못하고 계속 던지면 어떻게 될까요?**
> 
- 자바 `main()` 쓰레드의 경우 예외 로그를 출력하면서 시스템이 종료됩니다.
- 웹 애플리케이션의 경우 여러 사용자의 요청을 처리하기 때문에 하나의 예외 때문에 당연히 시스템이 종료되면 안 됩니다 . WAS가 해당 예외를 받아서 처리하는데, 주로 사용자에게 개발자가 지정한 오류 페이지를 보여줍니다.

# 3. 체크 예외 기본 이해

`Exception`과 그 하위 예외는 모두 컴파일러가 체크하는 체크 예외입니다. 단 `RuntimeException`은 체크 예외가 아닙니다.

체크 예외는 잡아서 처리하거나, 또는 밖으로 던지도록 선언해야 합니다. 그렇지 않으면 컴파일 오류가 발생합니다.

### **체크 예외 전체 코드**

```java
package hello.jdbc.exception;

import lombok.extern.slf4j.Slf4j;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.stereotype.Service;

import static org.assertj.core.api.Assertions.*;

@Slf4j
public class CheckedTest {
    @Test
    void checked_catch() {
        Service service = new Service();
        service.callCatch();
    }

    @Test
    void checked_throw() {
        Service service = new Service();
        assertThatThrownBy(() -> service.callThrow())
                .isInstanceOf(MyCheckedException.class);
    }

    /**
     * Exception 을 상속받은 예외는 체크 예외가 된다.
     */
    static class MyCheckedException extends Exception {
        public MyCheckedException(String message) {
            super(message);
        }
    }

    /**
     * Checked 예외는
     * 예외를 잡아서 처리하거나, 던지거나 둘중 하나를 필수로 선택해야 한다.
     */
    static class Service {
        Repository repository = new Repository();

        /**
         * 예외를 잡아서 처리하는 코드
         */
        public void callCatch() {
            try {
                repository.call();
            } catch (MyCheckedException e) {
                //예외 처리 로직
                log.info("예외 처리, message={}", e.getMessage(), e);
            }
        }

        /**
         * 체크 예외를 밖으로 던지는 코드
         * 체크 예외는 예외를 잡지 않고 밖으로 던지려면 throws 예외를 메서드에 필수로
         * 선언해야한다.
         */
        public void callThrow() throws MyCheckedException {
            repository.call();
        }
    }

    static class Repository {
        public void call() throws MyCheckedException {
            throw new MyCheckedException("ex");
        }
    }
}
```

**Exception을 상속받은 예외는 체크 예외**

```java
static class MyCheckedException extends Exception {
		public MyCheckedException(String message) {
				super(message);
		}
}
```

`MyCheckedException`는 `Exception`을 상속받습니다. `Exception`을 상속받으면 체크 예외가 됩니다.

참고로 `RuntimeException`을 상속받으면 언체크 예외가 됩니다. 이런 규칙은 자바 언어에서 문법으로 정해두고 있습니다.

예외가 제공하는 여러가지 기본 기능이 있는데, 그 중에 오류 메시지를 보관하는 기능도 있습니다. 예제에서 보는 것처럼 생성자를 통해서 해당 기능을 그대로 사용하면 편리합니다.

```java
@Test
void checked_catch() {
    Service service = new Service();
    service.callCatch();
}
```

`service.callCatch()`에서 예외를 처리했기 때문에 테스트 메서드까지 예외가 올라오지 않습니다.

실행 순서 분석

1. `test` → `service.callCatch()` → `repository.call()` [예외 발생, 던짐]
2. `test` ← `service.callCatch()` [예외 처리] ← `repository.call()`
3. `test` [정상 흐름] ← `service.callCatch()` ← `repository.call()`

`Repository.call()`에서 `MyUncheckedException` 예외가 발생하고, 그 예외를 `Service.callCatch()`에서 잡는 것을 확인할 수 있음

`log.info("예외 처리, message={}", e.getMessage(), e);`

**실행 결과**

```java
[Test worker] INFO hello.jdbc.exception.CheckedTest - 예외 처리,message=ex
hello.jdbc.exception.UncheckedTest$MyUncheckedException: ex
at hello.jdbc.exception.UncheckedTest$Repository.call(CheckedTest.java:65)
at hello.jdbc.exception.UncheckedTest$Service.callCatch(CheckedTest.java:46)
at hello.jdbc.exception.UncheckedTest.unchecked_catch(CheckedTest.java:15)
```

실행 결과 로그를 보면 첫줄은 우리가 남긴 로그가 그대로 남는 것을 확인할 수 있습니다.

그런데 두 번째 줄 부터 예외에 대한 스택 트레이스가 추가로 출력됩니다.

이 부분은 로그를 남길 때 로그의 마지막 인수에 예외 객체를 전달해주면 로그가 해당 예외의 스택 트레이스를 추가로 출력합니다.

`log.info("예외 처리, message={}", e.getMessage(), e);` ← 여기서 마지막에 있는 `e` 가 마지막 인수임.

**체크 예외를 잡아서 처리하는 코드**

```java
try {
    repository.call();
} catch (MyCheckedException e) {
    //예외 처리 로직
}
```

- 체크 예외를 잡아서 처리하려면 `catch(..)`를 사용해서 예외를 잡음
- 여기서는 `MyCheckedException` 예외를 잡아서 처리

**catch는 해당 타입과 그 하위 타입을 모두 잡을 수 있다**

```java
public void callCatch() {
    try {
        repository.call();
    } catch (Exception e) {
        //예외 처리 로직
    }
}
```

`catch`에 `MyCheckedException`의 상위 타입인 `Exception`을 적어주어도 `MyCheckedException`을 잡을 수 있습니다.

`catch`에 예외를 지정하면 해당 예외와 그 하위 타입 예외를 모두 잡아줍니다.

물론 정확하게 `MyCheckedException`만 잡고 싶다면 `catch`에 `MyCheckedException`을 적어주어야 하겠죠.

**예외를 처리하지 않고 밖으로 던지는 코드**

```java
@Test
void checked_throw() {
    Service service = new Service();
    assertThatThrownBy(() -> service.callThrow())
      .isInstanceOf(MyCheckedException.class);
}
```

`service.callThrow()`에서 예외를 처리하지 않고, 밖으로 던졌기 때문에 예외가 테스트 메서드까지 올라옵니다.

테스트에서는 기대한 것처럼 `MyCheckedException` 예외가 던져지면 성공으로 처리됩니다.

실행 순서 분석

1. `test` → `service.callThrow()` → `repository.call()` [예외 발생, 던짐]
2. `test` ← `service.callThrow()` [예외 던짐] ← `repository.call()`
3. `test` [예외 도착] ← `service.callThrow()` ← `repository.call()`

**체크 예외를 밖으로 던지는 코드**

```java
public void callThrow() throws MyCheckedException {
    repository.call();
}
```

체크 예외를 처리할 수 없을 때는 `method() throws 예외`을 사용해서 밖으로 던질 예외를 필수로 지정해주어야 합니다. 여기서는 `MyCheckedException`을 밖으로 던지도록 지정하였습니다.

**체크 예외를 밖으로 던지지 않으면 컴파일 오류 발생**

```java
public void callThrow() {
    repository.call();
}
```

`throws`를 지정하지 않으면 컴파일 오류가 발생합니다.

```java
Unhandled exception: hello.jdbc.exception.basic.CheckedTest.MyCheckedException
```

체크 예외의 경우 예외를 잡아서 처리하거나 또는 `throws`를 지정해서 예외를 밖으로 던진다는 선언을 필수로 해주어야 합니다.

**체크 예외를 밖으로 던지는 경우에도 해당 타입과 그 하위 타입을 모두 던질 수 있다**

```java
public void callThrow() throws Exception {
    repository.call();
}
```

`throws`에 `MyCheckedException`의 상위 타입인 `Exception`을 적어주어도 `MyCheckedException`을 던질 수 있습니다.

`throws`에 지정한 타입과 그 하위 타입 예외를 밖으로 던집니다.

물론 정확하게 `MyCheckedException`만 밖으로 던지고 싶다면 `throws`에 `MyCheckedException`을 적어주어야 합니다.

### **체크 예외의 장단점**

체크 예외는 예외를 잡아서 처리할 수 없을 때, 예외를 밖으로 던지는 throws 예외 를 필수로 선언해야 합니다. 그렇지 않으면 컴파일 오류가 발생합니다. 이것 때문에 장점과 단점이 동시에 존재합니다.

- 장점: 개발자가 실수로 예외를 누락하지 않도록 컴파일러를 통해 문제를 잡아주는 훌륭한 안전 장치가 됨.
- 단점: 하지만 실제로는 개발자가 모든 체크 예외를 반드시 잡거나 던지도록 처리해야 하기 때문에, 너무 번거로운 일이 됨. 크게 신경쓰고 싶지 않은 예외까지 모두 챙겨야 함. 추가로 의존관계에 따른 단점도 있음.

# 4. 언체크 예외 기본 이해

`RuntimeException`과 그 하위 예외는 언체크 예외로 분류됩니다.

언체크 예외는 말 그대로 컴파일러가 예외를 체크하지 않는다는 뜻입니다.

언체크 예외는 체크 예외와 기본적으로 동일하나 차이가 있다면 예외를 던지는 `throws`를 선언하지 않고 생략할 수 있습니다. 이 경우 자동으로 예외를 던집니다.

### **체크 예외 VS 언체크 예외**

- 체크 예외: 예외를 잡아서 처리하지 않으면 항상 `throws`에 던지는 예외를 선언해야 합니다.
- 언체크 예외: 예외를 잡아서 처리하지 않아도 `throws`를 생략할 수 있습니다.

### **언체크 예외 전체 코드**

```java
package hello.jdbc.exception;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThatThrownBy;

@Slf4j
public class UncheckedTest {

    @Test
    void unchecked_catch() {
        Service service = new Service();
        service.callCatch();
    }

    @Test
    void unchecked_throw() {
        Service service = new Service();
        assertThatThrownBy(() -> service.callThrow())
                .isInstanceOf(MyUncheckedException.class);
    }

    /**
     * RuntimeException을 상속받은 예외는 언체크 예외가 된다.
     */
    static class MyUncheckedException extends RuntimeException {
        public MyUncheckedException(String message) {
            super(message);
        }
    }

    /**
     * UnChecked 예외는
     * 예외를 잡거나, 던지지 않아도 된다.
     * 예외를 잡지 않으면 자동으로 밖으로 던진다.
     */
    static class Service {

        Repository repository = new Repository();

        /**
         * 필요한 경우 예외를 잡아서 처리하면 된다.
         */
        public void callCatch() {
            try {
                repository.call();
            } catch (MyUncheckedException e) {
                //예외 처리 로직
                log.info("예외 처리, message={}", e.getMessage(), e);
            }
        }

        /**
         * 예외를 잡지 않아도 된다. 자연스럽게 상위로 넘어간다.
         * 체크 예외와 다르게 throws 예외 선언을 하지 않아도 된다.
         */
        public void callThrow() {
            repository.call();
        }
    }

    static class Repository {
        public void call() {
            throw new MyUncheckedException("ex");
        }
    }
}
```

**언체크 예외를 잡아서 처리하는 코드**

```java
try {
    repository.call();
} catch (MyUncheckedException e) {
    //예외 처리 로직
    log.info("error", e);
}
```

- 언체크 예외도 필요한 경우 이렇게 잡아서 처리할 수 있습니다.

**언체크 예외를 밖으로 던지는 코드 - 생략**

```java
public void callThrow() {
    repository.call();
}
```

- 언체크 예외는 체크 예외와 다르게 `throws 예외`를 선언하지 않아도 됩니다.
- 말 그대로 컴파일러가 이런 부분을 체크하지 않기 때문에 언체크 예외인 것입니다.

**언체크 예외를 밖으로 던지는 코드 - 선언**

```java
public void callThrow() throws MyUncheckedException {
    repository.call();
}
```

- 참고로 언체크 예외도 `throws 예외`를 선언해도 됩니다. 물론 생략할 수 있습니다.
- 언체크 예외는 주로 생략하지만, 중요한 예외의 경우 이렇게 선언해두면 해당 코드를 호출하는 개발자가 이런 예외가 발생한다는 점을 IDE를 통해 좀 더 편리하게 인지할 수 있습니다. (컴파일 시점에 막을 수 있는 것은 아니고, IDE를 통해서 인지할 수 있는 정도)

### **언체크 예외의 장단점**

언체크 예외는 예외를 잡아서 처리할 수 없을 때, 예외를 밖으로 던지는 `throws 예외`를 생략할 수 있습니다. 이것 때문에 장점과 단점이 동시에 존재합니다.

- 장점: 신경쓰고 싶지 않은 언체크 예외를 무시할 수 있습니다. 체크 예외의 경우 처리할 수 없는 예외를 밖으로 던지려면 항상 `throws 예외`를 선언해야 하지만, 언체크 예외는 이 부분을 생략할 수 있습니다. 신경쓰고 싶지 않은 예외의 의존관계를 참조하지 않아도 되는 장점이 있습니다.
- 단점: 언체크 예외는 개발자가 실수로 예외를 누락할 수 있습니다. 반면에 체크 예외는 컴파일러를 통해 예외 누락을 잡아줍니다.

### **정리**

체크 예외와 언체크 예외의 차이는 예외를 처리할 수 없을 때 예외를 밖으로 던지는 부분에 있습니다. 이 부분을 필수로 선언해야 하는가 생략할 수 있는가의 차이가 있습니다.

# 5. 체크 예외 활용

### **예외 사용시 기본 원칙**

기본적으로 언체크(런타임) 예외를 사용합니다.

체크 예외는 비즈니스 로직상 의도적으로 던지는 예외에만 사용합니다.

- 해당 예외를 잡아서 반드시 처리해야 하는 문제일 때만 체크 예외를 사용해야 합니다.
- 체크 예외 예)
    - 계좌 이체 실패 예외
    - 결제시 포인트 부족 예외
    - 로그인 ID, PW 불일치 예외
    - 물론 이 경우에도 100% 체크 예외로 만들어야 하는 것은 아닙니다. 다만 계좌 이체 실패처럼 매우 심각한 문제는 개발자가 실수로 예외를 놓치면 안된다고 판단할 수 있으므로 체크 예외로 만들어 두면 컴파일러를 통해 놓친 예외를 인지할 수 있습니다.

### **체크 예외의 문제점**

위에서 알아보았듯이 체크 예외는 컴파일러가 예외 누락을 체크해주기 때문에 개발자가 실수로 예외를 놓치는 것을 막아줍니다. 그래서 항상 명시적으로 예외를 잡아서 처리하거나, 처리할 수 없을 때는 예외를 던지도록 `method() throws 예외`로 선언해야 합니다.

**체크 예외 문제점 - 그림**

![https://user-images.githubusercontent.com/52024566/196717600-19b9c7d8-67d9-40dd-b0e1-4aa985767346.png](https://user-images.githubusercontent.com/52024566/196717600-19b9c7d8-67d9-40dd-b0e1-4aa985767346.png)

리포지토리는 DB에 접근해서 데이터를 저장하고 관리합니다. 여기서는 `SQLException`체크 예외를 던집니다.

`NetworkClient`는 외부 네트워크에 접속해서 어떤 기능을 처리하는 객체입니다. 여기서는 `ConnectException` 체크 예외를 던집니다.

서비스는 리포지토리와 `NetworkClient`를 둘 다 호출합니다.

- 따라서 두 곳에서 올라오는 체크 예외인 `SQLException`과 `ConnectException`을 처리해야 합니다.
- 그런데 서비스는 이 둘을 처리할 방법을 모릅니다. `ConnectException`처럼 연결이 실패하거나 `SQLException`처럼 데이터베이스에서 발생하는 문제처럼 심각한 문제들은 대부분 애플리케이션 로직에서 처리할 방법이 없습니다.

- 결국 서비스는 `SQLException`과 `ConnectException`를 처리할 수 없으므로 둘다 밖으로 던집니다.
    - 체크 예외이기 때문에 던질 경우 아래와 같이 선언해야 합니다.
    - `method() throws SQLException, ConnectException`

- 당연히 컨트롤러도 두 예외를 처리할 방법이 없습니다.
    - 아래처럼 선언해서 예외를 밖으로 던져야 합니다.
    - `method() throws SQLException, ConnectException`

웹 애플리케이션이라면 서블릿의 오류 페이지나, 또는 스프링 MVC가 제공하는 `ControllerAdvice`에서 이런 예외를 공통으로 처리할 수 있습니다.

이런 문제들은 보통 사용자에게 어떤 문제가 발생했는지 자세히 설명하기가 어렵습니다. 그래서 사용자에게는 “서비스에 문제가 있습니다.” 라는 일반적인 메시지를 보여줍니다. (“데이터베이스에 어떤 오류가 발생했어요” 라고 알려주어도 일반 사용자는 이해할 수 없습니다. 그리고 보안에도 문제가 될 수 있습니다.)

API라면 보통 HTTP 상태코드 500(내부 서버 오류)을 사용해서 응답을 내려줍니다.

이렇게 해결이 불가능한 공통 예외는 별도의 오류 로그를 남기고, 개발자가 오류를 빨리 인지할 수 있도록 메일, 알림(문자, 슬랙)등을 통해서 전달 받아야 합니다. 예를 들어서 `SQLException`이 잘못된 SQL을 작성해서 발생했다면, 개발자가 해당 SQL을 수정해서 배포하기 전까지 사용자는 같은 문제를 겪게 됩니다.

체크 예외 문제점 - 코드 - `CheckedAppTest`

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

import java.net.ConnectException;
import java.sql.SQLException;

import static org.assertj.core.api.Assertions.assertThatThrownBy;

@Slf4j
public class CheckedAppTest {
  
    @Test
    void checked() {
        Controller controller = new Controller();
        assertThatThrownBy(() -> controller.request())
          .isInstanceOf(Exception.class);
    }
  
    static class Controller {
        Service service = new Service();
      
        public void request() throws SQLException, ConnectException {
            service.logic();
        }
    }
  
    static class Service {
        Repository repository = new Repository();
        NetworkClient networkClient = new NetworkClient();
      
        public void logic() throws SQLException, ConnectException {
            repository.call();
            networkClient.call();
        }
    }
  
    static class NetworkClient {
        public void call() throws ConnectException {
            throw new ConnectException("연결 실패");
        }
    }
  
    static class Repository {
        public void call() throws SQLException {
            throw new SQLException("ex");
        }
    }
}
```

서비스

- 체크 예외를 처리하지 못해서 밖으로 던지기 위해 `logic() throws SQLException, ConnectException`를 선언하였습니다.

컨트롤러

- 체크 예외를 처리하지 못해서 밖으로 던지기 위해 `request() throws SQLException, ConnectException`를 선언하였습니다.

그런데 여기서는 두 가지 문제가 있습니다

### **2가지 문제**

 1. 복구 불가능한 예외

대부분의 예외는 복구가 불가능합니다. 일부 복구가 가능한 예외도 있지만 아주 적습니다. 

`SQLException`의 예를 들면 데이터베이스에 무언가 문제가 있어서 발생하는 예외입니다. 

SQL 문법에 문제가 있을 수도 있고, 데이터베이스 자체에 뭔가 문제가 발생했을 수도 있습니다. 데이터베이스 서버가 중간에 다운 되었을 수도 있지요. 

이런 문제들은 대부분 복구가 불가능합니다. 특히나 대부분의 서비스나 컨트롤러는 이런 문제를 해결할 수 없습니다. 

따라서 이런 문제들은 일관성 있게 공통으로 처리해야 합니다. 오류 로그를 남기고 개발자가 해당 오류를 빠르게 인지하는 것이 필요합니다. 

서블릿 필터, 스프링 인터셉터, 스프링의 `ControllerAdvice`를 사용하면 이런 부분을 깔끔하게 공통으로 해결할 수 있습니다.

 2. 의존 관계에 대한 문제 

체크 예외의 또 다른 심각한 문제는 예외에 대한 의존 관계 문제 앞서 대부분의 예외는 복구 불가능한 예외라고 했습니다. 

그런데 체크 예외이기 때문에 컨트롤러나 서비스 입장에서는 본인이 처리할 수 없어도 어쩔 수 없이 `throws`를 통해 던지는 예외를 선언해야 합니다.

**체크 예외 throws 선언**

```java
class Controller {
    public void request() throws SQLException, ConnectException {
        service.logic();
    }
}

class Service {
    public void logic() throws SQLException, ConnectException {
        repository.call();
        networkClient.call();
    }
}
```

그런데 `throws SQLException, ConnectException`처럼 예외를 던지는 부분을 코드에 선언하는 것이 문제가 되는 이유가 무엇일까요?? 

바로 서비스, 컨트롤러에서 `java.sql.SQLException`을 **의존**하기 때문입니다

향후 리포지토리를 JDBC 기술이 아닌 다른 기술로 변경한다면, 그래서 `SQLException`이 아니라 예를 들어서 `JPAException`으로 예외가 변경된다면 
우리는 `SQLException`에 의존하던 모든 서비스, 컨트롤러의 코드를 `JPAException`에 의존하도록 고쳐야 합니다.

서비스나 컨트롤러 입장에서는 어차피 본인이 처리할 수도 없는 예외를 의존해야 하는 큰 단점이 발생합니다.

결과적으로 OCP, DI를 통해 클라이언트 코드의 변경 없이 대상 구현체를 변경할 수 있다는 장점이 체크 예외 때문에 발목을 잡힌다는 것이지요!

**체크 예외 구현 기술 변경시 파급 효과**

![https://user-images.githubusercontent.com/52024566/196717607-05c8a5df-0d11-404c-b357-8a2caf39c810.png](https://user-images.githubusercontent.com/52024566/196717607-05c8a5df-0d11-404c-b357-8a2caf39c810.png)

JDBC → JPA 같은 기술로 변경하면 예외도 함께 변경해야 합니다. 그리고 해당 예외를 던지는 모든 다음 부분도 함께 변경해야 합니다.

```java
logic() throws SQLException 
	...
logic() throws JPAException
```

(참고로 JPA 예외는 실제 이렇지는 않고, 이해하기 쉽게 예를 든 것)

### **정리**

처리할 수 있는 체크 예외라면 서비스나 컨트롤러에서 처리하겠지만, 지금처럼 데이터베이스나 네트워크 통신처럼 시스템 레벨에서 올라온 예외들은 대부분 복구가 불가능합니다. 그리고 실무에서 발생하는 대부분의 예외들은 이러한 시스템 예외들입니다.

문제는 이런 경우에 체크 예외를 사용하면 아래에서 올라온 복구 불가능한 예외를 서비스, 컨트롤러 같은 각각의 클래스가 모두 알고 있어야 합니다. 그래서 불필요한 의존관계 문제가 발생합니다.

**throws Exception**

`SQLException`, `ConnectException` 같은 시스템 예외는 컨트롤러나 서비스에서는 대부분 복구가 불가능하고 처리할 수 없는 체크 예외이므로 다음과 같이 처리해주어야 합니다.

```java
void method() throws SQLException, ConnectException {..}
```

그런데 다음과 같이 최상위 예외인 `Exception`을 던져도 문제를 해결할 수 있습니다.

```java
void method() throws Exception {..}
```

이렇게 하면 `Exception`은 물론이고 그 하위 타입인 `SQLException`, `ConnectException`도 함께 던지게 됩니다. 코드가 깔끔해지는 것 같지만, `Exception`은 최상위 타입이므로 모든 체크 예외를 다 밖으로 던지는 문제가 발생합니다.

결과적으로 체크 예외의 최상위 타입인 `Exception`을 던지게 되면 다른 체크 예외를 체크할 수 있는 기능이 무효화 되고, 중요한 체크 예외를 다 놓치게 됩니다. 

중간에 중요한 체크 예외가 발생해도 컴파일러는 `Exception`을 던지기 때문에 문법에 맞다고 판단해서 컴파일 오류도 발생하지 않겠죠.  

이렇게 하면 모든 예외를 다 던지기 때문에 체크 예외를 의도한 대로 사용하는 것이 아닙니다. 

따라서 꼭 필요한 경우가 아니면 이렇게 `Exception` 자체를 밖으로 던지는 것은 좋지 않은 방법입니다.

# 6. 언체크 예외 활용

**런타임 예외 사용 - 그림**

![https://user-images.githubusercontent.com/52024566/196953356-417165d7-667b-43b4-a775-3063ede85652.png](https://user-images.githubusercontent.com/52024566/196953356-417165d7-667b-43b4-a775-3063ede85652.png)

`SQLException`을 런타임 예외인 `RuntimeSQLException`으로 변환합니다.

`ConnectException` 대신에 `RuntimeConnectException`을 사용하도록 변경합니다.

이 예외들은 런타임 예외이기 때문에 서비스, 컨트롤러는 해당 예외들을 처리할 수 없다면 별도의 선언 없이 그냥 두면 됩니다.

런타임 예외 사용 변환 - 코드 - `UncheckedAppTest`

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

import java.sql.SQLException;

import static org.assertj.core.api.Assertions.assertThatThrownBy;

@Slf4j
public class UncheckedAppTest {
  
    @Test
    void unchecked() {
        Controller controller = new Controller();
        assertThatThrownBy(() -> controller.request())
          .isInstanceOf(Exception.class);
    }
  
    @Test
    void printEx() {
        Controller controller = new Controller();
        try {
            controller.request();
        } catch (Exception e) {
            //e.printStackTrace();
            log.info("ex", e);
        }
    }
  
    static class Controller {
        Service service = new Service();
      
        public void request() {
            service.logic();
        }
    }
  
    static class Service {
        Repository repository = new Repository();
        NetworkClient networkClient = new NetworkClient();

        public void logic() {
            repository.call();
            networkClient.call();
        }
    }
  
    static class NetworkClient {
        public void call() {
            throw new RuntimeConnectException("연결 실패");
        }
    }
  
    static class Repository {
        public void call() {
            try {
                runSQL();
            } catch (SQLException e) {
                throw new RuntimeSQLException(e);
            }
        }
      
        private void runSQL() throws SQLException {
            throw new SQLException("ex");
        }
    }
  
    static class RuntimeConnectException extends RuntimeException {
        public RuntimeConnectException(String message) {
            super(message);
        }
    }
  
    static class RuntimeSQLException extends RuntimeException {
        public RuntimeSQLException() {
        }
      
        public RuntimeSQLException(Throwable cause) {
            super(cause);
        }
    }
}
```

**예외 전환**

리포지토리에서 체크 예외인 `SQLException`이 발생하면 런타임 예외인 `RuntimeSQLException`으로 전환해서 예외를 던집니다.

- 참고로 이때 기존 예외를 포함해주어야 예외 출력 시 스택 트레이스에서 기존 예외도 함께 확인할 수 있습니다.

`NetworkClient`는 단순히 기존 체크 예외를 `RuntimeConnectException`이라는 런타임 예외가 발생하도록 코드를 변경하였습니다.

**런타임 예외 - 대부분 복구 불가능한 예외** 

시스템에서 발생한 예외는 대부분 복구가 불가능한 예외입니다. 

런타임 예외를 사용하면 서비스나 컨트롤러가 이런 복구 불가능한 예외를 신경쓰지 않아도 됩니다. 물론 이렇게 복구 불가능한 예외는 일관성 있게 공통으로 처리해야 합니다.

**런타임 예외 - 의존 관계에 대한 문제** 

런타임 예외는 해당 객체가 처리할 수 없는 예외는 무시하면 됩니다. 따라서 체크 예외 처럼 예외를 강제로 의존하지 않아도 됩니다.

**런타임 예외 throws 생략**

```java
class Controller {
    public void request() {
        service.logic();
    }
}

class Service {
    public void logic() {
        repository.call();
        networkClient.call();
    }
}
```

런타임 예외이기 때문에 컨트롤러나 서비스가 예외를 처리할 수 없다면 다음 부분을 생략할 수 있습니다.

```java
method() throws RuntimeSQLException, RuntimeConnectException
```

따라서 컨트롤러와 서비스에서 해당 예외에 대한 의존 관계가 발생하지 않습니다.

### **체크 예외 구현 기술 변경 시 파급 효과**

![https://user-images.githubusercontent.com/52024566/196953367-ac88c520-b8ca-4ece-892a-3c55966942df.png](https://user-images.githubusercontent.com/52024566/196953367-ac88c520-b8ca-4ece-892a-3c55966942df.png)

런타임 예외를 사용하면 중간에 기술이 변경되어도 해당 예외를 사용하지 않는 컨트롤러, 서비스에서는 코드를 변경하지 않아도 됩니다.

구현 기술이 변경되는 경우, 예외를 공통으로 처리하는 곳에서는 예외에 따른 다른 처리가 필요할 수 있지만 공통 처리하는 한곳만 변경하면 되기 때문에 변경의 영향 범위는 최소화됩니다.

### **정리**

처음 자바를 설계할 당시에는 자바 설계자들은 체크 예외가 더 나은 선택이라 생각했습니다. 그래서 자바가 기본으로 제공하는 기능들에는 체크 예외가 많습니다. 

그런데 시간이 흐르면서 복구 할 수 없는 예외가 너무 많아졌습니다. 특히 라이브러리를 점점 더 많이 사용하면서 처리해야 하는 예외도 더 늘어났지요. 체크 예외는 해당 라이브러리들이 제공하는 모든 예외를 처리할 수 없을 때마다 `throws`에 예외를 덕지덕지 붙어야 했습니다. 

그래서 개발자들은 `throws Exception`이라는 극단적인 방법도 자주 사용하게 되었습니다. 물론 이 방법은 사용하면 안 됩니다. 모든 예외를 던진다고 선언하는 것인데, 결과적으로 어떤 예외를 잡고 어떤 예외를 던지는지 알 수 없기 때문입니다. 

체크 예외를 사용한다면 잡을 건 잡고 던질 예외는 명확하게 던지도록 선언해야 바람직한 코드입니다. 

체크 예외의 이런 문제점 때문에 최근 라이브러리들은 대부분 런타임 예외를 기본으로 제공합니다. 

사실 위에서 예시로 설명한 JPA 기술도 런타임 예외를 사용한 것입니다. 스프링도 대부분 런타임 예외를 제공합니다. 

런타임 예외도 필요하면 잡을 수 있기 때문에 필요한 경우에는 잡아서 처리하고, 그렇지 않으면 자연스럽게 던지도록 두는 것입니다.. 그리고 예외를 공통으로 처리하는 부분을 앞에 만들어서 처리하면 됩니다.

추가로 런타임 예외는 놓칠 수 있기 때문에 문서화가 중요합니다.

### **런타임 예외는 문서화**

런타임 예외는 문서화를 잘해야 합니다.

또는 코드에 `throws 런타임예외`을 남겨서 중요한 예외를 인지할 수 있게 해야 합니다.

아래에서 예시를 확인할 수 있습니다.

**JPA EntityManager**

```java
/**
* Make an instance managed and persistent.
* @param entity entity instance
* @throws EntityExistsException if the entity already exists.
* @throws IllegalArgumentException if the instance is not an
* entity
* @throws TransactionRequiredException if there is no transaction when
* invoked on a container-managed entity manager of that is of type
* <code>PersistenceContextType.TRANSACTION</code>
*/
public void persist(Object entity);
```

예) 문서에 예외 명시

**스프링 JdbcTemplate**

```java
/**
* Issue a single SQL execute, typically a DDL statement.
* @param sql static SQL to execute
* @throws DataAccessException if there is any problem
*/
void execute(String sql) throws DataAccessException;
```

예) `method() throws DataAccessException`와 같이 문서화 + 코드에도 명시하였습니다.

- 런타임 예외도 `throws`에 선언할 수 있습니다. 물론 생략해도 됩니다.
- 던지는 예외가 명확하고 중요하다면, 코드에 어떤 예외를 던지는지 명시되어 있기 때문에 개발자가 IDE를 통해서 예외를 확인하기 편리합니다.
- 물론 컨트롤러나 서비스에서 `DataAccessException`을 사용하지 않는다면 런타임 예외이기 때문에 무시해도 됩니다.