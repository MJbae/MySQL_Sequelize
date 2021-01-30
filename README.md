# MySQL_Sequelize
## 1. Docker로 MySQL을 설치하는 이유

### 가. 로컬환경을 깨끗하게 유지하기 위해

- 로컬 환경에 DBMS를 직접 설치시 로컬환경에서 DB 사용에 따라 잔류한 데이터를 완전히 삭제하기 힘들다
- 가상환경 위에 DBMS를 설치한다면 해당 가상환경을 삭제하면서 관련 데이터도 함께 간편하게 삭제할 수 있음

### 나. 가상환경으로 Docker를 선택한 이유

- VM 기반의 리눅스 등에 DBMS를 설치할 수 있지만 Docker가 더 가볍고 사용하기 편리함

## 2. 설치 요구사항

- 요구사항

## 3. 설치 과정

### 가. 도커설치

- 정상 설치 확인 명령어: docker --version

### 나. mysql server 5.7 버전 이미지 다운로드

- 설치 명령어: docker pull mysql:5.7
- 버전을 명시하지 않으면 자동으로 최신버전을 설치함
- 이미지 정상 다운로드 확인 명령어: docker images

### 다. mysql 컨테이너 실행

- 컨테이너 실행 및 mysql 환경변수 변경 명령어 포함

    ```docker
    docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=password --name mysql_docker_test1 mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    ```

    → 컨테이너 실행 명령어: docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=password --name mysql_docker_test1 mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

    → 환경변수 변경 명령어: -character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

    → mysql config 설정 변경 latin1 → utf8

    → mysql에서 한글작업할 수 있도록 인코딩 설정 변경

- 컨테이너 정상 동작 확인 명령어: docker ps -a

    → status에 up이라고 표시되야 정상 동작

    → run 명령어는 컨테이너 create 과 run을 동시 수행

### 라. mysql 컨테이너에 bash로 접속

- bash 접속 명령어: docker exec -i -t mysql_docker_test1 bash
- mysql 접속 명령어: mysql -u root -p

### 마. mysql sever ↔ mysql client 정상 통신 확인

- mysql workbench에서 mysql server로 test connection 시도

    → 포트 맵핑 상태 점검

### 바. mysql User 생성 및 권한 부여

```sql
mysql> CREATE USER 'MJ'@'%' IDENTIFIED BY 'password';

Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'MJ'@'%';

Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;

Query OK, 0 rows affected (0.00 sec)

mysql> quit

출처: https://jang8584.tistory.com/267 [개발자의 길]
```

- 생성한 User로 server 접속

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ca38639-6c87-433b-bf72-efe2b17debe3/User_.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ca38639-6c87-433b-bf72-efe2b17debe3/User_.png)

## 4. mysql client library 설치

- JDBC 활용하여 java mysql-server 연동

```sql
package MJ.JavaBasic.pc_manager;

import com.mysql.jdbc.Connection;

import java.sql.DriverManager;
import java.sql.SQLException;

public class DBConnector {
    public static void main(String args[]) {
        Connection con = null;

        String server = "127.0.0.1:3309"; // MySQL 서버 주소
        String database = ""; // MySQL DATABASE 이름
        String user_name = "MJ"; //  MySQL 서버 아이디
        String password = "password"; // MySQL 서버 비밀번호

        // 1.드라이버 로딩
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            System.err.println(" !! <JDBC 오류> Driver load 오류: " + e.getMessage());
            e.printStackTrace();
        }

        // 2.연결
        try {
            con = (Connection) DriverManager.getConnection("jdbc:mysql://" + server + "/" + database + "?useSSL=false", user_name, password);
            System.out.println("정상적으로 연결되었습니다.");
        } catch (SQLException e) {
            System.err.println("con 오류:" + e.getMessage());
            e.printStackTrace();
        }

        // 3.해제
        try {
            if (con != null)
                con.close();
        } catch (SQLException ignored) {
        }
    }
}
```

## 에러 해결

### 가. mysql sever ↔ mysql client test connection 실패

- 현상

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67dac1a2-9d6e-436a-9078-a6de6c360e07/.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67dac1a2-9d6e-436a-9078-a6de6c360e07/.png)

- 원인

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/96b46f6b-0862-43cf-b3fd-fb43fb2ebb58/_.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/96b46f6b-0862-43cf-b3fd-fb43fb2ebb58/_.png)

- 문제원인 해결

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a3a58a5-5213-477b-be16-65bb31df8b5a/_.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a3a58a5-5213-477b-be16-65bb31df8b5a/_.png)

- 결과

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d5eef72-b1e0-4ab5-971f-399f20c5487d/.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d5eef72-b1e0-4ab5-971f-399f20c5487d/.png)

## Reference

- 설치과정 전반, [https://www.hanumoka.net/2018/04/29/docker-20180429-docker-install-mysql/](https://www.hanumoka.net/2018/04/29/docker-20180429-docker-install-mysql/)
- User 생성, [https://jang8584.tistory.com/267](https://jang8584.tistory.com/267)
- MySQL Connector 세팅, [https://whitepaek.tistory.com/18](https://whitepaek.tistory.com/18)
- JDBC 활용(MySQL), [https://victorydntmd.tistory.com/145](https://victorydntmd.tistory.com/145)
- 에러 해결, [https://stackoverflow.com/questions/64307077/docker-compose-only-one-usage-of-each-socket-address-protocol-network-address/64310265](https://stackoverflow.com/questions/64307077/docker-compose-only-one-usage-of-each-socket-address-protocol-network-address/64310265)
