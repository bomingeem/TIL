9장. 도메인 모델과 BOUNDED CONTEXT
===========

### 도메인 모델과 경계

- 모델은 특정 컨텍스트(문맥)하에서 완전한 의미를 갖는다.
- 같은 제품이라도 카탈로그 컨텍스트와 재고 컨텍스트의 의미가 서로 다르다. 이렇게 구분되는 경계를 갖는 컨텍스트를 ‘BOUNDED CONTEXT**’**라고 부른다.

### BOUNDED CONTEXT

- 모델의 경계를 결정하며 한 개의 BOUNDED CONTEXT는 논리적으로 한 개의 모델을 갖는다.
- 실제로 사용자에게 기능을 제공하는 물리적 시스템으로 도메인 모델은 이 BOUNDED CONTEXT 안에서 도메인을 구현한다.
- 조직 구조에 따라 BOUNDED CONTEXT가 결정된다.
- 여러 하위 도메인을 하나의 BOUNDED CONTEXT에서 개발할 때 주의할 점은 하위 도메인의 모델이 뒤섞이지 않도록 하는 것이다.
- 비록 한 개의 BOUNDED CONTEXT에서 여러 하위 도메인을 포함하더라도 **하위 도메인마다 구분되는 패키지를 갖도록 구현**해야 하위 도메인을 위한 모델이 서로 뒤섞이지 않아서 하위 도메인마다 BOUNDED CONTEXT를 갖는 효과를 낼 수 있다.
- BOUNDED CONTEXT는 각자 구현하는 하위 도메인에 맞는 모델을 갖는다.

### BOUNDED CONTEXT의 구현

- BOUNDED CONTEXT는 도메인 모델뿐만 아니라 도메인 기능을 사용자에게 제공하는 데 필요한 표현 영역, 응용 서비스, 인프라 영역 등을 모두 포함한다.
- 각 BOUNDED CONTEXT는 도메인에 알맞은 아키텍처를 사용한다.
- 한 BOUNDED CONTEXT에서 두 방식을 혼합해서 사용할 수도 있다. 대표적인 예가 CQRS 패턴이다.
- BOUNDED CONTEXT는 UI를 갖지 않을 수도 있으며 UI 서버를 통해 간접적으로 브라우저와 통신해서 사용자 요청을 처리하는 방법도 있다.

### BOUNDED CONTEXT 간 통합

- 두 팀이 관련된 BOUNDED CONTEXT를 개발하면 자연스럽게 두 BOUNDED CONTEXT간 통합이 발생한다.
- 카탈로그와 추천 BOUNDED CONTEXT 간 통합이 필요한 기능은 다음과 같다.
  - 사용자가 제품 상세 페이지를 볼 때, 보고 있는 상품과 유사한 상품 목록을 하단에 보여준다.

![외부연동](https://user-images.githubusercontent.com/47099798/231065201-4545be42-216e-4abb-a3ad-80fd16751f21.png)

- RecSystemClient는 REST API로부터 데이터를 읽어와 카탈로그 도메인에 맞는 상품 모델로 변환한다.
- REST API를 호출하는 것은 두 BOUNDED CONTEXT를 직접 통합하는 방법이다.
- 간접적으로 통합하는 방법도 있는데 대표적인 간접 통합 방식이 메시지 큐를 사용하는 것이다.
- 두 BOUNDED CONTEXT를 개발하는 팀은 메시징 큐에 담을 데이터의 구조를 협의하게 되는데 그 큐를 누가 제공하느냐에 따라 데이터 구조가 결정된다.

### BOUNDED CONTEXT 간 관계

- BOUNDED CONTEXT는 어떤 식으로든 연결되기 때문에 두 BOUNDED CONTEXT는 다양한 방식으로 관계를 맺는다.
- 두 관계 중 가장 흔한 관계는 한쪽에서 API를 제공하고 다른 한쪽에서 그 API를 호출하는 관계이다. REST API가 대표적이다.
- 이 관계에서 API를 사용하는 BOUNDED CONTEXT는 API를 제공하는 BOUNDED CONTEXT에 의존하게 된다.
- 상류 컴포넌트는 일종의 서비스 공급자 역할을 하며, 하류 컴포넌트는 그 서비스를 사용하는 고객 역할을 한다.
- 따라서, 상류 팀과 하류 팀은 개발 계획을 서로 공유하고 일정을 협의해서 결정해야 한다.
- 서비스를 제공할 하류 컴포넌트가 많다면 각 하류 팀의 요구사항을 수용해서 이를 단일 서비스로 공개한다. 이런 서비스를 가리켜 **‘공개 호스트 서비스(OPEN HOST SERVICE)’**라고 한다.
- RecSystemClient는 외부 시스템과의 연동을 처리하는데 외부 시스템의 도메인 모델이 내 도메인 모델을 침범하지 않도록 막아주는 역할을 한다. 즉 **‘안티코럽션 계층(Anticorruption Layer)’**이 된다.
- 두 BOUNDED CONTEXT가 같은 모델을 공유하는 경우도 있는데 이를 공유 커널이라고 부른다.
  - 중복을 줄여주지만 두 팀이 한 모델을 공유하기 때문에 한 팀에서 임의로 모델을 변경해서는 안 되며 두 팀이 밀접한 관계를 유지해야 한다.
- 독립 방식 관계는 두 BOUNDED CONTEXT 간에 통합을 하지 않으므로 서로 독립적으로 모델을 발전시킨다.
- 독립 방식에서는 수동으로 BOUNDED CONTEXT를 통합한다.
- 독립 방식으로 개발한 두 BOUNDED CONTEXT를 통합하기 위해 별도의 시스템을 만들어야 할 수도 있다.

### 컨텍스트 맵

- 컨텍스트 맵은 시스템의 전체 구조를 보여준다. 이는 하위 도메인과 일치하지 않는 BOUNDED CONTEXT를 찾아 도메인에 맞게 BOUNDED CONTEXT를 조절하고 사업의 핵심 도메인을 위해 조직 역량을 어떤 BOUNDED CONTEXT에 집중할지 파악하는 데 도움을 준다.

![컨텍스트맵](https://user-images.githubusercontent.com/47099798/231065234-17fd74c7-5132-40dd-a305-3551a14c70b2.png)

