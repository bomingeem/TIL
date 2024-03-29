스프링이란?
===========
필수 = 스프링 프레임워크, 스프링 부트  
선택 = 스프링 데이터, 스프링 세션, 스프링 시큐리티, 스프링 Rest Docs, 스프링 배치, 스프링 클라우드..  

### 스프링 프레임워크
* 핵심 기술 : 스프링 DI 컨테이너, AOP, 이벤트 기타  
* 웹 기술 : 스프링 MVC, 스프링 WebFlux  
* 데이터 접근 기술 : 트랜잭션, JDBC, ORM 지원, XML 지원  
* 기술 통합 : 캐시, 이메일, 원격접근, 스케줄링  
* 테스트 : 스프링 기반 테스트 지원  
* 언어 : 코틀린, 그루비  
* 최근에는 스프링 부트를 통해서 스프링 프레임워크의 기술들을 편리하게 사용  

### 스프링 부트
* __스프링을 편리하게 사용할 수 있도록 지원, 최근에는 기본으로 사용__  
* 단독으로 실행할 수 있는 스프링 애플리케이션을 쉽게 생성  
* Tomcat같은 웹 서버를 내장해서 별도의 웹 서버를 설정하지 않아도 됨  
* 손쉬운 빌드 구성을 위한 starter 종속성 제공  
* 스프링과 3rd party(외부) 라이브러리 자동 구성  
* 메트릭, 상태 확인, 외부 구성 같은 프로덕션 준비 기능 제공  
* 관례에 의한 간결한 설정  

### 스프링의 진짜 핵심  
* 스프링은 자바 언어 기반의 프레임워크  
* 자바 언어의 가장 큰 특징 - __객체 지향 언어__  
* 스프링은 객체 지향 언어가 가진 강력한 특징을 살려내는 프레임워크   
* 스프링은 __좋은 객체 지향__ 애플리케이션을 개발할 수 있게 도와주는 프레임워크  

### 객체지향 특징  
* 추상화
* 캡슐화
* 상속
* __다형성__

### 역할과 구현을 분리
#### 자바 언어
* 자바 언어의 다형성을 활용  
  * 역할 = 인터페이스  
  * 구현 - 인터페이스를 구현한 클래스, 구현 객체  
* 객체를 설계할 때 __역할__ 과 __구현__ 을 명확히 분리  
* 객체 설계시 역할(인터페이스)을 먼저 부여하고, 그 역할을 수행하는 구현 객체 만들기 

### 객체의 협력이라는 관계부터 생각
* 혼자 있는 객체는 없다
* 클라이언트: __요청__, 서버: __응답__
* 수 많은 객체 클라이언트와 객체 서버는 서로 협력 관계를 가진다

### 다형성의 본질
* 인터페이스를 구현한 객체 인스턴스를 __실행 시점__ 에 __유연__ 하게 __변경__ 할 수 있다  
* 다형성의 본질을 이해하려면 __협력__ 이라는 객체사이의 관계에서 시작해야함  
* __클라이언트를 변경하지 않고, 서버의 구현 기능을 유연하게 변경할 수 있다__  

### 스프링과 객체 지향
* 다형성이 가장 중요하다  
* 스프링은 다형성을 극대화해서 이용할 수 있게 도와준다  
* 스프링에서 이야기하는 제어의 역전(IoC), 의존관계 주입(DI)은 다형성을 활용해서 역할과 구현을 편리하게 다룰 수 있도록 지원한다  
* 스프링을 사용하면 마치 레고 블럭 조립하듯이, 공연 무대의 배우를 선택하듯이 구현을 편리하게 변경할 수 있다  

좋은 객체 지향 설계의 5가지 원칙(SOLID)
===========
1. SRP: 단일 책임 원칙  
2. OCP: 개방-폐쇄 원칙  
3. LSP: 리스코프 치환 원칙  
4. ISP: 인터페이스 분리 원칙  
5. DIP: 의존관계 역전 원칙  

### SRP (단일 책임 원칙: Single Responsibility Principle)
* 한 클래스는 하나의 책임만 가져야 한다  
* 이는 어떤 변화에 의해 클래스를 변경해야 하는 이유는 오직 하나뿐이어야 함을 의미한다  
* 객체지향에 있어서 책임이란 객체가 **할 수 있는 것** 과 **해야 하는 것** 으로 나뉘어져 있다
* 즉 한마디로 요약하자면 하나의 객체는 자신이 할 수 있는 것과 해야하는 것만 수행할 수 있도록 설계되어야 한다는 법칙이다

* 하나의 책임이라는 것은 모호하다  
  * 클 수 있고 작을 수 있다  
  * 문맥과 상황에 따라 다르다  
* __중요한 기준은 변경__ 이다. 변경이 있을 때 파급 효과가 적으면 단일 책임 원칙을 잘 따른 것
* 예) UI 변경, 객체의 생성과 사용을 분리  

### OCP (개방-폐쇄 원칙: Open Close Principle)
* 소프트웨어의 구성요소(컴포넌트, 클래스, 모듈, 함수)는 __확장에는 열려__ 있으나 __변경에는 닫혀__ 있어야 한다  
* 다형성을 활용해보자  
* 인터페이스를 구현한 새로운 클래스를 하나 만들어서 새로운 기능을 구현  

#### 문제점
* MemberService 클라이언트가 구현 클래스를 직접 선택
```
MemberRepository m = new MemoryMemberRepository(); //기존 코드  
MemberRepository m = new JdbcMemberRepository(); //변경 코드
```
* 구현 객체를 변경하려면 클라이언트 코드를 변경해야 한다  
* 분명 다형성을 사용했지만 OCP 원칙을 지킬 수 없다  
* 이 문제를 어떻게 해결해야 하나?  
* 객체를 생성하고, 연관관계를 맺어주는 별도의 조립, 설정자가 필요하다  

### LSP (리스코프 치환 원칙: The Liskov Substitution Principle)
* 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다  
* 다형성에서 하위 클래스는 인터페이스 규약을 다 지켜야 한다는 것, 다형성을 지원하기 위한 원칙, 인터페이스를 구현한 구현체는 믿고 사용하려면 이 원칙이 필요하다  
* 즉 자식클래스가 부모클래스를 오버라이딩하거나 추가적인 기능을 통해 부모의 상태를 변경시키는 것은 LSP 원칙을 위반하는 것이다  
* 즉 LSP는 서브 클래스가 슈퍼 클래스의 책임을 무시하거나 재정의 하지않고 확장만 수행한다는 것을 의미한다  
* 다시 말해 부모가 수행하고 있는 책임을 그대로 수행하면서 추가적인 필드나 기능을 제공하려난 경우에만 상속을 하는 것이 바람직하며 부모 클래스의 책임을 변화시키는 기능은 LSP에 위배 된다고 볼 수 있다


### ISP (인터페이스 분리 원칙: Interface Segregation Principle)
* 특정 클라이언트를 위한 인터페이스 여러개가 범용 인터페이스 하나보다 낫다  
* 자동차 인터페이스 → 운전 인터페이스, 정비 인터페이스로 분리  
* 사용자 클라이언트 → 운전자 클라이언트, 정비사 클라이언트로 분리  
* 분리하면 정비 인터페이스 자체가 변해도 운전자 클라이언트에 영향을 주지 않음  
* 인터페이스가 명확해지고 대체 가능성이 높아진다  

### DIP (의존관계 역전 원칙: Dependency Inversion Principle)
* 프로그래머는 "추상화에 의존해야지, 구체화에 의존하면 안된다" 의존성 주입은 이 원칙을 따르는 방법 중 하나다  
* 쉽게 이야기해서 구현 클래스에 의존하지 말고 인터페이스에 의존하라는 뜻  
* 앞에서 이야기한 __역할(Role)에 의존하게 해야 한다는 것과 같다__ 객체 세상도 클라이언트가 인터페이스에 의존해야 유역하게 구현체를 변경할 수 있다! 구현체에 의존하게 되면 변경이 아주 어려워진다  

* 그런데 OCP에서 설명한 MemberService는 인터페이스에 의존하지만 구현 클래스도 동시에 의존한다
* MemberService 클라이언트가 구현 클래스를 직접 선택
```
MemberRepository m = new MemoryMemberRepository();
```
* __DIP 위반__

## 정리
* 객체 지향의 핵심은 다형성  
* 다형성 만으로는 쉽게 부품을 갈아 끼우듯이 개발할 수 없다  
* 다형성 만으로는 구현 객체를 변경할 때 클라이언트 코드도 함께 변경된다  
* __다형성 만으로는 OCP, DIP를 지킬 수 없다__  
* 무언가 더 필요하다

객체 지향 설계와 스프링
===========
* __스프링은 다음 기술로 다형성 + OCP, DIP를 가능하게 지원__
  * DI(Dependency Injection): 의존관계, 의존성 주입  
  * DI 컨테이너 제공
* __클라이언트 코드의 변경 없이 기능 확장__

## 정리
* 모든 설계에 __역할__ 과 __구현__ 을 분리하자  
* 자동차, 공연의 예를 떠올려보자  
* 애플리케이션 설계도 공연을 설계 하듯이 배역만 만들어두고, 배우는 언제든지 __변경__ 할 수 있도록 만드는 것이 좋은 객체 지향 설계다  
* 이상적으로는 모든 설계에 인터페이스를 부여하자  

#### 실무 고민
* 하지만 인터페이스를 도입하면 추상화라는 비용이 발생  
* 기능을 확장할 가능성이 없다면 구현 클래스를 직접 사용하고, 향후 꼭 필요할 때 리팩토링해서 인터페이스를 도입하는 것도 방법이다  






