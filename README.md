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
