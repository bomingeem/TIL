4장. 리포지터리와 모델구현(JPA 중심)
===========

### 리포지터리 기본 기능 구현

- 인터페이스는 애그리거트 루트를 기준으로 작성한다.
- 주문 애그리거트는 Order 루트 엔티티를 비롯해 OrderLine, Orderer, ShoppingInfo 등 다양한 객체를 포함하는데, 이 구성요소 중에서 Order 루트 엔티티를 기준으로 리포지터리 인터페이스를 작성한다.

## 매핑 구현

### 엔티티와 밸류 기본 매핑 구현

- 애그리거트 루트는 엔티티이므로 **@Entity로 매핑 설정**한다.
- 한 테이블에 엔티티와 밸류 데이터가 같이 있다면 **밸류는 @Embeddable로 매핑 설정**, **밸류 타입 프로퍼티는 @Embedded로 매핑 설정**한다.

### 기본 생성자

- JPA의 @Entity와 @Embeddable로 클래스를 매핑하려면 기본 생성자를 제공해야 한다. 하이버네이트와 같은 JPA 프로바이더는 DB에서 데이터를 읽어와 매핑된 객체를 생성할 때 기본 생성자를 사용해서 객체를 생성한다.
- 기본 생성자는 JPA 프로바이더가 객체를 생성할 때만 사용한다. 기본 생성자를 다른 코드에서 사용하면 값이 없는 온전하지 못한 객체를 만들게 된다.

### 필드 접근 방식 사용

- 엔티티가 객체로서 제 역할을 하려면 외부에 set 메서드 대신 의도가 잘 드러나는 기능을 제공해야 한다.
- 상태 변경을 위한 setState() 메서드보다 주문 취소를 위한 cancel() 메서드가 도메인을 더 잘 표현하고, setShippingInfo() 메서드보다 배송지를 변경한다는 의미를 갖는 changeShippingInfo()가 도메인을 더 잘 표현한다.
- 엔티티를 객체가 제공할 기능 중심으로 구현하도록 유도하려면 프로퍼티 방식이 아닌 필드 방식으로 선택해서 불필요한 get/set 메서드를 구현하지 말아야 한다.

### AttributeConverter를 이용한 밸류 매핑 처리

- **NOTE: 이 부분의 경우 일급컬렉션으로 가지 않는 이상 컨버터 기능을 쓰지 않아도 되지 않을까? 싶다.**

```java
@Converter(autoApply = true)
public class MoneyConverter implements AttributeConverter<Money, Integer> {

    @Override
    public Integer convertToDatabaseColumn(Money money) {
        if (money == null)
            return null;
        else
            return money.getValue();
    }

    @Override
    public Money convertToEntityAttribute(Integer value) {
        if (value == null) return null;
        else return new Money(value);
    }
}
```

### 밸류 컬렉션: 별도 테이블 매핑

- 밸류 타입의 컬렉션은 별도 테이블에 보관한다.

  ![4.4.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ad25dbc-59b0-46a1-95d7-4e3cd11086a1/4.4.png)

    - 밸류 컬렉션을 별도 테이블로 매핑할 때는 @ElementCollection과 @CollectionTable을 함께 사용한다.

### 밸류 컬렉션: 한 개 컬럼 매핑

- AttributeConverter를 사용하면 밸류 컬렉션을 한 개 컬럼에 쉽게 매핑할 수 있다.
- **TODO: 의문점이 말이 @Converter를 오버라이딩 한 함수를 제공하는 것 뿐이지 내부 컨버터 관련 로직은 직접 구현해야 하는 것인데 그러면 Util성의 Converter 메서드를 사용하는 것과 특별한 차이가 없지 않을까?**

### 밸류를 이용한 아이디 매핑

- 기본 타입을  사용하는 것이 나쁘진 않지만 식별자라는 의미를 부각시키기 위해 식별자 자체를 별도 밸류 타입으로 만들 수도 있다.
- 밸류 타입을 식별자로 매핑하면 @Id 대신 @EmbeddedId 애노테이션을 사용한다.
- **TODO: 이것도 일급 컬렉션으로 봐도 되는걸까?**
- JPA에서 식별자 타입은 Serializable 타입이어야 하므로 식별자로 사용될 밸류 타입은 Serializable 인터페이스를 상속받아야 한다.
- JPA는 내부적으로 엔티티를 비교할 목적으로 equals() 메서드와 hashcode() 값을 사용하므로 식별자로 사용할 밸류 타입은 이 두 메서드를 알맞게 구현해야 한다.

### 별도 테이블에 저장하는 밸류 매핑

- 애그리거트에서 루트 엔티티를 뺀 나머지 구성요소는 대부분 밸류이다. 단지 별도 테이블에 데이터를 저장한다고 해서 엔티티인 것은 아니다.
- 애그리거트에 속한 객체가 밸류인지 엔티티인지 구분하는 방법은 고유 식별자를 갖는지 여부를 확인하는 것이다.
- 게시글 목록을 보여주는 화면은 articel 테이블의 데이터만 필요하지 article_content 테이블의 데이터는 필요하지 않는다. 그런데, @SecondaryTable을 사용하면 목록 화면에 보여줄 article을 조회 할 때 article_content 테이블까지 조인해서 데이터를 읽어오는데 이는 원하는 결과가 아니다.
- 이 문제를 해소하고자 ArticleContent를 엔티티로 매핑하고 Article에서 ArticleContent로의 로딩을 지연 로딩 방식으로 설정할 수도 있지만 이 방식은 엔티티가 아닌 모델을 엔티티로 만드는 것이므로 좋은 방법이 아니다.

### 밸류 컬렉션을 @Entity로 매핑하기

- 개념적으로 밸류인데 구현 기술의 한계나 팀 표준 때문에 @Entity를 사용해야 할 때도 있다.
- JPA는 @Embeddable 타입의 클래스 상속 매핑을 지원하지 않는다. 따라서 상속 구조를 갖는 밸류 타입을 사용하려면 @Embeddable 대신 @Entity를 이용한 상속 매핑으로 처리해야 한다.

### ID 참조와 조인 테이블을 이용한 단방향 M:N 매핑

- 애그리거트 간 집합 연관은 성능상의 이유로 피해야 한다고 하였지만 요구사항을 구현하는 데 집합 연관을 사용하는 것이 유리하다면 ID 참조를 이용한 단방향 집합 연관을 적용해 볼 수 있다.

```java
@Entity
@Table(name = "product")
public class Product {
    @EmbeddedId
    private ProductId id;

    **@ElementCollection**
    @CollectionTable(name = "product_category",
            joinColumns = @JoinColumn(name = "product_id"))
    private **Set<CategoryId> categoryIds;**
}
```

- 차이점이 있다면, 집합의 값에 밸류 대신 연관을 맺는 식별자가 온다는 점이다.
- Product를 삭제할 때 매핑에 사용한 조인 테이블의 데이터고 함께 삭제된다. 애그리거트를 직접 참조하는 방식을 사용했다면 영속성 전파나 로딩 전략을 고민해야 하는데 ID 참조 방식을 사용함으로써 이런 고민을 할 필요가 없다.
- **TODO: 이 부분은 이해하는데 조금 어려움이 있다..!**
