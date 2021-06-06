# 객체지향 프로그래밍
> 진정한 객체지햐 패러다임으로의 전환은 클래스가 아닌 객체에 초점을 맞출 때에만 얻을 수 있다.  
 - 어떤 클래스가 필요한지를 고민하기 전에 어떤 객체들이 필요한지 고민하라.  
 - 객체를 독립적인 존재가 아니라 기능을 구현하기 위해 협력하는 공동체의 일원으로 봐야한다.

## 도메인
 문제를 해결하기 위해 사용자가 프로그램을 사용하는 분야

## 예제 UML
<img width="754" alt="스크린샷 2021-06-04 오후 8 17 28" src="https://user-images.githubusercontent.com/60125719/120793629-15f12180-c572-11eb-97b2-fb4d45e31be6.png">

## Screening Class
```
public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
        this.movie = movie;
        this.sequence = sequence;
        this.whenScreened = whenScreened;
    }

    public LocalDateTime getStartTime() {
        return whenScreened;
    }

    public boolean isSequence(int sequence) {
        return this.sequence == sequence;
    }

    public Money getMovieFee() {
        return movie.getFee();
    }

    public Reservation reserve(Customer customer, int audienceCount) {
        return new Reservation(customer, this, calculateFee(audienceCount),
                audienceCount);
    }

    private Money calculateFee(int audienceCount) {
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}
```
> 눈여겨 봐야하는것은 변수의 접근제어는 private, 함수의 접근제어는 public인것. 경계의 명확성이 객체의 자율성을 보장한다. + 프로그래머에게 구현의 자유를 제공한다.

### 자율적인 객체
객체가 상태(state)와 행동(behavior)을 함께 가지는 복합적인 존재. 객체는 스스로 판단하고 행동하는 자율적인 존재.  
접근제어자를 활용해 캡슐화를 시키자.

### 프로그래머의 자유
프로그래머의 역할을 클래스 작성자(class creator)와 클라이언트 프로그래머(client programmer)로 구분하자.  
클래스 작성자는 새로운 데이터 타입을 프로그램에 추가하고, 클라이언트 프로그래머는 클래스 작성자가 추가한 데이터 타입을 사용한다.  
접근 제어자를 통해서!  
객체의 외부와 내부를 구분하면 클라이언트 프로그래머가 알아야 할 지식의 양이 줄어들고 클래스 작성자가 자유롭게 구현을 변경할 수 있는 폭이 넓어진다.

### 협력하는 객체들의 공동체

- Money Class
```
public class Money {
    public static final Money ZERO = Money.wons(0);

    private final BigDecimal amount;

    public static Money wons(long amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public static Money wons(double amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    Money(BigDecimal amount) {
        this.amount = amount;
    }

    public Money plus(Money amount) {
        return new Money(this.amount.add(amount.amount));
    }

    public Money minus(Money amount) {
        return new Money(this.amount.subtract(amount.amount));
    }

    public Money times(double percent) {
        return new Money(this.amount.multiply(
            BigDecimal.valueOf(percent)
        ));
    }

    public boolean isLessThan(Money other) {
        return amount.compareTo(other.amount) < 0;
    }

    public boolean isGreaterThanOrEqual(Money other) {
        return amount.compareTo(other.amount) >= 0;
    }
}
```

 - Reservation Class
```
public class Reservation {
    private Customer customer;
    private Screening Screening;
    private Money fee;
    private int audienceCount;

    public Reservation(Customer customer, Screening Screening, Money fee, int audienceCount) {
        this.customer = customer;
        this.Screening = Screening;
        this.fee = fee;
        this.audienceCount = audienceCount;
    }
}
```

영화를 예매하기 위해 Screening, Movie, Reservation 인스턴스들은 서로의 메서드를 호출하며 상호작용을 한다.
![KakaoTalk_Photo_2021-06-04-20-57-21](https://user-images.githubusercontent.com/60125719/120797817-79ca1900-c577-11eb-8c1b-150f8a48f258.jpeg)

#### 협력에 관한 짧은 이야기
객체는 다른 객체의 인터페이스에 공개된 행동을 수행하도록 요청(request)할 수 있다. 요청을 받은 객체는 자율적인 방법에 따라 요청을 처리한 후 응답(response)한다.  
객체가 다른 객체와 상호작용할 수 있는 유일한 방법은 메시지를 전송(send a message)하는 것뿐이다. 다른 객체에게 요청이 도착할 때 해당 객체가 메시지를 수신(receive a message)했다고 이야기 한다.   
-> 수신된 메시지를 처리하기 위한 자신만의 방법 : method


## 할인 요금 계산을 위한 협력 시작하기
```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```
-> 코드상에 할인 정책이 존재하지 않는다.

## 할인 정책과 할인 조건
할인정책은 금액 할인 정책과 비율 할인 정책 두가지가 있다.  
각각 AmountDiscountPolicy와 PercentDiscountPolicy로 구현할 예정. 
부모 클래스인 DiscountPlicy안에 중복 코드를 두고 상속받게 할것이다.  

```
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DiscountPolicy(DiscountCondition ... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    public Money calculateDiscountAmount(Screening screening) {
        for(DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) { // 할인 요금을 체크한다.
                return getDiscountAmount(screening);
            }
        }

        return Money.ZERO;
    }

    abstract protected Money getDiscountAmount(Screening Screening);
}
```
> DiscountPolicy는 비지니스 로직의 흐름은 제공하지만 실제 구현은 이를 상속받는 클래스에게 맡긴다. 이처럼 부모 클래스에 기본적인 알고리즘의 흐름을 구현하고 중간에 필요한 처리를 자식클래스에게 위임하는 디자인 패턴을 TEMPLATE METHOD패턴이라고 부른다.

 - DiscountCondition
```
public interface DiscountCondition {
    boolean isSatisfiedBy(Screening screening);
}

```

 - SequenceCondition
 ```
 public class SequenceCondition implements DiscountCondition {
    private int sequence;

    public SequenceCondition(int sequence) {
        this.sequence = sequence;
    }

    public boolean isSatisfiedBy(Screening screening) {
        return screening.isSequence(sequence);
    }
}
 ```

 - PeriodCondition
```
public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }

    public boolean isSatisfiedBy(Screening screening) {
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
                startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
                endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
    }
}
```

 - AmountDiscountPolicy
```
public class AmountDiscountPolicy extends DiscountPolicy {
    private Money discountAmount;

    public AmountDiscountPolicy(Money discountAmount, DiscountCondition... conditions) {
        super(conditions);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return discountAmount;
    }
}
```

- PercentDiscountPolicy
```
public class PercentDiscountPolicy extends DiscountPolicy {
    private double percent;

    public PercentDiscountPolicy(double percent, DiscountCondition... conditions) {
        super(conditions);
        this.percent = percent;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return screening.getMovieFee().times(percent);
    }
}

```

![KakaoTalk_Photo_2021-06-04-21-18-59](https://user-images.githubusercontent.com/60125719/120800090-8308b500-c57a-11eb-9a25-89cef7dd69c6.jpeg)

## 할인 정책 구성하기
하나의 영화에 대해 단 하나의 할인 정책만 설정할 수 있지만 할인 조건의 경우에는 여러개를 적용 할 수 있다
-> 
```
public class Movie {
    ...

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }
    ...
}
```

```
public abstract class DiscountPolicy {
    ...

    public DiscountPolicy(DiscountCondition ... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    ...
}
```

실제 사용 예제)

```
Movie avatar = new Movie("아바타", 
                        Duration.ofMinutes(120),
                        Money,wons(10000),
                        new AmountDiscountPolicy(Money,wons(800),
                                                new SequenceCondition(1),
                                                new SequenceCondition(10),
                                                new PeriodCondition(DayOfWeek.MONDAY, LocalTime.of(10, 0), LocalTime.of(11,59)),
                                                new PeriodCondition(DayOfWeek.THURSDAY, LocalTime.of(10, 0), LocalTime.of(20,59))
                        ));
```


## 컴파일 시간 의존성과 실행 시간 의존성
![KakaoTalk_Photo_2021-06-06-15-41-53](https://user-images.githubusercontent.com/60125719/120915267-cd567700-c6dd-11eb-8287-cdea3e1b9ffd.jpeg)
Movie는 실제 할인정책을 알지 못하는데 어떻게 할인을 적용할까?
```
Movie avatar = new Movie("아바타", 
                        Duration.ofMinutes(120),
                        Money.wons(10000),
                        new AmoutDiscountPolicy(Money.wons(800), ...));
```
-> 코드의 의존성과 실행시점의 의존성이 서로 다를 수 있다.

## 차이에 의한 프로그래밍(programming by difference)
### 부모 클래스와 다른 부분만을 추가해서 새로운 클래스를 쉽고 빠르게 만드는 방법

## 상속과 인터페이스

자식 클래스는 부모 클래스가 수신할 수 있는 모든 메시지를 수신 할 수 있다. -> 외부 객체는 자식 클래스를 부모클래스와 동일한 타입으로 간주할 수 있다.  
ex)
```
public Money calculateMovieFee(Screening screening) { 
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
}
// Movie 클래스에 존재하는 이 메서드는 Movie클래스를 상속받는 모든 클래스에 존재한다.
```
### 이처럼 자식 클래스가 부모 클래스를 대신하는 것을 업캐스팅(upcasting)이라고 부른다.

## 다형성
위에서 언급한 케이스를 예시로 들면. 
Movie 클래스는 DiscountPolicy에게 메시지를 전송하지만 실제 실행되는 메서드는 PercentDiscount인지 AmountDiscount인지에따라 달라진다.  
-> 다형성

## 인터페이스와 다형성
위에서 언급한 상속을 이용한 다형성과는 다르게  
순수하게 인터페이스만으로 다형성을 구현할 수 있다.

## 추상화의 힘
DiscoutPolicy는 AmountDiscountPolicy와 PercentDiscountPolicy보다 추상적이다.  
프로그래밍 언어 측면에서 DiscountPolicy가 더 추상적인 이유는 인터페이스에 초점을 맞췄기 때문.  
![KakaoTalk_Photo_2021-06-06-16-00-02](https://user-images.githubusercontent.com/60125719/120915735-769e6c80-c6e0-11eb-9240-9a2f7a529e9c.jpeg)
### 추상화의 장점
 - 추상화의 계층만 따로 떼어놓고 살펴보면 요구사항의 정책을 높은 수준에서 서술할 수 있다.
 - 추상화를 이용하면 설계가 좀 더 유연해진다.
 > 디자인패턴, 프레임워크의 개념도 이 개념을 이용한다.
 
 ## 유연한 설계
 
 ex) 스타워즈
 ```
public class Movie {
    public Money claculateMovieFee(Screening screening) {
        if (discountPolicy == null) {
            return fee;
        }
        
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
 ```
 > 할인 정책이 없는 경우를 예외케이스로 취급하기 때문에 일관성이 무너진다.
 -> 이 경우에 일관성을 지킬 수 있는 방법은 0원이라는 할인 요금을 계산할 책임을 그대로 DiscountPolicy계층에 유지 시키는 것이다.
 ```
 public class NoneDiscountPolicy extends DiscountPolicy {
     @Override
     protected Money getDiscountAmount(Screening screening) {
         return Money.ZERO;
     }
 }
 ```
 
 이제 Movie의 인스턴스에 NoneDiscountpolicy의 인스턴스를 연결해서 할인되지 않는 영화를 생성할 수 있다.
 ```
 Movie starWars = new Movie("스타워즈",
                            Duration.ofMinutes(210),
                            Money.wons(10000),
                            new NoneDiscountPolicy());
 ```
 ![KakaoTalk_Photo_2021-06-06-16-13-18](https://user-images.githubusercontent.com/60125719/120916046-258f7800-c6e2-11eb-8d75-402d8b8420c7.jpeg)
유연성이 필요한 곳에 추상화를 사용하라

## 추상 클래스와 인터페이스 트레이드오프

```
public interface DiscountPolicy {
    Money calculateDiscountAmount(Screening screening);
}
```

DiscountPolicy -> DefaultDiscountPolicy
```
public abstract class DefaultDiscountPolicy implements DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DefaultDiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    @Override
    public Money calculateDiscountAmount(Screening screening) {
        for(DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }

        return Money.ZERO;
    }

    abstract protected Money getDiscountAmount(Screening Screening);
}
```

```
public class NoneDiscountPolicy implements DiscountPolicy {
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        return Money.ZERO;
    }
}
```
![KakaoTalk_Photo_2021-06-06-16-22-29](https://user-images.githubusercontent.com/60125719/120916240-68058480-c6e3-11eb-908d-80404772de1b.jpeg)

## 코드 재사용
### 코드 재사용을 위해서는 상속보다 합성이 더 좋은방법
> 합성? 다른 객체의 인스턴스를 자신의 인스턴스 변수로 포함해서 재사용하라
-> 위의 예제에서 Movie가 DiscountPlicy의 코드를 재사용 하는 방법

## 상속
### 상속의 단점
 - 상속은 캡슐화를 위반한다.
 - 설계를 유연하지 못하게 만든다.
 
 #### 설계를 유연하지 못하게 만든다의 예시
 실행 시점에 금액 할인 정책인 영화를 비율 할인 정책으로 변경한다고 한다면?  
 상속을 사용한다면 AmountDiscountMovie의 인스턴스를 PercentDiscountMovie의 인스턴스로 변경해야한다.  
 인스턴스 변수로 연결한 기존 방법을 사용한다면?
 
 ```
public class Movie {
    private DiscountPolicy discountPolicy;
    
    public void changeDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
 ```
 ```
 Movie avatar = new Movie("아바타", 
                         Duration.ofMinutes(120),
                         Money.wons(10000),
                         new AmoutDiscountPolicy(Money.wons(800), …));
avatar.changeDiscountPolicy(new PercentDiscountPolicy(0.1 …));
 ```
 
 ## 합성
 ### 인터페이스에 정의된 메시지를 통해서만 코드를 재사용하는방법
 ### 합성은 상속이 지니는 두가지 단점을 모두 해결 할 수 있다.
