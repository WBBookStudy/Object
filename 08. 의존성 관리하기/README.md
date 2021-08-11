# 의존성 관리하기

## 01. 의존성 이해하기
### 변경과 의존성
어떤 객체가 협력하기 위해 다른 객체를 필요로 할 때 두 객체 사이에 의존성이 존재하게 된다. 의존성은 실행 시점과 구현 시점에 서로 다른 의미를 가진다.
 - 실행 시점: 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재해야 한다.
 - 구현 시점: 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경된다.

```Java
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

```Java
Movie avatar = new Movie("아바타",
 Duration.ofMinutes(120),
 Money.wons(10000),
 new AmountDiscountPolicy(...));
``` 
> 생성자를 통해 해결한다.

```Java
public class Movie {
	public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
		...
		this.discountPolicy = discountPolicy;
	}
}
```
> 메서드를 통해 해결한다. 이 방법은 객체를 생성한 이후에도 의존하고 있는 대상을 변경할 수 있도록 만들고 싶을때 유용하다.

```Java
public class Movie {
	public void setDiscountPolicy (DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy;
	}
}

Movie avatar = new Movie(...);
avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
```
> setter 메서드를 제공하여 해결한다. 이 방법은 실행 시점에 의존 대상을 변경할 수 있기 떄문에 설계를 좀 더 유연하게 만들 수는 있지만, 객체가 생성된 후에 협력에 필요한 의존 대상을 설정하기 때문에 객체를 생성하고 의존대상을 설정하기 전까지는 객체의 상태가 불완전할 수 있다.


```Java
Movie avatar = new Movie(..., newPercentDiscountPolicy(...));
...
avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
```
> 더 좋은 방법은 생성자 방법과 setter방법을 합성하는 것이다.

## 02. 유연한 설계
### 의존성과 결합도

```Java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private PercentDiscountPolicy percentDiscountPolicy;

    public Movie(String title, Duration runningTime, Money fee, PercentDiscountPolicy percentDiscountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.percentDiscountPolicy = percentDiscountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(percentDiscountPolicy.calculateDiscountAmount(screening));
    }
}
```
> 이 코드는 비율할인 정책을 적용하기 위해 Movie가 PercentDiscountPolicy에 의존하고 있다는 사실을 코드를 통해 드러낸다.
위의 코드는 PercentDiscountPolicy라는 구체적인 클래스에 의존하게 만들었기 때문에 다른 할인정책이 필요한 문맥에서 Movie클래스를 재사용 할 수 없게 만들어졌다.  
그렇다면 Movie클래스에서 PercentDiscountPolicy가 아닌 DiscountPolcy를 가지고 있다면? 재사용 할 수 있게 됨  
바람직한 의존성은 **재사용성**과 관련이 있다. -> 결합도  
어떤 두 요소 사이에 존재하는 의존성이 바람직할 때 두 요소가 **느슨합 결합도(loose coupling)** 또는 **약한 결합도(weak coupling)** 를 가진다고 말한다. 반대의 경우는? 단단한 결합도(tight coupling) 또는 강한 결합도(strong coupling)  

### 지식이 결합을 낳는다.
결합도의 정도는 한 요소가 자신이 의존하고 있는 다른 요소에 대해 알고 있는 정보의 양으로 결정된다.  
즉, 한 요소가 다른 요소에 대해 더 많은 정보를 알고 있을수록 두 요소는 강하게 결합된다.  
더 많이 알고 있다는 것은 더 적은 컨텍스트에서 재사용 가능하다는것을 의미한다.  

### 추상화에 의존하라
> 추상화란 어떤 양상, 세부사항, 구조를 좀 더 명확히 이해하기 위해 특정 절차는 물체를 의도적으로 생략하고 감춤으로써 복잡도를 극복하는 방법
#### 추상화와 결합도의 관점에서 의존 대상을 구분할 수 있다. (아래로 내려갈수록 클라이언트가 알아야 하는 지식의 양이 적어지기 때문에 결합도가 느슨해진다.)
 - 구체 클래스 의존성(concrete class dependency)
 - 추상 클래스 의존성(abstract class dependency)
 - 인터페이스 의존성(interface dependency)

> 추상클래스는 메서드의 내부 구현과 자식 클래스의 종류에 대한 지식을 숨길 수 있다. 하지만 추상클래스의 클라이언트는 여전히 협력하는 대상이 속한 클래스 상속 계층이 무엇인지에 대해서는 알고 있어야 한다. 인터페이스에 의존하면 상속 계층을 모르더라도 협력이 가능해진다. 

### 명시적인 의존성

```Java
public class Movie {
	...
	private DiscountPolicy discountPolicy;

	public Movie(String title, Duration, runningTime, Money fee) {
		...
		this.discountPolicy = new AmountDiscountPolicy(...);
	}
}
```
> 이 코드는 생성자에서 콘크리트 클래스인 AmountDiscountPolicy를 직접 사용하고 있기때문에 결합도가 높아졌다.
![KakaoTalk_20210729_164025202](https://user-images.githubusercontent.com/60125719/127451814-1470325b-0a0f-48b6-b843-2a3f377e05ed.jpg)

이 예제에서 알 수 있듯이 결합도를 느슨하게 만들기 위해서는 인스턴스 변수의 타입을 추상 클래스나 인터페이스로 선언하는 것만으로는 부족하다. 클래스 안에서 구체 클래스에 대한 모든 의존성을 제거해야한다. 앞에서 언급했던 해결방법 3가지를 생각해보자.

```Java
public class Movie {
	...
	private DiscountPolicy discountPolicy;

	public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
		...
		this.discountPolicy = discountPolicy;
	}
}
```
>  생성자의 인자가 추상 클래스 타입으로 선언됐기 때문에 이제 객체를 생성할 떄 생성자의 인자로 골라 넣을 수 있다!

#### 의존성의 대상을 생성자의 인자로 전달받는 방법과 생성자 안에서 직접 생성하는 방법의 가장 큰 차이는 퍼블릭 인터페이스를 통해 설정할 수 있게 하느냐이다. 이를 **명시적인 의존성(explicit dependency)**이라 한다. 반대로 내부에서 직접 생성하는 방법을 **숨거진 의존성(hidden dependency)** 라 한다.

### new는 해롭다
왜?
 - new 연산자를 사용하기 위해서는 구체 클래스의 이름을 직접 기술해야 한다. 따라서 new를 사용하는 클라이언트는 추상화가 아닌 구체 클래스에 의존할 수 밖에 없기 때문에 결합도가 높아진다.
 - new 연산자는 생성하려는 구체 클래스뿐만 아니라 어떤 인자를 이용해 클래스의 생성자를 호출해야하는지도 알아야 한다. 따라서 new를 사용하면 클라이언트가 알아야 하는 지식의 양이 늘어나기 때문에 결합도가 높아진다.

 ```Java
 public class Movie {
 	...
 	private DiscountPolicy discountPolicy;

 	public Movie(Strint title, Duration runningTime, Money fee) {
 		...
 		this.discountPolicy = new AmountDiscountPolicy(Money.wons(800),
 		new SequenceCondition(1),
 		new SequenceCondition(10),
 		new PeriodCondition(DayOfWeek.MONDAY,
 			LocalTime.of(10,0), LocalTime.of(11,59)),
 		new PeriodCondition(DayOfWeek.THURSDAY,
 			LocalTime.of(10,0), LocalTime.of(20,59)),
 		)
 	}
 }
 
 ```
 > Movie 클래스가 AmountDiscountPolicy의 인스턴스를 생성하기 위해서는 생성자에 전달되는 인자를 알고 있어야한다. SequenceCondition과 PeriodCondition도 마찬가지... 결합도를 높힌다.
 ![KakaoTalk_20210729_165422467](https://user-images.githubusercontent.com/60125719/127453602-651d8c40-382a-434c-88c8-46a5583fb189.jpg)

```Java
public class Movie {
	...
	private DiscountPolicy discountPolicy;

	public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
		...
		this.discountPolicy = discountPolicy;
	}
}

Movie avatar = new Movie("아바타",
	Duration.ofMinutes(120),
	Money.wons(10000),
	new AmountDiscountPolicy(Money.wons(800),
		new SequenceCondition(1),
		new SequenceCondition(10),
 		new PeriodCondition(DayOfWeek.MONDAY,
 			LocalTime.of(10,0), LocalTime.of(11,59)),
 		new PeriodCondition(DayOfWeek.THURSDAY,
 			LocalTime.of(10,0), LocalTime.of(20,59))
	)
);
```
> 생성자를 통해 외부의 인스턴스를 전달받아 의존성을 해결하자


### 가끔은 생성해도 무방하다

```Java
public class Movie {
	...
	private DiscountPolicy discountPolicy;
	
	public Movie(String title, Duration runningTime, Money fee) {
		this(title, runningTime, fee, new AmountDiscountPolicy(...));
	}

	public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
		...
		this.discountPolicy = discountPolicy;
	}
}
```
> 추가된 생성자 안에서 AmountDiscountPolicy 인스턴스를 생성하고 있다. 생성자에서 생성자를 호출함...
-> 간단하게 호출이 가능해졌다.

```Java
public class Movie {
	public Money calculateMovieFee(Screening screening) {
		return calculateMovieFee(screening, new AmountDiscountPolicy(...));
	}

	public Money calculateMovieFee(Screening screening, DiscountPolicy discountPolicy) {
		return fee.minus(discountPolicy.calculateDiscountAmount(screening));
	}
}
```
> 트레이드오프의 대상은 결합도와 사용성...

### 표준 클래스에 대한 의존은 해롭지 않다
의존성이 불편한 이유는? 항상 변경에 대한 영향을 암시하기 때문에 따라서 변경될 확률이 거의 없는 클래스라면 의존성이 문제가 되지 않는다. ex) JDK  
```Java
public abstract class DiscountPolicy {
	private List<DiscountCondition> conditions = new Array<>();

	public void switchConditions(List<DiscountCondition> conditions) {
		this.conditions = conditions;
	}
}
```

### 컨텍스트 확장하기

#### case1) 할인 혜택을 제공하지 않는 영화의 예매 요금을 계산하는 경우

```Java
public class Movie {
	public Movie(String title, Duration runningTime, Money fee) {
		this(title, runningTime, fee, null);
	}

	public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
		...
		this.discountPolicy = discountPolicy;
	}

	public Money calculateMoveFee(Screening screening) {
		if (discountPolicy == null) {
			return fee;
		}

		return fee.minus(discountPolicy.calculateDiscountAmount(screening));
	}
}

```
> 이 코드의 문제점은 DiscountPolicy 사이의 협업 방식에서 예외케이스(null인 경우)가 추가된것. 버그가 생길 수 있는 가능성이다.

```Java

public class NoneDiscountPolicy extends DiscountPolicy {
	@override
	protected Money getDiscountAmount(Screening screening) {
		return Money.ZERO;
	}
}

Movie avatar = new Movie("아바타",
	Duration.ofMinutes(120),
	Money.wons(10000),
	new NoneDiscountPolicy());
	)
```
> 아예 클래스를 추가해주자.

#### case2) 중복 적용이 가능한 할인 정책을 구현하자

```Java
public class OverlappedDiscountPolicy extends DiscountPolicy {
	private List<DiscountPolicy> discountPolicies = new ArrayList<>();

	public OverlappedDiscountPolicy(DiscountPolicy ... discountPolicies) {
		this.discountPolicies = Arrays.asList(discountPolicies);
	}

	@Override
	protected Money getDiscountAmount(Screening screeing) {
		Money result = Money.ZERO;
		for(DiscountPolicy each : discountPolicies) {
			result = result.plus(each.calculateDiscountAmount(screening));
		}
		return result;
	}
}

Movie avatar = new Movie("아바타",
	Duration.ofMinutes(120),
	Money.wons(10000),
	new OverlappedDiscountPolicy(
		new AmountDiscountPolicy(...),
		new PercentDiscountPolicy(...)
		)
	);

```
> Movie를 수정하지 않고도 중복된 할인조건을 추가했다 !

### 조합 가능한 행동
어떤 객체와 협력하느냐에 따라 객체의 행동이 달라지는 것은 유연하고 재사용 가능한 설계가 가진 특징  
유연하고 재사용 가능한 설계는 객체가 어떻게(how) 하는지를 장황하게 나열하지 않고도 객체들의 조합을 통해 무엇(what)을 하는지 표현하는 클래스들로 구성된다.  

```Java
Movie avatar = new Movie("아바타",
	Duration.ofMinutes(120),
	Money.wons(10000),
	newAmountDiscountPolicy(Money.wons(800),
		new SequenceCondition(1),
		new SequenceCondition(10),
 		new PeriodCondition(DayOfWeek.MONDAY,
 			LocalTime.of(10,0), LocalTime.of(11,59)),
 		new PeriodCondition(DayOfWeek.THURSDAY,
 			LocalTime.of(10,0), LocalTime.of(20,59))
	)
);
```
> 코드를 읽는 것만으로도 첫 번째 상영, 10번째 상영, 월요일 10시부터 12시 사이 상영, 목요일 10시부터 21시 상영의 경우에는 800원을 할인해준다는 사실을 알 수 있다.





































