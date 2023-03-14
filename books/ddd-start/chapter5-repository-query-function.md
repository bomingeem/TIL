5장. 리포지터리의 조회 기능(JPA 중심)
===========

### 검색을 위한 스펙

- 검색 조건의 조합이 다양할 경우 스펙을 이용해서 문제를 풀어야 한다.
- 스펙(Specification)은 애그리거트가 특정 조건을 충족하는지 여부를 검사한다.
- 리포지터리는 스펙을 전달받아 애그리거트를 걸러내는 용도로 사용한다.
- 특정 조건을 충족하는 애그리거트를 찾으려면 이제 원하는 스펙을 생성해서 리포지터리에 전달해 주기만 하면 된다.

### 스펙 조합

- 스펙의 장점은 조합에 있다. 두 스펙을 AND 연산자나 OR 연산자로 조합해서 새로운 스펙을 만들 수 있고, 조합한 스펙을 다시 조합해서 더 복잡한 스펙을 만들 수 있다.
- AndSpec을 이용하면 다음과 같이 여러 스펙을 하나의 스펙으로 만들어 리포지터리에 전달할 수 있다.
- OrSpec도 비슷하게 구현할 수 있다.

```java
Specification<Order> ordererSpec = new OrdererSpec("madvirus");
Specification<Order> orderDateSpec = new orderDateSpec(formData, toDate);
AndSpec<T> spec = new AndSpec(ordererSpec, orderDateSpec);
List<Order> orders = orderRepository.findAll(spec);
```

### JPA를 위한 스펙 구현

- 실제 구현에서는 쿼리의 where 절에 조건을 붙여서 필요한 데이터를 걸러야 한다.
- JPA를 위한 스펙은 CriteriaBuilder와 Predicate를 이용해서 검색 조건을 구현해야 한다.

### AND/OR 스펙 조합을 위한 구현

- AND 스펙

```java
public class AndSpecification<T> implements Specification<T> {
    private List<Specification<T>> specs;

    public AndSpecification(Specification<T> ... specs) {
        this.specs = Arrays.asList(specs);
    }

    @Override
    public Predicate toPredicate(Root<T> root, CriteriaBuilder cb) {
        Predicate[] predicates = specs.stream()
                .map(spec -> spec.toPredicate(root, cb))
                .toArray(size -> new Predicate[size]);
        return cb.and(predicates);
    }
}
```

- OR스펙

```java
public class OrSpecification<T> implements Specification<T> {
    private List<Specification<T>> specs;

    public OrSpecification(Specification<T>... specs) {
        this.specs = Arrays.asList(specs);
    }

    @Override
    public Predicate toPredicate(Root<T> root, CriteriaBuilder cb) {
        Predicate[] predicates = specs.stream()
                .map(spec -> spec.toPredicate(root, cb))
                .toArray(Predicate[]::new);
        return cb.or(predicates);
    }
}
```

- AndSpecification과 OrSpecification의 toPredicate() 메서드는 생성자로 전달받은 Specification 목록을 Predicate 목록으로 바꾸고 CriteriaBuilder의 and()와 or()를 사용해서 새로운 Predicate를 생성한다.

### 스펙을 사용하는 JPA 리포지터리 구현

- 리포지터리 인터페이스는 스펙을 사용하는 메서드를 제공해야 한다.

```java
public interface OrderRepository {
	public List<Order> findAll(Specification<Order> spec);
}

// 구현코드
@Override
    public List<Order> findAll(Specification<Order> spec) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Order> criteriaQuery = cb.createQuery(Order.class);
        Root<Order> root = criteriaQuery.from(Order.class);
        Predicate predicate = spec.toPredicate(root, cb);
        criteriaQuery.where(predicate);
        TypedQuery<Order> query = entityManager.createQuery(criteriaQuery);
        return query.getResultList();
    }
```

### 정렬 구현

- JPQL을 사용하는 경우에는 JPQL의 order by 절을 사용하면 된다.

```java
TypedQuery<Order> query = entityManager.createQuery(
                "select o from Order o " +
                        "where o.orderer.memberId.id = :ordererId " +
                        "order by o.number desc",
                Order.class);
```

### 페이징과 개수 구하기 구현

- JPA 쿼리는 setFirstResult()와 setMaxResults() 메서드를 제공한다.
- setFirstResult() 메서드는 읽어올 첫 번째 행 번호를 지정한다. 첫 행은 0번부터 시작한다
- setMaxResults() 메서드는 읽어올 행 개수를 지정한다.

```java
@Override
    public List<Order> findByOrdererId(String ordererId, int startRow, int fetchSize) {
        TypedQuery<Order> query = entityManager.createQuery(
                "select o from Order o " +
                        "where o.orderer.memberId.id = :ordererId " +
                        JpaQueryUtils.toJPQLOrderBy("o", "number.number desc"),
                Order.class);
        query.setParameter("ordererId", ordererId);
        **query.setFirstResult(startRow);
        query.setMaxResults(fetchSize);**
        return query.getResultList();
    }
```

### 조회 전용 기능 구현

- 리포지터리는 애그리거트의 저장소를 표현하는 것으로서 다음 용도로 리포지터리를 사용하는 것은 적합하지 않다.
1. 여러 애그리거트를 조합해서 한 화면에 보여주는 데이터 제공
  1. JPA의 지연 로딩과 즉시 로딩 설정, 연관 매핑으로 골치가 아플 것이며, 애그리거트 간에 직접 연관을 맺으면 ID로 참조할 때의 장점을 활용할 수 없게 된다.
2. 각종 통계 데이터 제공
  1. 다양한 테이블을 조인 및 DBMS 전용 기능을 사용해야 구할 수 있는데 JPQL이나 Criteria로 처리하기 힘들다.

### 동적 인스턴스 생성

- JPQL의 select 절에 new 키워드가 있다. new 키워드 뒤에 생성할 인스턴스의 완전한 클래스 이름을 지정하고 괄호 안에 생성자에 인자로 전달할 값을 지정한다.

```java
@Repository
public class JpaOrderViewDao implements OrderViewDao {
    @PersistenceContext
    private EntityManager em;

    @Override
    public List<OrderView> selectByOrderer(String ordererId) {
        String selectQuery =
                "select new com.myshop.order.query.dto.OrderView(o, m, p) "+
                "from Order o join o.orderLines ol, Member m, Product p " +
                "where o.orderer.memberId.id = :ordererId "+
                "and o.orderer.memberId = m.id "+
                "and index(ol) = 0 " +
                "and ol.productId = p.id "+
                "order by o.number.number desc";
        TypedQuery<OrderView> query =
                em.createQuery(selectQuery, OrderView.class);
        query.setParameter("ordererId", ordererId);
        return query.getResultList();
    }
}
```

- 조회 전용 모델을 만드는 이유는 표현 영역을 통해 사용자에게 데이터를 보여주기 위함이다.
- 동적 인스턴스의 장점은 JPQL을 그대로 사용하므로 객체 기준으로 쿼리를 작성하면서도 동시에 지연/즉시 로딩과 같은 고민을 원하는 모습으로 데이터를 조회할 수 있다는 점이다.

### 하이버네이트 @Subselect 사용

- 하이버네이트는 JPA 확장 기능으로 @Subselect를 제공한다.
- @Subselect는 쿼리 결과를 @Entity로 매핑할 수 있는 유용한 기능이다.

```java
@Entity
**@Immutable**
@Subselect("select o.order_number as number, " +
        "o.version, " +
        "o.orderer_id, " +
        "o.orderer_name, " +
        "o.total_amounts, " +
        "o.receiver_name, " +
        "o.state, " +
        "o.order_date, " +
        "p.product_id, " +
        "p.name as product_name " +
        "from purchase_order o inner join order_line ol " +
        "    on o.order_number = ol.order_number " +
        "    cross join product p " +
        "where " +
        "ol.line_idx = 0 " +
        "and ol.product_id = p.product_id"
)
@Synchronize({"purchase_order", "order_line", "product"})
public class OrderSummary {
    @Id
    private String number;
    private long version;
    private String ordererId;
    private String ordererName;
    private int totalAmounts;
    private String receiverName;
    private String state;
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "orderDate")
    private Date orderDate;
    private String productId;
    private String productName;

    protected OrderSummary() {
    }
}
```

- @Immutable, @Subselect, @Synchronize는 하이버네이트 전용 애노테이션인데 이 태그를 사용하면 테이블이 아닌 쿼리 결과를 @Entity로 매핑할 수 있다.
- DBMS가 여러 테이블을 조인해서 조회한 결과를 한 테이블처럼 보여주기 위한 용도로 뷰를 사용하는 것처럼 @Subselect를 사용하면 쿼리 실행 결과를 매핑할 테이블처럼 사용한다.
- 뷰를 수정할 수 없듯이 @Subselect로 조회한 @Entity 역시 수정할 수 없다. 실수로 수정하면 하이버네이트는 변경 내역을 반영하는 **update 쿼리를 실행**할 것이다. 그런데, 매핑할 테이블이 없으므로 에러가 발생한다.
- 이런 문제를 방지하기 위해 @Immutable을 사용한다. @Immutable을 사용하면 하이버네이트는 해당 엔티티의 매핑 필드/프로퍼티가 변경되어도 DB에 반영하지 않고 무시한다.
- @Synchronize는 해당 엔티티와 관련된 테이블 목록을 명시한다.
- 하이버네이트는 엔티티를 로딩하기 전에 지정한 테이블과 관련된 변경이 발생하면 플러시를 먼저한다.
- OrderSummary의 **@Synchronize**는 ‘purchase_order’ 테이블을 지정하고 있으므로 OrderSummary를 로딩하기 전에 purchase_order **테이블에 변경이 발생하면 관련 내역을 먼저 플러시**한다.
