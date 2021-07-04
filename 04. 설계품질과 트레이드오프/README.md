# 설계 품질과 트레이드오프
책임주도설계 -> 역할, 책임, 협력중에서 가장 중요한 것은 '책임'이다.  
객체지향 설계란 올바른 객체에게 올바른 책임을 할당하면서 낮은 결합도와 높은 응집도를 가진 구조를 창조하는 활동이다.  
-> 객체의 상태가 아니라 객체의 행동에 초점을 맞추자.  
-> 객체를 단순한 데이터의 집합으로 바라보는 시각은 객체의 내부 구현을 퍼블릭 인터페이스에 노출시키는 결과를 낳기 때문에 결과적으로 설계가 변경에 취약해진다.  -> 이 문제를 피하기 위해서는 객체의 책임에 초점을 맞추는것이다.

## 데이터 중심의 영화 에매 시스템
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



