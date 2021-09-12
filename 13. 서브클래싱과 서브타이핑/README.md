# 서브클래싱과 서브타이핑

### 상속의 두가지 용도
 - 타입 계층을 구현하는 것: 타입 계층의 관점에서 부모 클래스는 자식 클래스의 일반화(generalization)이고 자식 클래스는 부모 클래스의 특수화(specialization)다.
 - 코드 재사용: 재사용을 위해 상속을 사용할 경우 부모 클래스와 자식 클래스가 강하게 결합되기 때문에 변경하기 어려운 코드를 얻게 될 확률이 높다.
-> 상속을 사용하는 일차적인 목표는 코드 재사용이 아니라 타입 계층을 구현하는 것이어야 한다.  


## 01. 타입
### 개념 관점의 타입
우리가 인지하는 세상의 사물의 종류. 타입은 심볼, 내연, 외연의 세 가지 요소로 구성된다.  
(설명은 별로 중요하지 않을듯??)  
-> 공통의 특징을 공유하는 대상들의 분류이다.

### 프로그래밍 언어 관점의 타입
비트 묶음에 의미를 부여하기 위해 정의된 제약과 규칙. 프로그래밍 언어에서 타입은 두 가지 목적을 위해 사용된다.

- 타입에 수행될 수 있는 유효한 오퍼레이션의 집합을 정의한다. (+ 연산자 등)
- 타입에 수행되는 오퍼레이션에 대해 미리 약속된 문맥을 제공한다. (+ 연산자가 int끼리만 더해지고 String X int는 안된다는 규칙등.. 이런것들을 결정함)
-> 동일한 오퍼레이션을 적용할 수 있는 인스턴스들의 집합

### 객체지향 패러다임 관점의 타입
객체의 퍼블릭 인터페이스가 객체의 타입을 결정한다. 동일한 퍼블릭 인터페이스를 제공하는 객체들은 동일한 타입으로 분류된다.


## 02. 타입 계층
### 타입 사이의 포함관계
타입 계층을 구성하는 두 타입 간의 관계에서 더 일반적인 타입을 **슈퍼타입(supertype)** 이라고 부르고 더 특수한 타입을 **서브타입(subtype)** 이라고 부른다.
![KakaoTalk_Photo_2021-09-12-20-51-40](https://user-images.githubusercontent.com/60125719/132986691-0f038852-9675-465a-bb2f-12c9b68a1db0.jpeg)

위에 나온 일반화와 특수화의 정의

- 일반화: 다른 타입을 완전히 포함하거나 내포하는 타입을 식별하는 행위 또는 그 행위의 결과
- 특수화: 다른 타입 안에 전체적으로 포함되거나 완전히 내포되는 타입을 식별하는 행위 또는 그 행위의 결과

### 슈퍼타입과 서브타입
#### 슈퍼타입은 다음과 같은 특징을 가지는 타입을 가리킨다.
- 집합이 다른 집합의 모든 맴버를 포함한다.
- 타입 정의가 다른 타입보다 좀 더 일반적이다.
#### 서브타입은 다음과 같은 특징을 가지는 타입을 가리킨다.
- 집합에 포함되는 인스턴스들이 더 큰 집합에 포함된다.
- 타입 정의가 다른 타입보다 좀 더 구체적이다.

### 객체지향 프로그래밍과 타입 계층
퍼블릭 인터페이스의 관점에서 슈퍼타입과 서브타입을 다음과 같이 정의할 수 있다.
 - 슈퍼타입: 서브타입이 정의한 퍼블릭 인터페이스를 일반화시켜 상대적으로 범용적이고 넓은 의미로 정의한 것이다.
 - 서브타입: 슈퍼타입이 정의한 퍼블릭 인터페이스를 특수화시켜 상대적으로 구체적이고 좁은 의미로 정의한 것이다.

서브타입의 인스턴스는 슈퍼타입의 인스턴스로 간주될 수 있다.

## 03. 서브클래싱과 서브타이핑
타입 계층을 구현할 때 지켜야 하는 제약사항을 클래스와 상속의 관점에서 살펴보자.

### 언제 상속을 사용해야 하는가?
#### 마틴 오더스키는 다음과 같은 질문을 해보고 두 질문에 모두 '예'라고 답할 수 있는 경우에만 상속을 사용하라고 조언한다.
- 상속 관계가 is-a 관계를 모델링하는가?: 일반적으로 자식클래스는 부모클래스다 라고 말해도 이상하지 않다면 상속을 사용할 후보로 간주할 수 있다.
- 클라이언트 입장에서 부모 클래스의 타입으로 자식 클래스를 사용해도 무방한가?: 상속 계층을 사용하는 클라이언트의 입장에서 부모 클래스와 자식 클래스의 차이점을 몰라야한다. 이를 자식 클래스와 부모클래스 사이의 **행동 호환성**이라고 부른다.

### is-a 관계
**어떤 타입 S가 다른 타입T의 일종이라면 당연히 "타입 S는 타입T다(S is-a T)"라고 말할 수 있어야한다.**

ex)
```
1. 펭귄은 새다.
2. 새는 날 수 있다.
```
```Swift
class Bird {
	func fly() {
		...
	}
	...
}
```
```Swift
class Penguin: Bird {
	...
}
```
> 펭귄은 분명 새지만 날 수 없는 새다. 이 예시는 어휘적인 정의가 아니라 기대되는 행동에 따라 타입 계층을 구성해야 한다는 사실을 보여준다.

### 행동 호환성
펭귄이 새가 아니라는 사실을 받아들이기 위한 출발점은 타입이 행동과 관련이 있다는 사실에 주목하는 것이다.  
행동의 호환 여부를 판단하는 기준은 **클라이언트의 관점** 이다. 클라이언트가 두 타입이 동일하게 행동할 것이라고 기대한다면 두 타입을 타입 계층으로 묶을 수 있다.  

다음과 같이 클라이언트가 날 수 있는 새만을 원한다고 가정해보자
```Swift
func flyBird(Bird bird) {
	// 인자로 전달된 모든 bird는 날 수 있어야한다.
	bird.fly()
}
```
> 펭귄을 넣는다면? 하지만 펭귄은 날 수 없다.즉 이 둘은 상속 관계로 연결한 위 설계는 수정돼야 한다.

#### 위의 문제 해결방법
##### 방법1. Penguin의 fly 메서드를 오버라이딩해서 내부 구현을 비워두기
```Swift
class Penguin: Bird {
	...
	override func fly() { }
}
```
> 이제 펭귄은 못난다. 하지만 이 방법은 어떤 행동도 수행하지 않기 떄문에 모든 bird가 날 수 있다는 클라이언트의 기대를 만족시키지 못한다. 따라서 올바른 설계라 할 수 없다.

##### 방법2. Penguin의 fly메서드를 오버라이딩한 후 예외를 던지자
```Swift
class Penguin: Bird {
	override func fly() {
		throw MyError~
	}
}
```
> 이제 펭귄은 못난다. 하지만 이 경우에는 flyBird메서드에 전달되는 인자의 타입에 따라 메서드가 실패하거나 성공하게된다. 모든 bird가 날 수 있다는 클라이언트의 기대를 만족시키지 못한다. 따라서 올바른 설계라 할 수 없다.

##### 방법3. flyBird 메서드를 수정해서 인자로 전달된 bird의 타입이 Penguin이 아닐 경우에면 fly메시지를 전송하도록 하는방법.
```Swift
func flyBird(Bird bird) {
	// 인자로 전달된 모든 bird가 Penguin의 인스턴스가 아닐 경우에만 fly() 메시지를 전송한다.
	if (!(bird instenceof Penguin)) {
		bird.fly()
	}
}
```
> 만약 펭귄이 아니라 타조가 추가된다면? 위의 코드를 수정해야한다. 즉 결합도가 올라간다. 일반적으로 instanceof 처럼 객체의 타입을 확인하는 코드는 새로운 타입을 추가할 때마다 코드 수정을 요구하기 때문에 개방-폐쇄 원칙을 위반한다.

### 클라이언트의 기대에 따라 계층 분리하기
위의 문제를 해결할 수 있는 방법은 클라이언트의 기대에 맞게 상속 계층을 분리하는 것 뿐이다.

```Swift 
class Bird {
	...
}

class FlyingBird: Bird {
	func fly() { ... }
}

class Penguin: Bird {
	...
}
```
> 이제 flyBird메서드는 FlyingBird 타입을 이용해 날 수 있는 새만 인자로 전달돼야 한다는 사실을 코드에 명시할 수 있다.
![KakaoTalk_Photo_2021-09-12-21-43-29](https://user-images.githubusercontent.com/60125719/132988023-b8bc364e-3c5b-4de7-8692-d4465f7b96e1.jpeg)

#### 클라이언트에 따라 인터페이스를 분리하여 해결 할 수도 있다.
![KakaoTalk_Photo_2021-09-12-21-46-31](https://user-images.githubusercontent.com/60125719/132988106-6e78a788-58b5-4d74-97b3-0a028e235548.jpeg)
> 만약 펭귄이 Bird의 코드를 재사용해야 한다면? 펭귄의 퍼블릭 인터페이스에 fly오퍼레이션이 추가되기 때문에 이 방법을 사용할 수는 없다. 게다가 재사용을 위한 상속은 위험하다.


#### 합성을 이용해 해결하기
![KakaoTalk_Photo_2021-09-12-21-50-24](https://user-images.githubusercontent.com/60125719/132988216-1c6511e5-b010-4fb2-a06c-1ca1e1c6335d.jpeg)
> 클라이언트에 따라 인터페이스를 분리하면 변경에 대한 영향을 더 세밀하게 제어할 수 있게 된다. 만일 그림에서 Client1의 기대가 바뀌어서 Flyer의 인터페이스가 변경돼야 한다고 가정했을때 Flyer에 의존하고 있는 Bird가 영향을 받게 된다. 하지만 변경의 영향은 Bird에서 끝나고 Client2는 영향을 받지 않는다.  
-> 인터페이스 분리 원칙(Interface Segregation Principle, ISP)

### 서브클래싱과 서브타이핑
상속을 사용하는 두 가지 목적. 서브클래싱과 서브타이핑  
- 서브클래싱: 다른 클래스의 코드를 재사용할 목적으로 상속을 사용하는 경우를 가리킨다. 자식 클래스와 부모 클래스의 행동이 호환되지 않기 때문에 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 없다.
- 서브타이핑: 타입 계층을 구성하기 위해 상속을 사용하는 경우를 가리킨다. 영화 예메 시스템에서 구현한 DiscountPolicy 상속 계층이 서브타이핑에 해당한다. 서브타이핑에서는 자식 클래스와 부모 클래스의 행동이 호환되기 때문에 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 있다. 이때 부모 클래스는 자식 클래스의 슈퍼타입이 되고 자식 클래스는 부모 클래스의 서브타입이 된다.

-> 자식 클래스가 부모 클래스의 코드를 재사용할 목적으로 상속을 사용했다면 그것은 서브 클래싱. 부모 클래스의 인스턴스 대신 자식 클래스의 인스턴스를 사용할 목적으로 상속을 사용했다면 그것은 서브타이핑

##### 서브 타이핑 관계가 유지되기 위해서는 서브타입이 슈퍼타입이 하는 모든 행동을 동일하게 할 수 있어야 한다. (behavioral subsitution)


## 04. 리스코프 치환 원칙

바바라 리스코프에 의하면 상속 관계로 연결한 두 클래스가 서브타이핑 관계를 만족시키기 위해서는 다음의 조건을 만족시켜야한다.

> S형의 각 객체 o1에 대해 T형의 객체 o2가 하나 있고, T에 의해 정의된 모든 프로그램 P에서 T가 S로 치환될 때, P의 동작이 변하지 않으면 S는 T의 서브타입이다.
-> 서브타입은 그것의 기반 타입에 대해 대체 가능해야 한다.  
-> 10장에서 본 Stack과 Vector는 리스코프 치환 원칙을 위반하는 예시. Penguin과 Bird도 마찬가지. 

```Java
public class Rectangle {
	private int x, y, width, height;

	public Rectangle(int x, int y, int width, int height) {
		this.x = x;
		this.y = y;
		this.width = width;
		this.height = height;
	}

	public int getWidth() {
		return width;
	}

	public void setWidth(int width) {
		this.width = width;
	}

	public int getHeight() {
		return height;
	}

	public void setHeight(int height) {
		this.height = height;
	}

	public int getArea() {
		return width * height;
	}
}

```

```Java
public class Square extends Rectangle {
	public Square(int x, int y, int size) {
		super(x, y, size, size);
	}

	@Override
	public void setWidth(int width) {
		super.setWidth(width);
		super.setHeight(height);
	}

	@Override
	public void setHeight(int height) {
		super.setWidth(width);
		super.setHeight(height);
	}
}
```
> Square는 Rectangle의 자식 클래스이기 때문에 Rectangle이 사용되는 모든 곳에서 Rectangle로 업캐스팅 될 수 있다.

```Java
public void resize(Rectangle rectangle, int width, int height) {
	rectangle.setWidth(width);
	rectangle.setHeight(height);
	assert rectangle.getWidth() == width && rectangle.getHeight() == height;
}
```
> 높이와 너비를 다르게 준다.

```Java
Square square = new Square(10, 10, 10);
resize(square, 50, 100);
```
> 메서드 실행이 실패하게 된다.

#### resize 메서드의 관점에서 Rectangle 대신 Square를 사용할 수 없기 때문에 Square는 Rectangle이 아니다. 단순 코드를 재사용 할뿐.. 즉 두 클래스는 리스코프 치환 원칙을 위반하기 때문에 서브타이핑 관계가 아니라 서브클래싱 관계이다.

### 클라이언트와 대체 가능성
Square가 Rectangle을 대체할 수 없는 이유는 클라이언트의 관점에서 Square와 Rectangle이 다르기 때문이다.  
Rectangle을 사용하는 클라이언트는 Rectangle의 너비와 높이가 다를 수 있다는 가정하에 코드를 개발한다. 반면 Square는 너비와 높이가 항상 같다.  
리스코프 치환 원칙은 "클라이언트와 격리한 채로 본 모델은 의미 있게 검증하는 것이 불가능하다"는 아주 중요한 결론을 이끈다.  

### is-a 관계 다시 살펴보기
is-a 관계로 표현된 문장을 볼 때 마다 문장 앞에 "클라이언트 입장에서"라는 말이 빠져있다고 생각하자.  
-> (클라이언트 입장에서) 정사각형은 직사각형이다. (클라이언트 입장에서) 펭귄은 새다.  
클라이언트를 배제한 is-a관계는 혼란을 일으킨다.  
  
is-a 관계는 객체지향에서 중요한 것은 객체의 속성이 아니라 객체의 행동이라는 점을 강조한다.  

### 라스코프 치환 원칙은 유연한 설계의 기반이다
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

```
> 의존성 역전 원칙과 개방-폐쇄 원칙, 리스코프 치환 원칙이 한데 어우러져 설계를 확장 가능하게 만든 예시

- 의존성 역전 원칙: 구체 클래스인 Movie와 OverlappedDiscountPolicy 모두 추상 클래스인 DiscountPolicy에 의존한다. 상위 수준의 모듈인 Movie와 하위 수준의 모듈인 OverlappedDiscountPolicy는 모두 추상 클래스인 DiscountPolicy에 의존한다. 따라서 이 설계는 DIP를 만족한다.
- 리스코프 치환 원칙: DiscountPolicy와 협력하는 Movie의 관점에서 DiscountPolicy 대신 OverlappedDiscountPolicy와 협력하더라도 아무 문제가 없다. 다시 말해서 OverlappedDiscountPolicy는 클라이언트에 대한 영향 없이도 DiscountPolicy를 대체할 수 있다. 따라서 이 설계는 LSP를 만족한다
- 개방-폐쇄 원칙: 중복 할인 정책이라는 새로운 기능을 추가하기 위해 DiscountPolicy의 자식 클래스인 Overlapped DiscountPolicy를 추가하더라도 Movie에는 영향을 끼치지 않는다. 다시 말해서 기능 확장을 하면서 기존 코드를 수정할 필요는 없다. 따라서 이 설계는 OCP를 만족한다.














































