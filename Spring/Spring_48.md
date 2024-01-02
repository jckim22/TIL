# 체크 예외와 인터페이스
서비스 계층은 가급적 특정 구현 기술에 의존하지 않고, 순수하게 유지하는 것이 좋다. 이렇게 하려면 예외에 대한 의존 도 함께 해결해야한다.
예를 들어서 서비스가 처리할 수 없는 `SQLException` 에 대한 의존을 제거하려면 어떻게 해야할까?
서비스가 처리할 수 없으므로 리포지토리가 던지는 `SQLException` 체크 예외를 런타임 예외로 전환해서 서비스 계 층에 던지자. 이렇게 하면 서비스 계층이 해당 예외를 무시할 수 있기 때문에, 특정 구현 기술에 의존하는 부분을 제거하 고 서비스 계층을 순수하게 유지할 수 있다.

지금부터 코드로 이 방법을 적용해보자.


### 인터페이스 도입
먼저 `MemberRepository` 인터페이스도 도입해서 구현 기술을 쉽게 변경할 수 있게 해보자.

![](https://velog.velcdn.com/images/jckim22/post/d284276f-9b04-49d7-b840-83beb69ca6ee/image.png)


- 이렇게 인터페이스를 도입하면 `MemberService` 는 `MemberRepository` 인터페이스에만 의존하면 된다. 
- 이제 구현 기술을 변경하고 싶으면 DI를 사용해서 `MemberService` 코드의 변경 없이 구현 기술을 변경할 수 있다.


**MemberRepository 인터페이스** 
```java
 package hello.jdbc.repository;
 import hello.jdbc.domain.Member;
 public interface MemberRepository {
     Member save(Member member);
     Member findById(String memberId);
     void update(String memberId, int money);
     void delete(String memberId);
}
```
특정 기술에 종속되지 않는 순수한 인터페이스이다. 이 인터페이스를 기반으로 특정 기술을 사용하는 구현체를 만들면 된다.
**체크 예외 코드에 인터페이스 도입시 문제점 - 인터페이스** 
```java
 package hello.jdbc.repository;
 import hello.jdbc.domain.Member;
    
 import java.sql.SQLException;
 public interface MemberRepositoryEx {
     Member save(Member member) throws SQLException;
     Member findById(String memberId) throws SQLException;
     void update(String memberId, int money) throws SQLException;
     void delete(String memberId) throws SQLException;
}
```
인터페이스의 메서드에 `throws SQLException` 이 있는 것을 확인할 수 있다.

**특정 기술에 종속되는 인터페이스**
구현 기술을 쉽게 변경하기 위해서 인터페이스를 도입하더라도 `SQLException` 과 같은 특정 구현 기술에 종속적인 체크 예외를 사용하게 되면 인터페이스에도 해당 예외를 포함해야 한다. 하지만 이것은 우리가 원하던 순수한 인터페이 스가 아니다. JDBC 기술에 종속적인 인터페이스일 뿐이다. 인터페이스를 만드는 목적은 구현체를 쉽게 변경하기 위함 인데, 이미 인터페이스가 특정 구현 기술에 오염이 되어 버렸다. 향후 JDBC가 아닌 다른 기술로 변경한다면 인터페이 스 자체를 변경해야 한다.

**런타임 예외와 인터페이스**
런타임 예외는 이런 부분에서 자유롭다. 인터페이스에 런타임 예외를 따로 선언하지 않아도 된다. 따라서 인터페이스가 특정 기술에 종속적일 필요가 없다.


### MyException

```java
public class MyDbException extends RuntimeException {

    public MyDbException() {
    }

    public MyDbException(String message) {
        super(message);
    }

    public MyDbException(String message, Throwable cause) {
        super(message, cause);
    }

    public MyDbException(Throwable cause) {
        super(cause);
    }
}

```

SQLException을 런타임 Exception으로 보내기 위해 그에 맞는 언체크 예외를 만들었다.

### Repository

```java
@Slf4j
public class MemberRepositoryV4_1 implements MemberRepository {

    private final DataSource dataSource;

    public MemberRepositoryV4_1(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Member save(Member member) {
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
            throw new MyDbException(e);
        } finally {
            close(con, pstmt, null);
        }

    }

    @Override
    public Member findById(String memberId) {
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
            throw new MyDbException(e);
        } finally {
            close(con, pstmt, rs);
        }

    }

    @Override
    public void update(String memberId, int money) {
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
            throw new MyDbException(e);
        } finally {
            close(con, pstmt, null);
        }

    }

    @Override
    public void delete(String memberId) {
        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            throw new MyDbException(e);
        } finally {
            close(con, pstmt, null);
        }

    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        //주의! 트랜잭션 동기화를 사용하려면 DataSourceUtils를 사용해야 한다.
        DataSourceUtils.releaseConnection(con, dataSource);
    }


    private Connection getConnection() throws SQLException {
        //주의! 트랜잭션 동기화를 사용하려면 DataSourceUtils를 사용해야 한다.
        Connection con = DataSourceUtils.getConnection(dataSource);
        log.info("get connection={}, class={}", con, con.getClass());
        return con;
    }


}

```
MemberRepository 인터페이스(추상화)에 의존하게 했고 SqlException이 터지면 MyDBException으로 바꿔서 예외를 처리했다.


#### Service
```java
@Slf4j
public class MemberServiceV4 {

    private final MemberRepository memberRepository;

    public MemberServiceV4(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Transactional
    public void accountTransfer(String fromId, String toId, int money) {
        bizLogic(fromId, toId, money);
    }

    private void bizLogic(String fromId, String toId, int money) {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);

        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체중 예외 발생");
        }
    }

}

```
그렇게 됨으로써 Service 계층에서 드디어 구현체인 Repository에 의존하지 않고 인터페이스에 의존하게 되었으며 체크 예외를 던질 필요가 없어졌기 때문에 OCP와 DIP를 지킬 수 있게 되었다.

#### Test
```java
@Slf4j
@SpringBootTest
class MemberServiceV4Test {

    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private MemberServiceV4 memberService;

    @TestConfiguration
    static class TestConfig {

        private final DataSource dataSource;

        public TestConfig(DataSource dataSource) {
            this.dataSource = dataSource;
        }

        @Bean
        MemberRepository memberRepository() {
//            return new MemberRepositoryV4_1(dataSource);
//            return new MemberRepositoryV4_2(dataSource);
            return new MemberRepositoryV5(dataSource);
        }

        @Bean
        MemberServiceV4 memberServiceV4() {
            return new MemberServiceV4(memberRepository());
        }
    }

    @AfterEach
    void after() {
        memberRepository.delete(MEMBER_A);
        memberRepository.delete(MEMBER_B);
        memberRepository.delete(MEMBER_EX);
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
    void accountTransfer() {
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
    void accountTransferEx() {
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
        Member findMemberB = memberRepository.findById(memberEx.getMemberId());
        assertThat(findMemberA.getMoney()).isEqualTo(10000);
        assertThat(findMemberB.getMoney()).isEqualTo(10000);
    }
    
}

```

@TestConfiguration으로 주입받을 구현체를 설정하고 테스트를 하면 테스트를 통과하는 것을 볼 수 있다.


**정리**
- 체크 예외를 런타임 예외로 변환하면서 인터페이스와 서비스 계층의 순수성을 유지할 수 있게 되었다.
- 덕분에 향후 JDBC에서 다른 구현 기술로 변경하더라도 서비스 계층의 코드를 변경하지 않고 유지할 수 있다.

**남은 문제**

리포지토리에서 넘어오는 특정한 예외의 경우 복구를 시도할 수도 있다. 그런데 지금 방식은 항상 `MyDbException` 이라는 예외만 넘어오기 때문에 예외를 구분할 수 없는 단점이 있다. 만약 특정 상황에는 예외를 잡아서 복구하고 싶으 면 예외를 어떻게 구분해서 처리할 수 있을까?


# 데이터 접근 예외 직접 만들기
데이터베이스 오류에 따라서 특정 예외는 복구하고 싶을 수 있다.
예를 들어서 회원 가입시 DB에 이미 같은 ID가 있으면 ID 뒤에 숫자를 붙여서 새로운 ID를 만들어야 한다고 가정해보자.

ID를 `hello` 라고 가입 시도 했는데, 이미 같은 아이디가 있으면 `hello12345` 와 같이 뒤에 임의의 숫자를 붙여서 가 입하는 것이다.

데이터를 DB에 저장할 때 같은 ID가 이미 데이터베이스에 저장되어 있다면, 데이터베이스는 오류 코드를 반환하고, 이 오류 코드를 받은 JDBC 드라이버는 `SQLException` 을 던진다. 그리고 `SQLException` 에는 데이터베이스가 제공 하는 `errorCode` 라는 것이 들어있다.

![](https://velog.velcdn.com/images/jckim22/post/c21df490-2589-4e1d-8159-feda27529951/image.png)


**H2 데이터베이스의 키 중복 오류 코드**
```java
e.getErrorCode() == 23505 
```
`SQLException` 내부에 들어있는 `errorCode` 를 활용하면 데이터베이스에서 어떤 문제가 발생했는지 확인할 수 있 다.



**H2 데이터베이스 예**
`23505` : 키 중복 오류
`42000` : SQL 문법 오류
참고로 같은 오류여도 각각의 데이터베이스마다 정의된 오류 코드가 다르다. 따라서 오류 코드를 사용할 때는 데이터베
이스 메뉴얼을 확인해야 한다.

**예) 키 중복 오류 코드** H2 DB: `23505`
MySQL: `1062`




서비스 계층에서는 예외 복구를 위해 키 중복 오류를 확인할 수 있어야 한다. 그래야 새로운 ID를 만들어서 다시 저장을 시도할 수 있기 때문이다. 이러한 과정이 바로 예외를 확인해서 복구하는 과정이다. 리포지토리는 `SQLException` 을 서비스 계층에 던지고 서비스 계층은 이 예외의 오류 코드를 확인해서 키 중복 오류(`23505` )인 경우 새로운 ID를 만들 어서 다시 저장하면 된다.
그런데 `SQLException` 에 들어있는 오류 코드를 활용하기 위해 `SQLException` 을 서비스 계층으로 던지게 되면, 서비스 계층이 `SQLException` 이라는 JDBC 기술에 의존하게 되면서, 지금까지 우리가 고민했던 서비스 계층의 순수 성이 무너진다.

이 문제를 해결하려면 앞서 배운 것 처럼 리포지토리에서 예외를 변환해서 던지면 된다. 
`SQLException` -> `MyDuplicateKeyException`

**MyDuplicateKeyException**

```java
public class MyDuplicateKeyException extends MyDbException {

    public MyDuplicateKeyException() {
    }

    public MyDuplicateKeyException(String message) {
        super(message);
    }

    public MyDuplicateKeyException(String message, Throwable cause) {
        super(message, cause);
    }

    public MyDuplicateKeyException(Throwable cause) {
        super(cause);
    }
}

```
- 기존에 사용했던 `MyDbException` 을 상속받아서 의미있는 계층을 형성한다. 이렇게하면 데이터베이스 관련 예 외라는 계층을 만들 수 있다.
- 그리고 이름도 `MyDuplicateKeyException` 이라는 이름을 지었다. 이 예외는 데이터 중복의 경우에만 던져 야 한다.
- 이 예외는 우리가 직접 만든 것이기 때문에, JDBC나 JPA 같은 특정 기술에 종속적이지 않다. 따라서 이 예외를 사용하더라도 서비스 계층의 순수성을 유지할 수 있다. (향후 JDBC에서 다른 기술로 바꾸어도 이 예외는 그대로 유지할 수 있다.)


```java
@Slf4j
public class ExTranslatorV1Test {

    Repository repository;
    Service service;

    @BeforeEach
    void init() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        repository = new Repository(dataSource);
        service = new Service(repository);
    }

    @Test
    void duplicateKeySave() {
        service.create("myId");
        service.create("myId");//같은 ID 저장 시도
    }

    @Slf4j
    @RequiredArgsConstructor
    static class Service {
        private final Repository repository;

        public void create(String memberId) {
            try {
                repository.save(new Member(memberId, 0));
                log.info("saveId={}", memberId);
            } catch (MyDuplicateKeyException e) {
                log.info("키 중복, 복구 시도");
                String retryId = generateNewId(memberId);
                log.info("retryId={}", retryId);
                repository.save(new Member(retryId, 0));
            } catch (MyDbException e) {
                log.info("데이터 접근 계층 예외", e);
                throw e;
            }
        }

        private String generateNewId(String memberId) {
            return memberId + new Random().nextInt(10000);
        }

    }

    @RequiredArgsConstructor
    static class Repository {
        private final DataSource dataSource;

        public Member save(Member member) {
            String sql = "insert into member(member_id, money) values(?,?)";
            Connection con = null;
            PreparedStatement pstmt = null;

            try {
                con = dataSource.getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setString(1, member.getMemberId());
                pstmt.setInt(2, member.getMoney());
                pstmt.executeUpdate();
                return member;
            } catch (SQLException e) {
                //h2 db
                if (e.getErrorCode() == 23505) {
                    throw new MyDuplicateKeyException(e);
                }
                throw new MyDbException(e);
            } finally {
                JdbcUtils.closeStatement(pstmt);
                JdbcUtils.closeConnection(con);
            }
        }
    }
}

```
1. 보면 레포지토리에서 중복으로 인한 SQLException이 터졌다.
2. 에러코드는 중복이기 때문에 23505이 나왔고 우리는 이걸 조건식으로 확인해서 MyDuplicateKeyExcetpion을 던지기로 한다.
3. Service에서는 MyDuplicateKeyException을 catch해서 그에 맞는 처리를 한다.
여기서는 아이디 뒤에 랜덤한 숫자를 붙여준다.

**정리**
- SQL ErrorCode로 데이터베이스에 어떤 오류가 있는지 확인할 수 있었다.
- 예외 변환을 통해 `SQLException` 을 특정 기술에 의존하지 않는 직접 만든 예외인
`MyDuplicateKeyException` 로 변환 할 수 있었다.
- 리포지토리 계층이 예외를 변환해준 덕분에 서비스 계층은 특정 기술에 의존하지 않는 `MyDuplicateKeyException` 을 사용해서 문제를 복구하고, 서비스 계층의 순수성도 유지할 수 있었다.

**남은 문제**
- SQL ErrorCode는 각각의 데이터베이스 마다 다르다. 결과적으로 데이터베이스가 변경될 때 마다 ErrorCode 도 모두 변경해야 한다.
   - 예) 키 중복 오류 코드 H2: `23505`
       - MySQL: `1062`
- 데이터베이스가 전달하는 오류는 키 중복 뿐만 아니라 락이 걸린 경우, SQL 문법에 오류 있는 경우 등등 수십 수 백가지 오류 코드가 있다. 이 모든 상황에 맞는 예외를 지금처럼 다 만들어야 할까? 추가로 앞서 이야기한 것 처럼 데이터베이스마다 이 오류 코드는 모두 다르다.


# 스프링 예외 추상화 이해
스프링은 앞서 설명한 문제들을 해결하기 위해 데이터 접근과 관련된 예외를 추상화해서 제공한다.

**스프링 데이터 접근 예외 계층**
![](https://velog.velcdn.com/images/jckim22/post/5bc26beb-50ee-4347-aba1-6cb635d4c151/image.png)

![](https://velog.velcdn.com/images/jckim22/post/5ae17454-04a8-45ce-aab0-85f8f1564ea0/image.png)


#### 스프링이 제공하는 예외 변환기
스프링은 데이터베이스에서 발생하는 오류 코드를 스프링이 정의한 예외로 자동으로 변환해주는 변환기를 제공한다.
코드를 통해 스프링이 제공하는 예외 변환기를 알아보자. 먼저 에러 코드를 확인하는 부분을 간단히 복습해보자.
```java
@Slf4j
public class SpringExceptionTranslatorTest {

    DataSource dataSource;

    @BeforeEach
    void init() {
        dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
    }

    @Test
    void sqlExceptionErrorCode() {
        String sql = "select bad grammar";

        try {
            Connection con = dataSource.getConnection();
            PreparedStatement stmt = con.prepareStatement(sql);
            stmt.executeQuery();
        } catch (SQLException e) {
            assertThat(e.getErrorCode()).isEqualTo(42122);
            int errorCode = e.getErrorCode();
            log.info("errorCode={}", errorCode);
            log.info("error", e);
        }
    }

    @Test
    void exceptionTranslator() {
        String sql = "select bad grammar";

        try {
            Connection con = dataSource.getConnection();
            PreparedStatement stmt = con.prepareStatement(sql);
            stmt.executeQuery();
        } catch (SQLException e) {
            assertThat(e.getErrorCode()).isEqualTo(42122);

            //org.springframework.jdbc.support.sql-error-codes.xml
            SQLErrorCodeSQLExceptionTranslator exTranslator = new SQLErrorCodeSQLExceptionTranslator(dataSource);
            DataAccessException resultEx = exTranslator.translate("select", sql, e);
            log.info("resultEx", resultEx);
            assertThat(resultEx.getClass()).isEqualTo(BadSqlGrammarException.class);
        }

    }
}

```
exceptionTranslator 테스트를 보자

SQLErrorCodeSQLExceptionTranslator를 사용해서 DataAccessException라는 스프링 예외 최상츠 예외에 result를 받는다.
그 result는 sql을 통해 어떤 예외인지 구별하게 된다.
여기서는 잘못된 문법의 sql이 들어가서 나오는 예외이기 때문에 BadSqlGrammarException이 터질 것이다.


각각의 DB마다 SQL ErrorCode는 다르다. 그런데 스프링은 어떻게 각각의 DB가 제공하는 SQL ErrorCode까지 고 려해서 예외를 변환할 수 있을까?

비밀은 바로 다음 파일에 있다. 
**sql-error-codes.xml**
```xml
 <bean id="H2" class="org.springframework.jdbc.support.SQLErrorCodes">
     <property name="badSqlGrammarCodes">
         <value>42000,42001,42101,42102,42111,42112,42121,42122,42132</value>
     </property>
     <property name="duplicateKeyCodes">
         <value>23001,23505</value>
     </property>
 </bean>
 <bean id="MySQL" class="org.springframework.jdbc.support.SQLErrorCodes">
     <property name="badSqlGrammarCodes">
         <value>1054,1064,1146</value>
     </property>
     <property name="duplicateKeyCodes">
         <value>1062</value>
     </property>
 </bean>
```
- `org.springframework.jdbc.support.sql-error-codes.xml`
- 스프링 SQL 예외 변환기는 SQL ErrorCode를 이 파일에 대입해서 어떤 스프링 데이터 접근 예외로 전환해야 할 지 찾아낸다. 예를 들어서 H2 데이터베이스에서 `42000` 이 발생하면 `badSqlGrammarCodes` 이기 때문에 `BadSqlGrammarException` 을 반환한다.


**정리**
- 스프링은 데이터 접근 계층에 대한 일관된 예외 추상화를 제공한다.
- 스프링은 예외 변환기를 통해서 `SQLException` 의 `ErrorCode` 에 맞는 적절한 스프링 데이터 접근 예외로 변 환해준다.
- 만약 서비스, 컨트롤러 계층에서 예외 처리가 필요하면 특정 기술에 종속적인 `SQLException` 같은 예외를 직 접 사용하는 것이 아니라, 스프링이 제공하는 데이터 접근 예외를 사용하면 된다.
- 스프링 예외 추상화 덕분에 특정 기술에 종속적이지 않게 되었다. 이제 JDBC에서 JPA같은 기술로 변경되어도 예외로 인한 변경을 최소화 할 수 있다. 향후 JDBC에서 JPA로 구현 기술을 변경하더라도, 스프링은 JPA 예외를 적절한 스프링 데이터 접근 예외로 변환해준다.
- 물론 스프링이 제공하는 예외를 사용하기 때문에 스프링에 대한 기술 종속성은 발생한다.
   - 스프링에 대한 기술 종속성까지 완전히 제거하려면 예외를 모두 직접 정의하고 예외 변환도 직접 하면 되지 만, 실용적인 방법은 아니다.



# 스프링 예외 추상화 적용
이제 우리가 만든 애플리케이션에 스프링이 제공하는 데이터 접근 예외 추상화와 SQL 예외 변환기를 적용해보자.

**MemberRepositoryV4_2**

```java
@Slf4j
public class MemberRepositoryV4_2 implements MemberRepository {

    private final DataSource dataSource;
    private final SQLExceptionTranslator exTranslator;

    public MemberRepositoryV4_2(DataSource dataSource) {
        this.dataSource = dataSource;
        this.exTranslator = new SQLErrorCodeSQLExceptionTranslator(dataSource);
    }

    @Override
    public Member save(Member member) {
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
            throw exTranslator.translate("save", sql, e);
        } finally {
            close(con, pstmt, null);
        }

    }

    @Override
    public Member findById(String memberId) {
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
            throw exTranslator.translate("findById", sql, e);
        } finally {
            close(con, pstmt, rs);
        }

    }

    @Override
    public void update(String memberId, int money) {
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
            throw exTranslator.translate("update", sql, e);
        } finally {
            close(con, pstmt, null);
        }

    }

    @Override
    public void delete(String memberId) {
        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            throw exTranslator.translate("delete", sql, e);
        } finally {
            close(con, pstmt, null);
        }

    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        //주의! 트랜잭션 동기화를 사용하려면 DataSourceUtils를 사용해야 한다.
        DataSourceUtils.releaseConnection(con, dataSource);
    }


    private Connection getConnection() throws SQLException {
        //주의! 트랜잭션 동기화를 사용하려면 DataSourceUtils를 사용해야 한다.
        Connection con = DataSourceUtils.getConnection(dataSource);
        log.info("get connection={}, class={}", con, con.getClass());
        return con;
    }
}
```

**SQLExceptionTranslator 생성**

```java
    private final DataSource dataSource;
    private final SQLExceptionTranslator exTranslator;

    public MemberRepositoryV4_2(DataSource dataSource) {
        this.dataSource = dataSource;
        this.exTranslator = new SQLErrorCodeSQLExceptionTranslator(dataSource);
    }

```
그 후 아래처럼 Translator로 그에 해당하는 예외를 던진다.

```java
 catch (SQLException e) {
            throw exTranslator.translate("delete", sql, e);
```

**MemberServiceV4Test - 수정** 
```java
 @Bean
 MemberRepository memberRepository() {
  //return new MemberRepositoryV4_1(dataSource); //단순 예외 변환
 return new MemberRepositoryV4_2(dataSource); //스프링 예외 변환 }
```

`MemberRepository` 인터페이스가 제공되므로 스프링 빈에 등록할 빈만 `MemberRepositoryV4_1` 에서 `MemberRepositoryV4_2` 로 교체하면 리포지토리를 변경해서 테스트를 확인할 수 있다.

**정리**
드디어 예외에 대한 부분을 깔끔하게 정리했다.
스프링이 예외를 추상화해준 덕분에, 서비스 계층은 특정 리포지토리의 구현 기술과 예외에 종속적이지 않게 되었다. 따 라서 서비스 계층은 특정 구현 기술이 변경되어도 그대로 유지할 수 있게 되었다. 다시 DI를 제대로 활용할 수 있게 된 것이다.
추가로 서비스 계층에서 예외를 잡아서 복구해야 하는 경우, 예외가 스프링이 제공하는 데이터 접근 예외로 변경되어서 서비스 계층에 넘어오기 때문에 필요한 경우 예외를 잡아서 복구하면 된다.

# JDBC 반복 문제 해결 - JdbcTemplate
지금까지 서비스 계층의 순수함을 유지하기 위해 수 많은 노력을 했고, 덕분에 서비스 계층의 순수함을 유지하게 되었
다. 이번에는 리포지토리에서 JDBC를 사용하기 때문에 발생하는 반복 문제를 해결해보자.


**JDBC 반복 문제**
- 커넥션 조회, 커넥션 동기화
- `PreparedStatement` 생성 및 파라미터 바인딩
- 쿼리 실행
- 결과 바인딩
- 예외 발생시 스프링 예외 변환기 실행
- 리소스 종료



리포지토리의 각각의 메서드를 살펴보면 상당히 많은 부분이 반복된다. 이런 반복을 효과적으로 처리하는 방법이 바로 템플릿 콜백 패턴이다.
스프링은 JDBC의 반복 문제를 해결하기 위해 `JdbcTemplate` 이라는 템플릿을 제공한다.


```java
@Slf4j
public class MemberRepositoryV5 implements MemberRepository {

    private final JdbcTemplate template;

    public MemberRepositoryV5(DataSource dataSource) {
        this.template = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) {
        String sql = "insert into member(member_id, money) values (?, ?)";
        template.update(sql, member.getMemberId(), member.getMoney());
        return member;
    }

    @Override
    public Member findById(String memberId) {
        String sql = "select * from member where member_id = ?";
        return template.queryForObject(sql, memberRowMapper(), memberId);
    }

    @Override
    public void update(String memberId, int money) {
        String sql = "update member set money=? where member_id=?";
        template.update(sql, money, memberId);
    }

    @Override
    public void delete(String memberId) {
        String sql = "delete from member where member_id=?";
        template.update(sql, memberId);
    }

    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        };
    }

}

```
위처럼 반복을 제거한다.
조회 빼고는 전부 update를 사용하는데 그냥 sql과 순서에 맞는 바인딩할 값들을 넣어주면 끝이다.
그럼 알아서 커넥션, 동기화, 바인딩, 실행, 예외처리, 종료 등을 해준다.

조회할 때는 쿼리만으로 객체를 얻을 수 있어야하는데 이 때는 queryForObject를 사용한다.
그 중 2번째 인자로 RowMapper가 들어가는데 이에 대해 궁금하여 찾아보니
```java
RowMapper<Member> rowMapper = new RowMapper<Member>() {
    @Override
    public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
        Member member = new Member();
        member.setMemberId(rs.getString("member_id"));
        member.setMoney(rs.getInt("money"));
        return member;
    }
};
```
rowMapper에는 mapRow라는 메소드가 선언되어있다.
여기서는 sql을 통한 rs를 갖고 있고 그 값들을 이용해서 객체와 매핑하여 리턴한다.
위는 익명클래스를 활용한 것이고 아래가 람다식을 활용한 것이다.

```java
    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        };
    }
```

# 정리
      
- 서비스 계층의 순수성
   - 트랜잭션 추상화 + 트랜잭션 AOP 덕분에 서비스 계층의 순수성을 최대한 유지하면서 서비스 계층에서 트랜잭션을 사용할 수 있다.
   - 스프링이 제공하는 예외 추상화와 예외 변환기 덕분에, 데이터 접근 기술이 변경되어도 서비스 계층의 순수 성을 유지하면서 예외도 사용할 수 있다.
   - 서비스 계층이 리포지토리 인터페이스에 의존한 덕분에 향후 리포지토리가 다른 구현 기술로 변경되어도 서
비스 계층을 순수하게 유지할 수 있다.
- 리포지토리에서 JDBC를 사용하는 반복 코드가 `JdbcTemplate` 으로 대부분 제거되었다.
