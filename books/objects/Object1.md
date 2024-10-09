# [오브젝트] Chapter 01. 객체, 설계
저자: 조영호

---

# 티켓 판매 애플리케이션 구현하기
극장에서 이벤트를 개최했고 당첨된 관람객은 초대권을 소유하였다.   
극장에서 공연 입장 시 초대권 보유 여부로 이벤트 당첨 여부를 체크하고, 초대권이 없다면 값을 지불하고 티켓을 구매해야 한다.

## 초기 구현
아래와 같이 클래스를 구성한다.
- 이벤트에 발송되는 초대장 `Invitation`
  - 공연을 관람할 수 있는 초대일자
  - 보유 시 티켓과 교환
- 티켓 (공연을 관람하기 위해 소지해야 함) `Ticket`
- 관람객 `Audience`
- 관람객이 소유한 가방 `Bag`
  - 초대장, 현금, 티켓 보유
- 매표소 `TicketOffice`
  - 초대장을 티켓으로 교환
  - 판매 티켓, 판매 금액 보관
- 판매원 `TicketSeller`
  - 매표소에서 근무
- 극장 `Theater`

그리고 `Theater`클래스가 관람객을 맞이할 수 있도록 `enter` 메서드를 구현한다.

```java
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

---

## 문제점 파악
### 소프트웨어 모듈이 가져야 하는 세 가지 기능
(로버트 마틴의 "클린 소프트웨어: 애자일 원칙과 패턴, 그리고 실천 방법")
1. 제대로 동작하는 것
2. 변경을 위해 존재하는 것
3. 코드를 읽는 사람과 의사소통하는 것

위의 코드는 1번의 항목처럼 제대로 동작은 하겠지만 2, 3번을 충족하지 못 한다.
- 변경 용이성
  - `Audience`와 `TicketSeller` 변경 시 `Theater`도 함께 변경해야 한다
  - 현재 `Theater`는 '관람객이 가방을 들고 있고 판매원이 매표소에서만 티켓을 판매한다'는 지나치게 세부적인 사실에 의존 중
- 읽는 사람과의 의사소통
  - 이해 가능한 코드: 그 동작이 우리의 예상에서 크게 벗어나지 않는 코드
  - 현재의 코드는 우리의 상식과는 너무나도 다르게 동작하기 때문 (극장이 매표소에 보관 중인 티켓에 접근하고, 관람객의 가방에서 초대권을 찾아보고 돈을 가져가서 티켓과 교환해주기 등)

그렇다고 해서 객체 사이의 의존성을 완전히 없애는 것이 정답은 아니다.   
설계의 목표: 객체 사이의 결합도를 낮춰 변경이 용이한 설계를 만드는 것

---

## 설계 개선
`Theater`가 관람객의 가방과 판매원의 매표소에 직접 접근하기 때문에 코드를 이해하기 어렵다.
우리의 직관은 '관람객과 판매원이 자신의 일을 스스로 처리해야 한다'고 생각한다.

해결 방법은 `Theater`가 `Audience`와 `TicketSeller`에 관해 너무 세세한 부분까지 알지 못 하도록 정보를 차단하면 된다.
관람객과 판매원을 자율적인 존재로 만든다.

### 자율성 높이기
**1. `Theater`의 `enter`메서드에서 `TicketOffice`에 접근하는 모든 코드를 `TicketSeller` 내부로 숨기기**
```java
public class TicketSeller {
    private TicketOffice ticketOffice;
    
    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }
    
    public void sellTo(Audience audience) {
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

public class Theater {
    private TicketSeller ticketSeller;
    
    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }
    
    public void enter(Audience audience) {
        ticketSeller.sellTo(audience);
    }
}
```
이와 같이 개념적이나 물리적으로 객체 내부의 세부적인 사항을 감추는 것을 '캡슐화'라고 한다

* 개선된 점
  * `Theater`는 `ticketOffice`가 `TicketSeller` 내부에 존재한다는 사실을 알지 못 함
  * `Theater`는 `TicketSeller`의 인터페이스에만 의존
    * 내부에 `TicketOffice` 인스턴스를 포함하고 있다는 사실은 구현의 영역

**2. `Audience`의 캡슐화 개선**
```java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

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

public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public TicketOffice getTicketOffice() {
        return ticketOffice;
    }
}
```
* 개선된 점
  * `Audience`가 `Bag`을 직접 처리하기 때문에 외부에서는 더 이상 `Audience`가 `Bag`을 소유하고 있다는 사실을 알 필요가 없다

---

## 개선점과 객체지향
### 개선된 점
수정된 `Audience`와 `TicketSeller`는 자신이 가지고 있는 소지품을 스스로 관리한다. 코드를 읽는 사람과 의사소통이라는 관점에서 개선되었다.   
`Audience`나 `TicketSeller`의 내부 구현을 변경하더라도 `Theater`를 함께 변경할 필요가 없어졌다.

### 캡슐화와 응집도
핵심은 객체 내부의 상태를 캡슐화 하고 객체 간에 오직 메시지를 통해서만 상호작용 하도록 만드는 것이다.   
밀접하게 연관된 작업만을 수행하고 연관성 없는 작업은 다른 객체에게 위임하는 객체를 '응집도가 높다'고 한다.

객체의 응집도를 높이기 위해서는 객체 스스로 자신의 데이터를 책임져야 한다. 객체는 자신의 데이터를 스스로 처리하는 자율적인 존재여야 한다.

### 절차지향과 객체지향
수정하기 전에는 `Theater`의 `enter`메서드 안에서 모든 처리가 존재했다. 이 관점에서 `enter` 메서드는 프로세스이며, `Audience`, `TicketSeller`, `Bag`, `TicketOffice`는 데이터다.   
이처럼 프로세스와 데이터를 별도의 모듈에 위치시키는 방식을 절차적 프로그래밍이라고 부른다.   
모든 처리가 하나의 클래스 안에 위치하고 나머지 클래스는 단지 데이터의 역할만 수행하기 때문이다.

수정 후에는 자신의 데이터를 스스로 처리하도록 프로세스의 적절한 단계를 `Audience`와 `TicketSeller`로 이동시켰다.   
데이터를 사용하는 프로세스가 데이터를 소유하고 있는 `Audience`와 `TicketSeller` 내부르 옮겨졌다.   
이처럼 데이터와 프로세스가 동일한 모듈 내부에 위치하도록 프로그래밍 하는 방식을 객체지향 프로그래밍이라고 부른다.

### 책임의 이동
두 방식 사이의 근본적인 차이를 만드는 것은 책임의 이동이다.   
수정 전에는 책임이 `Theater`에 집중 되어 있었다면, 수정 후에는 각 객체에 적절하게 분산 되어 있다.   
따라서 각 객체는 자신을 스르로 책임진다.   

(이 관점에서 객체지향 프로그래밍을 데이터와 프로세스를 하나의 단위로 통합해 놓는 방식으로 표현하기도 한다. 비록 이 관점이 구현 관점에서만 바라본 지극히 편협한 시각이지만, 갓 입문한 사람에게는 어느 정도 도움이 되는 실용적인 조언이다.)

객체지향 설계의 핵심은 적절한 객체에 적절한 책임을 할당하는 것이다.

---

## 모순점
### 더 개선할 점이 존재한다
`Bag`은 아직 수동적인 존재다. `Bag`의 내부 상태에 접근하는 모든 로직을 `Bag` 안으로 캡슐화해서 결합도를 낮추면 된다.
```java
public class Bag {
    private Long amount;
    private Ticket ticket;
    private Invitation invitation;

    public Long hold(Ticket ticket) {
        if (hasInvitation()) {
            setTicket(ticket);
            return 0L;
        } else {
            setTicket(ticket);
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }

    private void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    private boolean hasInvitation() {
        return invitation != null;
    }

    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```
`TicketSeller` 역시 `TicketOffice`의 자율권을 침해한다.
```java
public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    public TicketOffice(Long amount, Ticket... tickets) {
        this.amount = amount;
        this.tickets.addAll(Arrays.asList(tickets));
    }
    
    public void sellTicketTo(Audience audience) {
        plusAmount(audience.buy(getTicket()));
    }

    private Ticket getTicket() {
        return tickets.remove(0);
    }

    private void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```
그러나 이렇게 변경하면 `TicketOffice`와 `Audience` 사이에 의존성이 추가된다.   
현재에서는 `Audience`에 대한 결합도와 `TicketOffice`의 자율성 모두를 만족시킬 방법이 없다.   
둘 중 하나를 택해야 한다.

### 모순점이 존재한다
사실 실생활에서 `Theater`, `Bag`, `TicketOffice`는 자율적인 존재가 아니다.   
그럼에도 관람객, 판매원과 같은 생물로 다뤘다. 무생물 역시 스스로 행동하고 자기 자신을 책임지는 자율적인 존재로 취급한 것이다.

**비록 현실에서는 수동적인 존재라고 하더라도 이란 객체지향의 세계에 들어오면 모든 것이 능동적이고 자율적인 존재로 바뀐다.**
이 원칙을 가리켜 의인화라고 부른다.   
앞에서는 실세계에서 생물처럼 스스로 생각하고 행동하도록 소프트웨어 객체를 설계하는 것이 이해하기 쉬운 코드를 작성하는 것이라고 했지만, 훌륭한 객체지향 설계란 소프트웨어를 구성하는 모든 객체들이 자율적으로 행동하는 설계를 가리킨다.

---

## 마무리
- 좋은 설계란 무엇인가? 오늘 완성해야 하는 기능을 구현하는 코드를 짜야 하는 동시에 내일 쉽게 변경할 수 있는 코드를 짜야 한다. 좋은 설계란 오늘 요구하는 기능을 온전히 수행하면서 내일의 변경을 매끄럽게 수용할 수 있는 설계다.
- 변경을 수용할 수 있는 설계가 중요한 이유? 요구사항이 항상 변경되기 때문이다.

훌륭한 객체지향 설계란 협력하는 객체 사이의 의존성을 적절하게 관리하는 설계다. 객체 간의 의존성은 애플리케이션을 수정하기 어렵게 만드는 주범이다.
