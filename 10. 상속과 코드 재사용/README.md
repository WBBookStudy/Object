# 상속과 코드 재사용

## 01. 상속과 중복 코드
중복된 코드는 의심과 불신을 뿌린다.

### DRY(Don't Repeat Yourself) 원칙
> 중복된 코드는 변경을 방해한다. 어떤 코드가 중복인지 찾아야하며, 그 코드들을 일관되게 수정해야한다.  
코드 안에 중복이 존재해서는 안 된다.

### 중복과 변경
#### 중복코드 살펴보기

```Java
public class Call {
    private LocalDateTime from;
    private LocalDateTime to;

    public Call(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }

    public Duration getDuration() {
        return Duration.between(from, to);
    }

    public LocalDateTime getFrom() {
        return from;
    }
}
```
```Java
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    public void call(Call call) {
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Money getAmount() {
        return amount;
    }

    public Duration getSeconds() {
        return seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result;
    }
}
```

```Java
Phone phone = new Phone(Money.wons(5), Duration.ofSeconds(10));
phone.call(new Call(LocalDateTime.of(2018,1,1,12,10,0)),
        Call(LocalDateTime.of(2018,1,1,12,11,0)));
phone.call(new Call(LocalDateTime.of(2018,1,2,12,10,0)),
        Call(LocalDateTime.of(2018,1,2,12,11,0)));
phone.calculateFee; // => Money.wons(60)
```

-> 요구사항 추가 ! 심야요금제를 추가해주세요
```Java
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            }
        }

        return result;
    }
}
```

> 복붙으로 구현 시간을 절약한 대가로 지불해야 하는 비용은 예상보다 크다.

#### 중복 코드 수정하기
이번엔 통화 요금에 부과할 세금을 계산하는 기능을 추가해보자

```Java
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    private double taxRate;

    public Phone(Money amount, Duration seconds, double taxRate) {
        this.amount = amount;
        this.seconds = seconds;
        this.taxRate = taxRate;
    }

    public void call(Call call) {
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Money getAmount() {
        return amount;
    }

    public Duration getSeconds() {
        return seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result.plus(result.times(taxRate));
    }
}
```

```Java 
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    private double taxRate;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
        this.taxRate = taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            }
        }

        return result.minus(result.times(taxRate));
    }
}
```

> 많은 코드 더미 속에서 어떤 코드가 중복인지를 파악하는 일은 쉬운 일이 아니다. 더욱이 더 큰 문제는 중복 코드를 서로 다르게 수정하기 쉽다는 점이다. 사실 예제에도 calculateFee 메서드에서 문제가 있다.

#### 타입 코드 사용하기

```Java
public class Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    enum PhoneType { REGULAR, NIGHTLY }

    private PhoneType type;

    private Money amount;
    private Money regularAmount;
    private Money nightlyAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this(PhoneType.REGULAR, amount, Money.ZERO, Money.ZERO, seconds);
    }

    public Phone(Money nightlyAmount, Money regularAmount,
                 Duration seconds) {
        this(PhoneType.NIGHTLY, Money.ZERO, nightlyAmount, regularAmount,
                seconds);
    }

    public Phone(PhoneType type, Money amount, Money nightlyAmount,
                 Money regularAmount, Duration seconds) {
        this.type = type;
        this.amount = amount;
        this.regularAmount = regularAmount;
        this.nightlyAmount = nightlyAmount;
        this.seconds = seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            if (type == PhoneType.REGULAR) {
                result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                    result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                } else {
                    result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                }
            }
        }

        return result;
    }
}
```
> 두 클래스를 하나로 합쳤지만 합친 클래스는 결합도가 너무 높아졌다.

### 상속을 이용해서 중복 코드 제거하기

```Java
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    public void call(Call call) {
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Money getAmount() {
        return amount;
    }

    public Duration getSeconds() {
        return seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result;
    }
}
```

```Java
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        super(regularAmount, seconds);
        this.nightlyAmount = nightlyAmount;
    }

    @Override
    public Money calculateFee() {
        // 부모클래스의 calculateFee 호출
        Money result = super.calculateFee();

        Money nightlyFee = Money.ZERO;
        for(Call call : getCalls()) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                nightlyFee = nightlyFee.plus(
                        getAmount().minus(nightlyAmount).times(
                                call.getDuration().getSeconds() / getSeconds().getSeconds()));
            }
        }

        return result.minus(nightlyFee);
    }
}
```
> 개발자의 가정을 이해하기 전에는 코드를 이해하기 어렵다. 상속을 염두해 두고 설계되지 않은 클래스를 상속을 이용해 재사용하는 것은 생각처럼 쉽지 않다. 상속은 결합도를 높힌다.( 부모 클래스의 이해도를 필요로 하기 때문 )

### 강하게 결합된 Phone과 NightlyDiscountPhone
세금을 부과하는 요구사항이 추가가 된다면?

```Java
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    private double taxRate;

    public Phone(Money amount, Duration seconds, double taxRate) {
        this.amount = amount;
        this.seconds = seconds;
        this.taxRate = taxRate;
    }

    public void call(Call call) {
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Money getAmount() {
        return amount;
    }

    public Duration getSeconds() {
        return seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result.plus(result.times(taxRate));
    }

    public double getTaxRate() {
        return taxRate;
    }
}
```

```Java
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        super(regularAmount, seconds, taxRate);
        this.nightlyAmount = nightlyAmount;
    }

    @Override
    public Money calculateFee() {
        // 부모클래스의 calculateFee() 호출
        Money result = super.calculateFee();

        Money nightlyFee = Money.ZERO;
        for(Call call : getCalls()) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                nightlyFee = nightlyFee.plus(
                        getAmount().minus(nightlyAmount).times(
                                call.getDuration().getSeconds() / getSeconds().getSeconds()));
            }
        }

        return result.minus(nightlyFee.plus(nightlyFee.times(getTaxRate())));
    }
}
```
> NightlyDiscountPhone 를 Phone의 자식클래스로 만든 이유는 중복된 코드를 제거하기 위함이었다. 하지만 세금을 부과하는 로직을 추가하기 위해 Phone을 수정할 때 유사한 코드를 NightlyDiscountPhone 에도 추가해야만 했다.

#### 상속을 위한 경고1
> 자식 클래스의 메서드 안에서 super 참조를 이용해 부모 클래스의 메서드를 직접 호출할 경우 두 클래스는 강하게 결합된다. super 호출을 제거할 수 있는 방법을 찾아 결합도를 제거하라.


## 02. 취약한 기반 클래스 문제
**취약한 기반 클래스 문제(Fragile Base Class Problem)** : 부모 클래스의 변경에 의해 자식 클래스가 영향을 받는 현상.  
취약한 기반 클래스의 문제는 캡슐화를 약화시키고 결합도를 높힌다. 객체지향의 기반은 캡슐화를 통한 변경의 통제란걸 기억하자.  

### 불필요한 인터페이스 상속 문제
![KakaoTalk_Photo_2021-08-13-13-12-27](https://user-images.githubusercontent.com/60125719/129303900-3f420ded-81b9-4d69-91a2-1333252030e5.jpeg)
> 자바의 초기버전에서 상속을 잘못한 대표적인 사례.
Stack 이 Vector를 상속받기 때문에 Stack의 퍼블릭 인터페이스에 Vector의 퍼블릭 인터페이스가 합쳐졌다.

```Java
Stack<String> stack = new Stack<>();
stack.push("1")
stack.push("2")
stack.push("3")

stack.add(0, "4");

assertEquals("4", stack.pop()); // fail
```
> 덕분에 위의 테스트가 실패하며, 실제 코드상에서 예상치 못한 버그가 생길 수 있다.

![KakaoTalk_Photo_2021-08-13-13-16-25](https://user-images.githubusercontent.com/60125719/129304178-626e5c57-c6a1-4627-9294-bdd577b8f297.jpeg)
> 제네릭이 도입되기 이전에 만들어졌기 떄문에 컴파일러가 키와 값의 타입이 String인지 여부를 체크 할 수 있는 방법이 없다.
```Java
Properties properties = new Properties()
properties.setProperty("Bjarne Stroustrup", "C++");
properties.setProperty("James Gosling", "Java");

properties.put("Dennis Ritchie", 67);

assertEquals("C", properties.getProperty("Dennis Ritchie")); // fail

```
> 메서드가 반환할 값의 타입이 String이 아닐 경우 null을 반환하도록 구현돼어 있기 때문

#### 상속을 위한 경고2
> 상속받은 부모 클래스의 메서드가 자식 클래스의 내부 구조에 대한 규칙을 꺠트릴 수 있다.

### 메서드 오버라이딩의 오작용 문제

```Java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

```

```Java
InstrumentedHashSet<String> languages = new InstrumentedHashSet<>();
languages.addAll(Arrays.asList("Java", "Ruby", "Scala"));
```
> addCount의 갯수가 6이 되버린다. 이유는? 부모 클래스인 HashSet의 addAll 메서드 안에서 add 메서드를 실행시키기 떄문!  
> 이 문제를 해결할 수 있는 방법은 InstrumentedHashSet의 addAll 메서드를 제거하는것.


```Java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    public int getAddCount() {
        return addCount;
    }
}
```
> 해결은 한듯 하지만 오버라이딩 된 addAll 메서드의 구현이 HashSet과 동일해져 버렸다. -> 코드가 중복됨

#### 상속을 위한 경고3
> 자식 클래스가 부모 클래스의 메서드를 오버라이딩 할 경우 부모 클래스가 자신의 메서드를 사용하는 방법에 자식 클래스가 결합될 수 있다.

#### 설계는 트레이드오프 활동이다. 상속은 코드 재사용을 위해 캡슐화를 희생한다. 완벽한 캡슐화를 원한다면 코드 재사용을 포기하거나 상속 이외의 다른 방법을 사용해야 한다.

### 부모 클래스와 자식 클래스의 동시 수정 문제

```Java
public class Song {
    private String singer;
    private String title;

    public Song(String singer, String title) {
        this.singer = singer;
        this.title = title;
    }

    public String getSinger() {
        return singer;
    }

    public String getTitle() {
        return title;
    }
}
```

```Java
public class Playlist {
    private List<Song> tracks = new ArrayList<>();

    public void append(Song song) {
        getTracks().add(song);
    }

    public List<Song> getTracks() {
        return tracks;
    }
}
```

```Java
public class PersonalPlaylist extends Playlist {
    public void remove(Song song) {
        getTracks().remove(song);
    }
}

```

**요구사항 변경**: Playlist에서 노래의 목록뿐만 아니라 가수별 노래의 제목도 함께 관리해야한다.

```Java
public class Playlist {
    private List<Song> tracks = new ArrayList<>();
    private Map<String, String> singers = new HashMap<>();

    public void append(Song song) {
        tracks.add(song);
        singers.put(song.getSinger(), song.getTitle());
    }

    public List<Song> getTracks() {
        return tracks;
    }

    public Map<String, String> getSingers() {
        return singers;
    }
}
```
> 안타깝게도 위 수정 내용이 정상적으로 동작하려면 PersonalPlaylist의 remove 메서드도 함께 수정해야한다.

```Java
public class PersonalPlaylist extends Playlist {
    public void remove(Song song) {
        getTracks().remove(song);
        getSingers().remove(song.getSinger());
    }
}

```
> 자식 클래스가 부모 클래스의 메서드를 오버라이딩하거나 불필요한 인터페이스를 상속받지 않았음에도 부모 클래스를 수정할 때 자식 클래스를 함께 수정해야 할 수도 있다.

#### 상속을 위한 경고4
> 클래스를 상속하면 결합도로 인해 자식 클래스와 부모 클래스의 구현을 영원히 변경하지 않거나, 자식 클래스와 부모 클래스를 동시에 변경하거나 둘 중 하나를 선택할 수밖에 없다.





















































