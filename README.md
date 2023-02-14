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