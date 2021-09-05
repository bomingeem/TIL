3장. 함수
===========

어떤 프로그램이든 가장 기본적인 단위가 함수다. 이 장은 함수를 잘 만드는 법을 소개한다.  

### 작게 만들어라!
함수를 만드는 첫째 규칙은 '작게!'다. 함수를 만드는 두번째 규칙은 '더 작게!'다.  

#### 블록과 들여쓰기
if문/else문/while문 등에 들어가는 블록은 한 줄이어야 한다는 의미다.  
대게 거기서 함수를 호출하는데 그러면 바깥을 감싸는 함수가 작아질 뿐만 아니라, 블록 안에서 호출하는 함수 이름을 적절히 짓는다면 코드를 이해하기도 쉬워진다.  
함수에서 들여쓰기 수준은 1단이나 2단을 넘어서면 안된다. 그래야 읽고 이해하기 쉬워진다.  


### 한 가지만 해라!
함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.  
함수가 '한 가지'만 하는지 판단하는 방법이 하나 더 있다. 단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.  


### 함수 당 추상화 수준은 하나로!  
함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.  
한 함수 내에 추상화 수준을 섞으면 코드를 읽는 사람이 헷갈린다. 특정 표현이 근본 개념인기 아니면 세부사항인지 구분하기 어려운 탓이다.  
근본 개념과 세부사항을 뒤섞기 시작하면, 깨어진 창문처럼 사람들이 함수에 세부사항을 점점 더 추가한다.  

#### 위에서 아래로 코드 읽기: **내려가기** 규칙
코드는 위에서 아래로 이야기처럼 읽혀야 좋다. 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한번에 한 단계씩 낮아진다. 이것을 **내려가기** 규칙이라 부른다.  
핵심은 짧으면서도 '한 가지'만 하는 함수다. 위에서 아래로 TO 문단을 읽어내려 가듯이 코드를 구현하면 추상화 수준을 일관되게 유지하기가 쉬워진다.  


### Switch문
switch문은 작게 만들기 어렵다. 본질적으로 switch문은 N가지를 처리한다.  
하지만 각 switch문을 저차원 클래스에 숨기고 절대로 반복하지 않는 방법은 있다. 물론 다형성을 이용한다.  


### 서술적인 이름을 사용하라!  
좋은 이름이 주는 가치는 아무리 강조해도 지나치지 않는다. 코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드로 불러도 되겠다.  
함수가 작고 단순할수록 서술적인 이름을 고르기도 쉬워진다. 이름이 길어도 괜찮다. 길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 길고 서술적인 이름이 길고 서술적인 주석보다 좋다.  
함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다. 그런 다음, 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다.  
서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다.  
이름을 붙일 때는 일관성이 있어야 한다. 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다.  
includeSetupAndTeardownPages, includeSetupPages, includeSuiteSetupPage, includeSetupPage 등이 좋은 예다.  


### 함수 인수 
함수에서 이상적인 인수 개수는 0개(무항)다. 다음은 1개(단항)고, 다음은 2개(이항)다. 3개(삼항)는 가능한 피하는 편이 좋다. 4개 이상(다항)은 특별한 이유가 필요하다.  
인수는 개념을 이해하기 어렵게 만든다. 최선은 입력 인수가 없는 경우이며, 차선은 입력 인수가 1개뿐인 경우다.  

#### 많이 쓰는 단항 형식
함수에 인수 1개를 넘기는 이유로 가장 흔한 경우는 두 가지다. 하나는 인수에 질문을 던지는 경우다. 다른 하나는 인수를 뭔가로 변환해 결과를 반환하는 경우다.  
다소 드물게 사용하지만 그래도 아주 유용한 단항 함수 형식이 이벤트다. 이벤트 함수는 입력 인수만 있다. 출력 인수는 없다.  
이벤트 함수는 조심해서 사용한다. 이벤트라는 사실이 코드에 명확히 드러나야 한다. 그러므로 이름과 문맥을 주의해서 선택한다.  
지금까지 설명한 경우가 아니라면 단항 함수는 가급적 피한다.  
예를 들어 void includeSetupPageInto(StringBuffer pageText)는 피한다. 변환 함수에서 출력 인수를 사용하면 혼란을 일으킨다.  
입력 인수를 변환하는 함수라면 변환결과는 반환값으로 돌려준다. StringBuffer transform(StringBuffer in)이 void transform(StringBuffer out)보다 좋다.  

#### 플래그 인수
플래스 인수는 추하다. 함수로 bool값을 넘기는 관례는 정말로 끔찍하다. 왜냐하면 함수가 한꺼번에 여러 가지를 처리한다고 대놓고 공표를 하는 셈이니까!  

#### 이항 함수
인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다. 예를 들어 writeField(name)는 writeField(outputStream, name)보다 이해하기 쉽다.  
물론 이항 함수가 적절한 경우도 있다. Point p = new Point(0, 0)가 좋은 예이다. 직교 좌표계 점은 일반적으로 인수 2개를 취한다.  
이항 함수가 무조건 나쁘다는 소리는 아니다. 프로그램을 짜다보면 불가피한 경우도 생긴다. 하지만 그만큼 위험이 따른다는 사실을 이해하고 가능하면 단항함수로 바꾸도록 애써야 한다.  

#### 삼항 함수
인수가 3개인 함수는 인수가 2개인 함수보다 훨씬 더 이해하기 어렵다. 순서, 주춤, 무시로 야기되는 문제가 두 배 이상 늘어난다.  
예를 들어, assertEquals(message, expected, actual)이라는 함수를 살펴보면 첫 인수가 expected라고 예상하지 않았는가?  
사실은 **매번 함수를 볼 때마다** 주춤했다가 message를 무시해야 한다는 사실을 상기했다.  

#### 인수 객체
인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어본다.  
```
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point cneter, double radius);
```
객체를 생성해 인수를 줄이는 방법이 눈속임이라 여겨질지 모르지만 그렇지 않다.  
위 예제에서 x와 y를 묶었듯이 변수를 묶어 넘기려면 이름을 붙여야 하므로 결국은 개념을 표현하게 된다.  

#### 인수 목록
때로는 인수 개수가 가변적인 함수도 필요하다. String.format 메서드가 좋은 예다. 
```
String.format("%s worked %.2f" hours.", name, hours);
```
위 예제처럼 가변 인수 전부를 동등하게 취급하면 List 형 인수 하나로 취급할 수 있다.  
가변 인수를 취하는 모든 함수에 같은 원리가 적용된다. 가변 인수를 취하는 함수는 단항, 이항, 삼항 함수로 취급할 수 있다.  

#### 동사와 키워드
함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다.  
단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다. 예를 들면 write(name)보다 좀 더 나은 이름은 writeField(name)이다.  
마지막 예제는 함수 이름에 **키워드** 를 추가하는 형식이다. 즉 함수 이름에 인수 이름을 넣는다.  
예를 들어, assertEquals보다 assertExpectedEqualsActual(expected, actual)이 더 좋다.  


### 부수 효과를 일으키지 마라!
부수 효과는 거짓말이다. 함수에서 한 가지를 하겠다고 약속하고선 **남몰래** 다른 짓도 하니까.  
때로는 예상치 못하게 클래스 변수를 수정하거나 함수로 넘어온 인수나 시스템 전역 변수를 수정한다. 많은 경우 시간적인 결함이나 순서 종속성을 초래한다.  
```
public class UserValidator {
	private Cryptographer cryptographer;

	public boolean checkPassword(String userName, String password) {
	  User user = UserGateway.findByName(userName);
	  if (user != User.NULL) {
	    String codePhrase = user.getPhraseEncodedByPassword();
	    String phrase = cryptographer.decrypt(codedPhrase, password);
	    if ("Valid Password".equals(phrase)) {
	      Session.initialize();
	      return true;
	    }
	  }
	  return false;
	}
}
```
여기서, 함수가 일으키는 부수 효과는 Session.initialize() 호출이다. checkPassword 함수는 이름 그대로 암호를 확인하는데 이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다.  
그래서 함수 이름만 보고 함수를 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처한다. 이런 부수 효과가 시간적인 결합을 초래한다.  

#### 출력 인수
일반적으로 출력 인수는 피해야 한다. 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다.  


### 명령과 조회를 분리하라!
함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야한다. 둘 다 하면 안된다. 객체 상태를 변경하거나 객체 정보를 반환하거나 둘 중 하나다.  
```
public boolean set(String attribute, String value);
if (set("username", "unclebob"))..
```
함수를 구현한 개발자는 "set"을 동사로 의도했다. 하지만 if문에 넣고 보면 형용사로 느껴진다. 진짜 해결책은 명령과 조회를 분리해 혼란을 뿌리뽑는 방법이다.  
```
if (attributeExists("username")) {
  setAttribute("username", "unclebob");
  ...
}
```


### 오류 코드보다 예외를 사용하라!
명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다.
```
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
	  if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
	    logger.log("page deleted");
	  } else {
	    logger.log("configKey not deleted");
	  }
	} else {
	  logger.log("deleteReference from registry failed");
	}
} else {
	logger.log("delete failed");
	return E_ERROR;
}
```
반면 오류 코드 대신 예외를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해진다.  
```
try {
	deletePage(page);
	registry.deleteReference(page.name);
	configKeys.deleteKey(page.name.makeKey());
} catch (Exception e) {
	logger.log(e.getMessage());
}
```

#### Try/Catch 블록 뽑아내기
try/catch 블록은 원래 추하다. 코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 뒤섞는다. 그러므로 별도 함수로 뽑아내는 편이 좋다.
```
public void delete(Page page) {
	try {
	  deletePageAndAllReferences(page);
	} catch (Exception e) {
	  logError(e);
	}
}

private void deletePageAndAllReferences(Page page) throws Exception {
	deletePage(page);
	registry.deleteReference(page.name);
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
	logger.log(e.getMessage());
}
```
위에서 delete 함수는 모든 오류를 처리한다. 그래서 코드를 이해하기 쉽다.  
이렇게 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.  

#### 오류 처리도 한 가지 작업이다.
함수는 '한 가지' 작업만 해야 한다. 오류 처리도 '한 가지' 작업에 속한다. 그러므로 오류를 처리하는 함수는 오류만 처리해야 마땅하다.


### 반복하지 마라!
중복은 문제다. 코드 길이가 늘어날 뿐 아니라 알고리즘이 변하면 손봐야 하니까. 게다가 어느 한곳이라도 빠뜨리는 바람에 오류가 발생활 확률도 높다.  


### 구조적 프로그래밍
데이크스트라는 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나만 존재해야 한다고 말했다. 즉, 함수는 return문이 하나여야 한다는 말이다.  
루프 안에서 break나 continue를 사용해선 안 되며 goto는 **절대로** 안된다.  
함수가 작다면 위 규칙은 별 이익을 제공하지 못한다. 함수가 아주 클 때만 상당한 이익을 제공한다.  
그로므로 함수를 작게 만든다면 간혹 return, break, continue를 여러 차례 사용해도 괜찮다.


### 함수를 어떻게 짜죠?
소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다. 함수를 짤 때도 마찬가지다.  
처음에는 길고 복잡하다. 들여쓰기 단계도 많고 중복된 루프도 많다. 인수 목록도 아주 길다. 이름은 즉흥적이고 코드는 중복된다.  
하지만 나는 그 서투른 코드를 빠짐없이 테스트하는 단위 테스트 케이스도 만든다.  
그런 다음 코드를 다듬고 함수를 만들고 이름을 바꾸고 중복을 제거하고 메서드를 줄이고 순서를 바꾼다. 때로는 전체 클래스를 쪼개기도 하고 이 와중에도 코드는 항상 단위 테스트를 통과한다.  


### 결론
모든 시스템은 프로그래머가 설계한 도메인 특화 언어로 만들어진다. 함수는 그 언어에서 동사며, 클래스는 명사다.  
여기서 설명한 규칙을 따른다면 길이가 짧고 이름이 좋고 체계가 잡힌 함수가 나올 것이다.  
하지만 진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다는 사실을 명심하기 바란다.  
우리가 작성하는 함수가 분명하고 정확한 언어로 깔끔하게 같이 맞아떨어져야 이야기를 풀어가기가 쉬어진다는 사실을 기억하기 바란다.
