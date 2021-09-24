# 일관성 있는 협력
가능하면 유사한 기능을 구현하기 위해 유사한 협력 패턴을 사용하자.  

## 01. 핸드폰 과금 시스템 변경하기

### 기본 정책 확장

 |:---:|:---:|:---:|
 | 유형 | 형식 | 예 |
 | 고정요금 방식 | A초당 B원 | 10초당 18원 |
 | 요일별 방식 | 평일에는 A초당 B원 | 평일에는 10초당 38원 |
 | 구간별 방식 | 초기 A분 동안 B초당 C원, A분~D분까지 B초당 D원 | 초기 1분동안 10초당 50원 초기 1분 이후 10초당 20원 |


 - 고정요금 방식: 일정 시간 단위로 동일한 요금을 부과하는 방식.
 - 시간대별 방식: 하루 24시간을 특정한 시간 구간으로 나눈 후 각 구간별로 서로 다른 요금을 부과하는 방식.
 - 요일별 방식: 요일별로 요금을 차등 부과하는 방식
 - 구간별 방식: 전체 통화 시간을 일정한 통화 시간에 따라 나누고 각 구간별로 요금을 차등 부과하는 방식
 
![KakaoTalk_Photo_2021-09-24-14-51-38](https://user-images.githubusercontent.com/60125719/134625225-8b5fa12e-ba75-4c46-b81d-3eff75102c66.jpeg)
> 경우의 수

![KakaoTalk_Photo_2021-09-24-14-56-48](https://user-images.githubusercontent.com/60125719/134625359-40e31e94-87b8-44fa-b4a3-ddae9ae42b3a.jpeg)
> 이대로 만들것임

### 고정요금 방식 구현하기

```Java
public class FixedFeePolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;

    public FixedFeePolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```
> 기존의 요금제를 이름만 바꿔놓음


### 시간대별 방식 구현하기

![KakaoTalk_Photo_2021-09-24-15-46-48 001](https://user-images.githubusercontent.com/60125719/134630426-845bab66-ef17-4ec5-8bad-060bb686279f.jpeg)
![KakaoTalk_Photo_2021-09-24-15-46-49 002](https://user-images.githubusercontent.com/60125719/134630442-3a6e6976-d996-40e4-a54e-3236c079f0cf.jpeg)
![KakaoTalk_Photo_2021-09-24-15-46-49 003](https://user-images.githubusercontent.com/60125719/134630446-f323b72b-5e48-45c4-b4e4-a491cd0fbdbf.jpeg)
> 시간대별 방식의 통화 요금을 계산하기 위해서는 통화의 시작 시간과 종료 시간뿐만 아니라 시작 일자와 종료 일자도 함꼐 고려해야한다.

```Java
public class DateTimeInterval {
    private LocalDateTime from;
    private LocalDateTime to;

    public static DateTimeInterval of(LocalDateTime from, LocalDateTime to) {
        return new DateTimeInterval(from, to);
    }

    public static DateTimeInterval toMidnight(LocalDateTime from) {
        return new DateTimeInterval(from, LocalDateTime.of(from.toLocalDate(), LocalTime.of(23, 59, 59, 999_999_999)));
    }

    public static DateTimeInterval fromMidnight(LocalDateTime to) {
        return new DateTimeInterval(LocalDateTime.of(to.toLocalDate(), LocalTime.of(0, 0)), to);
    }

    public static DateTimeInterval during(LocalDate date) {
        return new DateTimeInterval(
                LocalDateTime.of(date, LocalTime.of(0, 0)),
                LocalDateTime.of(date, LocalTime.of(23, 59, 59, 999_999_999)));
    }

    private DateTimeInterval(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }

    public Duration duration() {
        return Duration.between(from, to);
    }

    public LocalDateTime getFrom() {
        return from;
    }

    public LocalDateTime getTo() {
        return to;
    }

    public List<DateTimeInterval> splitByDay() {
        if (days() > 0) {
            return split(days());
        }

        return Arrays.asList(this);
    }

    private long days() {
        return Duration.between(from.toLocalDate().atStartOfDay(), to.toLocalDate().atStartOfDay()).toDays();
    }

    private List<DateTimeInterval> split(long days) {
        List<DateTimeInterval> result = new ArrayList<>();
        addFirstDay(result);
        addMiddleDays(result, days);
        addLastDay(result);
        return result;
    }

    private void addFirstDay(List<DateTimeInterval> result) {
        result.add(DateTimeInterval.toMidnight(from));
    }

    private void addMiddleDays(List<DateTimeInterval> result, long days) {
        for(int loop=1; loop < days; loop++) {
            result.add(DateTimeInterval.during(from.toLocalDate().plusDays(loop)));
        }
    }

    private void addLastDay(List<DateTimeInterval> result) {
        result.add(DateTimeInterval.fromMidnight(to));
    }

    public String toString() {
        return "[ " + from + " - " + to + " ]";
    }
}

```
> 기간을 편하게 관리할 수 있는 DateTimeInterval 클래스를 추가하자.

```Java
public class Call {
	private DateTimeInterval interval;

	public Call(LocalDateTime from, LocalDateTime to) {
		this.interval = DateTimeInterval.of(from, to);
	}

	public Duration getDuration() {
		return interval.duration();
	}

	public LocalDateTime getFrom() {
		return interval.getFrom();
	}

	public LocalDateTime getTo() {
		return interval.getTo();
	}

	public DateTimeInterval getInterval() {
		return interval;
	}

	public List<DateTimeInterval> splitByDay() {
		return interval.splitByDay();
	}
}
```
> Call Class
##### 전체 통화 시간을 일자와 시간 기준으로 분할해서 계산해보자. 이를 위해 요금 계산 로직을 다음과 같이 두 개의 단계로 나눠 구현할 필요가 있다.
 - 통화 기간을 일자별로 분리한다.
 - 일자별로 분리된 기간을 다시 시간대별 규칙에 따라 분리한 후 각 기간에 대해 요금을 계산한다.
책임을 할당하는 기본 원칙은 책임을 수행하는 데 필요한 정보를 가장 잘 알고있는 정보 전문가에게 할당하는 것이다.  
여기서 기간을 처리하는 방법의 전문가는 DateTimeInterval  
시간대별 기준을 잘 알고 있는 요금 전문가는 TimeOfDayDiscountPolicy(구현예정)  

![KakaoTalk_Photo_2021-09-24-15-58-03](https://user-images.githubusercontent.com/60125719/134631750-93810438-73c9-4b68-a80e-92aa206d9b27.jpeg)
 
##### ex)
![KakaoTalk_Photo_2021-09-24-16-00-03 001](https://user-images.githubusercontent.com/60125719/134631979-511a8e2c-faf5-418c-95b3-31493a72485d.jpeg)
![KakaoTalk_Photo_2021-09-24-16-00-04 002](https://user-images.githubusercontent.com/60125719/134631996-ca265f80-ba68-46ef-8f7e-2fa7ae6df897.jpeg)

```Java
public class TimeOfDayDiscountPolicy extends BasicRatePolicy {
    private List<LocalTime> starts = new ArrayList<LocalTime>();
    private List<LocalTime> ends = new ArrayList<LocalTime>();
    private List<Duration> durations = new ArrayList<Duration>();
    private List<Money>  amounts = new ArrayList<Money>();

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.splitByDay()) {
            for(int loop=0; loop < starts.size(); loop++) {
                result.plus(amounts.get(loop).times(Duration.between(from(interval, starts.get(loop)),
                        to(interval, ends.get(loop))).getSeconds() / durations.get(loop).getSeconds()));
            }
        }
        return result;
    }

    private LocalTime from(DateTimeInterval interval, LocalTime from) {
        return interval.getFrom().toLocalTime().isBefore(from) ? from : interval.getFrom().toLocalTime();
    }

    private LocalTime to(DateTimeInterval interval, LocalTime to) {
        return interval.getTo().toLocalTime().isAfter(to) ? to : interval.getTo().toLocalTime();
    }
}
```

```Java
public class DateTimeInterval {
    private LocalDateTime from;
    private LocalDateTime to;

    public static DateTimeInterval of(LocalDateTime from, LocalDateTime to) {
        return new DateTimeInterval(from, to);
    }

    public static DateTimeInterval toMidnight(LocalDateTime from) {
        return new DateTimeInterval(from, LocalDateTime.of(from.toLocalDate(), LocalTime.of(23, 59, 59, 999_999_999)));
    }

    public static DateTimeInterval fromMidnight(LocalDateTime to) {
        return new DateTimeInterval(LocalDateTime.of(to.toLocalDate(), LocalTime.of(0, 0)), to);
    }

    public static DateTimeInterval during(LocalDate date) {
        return new DateTimeInterval(
                LocalDateTime.of(date, LocalTime.of(0, 0)),
                LocalDateTime.of(date, LocalTime.of(23, 59, 59, 999_999_999)));
    }

    private DateTimeInterval(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }

    public Duration duration() {
        return Duration.between(from, to);
    }

    public LocalDateTime getFrom() {
        return from;
    }

    public LocalDateTime getTo() {
        return to;
    }

    public List<DateTimeInterval> splitByDay() {
        if (days() > 0) {
            return split(days());
        }

        return Arrays.asList(this);
    }

    private long days() {
        return Duration.between(from.toLocalDate().atStartOfDay(), to.toLocalDate().atStartOfDay()).toDays();
    }

    private List<DateTimeInterval> split(long days) {
        List<DateTimeInterval> result = new ArrayList<>();
        addFirstDay(result);
        addMiddleDays(result, days);
        addLastDay(result);
        return result;
    }

    private void addFirstDay(List<DateTimeInterval> result) {
        result.add(DateTimeInterval.toMidnight(from));
    }

    private void addMiddleDays(List<DateTimeInterval> result, long days) {
        for(int loop=1; loop < days; loop++) {
            result.add(DateTimeInterval.during(from.toLocalDate().plusDays(loop)));
        }
    }

    private void addLastDay(List<DateTimeInterval> result) {
        result.add(DateTimeInterval.fromMidnight(to));
    }

    public String toString() {
        return "[ " + from + " - " + to + " ]";
    }
}

```

### 요일별 방식 구현하기

```Java
public class DayOfWeekDiscountRule {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();
    private Duration duration = Duration.ZERO;
    private Money amount = Money.ZERO;

    public DayOfWeekDiscountRule(List<DayOfWeek> dayOfWeeks,
                                 Duration duration, Money  amount) {
        this.dayOfWeeks = dayOfWeeks;
        this.duration = duration;
        this.amount = amount;
    }

    public Money calculate(DateTimeInterval interval) {
        if (dayOfWeeks.contains(interval.getFrom().getDayOfWeek())) {
            return amount.times(interval.duration().getSeconds() / duration.getSeconds());
        }

        return Money.ZERO;
    }
}
```
> DayOfWeekDiscountRule 클래스는 규칙을 정의하기 위해 필요한 요일의 목록, 단위시간, 단위요금을 인스턴스 변수로 포함한다. calculate메서드는 파라미터로 전달된 interval이 요일 조건을 만족시킬 경우 단위 시간과 단위 요금을 이용해 통화 요금을 계산한다.

```Java
public class DayOfWeekDiscountPolicy extends BasicRatePolicy {
    private List<DayOfWeekDiscountRule> rules = new ArrayList<>();

    public DayOfWeekDiscountPolicy(List<DayOfWeekDiscountRule> rules) {
        this.rules = rules;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.getInterval().splitByDay()) {
            for(DayOfWeekDiscountRule rule: rules) { result.plus(rule.calculate(interval));
            }
        }
        return result;
    }
}
```
> 요일별 방식 역시 통화 기간이 여러 날에 걸쳐 있을 수 있다는 사실을 기억하라

### 구간별 방식 구현하기
지금까지 구현했던 클래스들은 개념적으로는 연관돼 있지만 구현 방식에 있어서는 일관성이 없다.  
비일관성은 두 가지 상황에서 발목을 잡는다. 
1. 새로운 구현을 추가해야 하는 상황
2. 기존의 구현을 이해해야 하는 상황. 
-> 유사한 기능을 서로 다른 방식으로 구현해서는 안 된다.

## 02. 설계에 일관성 부여하기
 - 변하는 개념을 변하지 않는 개념으로부터 분리하라
 - 변하는 개념을 캡슐화하라

### 조건 로직 대 객체 탐색

```Java
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;
        for(DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
            	// 기간 조건인 경우
            } else {
            	// 회차 조건인 경우
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
                	// 금액 할인 정책인 경우
                    break;
                case PERCENT_DISCOUNT:
                	// 비율 할인 정책인 경우
                    break;
                case NONE_DISCOUNT:
                 	// 할인 정책이 없는 경우
                    break;
            }

            fee = movie.getFee().minus(discountAmount).times(audienceCount);
        } else {
        	// 할인 적용이 불가능한 경우
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```
> 이 설계가 나쁜 이유는 변경의 주기가 서로 다른 코드가 한 클래스 안에 뭉쳐있기 떄문! 또한 새로운 할인 정책이나 할인 조건을 추가하기 위해서는 기존 코드의 내부를 수정해야 하기 때문에 오류가 발생할 확률이 높다.

```Java
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DiscountPolicy(DiscountCondition ... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    public Money calculateDiscountAmount(Screening screening) {
        for(DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }

        return screening.getMovieFee();
    }

    abstract protected Money getDiscountAmount(Screening Screening);
}
```

```Java
public class Movie {
	...

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```
> 객체지향은 단순히 객체를 믿고 메시지를 전달해줄 뿐이다.

![KakaoTalk_Photo_2021-09-24-17-21-24 001](https://user-images.githubusercontent.com/60125719/134642405-e606a675-fa35-4d65-a493-f21614b06377.jpeg)
![KakaoTalk_Photo_2021-09-24-17-21-25 002](https://user-images.githubusercontent.com/60125719/134642442-ba85cf5f-799b-4a0e-9bc4-632f6abfb586.jpeg)
> Movie는 현재의 할인 정책이 어떤 종류이지 판단하지 않는다. 단지 DiscountPolicy로 향하는 참조를 통해 메시지를 전달할 뿐이다.  

클래스 분리는 어떤 기준으로?? -> SRP의 원칙을 따르도록  

##### 일관성 있는 협력을 위한 지침
 - 변하는 개념을 변하지 않는 개념으로부터 분리하라
 - 변하는 개념을 캡슐화하라

> 구성 요소를 캡슐화하는 실행 지침은 객체지향의 핵심 덕목 중 하나다. 시스템을 책임을 캡슐화한 섬들로 분리하고 그 섬들 간의 결합도를 제한하라.

### 캡슐화 다시 살펴보기
캡슐화는 데이터 은닉 이상의 의미를 가진다.  
캡술화란 변하는 어떤것이든 감추는 것. 데이터 뿐만 아니라 개념을 감추자.  

![KakaoTalk_Photo_2021-09-24-17-29-37](https://user-images.githubusercontent.com/60125719/134643485-08f29d80-33ed-4d27-9a31-102b9f67033e.jpeg)
![KakaoTalk_Photo_2021-09-24-17-29-41](https://user-images.githubusercontent.com/60125719/134643504-4899da07-65ab-4cf0-85d8-bb95fca5793a.jpeg)
> 캡슐화란 단지 데이터 은닉을 의미하는 것이 아니다. 코드 수정으로 인한 파급효과를 제어할 수 있는 모든 기법이 캡슐화의 일종이다. 

#### 변하는 부분을 분리해서 타입 계층을 만든다.
변하지 않는 부분으로부터 변하는 부분을 분리한다. 변하는 부분들의 공통적인 행동을 추상 클래스나 인터페이스로 추상화한 후 변하는 부분들이 이 추상 클래스나 인터페이스를 상속받게 만든다. 이제 변하는 부분은 변하지 않는 부분의 서브타입이 된다. 

#### 변하지 않는 부분의 일부로 타입 계층을 합성한다.
앞에서 구현한 타입 계층을 변하지 않는 부분에 합성한다. 변하지 않는 부분에서는 변경되는 구체적인 사항에 결합돼서는 안된다. 의존성 주입과 같이 결합도를 느슨하게 유지할 수 있는 방법을 이용해 오직 추상화에만 의존하게 만든다. 이제 변하지 않는 부분은 변하는 부분의 구체적인 종류에 대해서는 알지 못할 것이다. 변경이 캡슐화 된 것이다.















































