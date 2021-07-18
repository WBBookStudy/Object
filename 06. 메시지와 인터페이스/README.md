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
객체지향의 기본은 함께 변경될 확률이 높은 정보와 행동을 하나의 단위로 통합하는 것. 묻지 말고 시켜라 원칙을 따르면 객체의 정보를 이용하는 행동을 객체의 외부가 아닌 내부에 위치시키기 때문에 자연스럽게 정보와 행동을 동일한 클래스 안에 두게 된다. -> 정보 전문가에게 책임을 할당하게되고, 높은 응집도를 가진 클래스를 얻을 확률이 높아진다.  
내부의 상태를 이용해 어떤 결정을 내리는 로직이 객체 외부에 존재하는가? 그렇다면 해당 객체가 책임져야 하는 어떤 행동이 객체 외부로 누수된 것 !  
훌륭한 인터페이스를 수확하기 위해서는 객체가 어떻게 작업을 수행하는지를 노출해서는 안된다. **인터페이스는 객체가 어떻게 하는지가 아니라 무엇을 하는지를 서술**해야한다. 

### 의도를 드러내는 인터페이스
Smalltalk Best Practice Patterns - kent Beck 에서의 메서드를 명명하는 두가지 방법
 1. 메서드가 작업을 어떻게 수행하는지를 나타낸다.
```
public class PeriodCondition {
 public boolean isStisfiedByPeriod(Screening screening) { ... }
}

public class SequenceCondition {
 public boolean isSatisfiedBySequence(Screening screening) { ... }
}
```
-> 이런 스타일은 좋지 않다. 
 - 메서드에 대해 제대로 커뮤니케이션하지 못한다. 위의 함수 두개 다 할인조건을 판단하는 동일한 작업을 수행. 하지만 메서드의 이름이 다르기 때문에 두 메서드의 내부구현을 정확하게 이해하지 못한다면 두 메서드가 동일한 작업을 수행한다는 사실을 아라채기 어려움.
 - 메서드 수준에서 캡슐화를 위반. 협력하는 객체의 종류를 알 수 있다. -> 책임을 수행하는 방법을 드러내는 메서드를 사용한 설계는 변경에 취약할 수 밖에 없다.
 2. '어떻게'가 아니라 '무엇'을 하는지를 드러내는 것.
```
public class PeriodCondition {
 public boolean isStatisfiedBy(Screening screening) { ... }
}

public class SequenceCondition {
 public boolean isSatisfiedBy(Screening screening) { ... }
}
```
-> 이것을 인터페이스로 묶자.
```
public interface DiscountCondition {
 boolean isSatisfiedBy(Screening screening);
}

public class PeriodCondition implements DiscountCondition {
 public boolean isStatisfiedBy(Screening screening) { ... }
}

public class SequenceCondition implements DiscountCondition {
 public boolean isSatisfiedBy(Screening screening) { ... }
}
```
> 이처럼 어떻게 하느냐가 아니라 무엇을 하느냐에 따라 메서드의 이름을 짓는 패턴을 **의도를 들어내는 선택자(Intention Revealing Selector)**라고 부른다.
> 의도를 들어내는 선택자 뿐 아니라 의도를 드러내는 인터페이스도 있음.. 요약하면 구현과 관련된 모든 정보를 캡슐화하고 객체의 퍼블릭 인터페이스는 협력과 관련된 의도만을 표현해야한다는 내용

### 함께 모으기

#### 디미터 법칙을 위반하는 티켓 판매 도메인
```
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee()); // 디미터 법칙을 위반할 때 나타나는 기차 충돌 스타일의 전형적인 모습. Theater는 audience뿐 아니라 내부의 bag에게도 메시지를 전송, 결과적으로 Theater는 Audience의 퍼블릭 인터페이스 뿐 아니라 내부구조에 대해서도 결합된다.
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```
> 위의 주석에 관련한 문제점 : 1. Audience의 Bag은 이제 수정하는 순간 Theater 클래스도 같이 수정해주어야 하는 상황이 됨 2. 이 클래스의 작성자는 Audience의 클래스를 빠삭하게 알고 있어야 코드 작성이 가능함.
![KakaoTalk_20210718_160602108](https://user-images.githubusercontent.com/60125719/126058755-706deb8a-3607-4c11-81dd-fd56579035e3.jpg)

#### 묻지 말고 시켜라
Theater는 TicketSeller와 Audience의 내부 구조에 관해 묻지 말고 원하는 작업을 시켜야 한다.  
먼저 Theater가 TicketSeller에게 자신이 원하는 일을 시키도록 수정하자.
```
public class TicketSeller {
 public void setTicket(Audience audience) {
  if (audience.getBag().hasInvitation()) {
      Ticket ticket = ticketOffice.getTicket();
      audience.getBag().setTicket(ticket);
  } else {
      Ticket ticket = ticketOffice.getTicket();
      audience.getBag().minusAmount(ticket.getFee());
      ticketOffice.plusAmount(ticket.getFee());
      audience.getBag().setTicket(ticket);
  }
 }
}

```
>  Theater는 자신의 속성으로 포함하고 있는 TicketSeller의 인스턴스에게만 메시지를 전송하게 됐다. -> 디미터 법칙 준수

```
public class Theater {
 public void enter(Audience audience) {
  ticketSeller.setTicket(audience);
 }
}
```

```
public class Audience {
 public Long setTicket(Ticket ticket) {
  if (bag.hasInvitation()) {
   bag.setTicket(ticket);
   return 0L;
  } else {
   bag.setTicket(ticket);
   bag.minusAmount(ticket.getFee());
   return ticket.getFee();
  }
 }
}
```
> 이제 TicketSeller는 속성으로 포함하고 있는 TicketOffice의 인터페이스와 인자로 전달된 Audience에게만 메시지를 전송한다. 따라서 TicketSeller 역시 디미터 법칙을 준수한다.

```
public class TicketSeller {
 public void setTicket(Audience audience) {
  ticketOffice.plusAmount(
  audience.setTicket(ticketOffice.getTicket()));
 }
}
```
> 문제는 Audience다. Audience의 setTicket 메서드를 살펴보면 Audience가 Bag에게 원하는 일을 시키기 전에 hasInvitation 메서드를 이용해 초대권을 가지고 있는지를 묻는다. Audience의 setTicket 메서드 구현을 Bag의 setTicket 메서드로 옮기자

```
public class Bag {
    private Long amount;
    private Invitation invitation;
    private Ticket ticket;

    public Bag(long amount) {
        this(null, amount);
    }

    public Bag(Invitation invitation, long amount) {
        this.invitation = invitation;
        this.amount = amount;
    }

    public Long setTicket(Ticket ticket) {
        if (hasInvitation()) {
            this.ticket = ticket;
            return 0L;
        } else {
            this.ticket = ticket;
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }

    private boolean hasInvitation() {
        return invitation != null;
    }

    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```
> Audience의 setTicket 메서드가 Bag의 setTicket 메서드를 호출하도록 수정하면 묻지 말고 시켜라 스타일을 따르고 디미터 법칙을 준수하는 Audience를 얻을 수 있다.

```

public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Long setTicket(Ticket ticket) {
        return bag.setTicket(ticket);
    }
}
```

##### 디미터 법칙과 묻지 말고 시켜라 원칙에 따라 코드를 리펙터링한 후에 Audience 스스로 자신의 상태를 제어하게 됐다는 점에 주목하자. 

#### 인터페이스에 의도를 드러내자
Theater가 TicketSeller에게 setTicket 메시지를 전송해서 얻고 싶었던 결과: Audience에게 티켓을 판매하는 것. 즉 setTicket이 아닌 sellTo가 의도를 더 명확하게 표현하는 메시지.  
TicketSeller가 Audience에게 setTicket 메시지를 전송하는 이유는?: Audience가 티켓을 사도록 만들겠다. 즉 buy가 더 어울리는 메시지.  
Audience가 Bag에게 setTicket메시지를 전송하는 의도는?: 티켓을 보관하도록 하기 위해서. 즉 hold가 더 어울리는 메시지.

```
public class TicketSeller {
 public void sellTo(Audience audience) { ... }
}

public class Audience {
 public Long buy(Ticket ticket) { ... }
}

public class Bag {
 public Long hold(Ticket ticket) { ... }
}
```


## 원칙의 함정
> 설계는 트레이드오프의 산물. 초보자는 원칙을 맹목적으로 추종한다. 원칙이 현재 상황에 부적합하다고 판단된다면 과감하게 원칙을 무시하라.


### 디미터 법칙은 하나의 도트(.)를 강제하는 규칙이 아니다.
ex)
```
IntStream.of(1, 15, 20, 3, 9).filter(x -> x > 10).distinct().count();
```
> 디미터 법칙을 어긴걸까?
> 위 코드에서 of, filter, distinct 메서드는 모두 IntStream이라는 동일한 클래스의 인스턴스를 반환함. 즉, 이들은 IntStream의 인스턴스를 또 다른 IntStream의 인스턴스로 변환하므로 이 코드는 디미터 법칙을 위반하지 않는다. 디미터 법칙은 결합도와 관련된 것이며, 이 결합도가 문제가 되는 것은 객체의 내부 구조가 외부로 노출되는 경우로 한정된다. 
> 이런 종류의 코드와 마주쳐야 하는 순간이 온다면 '과연 여러개의 도트를 사용한 코드가 객체의 내부 구조를 노출하고 있는가?'에 대해 답해보면 된다.

### 결합도와 응집도의 충돌
일반적으로 어떤 객체의 상태를 물어본 후 반환된 상태를 기반으로 결정을 내리고 그 결정에 따라 객체의 상태를 변경하는 코드는 묻지말고 시켜라 스타일로 변경해야 한다.
```
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee()); 
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```
> Theater는 Audience 내부에 포함된 Bag에 대해 질문한 후 반환된 결과를 이용해 Bag의 상태를 변경한다. -> Audience의 캡슐화를 위반

```
public class Audience {
 public Long buy(Ticket ticket) {
  if (bag.hasInvitation()) {
   bag.setTicket(ticket);
   return 0L;
  } else {
   bag.setTicket(ticket);
   bag.minusAmount(ticket.getFee());
   return ticket.getFee();
  }
 }
}
```
> 이제 Audience는 상태와 함께 상태를 조작하는 행동도 포함하기 때문에 응집도가 높아졌다. 
> 이 예제에서 알 수 있는 것처럼 위임 메서드를 통해 객체 내부 구조를 감추는 것은 협력에 참여하는 객체들의 결합도를 낮출 수 있는 동시에 객체의 응집도를 높일 수 있는 가장 효과적인 방법이다.

+++ 이 외의 예제들을 200 페이지에서 보는것을 추천


