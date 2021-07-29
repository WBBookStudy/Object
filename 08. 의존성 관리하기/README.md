# 의존성 관리하기

## 01. 의존성 이해하기
### 변경과 의존성
어떤 객체가 협력하기 위해 다른 객체를 필요로 할 때 두 객체 사이에 의존성이 존재하게 된다. 의존성은 실행 시점과 구현 시점에 서로 다른 의미를 가진다.
 - 실행 시점: 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재해야 한다.
 - 구현 시점: 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경된다.

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
                startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0&&
                endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
    }
}
```
> PeriodCondition 클래스의 isSatisfiedBy 메서드는 Screening 인스턴스에게 getStartTime 메시지를 전송한다.
 - 실행 시점에 PeriodCondition의 인스턴스가 정상적으로 동작하려면 Screening 인스턴스가 존재해야한다.
 - PeriodCondition의 인스턴스가 정상적으로 동작하려면 getStartTime 메시지를 이해할 수 있어야한다.
> 이처럼 어떤 객체가 예정된 작업을 정상적으로 수행하기 위해 다른 객체를 필요로 하는 경우 두 객체 사이에 의존성이 존재한다고 말한다. 의존성은 방향성을 가지며 단방향이다.
![KakaoTalk_20210729_151921185](https://user-images.githubusercontent.com/60125719/127441727-b7fe9370-beab-4c93-b5cd-663724bb7ecf.jpg)
두 요소 사이의 의존성은 의존되는 요소가 변경될 때 의존하는 요소도 함께 변경될 수 있다는것을 의미한다.  

> DayOfWeek의 클래스 이름을 변경한다고 한다면?
-> PeriodCondition 클래스에 정의된 인스턴스 변수의 타입 선언도 함께 수정해주어야 한다. 이런 점은 LocalTime과 Screening의 이름이 바뀌어도 동일함.  

![KakaoTalk_20210729_152817982](https://user-images.githubusercontent.com/60125719/127442646-5e4bd9c5-9f4f-4b6b-90fd-bdb1dbc1ed26.jpg)


### 의존성 전의 (transitive dependency)
> 의존성 전의? ex) PeriodCondition이 Screening에 의존할 경우 PeriodCondition은 Screening이 의존하는 대상에 대해서도 자동적으로 의존하게 된다는 것
![KakaoTalk_20210729_153126072](https://user-images.githubusercontent.com/60125719/127443099-c3ea7511-6d8a-4641-86aa-6e5564266b86.jpg)
- 이런이유로 의존성을 **직접 의존성** 과 **간접 의존성**으로 나눌 수 있다.
- 의존성은 객체 뿐 아니라 모듈이나 더 큰 규모의 시스템일 수 도 있다.

### 런타임 의존성과 컴파일타임 의존성
런타임 의존성과 컴파일타임 의존성은 다를 수 있다.  
ex)
#### A) 컴파일 타임
![KakaoTalk_20210729_153724632](https://user-images.githubusercontent.com/60125719/127443670-0bba3a16-2121-423b-a7a3-685bfdb42a2d.jpg)
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
> Movie 클래스에서 AmountDiscountPolicy 클래스와 PercentDiscountPolicy 클래스로 향하는 어떤 의존성도 존재하지 않는다.

#### B) 런타임

![KakaoTalk_20210729_154027060](https://user-images.githubusercontent.com/60125719/127444101-579b23ad-b792-4793-bf1c-8377748198e0.jpg)
> 실제로는 런타임시 해당 의존성을 가진다.

#### 어떤 클래스의 인스턴스가 다양한 클래스의 인스턴트와 협력하기 위해서는 협력할 인스턴스의 구체적인 클래스를 알아서는 안된다. 실제로 협력할 객체가 어떤 것인지는 런타임에서 해결해야한다.


### 컨텍스트 독립성
클래스가 특정한 문맥에 강하게 결합될수록 다른 문맥에서 사용하기는 더 어려워진다.(ex PercentDiscountPolicy) 클래스가 사용될 특정한 문맥에 대해 최소한의 가정만 이뤄져 있다면 다른 문맥에서 재사용하기가 더 수월해진다. (ex DiscountPolicy) 이를 **컨텍스트 독립성** 이라고 부른다.

### 의존성 해결하기
컴파일타임 의존성을 실행 컨텍스트에 맞는 적절한 런타임 의존성으로 교체하는것을 **의존성 해결** 이라고 부른다. 의존성을 해결하기 위해서는 일반적으로 다음 세 가지 방법을 사용한다.
 1. 객체를 생성하는 시점에 생성자를 통해 의존성 해결
 2. 객체 생성 후 setter 메서드를 통해 의존성 해결
 3. 메서드 실행 시 인자를 이용해 의존성 해결

```
Movie avatar = new Movie("아바타",
 Duration.ofMinutes(120),
 Money.wons(10000),
 new AmountDiscountPolicy(...));
``` 
> 생성자를 통해 해결한다.

```
public class Movie {
	public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
		...
		this.discountPolicy = discountPolicy;
	}
}
```
> 메서드를 통해 해결한다. 이 방법은 객체를 생성한 이후에도 의존하고 있는 대상을 변경할 수 있도록 만들고 싶을때 유용하다.

```
public class Movie {
	public void setDiscountPolicy (DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy;
	}
}

Movie avatar = new Movie(...);
avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
```
> setter 메서드를 제공하여 해결한다. 이 방법은 실행 시점에 의존 대상을 변경할 수 있기 떄문에 설계를 좀 더 유연하게 만들 수는 있지만, 객체가 생성된 후에 협력에 필요한 의존 대상을 설정하기 때문에 객체를 생성하고 의존대상을 설정하기 전까지는 객체의 상태가 불완전할 수 있다.


```
Movie avatar = new Movie(..., newPercentDiscountPolicy(...));
...
avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
```
> 더 좋은 방법은 생성자 방법과 setter방법을 합성하는 것이다.











































