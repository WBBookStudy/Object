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
