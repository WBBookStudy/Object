# 객체지향 프로그래밍
> 진정한 객체지햐 패러다임으로의 전환은 클래스가 아닌 객체에 초점을 맞출 때에만 얻을 수 있다.
 - 어떤 클래스가 필요한지를 고민하기 전에 어떤 객체들이 필요한지 고민하라.
 - 객체를 독립적인 존재가 아니라 기능을 구현하기 위해 협력하는 공동체의 일원으로 봐야한다.

## 도메인
 문제를 해결하기 위해 사용자가 프로그램을 사용하는 분야

## 예제 UML
<img width="754" alt="스크린샷 2021-06-04 오후 8 17 28" src="https://user-images.githubusercontent.com/60125719/120793629-15f12180-c572-11eb-97b2-fb4d45e31be6.png">

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
![IMG_4974](https://user-images.githubusercontent.com/60125719/120797479-0f18dd80-c577-11eb-85a2-daafc57c0351.jpg)




























