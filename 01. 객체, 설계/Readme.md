# 1장. 객체, 설계

## 1. 티켓 판매 어플리케이션 구현하기
 - 소극장에서 이벤트를 통해 무료 입장권을 분배하고 입장권이 있는 사람과 없는 사람을 판단하여 있는 사람은 바로 입장하고 없는 사람은 티켓을 구매한 뒤 입장하는 방식의 프로그램을 개발한다.

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
 - 소극장에 관람객이 온다면 티켓 판매원은 관람객의 가방을 열고 초대장이 있는지 확인한다.
 - 초대장이 있다면 판매원은 매표소에 있는 티켓을 관람객의 가방에 넣는다.
 - 만약 초대장이 없다면 티켓 판매원은 가방을 열어 돈을 뺴서 매표소에 있는 금액 보관함에 금액을 추가시킨다.
 - 그리고 매표소에서 티켓 하나를 가져와 관람객의 가방 안에 넣는다.
