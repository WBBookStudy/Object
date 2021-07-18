# 메시지와 인터페이스
> 훌륭한 객체지향 코드를 얻기 위해서는 클래스가 아니라 객체를 지향해야 한다.

## 협력과 메시지
### 클라이언트-서버 모델
메시지는 객체 사이의 협력을 가능하게 하는 매개체다. 두 객체 사이의 협력 관계를 설명하기 위해 사용하는 전통적인 메타포는 **클라이언트-서버 모델**이다.  
협력 안에서 메시지를 전송하는 객체를 클라이언트, 메시지를 수신하는 객체를 서버라고 부른다.
![KakaoTalk_Photo_2021-07-16-11-20-56](https://user-images.githubusercontent.com/60125719/125881830-58aae45e-d53d-4d75-afd8-bae68044d24f.jpeg)

![KakaoTalk_Photo_2021-07-16-11-23-00](https://user-images.githubusercontent.com/60125719/125881990-66066f07-f6cf-45ef-986c-8b1efeee220b.jpeg)

> 객체는 협력에 참여하는 동안 클라이언트와 서버의 역할을 동시에 수행하는 것이 일반적이다.
> 요점은 객체가 독립적으로 수행할 수 있는 것보다 더 큰 책임을 수행하기 위해서는 다른 객체와 협력해야 한다는 것

### 메시지와 메시지 전송
 - **메시지(message)**는 객체들이 협력하기 위해 사용할 수 있는 유일한 의사소통
 - 한 객체가 다른 객체에게 도움을 요청하는것을 **메시지 전송(message sending)** 또는 **메시지 패싱(message passing)**
 - 메시지를 전송하는 객체: **메시지 전송자(message sender)**
 - 메시지를 수신하는 객체: **메시지 수신자(message receiver)**
 - 메시지는 **오퍼레이션명(operation name)**과 **인자(argument)**로 구성된다.
 - 메시지 전송은 메시지에 메시지 수신자를 추가한것. 즉 메시지 전송은 메시지 수신자, 오퍼레이션명, 인자의 조합
 ![KakaoTalk_Photo_2021-07-16-11-29-55](https://user-images.githubusercontent.com/60125719/125882523-de2337da-80ef-4418-b3b7-c3995f5cd3a9.jpeg)
 
 ### 메시지와 메서드
 메시지를 수신했을 때 실제로 어떤 코드가 실행되는지 메시지는 메시지 수신자의 실제 타입이 무엇인가에 달려 있다.  
 > ex) condition.isSatisfiedBy(screening)이라는 메시지 전송구문에서 메시지 수신자인 condition은 단순 인터페이스일 뿐, 실체화한 condition이 PeriodCondition인지, SequenceCondition인지에 따라 실행되는 코드가 달라진다. 이처럼 메시지를 수신했을 때 실제로 실행되는 함수 또는 프로시저를 **메서드**라고 부른다.  
 
 메시지 전송을 코드 상에 표기하는 시점에는 어떤 코드가 실행될 것인지를 정확하게 알 수 없다. -> 실행 시점에 실제로 실행되는 코드는 메시지를 수신하는 객체의 타입에 따라 달라지기 때문 !  
 -> 결국 결합도가 낮아지고, 유연 + 확장 가능한 코드를 작성 할 수 있게 된다.
 
### 퍼블릭 인터페이스와 오퍼레이션
 - 퍼블릭 인터페이스: 객체가 의사소통을 위해 외부에 공개하는 메시지의 집합
 - 오퍼레이션: 퍼블릭 인터페이스에 포함된 메시지
 
 ![KakaoTalk_Photo_2021-07-16-13-46-50](https://user-images.githubusercontent.com/60125719/125893262-0a37517c-3139-4a63-ba5d-0b94b501cdce.jpeg)
 
 ### 시그니처
 > 오퍼레이션(또는 메서드)의 이름과 파라미터 목록을 합친것.
 
 ### 용어정리
  - 메시지: 객체가 다른 객체와 협력하기 위해 사용하는 의사소통 메커니즘. 일반적으로 객체의 오퍼레이션이 실행되도록 요청하는 것을 "메시지 전송"이라고 부른다. 메시지는 협력에 참여하는 전송자와 수신자 양쪽 모두를 포함하는 개념이다.
  - 오퍼레이션: 객체가 다른 객체에게 제공하는 추상적인 서비스다. 메시지가 전송자와 수신자 사이의 협력 관계를 강조하는 데 비해 오퍼레이션은 메시지를 수신하는 객체의 인터페이스를 강조한다.
  - 메서드: 메시지에 응답하기 위해 실행되는 코드 블록
  - 퍼블릭 인터페이스: 객체가 협력에 참여하기 위해 외부에서 수신할 수 있는 메시지의 묶음
  - 시그니쳐: 오퍼레이션이나 메서드의 명세를 나타낸것. 이름과 인자의 목록을 포함한다.


## 인터페이스와 설계 품질
> 좋은 인터페이스는 **최소한의 인터페이스**와 **추상적인 인터페이스**를 가지고 있어야한다. 이것을 만족하기 위해서 책임주도 설계 방법을 따르자

### 디미터 법칙

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
> 이 코드의 가장 큰 단점은 ReservationAgency와 인자로 전달된 Screening과 ReservationAgency 사이의 결합도가 너무 높기 때문에 Screening의 내부 구현을 변경할 때마다 ReservationAgency도 함께 변경된다는 것. 문제의 원인은 ReservationAgency가 Screening뿐만 아니라 Movie와 DiscountCondition에도 직접 접근하기 때문.

![KakaoTalk_Photo_2021-07-16-14-22-12](https://user-images.githubusercontent.com/60125719/125895965-3663e7fe-2053-424f-a24f-fe19f2bc41c8.jpeg)
> **디미터 법칙**을 이용해 해결하자.

#### 디미터 법칙을 따르기 위해서는 클래스가 특정한 조건을 만족하는 대상에게만 메시지를 전송하도록 프로그래밍해야 한다. 모든 클래스 C와 C에 구현된 모든 메서드 M에 대해서, M이 메시지를 전송할 수 있는 모든 객체는 다음에 서술된 클래스의 인스턴스여야 한다. 이때 M에 의해 생성된 객체나 M이 호출하는 메서드에 의해 생성된 객체, 전역 변수로 선언된 객체는 모두 M의 인자로 간주한다.
 - M의 인자로 전달된 클래스(C 자체를 포함)
 - C의 인스턴스 변수 클래스

또는
 - this 객체
 - 메서드의 매개변수
 - this의 속성
 - this의 속성인 컬렉션의 요소
 - 메서드 내에 생성된 지역 객체
 
#### 디미터 법칙을 따른  ReservationAgency
 ```
 public class ReservationAgency {
     public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
         Money fee = screening.calculateFee(audienceCount);
         return new Reservation(customer, screening, fee, audienceCount);
     }
 }
```

![KakaoTalk_Photo_2021-07-16-14-22-12](https://user-images.githubusercontent.com/60125719/125896966-960ad739-ae2d-4a93-ac00-63212eeef804.jpeg)
#### 디미터 법칙과 캡슐화
디미터 법칙은 캡슐화를 다른 관점에서 표현한것. 디미터 법칙이 가치 있는 이유는 클래스를 캡슐화하기 위해 따라야하는 구체적인 지침을 제공하기 때문. 
##### 디미터 법칙을 위반하는 코드의 전형적인 모습
```
screening.getMovie().getDiscountConditions();
```
##### 위의 코드를 디미터법칙을 적용한다면
```
screening.calculateFee(audienceCount);
```

### 묻지말고 시켜라
> 훌륭한 메시지는 객체의 상태에 관해 묻지 말고 원하는 것을 시켜야한다. 절차적인 코드는 정보를 얻은 후에 결정한다. 객체지향 코드는 객체에게 그것을 하도록 시킨다.


