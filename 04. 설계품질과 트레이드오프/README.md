# 설계 품질과 트레이드오프
책임주도설계 -> 역할, 책임, 협력중에서 가장 중요한 것은 '책임'이다.  
객체지향 설계란 올바른 객체에게 올바른 책임을 할당하면서 낮은 결합도와 높은 응집도를 가진 구조를 창조하는 활동이다.  
-> 객체의 상태가 아니라 객체의 행동에 초점을 맞추자.  
-> 객체를 단순한 데이터의 집합으로 바라보는 시각은 객체의 내부 구현을 퍼블릭 인터페이스에 노출시키는 결과를 낳기 때문에 결과적으로 설계가 변경에 취약해진다.  -> 이 문제를 피하기 위해서는 객체의 책임에 초점을 맞추는것이다.

## 데이터 중심의 영화 예매 시스템
> 상태(데이터)중심 설계와 책임중심의 설계중 어떤것이 좋을까?

### 데이터를 준비하자
#### 데이터 중심의 설계란?
객체 내부에 저장되는 데이터를 기반으로 시스템을 분할하는 방법. '데이터가 무엇인가'를 중심으로

ex) 데이터 중심의 설계 Movie Class
```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions; // 인스턴스 변수로 직접 포함되어있다.
    
    private MovieType movieType; //할인 정책의 종류
    private Money discountAmount;
    private double discountPercent;
    // discountAmount, discountPercent를 직접 내부의 프로퍼티로 가지고있다. (이전에는 DiscountPolicy라는 클래스를 별도로 만들었었다.)
    
    public Movie(String title, Duration runningTime, Money fee, double discountPercent, DiscountCondition... discountConditions) {
        this(MovieType.PERCENT_DISCOUNT, title, runningTime, fee, Money.ZERO, discountPercent, discountConditions);
    }

    public Movie(String title, Duration runningTime, Money fee, Money discountAmount, DiscountCondition... discountConditions) {
        this(MovieType.AMOUNT_DISCOUNT, title, runningTime, fee, discountAmount, 0, discountConditions);
    }

    public Movie(String title, Duration runningTime, Money fee) {
        this(MovieType.NONE_DISCOUNT, title, runningTime, fee, Money.ZERO, 0);
    }

    private Movie(MovieType movieType, String title, Duration runningTime, Money fee, Money discountAmount, double discountPercent,
                  DiscountCondition... discountConditions) {
        this.movieType = movieType;
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountAmount = discountAmount;
        this.discountPercent = discountPercent;
        this.discountConditions = Arrays.asList(discountConditions);
    }

    public MovieType getMovieType() {
        return movieType;
    }

    public void setMovieType(MovieType movieType) {
        this.movieType = movieType;
    }

    public Money getFee() {
        return fee;
    }

    public void setFee(Money fee) {
        this.fee = fee;
    }

    public List<DiscountCondition> getDiscountConditions() {
        return Collections.unmodifiableList(discountConditions);
    }

    public void setDiscountConditions(List<DiscountCondition> discountConditions) {
        this.discountConditions = discountConditions;
    }

    public Money getDiscountAmount() {
        return discountAmount;
    }

    public void setDiscountAmount(Money discountAmount) {
        this.discountAmount = discountAmount;
    }

    public double getDiscountPercent() {
        return discountPercent;
    }

    public void setDiscountPercent(double discountPercent) {
        this.discountPercent = discountPercent;
    }
}

enum MovieType {
    AMOUNDT_DISCOUNT, // 금액할인
    PERCENT_DISCOUNT, // 비율할인
    NONE_DISCOUNT // 미적용
}

```
참조) 기존 책임주도 설계 Movie Class
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
ex) Discount Condition
```
enum DiscountConditionType {
    SEQUENCE, // 순번조건
    PERIOD // 기간조건
}

public class DiscountCondition {
    private DiscountConditionType type;

    private int sequence;

    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public DiscountConditionType getType() {
        return type;
    }

    public void setType(DiscountConditionType type) {
        this.type = type;
    }

    public DayOfWeek getDayOfWeek() {
        return dayOfWeek;
    }

    public void setDayOfWeek(DayOfWeek dayOfWeek) {
        this.dayOfWeek = dayOfWeek;
    }

    public LocalTime getStartTime() {
        return startTime;
    }

    public void setStartTime(LocalTime startTime) {
        this.startTime = startTime;
    }

    public LocalTime getEndTime() {
        return endTime;
    }

    public void setEndTime(LocalTime endTime) {
        this.endTime = endTime;
    }

    public int getSequence() {
        return sequence;
    }

    public void setSequence(int sequence) {
        this.sequence = sequence;
    }
}
```

ex) Screening
```
public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    public Movie getMovie() {
        return movie;
    }

    public void setMovie(Movie movie) {
        this.movie = movie;
    }

    public LocalDateTime getWhenScreened() {
        return whenScreened;
    }

    public void setWhenScreened(LocalDateTime whenScreened) {
        this.whenScreened = whenScreened;
    }

    public int getSequence() {
        return sequence;
    }

    public void setSequence(int sequence) {
        this.sequence = sequence;
    }
}
```

ex) Reservation
```
public class Reservation {
    private Customer customer;
    private Screening screening;
    private Money fee;
    private int audienceCount;

    public Reservation(Customer customer, Screening screening, Money fee,
                       int audienceCount) {
        this.customer = customer;
        this.screening = screening;
        this.fee = fee;
        this.audienceCount = audienceCount;
    }

    public Customer getCustomer() {
        return customer;
    }

    public void setCustomer(Customer customer) {
        this.customer = customer;
    }

    public Screening getScreening() {
        return screening;
    }

    public void setScreening(Screening screening) {
        this.screening = screening;
    }

    public Money getFee() {
        return fee;
    }

    public void setFee(Money fee) {
        this.fee = fee;
    }

    public int getAudienceCount() {
        return audienceCount;
    }

    public void setAudienceCount(int audienceCount) {
        this.audienceCount = audienceCount;
    }
}
```

ex) Customer
```
public class Customer {
    private String name;
    private String id;

    public Customer(String name, String id) {
        this.id = id;
        this.name = name;
    }
}
```
![KakaoTalk_Photo_2021-07-03-21-48-19](https://user-images.githubusercontent.com/60125719/124354753-7e522200-dc48-11eb-9366-97b0bb7f4170.jpeg)


### 영화를 예매하자

ex) ReservationAgency - 데이터 클래스들을 조합해서 영화 예매 절차를 구현하는 클래스
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

## 설계 트레이드오프

### 캡슐화
내부구현을 외부로부터 감추자. 
객체를 사용하면 변경 가능성이 높은 부분은 내부에 숨기고 외부에는 상대적으로 안정적인 부분만 공개함으로써 변경의 여파를 통제할 수 있다.  
> 여기서 변경될 가능성이 높은 부분을 구현이라고 부르고 상대적으로 안정적인 부분을 인터페이스라고 부른다.

### 응집도와 결합도
1. 응집도는 모듈에 포함된 내부 요소들이 연관돼 있는 정도를 나타낸다. 객체지향의 관점에서 응집도는 객체 또는 클래스에 얼마나 관련 높은 책임들을 할당했는지를 나타낸다.
2. 결합도는 의존성의 정도를 나타내며 다른 모듈에 대해 얼마나 많은 지식을 갖고 있는지를 나타낸다. 객체지향에서는 객체 또는 클래스가 협력에 필요한 적절한 수준의 관계만을 유지하고 있어야한다.
3. 좋은 설계란 오늘의 기능을 수행하면서 내일의 변경을 수용할 수 있는 설계이다. 좋은 설계를 위해서는 응집도와 결합도가 포인트임.

#### 응집도와 결합도의 측정방법
 - 응집도: 변경이 발생할 떄 모듈 내부에서 발생하는 변경의 정도로 측정
 - 결합도: 한 모듈이 변경되기 위해서 다른 모듈의 변경을 요구하는 정도

#### 응집도와 변경 사이의 관계
![KakaoTalk_Photo_2021-07-03-22-03-50](https://user-images.githubusercontent.com/60125719/124355118-8e6b0100-dc4a-11eb-8506-7d9ce68f97b0.jpeg)

#### 변경과 결합도
![KakaoTalk_Photo_2021-07-03-22-58-49](https://user-images.githubusercontent.com/60125719/124356691-45b74600-dc52-11eb-83d8-8957bfee0245.jpeg)



## 데이터 중심의 영화 예매 시스템의 문제점
 1. 캡슐화 위반
 2. 높은 결합도
 3. 낮은 응집도
 
### 캡슐화 위반
```
public class Movie {
    private Money fee;

    public Money getFee() {
        return fee;
    }

    public void setFee(Money fee) {
        this.fee = fee;
    }
}
```
-> private 으로 캡슐화의 원칙을 지키고있는것처럼 보이지만 Movie클래스 내부에 Money타입의 fee가 있는것이 명확히 보인다.  
> 이처럼 접근자와 수정자에게 과도하게 의존하는 설계방식을 추측에 의한 설계 전략이라고 한다.

### 높은 결합도

```
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        ...
        Money fee;
        if (discountable) {
            ...

            fee = movie.getFee().minus(discountAmount).times(audienceCount);
        } else {
            fee = movie.getFee().times(audienceCount);
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```
> 만일 fee의 타입이 Money가 아닌 다른 타입으로 변경되야한다면? 함수 내부를 수정해야만 한다. -> 결국 private fee는 캡슐화에 실패했기때문에 public fee와 다를게 없어진다. -> 클라이언트가 객체의 구현에 강하게 결합된다.


![KakaoTalk_Photo_2021-07-05-18-40-24](https://user-images.githubusercontent.com/60125719/124451649-92209400-ddc0-11eb-803d-d114634d6435.jpeg)
> 대부분의 제어로직을 가지고있는 제어 객체인 ReservationAgency가 모든 데이터 객체에 의존을 하고있다.
ReservationAgency는 모든 의존성이 모이는 결합도의 집결지. 시스템 안의 어떤 변경도 ReservationAgency의 변경을 유발한다.

### 낮은 응집도
#### 서로 다른 이유로 변경되는 코드가 하나의 모듈 안에 공존할때 모듈 응집도가 낮다고 말한다.
#### 문제점 (예시) -> 다음과 같은 수정사항이 발생하는 경우에 ReservationAgency의 코드를 수정해야 한다.
 - 할인 정책이 추가 될 경우
 - 할인 정책별로 할인 요금을 계산하는 방법이 변경될 경우
 - 할인 조건이 추가되는 경우
 - 할인 조건별로 할인 여부를 판단하는 방법이 변경될 경우
 - 예매 요금을 계산하는 방법이 변경될 경우

#### 낮은 응집도가 왜 문제가 될까?
 - 어떤 코드를 수정한 후에 아무런 상관도 없던 코드에 문제가 생길 수 있다.
 - 하나의 요구사항 변경을 반영하기 위해 동시에 여러 모듈을 수정해야한다.
 
 
 ## 자율적인 객체를 향해
 ### 캡슐화를 지켜라
 - 단순히 프로퍼티를 private으로 가지고있는것이 아니라 해당 프로퍼티 자체를 밖에서 알지 못하도록 해야한다.
 
 ### 스스로 자신의 데이터를 책임지는 객체
 우리가 상태와 행동을 객체라는 하나의 단위로 묶는 이유는 객체 스스로 자신의 상태를 처리할 수 있게 하기위해서임.  
 따라서 객체를 설계할 때 "이 객체가 어떤 데이터를 포함해야 하는가?" 라는 질문은 다음과 같은 두개의 질문으로 분리해야한다.
  - 이 객체가 어떤 데이터를 포함해야 하는가?
  - 이 객체가 데이터에 대해 수행해야 하는 오퍼레이션은 무엇인가?
  
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

    public DiscountConditionType getType() {
        return type;
    }

    public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime time) {
        if (type != DiscountConditionType.PERIOD) {
            throw new IllegalArgumentException();
        }

        return this.dayOfWeek.equals(dayOfWeek) &&
                this.startTime.compareTo(time) <= 0 &&
                this.endTime.compareTo(time) >= 0;
    }

    public boolean isDiscountable(int sequence) {
        if (type != DiscountConditionType.SEQUENCE) {
            throw new IllegalArgumentException();
        }

        return this.sequence == sequence;
    }
}
```
> 앞에서 DiscountCondition이 관리해야하는 데이터는 결정해 놓았다. 이 데이터에 대해 수행할 수 있는 오퍼레이션을 추가하기 위해 두개의 isDiscountable메서드를 추가했다.

```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;

    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;

    public Movie(String title, Duration runningTime, Money fee, double discountPercent, DiscountCondition... discountConditions) {
        this(MovieType.PERCENT_DISCOUNT, title, runningTime, fee, Money.ZERO, discountPercent, discountConditions);
    }

    public Movie(String title, Duration runningTime, Money fee, Money discountAmount, DiscountCondition... discountConditions) {
        this(MovieType.AMOUNT_DISCOUNT, title, runningTime, fee, discountAmount, 0, discountConditions);
    }

    public Movie(String title, Duration runningTime, Money fee) {
        this(MovieType.NONE_DISCOUNT, title, runningTime, fee, Money.ZERO, 0);
    }

    private Movie(MovieType movieType, String title, Duration runningTime, Money fee, Money discountAmount, double discountPercent,
                  DiscountCondition... discountConditions) {
        this.movieType = movieType;
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountAmount = discountAmount;
        this.discountPercent = discountPercent;
        this.discountConditions = Arrays.asList(discountConditions);
    }

    public MovieType getMovieType() {
        return movieType;
    }

    public Money calculateAmountDiscountedFee() {
        if (movieType != MovieType.AMOUNT_DISCOUNT) {
            throw new IllegalArgumentException();
        }

        return fee.minus(discountAmount);
    }

    public Money calculatePercentDiscountedFee() {
        if (movieType != MovieType.PERCENT_DISCOUNT) {
            throw new IllegalArgumentException();
        }

        return fee.minus(fee.times(discountPercent));
    }

    public Money calculateNoneDiscountedFee() {
        if (movieType != MovieType.NONE_DISCOUNT) {
            throw new IllegalArgumentException();
        }

        return fee;
    }

    public boolean isDiscountable(LocalDateTime whenScreened, int sequence) {
        for(DiscountCondition condition : discountConditions) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                if (condition.isDiscountable(whenScreened.getDayOfWeek(), whenScreened.toLocalTime())) {
                    return true;
                }
            } else {
                if (condition.isDiscountable(sequence)) {
                    return true;
                }
            }
        }

        return false;
    }
}
```
> 요금을 계산하는 오퍼레이션과 할인 여부를 판단하는 오퍼레이션을 추가했다.


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

    public Money calculateFee(int audienceCount) {
        switch (movie.getMovieType()) {
            case AMOUNT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculateAmountDiscountedFee().times(audienceCount);
                }
                break;
            case PERCENT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculatePercentDiscountedFee().times(audienceCount);
                }
            case NONE_DISCOUNT:
                movie.calculateNoneDiscountedFee().times(audienceCount);
        }

        return movie.calculateNoneDiscountedFee().times(audienceCount);
    }
}
```
> Movie가 금액할인이나 비율할인을 지원할경우 적절하게 요금을 계산해준다.


before  
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

after
```
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        Money fee = screening.calculateFee(audienceCount);
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

![KakaoTalk_Photo_2021-07-05-19-01-10](https://user-images.githubusercontent.com/60125719/124454400-694dce00-ddc3-11eb-991a-2257bf235a45.jpeg)
> 결합도 측면에서 이전보다 개선된 설계

## 하지만 여전히 부족하다
문제는?
### 캡슐화 위반

```
public class DiscountCondition {
    private DiscountConditionType type;

    private int sequence;

    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public DiscountConditionType getType() {
        ...
    }

    public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime time) {
        ...
    }

    public boolean isDiscountable(int sequence) {
        ...
    }
}

```
> isDiscountable 의 파라미터를 통해 해당 함수가 어떤 정보를 이용하는지 다 알 수 있다. 다른 함수도 마찬가지 !

```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;

    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;

    public MovieType getMovieType() {
        ...
    }

    public Money calculateAmountDiscountedFee() {
        ...
    }

    public Money calculatePercentDiscountedFee() {
        ...
    }

    public Money calculateNoneDiscountedFee() {
        ...
    }
}
```
>  calculate함수들을 통해서 정책들(내부구현)을 인터페이스에 노출시키고있다.

### 캡슐화의 진정한 의미
캡슐화는 단순이 프로퍼티등을 숨긴다는 의미가 아닌 변경될 수 있는 어떤것이라도 외부로부터 감춘다 라는 의미다.

### 높은 결합도
 DiscountCondition의 내부구현이 밖으로 노출되어있기때문에 Movie클래스와의 결합도가 높다
```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;

    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;

    public boolean isDiscountable(LocalDateTime whenScreened, int sequence) {
        for(DiscountCondition condition : discountConditions) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                if (condition.isDiscountable(whenScreened.getDayOfWeek(), whenScreened.toLocalTime())) {
                    return true;
                }
            } else {
                if (condition.isDiscountable(sequence)) {
                    return true;
                }
            }
        }

        return false;
    }
}

```
 - DiscountCondition의 기간 할인 조건의 명칭이 PERIOD에서 다른 값으로 변경된다면 Movie를 수정해야한다
 - DiscountCondition의 종류가 추가되거나 삭제된다면 Movie안의 if ~ else 구문을 수정해야한다
 - 각 DiscountCondition의 만족 여부를 판단하는 데 필요한 정보가 변경된다면 Movie의 isDiscountable 메서드로 전달된 파라미터를 변경해야 한다. 이로인해 Movie의 isDiscountable 메서드 시그니처도 함께 변경될 것이고 결과적으로 이 메서드에 의존하는 Screening에 대한 변경을 초래할것이다.
 
 ### 낮은 응집도
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

     public Money calculateFee(int audienceCount) {
         switch (movie.getMovieType()) {
             case AMOUNT_DISCOUNT:
                 if (movie.isDiscountable(whenScreened, sequence)) {
                     return movie.calculateAmountDiscountedFee().times(audienceCount);
                 }
                 break;
             case PERCENT_DISCOUNT:
                 if (movie.isDiscountable(whenScreened, sequence)) {
                     return movie.calculatePercentDiscountedFee().times(audienceCount);
                 }
             case NONE_DISCOUNT:
                 movie.calculateNoneDiscountedFee().times(audienceCount);
         }

         return movie.calculateNoneDiscountedFee().times(audienceCount);
     }
 }
 ```
 > 할인 조건을 변경하기위해선 이 클래스도 수정이 되야한다. 하나의 변경을 수용하기 위해 코드의 여러 곳을 동시에 변경해야 한다는 것은 설계의 응집도가 낮다는 뜻이다.
 
 ## 데이터 중심 설계의 문제점
  - 데이터 중심의 설계는 본질적으로 너무 이른 시기에 데이터에 관해 결정하도록 강요한다.
  - 데이터 중심의 설계에서는 협력이라는 문맥을 고려하지 않고 객체를 고립시킨 채 오퍼레이션을 결정한다.
  -> 결과적으로 캡슐화 실패, 응집도와 결합도에 문제가 생김 -> 변경에 취약한 코드를 생산함  
  -> 데이터 중심 설계에서 초점은 객체의 외부가 아닌 내부로 향하게된다. 이미 데이터가 정해져있으니, 메서드들도 끼워맞추기 식으로 개발하게 됨.
