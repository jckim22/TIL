>들어가기 앞서
**DTO(data transfer object)** 
데이터 전송 객체
- DTO는 기능은 없고 데이터를 전달만 하는 용도로 사용되는 객체를 뜻한다.
- 참고로 DTO에 기능이 있으면 안되는가?  
  - 그것은 아니다. 객체의 주 목적이 데이터를 전송하는 것이라면 DTO라 할 수 있다.
- 객체 이름에 DTO를 꼭 붙여야 하는 것은 아니다. 대신 붙여두면 용도를 알 수 있다는 장점은 있다.
- 아이템 검색 조건을 뜻하는 `ItemSearchCond` 도 DTO 역할을 하지만, 이 프로젝트에서 `Cond` 는 검색 조건으로 사용한다 는 규칙을 정했다. 따라서 DTO를 붙이지 않아도 된다. `ItemSearchCondDto` 이렇게 하면 너무 복잡해진다. 그리고 `Cond` 라는것만봐도용도를알수있다.
- 참고로 이런 규칙은 정해진 것이 없기 때문에 해당 프로젝트 안에서 일관성 있게 규칙을 정하면 된다.

# JdbcTemplate

**장점**
- 설정의 편리함
  - JdbcTemplate은 `spring-jdbc` 라이브러리에 포함되어 있는데, 이 라이브러리는 스프링으로 JDBC를
사용할 때 기본으로 사용되는 라이브러리이다. 그리고 별도의 복잡한 설정 없이 바로 사용할 수 있다.
- 반복 문제 해결
JdbcTemplate은 템플릿 콜백 패턴을 사용해서, JDBC를 직접 사용할 때 발생하는 대부분의 반복 작업을 대신 처리해준다.
개발자는 SQL을 작성하고, 전달할 파리미터를 정의하고, 응답 값을 매핑하기만 하면 된다.
우리가 생각할 수 있는 대부분의 반복 작업을 대신 처리해준다.
커넥션 획득
`statement` 를 준비하고 실행
결과를 반복하도록 루프를 실행
커넥션 종료, `statement` , `resultset` 종료 트랜잭션 다루기 위한 커넥션 동기화
예외 발생시 스프링 예외 변환기 실행


## JdbcTemplate 설정

```java
//JdbcTemplate 추가
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
//H2 데이터베이스 추가
runtimeOnly 'com.h2database:h2'
```


# JdbcTemplate 적용1 - 기본
이제부터 본격적으로 JdbcTemplate을 사용해서 메모리에 저장하던 데이터를 데이터베이스에 저장해보자. `ItemRepository` 인터페이스가 있으니 이 인터페이스를 기반으로 JdbcTemplate을 사용하는 새로운 구현체를 개발하자.


> ## 들어가기 전..
#### 람다
```java
        template.update(connection -> {
            //자동 증가 키
            PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
            ps.setString(1, item.getItemName());
            ps.setInt(2, item.getPrice());
            ps.setInt(3, item.getQuantity());
            return ps;
        }, keyHolder);
```
update의 첫번째 인자로 PreaparedStatement가 들어온다.
PreaparedStatement은 함수형 인터페이스로써 위처럼 connection을 받고 메소드 구현이 가능하다.
```java
    private RowMapper<Item> itemRowMapper() {
        return ((rs, rowNum) -> {
            Item item = new Item();
            item.setId(rs.getLong("id"));
            item.setItemName(rs.getString("item_name"));
            item.setPrice(rs.getInt("price"));
            item.setQuantity(rs.getInt("quantity"));
            return item;
        });
    }
```
RowMapper 역시 함수형 인터페이스이다.
```java
@FunctionalInterface
public interface RowMapper<T> {
	/**
	 * Implementations must implement this method to map each row of data
	 * in the ResultSet. This method should not call {@code next()} on
	 * the ResultSet; it is only supposed to map values of the current row.
	 * @param rs the ResultSet to map (pre-initialized for the current row)
	 * @param rowNum the number of the current row
	 * @return the result object for the current row (may be {@code null})
	 * @throws SQLException if an SQLException is encountered getting
	 * column values (that is, there's no need to catch SQLException)
	 */
	@Nullable
	T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```
위와 같은 인터페이스이기 때문에 ResultSet과 int를 받고 구현 가능하다.
### Stream 병렬처리
```java
public class Main {
    public static void main(String[] args) {
        // 순차 처리
        Arrays.asList(1, 2, 3, 4, 5)
              .stream()
              .forEach(System.out::println);
        // 병렬 처리
        Arrays.asList(1, 2, 3, 4, 5)
              .parallelStream()
              .forEach(System.out::println);
    }
}
```


```java
@Slf4j
public class JdbcTemplateItemRepositoryV1 implements ItemRepository {

    private final JdbcTemplate template;

    public JdbcTemplateItemRepositoryV1(DataSource dataSource) {
        this.template = new JdbcTemplate(dataSource);
    }

    @Override
    public Item save(Item item) {
        String sql = "insert into item(item_name, price, quantity) values (?,?,?)";
        KeyHolder keyHolder = new GeneratedKeyHolder();
        template.update(connection -> {
            //자동 증가 키
            PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
            ps.setString(1, item.getItemName());
            ps.setInt(2, item.getPrice());
            ps.setInt(3, item.getQuantity());
            return ps;
        }, keyHolder);

        long key = keyHolder.getKey().longValue();
        item.setId(key);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "update item set item_name=?, price=?, quantity=? where id=?";
        template.update(sql,
                updateParam.getItemName(),
                updateParam.getPrice(),
                updateParam.getQuantity(),
                itemId);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "select id, item_name, price, quantity from item where id = ?";
        try {
            Item item = template.queryForObject(sql, itemRowMapper(), id);
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        String sql = "select id, item_name, price, quantity from item";
        //동적 쿼리
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }

        boolean andFlag = false;
        List<Object> param = new ArrayList<>();

        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',?,'%')";
            param.add(itemName);
            andFlag = true;
        }

        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= ?";
            param.add(maxPrice);
        }

        log.info("sql={}", sql);
        return template.query(sql, itemRowMapper(), param.toArray());
    }

    private RowMapper<Item> itemRowMapper() {
        return ((rs, rowNum) -> {
            Item item = new Item();
            item.setId(rs.getLong("id"));
            item.setItemName(rs.getString("item_name"));
            item.setPrice(rs.getInt("price"));
            item.setQuantity(rs.getInt("quantity"));
            return item;
        });
    }
}

```

위처럼 JdbcTemplate로 Repository를 구현했다.

>**itemRowMapper()**
데이터베이스의 조회 결과를 객체로 변환할 때 사용한다.
JDBC를 직접 사용할 때 `ResultSet` 를 사용했던 부분을 떠올리면 된다.
차이가 있다면 다음과 같이 JdbcTemplate이 다음과 같은 루프를 돌려주고, 개발자는 `RowMapper` 를 구현해서 그 내
부 코드만 채운다고 이해하면 된다. 
```java
while(resultSet 이 끝날 때 까지) { rowMapper(rs, rowNum)
}
```


# JdbcTemplate - 동적 쿼리 문제
결과를 검색하는 `findAll()` 에서 어려운 부분은 사용자가 검색하는 값에 따라서 실행하는 SQL이 동적으로 달려져야 한다는 점이다.
예를 들어서 다음과 같다.

검색 조건이 없음
```sql
select id, item_name, price, quantity from item 
```
상품명( `itemName` )으로 검색 
```sql
 select id, item_name, price, quantity from item
  where item_name like concat('%',?,'%')
```
최대 가격( `maxPrice` )으로 검색 
```sql
 select id, item_name, price, quantity from item
  where price <= ?
```
상품명( `itemName` ), 최대 가격( `maxPrice` ) 둘다 검색 
```sql
 select id, item_name, price, quantity from item
  where item_name like concat('%',?,'%')
and price <= ?
```

결과적으로 4가지 상황에 따른 SQL을 동적으로 생성해야 한다. 동적 쿼리가 언듯 보면 쉬워 보이지만, 막상 개발해보 면생각보다다양한상황을고민해야한다.예를들어서어떤경우에는 `where` 를앞에넣고어떤경우에는 `and` 를넣
             
어야 하는지 등을 모두 계산해야 한다.
그리고 각 상황에 맞추어 파라미터도 생성해야 한다.
물론 실무에서는 이보다 훨씬 더 복잡한 동적 쿼리들이 사용된다.
참고로 이후에 설명할 MyBatis의 가장 큰 장점은 SQL을 직접 사용할 때 동적 쿼리를 쉽게 작성할 수 있다는 점이다.

**로그 추가**
JdbcTemplate이 실행하는 SQL 로그를 확인하려면 `application.properties` 에 다음을 추가하면 된다.
`main` , `test` 설정이 분리되어 있기 때문에 둘다 확인하려면 두 곳에 모두 추가해야 한다.

```
 #jdbcTemplate sql log
 logging.level.org.springframework.jdbc=debug
```



# JdbcTemplate - 이름 지정 파라미터 
## 1 순서대로 바인딩
JdbcTemplate을 기본으로 사용하면 파라미터를 순서대로 바인딩 한다. 예를 들어서 다음 코드를 보자.

```java
 String sql = "update item set item_name=?, price=?, quantity=? where id=?";
 template.update(sql,
         itemName,
         price,
         quantity,
         itemId);
```
여기서는 `itemName` , `price` , `quantity` 가 `SQL에 있는 ?` 에 순서대로 바인딩 된다. 따라서 순서만 잘 지키면 문제가 될 것은 없다. 그런데 문제는 변경시점에 발생한다.

누군가 다음과 같이 SQL 코드의 순서를 변경했다고 가정해보자. ( `price` 와 `quantity` 의 순서를 변경했다.) 
```java
 String sql = "update item set item_name=?, quantity=?, price=? where id=?";
 template.update(sql,
         itemName,
         price,
         quantity,
         itemId);
```
이렇게 되면 다음과 같은 순서로 데이터가 바인딩 된다. 
`item_name=itemName, quantity=price, price=quantity`

**개발을 할 때는 코드를 몇줄 줄이는 편리함도 중요하지만, 모호함을 제거해서 코드를 명확하게 만드는 것이 유지보수 관 점에서 매우 중요하다.**

### 이름 지정 바인딩
JdbcTemplate은 이런 문제를 보완하기 위해 `NamedParameterJdbcTemplate` 라는 이름을 지정해서 파라미터를 바인딩 하는 기능을 제공한다.

```java
@Slf4j
public class JdbcTemplateItemRepositoryV2 implements ItemRepository {

    private final NamedParameterJdbcTemplate template;

    public JdbcTemplateItemRepositoryV2(DataSource dataSource) {
        this.template = new NamedParameterJdbcTemplate(dataSource);
    }

    @Override
    public Item save(Item item) {
        String sql = "insert into item(item_name, price, quantity) " +
                "values (:itemName, :price, :quantity)";

        SqlParameterSource param = new BeanPropertySqlParameterSource(item);

        KeyHolder keyHolder = new GeneratedKeyHolder();
        template.update(sql, param, keyHolder);

        long key = keyHolder.getKey().longValue();
        item.setId(key);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "update item " +
                "set item_name=:itemName, price=:price, quantity=:quantity " +
                "where id=:id";

        SqlParameterSource param = new MapSqlParameterSource()
                .addValue("itemName", updateParam.getItemName())
                .addValue("price", updateParam.getPrice())
                .addValue("quantity", updateParam.getQuantity())
                .addValue("id", itemId); //이 부분이 별도로 필요하다.

        template.update(sql, param);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "select id, item_name, price, quantity from item where id = :id";
        try {
            Map<String, Object> param = Map.of("id", id);
            Item item = template.queryForObject(sql, param, itemRowMapper());
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        SqlParameterSource param = new BeanPropertySqlParameterSource(cond);

        String sql = "select id, item_name, price, quantity from item";
        //동적 쿼리
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }

        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',:itemName,'%')";
            andFlag = true;
        }

        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= :maxPrice";
        }

        log.info("sql={}", sql);
        return template.query(sql, param, itemRowMapper());
    }

    private RowMapper<Item> itemRowMapper() {
        return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원
    }
}

```

**save()**
SQL에서 다음과 같이 `?` 대신에 `:파라미터이름` 을 받는 것을 확인할 수 있다.
```sql
 insert into item (item_name, price, quantity) " +
              "values (:itemName, :price, :quantity)"
```
추가로 `NamedParameterJdbcTemplate` 은 데이터베이스가 생성해주는 키를 매우 쉽게 조회하는 기능도 제공해준 다.


RowMaper도

```java
BeanPropertyRowMapper.newInstance(Item.class);
```

로 구현을 끝낼 수 있다.

### 이름 지정 파라미터
파라미터를 전달하려면 `Map` 처럼 `key` , `value` 데이터 구조를 만들어서 전달해야 한다.
여기서 `key` 는 `:파리이터이름` 으로 지정한, 파라미터의 이름이고 , `value` 는 해당 파라미터의 값이 된다.
다음 코드를 보면 이렇게 만든 파라미터( `param` )를 전달하는 것을 확인할 수 있다.


`template.update(sql, param, keyHolder);`

이름 지정 바인딩에서 자주 사용하는 파라미터의 종류는 크게 3가지가 있다. 
- `Map`
- `SqlParameterSource`
   - `MapSqlParameterSource` 
   -  `BeanPropertySqlParameterSource`
   
   
![](https://velog.velcdn.com/images/jckim22/post/a378de76-3de0-4892-9000-8c1da6ebe026/image.png)

- 여기서 보면 `BeanPropertySqlParameterSource` 가 많은 것을 자동화 해주기 때문에 가장 좋아보이지만, `BeanPropertySqlParameterSource` 를 항상 사용할 수 있는 것은 아니다.
- 예를 들어서 `update()` 에서는 SQL에 `:id` 를 바인딩 해야 하는데, `update()` 에서 사용하는 `ItemUpdateDto` 에는 `itemId` 가 없다. 따라서 `BeanPropertySqlParameterSource` 를 사용할 수 없고, 대신에 `MapSqlParameterSource` 를 사용했다.

### BeanPropertyRowMapper

`BeanPropertyRowMapper` 는 `ResultSet` 의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환한다.
예를 들어서 데이터베이스에서 조회한 결과가 `select id, price` 라고 하면 다음과 같은 코드를 작성해준다. (실제
로는 리플렉션 같은 기능을 사용한다.) 
```java
 Item item = new Item();
 item.setId(rs.getLong("id"));
item.setPrice(rs.getInt("price"));
```

데이터베이스에서 조회한 결과 이름을 기반으로 `setId()` , `setPrice()` 처럼 자바빈 프로퍼티 규약에 맞춘 메서드 를 호출하는 것이다.


> #### 별칭, 관계의 불일치
`snake_case` 는 자동으로 해결되니 그냥 두면 되고, 컬럼 이름과 객체 이름이 완전히 다른 경우에는 조회 SQL에서 별칭을 사용하면 된다.

# JdbcTemplate - SimpleJdbcInsert
JdbcTemplate은 INSERT SQL를 직접 작성하지 않아도 되도록 `SimpleJdbcInsert` 라는 편리한 기능을 제공한다.


- `withTableName` : 데이터를 저장할 테이블 명을 지정한다.
- `usingGeneratedKeyColumns` : `key` 를 생성하는 PK 컬럼 명을 지정한다.
- `usingColumns` : INSERT SQL에 사용할 컬럼을 지정한다. 특정 값만 저장하고 싶을 때 사용한다. 생략할 수 있다.

>`SimpleJdbcInsert` 는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다. 따라서 어떤 컬럼이 있는지 확인 할 수 있으므로 `usingColumns` 을 생략할 수 있다. 만약 특정 컬럼만 지정해서 저장하고 싶다면 `usingColumns` 를 사용하면 된다.

**save()**
`jdbcInsert.executeAndReturnKey(param)` 을 사용해서 INSERT SQL을 실행하고, 생성된 키 값도 매우 편
리하게 조회할 수 있다.
```sql
 public Item save(Item item) {
     SqlParameterSource param = new BeanPropertySqlParameterSource(item);
     Number key = jdbcInsert.executeAndReturnKey(param);
     item.setId(key.longValue());
     return item;
}
```


## 기능 정리


### 주요 기능
JdbcTemplate이 제공하는 주요 기능은 다음과 같다.
- `JdbcTemplate` 순서 기반 파라미터 바인딩을 지원한다. 
- `NamedParameterJdbcTemplate` 이름 기반 파라미터 바인딩을 지원한다. (권장) 
- `SimpleJdbcInsert` INSERT SQL을 편리하게 사용할 수 있다.
- `SimpleJdbcCall` 스토어드 프로시저를 편리하게 호출할 수 있다.


>
스토어드 프로시저를 사용하기 위한 `SimpleJdbcCall` 에 대한 자세한 내용은 다음 스프링 공식 메뉴얼을 참고 하자. https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-simple-jdbc-call-1


## 조회
**단건 조회 - 숫자 조회** 
```java
 int rowCount = jdbcTemplate.queryForObject("select count(*) from t_actor",
Integer.class); 
```
        
  하나의 로우를 조회할 때는 `queryForObject()` 를 사용하면 된다. 지금처럼 조회 대상이 객체가 아니라 단순 데이터 하나라면 타입을 `Integer.class` , `String.class` 와 같이 지정해주면 된다.
**단건 조회 - 숫자 조회, 파라미터 바인딩** 
```java
 int countOfActorsNamedJoe = jdbcTemplate.queryForObject(
         "select count(*) from t_actor where first_name = ?", Integer.class,
"Joe"); 
```
숫자 하나와 파라미터 바인딩 예시이다.
**단건 조회 - 문자 조회**
```java
 String lastName = jdbcTemplate.queryForObject(
         "select last_name from t_actor where id = ?",
         String.class, 1212L);
```
문자 하나와 파라미터 바인딩 예시이다.
**단건 조회 - 객체 조회**
```java
 Actor actor = jdbcTemplate.queryForObject(
         "select first_name, last_name from t_actor where id = ?",
         (resultSet, rowNum) -> {    
    Actor newActor = new Actor();
    newActor.setFirstName(resultSet.getString("first_name"));
    newActor.setLastName(resultSet.getString("last_name"));
    return newActor;
}, 1212L);
 ```
 객체 하나를 조회한다. 결과를 객체로 매핑해야 하므로 `RowMapper` 를 사용해야 한다. 여기서는 람다를 사용했다. 
 **목록 조회 - 객체**
```java
 List<Actor> actors = jdbcTemplate.query(
         "select first_name, last_name from t_actor",
         (resultSet, rowNum) -> {
             Actor actor = new Actor();
             actor.setFirstName(resultSet.getString("first_name"));
             actor.setLastName(resultSet.getString("last_name"));
             return actor;
 });
여러 로우를 조회할 때는 `query()` 를 사용하면 된다. 결과를 리스트로 반환한다.
결과를 객체로 매핑해야 하므로 `RowMapper` 를 사용해야 한다. 여기서는 람다를 사용했다.
```
  
**목록 조회 - 객체** 
```java
 private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
     Actor actor = new Actor();
     actor.setFirstName(resultSet.getString("first_name"));
     actor.setLastName(resultSet.getString("last_name"));
     return actor;
 };
 public List<Actor> findAllActors() {
     return this.jdbcTemplate.query("select first_name, last_name from t_actor",
 actorRowMapper);
 }
```
여러 로우를 조회할 때는 `query()` 를 사용하면 된다. 결과를 리스트로 반환한다. 여기서는 `RowMapper` 를 분리했다. 이렇게 하면 여러 곳에서 재사용 할 수 있다.




### 변경(INSERT, UPDATE, DELETE)
데이터를 변경할 때는 `jdbcTemplate.update()` 를 사용하면 된다. 참고로 `int` 반환값을 반환하는데, SQL 실행 결과에 영향받은 로우 수를 반환한다.

**등록**
```java
 jdbcTemplate.update(
         "insert into t_actor (first_name, last_name) values (?, ?)",
         "Leonor", "Watling");
```
**수정** 
```java
 jdbcTemplate.update(
         "update t_actor set last_name = ? where id = ?",
         "Banjo", 5276L);
```
**삭제** 
```java
 jdbcTemplate.update(
         "delete from t_actor where id = ?",
         Long.valueOf(actorId));
```





## 정리
실무에서 가장 간단하고 실용적인 방법으로 SQL을 사용하려면 JdbcTemplate을 사용하면 된다.
JPA와 같은 ORM 기술을 사용하면서 동시에 SQL을 직접 작성해야 할 때가 있는데, 그때도 JdbcTemplate을 함께 사용하면 된다.
그런데 JdbcTemplate의 최대 단점이 있는데, 바로 동적 쿼리 문제를 해결하지 못한다는 점이다. 그리고 SQL을 자바 코드로 작성하기 때문에 SQL 라인이 코드를 넘어갈 때 마다 문자 더하기를 해주어야 하는 단점도 있다.
동적 쿼리 문제를 해결하면서 동시에 SQL도 편리하게 작성할 수 있게 도와주는 기술이 바로 MyBatis 이다.
   
   


