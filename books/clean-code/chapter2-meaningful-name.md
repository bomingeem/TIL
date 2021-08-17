2장. 의미 있는 이름
===========

### 들어가면서
이름을 잘 지으면 여러모로 편하다. 이 장에서는 이름을 잘 짓는 간단한 규칙을 몇 가지 소개한다.  


### 의도를 분명히 밝혀라
좋은 이름을 지으려면 시간이 걸리지만 좋은 이름으로 절약하는 시간이 훨씬 더 많다. 그러므로 이름을 주의깊게 살펴 더 나은 이름이 떠오르면 개선하기 바란다.  
변수의 존재 이유는? 수행 기능은? 사용 방법은? 에 대한 물음이 따로 주석이 필요하지 않아도 변수나 함수 그리고 클래스 이름에 드러나야 한다.
아래와 같이 예시를 들었다.


```
//bad case
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<int[]>();
    for (int[] x : theListr) {
        if (x[0] == 4)
          list1.add(x);
        return list1;
    }
}
```


```
//good case
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for (int[] cell : gameBoard)
        if (cell[STATUS_VALUE] == FLAGGED)
          flaggedCells.add(cell);
        return flaggedCells;
}
```

단순히 이름만 고쳤는데도 함수가 하는 일을 이해하기 쉬워졌다. 바로 이것이 좋은 이름이 주는 위력이다.  

### 그릇된 정보를 피하라
프로그래머는 코드에 그릇된 단서를 남겨서는 안된다. 그릇된 단서는 코드 의미를 흐린다.  
나름대로 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해도 안된다.  
 - 프로그래머에게 List라는 단어는 특수한 의미다. 계정을 담는 컨테이너가 실제 List가 아니라면 프로그래머에게 그릇된 정보를 제공하는 셈이다. 그러므로 accountGroup, bunchOfAccounts, 아니면 Accounts라 명명한다.  
 - 서로 흡사한 이름을 사용하지 않도록 주의한다.


### 의미 있게 구분하라
 - 컴파일러를 통과할지라도 연속된 숫자를 덧붗이거나 불용어(noise word)를 추가하는 방식은 적절하지 못하다.  
 - 연속적인 숫자를 덧붙인 이름(a1, a2, .. aN)은 의도적인 이름과 정반대다.  
 - 클래스 이름에 Info, Data와 같은 불용어를 붙이지 말자.
 - 명확한 관례가 없다면 변수 moneyAmount는 money와 구분이 안된다. customerInfo는 customer와, accountData는 account와, theMessage는 message와 구분이 안된다.  

### 발음하기 쉬운 이름을 사용하라 
발음하기 쉬운 이름을 선택한다.

```
//bad case
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
};
```


```
//good case
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
};
```

### 검색하기 쉬운 이름을 사용하라
 - 이름 길이는 범위 크기레 비례해야 한다.


### 인코딩을 피하라
* 헝가리식 표기법
  변수명에 해당 변수의 타입을 적지 말자.  
* 멤버 변수 접두어
  멤버 변수 접두어를 사용하지 말자. ex: m_dsc;  
* 인터페이스 클래스와 구현 클래스
  개인적으로 인터페이스 이름은 접두어를 붙이지 않는 편이 좋다고 생각한다.  
  옛날 코드에서 많이 사용하는 접두어 I는 주의를 흐트리고 과도한 정보를 제공한다.

