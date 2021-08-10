# 유연한 설계

## 01. 개방- 폐쇄 원칙
> 소프트웨어 개체(클래스, 모듈, 함수 등등)는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.

- 확장에 대해 열려 있다: 애플리케이션 요구사항이 변경될 때 이 변경에 맞게 새로운 '동작'을 추가해서 애플리케이션의 기능을 확장할 수 있다.
- 수정에 대해 닫혀 있다: 기존의 '코드'를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다.

### 컴파일 타임 의존성을 고정시키고 런타임 의존성을 변경하라

![KakaoTalk_Photo_2021-08-10-13-36-33](https://user-images.githubusercontent.com/60125719/128808947-0d778fd2-ec51-4ca0-b129-e843c3cf680b.jpeg)
> 기존 영화 할인정책 클래스의 경우로 볼때는 이미 개방-폐쇄 원칙을 따르고 있다. 새로운 할인정책이 생기면 클래스를 추가할 수 있으며, 중복할인 정책을 추가하더라도 기존의 코드는 수정이 일어나지 않는다.
![KakaoTalk_Photo_2021-08-10-13-40-11](https://user-images.githubusercontent.com/60125719/128809200-5a833e85-78cf-48b3-bd7b-29e7b7803c29.jpeg)

### 추상화가 핵심이다
> 개방 - 폐쇄 원칙의 핵심은 **추상화에 의존하는것** 이다.

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
> 변하지 않는 부분을 고정하고 변하는 부분을 생략하는 추상화 메커니즘을 사용하여 개방-폐쇄의 원칙을 따르자

```Java
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
> DiscountPolicy에 대해서만 의존한다. 의존성은 변경의 영향을 의미하고 DiscountPolicy는 변하지 않는 추상화이다. DiscountPolicy의 자식 클래스를 추가하더라도 이 클래스는 수정이 될 일이 없기 때문에 수정에 대해 닫혀있다.

## 02. 생성 사용 분리

결함도가 높아질수록 개방-폐쇄 원칙을 따르는 구조를 설계하기가 어려워진다.

```Java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = new AmountDiscountPolicy(...);
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```
> AmountDiscountPolicy 구체 클래스가 생성자에 들어가있다. 동작을 추가하거나 변경하기 위해 기존의 코드를 수정하도록 되어있는 구조. 개방 폐쇄 원칙을 위반한다. 또한 이 클래스는 객체를 생성하고 사용하는것까지의 동작을 클래스 내부에서 처리하고있다.

![KakaoTalk_Photo_2021-08-10-13-53-01](https://user-images.githubusercontent.com/60125719/128810162-5d64459c-1e2a-4f75-a90c-6dedee10c307.jpeg)

#### 유연하고 재사용 가능한 설계를 원한다면 객체와 관련된 두 가지 책임을 서로 다른 객체로 분리해야 한다. 하나는 객체를 생성하는것이고, 하나는 객체를 사용하는 것이다.
사용으로부터 생성을 분리하는 데 사용되는 가장 보편적인 방법은 객체를 생성할 책임을 클라이언트로 옮기는 것이다.

```Java
public class Client {
    public Money getAvatarFee() {
        Movie avatar = new Movie("아바타",
                Duration.ofMinutes(120),
                Money.wons(10000),
                new AmountDiscountPolicy(
                    Money.wons(800),
                    new SequenceCondition(1),
                    new SequenceCondition(10)));
        return avatar.getFee();
    }
}
```

![KakaoTalk_Photo_2021-08-10-13-56-49](https://user-images.githubusercontent.com/60125719/128810455-6bfce858-fc84-4c04-8fe7-870aed71ca92.jpeg)

### FACTORY 추가하기
위의 코드 역시 Movie의 인스턴스를 생성하는 동시에 getFee 메시지도 함께 전송하고 있다. Client 역시 생성과 사용의 책임을 함께 지니고 있는것.  
이를 해결하기 위해 아까와 동일한 방법으로 해결하려 Movie클래스 생성을 밖으로 꺼내려고 하면 Movie클래스를 밖에서 알아야한다. 하지만 그렇게 하고싶지 않다면?  

```Java
public class Factory {
    public Movie createAvatarMovie() {
        return new Movie("아바타",
                Duration.ofMinutes(120),
                Money.wons(10000),
                new AmountDiscountPolicy(
                    Money.wons(800),
                    new SequenceCondition(1),
                    new SequenceCondition(10)));
    }
}
```

```Java
public class Client {
    private Factory factory;

    public Client(Factory factory) {
        this.factory = factory;
    }

    public Money getAvatarFee() {
        Movie avatar = factory.createAvatarMovie();
        return avatar.getFee();
    }
}
```

이와 같이 객체 생성과 관련된 책임만 전담하는 별도의 객체를 추가하자. 
![KakaoTalk_Photo_2021-08-10-14-03-16](https://user-images.githubusercontent.com/60125719/128810923-fef9030a-aa63-45bc-ac5f-2eac53689263.jpeg)

### 순수한 가공물에게 책임 할당하기

앞의 FACTORY는 도메인 모델에 속하지 않는다. 단순 기술적인 이유로 해당 구조로 개발된것.  
시스템을 객체로 분해하는 두가지 방법 **표현적 분해(representational decomposition)**, **행위적 분해(behavioral decomposition)**
 - 표면적 분해: 도메인에 존재하는 사물 또는 개념을 표현하는 객체들을 이용해 시스템을 분해하는것
 - 행위적 분해: 도메인 개념을 표현한 객체가 아닌 설계자가 편의를 위해 임의로 만들어낸 가공의 객체에게 책임을 할당해서 문제를 해결. 도메인과 무관한 인공적인 객체 **Pure Fabrication** 생성
 


