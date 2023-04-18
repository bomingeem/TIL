10장. 이벤트
===========

### 시스템 간 강결합의 문제

- 보통 결제 시스템은 외부에 존재하므로 RefundService는 외부의 환불 시스템 서비스를 호출하는데, 이때 두 가지 문제가 발생한다
1. 외부 서비스가 정상이 아닐 경우 트랜잭션 처리를 어떻게 해야 할지 애매함
    a. 익셉션이 발생할 경우 트랜잭션을 롤백? 커밋?
2. 성능
    a. 응답 시간이 길어지면 그만큼 대기 시간 발생
- 두 가지 문제 외에도 도메인 객체에 서비스를 전달하면 추가로 설계상 문제가 나타날 수 있다
    1. 주문 로직과 결제 로직이 섞이는 문제 발생

    ```java
    public void cancel(RefundService refundService) {
            // 주문로직
            verifyNotYetShipped();
            this.state = OrderState.CANCELED;
            
            // 결제로직
            this.refundStatus = Status.REFUND_STARTED;
            try {
                refundService.refund(getPaymentId());
                this.refundStatus = Status.REFUND_COMPLETED;
            } catch (Exception ex) { }
        }
    ```

    2. 기능을 추가할 때 발생
        1. 기능을 추가할 때마다 파라미터가 함께 추가되면 다른 로직이 더 많이 섞이고 트랜잭션 처리가 더 복잡해짐
        2. refundService는 성공하고 noticeService는 실패한다면?
        3. refundService와 noticeService 중 무엇을 먼저 처리하나?
- 지금까지 언급한 문제가 발생하는 이유: 주문 BOUNDED CONTEXT와 결제 BOUNDED CONTEXT 간 **강결합(high coupling)** 때문
    - 이런 강한 결합을 없앨 수 있는 방법은 바로 ‘**이벤트**를 사용하는 것’


### 이벤트 개요

- 이벤트라는 용어는 ‘과거에 벌어진 어떤 것’
- 이벤트는 발생하는 것에서 끝나지 않으며 그 이벤트에 반응하여 원하는 동작을 수행하는 기능을 구현한다

### 이벤트 관련 구성요소

- 도메인 모델에 이벤트를 도입하려면 네 개의 구성요소를 구현해야 함

![이벤트관련구성요소](https://user-images.githubusercontent.com/47099798/232768950-2f6d9608-fbd0-4b60-854b-fb0168266c50.png)

- 도메인 모델에서 이벤트 주체는 엔티티, 밸류, 도메인 서비스와 같은 도메인 객체
- 도메인 로직을 실행해서 ‘상태가 바뀌면 관련 이벤트 발생’
- ‘이벤트 헨들러’는 이벤트 생성 주체ㅔ가 발생한 이벤트에 반응
- 이벤트 생성 주체와 이벤트 핸들러를 연결해 주는 것이 ‘이벤트 디스패처’
    - 이벤트 디스패처의 구현 방식에 따라 생성과 처리를 동기나 비동기로 실행

### 이벤트의 구성

- 이벤트 종류: 클래스 이름으로 이벤트 종류를 표현
- 이벤트 발생 시간
- 추가 데이터: 주문번호, 신규 배송지 정보 등 이벤트와 관련된 정보

### 이벤트 용도

1. 트리거
    1. 도메인의 상태가 바뀔 때 다른 후처리를 실행하기 위한 트리거로 사용
2. 서로 다른 시스템 간의 데이터 동기화

### 이벤트 장점

1. 서로 다른 도메인 로직이 섞이는 것을 방지

```java
public void cancel() {
        verifyNotYetShipped();
        this.state = OrderState.CANCELED;

				this.refundStatus = State.REFUND_STARTED;
        Events.raise(new OrderCanceledEvent(number.getNumber()));
    }
```

1. 기능 확장 용이

### 이벤트, 핸들러, 디스패처 구현

- 이벤트 클래스
- EventHandler: 이벤트 핸들러를 위한 상위 타입, 모든 핸들러는 이 인터페이스를 구현
- Events: 이벤트 디스패처, 이벤트 발행, 이벤트 핸들러 등록, 이벤트를 핸들러에 등록하는 등 기능 제공

### 이벤트 클래스

- 이벤트는 과거에 벌어진 상태 변화나 사건을 의미하므로 클래스의 이름을 결정할 때에는 ‘과거 시제 사용’
- 이벤트를 처리하는 데 필요한 최소한의 데이터 포함
- 공통으로 갖는 프로퍼티가 존재할 경우 관련 상위 클래스를 만들어 상속받아 사용

```java
public class OrderCanceledEvent extends Event {
    // 이벤트는 핸들러에서 이벤트를 처리하는 데 필요한 데이터를 포함
    // 발생 시간이 필요한 각 이벤트 클래스는 Event를 상속받아 구현
    private String orderNumber;

    public OrderCanceledEvent(String number) {
			  super();
        this.orderNumber = number;
    }

    public String getOrderNumber() {
        return orderNumber;
    }
}
```

### EventHandler 인터페이스

- EventHandler 인터페이스는 이벤트 핸들러를 위한 상위 인터페이스
- EventHandler 인터페이스를 상속받는 클래스는 handle() 메서드를 이용해서 필요한 기능 구현

```java
public interface EventHandler<T> {
    void handle(T event);

    default boolean canHandle(Object event) {
        Class<?>[] typeArgs = TypeResolver.resolveRawArguments(
                EventHandler.class, this.getClass());
        return typeArgs[0].isAssignableFrom(event.getClass());
    }
}
```

### 이벤트 디스패처인 Events 구현

- 도메인을 사용하는 응용 서비스는 이벤트를 받아 처리할 핸들러를 Events.handle()로 등록하고, 도메인 기능을 실행
- Events는 핸들러 목록을 유지하기 위해 ThreadLocal 변수를 사용
- 사용자의 요청을 처리한 뒤 Events.reset()을 실행하지 않으면 스레드 handlers가 담고 있는 list에 계속 핸들러 객체가 쌓이게 되어 결국 메모리 부족 에러가 발생

```java
@Transactional
    public void cancel(OrderNo orderNo, Canceller canceller) {
        **Events.handle((OrderCanceledEvent evt) -> refundService.refund(evt.getOrderNumber()));**

        Order order = findOrder(orderNo);
        if (!cancelPolicy.hasCancellationPermission(order, canceller)) {
            throw new NoCancellablePermission();
        }
        order.cancel();

        **Events.reset();**
    }
```

### 흐름 정리

![이벤트처리흐름](https://user-images.githubusercontent.com/47099798/232769026-645d357e-cef5-4cd1-a613-af350473d3c9.png)

1. 이벤트 처리에 필요한 이벤트 핸들러 생성
2. 이벤트 발생 전 이벤트 핸들러를 Events.handle() 메서드를 이용해서 등록
3. 이벤트를 발생하는 도메인 기능 실행
4. 도메인은 Events.raise()를 이용해서 이벤트 발생
5. Events.raise()는 등록된 핸들러의 canHandle()을 이용해서 이벤트를 처리할 수 있는지 확인
6. 핸들러가 이벤트를 처리할 수 있다면 handle() 메서드를 이용해서 이벤트 처리
7. Events.raise() 실행을 끝나고 리턴
8. 도메인 기능 실행을 끝내고 리턴
9. Events.raise()을 이용해서 ThreadLocal을 초기화
- 코드 흐름을 보면 응용 서비스와 동일한 트랜잭션 범위에서 핸들러의 handle()이 실행된다
- 즉, 도메인의 상태 변경과 이벤트 핸들러는 ‘같은 트랜잭션 범위에서 실행’

### AOP를 이용한 Events.reset() 실행

- 모든 응용 서비스마다 Events.reset()을 실행하는 코드는 중복에 해당하므로 AOP를 이용한다

```java
@Aspect
@Order(0)
@Component
public class EventsResetProcessor {
    private ThreadLocal<Integer> nestedCount = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return new Integer(0);
        }
    };

    @Around("execution(public * com.myshop..*Service.*(..))")
    public Object doReset(ProceedingJoinPoint joinPoint) throws Throwable {
        nestedCount.set(nestedCount.get() + 1);
        try {
            return joinPoint.proceed();
        } finally {
            nestedCount.set(nestedCount.get() - 1);
            if (nestedCount.get() == 0) {
                Events.reset();
            }
        }
    }
}
```

### 동기 이벤트 처리 문제

- 이벤트를 사용해 강결합 문제는 해소했지만 외부 서비스에 영향을 받는 문제가 있다.
- 외부 시스템과의 연동을 동기로 처리할 때 발생하는 성능과 트랜잭션 범위 문제를 해소하는 방법 중 하나가 ‘이벤트를 비동기로 처리’하는 것

```java
@Transactional **// 외부 연동 과정에서 익셉션이 발생하면 트랜잭션 처리는?**
    public void cancel(OrderNo orderNo) {
        **// refundService.refund()가 오래 걸리면?**
        Events.handle((OrderCanceledEvent evt) -> refundService.refund(evt.getOrderNumber()));

        Order order = findOrder(orderNo);
        order.cancel();
    }
```

### 비동기 이벤트 처리

- 우리가 구현해야 할 것 중에서 후속 조치를 바로 할 필요 없이 일정 시간 안에만 처리하는 되는 경우가 적지 않다. ex) 이메일 인증, 주문 취소 후 바로 결제 취소 하지 않는 경우
- A 이벤트가 발생하면 별도 스레드로 B를 수행하는 핸들러를 실행하는 방식으로 요구사항 구현
- 네 가지 방식으로 비동기 이벤트 처리 구현
    1. 로컬 핸들러를 비동기로 실행
    2. 메시지 큐 사용
    3. 이벤트 저장소와 이벤트 포워더 사용
    4. 이벤트 저장소와 이벤트 제공 API 사용

### 로컬 핸들러의 비동기 실행

- 이벤트 핸들러를 별도 스레드로 실행
- 비동기로 실행할 이벤트 핸들러는 `executor.submit()` 을 이용해 스레드 풀에 핸들러 실행 작업을 등록하는 반면 동기로 실행할 이벤트 핸들러는 바로 실행 `handler.handle(event)`

```java
@Transactional
    public void cancel(OrderNo orderNo) {
        Events.handleAsync((OrderCanceledEvent evt) -> refundService.refund(evt.getOrderNumber()));

        Order order = findOrder(orderNo);
        order.cancel(); // Events.raise(new OrderCanceledEvent()) 실행
    }
```

- Events.handleAsync()로 등록한 이벤트 핸들러는 별도 스레드로 실행되므로 refund() 코드와 cancel() 메서드와 관련된 트랜잭션 범위에 묶이지 않는다.
- 별도 스레드를 이용해서 이벤트 핸들러를 실행하면 이벤트 발생 코드와 같은 트랜잭션 범위에 묶을 수 없기 때문에 한 트랜잭션으로 실행해야 하는 이벤트 핸들러는 비동기로 처리해서는 안된다.

### 메시징 시스템을 이용한 비동기 구현

- 이벤트가 발생하면 이벤트 디스패처는 이벤트를 메시지 큐에 보낸다. 메시지 큐는 이벤트를 메시지 리스너에 전달하고, 메시지 리스너는 알맞은 이벤트 핸들러를 이용해서 이벤트를 처리
- 이때 이벤트를 메시지 큐에 저장하는 과정과 메시지 큐에서 이벤트를 읽어와 처리하는 과정은 별도 스레드나 프로세스로 처리

![메시징큐](https://user-images.githubusercontent.com/47099798/232769087-6d5cb02e-e43f-4c76-a913-e73571cbc1b8.png)

- RabbitMQ처럼 많이 사용되는 메시징 시스템은 글로벌 트랜잭션 지원과 함께 클러스터와 고가용성을 지원하기 때문에 안정적으로 메시지를 전달할 수 있다는 장점이 있다. ex) Kafka
