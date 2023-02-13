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

이제 환결 설정은 끝이 났습니다.

