7장. 도메인 서비스
===========

### 여러 애그리거트가 필요한 기능

- 도메인 영역의 코드를 작성하다 보면 한 애그리거트로 기능을 구현할 수 없을 때가 있다. 대표적인 예가 ‘결제 금액 계산 로직’이다.
- 생각해 볼 수 있는 방법은 주문 애그리거트가 필요한 애그리거트나 필요 데이터를 모두 가지도록 한 뒤 할인 금액 계산 책임을 주문 애그리거트에 할당하는 것이다.

```java
public class Order {
    private Orderer orderer;
    private List<OrderLine> orderLines;
    private List<Coupon> usedCoupons;
    
    private Money calculatePayAmounts() {
        Money totalAmounts = calculateTotalAmounts();
        // 쿠폰별로 할인 금액을 구한다.
        coupons.stream()
                .map(coupon -> calculateDiscount(coupon))
                .reduce(Money(0), (v1, v2) -> v1.add(v2));
        // 회원에 따른 추가 할인을 구한다.
        Money membershipDiscount = calculateDiscount(orderer.getMember().getGrade());
        // 실제 결제 금액 계산
        return totalAmounts.minus(discount).minus(membershipDiscount);
    }
    
    private Money calculateDiscount(Coupon coupon) {
        // orderLines의 각 상품에 대해 쿠폰을 적용해서 할인 금액 계산하는 로직
        // 쿠폰의 적용 조건 등을 확인하는 코드
        // 정책에 따라 복잡한 if-else와 계산 코드
    }
    
    private Money calculateDiscount(MemberGrade grade) {
        // 등급에 따라 할인 금액 계산
    }
}
```

- 여기서 고민거리는 결제 금액 계산 로직이 ‘주문 애그리거트의 책임이 맞느냐’에 대한 것이다.
- 이렇게 한 애그리거트에 넣기에 애매한 도메인 기능을 특정 애그리거트에서 억지로 구현하면 안된다.
    - 애그리거트는 자신의 책임 범위를 넘어서는 기능을 구현하기 때문에 코드가 길어지고 외부에 대한 의존이 높아지게 된다.
    - 애그리거트의 범위를 넘어서는 도메인 개념이 애그리거트에 숨어들어서 명시적으로 드러나지 않게 된다.

### 도메인 서비스

- 위와 같은 한 애그리거트에 넣기 애매한 도메인 개념을 구현하려면 ‘도메인 서비스’를 이용해서 도메인 개념을 명시적으로 드러내면 된다.
- 애그리거트나 밸류와 같은 다른 구성요소와 다른점은 ‘상태 없이 로직만 구현’한다는 점이다. 도메인 서비스를 구현하는 데 필요한 상태는 애그리거트나 다른 방법으로 전달받는다.

```java
public class DiscountCalculationService {
    public Money calculateDiscountAmounts(List<OrderLine> orderLines,
                                          List<Coupon> coupons,
                                          MemberGrade grade) {
        Money couponDiscount = coupons.stream()
                .map(coupont -> calculateDiscount(coupon))
                .reduce(Money(0), (v1, v2) -> v1.add(v2));
        Money membershipDiscount = calculateDiscount(orderer.getMember().getGrade());
        
        return couponDiscount.add(membershipDiscount);
    }
    private Money calculateDiscount(Coupon coupon) { }
    private Money calculateDiscount(MemberGrade grade) { }
}
```

```java
public class Order {
    public void calculateAmounts(DiscountCalculationService discountCalculationService,
                                 MemberGrade grade) {
        Money totalAmounts = getTotalAmounts();
        money discountAmounts = discountCalculationService.calculateDiscountAmounts(this.orderLines, this.coupons, grade);
        this.paymentAmounts = totalAmounts.multiply(discountAmounts);
        
    }
}
```

- 도메인 서비스는 도메인 로직을 수행하지 응용 로직을 수행하지는 않는다. 트랜잭션 처리와 같은 로직은 응용 로직이므로 응용 서비스에서 처리해야 한다.
    - 특정 기능이 응용 서비스인지 도메인 서비스인지 감을 잡이 어려울 때는 해당 로직이 ‘애그리거트의 상태를 변경’하거나 ‘애그리거트의 상태 값을 계산’하는지 검사해 보면 된다.
    - ex: 계좌 이체 로직은 계좌 애그리거트의 상태를 변경, 결제 금액 로직은 주문 애그리거트의 주문 금액을 계산

### 도메인 서비스의 패키지 위치

- 도메인 서비스의 위치는 다른 도메인 구성 요소와 동일한 패키지에 위치한다.

  ![7-domain-package.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a390bc1-394d-4aff-a755-d7d85bde03c6/7-domain-package.png)

  ### 도메인 서비스의 인터페이스와 클래스

    - 도메인 서비스의 로직이 고정되어 있지 않은 경우 도메인 서비스 자체를 인터페이스로 구현하고 이를 구현한 클래스를 둘 수도 있다.
