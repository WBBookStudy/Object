# 합성과 유연한 설계

코드를 재사용하기 위해서는 객체 합성이 클래스 상속보다 더 좋은 방법이다.  
상속 관계는 클래스 사이의 정적인 관계인 데 비해 합성 관계는 객체 사이의 동직인 관계이다. -> 코드 작성 시점에서 결정한 상속 관게는 변경이 불가능하지만 합성 관계는 실행시점에서 동적으로 변경 할 수 있다.  

## 01. 상속을 합성으로 변경하기
#### 코드 재사용을 위해 상속을 남용했을 때 직면할 수 있는 문제
 - 불필요한 인터페이스 상속 문제
 - 메서드 오버라이딩의 오작용 문제
 - 부모 클래스와 자식 클래스의 동시 수정 문제

합성을 사용하면 상속이 초래하는 세 가지 문제점을 해결 할 수 있다.

### 불필요한 인터페이스 상속문제: java.util.Properties와 java.util.Stack

```Java
public class Properties {
    private Hashtable<String, String> properties = new Hashtable <>();

    public String setProperty(String key, String value) {
        return properties.put(key, value);
    }

    public String getProperty(String key) {
        return properties.get(key);
    }
}
```
> Properties클래스에서 상속 관계를 제거하고 Hashtable을 Peroperties의 인스턴스 변수로 포함시키면 합성 관계로 변경할 수 있다.
-> 이제 더 이상 불필요한 Hashtable의 오퍼레이션들이 Properties 클래스의 퍼블릭 인터페이스를 오염시키지 않는다.

```Java
public class Stack<E> {
    private Vector<E> elements = new Vector<>();

    public E push(E item) {
        elements.addElement(item);
        return item;
    }

    public E pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }
        return elements.remove(elements.size() - 1);
    }
}
```
> Stack 클래스의 변경

### 메서드 오버라이딩의 오작용 문제: InstrumentedHashSet

```Java
public class InstrumentedHashSet<E> {
    private int addCount = 0;
    private Set<E> set;

    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }

    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```
> HashSet 인스턴스를 내부에 포함한 후 HashSet의 퍼블릭 인터페이스에서 제공하는 오퍼에이션들을 이용해 필요한 기능을 구현하면 된다.

#### 위의 경우에는 이전과 같이 불필요한 오퍼레이션들이 퍼블릭 인터페이스에 스며드는것을 방지하기 위함이었지만 이번 경우에는 HashSet이 제공하는 퍼블릭 인터페이스를 그대로 제공해야 한다.

```Java
public class InstrumentedHashSet<E> implements Set<E> {
    private int addCount = 0;
    private Set<E> set;

    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    @Override public boolean remove(Object o) {
        return set.remove(o);
    }

    @Override public void clear() {
        set.clear();
    }

    @Override public boolean equals(Object o) {
        return set.equals(o);
    }

    @Override public int hashCode() {
        return set.hashCode();
    }

    @Override public Spliterator<E> spliterator() {
        return set.spliterator();
    }

    @Override public int size() {
        return set.size();
    }

    @Override public boolean isEmpty() {
        return set.isEmpty();
    }

    @Override public boolean contains(Object o) {
        return set.contains(o);
    }

    @Override public Iterator<E> iterator() {
        return set.iterator();
    }

    @Override public Object[] toArray() {
        return set.toArray();
    }

    @Override public <T> T[] toArray(T[] a) {
        return set.toArray(a);
    }

    @Override public boolean containsAll(Collection<?> c) {
        return set.containsAll(c);
    }

    @Override public boolean retainAll(Collection<?> c) {
        return set.retainAll(c);
    }

    @Override public boolean removeAll(Collection<?> c) {
        return set.removeAll(c);
    }
}
```
> 이것이 포워딩!

### 부모 클래스와 자식 클래스의 동시 수정 문제: PersonalPlaylist

```Java
public class PersonalPlaylist {
    private Playlist playlist = new Playlist();

    public void append(Song song) {
        playlist.append(song);
    }

    public void remove(Song song) {
        playlist.getTracks().remove(song);
        playlist.getSingers().remove(song.getSinger());
    }
}
```
> 안타깝게도 playlist의 경우에는 합성으로 변경하더라도 가수별 노래 목록을 유지하기 위해 Playlist와 PersonalPlaylist를 함께 수정해야 하는 문제가 해결되지는 않는다.
-> 그렇다 하더라도 여전히 상속보다 합성을 사용하는것이 좋다. 향후에 Playlist의 내부 구현을 변경하더라도 파급효과를 최대한 personalPlaylist 내부로 캡슐화할 수 있기 때문!

## 02. 상속으로 인한 조합의 폭발적인 증가
상속으로 인해 결합도가 높아지면 코드를 수정하는 데 필요한 작업의 양이 과도하게 늘어나는 경향이 있다.
- 하나의 기능을 추가하거나 수정하기 위해 불필요하게 많은 수의 클래스를 추가하거나 수정해야 한다.
- 단일 상속만 지원하는 언어에서는 상속으로 인해 오히려 중복 코드의 양이 늘어날 수 있다.
-> 합성을 사용하면 상속으로 인해 발생하는 클래스의 증가와 중복 코드 문제를 간단하게 해결할 수 있다.

### 기본 정책과 부가 정책 조합하기
핸드폰 과금 시스템에 새로운 요구사항을 추가해보자.
![KakaoTalk_Photo_2021-08-29-14-42-21](https://user-images.githubusercontent.com/60125719/131239981-67116dc3-5e62-49d8-88ce-b374a93fb1a7.jpeg)

부가 정책은 다음과 같은 특성을 가진다
 - 기본정책의 계산 결과에 적용된다.
 - 선택적으로 적용할 수 있다.
 - 조합 기능하다.
 - 부가 정책은 임의의 순서로 적용 가능하다.

![KakaoTalk_Photo_2021-08-29-14-45-24](https://user-images.githubusercontent.com/60125719/131240060-886cda2a-f439-498c-ae8b-290d5c174f43.jpeg)

### 상속을 이용해서 기본 정책 구현하기

```Java
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }

        return result;
    }

    abstract protected Money calculateCallFee(Call call);
}

public class RegularPhone extends Phone {
    private Money amount;
    private Duration seconds;

    public RegularPhone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
}
```
> 기존의 클래스들

### 기본 정책에 세금 정책 조합하기
일반 요금제에 세금 정책을 조합해야한다면? 가장 간단한 방법은 RegularPhone클래스를 상속받은 TaxableRegularPhone 클래스를 추가하는것.

```Java
public class TaxableRegularPhone extends RegularPhone {
    private double taxRate;

    public TaxableRegularPhone(Money amount, Duration seconds,
                               double taxRate) {
        super(amount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    public Money calculateFee() {
        Money fee = super.calculateFee();
        return fee.plus(fee.times(taxRate));
    }
}
```
> 부모 클래스의 메서드를 재사용하기 위해 super 호출을 사용하면 원하는 결과를 쉽게 얻을 수는 있지만 자식 클래스와 부모 클래스 사이의 결합도가 높아진다. 결합도를 낮추는 방법은 자식 클래스가 부모 클래스의 메서드를 호출하지 않도록 부모 클래스에 추상 메서드를 제공하는 것이다.

```Java
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }

        return afterCalculated(result);
    }

    protected abstract Money calculateCallFee(Call call);
    protected abstract Money afterCalculated(Money fee);
}
```
> 먼저 Phone 클래스에 새로운 추상 메서드인 afterCalculated를 추가하자.

```Java
public class RegularPhone extends Phone {
    private Money amount;
    private Duration seconds;

    public RegularPhone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee;
    }
}
```
> 일반 요금제를 구현하는 RegularPhone은 요금을 수정할 필요가 없기 때문에 afterCalculated 메서드에서 파라미터로 전달된 요금을 그대로 반환하자

```Java
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee;
    }
}


```
> NightlyDiscountPhone

#### 위 코드에서 알 수 있듯 부모클래스에 추상 메서드를 추가하면 모든 자식 클래스들이 추상 메서드를 오버라이드 해야한다. 모든 추상 메서드의 구현이 동일하다는 사실에도 주목하자.

#### 중복을 피해기 위해서는?
```Java
public abstract class Phone {
    ...
    protected Money afterCalculated(Money fee) {
        return fee;
    }

    protected abstract Money calculateCallFee(Call call);
}
```
> 이와 같이 훅 메서드(hook method)를 구현해 주어야한다.

```Java
public class TaxableRegularPhone extends RegularPhone {
    private double taxRate;

    public TaxableRegularPhone(Money amount, Duration seconds,
                               double taxRate) {
        super(amount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    public Money calculateFee() {
        Money fee = super.calculateFee();
        return fee.plus(fee.times(taxRate));
    }
}
```
> TaxableRegularPhone은 RegularPhone이 계산한 요금에 세금을 부과한다.

```Java
public class TaxableNightlyDiscountPhone extends NightlyDiscountPhone {
    private double taxRate;

    public TaxableNightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        super(nightlyAmount, regularAmount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
```
> 심야요금제에 세금을 부과하자

![KakaoTalk_Photo_2021-08-29-20-50-40](https://user-images.githubusercontent.com/60125719/131249371-1ba2f767-dbda-46e7-8002-9cfec2db159c.jpeg)
> 문제는 TaxableNightlyDiscountPhone과 TaxableRegularPhone 사이에 코드를 중복했다는것!

### 기본 정책에 기본 요금 할인 정책 조합하기
> 기본 요금 할인 정책이란? 매달 청구되는 요금에서 고정된 요금을 차감하는 정책. 예를들어, 매달 1000원을 할인해주는 요금제가 있다면 이 요금제에는 부가 정책으로 기본 요금 할인 정책이 조합되어있다고 보면 됨.

```Java
public class RateDiscountableRegularPhone extends RegularPhone {
    private Money discountAmount;

    public RateDiscountableRegularPhone(Money amount, Duration seconds, Money discountAmount) {
        super(amount, seconds);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```
> 일반 요금제와 기본 요금 할인 정책을 조합하고 싶다면 RegularPhone을 상속받는 RateDiscountableRegularPhone 을 추가하자

```Java
public class RateDiscountableNightlyDiscountPhone extends NightlyDiscountPhone {
    private Money discountAmount;

    public RateDiscountableNightlyDiscountPhone(Money nightlyAmount,
                                                Money regularAmount, Duration seconds, Money discountAmount) {
        super(nightlyAmount, regularAmount, seconds);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```
> 심야 할인 요금제와 기본 요금 할인 정책을 조합하고 싶다면 NightlyDiscountPhone을 상속받는 RateDiscountableNightlyDiscountPhone 클래스를 추가하자

![KakaoTalk_Photo_2021-08-29-20-56-41](https://user-images.githubusercontent.com/60125719/131249554-6d6cec58-5199-43c0-a1c1-d096de8ae638.jpeg)
> 이번에도 부가 정책을 구현한 RateDiscountableRegularPhone와 RateDiscountableNightlyDiscountPhone의 코드가 중복되버렸다.

### 중복 코드의 덫에 걸리다.
부가 정책은 자유롭게 조합할 수 있어야 하고 적용되는 순서 역시 임의로 결정할 수 있어야 한다.  
요구사항에 따르면 세금 정책과 기본 요금 할인 정책을 함께 적용하는 것도 가능해야 하고, 세금 정책을 적용한 후에 기본 요금 할인 정책을 적용하거나 기본 요금 할인정책을 적용한 후에 세금 정책을 적용하는 것도 가능해야 한다.  
상속을 이용한 해결방법은 모든 가능한 조합별로 자식 클래스를 하나씩 추가하는 것이다.  

```Java
public class TaxableAndRateDiscountableRegularPhone extends TaxableRegularPhone {
    private Money discountAmount;

    public TaxableAndRateDiscountableRegularPhone(Money amount, Duration seconds, double taxRate, Money discountAmount) {
        super(amount, seconds, taxRate);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return super.afterCalculated(fee).minus(discountAmount);
    }
}
```
> 세금이 부과된 요금을 계산한 후 기본 요금 할인정책을 적용하고싶을 때

```Java
public class RateDiscountableAndTaxableRegularPhone
        extends RateDiscountableRegularPhone {
    private double taxRate;

    public RateDiscountableAndTaxableRegularPhone(Money amount, Duration seconds, Money discountAmount, double taxRate) {
        super(amount, seconds, discountAmount);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return super.afterCalculated(fee).plus(fee.times(taxRate));
    }
}
```
> 표준 요금제에 기본 요금 할인 정책을 먼저 적용한 후 세금을 나중에 부과하고싶음

```Java
public class TaxableAndDiscountableNightlyDiscountPhone extends TaxableNightlyDiscountPhone {
    private Money discountAmount;

    public TaxableAndDiscountableNightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds,
          double taxRate, Money discountAmount) {
        super(nightlyAmount, regularAmount, seconds, taxRate);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return super.afterCalculated(fee).minus(discountAmount);
    }
}
```
> 심야 할인 요금제의 계산 결과에 세금 정책을 적용한 후 기본 요금 할인 정책을 적용하는 케이스

```Java
public class RateDiscountableAndTaxableNightlyDiscountPhone
        extends RateDiscountableNightlyDiscountPhone {
    private double taxRate;

    public RateDiscountableAndTaxableNightlyDiscountPhone(Money nightlyAmount, Money regularAmount,
            Duration seconds, Money discountAmount, double taxRate) {
        super(nightlyAmount, regularAmount, seconds, discountAmount);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return super.afterCalculated(fee).plus(fee.times(taxRate));
    }
}
```
> 심야 할인 요금제의 계산 결과에 기본 요금 할인 정책을 적용한 후 세금 정책을 적용

![KakaoTalk_Photo_2021-08-29-21-04-33](https://user-images.githubusercontent.com/60125719/131249730-fc7f78a3-80a8-4e29-857a-1d264340643b.jpeg)
> 개 복잡

#### 이제 새로운 기본 정책을 추가해야한다면????? 예를들어 고정요금제
![KakaoTalk_Photo_2021-08-29-21-06-14](https://user-images.githubusercontent.com/60125719/131249786-2a398019-2787-4b9d-9cce-3ab33a792a58.jpeg)
> ....

#### 이처럼 상속의 남용으로 하나의 기능을 추가하기 위해 필요 이상으로 많은 수의 클래스를 추가해야하는 경우를 가리켜 **클래스 폭발(class explosion)** 문제라고 함.
이 문제를 해결할 수 있는 최선의 방법은 상속을 포기하는 것.


## 03. 합성 관계로 변경하기
상속 관계는 컴파일타임에 결정되고 고정되기 때문에 코드를 실행하는 도중에는 변경할 수 없다. 따라서 여러 기능을 조합해야 하는 설계에 상속을 이용하면 모든 조합 가능한 경우별로 클래스를 추가해야한다.  
합성은 컴파일타임 관계를 런타임 관계로 변경함으로써 이 문제를 해결한다.  

### 기본 정책 합성하기
가장 먼저 해야할 일은 각 정책을 별도의 클래스로 구현하는것.

```Java
public interface RatePolicy {
    Money calculateFee(Phone phone);
}

```
> 기본 정책과 부가 정책을 포괄하는 RatePolicy 인터페이스. 

```Java
public abstract class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;

        for(Call call : phone.getCalls()) {
            result.plus(calculateCallFee(call));
        }

        return result;
    }

    protected abstract Money calculateCallFee(Call call);
}
```
> 기본 정책을 구성하는 일반 요금제와 심야 할인 요금제는 개별 요금을 계산하는 방식을 제외한 전체 처리 로직이 거의 동일하다.  이 중복 코드를 담을 추상클래스 BasicRatePolicy를 추가하자.

```Java
public class RegularPolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;

    public RegularPolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```
> 일반 요금제

```Java
public class NightlyDiscountPolicy extends BasicRatePolicy {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;

    public NightlyDiscountPolicy(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }

        return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```
> 심야 요금제

```Java
public class Phone {
    private RatePolicy ratePolicy;
    private List<Call> calls = new ArrayList<>();

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }

    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }

    public Money calculateFee() {
        return ratePolicy.calculateFee(this);
    }
}
```
> 이제 기본 정책을 이용해 요금을 계산할 수 있도록 Phone을 수정하자.

#### Phone 내부에 RatePolicy에 대한 참조자가 포함돼 있다는 것에 주목하자. 이것이 바로 합성이다.

![KakaoTalk_Photo_2021-08-29-21-23-37](https://user-images.githubusercontent.com/60125719/131250220-8cf3c187-e611-4e95-92ad-b764eeccc2e2.jpeg)

#### 일반 요금제의 규칙에 따라 통화요금을 계산하고 싶다면?
```Java
Phone phone = new Phone(new RegularPolicy(Money.wons(10), Duration.ofSeconds(10)));
```

#### 심야 할인 요금제의 규칙에 따라 통화 요금을 계산하고 싶다면?
```Java
Phone phone = new Phone(new NightlyDiscountPolicy(Money.wons(5), Money.wons(10), Duration.ofSeconds(10)));
```

### 부가 정책 적용하기

![KakaoTalk_Photo_2021-08-29-21-29-34](https://user-images.githubusercontent.com/60125719/131250365-9fe84415-1301-4367-bde4-e2e565de3703.jpeg)

부가 정책을 추가해보자. 세금 정책을 추가해야한다면 RegularPolicy의 계산이 끝나고 Phone에게 반환되기 전에 적용돼야 한다.

![KakaoTalk_Photo_2021-08-29-21-31-01](https://user-images.githubusercontent.com/60125719/131250400-4c02e5bd-560d-4437-83e0-cb225a266acb.jpeg)

만약 일반 요금제에 기본 요금 할인 정책을 적용한 후에 세금 정책을 적용해야한다면?

![KakaoTalk_Photo_2021-08-29-21-31-52](https://user-images.githubusercontent.com/60125719/131250427-0be017d6-ecbd-4f65-b857-316ac6672c2c.jpeg)

- 부가 정책은 기본 정책이나 다른 부가 정책의 인스턴스를 참조할 수 있어야 한다. 다시 말해서 부가 정책의 인스턴스는 어떤 종류의 정책과도 합성될 수 있어야 한다.
- Phone의 입장에서는 자신이 기본 정책의 인스턴스에게 메시지를 전송하고 있는지, 부가 정책의 인스턴스에게 메시지를 전송하고 있는지를 몰라야 한다. 다시 말해서 기본 정책과 부가 정책은 협력 안에서 동일한 '역할'을 수행해야 한다. 이것은 부가 정책이 동일한 RatePolicy 인터페이스를 구현해야 한다는 것을 의미한다.

```Java
public abstract class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;

    public AdditionalRatePolicy(RatePolicy next) {
        this.next = next;
    }

    @Override
    public Money calculateFee(Phone phone) {
        Money fee = next.calculateFee(phone);
        return afterCalculated(fee) ;
    }

    abstract protected Money afterCalculated(Money fee);
}
```
> 부가 정책을 AdditionalRatePolicy 추상클래스로 구현하자
#### 다른 요금 정책과 조합될 수 있도록 RatePolicy 타입의 next라는 이름을 가진 인스턴스 변수를 내부에 포함한다.

```Java
public class TaxablePolicy extends AdditionalRatePolicy {
    private double taxRatio;

    public TaxablePolicy(double taxRatio, RatePolicy next) {
        super(next);
        this.taxRatio = taxRatio;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRatio));
    }
}

```
> 세금 정책을 구현하자

```Java
public class RateDiscountablePolicy extends AdditionalRatePolicy {
    private Money discountAmount;

    public RateDiscountablePolicy(Money discountAmount, RatePolicy next) {
        super(next);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```
> 기본 요금 할인정책도 구현하자

![KakaoTalk_Photo_2021-08-29-21-38-59](https://user-images.githubusercontent.com/60125719/131250621-05cc550c-eba2-49a2-95ec-90904531405c.jpeg)

### 기본 정책과 부가 정책 합성하기

```Java
Phone phone = new Phone(new TaxablePolicy(0.05, new RegularPolicy(...)));

```
> 일반 요금제에 세금 정책을 조합할 경우

```Java
Phone phone = new Phone(new TaxablePolicy(0.05, new RateDiscountablePolicy(Money.wons(1000), new RegularPolicy(...))));
```
> 일반 요금제에 기본 요금 할인 정책을 조합한 결과에 세금 정책을 조합하고 싶을 때

```Java
Phone phone = new Phone(new RateDiscountablePolicy(Money.wons(1000), new TaxablePolicy(0.05, new RegularPolicy(...))));
```
> 위의 예제에서 세금 정책과 기본 요금 할인 정책이 적용되는 순서를 바꾸고 싶다면?? 이렇게 !

```Java
Phone phone = new Phone(new RateDiscountablePolicy(Money.wons(1000), new TaxablePolicy(0.05, new NightlyDiscountPolicy(...))));
```
> 심야 할인 요금제에도 하고싶다면?

### 새로운 정책 추가하기
아까와 같이 새로운 정책을 추가해보자

![KakaoTalk_Photo_2021-08-29-21-44-45](https://user-images.githubusercontent.com/60125719/131250772-ba425823-2d39-4adb-a874-92727abbca5e.jpeg)
> 한개의 클래스 추가만으로 해결!

#### 약정할인 정책을 추가하고싶다?!
![KakaoTalk_Photo_2021-08-29-21-44-50](https://user-images.githubusercontent.com/60125719/131250791-83688c7d-a9d0-420e-9df9-c265dd5f3782.jpeg)
> 👍

### 객체 합성이 클래스 상속보다 더 좋은 방법이다.
객체 합성이 코드를 재사용하면서도 건전한 결합도를 유지할 수 있는 더 좋은 방법이다.

















































