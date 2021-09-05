# 다형성

코드 재사용을 목적으로 상속을 사용하면 변경하기 어렵고 유연하지 못한 설계에 이를 확률이 높아진다.
상속의 목적은 코드 재사용이 아니다.
상속은 타입 계층을 구조화하기 위해 사용해야 한다.
타입 계층은 객체지향 프로그래밍의 중요한 특성 중의 하나인 다형성의 기반을 제공한다.

## 01. 다형성
 - 다형성(Polymorphism) 이라는 단어는 ploy(많은) + morph(형태)의 합성어로 많은 형태를 가질 수 있는 능력을 의미한다.

다형성은 하나의 추상 인터페이스에 대해 코드를 작성하고 이 추상 인터페이스에 대해 서로 다른 구현을 연결할 수 있는 능력으로 정의된다.
다형성은 여러 타입을 대상으로 동작할 수 있는 코드를 작성할 수 있는 방법이다.
![polymorphism](https://user-images.githubusercontent.com/50142323/132130858-371b11b9-cff4-4276-b42d-3853bc34b910.jpeg)

 - 오버로딩 다형성
  - 하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우
```Java
public class Money {
    public String plus(Money amount) { ... }
    public String plus(BigDecimal amount) { ... }
    public String plus(long amount) { ... }
}
```
 - 강제 다형성
  - 자동적인 타입 변환 방식
