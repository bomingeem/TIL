3장. 애그리거트
===========

### 애그리거트

- 복잡한 도메인을 이해하고 관리하기 쉬운 단위로 만들려면 상위 수준에서 모델을 조망할 수 있는 방법이 필요한데, 그 방법이 바로 애그리거트이다.
- 애그리거트는 관련된 객체를 하나의 군으로 묶어준다. 수많은 객체를 애그리거트로 묶어서 바라보면 좀 더 상위 수준에서 도메인 모델 간의 관계를 파악할 수 있다.
- 애그리거트는 관련된 모델을 하나로 모은 것이기 때문에 한 애그리거트에 속한 객체는 유사하거나 동일한 라이프사이클을 갖는다.
- 애그리거트는 경계를 갖는다. 한 애그리거트에 속한 객체는 다른 애그리거트에 속하지 않는다. 애그리거트는 독립된 객체 군이며, 각 애그리거트는 자기 자신을 관리할 뿐 다른 애그리거트를 관리하지 않는다.

### 애그리거트 루트

- 애그리거트에 속한 모든 객체가 일관된 상태를 유지하려면 애그리거트 전체를 관리할 주체가 필요한데 이 책임을 지는 것이 바로 애그리거트의 루트 엔티티이다.
- 애그리거트 루트 엔티티는 애그리거트의 대표 엔티티로 애그리거트에 속한 객체는 애그리거트 루트 엔티티에 직접 또는 간접적으로 속한다.

### 도메인 규칙과 일관성

- 애그리거트 루트의 핵심 역할은 애그리거트의 일관성이 깨지지 않도록 하는 것이다. 이를 위해 애그리거트 루트는 애그리거트가 제공해야 할 도메인 기능을 구현한다.
- 예를 들어, 주문 애그리거트는 배송지 변경, 상품 변경과 같은 기능을 제공하는데 애그리거트 루트인 Order가 이 기능을 구현한 메서드를 제공한다.

```java
public class Order {

	// 애그리거트 루트는 도메인 규칙을 구현한 기능을 제공한다.
	public void changeShippingInfo(ShippingInfo newShippingInfo) {
	        verifyNotYetShipped();
	        setShippingInfo(newShippingInfo);
	}

	private void verifyNotYetShipped() {
	        if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
	            throw new IllegalStateException("aleady shipped");
		      }
		  }
	}
```

- 애그리거트 루트가 아닌 다른 객체가 애그리거트에 속한 객체를 직접 변경하면 안된다. 이는 애그리거트 루트가 강제하는 규칙을 적용할 수 없어 모델의 일관성을 깨는 원인이 된다.
- 이 코드는 애그리거트 루트인 Order에서 ShippingInfo를 가져와 직접 정보를 변경하고 있다. 주문 상태와 상관없이 배송지 주소를 변경할 수 있는데, 이는 업무 규칙을 무시하고 DB 테이블에서 직접 데이터를 수정하는 것과 같은 결과를 만든다. 즉, 논리적인 데이터 일관성이 깨지게 되는 것이다.

```java
ShippingInfo si = order.getShippingInfo();

// 주요 도메인 로직이 중복되는 문제
if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
    throw new IllegalStateException("aleady shipped");
}

si.setAddress(newAddress);
```

- 불필요한 중복을 피하고 애그리거트 루트를 통해서만 도메인 로직을 구현하게 만들려면 도메인 모델에 대해 다음의 두 가지를 습관적으로 적용해야 한다.
    - 단순히 필드를 변경하는 set 메서드를 공개(public) 범위로 만들지 않는다.
    - 밸류 타입은 불변으로 구현한다.
- 밸류 객체가 불변이면 밸류 객체의 값을 변경하는 방법은 새로운 밸류 객체를 할당하는 것뿐이다.

```java
public class Order {
	private ShippingInfo shippingInfo;
	public void changeShippingInfo(ShippingInfo newShippingInfo) {
	        verifyNotYetShipped();
	        setShippingInfo(newShippingInfo);
	}

	private void setShippingInfo(ShippingInfo shippingInfo) {
		// 밸류가 불변이면, 새로운 객체를 할당해서 값을 변경해야 한다.
		// 불변이므로 this.shippingInfo.setAddress(newShippingInfo.getAddress())와 같은 코드를 사용할 수 없다.
		this.shippingInfo = newShippingInfo;
	}

	private void verifyNotYetShipped() {
	        if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
	            throw new IllegalStateException("aleady shipped");
		      }
		  }
	}
```

- 밸류 타입의 내부 상태를 변경하려면 애그리거트 루트를 통해서만 가능하다. 그러므로, 애그리거트 루트가 도메인 규칙을 올바르게만 구현하면 애그리거트 전체의 일관성을 올바르게 유지할 수 있다.

### 트랜잭션 범위

- 동일하게 한 트랜잭션에서는 한 개의 애그리거트만 수정해야 한다. 한 트랜잭션에서 두 개 이상의 애그리거트를 수정하면 트랜잭션 충돌이 발생할 가능성이 더 높아지기 때문에 한번에 수정하는 애그리거트 개수가 많아질수록 전체 처리량이 떨어지게 된다.
- 한 트랜잭션에서 한 애그리거트만 수정한다는 것은 애그리거트에서 다른 애그리거트를 변경하지 않는다는 것을 뜻한다.
- 애그리거트는 서로 최대한 독립적이어야 하는데 한 애그리거트가 다른 애그리거트의 기능에 의존하기 시작하면 애그리거트 간 결합도가 높아지게 된다.

### 리포지터리와 애그리거트

- 애그리거트는 개념상 완전한 한 개의 도메인 모델을 표현하므로 객체의 영속성을 처리하는 리포지터리는 애그리거트 단위로 존재한다.
- Order와 OrderLine을 물리적으로 각각 별도의 DB 테이블에 저장한다고 해서 Order와 OrderLine을 위한 리포지터리를 각각 만들지 않는다. Order를 위한 리포지터리만 존재한다.

```java
// 무슨 의미인지 잘 모르겠다..
// 리포지터리에 애그리거트를 저장하면 애그리거트 전체를 영속화해야 한다.
orderRepository.save(order);
```

- 동일하게 애그리거트를 구하는 리포지터리 메서드는 완전한 애그리거트를 제공해야 한다.

```java
// 무슨 의미인지 잘 모르겠다..
// 리포지터리는 완전한 order를 제공해야 한다.
Order order = orderRepository.findById(orderId);

// order가 온전한 애그리거트가 아니면 기능 실행 도중 NullPointerException과 같은 문제가 발생한다.
order.cancel();
```

### ID를 이용한 애그리거트 참조

- ID를 이용한 참조는 DB 테이블에서의 외래키를 사용해서 참조하는 것과 비슷하게 다른 애그리거트를 참조할 때 ID 참조를 사용한다는 점이다. 단, 애그리거트 내의 엔티티를 참조할 때는 객체 레퍼런스로 참조한다.
- ID를 이용한 참조 방식을 사용하면 복잡도를 낮추는 것과 함께 한 애그리거트에서 다른 애그리거트를 수정하는 문제를 원천적으로 방지할 수 있다.
- 애그리거트별로 다른 구현 기술을 사용하는 것도 가능해진다. → **이 부분은 ID를 이용한 애그리거트 참조가 아닐지라도 가능한 것이 아닐까? 무슨 의미인지 잘 모르겠다..**
