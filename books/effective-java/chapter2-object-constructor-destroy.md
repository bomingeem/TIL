2장. 객체 생성과 파괴
===========

### 생성자 대신 정적 팩터리 메서드를 고려하라
클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다.
```
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean:FALSE;
}
```

* 정적 팩토리 메서드가 생성자보다 좋은 장점
1. 이름을 가질 수 있다.
  - 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 주어주자.
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
  - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
  - 서비스 제공자 프레임워크에서의 제공자(provider)는 서비스의 구현체다. 그리고 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.
  - 서비스 제공자 프레임워크는 **3개의 핵심 컴포넌트** 로 이루어진다. 구현체의 동작을 정의하는 서비스 인터페이스(service interface), 제공자가 구현체를 등록 할 때 사용하는 제공자 등록 API(provider registration API), 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API(service access API)


* 단점
1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
  - 다음은 정적 팩터리 메서드에 흔히 사용하는 명명 방식들이다.
    - **from** : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드 ex) Date d = Date.from(instant);
    - **of** : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드 ex) Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
    - **valueOf** : from과 of의 더 자세한 버전 ex) BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
    - **instance** 혹은 **getInstance** : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다. ex) StackWalker luke = StackWalker.getInstance(options);
    - **create** 혹은 **newInstance** : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다. ex) Object newArray = Array.newInstance(classObject, arrayLen);
    - **getType** : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다. ex) FileStore fs = Files.getFileStore(path);
    - **newType** : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다. ex) BufferedReader br = Files.newBufferedReader(path);
    - **type** : getType과 newType의 간결한 버전 ex) List<Complaint> litany = Collections.list(legacyLitany);


* 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.