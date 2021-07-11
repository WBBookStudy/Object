# 책임 할당하기
 - 책임을 할당할 때 가장 어려운점은 어떤 객체에게 어떤 책임을 할당할지를 결정하는것
 - 책임 할당 과정은 일종의 트레이드오프 활동
 
## 책임 주도 설계를 향해
데이터 중심의 설계에서 책임 중심의 설계로 전환하기위해서 따라야하는 두가지 원칙
 - 데이터보다 행동을 먼저 결정하라
 - 협력이라는 문맥 안에서 책임을 결정하라

### 데이터보다 행동을 먼저 결정하라
 - 객체에게 중요한 것은 데이터가 아니라 외부에 제공하는 행동이다. (행동이랑 곧 객체의 책임)
 - 데이터는 객체가 책임을 수행하는 데 필요한 재료들일뿐 -> 너무 이른 시기에 데이터에 초점을 맞추면 캡슐화에 실패하고, 결합도가 높아진다.
 - 우리에게 필요한것은? 객체의 데이터에서 행동으로 무게중심을 옮겨야함. 가장 간단한 해결방법은 객체를 설계하기 위한 질문의 순서를 바꿔보자   
  > "이 객체가 포함해야 하는 데이터가 무엇인가" -> "데이터를 처리하는 데 필요한 오퍼레이션은 무엇인가"  
  ->  
  > "이 객체가 수행해야 하는 책임은 무엇인가" -> "이 책임을 수행하는 데 필요한 데이터는 무엇인가"
 - 객체에게 어떤 책임을 할당해야할까? -> 협력에서 찾을 수 있다.
 
 ### 협력이라는 문맥 안에서 책임을 결정하라
  - 객체에게 할당된 책임의 품질은 협력에 적합한 정도로 결정된다. 책임은 객체의 입장이 아니라 객체가 참여하는 협력에 적합해야 한다.
  - 협력을 시작하는 주체는 메시지 전송자이기 때문에 협력에 적합한 책임이란 메시지 수신자가 아니라 메시지 전송자에게 적합한 책임을 의미
  - 협력에 적합한 책임을 얻기 위해서는 객체를 결정한 후 메시지를 선택하는 것이 아니라 메시지를 결정한 후 객체를 선택해야함.
  > "이 클래스가 필요하다는 점은 알겠는데 이 클래스는 무엇을 해야하지?" -> "메시지를 전송해야 하는데 누구에게 전송해야하지?"
  - 메시지가 클라이언트의 의도를 표현하고, 객체를 결정하기 전에 객체가 수신할 메시지를 먼저 결정한다는 점을 기억하자.
  - 메시지를 먼저 결정하기 때문에 메시지 송신자는 메시지 수신자에 대한 어떠한 가정도 할 수 없다 -> 캡슐화 성공
  
 ### 책임 주도 설계
 3장에서 설명한 책임 주도 설계의 흐름 나열
  1. 시스템이 사용자에게 제공해야 하는 기능인 시스템 책임을 파악한다.
  2. 시스템 책임을 더 작은 책임으로 분할한다.
  3. 분할된 책임을 수행할 수 있는 적절한 객체 또는 역할을 찾아 책임을 할당한다.
  4. 객체가 책임을 수행하는 도중 다른 객체의 도움이 필요한 경우 이를 책임질 적절한 객체 또는 역할을 찾는다.
  5. 해당 객체 또는 역할에게 책임을 할당함으로써 두 객체가 협력하게 한다.
  
## 책임 할당을 위한 GRASP 패턴
General Responsibility Assignment Software Pattern(일반적인 책임 할당을 위한 소프트웨어 패턴)

### 도메인 개념에서 출발하기
#### 설계를 시작하기 전에 도메인에 대한 대략적인 모습을 그려보자
![KakaoTalk_Photo_2021-07-11-11-47-17](https://user-images.githubusercontent.com/60125719/125181230-ccb18300-e23d-11eb-8917-3a7e0550a05d.jpeg)
 - 도메인 모개념을 정리하는 데 너무 많은 시간을 들이지 말고 빠르게 설계와 구현을 진행하자
 - 올바른 도메인 모델이란 존재하지 않는다. 필요한것은 도메인을 그대로 투영한 모델이 아니라 구현이 도움이 되는 모델이다.
 
### 정보 전문가에게 책임을 할당하라
 - 책임주도 설계방식의 첫 단계는 어플리케이션이 제공해야 하는 기능을 어플리케이션의 책임으로 생각하는것. 이 책임을 어플리케이션에 대해 전송된 메시지로 간주하고 이 메시지를 책임질 것.
 - 따라서 첫 번째 질문은? -> **"메시지를 전송할 객체는 무엇을 원하는가?"**  
 ex)예매하라
 <img width="190" alt="스크린샷 2021-07-11 오후 1 17 24" src="https://user-images.githubusercontent.com/60125719/125182599-59fad480-e24a-11eb-965d-d59a35a62722.png">
 
  - 메시지를 결정했으니, 메시지에 적합한 객체를 선택해야함. 두번째 질문은? -> **"메시지를 수신할 적합한 객체는 누구인가?"**
  - 객체는 자신의 상태를 스스로 처리하는 자율적인 존재여야 한다. 즉, **객체에게 책임을 할당하는 첫 번째 원칙은 책임을 수행할 정보를 알고 있는 객체에게 책임을 할당하는것.** GRASP에서는 이를 **Information expert(정보전문가) 패턴**이라고 부름
  - information expert 패턴을 사용하게되면 응집도가 올라감으로써 결합도도 낮아지고, 유지보수하기 쉬운 시스템을 구축할 수 있다.
  <img width="436" alt="스크린샷 2021-07-11 오후 1 23 23" src="https://user-images.githubusercontent.com/60125719/125182698-31270f00-e24b-11eb-8474-627f145cc82d.png">
  
  - 예시에서 'Screening'은 '예매하라'라는 메시지를 처리하기 위해서는 다른 객체의 도움이 필요하다.
  <img width="884" alt="스크린샷 2021-07-11 오후 1 28 32" src="https://user-images.githubusercontent.com/60125719/125182799-f2458900-e24b-11eb-8aad-0583ceb29ec3.png">
  
  - Movie클래스 역시 계산을 하기 위해서 다른 객체의 도움이 필요하다.
  <img width="953" alt="스크린샷 2021-07-11 오후 1 30 01" src="https://user-images.githubusercontent.com/60125719/125182811-1dc87380-e24c-11eb-8ecf-9df384217e22.png">
  
  
### 높은 응집도와 낮은 결합도
#### 위의 예시에서 Movie가 DiscountCondition에 할인여부 판단 메시지를 전송하는 대신 Screening이 직접 DiscountCondition과 협력하게 된다면?
![KakaoTalk_Photo_2021-07-11-13-33-49](https://user-images.githubusercontent.com/60125719/125182859-a810d780-e24c-11eb-89dc-886a4caf2d38.jpeg)
> 차이가 없어보인다..?
-> 왜 Movie가 DiscountCondition과 협력하는 방법을 선택했을까?
> 응집도와 결합도 때문
여러가지 설계가 있을텐데 그중 높은 응집도와 낮은 결합도를 얻을 수 있는 설계를 선택하자. 이것을 GRASP에서는 ** Low Coupling(낮은 결합도)** 패턴과 **High Cohesion(높은 응집도)** 패턴이라고 부른다.

### 창조자에게 객체 생성 책임을 할당하라
영화 예매 협럭의 최종 결과물은 Reservation 인스턴스를 생성하는것. 이것은 협력에 참여하는 어떤 객체에게는 Reservation인스턴스를 생성할 책임을 할당해야 하는것.
#### Creator 패턴
객체 A를 생성해야 할 때 어떤 객체에게 객체 생성 책임을 할당해야 하는가? 아래 조건을 최대한 많이 만족하는 B에게 객체 생성 책임을 할당하라
 - B가 A객체를 포함하거나 참조한다.
 - B가 A객체를 기록한다.
 - B가 A객체를 긴밀하게 사용한다.
 - B가 A객체를 초기화 하는데 필요한 데이터를 가지고 있다(이 경우 B는 A에 대한 정보 전문가다)
 > 이미 결합돼 있는 객체에게 생성 책임을 할당하는것은 설계의 전체적인 결합도에 영향을 미치지 않는다. 즉, Creator 패턴은 이미 존재하는 객체 사이의 관계를 이용하기 때문에 설계가 낮은 결합도를 유지할 수 있다.
 
 Reservation을 잘 알고 있거나, 긴밀하게 사용하거나 초기화에 필요한 데이터를 가지고 있는 객체는? -> Screening
 ![KakaoTalk_Photo_2021-07-11-13-43-07](https://user-images.githubusercontent.com/60125719/125183030-f1adf200-e24d-11eb-9d37-09b7b9ca7221.jpeg)
 
  
## 구현을 통한 검증

#### Screening
```
public class Screening {
    private Movie movie; // 계산하라 메시지를 전송해야 함으로 Movie클래스의 객체를 가지고 있어야한다. (참조 포함)
    private int sequence;
    private LocalDateTime whenScreened;

    public Reservation reserve(Customer customer, int audienceCount) { //예메하라 메시지에 응답 할 수 있어야한다.
        return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);  calculateFee로 계산한다음 Reservation을 만들어 반환한다.
    }

    private Money calculateFee(int audienceCount) { 
        return movie.calculateMovieFee(this).times(audienceCount); // 계산 
        // Screening이 Movie내부 구현에 대한 어떤 지식도 없이 전송할 메시지를 결정했다는점을 주목하자.
        // 메시지가 변경되지 않는 한. Movie에 어떤 수정을 가하더라도 Screening에는 영향을 미치지 않는다.
    }

    public LocalDateTime getWhenScreened() {
        return whenScreened;
    }

    public int getSequence() {
        return sequence;
    }
}
```
#### Movie
```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;

    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;
    // 영화의 할인요금을 계산하기 위한 인스턴스 변수이다.

    public Money calculateMovieFee(Screening screening) { // Screening과의 협력 필요로 인해 만들어진 영화요금 계산 함수이다.
        if (isDiscountable(screening)) {
            return fee.minus(calculateDiscountAmount());
        }

        return fee;
    }

    private boolean isDiscountable(Screening screening) { // 순회하면서 요금조건을 반환한다
        return discountConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }

    // 이하는 조건에 따라 요금할인을 계산하는 메서드이다.
    private Money calculateDiscountAmount() {
        switch(movieType) {
            case AMOUNT_DISCOUNT:
                return calculateAmountDiscountAmount();
            case PERCENT_DISCOUNT:
                return calculatePercentDiscountAmount();
            case NONE_DISCOUNT:
                return calculateNoneDiscountAmount();
        }

        throw new IllegalStateException();
    }

    private Money calculateAmountDiscountAmount() {
        return discountAmount;
    }

    private Money calculatePercentDiscountAmount() {
        return fee.times(discountPercent);
    }

    private Money calculateNoneDiscountAmount() {
        return Money.ZERO;
    }
}
```
#### MovieType
```
public enum MovieType {
    AMOUNT_DISCOUNT,    // 금액 할인 정책
    PERCENT_DISCOUNT,   // 비율 할인 정책
    NONE_DISCOUNT       // 미적용
}

```

#### DiscountCondition
```
public class DiscountCondition {
    private DiscountConditionType type;
    private int sequence;
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public boolean isSatisfiedBy(Screening screening) { // Movie클래스와의 협력을 위해 필요한 메서드
        if (type == DiscountConditionType.PERIOD) {
            return isSatisfiedByPeriod(screening);
        }

        return isSatisfiedBySequence(screening);
    }

    // 조건에 따라 적절한 메서드를 호출한다.
    private boolean isSatisfiedByPeriod(Screening screening) {
        return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
                startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                endTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0;
    }

    private boolean isSatisfiedBySequence(Screening screening) {
        return sequence == screening.getSequence();
    }
}

```
#### DiscountConditionType
```
public enum DiscountConditionType {
    SEQUENCE,       // 순번조건
    PERIOD          // 기간 조건
}

```

### DiscountCondition 개선하기
 가장 큰 문제점은 변경에 취약한 클래스를 포함하고 있다는 것이다.  
 변경에 취약한 클래스란? 코드를 수정해야 하는 이유를 둘 이상 가지는 클래스  
 현재 코드에서는 DiscountCondition이 해당됨. 
  - 새로운 할인 조건 추가
  > isSatisfiedBy 메서드 안의 if ~ else 구분을 수정해야한다. 새로운 할인조건이 새로운 데이터를 요구한다면 DiscountCondition에 속성을 추가하는 작업도 필요하다.
  - 순번 조건을 판단하는 로직 변경
  > isSatisfiedBySequence메서드의 내부 구현을 수정해야 한다. 물론 순번 조건을 판단하는 데 필요한 데이터가 변경된다면 DiscountCondition 의 sequence속성 역시 변경해야한다.
  - 기간 조건을 판단하는 로직이 변경되는 경우
  > isSatisfiedByPeriod 메서드의 내부 구현을 수정해야한다. 물론 기간 조건을 판단하는 데 필요한 데이터가 변경된다면 DiscountCondition의 dayOfWeen, startTime, endTime 속성 역시 변경해야한다.
  
  -> 응집도가 낮다. 그렇기 때문에 변경의 이유에 따라 클래스를 분리해야한다.
  > isSatisfiedByPeriod 메서드와 isSatisfiedBySequence는 서로 전혀 다른 이유로 변경될 가능성이 높다. 즉 다른 시간에 서로 각각의 이유로 수정될 가능성이 높다.
  
  #### 코드를 통해 변경의 이유가 두개 이상인 클래스를 찾기
   - 인스턴스 변수가 초기화되는 시점을 살펴보기
   > 응집도가 높은 클래스는 인스턴스를 생성할때 모든 속성을 함께 초기화 하는 반면 응집도가 낮으면 인스턴스 변수의 초기화 시점이 여러군대이다.
   > DiscountCondition클래스를 다시 살펴보면 DiscountCondition이 순번 조건을 표현하는 경우 sequence는 초기화 되지만 dayOfWeek,  startTime, endTime은 초기화 되지 않는다. -> 함께 초기화되는 속성을 기준으로 코드를 분리해아한다.
   - 메서드들이 인스턴스 변수를 사용하는 방식을 살펴보기
   > 모든 메서드가 객체의 모든 속성을 사용한다면 클래스의 응집도는 높다. 반면 메서드들이 사용하는 속성에 따라 그룹이 나뉜다면 클래스의 응집도가 낮다고 볼 수 있다.
   >DisdountCondition클래스의 경우 isSatisfiedBySequence메서드와 isSatisfiedPeriod 메서드가 이 경우에 해당된다. 응집도를 높이기 위해서는 속성 그룹과 해당 그룹에 접근하는 메서드 그룹을 기준으로 코드를 분리해아한다.
   
 #### 클래스 응집도 판단하기(요약정리?)
  - 클래스가 둘 이상의 이유로 변경되어야 한다.
  - 클래스의 인스턴스를 초기화 하는 시점이 두개 이상이다.
  - 메서드 그룹이 속성 그룹을 사용하는지 여부로 나뉜다.
  
### 타입 분리하기
DiscountCondition의 가장 큰 문제는 순번 조건과 기간 조건이라는 두 개의 독립적인 타입이 하나의 클래스 안에 공존하고 있다는 점.  
두 타입을 SequenceCondition과 PeriodCondition이라는 두 개의 클래스로 분리하자

#### PeriodCondition
```
public class PeriodCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }

    public boolean isSatisfiedBy(Screening screening) {
        return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
                startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }
}
```

#### SequenceCondition
```
public class SequenceCondition {
    private int sequence;

    public SequenceCondition(int sequence) {
        this.sequence = sequence;
    }

    public boolean isSatisfiedBy(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```
클래스를 분리하면 앞에서 언급했던 문제를 해결 할 수 있다. 하지만..   
Movie의 인스턴스는 SequenceCondition과 PeriodCondition이라는 두 개의 서로 다른 클래스의 인스턴스 모두와 협력할 수 있어야 한다.
![KakaoTalk_Photo_2021-07-11-13-43-07](https://user-images.githubusercontent.com/60125719/125183804-48b6c580-e254-11eb-87a2-94e5c74c10a9.jpeg)

해결방법은?

#### Movie
```
public class Movie {

    private List<PeriodCondition> periodConditions;
    private List<SequenceCondition> sequenceConditions;

    private boolean isDiscountable(Screening screening) {
        return checkPeriodConditions(screening) ||
                checkSequenceConditions(screening);
    }

    private boolean checkPeriodConditions(Screening screening) {
        return periodConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }

    private boolean checkSequenceConditions(Screening screening) {
        return sequenceConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
}
```
> Movie클래스 안에서 SequenceCondition의 목록과 PeriodConditions의 목록을 따로 유지하는것
#### 위의 클래스의 문제
1. Movie클래스가 PeriodConditions과 SequenceCondition 클래스 양쪽 모두에게 결합된다. 코드 수정전엔 Movie가 DiscoundCondition이라는 하나의 클래스에만 결합돼 있었다.
2. 수정 후에 새로운 할인조건을 추가하기가 더 어려워졌다.
3. 클래스를 분리하기 전에는 DiscountCondition의 내부 구현만 수정하면 Movie에는 아무런 영항을 미치지 않았지만 수정후에는 할인조건을 추가하려면 Movie도 함께 수정해야한다.
-> DiscoountCondition의 입장에서 보면 응집도가 높아졌지만 변경과 캡슐화라는 관점에서 보면 설계의 품질이 나빠져버림

### 다형성을 통해 분리하기
![KakaoTalk_Photo_2021-07-11-15-16-36](https://user-images.githubusercontent.com/60125719/125184711-05ac2080-e25b-11eb-9a2d-84e22bcbfe55.jpeg)

#### DiscountCondition
```
public interface DiscountCondition {
    boolean isSatisfiedBy(Screening screening);
}

```

#### Condition들
```
public class PeriodCondition implements DiscountCondition {
...
}

public class SequenceCondition implements DiscountCondition {
...
}
```

#### Movie
```
public class Movie {
    private List<DiscountCondition> discountConditions;

    public Money calculateMovieFee(Screening screening) {
        if (isDiscountable(screening)) {
            return fee.minus(calculateDiscountAmount());
        }

        return fee;
    }

    private boolean isDiscountable(Screening screening) {
        return discountConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
}
```

> 이렇게 한다면 isSatisfiedBy 메시지만 이해 할 수 있으면 된다.

#### Polymorphism 패턴
>  객체의 타입에 따라 변하는 로직이 있을때는 타입을 명시적으로 정의하고 각 타입에 다형적으로 행동하는 책임을 할당하라. 타입을 검사해서 타입에 따라 여러 대안들을 수행하는 조건적인 논리를 사용하지 말아라

### 변경으로부터 보호하기

위의 그림에서 볼 수 있듯이 DiscountDondition의 두 서브 클래스는 서로 다른 이유로 변경된다.  
새로운 할인조건을 추가하는 경우에도 구체적인 PeriodCondition과 SequenceCondition의 존재를 감추기때문에 Movie가 영형이 받는 일은 없으며, 오직 DiscountCondition의 인터페이스를 실체화하는 클래스를 추가하는것으로만으로도 확장이 가능하다.  이것을 GRASP에서는 **Protected Variations(변경 보호) 패턴** 이라고 부른다

### Movie클래스 개선하기
Movie역시 DiscountCondition과 동일한 문제가 있다. 금액 할인 정책 영화와 비율 할인 정책 영화라는 두 가지 타입을 하나의 클래스 안에 구현하고 있기 때문에 하나 둘 이상의 이유로 변경 될 수 있다.  
Polymorphism패턴을 이용해보자

#### Movie
```
public abstract class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;
    // movieType, discountAmount, discountPercent를 삭제 자식클래스에 할당할 예정

    public Movie(String title, Duration runningTime, Money fee, DiscountCondition... discountConditions) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountConditions = Arrays.asList(discountConditions);
    }

    public Money calculateMovieFee(Screening screening) { // 오버라이딩이 가능하게 만들자
        if (isDiscountable(screening)) {
            return fee.minus(calculateDiscountAmount());
        }

        return fee;
    }

    private boolean isDiscountable(Screening screening) {
        return discountConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }

    protected Money getFee() { // 기본요금이 필요해졌다
        return fee;
    }

    abstract protected Money calculateDiscountAmount();
}

```
#### AmountDiscountMovie
```
public class AmountDiscountMovie extends Movie {
    private Money discountAmount;

    public AmountDiscountMovie(String title, Duration runningTime, Money fee, Money discountAmount,
                               DiscountCondition... discountConditions) {
        super(title, runningTime, fee, discountConditions);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money calculateDiscountAmount() {
        return discountAmount;
    }
}
```
#### PercentDiscountMovie
```
public class PercentDiscountMovie extends Movie {
    private double percent;

    public PercentDiscountMovie(String title, Duration runningTime, Money fee, double percent,
                                DiscountCondition... discountConditions) {
        super(title, runningTime, fee, discountConditions);
        this.percent = percent;
    }

    @Override
    protected Money calculateDiscountAmount() {
        return getFee().times(percent);
    }
}

```
#### NoneDiscountMovie
```
public class NoneDiscountMovie extends Movie {
    public NoneDiscountMovie(String title, Duration runningTime, Money fee) {
        super(title, runningTime, fee);
    }

    @Override
    protected Money calculateDiscountAmount() {
        return Money.ZERO;
    }
}
```
![KakaoTalk_Photo_2021-07-11-15-35-48](https://user-images.githubusercontent.com/60125719/125185115-b4e9f700-e25d-11eb-92bc-cf786dca9fad.jpeg)

#### 도메인의 구조가 코드의 구조를 이끈다.

### 변경과 유연성
#### 개발자가 변경에 대비할 수 있는 방법은?
1. 코드를 이해하고 수정하기 쉽도록 단순하게 설계한다.
2. 코드를 수정하지 않고도 변경을 수용할 수 있도록 코드를 더 유연하게 만든다.
-> **합성**을 이용하자
![KakaoTalk_Photo_2021-07-11-15-40-03](https://user-images.githubusercontent.com/60125719/125185194-49ecf000-e25e-11eb-98cd-8e66fd23510f.jpeg)

#### 코드의 구조가 도메인의 구조에 대한 새로운 통찰력을 제공한다.
![KakaoTalk_Photo_2021-07-11-15-42-11](https://user-images.githubusercontent.com/60125719/125185226-933d3f80-e25e-11eb-9609-804000affdab.jpeg)

## 책임 주도 설계의 대안
일단 동작하는 코드를 작성하고 추후에 수정하자.  
겉으로 보이는 동작은 바꾸지 않은 채 내부 구조를 변경하는것은 리펙토링

### 메서드 응집도

#### ReservationAgency
```
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;
        for(DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
                        condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                        condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
            } else {
                discountable = condition.getSequence() == screening.getSequence();
            }

            if (discountable) {
                break;
            }
        }

        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;
            switch(movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    discountAmount = movie.getFee().times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }

            fee = movie.getFee().minus(discountAmount).times(audienceCount);
        } else {
            fee = movie.getFee().times(audienceCount);
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```
reserve메서드가 너무 길다.. 긴 메서드는 단점이 많다
 - 어떤 일을 수행하는지 한눈에 파악하기 힘들다. 파악하는 시간이 오래걸린다.
 - 하나의 메서드 안에서 너무 많은 작업을 처리하기 때문에 변경이 필요할 때 수정해야 할 부분을 찾기 어렵다.
 - 메서드 내부의 일부 로직만 수정하더라도 메서드의 나머지 부분에서 버그가 발생할 확률이 높다.
 - 로직의 일부만 재사용하는것이 불가능하다.
 - 코드를 재사용하는 유일한 방법은 원하는 코드를 복사해서 붙여넣는 것뿐이므로 코드 중복을 초래하기 쉽다.

응집도 높은 메서드는 변경되는 이유가 다 하나여야 한다.  
객체로 책임을 분배할 때 가장 먼저 할 일은 메서드를 응집도 있는 수준으로 분해하는 것이다.

#### 개선된 ReservationAgency
```
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        boolean discountable = checkDiscountable(screening);
        Money fee = calculateFee(screening, discountable, audienceCount);
        return createReservation(screening, customer, audienceCount, fee);
    }

    private Money calculateFee(Screening screening, boolean discountable,
                               int audienceCount) {
        if (discountable) {
            return screening.getMovie().getFee()
                    .minus(calculateDiscountedFee(screening.getMovie()))
                    .times(audienceCount);
        }

        return  screening.getMovie().getFee();
    }

    private Money calculateDiscountedFee(Movie movie) {
        switch(movie.getMovieType()) {
            case AMOUNT_DISCOUNT:
                return calculateAmountDiscountedFee(movie);
            case PERCENT_DISCOUNT:
                return calculatePercentDiscountedFee(movie);
            case NONE_DISCOUNT:
                return calculateNoneDiscountedFee(movie);
        }

        throw new IllegalArgumentException();
    }

    private Money calculateAmountDiscountedFee(Movie movie) {
        return movie.getDiscountAmount();
    }

    private Money calculatePercentDiscountedFee(Movie movie) {
        return movie.getFee().times(movie.getDiscountPercent());
    }

    private Money calculateNoneDiscountedFee(Movie movie) {
        return movie.getFee();
    }

    private Reservation createReservation(Screening screening,
                                          Customer customer, int audienceCount, Money fee) {
        return new Reservation(customer, screening, fee, audienceCount);
    }
    
    private boolean isDiscountable(Screening screening) {
        if (type == DiscountConditionType.PERIOD) {
            return isSatisfiedByPeriod(screening);
        }

        return isSatisfiedBySequence(screening);
    }

    private boolean isSatisfiedByPeriod(Screening screening) {
        return screening.getWhenScreened().getDayOfWeek().equals(dayOfWeek) &&
                startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }

    private boolean isSatisfiedBySequence(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```
> 메서드 들의 응집도 자체는 높아졌지만 이 메서드들을 담고 있는 ReservationAgency의 응집도는 여전히 낮다. 하지만 변경이 유연해지고 이해하기가 쉬워진다.

### 객체를 자율적으로 만들자

#### DiscountCondition
```
public class DiscountCondition {
    private DiscountConditionType type;

    private int sequence;

    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public DiscountCondition(int sequence){
        this.type = DiscountConditionType.SEQUENCE;
        this.sequence = sequence;
    }

    public DiscountCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime){
        this.type = DiscountConditionType.PERIOD;
        this.dayOfWeek= dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }

    public boolean isDiscountable(Screening screening) {
        if (type == DiscountConditionType.PERIOD) {
            return isSatisfiedByPeriod(screening);
        }

        return isSatisfiedBySequence(screening);
    }

    private boolean isSatisfiedByPeriod(Screening screening) {
        return screening.getWhenScreened().getDayOfWeek().equals(dayOfWeek) &&
                startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }

    private boolean isSatisfiedBySequence(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```

#### 개선된 ReservationAgency
```
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        boolean discountable = checkDiscountable(screening);
        Money fee = calculateFee(screening, discountable, audienceCount);
        return createReservation(screening, customer, audienceCount, fee);
    }

    private boolean checkDiscountable(Screening screening) { // 체크하기 위해 추가했다.
        return screening.getMovie().getDiscountConditions().stream()
                .anyMatch(condition -> condition.isDiscountable(screening));
    }

    private Money calculateFee(Screening screening, boolean discountable,
                               int audienceCount) {
        if (discountable) {
            return screening.getMovie().getFee()
                    .minus(calculateDiscountedFee(screening.getMovie()))
                    .times(audienceCount);
        }

        return  screening.getMovie().getFee();
    }

    private Money calculateDiscountedFee(Movie movie) {
        switch(movie.getMovieType()) {
            case AMOUNT_DISCOUNT:
                return calculateAmountDiscountedFee(movie);
            case PERCENT_DISCOUNT:
                return calculatePercentDiscountedFee(movie);
            case NONE_DISCOUNT:
                return calculateNoneDiscountedFee(movie);
        }

        throw new IllegalArgumentException();
    }

    private Money calculateAmountDiscountedFee(Movie movie) {
        return movie.getDiscountAmount();
    }

    private Money calculatePercentDiscountedFee(Movie movie) {
        return movie.getFee().times(movie.getDiscountPercent());
    }

    private Money calculateNoneDiscountedFee(Movie movie) {
        return movie.getFee();
    }

    private Reservation createReservation(Screening screening,
                                          Customer customer, int audienceCount, Money fee) {
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```
