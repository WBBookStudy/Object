> 이전 04장 항목 초반부가 중복되는 부분이 있음

## 04. 동적 메서드 탐색과 다형성
객체지향 시스템은 다음과 같은 규칙으로 실행할 메서드를 선택한다
 1. 먼저, 자신을 생성한 클래스에 적합한 메서드가 있는지 검사한다. 존재하면 메서드를 실행하고 탐색을 종료한다.
 2. 메서드를 찾이 못했다면, 부모 클래스에서 메서드 탐색을 계속한다. 적합한 메서드를 찾을 때까지 상속 계층을 따라 올라가며 계속된다.
 3. 상속 계층의 최상위 클래스에 이르렀지만 메서드를 발견하지 못한 경우 예외를 발생시키며 탐색을 중단한다.

### 04-1. 자동적인 메시지 위임
 - 상속을 이용할 경우 메시지를 처리할 방법을 알지 못할 경우 메시지에 대한 처리를 부모 클래스에 위임한다.
 - 적절한 메서드를 찾을 때까지 상속 계층을 따라 부모 클래스로 처리가 위임된다.
 - 상속을 이용할 경우 프로그래머가 메시지 위임과 관련된 코드를 명시적으로 작성할 필요가 없다. 메시지는 상속 계층을 따라 자동적으로 위임된다.
 - 자식 클래스 > 부모 클래스의 방향으로 메시지 처리가 위임되기 때문에 자식 클래스에서 어떤 메서드를 구현하고 있느냐에 따라 부모 클래스에 구현된 메서드의 운명이 결정된다.
 - 메서드 오버라이딩 : 자식 클래스의 메서드가 부모 클래스의 메서드를 감추게 된다.
```Java
Lecture lecture = new GradeLecture(...);
lecture.evalute();
```
![002](https://user-images.githubusercontent.com/50142323/132516639-3d24b916-1c28-43ab-9129-7bb6bbfd0132.png)
>Lecture.evaluate -> 는 GradeLecture.evaluate에 재정의 되어 있다.\
>실행시, self 참조에 의해 자기 자신의 클래스를 먼저 탐색하게 된다. 따라서, GradeLecture.evaluate()가 실행되게 되고, 오버라이딩된 부모 클래스의 메서드를 감추게 한다.
 - 메서드 오버로딩 : 자식 클래스의 메서드와 부모 클래스의 메서드가 공존한다.
```Java
Lecture lecture = new GradeLecture(...);
lecture.avaerage();
```
![003](https://user-images.githubusercontent.com/50142323/132517110-481faabb-9f87-47cb-9038-a5d8400723a2.png)
>오버라이딩과 마찬가지로 selft 참조를 하게 된다. GradeLecture에는 시그니처가 다르기에 부모 클래스에서 해당 시그니처를 찾게 된다.\
>이처럼 시그니처가 다르기 때문에 동일한 이름의 메서드가 공존하는 경우를 오버로딩이라 부른다.

### 04-2. 동적인 문맥
 - 메세지를 수신한 객체가 무엇이냐에 따라 메서드 탐색을 위한 문맥이 동적으로 바뀐다.
 - 이 동적인 문맥을 결정하는 것이 메세지를 수신한 객체를 가리키는 self 참조이며 self 참조 가 동적 문맥을 결정한다는 것은, 종종 어떤 메서드가 실행될지 예상하기 어렵게 만든다. 대표적인 경우가 self 전송이다.
 - self 전송은 자식 클래스에서 부모 클래스 방향으로 진행되는 동적 메서드 탐색 경로를 다시 self 참조가 가리키는 원래의 자식 클래스로 이동시킨다.
```Java
public class Lecture {
    public String stats(){
        return getEvaluationMethod();
    }

    private String getEvaluationMethod() {
        return "Pass or Fail";
    }
}
```
```Java
public class GradeLecture extends Lecture {
    @Override
    public String getEvaluationMethod() {
        return "Grade";
    }
}
```
>GradeLecture 에 stats 메시지를 전송하게 된다면

![004](https://user-images.githubusercontent.com/50142323/132522722-be1faf5a-27d6-4bf1-ac44-64e093ef9beb.png)

 1.self 참조는 GradeLecture 인스턴스를 가리키도록 설정되고 탐색은 GradeLecture 부터 시작한다.\
 2.GradeLecture 클래스에는 stats 메세지를 처리할 메서드가 없기 때문에 부모 클래스인 Lecture 에서 메서드 탐색을 계속 하다가, Lecture 에서 stats 메서드를 발견하고 실행한다.\
 3.실행 중에, self 참조가 가리키는 getEvaluationMethod 메세지를 전송하는 구문과 마주친다.\
 4.메서드 탐색은 self 참조가 가리키는 객체에서 다시 시작하게 된다.
 
> self 전송이 깊은 상속 계층과 계층 중간중간에 함정처럼 숨겨져있는 메서드 오버라이딩과 만나면 극단적으로 이해하기 어려운 코드가 만들어지게 된다.

### 04-3. 이해할수 없는 메시지
 - 동적 메서드 탐색을 통해, 메시지를 수신하면 부모 클래스까지 탐색하여 찾게된다. 만약, 최상위 계층까지 탐색후 원하는 메시지가 없다면 어떻게 될까?
 - 프로그래밍 언어가 정적 타입 언어에 속하는지, 동적 타입 언어에 속하는지에 따라 달라지게 된다.
 - 정적 타입 언어와 이해할수 없는 메시지
>ex) 정적 타입언어인 자바에서 Lecture의 인스턴스가 이해할수 없는 unknownMessage 메시지를 전송한다면?
```Java
Lecture lecture = new GradeLecture(...);
lecture.unknownMessage(); // 컴파일 에러 발생
```
> 정적 타입 언어에서는 상속 계층 전체를 탐색후 메시지를 처리할 메서드를 발견하지 못하면 컴파일 에러를 발생시킨다.

 - 동적 타입 언어와 이해할수 없는 메시지
> 동적 타입 언어도 자식으로 부터 부모 방향으로 메서드를 탐색한다.\
> 차이점이라면 컴파일단계가 존재하지 않아 실제 코드를 실행해보기 전엔 메시지 처리 가능 여부를 알 수 없다는 점이다.\
> 동적 타입 언어중 스몰토크는 메시지를 찾지 못했을때 doesNotUnderstand 메시지를 전송하고 루비의 경우 method_missing 메시지를 전송한다.
> 하지만, 이해할 수 없는 메세지에 대해 예외를 던지는 것 외에도 doesNotUnderstand 나 method_missing 메시지에 응답 할 수 있는 메서드를 구현할 수 있다. (유연하다)
