1장. 도메인 모델 시작
===========

### 도메인

- 도메인은 여러 하위 도메인으로 구성된다.
    - ex: 온라인 서점 - 주문, 정산, 회원, 혜택, 결제, 배송, 카탈로그, 리뷰..
- 소프트웨어가 도메인의 모든 기능을 제공하지는 않는다.

### 도메인 모델

- 도메인 모델을 사용하면 여러 관계자들이 동일한 모습으로 도메인을 이해하고 도메인 지식을 공유하는 데 도움이 된다.
- 기본적으로 도메인 자체를 이해하기 위한 개념 모델이다.

### 도메인 모델 패턴

- 일반적인 애플리케이션의 아키텍처는 네 개의 계층으로 구성된다.
- **표현 - 응용 - 도메인 - 인프라스트럭처**

| 사용자 인터페이스(UI) 또는 표현(Presentation) | 사용자의 요청을 처리하고 사용자에게 정보를 보여준다. 여기서 사용자는 소프트웨어를 사용하는 사람뿐만 아니라 외부 시스템도 사용자가 될 수 있다. |
| --- | --- |
| 응용(Application) | 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다. |
| 도메인 | 시스템이 제공할 도메인의 규칙을 구현한다. |
| 인프라스트럭처(Infrastructure) | 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다. |
- 도메인 계층은 도메인의 핵심 규칙을 구현한다. 주문 도메인의 경우 **‘출고 전에 배송지를 변경할 수 있다’**는 규칙과 **‘주문 취소는 배송 전에만 할 수 있다’**는 규칙을 구현한 코드가 도메인 계층에 위치하게 된다.

```java
public class Order {
	private OrderState state;
	private ShippingInfo shippingInfo;

	public void changeShippingInfo(ShippingInfo newShippingInfo) {
		if (!state.isShippingChangeable()) {
			throw new IllegalStateException("can't change shipping in" + state);
		}
		this.shippingInfo = newShippingInfo; 
	}

	public enum OrderState {
		PAYMENT_WAITING {
			public boolean isShippingChangeable() {
				return true;
			}
		},
		PREPARING {
			public boolean isShippingChangeable() {
				return true;
			}
		},
		SHIPPED,
		DELIVERING,
		DELIVERY_COMPLETED;

		public boolean isShippingChangeable() {
			return false;
		}
	}
}
```

- 즉, OrderState는 주문 대기 중이거나 상품 준비 중에는 배송지를 변경할 수 있다는 도메인 규칙을 구현하고 있다.

```java
public class Order {
	private OrderState state;
	private ShippingInfo shippingInfo;

	public void changeShippingInfo(ShippingInfo newShippingInfo) {
		if (isShippingChangeable()) {
			throw new IllegalStateException("can't change shipping in" + state);
		}
		this.shippingInfo = newShippingInfo; 
	}

	private boolean isShippingChangeable() {
		return state == OrderState.PAYMENT_WAITING || state == OrderState.PREPARING;
	}
}

public enum OrderState {
	PAYMENT_WAITING,
	PREPARING,
	SHIPPED,
	DELIVERING,
	DELIVERY_COMPLETED;
}
```

- 배송지 변경 가능 여부를 판단하는 기능이 Order에 있든, OrderState에 있든 중요한 점은 주**문과 관련된 중요 업무 규칙을 주문 도메인 모델인 Order나 OrderState에서 구현한다는 점**이다. 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다.
- 처음부터 완벽한 개념 모델을 만들기보다는 전반적인 개요를 알 수 있는 수준으로 개념 모델을 작성해야 한다. 프로젝트 초기에는 개요 수준의 개념 모델로 도메인에 대한 전체 윤곽을 이해하는 데 집중하고, 구현하는 과정에서 개념 모델을 구현 모델로 점진적으로 발전시켜 나가야 한다.

### 도메인 모델 도출

- 기획서, 유스케이스, 사용자 스토리와 같은 요구사항과 관련자와의 대화를 통해 도메인을 이해하고 이를 바탕으로 도메인 모델 초안을 만들어야 비로소 코드를 작성할 수 있다.
- OrderLine은 한 상품(product 필드)을 얼마에(price 필드), 몇 개 살지(count 필드)를 필드에 담고 있고 calculateAmounts() 메서드로 구매 가격을 구하는 로직을 구현하고 있다.

```java
public class OrderLine {
    private Product product;
    private int price;
    private int quantity;
    private int amounts;

    public OrderLine(Product product, int price, int quantity, int amounts) {
        this.product = product;
        this.price = price;
        this.quantity = quantity;
        this.amounts = amounts;
    }
    
    private int calculateAmounts() {
        return price * quantity;
    }
    
    public int getAmounts() { ... }
}
```

- Order와 OrderLine과의 관계
    - 최소 한 종류 이상의 상품을 주문해야 한다.
    - 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.

```java
public class Order {
    private List<OrderLine> orderLines;
    private int totalAmounts;

    public Order(List<OrderLine> orderLines) {
        setOrderLines(orderLines);
    }

    public void setOrderLines(List<OrderLine> orderLines) {
        verifyAtLeastOneOrMoreOrderLines(orderLines);
        this.orderLines = orderLines;
        calculateTotalAmounts();
    }

    private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
        if (orderLines == null || orderLines.isEmpty()) {
            throw new IllegalArgumentException("no OrderLine");
        }
    }

    private void calculateTotalAmounts() {
        this.totalAmounts = new Money(orderLines.stream()
                .mapToInt(x -> x.getAmounts().getValue()).sum());
    }
}
```

- 앞서 요구사항 중에 ‘주문할 때 배송지 정보를 반드시 지정해야 한다’는 내용이 있다. 이는 Order를 생성할 때 OrderLine의 목록뿐만 아니라 ShippingInfo도 함께 전달해야 함을 의미한다.

```java
public class Order {
    private List<OrderLine> orderLines;
    private int totalAmounts;
    private ShippingInfo shippingInfo;

    public Order(List<OrderLine> orderLines, ShippingInfo shippingInfo) {
        setOrderLines(orderLines);
        setShippingInfo(shippingInfo);
    }

    public void setOrderLines(List<OrderLine> orderLines) {
        verifyAtLeastOneOrMoreOrderLines(orderLines);
        this.orderLines = orderLines;
        calculateTotalAmounts();
    }
    
    public void setShippingInfo(ShippingInfo shippingInfo) {
        if (shippingInfo == null) {
            throw new IllegalArgumentException("no ShippingInfo");
        }
        this.shippingInfo = shippingInfo;
    }

    private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
        if (orderLines == null || orderLines.isEmpty()) {
            throw new IllegalArgumentException("no OrderLine");
        }
    }

    private void calculateTotalAmounts() {
        this.totalAmounts = new Money(orderLines.stream()
                .mapToInt(x -> x.getAmounts().getValue()).sum());
    }
}
```

- 배송지 변경이나 주문 취소 기능은 출고 전에만 가능하다는 제약 규칙이 있으므로 이 규칙을 적용

```java
public class Order {
    private OrderState state;
    
    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        verifyNotYetShipped();
        setShippingInfo(newShippingInfo);
    }
    
    public void cancel() {
        this.state = OrderState.CANCELED;
    }
    
    private void verifyNotYetShipped() {
        if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
            throw new IllegalStateException("aleady shipped");
        }
    }
}
```

### 엔티티와 밸류

- 도출한 모델은 크게 **엔티티(Entity)**와 **밸류(Value)**로 구분할 수 있다.

### 엔티티

- 엔티티의 가장 큰 특징은 식별자를 갖는다는 것이다.
    - ex: Order는 엔티티로서 orderNumber를 식별자로 갖는다.
    - 주문에서 배송지 주소가 바뀌거나 상태가 바뀌더라도 주문번호가 바뀌지 않는 것처럼 엔티티의 식별자는 바뀌지 않는다.
- 엔티티의 식별자는 바뀌지 않고 고유하기 때문에 두 엔티티 객체의 식별자가 같으면 두 엔티티는 같다고 판단할 수 있다.

### 엔티티의 식별자 생성

- 엔티티의 식별자를 생성하는 시점은 도메인의 특징과 사용하는 기술에 따라 달라진다. 흔히 식별자는 다음 중 한 가지 방식으로 생성한다.
    - 특정 규칙에 따라 생성
    - UUID 사용
    - 값을 직접 입력
    - 일련번호 사용(시퀀스나 DB의 자동 증가 컬럼 사용)

### 밸류 타입

- 밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다. 예를 들어, 받는 사람을 위한 밸류 타입인 Receiver를 다음과 같이 작성할 수 있다.

```java
public class Receiver {
    private String name;
    private String phoneNumber;

    public Receiver(String name, String phoneNumber) {
        this.name = name;
        this.phoneNumber = phoneNumber;
    }

    public String getName() {
        return name;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }
}
```

```java
public class Address {
    private String address1;
    private String address2;
    private String zipcode;

    public Address(String address1, String address2, String zipcode) {
        this.address1 = address1;
        this.address2 = address2;
        this.zipcode = zipcode;
    }
}
```

- 밸류 타입을 이용해서 ShippingInfo 클래스를 다시 구현해보자

```java
public class ShippingInfo {
    private Receiver receiver;
    private Address address;
}
```

- 밸류 타입이 꼭 두 개 이상의 데이터를 가져야 하는 것은 아니다. 의미를 명확하게 표현하기 위해 밸류 타입을 사용하는 경우도 있다.

```java
public class OrderLine {
    private Product product;
    private Money price;
    private int quantity;
    private Money amounts;
}
```

- 밸류 타입을 사용할 때의 또 다른 장점은 밸류 타입을 위한 기능을 추가할 수 있다는 것이다. 예를 들어, Money 타입은 다음과 같이 돈 계산을 위한 기능을 추가할 수 있다.

```java
public class Money {
    private int value;

    public Money(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
    
    public Money add(Money money) {
        return new Money(this.value + money.value);
    }
    
    public Money multiply(int multiplier) {
        return new Money(value * multiplier);
    }
}
```

- Money를 사용하는 코드는 이제 ‘정수 타입 연산’이 아니라 ‘돈 계산’이라는 의미로 코드를 작성 할 수 있게 된다.
- 밸류 객체의 데이터를 변경할 때는 기존 데이터를 변경하기보다는 변경한 데이터를 갖는 새로운 밸류 객체를 생성하는 방식은 선호한다.
