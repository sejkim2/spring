# JDBC

## JDBC를 사용하는 이유
<img width="562" alt="스크린샷 2024-10-06 오후 9 17 49" src="https://github.com/user-attachments/assets/4d5113f4-b5ce-4037-b884-4f6b43978261">

어플리케이션 서버와 db 간 통신은 3가지 단계로 나뉜다.
1. 커넥션 연결 (tcp/ip)
2. 어플리케이션 서버에서 db로 sql 전달
3. db에서 어플리케이션 서버로 쿼리 응답

문제는 이 세가지 방법을 구현하는 방법이 db 회사마다 다르다. 이를 위해 나온 것이 jdbc 표준이다.

<img width="565" alt="스크린샷 2024-10-06 오후 9 22 16" src="https://github.com/user-attachments/assets/b9e2feb7-edf7-4e42-9953-6b5a2d9be060">

1. 커넥션 연결 - Connection
2. sql 전달 - Statement
3. 쿼리 응답 - ResultSet

db 회사는 위 세가지 표준 인터페이스를 기준으로 구현하며, 우리는 추상화된 함수만 사용하면 동일한 방법으로 어플리케이션 서버와 db 간 통신을 개발할 수 있다.

## db를 다루는 최신 기술

1. SQL Mapper

<img width="569" alt="스크린샷 2024-10-06 오후 9 29 39" src="https://github.com/user-attachments/assets/5a2597a5-33c2-4529-9a25-4f4a2bd20122">

> JdbcTemplate
> MyBatis

* 쿼리 응답을 객체로 변환
* jdbc 특유의 반복되는 코드 제거
* SQL 문을 직접 작성해야 한다.

2. ORM (Object Relational Mapping)

<img width="557" alt="스크린샷 2024-10-06 오후 9 33 46" src="https://github.com/user-attachments/assets/8e13869e-5df5-4dd6-949e-b116cf71716b">

> JPA

* 객체와 관계형 데이터베이스 테이블을 매핑
* SQL 문 작성 직접 작성 x (알아서 만들어준다.)

## jdbc 사용

1. 커넥션 연결 - connection
<img width="570" alt="스크린샷 2024-10-07 오후 9 30 31" src="https://github.com/user-attachments/assets/ff09d84a-9f41-43fc-980f-d1f685f8416d">

```
public class DBConnectionUtil {
    public static Connection getConnection() {
        try {
            Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
            return connection;
        } catch (SQLException e) {
            throw new IllegalStateException(e);
        }
    }
}
```
* DriverManager는 URL 정보를 처리할 수 있는 드라이버를 탐색하고 탐색 완료하면 그 드라이버로 데이터베이스에 연결하여 커넥션을 반환한다.

2. sql 전달 - statement
3. 쿼리 응답 - resultset
```
public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values(?, ?)";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = DBConnectionUtil.getConnection();

            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());  // ?가 value로 변환됨
            pstmt.setInt(2, member.getMoney());
            pstmt.executeUpdate();

            return member;
        } catch (SQLException e) {
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }
```
* String으로 된 sql 문을 statement에 넣어서 실제 sql 문으로 변환
* sql injection 공격 (사용자가 입력 창에 sql문을 직접 넣는 공격)을 예방하기 위해 statement의 자식인 PrepardStatement를 사용하여 파라미터 바인딩 방식을 사용
* setString(n, value) : String sql의 ?에 1부터 n까지 차례로 value로 매핑된다.
* pstmt.executeUpdate : sql을 데이터베이스에 전달

리소스 회수
```
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
```
* close() 함수를 사용할 때 발생할 수 있는 예외 때문에 코드가 복잡해지는 단점이 있음

