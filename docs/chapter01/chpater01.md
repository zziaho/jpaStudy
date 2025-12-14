# Chapter 01. JPA 소개

---

## 📁 SQL을 직접 다룰 때 발생하는 문제점

### 1. 반복적인 CRUD 작업

하나의 객체를 데이터베이스에 CRUD 하려면 너무나 많은 SQL과 JDBC API를 코드로 작성해야한다.

예를들어 조회의 경우: 조회용 SQL 작성 → JDBC API를 사용해 SQL 실행 → 조회 결과를 객체로 매핑

또, 등록의 경우: 등록용 SQL 작성 → 객체 값을 등록 SQL에 전달 → JDBC API를 사용해 SQL 실행

이와 같은 반복적인 작업을 테이블마다 진행해야 하는 문제점이 발생한다.

### 2. SQL에 의존적인 개발

객체에 필드가 하나 추가되면 관련된 모든 SQL을 수정해야 한다.

또한 연관된 객체가 존재하는 경우 이 연관된 객체를 사용할 수 있을지 없을지는 전적으로 SQL에 달려있으며,

데이터 접근 계층을 사용하여 SQL을 숨기더라도 어쩔 수 없이 DAO를 열어 SQL이 실행되는지 확인해야한다.

### 3. 패러다임의 불일치

객체와 관계형 데이터베이스는 지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법 또한 다르다.

따라서 아래와 같은 문제점들이 발생한다.

#### 3.1 상속

객체는 상속 기능이 존재하지만 테이블은 상속 기능이 존재하지 않는다.

따라서 다음과 같은 문제점이 발생한다.

- 자식객체를 저장하려면 부모객체 테이블과 자녀객체 테이블에 나눠서 저장해야한다.
- 조회할 때도 두 테이블을 조인해서 데이터를 가져온 후 자식객체를 생성해야한다.

#### 3.2 연관관계

객체는 참조를 사용하여 연관관계를 맺고, 테이블은 외래 키를 사용하여 연관관계를 맺는다.

따라서 객체를 테이블에 맞추면 객체지향적이지 않고, 객체지향적으로 모델링하면 CRUD가 복잡해지는 문제점이 발생한다.

#### 3.3 객체 그래프 탐색

객체는 자유롭게 객체 그래프를 탐색할 수 있어야하나 SQL을 직접 다루게 되면 처음 실행하는 SQL에 따라 탐색 범위가 정해진다.

```java
class Member {
    String id;
    Team team;        // 참조로 연관관계
    String username;
}

class Team {
    Long id;
    String name;
}

// Member만 조회하는 SQL
SELECT * FROM MEMBER WHERE MEMBER_ID = ?

Member member = memberDAO.find(memberId);
member.getTeam();   // null => SQL로 조회하지 않았으므로
```

#### 3.4 비교

SQL을 통해 데이터베이스에서 조회할 때마다 새로운 객체가 생성된다.

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);

member1 == member2;  // false, 다른 인스턴스
```

---

이러한 패러다임의 차이를 극복하기 위해 개발자들은 너무 많은 시간과 코드를 소비해야 하는 문제점이 있다.

이 패러다임 불일치 문제를 극복하기 위한 결과물이 JPA다.

---

## 📁 JPA란 무엇인가?

Java Persistence API의 줄임말로 자바 진영의 ORM(Object-Relational Mapping) 기술 표준이다.

ORM은 말 그대로 객체와 관계형 데이터베이스를 매핑한다는 뜻이다.

JPA와 같은 ORM 프레임워크를 사용하면 ORM 프레임워크가 객체와 테이블을 매핑하여 패러다임의 불일치 문제를 개발자 대신 해결해준다.

예를들어 객체를 저장할 때 INSERT SQL을 직접 작성하는 것이 아니라 마치 자바 컬렉션에 저장하듯이 ORM 프레임워크에 저장하면 ORM 프레임워크가 적절한 SQL을 생성하여 데이터베이스에 객체를 저장해준다.

### 예시

JPA를 사용해서 객체를 저장하는 코드는 아래와 같다.

```java
// 저장할 객체 생성 및 값 세팅
Member member1 = new Member();
member.setId("member1");
member.setUsername("회원1");

jpa.persist(member1); // 저장
```

이 코드를 실행하면 JPA는 자동으로 아래와 같은 SQL을 실행한다.

```sql
INSERT INTO MEMBER (MEMBER_ID, NAME) VALUES ('member1', '회원1')
```

---

## 📁 JPA를 사용해야 하는 이유 (장점)

### 1. 생산성

반복적인 코드와 CRUD용 SQL을 개발자가 직접 작성하지 않아도 되기 때문에 생산성이 높아진다.

#### 1.1 SQL 직접 사용시:

```java
String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES(?, ?)";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, member.getId());
pstmt.setString(2, member.getName());
pstmt.executeUpdate();
// ... 예외 처리, 자원 정리
```

#### 1.2 JPA 사용시:

```java
jpa.persist(member);
```

### 2. 유지보수

SQL을 직접 다루면 객체에 필드가 하나만 추가되어도 관련된 모든 SQL을 변경해야 하지만,

JPA를 사용하면 이런 과정을 JPA가 대신 처리 해 주기 때문에 유지보수 해야하는 코드 수가 줄어든다.

#### 2.1 SQL 직접 사용시:

```java
// 모든 SQL과 ResultSet 매핑 코드를 수정해야 함
String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME, TEL) VALUES(?, ?, ?)";  // 수정
pstmt.setString(3, member.getTel());  // 추가

String sql = "SELECT MEMBER_ID, NAME, TEL FROM MEMBER";  // 수정
member.setTel(rs.getString("TEL"));  // 추가
```

#### 2.2 JPA 사용시:

```java
// 엔티티(객체)에만 필드 추가
@Entity
public class Member {
    private String id;
    private String username;
    private String tel;  // 추가만 하면 됨
}
// SQL은 JPA가 자동으로 처리
```

### 3. 패러다임의 불일치 해결

위에서 언급했던 패러다임의 불일치(상속, 연관관계, 객체 그래프 탐색, 비교) 문제를 JPA가 해결 해 준다.

패러다임의 불일치 문제를 어떻게 해결하는지는 앞으로 이어지는 챕터들에서 지속적으로 다룰 예정이다.

### 4. 성능

JPA는 애플리케이션과 데이터베이스 사이에서 다양한 성능 최적화 기회를 제공한다.

1️⃣ 1차 캐시와 동일성 보장

2️⃣ 트랜잭션을 지원하는 쓰기 지연

3️⃣ 지연 로딩과 즉시 로딩

4️⃣ 변경 감지(Dirty Checking)

이 내용들은 이후에 나올 챕터에서 더 자세하게 다뤄볼 예정이다.

### 5. 데이터베이스 독립성

관계형 데이터베이스는 같은 기능도 벤더마다 사용법이 다른 경우가 많다.

단적인 예로 페이징 처리가 데이터베이스마다 달라서 사용법을 각각 학습해야 한다.

JPA는 애플리케이션과 데이터베이스 사이에 추상화된 데이터 접근 계층을 제공하여 애플리케이션이 특정 데이터베이스 기술에 종속되지 않도록 한다.

만약 데이터베이스를 변경하면 JPA에게 다른 데이터베이스를 사용한다고 알려주기만 하면 된다.