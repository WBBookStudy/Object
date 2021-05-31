# 1장. 객체, 설계

## 1. 티켓 판매 어플리케이션 구현하기
 - 극장에서 이벤트를 통해 무료 입장권을 분배하고 입장권이 있는 사람과 없는 사람을 판단하여 있는 사람은 바로 입장하고 없는 사람은 티켓을 구매한 뒤 입장하는 방식의 프로그램을 개발한다.

### 1-1. Invitation 클래스
```
/**
 * 이벤트 당첨자에게 발송되는 초대장
 */
public class Invitation {
    private LocalDateTime when;
}
```

### 1-2. Ticket 클래스
```
/**
 * 공연을 관람하기 원하는 모든 사람들이 가지고 있는 티켓
 */
public class Ticket {
    private Long fee;

    public Long getFee() {
        return fee;
    }
}
```

### 1-3. Bag 클래스
```
/**
 * 소지품을 보관할 용도인 가방
 */
public class Bag {

    private Long amount; // 현금
    private Invitation invitation; // 초대장
    private Ticket ticket; // 티켓

    public Bag(Long amount) {
        this.amount = amount;
    }

    public Bag(Long amount, Invitation invitation) {
        this.amount = amount;
        this.invitation = invitation;
    }

    /**
     * 초대장이 있는지 없는지 확인하는 메서드
     * @return 초대장이 있는지 true
     */
    public boolean hasInvitation() {
        return invitation != null;
    }

    /**
     * 티켓이 있는지 없는지 확인하는 메서드
     * @return 초대장이 있으면 true
     */
    public boolean hasTicket() {
        return ticket != null;
    }

    /**
     * 티켓을 매개변수로 받고 Bag 객체의 ticket 멤버 변수에 세팅
     * @param ticket-티켁 객체
     */
    public void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    /**
     * 현금을 감소시키는 메서드
     * @param amount-금액
     */
    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    /**
     * 현금을 증가시키는 메서드
     * @param amount-금액
     */
    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```

### 1-4. TicketOffice 클래스
```
/**
 * 극장에 입장하기 위한 매표소
 */
public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    public TicketOffice(Long amount, Ticket... tickets) {
        this.amount = amount;
        this.tickets.addAll(Arrays.asList(tickets));
    }


    /**
     * 티켓을 교환해죽거나 티켓을 판매해줄 때 tickets 컬렉션에 있는 첫 번째 요소륿 반환
     * @return 티켓 하나
     */
    public Ticket getTicket() {
        return tickets.remove(0);
    }

    /**
     * 매표소 금액을 차감시키는 메서드
     * @param amount-매표소에 있는 총 금액
     */
    public void minusAmount(Long amount) {
        this.amount -= amount;
    }
    /**
     * 티켓을 판매할 때 증가하는 수익을 지정하는 메서드
     * @param amount-매표소에 있는 총 금액
     */
    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```

### 1-5. TicketSeller 클래스
```
/**
 * 티켓을 판매하기 위한 판매원 클래스
 * 초대장을 티켓으로 교환,
 * 티켓을 판매 하는 역할을 수행한다.
 */
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    /**
     * ticketOffice 의 getter
     * @return TicketOffice
     */
    public TicketOffice getTicketOffice() {
        return ticketOffice;
    }
}
```

### 1-6. Theater 클래스
```
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    /**
     * @param audience-티켓을 구매하려는 Audience 객체
     * 극장에 입장할 때 audience 가 가지고 있는 bag 객체에서 각각을 조회
     * 초대장이 있을 떄 : 초대장이 있으면 금액 차감 없이 티켓을 줌
     * 초대장이 없을 떄 : audience 의 금액을 차감하여 티켓을 줌
     */
    public void enter(Audience audience) {
        if(audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        }else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```

## 2. 문제점
### 위 코드들이 나타내는 일련의 프로세스를 정리하면
 - 극장에 관람객이 온다면 티켓 판매원은 관람객의 가방을 열고 초대장이 있는지 확인한다.
 - 초대장이 있다면 판매원은 매표소에 있는 티켓을 관람객의 가방에 넣는다.
 - 만약 초대장이 없다면 티켓 판매원은 가방을 열어 돈을 뺴서 매표소에 있는 금액 보관함에 금액을 추가시킨다.
 - 그리고 매표소에서 티켓 하나를 가져와 관람객의 가방 안에 넣는다.

### 무엇이 문제인가?
 - 직관적으로 생각해봤을 때 우리는 표를 구매할 때 우리 스스로 가방을 열어 초대장을 확인하고 초대장이 없으면 우리가 직접 돈을 지불하고 계산하여 티켓을 받고 가방에 넣는다. 
 - 위의 예제에서는 관람객(우리)은 단순히 수동적인 존재이며 극장 이라는 객체가 모든 일을 처리한다. 뿐만 아니라 판매원 또한 극장 자체가 판매원의 허락없이 티켓을 가져가고 돈을 적립하고 있다.
 - 각각의 클래스는 데이터로서의 역할만을 담당하고 있고 극장 이라는 클래스가 모든 일을 처리하고 있다.
 - 다시 말해 데이터에서 변화가 발생하는 경우 극장에서의 로직은 언제나 변경되어야 하는 대상이 되며 극장 은 너무나 많은 클래스와 의존성을 맺고 있다고 볼 수 있다.

###  로버트 마틴이 말하는 소프트웨어 모듈의 3가지 특성
 - 정상적으로 동작해야 한다.
 - 변경이 용이해야 한다.
 - 이해하기 쉬워야 한다.

## 3. 해결책 : 자율적인 객체
 - 각각의 객체(관람객, 판매원..)가 자율적인 존재 로서 본인의 역할과 책임을 수행하고 있지 않기 때문에 외부 클래스(극장)에서 이를 관리하게 되고 그렇기 때문에 직관적으로 이해하기 어렵고 하나의 클래스(극장)가 너무 많은 곳에 의존함으로써 결합도가 높아진다. 해결책은 당연히 각각의 객체에게 자율성을 부여하는 것이다.
 - 해결책으로는 각각의 객체에게 자율성을 부여하여 본인의 역할과 책임을 수행하게 한다.

###  캡슐화
 - 티켓을 판매하는 기능은 극장의 역할이 아니다. 판매원이 해야하는 역할이다. 이를 위해 기존에 극장에서 담당하는 enter의 내부 메서드를 판매원이 담당할수 있도록 변경하였다.
 - 객체 내부의 세부적인 사항을 캡슐화 하여 외부 객체에게 내부 접근을 불가능하게 하여 객체 사이의 결합도를 낮추어 변경에 용이하다. 
```
public class Theater {
    	private TicketSeller ticketSeller;
    
    	public Theater(TicketSeller ticketSeller) {
    		 this.ticketSeller = ticketSeller;
    	}
    
    	public void enter(Audience audience) {
    		 ticketSeller.toSell(audience);   
    	}
}
    
public class TicketSeller {
    	private TicketOffice ticketOffice;
    
    	public TicketSeller(TicketOffice ticketOffice) {
    		 this.ticketOffice = ticketOffice;
    	}
    
     //기존 극장에서 담당하는 enter의 내부 메서드를 toSell 메서드로 작성
    	public void toSell(Audience audience) {
       if(audience.getBag().hasInvitation()) {
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

 - 마찬가지로 관객이 직접 가방을 꺼내지 않고 극장이 이를 꺼내는 것이 문제가 되었다. 이 또한 고쳐보자
```
public class TicketSeller {
    	private TicketOffice ticketOffice;
    
    	public TicketSeller(TicketOffice ticketOffice) {
    		 this.ticketOffice = ticketOffice;
    	}
    
    	public void toSell(Audience audience) {
    		 ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    	}
}
    
public class Audience {
    	private Bag bag;
    
    	public Audience(Bag bag) {
    		 this.bag = bag;
    	}
    
    	public Long buy(Ticket ticket) {
     
    		 if(bag.hasInvitation()) {
    			   bag.setTicket(ticket);
    			   return 0L;
    		 }else {
    			   bag.setTicket(ticket);
    			   bag.minusAmount(ticket.getFee());
    			   return ticket.getFee();
    		 }
   	}
}
```
 - 기존에 극장이 가방에서 돈을 가져가던 부분이 리팩토링을 통하여 관객 스스로 자신의 가방안에 초대장이 들어있는지를 스스로 확인한다.
 - Audience가 Bag을 직접 처리하기 때문에 외부에서는 더이상 Audience가 Bag을 소유하고 있다는것을 알 필요가 없다.
 - TicketSeller 또한 Audience의 buy 메서드를 호출하게 하여 Audience의 인터페이스에만 의존하게 되었다
 - 구매하는 로직이 변경되더라도 Audience에서만 로직을 변경하면 다른 클래스는 영향을 받지 않는다. 즉 의존성이 낮아지고 결합도가 낮아졌다고 볼 수 있다.

## 4. 절차지향과 객체지향
 - 절차지향의 관점에서 Theater에서 enter는 프로세스이고 나머지 클래스들 (Audience.. ) 등은 데이터를 말한다. 프로세스와 데이터를 별도의 모듈에 위치시키는 방식을 절차 지향 프로그래밍이라고 부른다.
 - 이와 반대로 프로세스가 동일한 모듈 내부에 위치하도록 하는 방식을 객체지향 프로그래밍이라 부른다.

## 5. 책임의이동
결국 두 방식의 근본적인 차이는 책임의 이동인데, 위에서 처음 표현한 코드는 Theater에 책임이 집중되어 있었다. 수정후에는 각 객체의 책임이 적절하게 분배되고, 따라서 각 객체들은 스스로 책임을 지게된다.
 - 책임이란? - 기능을 가리키는 객체지향 세계의 용어로 생각해도 무방하다.


## 5. 추가적인 개선
 - Bag는 실제로는 무생물 이기에 자율성을 갖지 않지만 객체지향의 세계에서는 각각의 객체는 자율성을 가지게 된다.
 - 객체지향은 현실세계를 모방하는 것이 아닌 재창조하는 과정 이라고 설명되며 각각의 존재들은 의인화 및 은유 를 통해 표현된다.
 - Bag가 hold 라는 메서드를 통해 직접 초대장을 확인하고 인자로 입력받은 초대장을 가방에 입력하는 형태로 변경함으로써 Bag에게 자율권을 부여한다.
```
public class Bag {
   	private Long amount;
   	private Invitation invitation;
   	private Ticket ticket;
   
   	public Bag(Long amount) {
   		this(null,amount);
   	}
   
   	public Bag(Invitation invitation, long amount) {
   		this.amount = amount;
   		this.invitation = invitation;
   	}
   
   	public Long hold(Ticket ticket) {
   		if(hasInvitation()) {
   			setTicket(ticket);
   			return 0L;
   		} else {
   			setTicket(ticket);
   			minusAmount(ticket.getFee());
   			return ticket.getFee();
   		}
   	}
   
   	private boolean hasInvitation() {
   		return invitation != null;
   	}
   
   	public boolean hasTicket() {
   		return ticket != null;
   	}
   
   	private void setTicket(Ticket ticket) {
   		this.ticket = ticket;
   	}
   
   	private void minusAmount(Long amount) {
   		this.amount -= amount;
   	}
   
   	public void plusAmount(Long amount) {
   		this.amount += amount;
   	}
 }
```

 - 

 
