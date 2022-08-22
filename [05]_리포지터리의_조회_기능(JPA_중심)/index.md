# #5 리포지터리의 조회 기능(JPA 중심)

## 검색을 위한 스펙

- 리포지터리

  - 리포지터리는 애그리거트의 저장소이다.
  - 애그리거트를 저장하고 찾고 삭제하는 것이 리포지터리의 기본 기능이다.

- 스펙 활용하기

  - 검색 조건의 조합이 다양해지면 모든 조합별로 find 메서드를 정의할 수 없다.

  - 검색 조건이 다양할 경우 스펙을 활용해야한다.

  - 스펙은 애그리거트가 특정 조건을 충족하는지 여부를 검사한다

  - ```java
    public interface Specification<T> {
      public boolean isSatisfiedBy(T agg);
    }
    
    public class OrdererSpec implements Specification<Order> {
    	private String ordererId;
    
    	public boolean isSatisfiedBy(Order agg) {
    		return agg.getOrdererId().getMemberId().getId().equals(ordererId);
    	}
    }
    ```

- 리포지터리는 스펙을 전달받아 애그리거트를 걸러내는 용도로 사용한다.

```java
public class MemoryOrderRepository implements OrderRepository {
  public List<Order> findAll(Specification spec) {
    List<Order> allOrders = findAll();
    return allOrders.stream().filter(order -> spec.isSatisfiedBy(order)).collect(toList());
  }
}


// 사용
Specification<Order> ordererSpec = new OrdererSpec("madvirus");
List<Order> orders = orderRepository.findAll(ordererSpec);
```



### 스펙 조합

- 스펙의 장점
  - 두 스펙을 AND 연산자나 OR 연산자로 조합해서 새로운 스펙을 만들 수 있다.
  - 조합한 스펙을 다시 조합해서 더 복잡한 스펙을 만들 수 있다.

```java
public class AndSpec<T> implements Specification<T> {
  private List<Specification<T>> specs;

  public AndSpecification(Specification<T> ... specs) {
    this.specs = Arrays.asList(specs);
  }

  public boolean isSatisfiedBy(T agg) {
    for (Specification<T> spec : specs) {
      if (!spec.isSatisfiedBy(agg)) return false;
    }
    return true
  }
}

// 사용

Specification<Order> ordererSpec = new OrdererSpec("madVirus");
Specification<Order> orderDateSpec = new OrderDateSpec(fromDate, toDate);
AndSpec<T> spec = new AndSpec(ordererSpec, orderDateSpec);
List<Order> orders = orderRepository.findAll(spec);

```



## JPA를 위한 스펙 구현

- 스펙만 사용하기?
  - 단순히 스펙만 사용한다면 리포지터리 코드는 모든 애그리거트를 조회한 다음에 스펙을 이용해 걸러내는데 애그리거트가 10만개일 경우 10만개를 메모리에서 검사하게된다. 이는 성능상 매우 비효율적이다.
  - 따라서 실제 구현에서는 쿼리의 where 절에 조건을 붙여서 필요한 데이터를 걸러야 한다.
- Criteria-Builder와 Predicate 사용하기
  - JPA는 다양한 검색 조건을 조합하기 위해 Criteria-Builder와 Predicate를 사용하므로 JPA를 위한 스펙은 CriteriaBuilder와  Predicate를 사용해서 검색 조건을 구현해야 한다.

### JPA 스펙 구현

- JPA를 사용하는 리포지터리를 위한 스펙의 인터페이스는 아래와 같이 정의할 수 있다.

```java
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;

public interface Specification<T> {
  Predicate toPredicate(Root<T> root, CriteriaBuilder cb);
}

// 구현

public class OrdererSpec implements Specification<Order> {
  private String ordererId;

  public OrdererSpec(String ordererId) {
    this.ordererId = ordererId;
  }

  @Override
  public Predicate toPredicate(Root<Order> root, CriteriaBuilder cb) {
    return cb.equal(root.get(Order_.orderer)
                    .get(Ordrer_.memberId).get(MemberId_.id), ordereId);
  }
}
```

- 응용 서비스는 원하는 스펙을 생성하고 리포지터리에 전달해서 필요한 애그리거트를 검색하면 된다.

```kotlin
Specification<Order> ordererSpec = new OrdererSpec("madVirus");
List<Order> orders = orderRepository.findAll(ordererSpec);
```



### AND/OR 스펙 조합을 위한 구현

- JPA를 위한 AND 스펙

```java
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;
import java.util.Arrays;
import java.util.List;

public class AndSpecification<T> implements Specification<T> {
  private List<Specification<T>> specs;

  public AndSpecification(Specification<T> ... specs) {
    this.specs = Arrays.asList(specs);
  }
  public Predicate toPredicate(Root<T> root, CriteriaBuilder cb) {
    Pridicate[] predicates = specs.stream()
      .map(spec -> spec.toPredicate(root, cb))
      .toArray(size -> new Predicate[size]);
    return cb.and(predicates);    
  }
}
```

- JPA를 위한 OR 스펙

```java
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;
import java.util.Arrays;
import java.util.List;

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

- 스펙을 생성해주는 팩토리 클래스

```kotlin
public class Specs {
  public static <T> Specification<T> and(Specification<T> ... specs) {
    return new AndSpecification<>(specs);
  }
  public static <T> Specification<T> or(Specification<T> ... specs) {
    return new OrSpecification<>(specs);
  }
}
```

- 스펙 조합하기

```java
Specification<Order> specs = Specs.and(
  OrderSpecs.orderer("madvirus"), OrderSpecs.between(fromTime, toTime)
);
```



### 스펙을 사용하는 JPA 리포지터리 구현

- 이제 남은 작업은 스펙을 사용하도록 리포지터리를 구현하는 것이다.

```java
public interface OrderRepository {
  public List<Order> findAll(Specification<Order> spec);
  //...
}

@Repository
public class JpaOrderRepository implements OrderRepository {
  //...
  @Override
  public List<Order> findAll(Specification<Order> spec) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Order> criteriaQuery = cb.createQuery(Order.class);
    Root<Order> root = criteriaQuery.from(Order.class); //  검색 조건 대상이 되는 루트(Order 클래스) 생성
    Predicate predicate = spec.toPredicate(root, cb);   //  파라미터로 전달받은 스펙을 이용해서 Predicate 생성
    criteriaQuery.where(predicate);                     //  쿼리의 조건으로 생성한 Predicate를 전달
    criteriaQuery.orderBy(
      cb.desc(root.get(Order_.number).get(OrderNo_.number))
    );
    TypedQuery<Order> query = entityManager.createQuery(criteriaQuery);
    return query.getResultList();
  }
}
```

- 리포지터리 구현 기술 의존
  - "도메인 모델은 구현 기술에 의존하지 않아야 한다."
  - 하지만 JPA의 Root와 CriteriaBuilder에 의존하고 있으므로 사용하는 리포지터리 인터페이스는 이미 JPA에 의존하는 모양이 된다.
- 그렇다면 JPA에 의존하지 않는 완전히 중립적인 형태로 구현해서 도메인 구현 기술에 완전히 의존하지 않도록 만들어야하나?
  - "NO"
  - 리포지터리 구현 기술에 의존하지 않게하려면 많은 부분을 추상화해야 하는데 이는 많은 노력을 요구한다.
  - 실제로 추상화를 하더라도 리포지터리 구현 기술을 바꿀정도의 변화는 매우 드물기때문에 큰 이점이 없다.



## 정렬 구현

- JPQL을 사용하는 경우에는 JPQL의 order by 절을 사용하면 된다.

```java
TypedQuery<Order> query = entityManager.createQuery(
        "select o from Order o " +
        "where o.orderer.memberId.id = :ordererId " + 
        "order by o.number.number desc",
    Order.class);
```

- 정렬 순서를 지정하는 가장 쉬운 방법은 다음과 같이 문자열을 사용하는 것이다.

```java
List<Order> orders = orderRepository.findAll(someSpec, "number.number desc");
```

- JPA 리포지터리 구현 클래스는 문자열을 파싱해서 JPA Criteria의 Order로 변환하거나 JQPL의  order by로 변환하면 된다.

```java
@Repository
public class JpaOrderRepository implements OrderRepository {
  //...

  @Override
  public List<Order> findAll(Specification<Order> spec, String ... orders) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Order> criteriaQuery = cb.createQuery(Order.class);
    Root<Order> root = criteriaQuery.from(Order.class);
    Predicate predicate = spec.toPredicate(root, cb);
    criteriaQuery.where(predicate);
    if (orders.length > 0) {
      criteriaQuery.orderBy(JpaQueryUtils.toJpaOrders(root, cb, orders));
    }
    TypedQuery<Order> query = entityManager.createQuery(criteriaQuery);
    return query.getResultList();
  }
}

public class JpaQueryUtils {
  public static <T> List<Order> toJpaOrders(Root<T> root, CriteriaBuilder cb, String ... orders) {
    String[] orderClause = orderStr.split(" ");
    boolean ascending = true;
    if (orderClause.length == 2 && orderClause[1].equalsIgnoreCase("desc")) {
      ascending = false;
    }

    String[] paths = orderClause[0].split("\\.");
    Path<Object> path = root.get(paths[0]);
    for (int i = 1; i < paths.length; i++) {
      path = path.get(paths[i]);
    }
    return ascending ? cb.asc(path) : cb.desc(path);
  }
}
```



## 페이징 개수 구하기 구현

- JPA 쿼리는 setFirstResult()와 setMaxResults() 메서드를 제공하고 있는데 이 두 메서드를 이용해서 페이징을 구현할 수 있다.

```java
@Override
public List<Order> findByOrdererId(String ordererId, int startRow, int fetchSize) {
  TypedQuery<Order> query = entityManager.createQuery(
    "select o from Order o " +
    "where o.orderer.memberId.id = :ordererId " +
    "order by o.number.number desc"), Order.class);
  )

  query.setParameter("ordererId", ordererId);
  query.setFirstResult(startRow); //  읽어올 첫 번째 행 번호를 지정
  query.setMaxResults(fetchSize); //  읽어올 행 개수를 지정
  return query.getResultList();
}

List<Order> orders = findByOrdererId("madvirus", 45, 15);
```



- **그냥 Spring Data JPA를 사용하자.**
  - 스펙, 정렬, 페이징을 위한 구현 코드는 모드 Spring Data JPA에 이미 구현되어 있다.
  - 따라서 직접 구현하기전에 이미 검증되고 많은곳에서 사용되는 오픈소스가 있다면 도입해보는게 좋다고 생각한다. (개인적인 생각)



## 조회 전용 기능 구현

- 다음 용도로 리포지터리를 사용하는 것은 적합하지 않다.
  - 여러 애그리거트를 조합해서 한화면에 보여주는 데이터 제공
  - 각종 통계 데이터 제공
- 그럼 어떻게?
  - 이런 기능은 조회 전용 쿼리로 처리해야 하는 것들이다.
  - JPA와 하이버네이트를 사용하면 동적 인스턴스 생성, 하이버네이트의 @Subselect 확장 기능, 네이티브 쿼리를 이용해서 조회 전용 기능을 구현할 수 있다.



### 동적 인스턴스 생성

- JPA는 쿼리 결과에서 임의의 객체를 동적으로 생성할 수 있는 기능을 제공하고 있다.

```java
@Repository
public class JpaOrderViewDao implements OrderViewDao {
  //...
  @Override
  public List<OrderView> selectByOrderer(String ordererId) {
    String selectQuery =
      "select new com.myshop.order.application.dto.OrderView(o, m, p) " +
      "from Order o join o.orderLines ol, Member m, Product p " +
      "where o.orderer.memberId.id = :ordererId " +
      "and o.orderer.memberId = m.id " +
      "and index(ol) = 0 " +
      "and ol.productId = p.id " +
      "order by o.number.number desc";
    TypedQuery<OrderView> query =
      em.createQuery(selectQuery, OrderView.class);
    query.setParameter("ordererId", ordererId);
    return query.getResultList();
  }
}

// 실제 호출되는 OrderView의 생성자
	public class OrderView(Order order,Member member,Product product){
		this.number = order.getNumber().getNumber(); //매핑
		this.totalAmounts = order.getTotalAmounts().getValue();
		...
		this.productName = product.getName();
	}

```

- 조회 전용 모델을 만드는 이유는 표현 영역을 통해 사용자에게 데이터를 보여주기 위함이다.
- 동적 인스턴스의 장점은 JPQL을 그대로 사용하므로 객체 기준으로 커리를 작성하면서도 동시에 지연/즉시 로딩과 같은 고민 없이 원하는 모습으로 데이터를 조회할 수 있다는 점이다.



### 하이버네이트 @Subselect 사용

- 하이버네이트는 JPA 확장 기능으로 @Subslect를 제공한다.
- @Subselect는 쿼리 결과를 @Entity로 매핑할 수 있는 유용한 기능이다.

```java
@Entity
@Immutable
@Subselect("select o.order_number as number, " +
				"o.orderer_id, o.orderer_name, o.total_amounts, " + 
				"o.receiver_name, o.state, o.order_date, " +
				"p.product_id, p.name as product_name" +
					"from purchase_order o inner join order_line ol" +
					"  on o.order_number = ol.order_number" +
					"  cross join product p" +
						"where ol.line_idx = 0 and ol.product_id = p.product_id"
)

@Synchronize({"purchase_order", "order_line", "product"})
public class OrderSummary{
	@Id
	private String number;
	private String ordererId;
	private String ordererName;
	private int totalAmounts;
	...
	private String productName

	protected OrderSummary(){}
	//..get 메서드
}
```

- @Subselect
  - 하이버네이트는 @Subselect를 사용하는 쿼리의 결과를 뷰처럼 사용한다.
  - 뷰를 수정할 수 없듯이 @subselect로 조회한 엔티티는 수정할 수 없다.
- @Immutable
  - 만약 수정을 한다고하더라도 실제 매핑되는 테이블은 없기 때문에 에러가 발생하는데 이런 문제를 방지하기 위해 @Immutable을 사용한다.
  - @Immutable을 사용한 엔티티는 해당 필드/프로퍼티가 변경되어도 DB에 반영하지 않고 무시한다.
- @Synchronize
  - @Synchronize에는 해당 엔티티와 관련된 테이블 목록을 명시한다.
  - 하이버네이트는 엔티티를 로딩하기 전에 지정한 테이블과 관련된 변경이 발생하면 플러시를 먼저하고 반영이 완료된 데이터를 가져온다.
