7장. 오류 처리
===========

오류 처리는 프로그램에 반드시 필요한 요소 중 하나일 뿐이다.  
뭔가 잘못된 가능성은 늘 존재한다. 뭔가 잘못되면 바로 잡을 책임은 바로 우리 프로그래머에게 있다.  


### 오류 코드보다 예외를 사용하라
```
public class DeviceController {
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        //디바이스 상태를 점검한다
        if (handle != DeviceHandle.INVALID) {
            //레코드 필드에 디바이스 상태를 저장한다
            retrieveDeviceRecord(handle);
            //디바이스가 일시정지 상태가 아니라면 종료한다
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                logger.log("Device suspended. Unable to shut down");
            }
        } else {
            logger.log("Invalid handle for : " + DEV1.toString());
        }
    }
}
```
위와 같은 방법을 사용하면 호출자 코드가 복잡해진다. 함수를 호출한 즉시 오류를 확인해야 하기 때문이다.  
그래서 오류가 발생하면 예외를 던지는 편이 낫다. 그러면 호출자 코드가 더 깔끔해진다.  

```
public class DeviceController {
    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }
    
    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);

        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }
    
    private DeviceHandle getHandle(DeviceId id) { 
        ...
        throw new DeviceShutDownError("Invalid handle for : " + id.toString());
    }
}
```


### Try-Catch-Finally 문부터 작성하라
어떤 면에서 try 블록은 트랜잭션과 비슷하다. try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.  
그러므로 예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다. 그러면 try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.  

다음은 파일이 없으면 예외를 던지는지 알아보는 단위 테스트이다.  
```
@Test(expected = StprageException.class)
    public void retrieveSectionShouldThrowOnInvalidFileName() {
        sectionStore.retrieveSection("invalid - file");
    }
```

단위 테스트에 맞춰 다음 코드를 구현했다.
```
public List<RecordedGrip> retrieveSection(String sectionName) {
        return new ArrayList<RecordedGrip>();
    }
```
그런데 코드가 예외를 던지지 않으므로 단위 테스트는 실패한다.  

아래와 같이 코드가 예외를 던지므로 이제 테스트가 성공한다.  
```
public List<RecordedGrip> retrieveSection(String sectionName) {
        try {
            FileInputStream stream = new FileInputStream(sectionName);
            stream.close();
        } catch (FileNotFoundException e) {
            throw new StorageException("retrieval error", e);
        }
        return new ArrayList<RecordedGrip>();
    }
```
먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다.  
그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하게 쉬워진다.  


### 미확인(unchecked) 예외를 사용하라
당시는 확인된 예외를 멋진 아이디어라 생각했다. 실제로도 확인된 예외는 몇가지 장점을 제공한다.  
하지만 지금은 안정적인 소프트웨어를 제작하는 요소로 확인된 예외가 반드시 필요하지는 않다는 사실이 분명해졌다.  
그러므로 우리는 확인된 오류가 치르는 비용에 상응하는 이익을 제공하는지 (철저히) 따져봐야 한다.  


### 예외에 의미를 제공하라
예외를 던질 때는 전후 상황을 충분히 덧붙인다. 그러면 오류가 발생한 원인과 위치를 찾기가 쉬워진다.  
자바는 모든 예외에 호출 스택을 제공한다. 하지만 실패한 코드의 의도를 파악하려면 호출 스택만으로 부족하다.  
오류 메시지에 정보를 담아 예외와 함께 던진다. 실패한 연산 이름과 실패 유형도 언급하고 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다.


### 호출자를 고려해 예외 클래스를 정의하라
애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 **오류를 잡아내는 방법** 이 되어야 한다.  


### 정상 흐름을 정의하라
특수 사례 패턴이란, 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다.  
그러면 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다. 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하므로  


### null을 반환하지 마라
```
public void registerItem(Item item) {
        if (item != null) {
            ItemRegistry registry = persistenStore.getItemRegistry();
            if (registry != null) {
                Item existing = registry.getItem(item.getID());
                if (existing.getBillingPeriod().hasRetailOwner()) {
                    existing.register(item);
                }
            }
        }
    }
```
이런 코드 기반에서 코드를 짜왔다면 나쁘다고 느끼지 않을지도 모른다. 하지만 위 코드는 나쁜 코드다.  
null을 반환하는 코드는 일거리를 늘릴 뿐민 아니라 호출자에게 문제를 떠넘긴다. 누구 하나라도 null 확인을 빼먹는다면 애플리케이션 통제 불능에 빠질지도 모른다.  
persistenStore가 null이라면 실행 시 어떤 일이 벌어질까? NullPointerException이 발생한다.  
위 코드는 null 확인이 **너무 많아** 문제다. 메서드에서 null을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환한다.  

### null을 전달하지 마라
메서드에서 null을 반환하는 방식도 나쁘지만 메서드로 null을 전달하는 방식은 더 나쁘다. 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다.  
대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다. 그렇다면 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.  
즉, 인수로 null이 넘어오면 코드에 문제가 있다는 말이다.  


### 결론
깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. 이 둘은 상충하는 목표가 아니다.  
오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다.  
오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.  

