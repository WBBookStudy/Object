> 이전 작성글 중 04장 항목 초반부가 중복되는 부분이 있음

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

 1. self 참조는 GradeLecture 인스턴스를 가리키도록 설정되고 탐색은 GradeLecture 부터 시작한다.
 2. GradeLecture 클래스에는 stats 메세지를 처리할 메서드가 없기 때문에 부모 클래스인 Lecture 에서 메서드 탐색을 계속 하다가, Lecture 에서 stats 메서드를 발견하고 실행한다.
 3. 실행 중에, self 참조가 가리키는 getEvaluationMethod 메세지를 전송하는 구문과 마주친다.
 4. 메서드 탐색은 self 참조가 가리키는 객체에서 다시 시작하게 된다.
 
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
  - 동적 타입 언어도 자식으로 부터 부모 방향으로 메서드를 탐색한다.
  - 정적 타입 언어와의 차이점은 컴파일단계가 존재하지 않아 실제 코드를 실행해보기 전엔 메시지 처리 가능 여부를 알 수 없다는 점이다.
  - 동적 타입 언어중 스몰토크는 메시지를 찾지 못했을때 doesNotUnderstand 메시지를 전송하고 루비의 경우 method_missing 메시지를 전송한다.
  - 하지만, 이해할 수 없는 메세지에 대해 예외를 던지는 것 외에도 doesNotUnderstand 나 method_missing 메시지에 응답 할 수 있는 메서드를 구현할 수 있다.(유연하다)
 - 동적 타입 언어의 장단점
  1. 동적 타입 언어는 이해할 수 없는 메시지를 처리할 수 있는 능력을 가짐으로써 메시지가 선언된 인터페이스와 메서드가 정의된 구현을 분리할 수 있다.
  2. 하지만 동적 타입 언어의 이러한 동적인 특성과 유연성은 코드를 이해하고 수정하기 어렵게 만든다.
 - 정적 타입 언어의 장단점
  1. 정적 타입 언어는 모든 메시지는 컴파일타임에 확인되고, 이해할 수 없는 메시지는 컴파일에러로 이어진다.
  2. 이해할수 없는 메시지를 처리할 수 있는 유연성은 잃게 되지만 실행 시점에 오류가 발생할 가능성을 줄임으로서, 좀 더 프로그램이 안정적으로 실행된다.

### 04-4. self 대 super
 - self 참조의 가장 큰 특징은 동적이라는 점이다. self 참조는 메시지를 수신한 객체의 클래스에 따라 메서드 참색을 위한 문맥을 실행 시점에 결정한다.
 - self의 이런 특성과 대비해서 언급할 만한 가치가 있는 것이 super 참조이다.
 - 자식 클래스에서 부모 클래스의 구현을 재사용해야 하는 경우가 있는데, 대부분의 객체지향 언어들은 자식 클래스에서 부모 클래스의 인스턴스 변수나 메서드에 접근하기 위해 사용할 수 있는 super 참조라는 내부 변수를 제공한다.
```Java
public class GradeLecture extends Lecture {
    @Override
    public String evaluate() {
        return super.evalute() + ", " + gradesStatistics();
    }
}
```
>super 참조의 정확한 의도는 지금 self 참조가 가리키는 클래스의 부모 클래스에서 부터 메서드 탐색을 시작하세요 이다.\
>만약 부모 클래스에서 원하는 메서드를 찾지 못한다면 더 상위의 부모 클래스로 이동하면서 메서드가 존재하는지 검사한다.\
>super 참조를 통해 실행하고자 하는 메서드가 반드시 부모 클래스에 위치하지 않아도 되는 유연성을 제공하며, 메서드가 조상 클래스 어딘가에 있기만 하면 성공적으로 탐색된다.

![005](https://user-images.githubusercontent.com/50142323/132528905-eb88f6bc-6638-4062-8e58-e027534d691c.png)
 - 이처럼 super 참조를 통해 메시지를 전송하는 것은 마치 부모클래스의 인스턴스에게 메시지를 전송하는 것처럼 보이기에, super 전송(super send) 이라고 부른다.

## 05. 상속 대 위임

### 05-1. 위임과 self 참조
 - 상속을 이용하면 자식 클래스에서 메시지를 처리하지 못하는 경우 상속 계층에 따라 메시지를 위임한다. 이 경우 self참조는 메시지를 위임하더라도 항상 메시지를 수신한 객체를 가리킨다.

![006](https://user-images.githubusercontent.com/50142323/132530316-284c58ab-9b2a-4438-a157-18e438345bef.jpeg)

 - 코드 예제가 있지만 ruby로 작성되어서 생략...(426p 참고)

> 상속 계층을 구성하는 객체들 사이에서는 self 참조를 공유하기 때문에 개념적으로 각 인스턴스에서 self 참조를 공유하는 self 변수를 포함하는 것처럼 표현할 수 있다.\
> 상속은 동적으로 메서드를 탐색하기 위해 현재의 실행문맥을 가지고 있는 self 참조를 전달한다. 그리고 이 객체들 사이에서는 메시지를 전달하는 과정이 자동으로 이뤄진다.\
> 이처럼 자신이 수신한 메시지를 다른 객체에게 동일하게 전달해서 처리를 요청하는것을 위임이라고 한다.

### 05-2. 프로토타입 기반의 객체지향 언어
 - 다른 언어에서는 클래스가 아닌 객체를 이용해서도 상속을 흉내내고 있다.
 - 클래스가 존재하지 않고 오직 객체만 존재하는 프로토타입 기반의 객체지향 언어에서 상속을 구현하는 유일한 방법은 객체 사이의 위임을 이용하는 것이다.
 - 클래스 기반의 객체지향 언어들이 상속을 이용해 클래스 사이에 self 참조를 자동으로 전달하는 것처럼 프로토타입 기반의 객체지향 언어들 역시 위임을 이용해 객체 사이에 self 참조를 자동으로 전달한다.

>현재 가장 널리 사용되는 프로토타입 기반의 객체지향 언어는 자바스크립트 이다.\
>자바스크립트의 모든 객체들은 다른 객체를 가리키는 용도로 사용되는 prototype 이라는 이름의 링크를 가진다.\
>prototype은 언어 차원에서 제공되기 때문에 self 참조를 직접 전달하거나 메세지 포워딩을 번거롭게 구현하지 않아도 된다.

```JavaScript
function Lecture(name, scores) {
  this.name = name;
  this.scores = scores;
}

Lecture.prototype.stats = function() {
  return "Name: " + this.name + ", Evaludation Method: " + this.getEvaluationMethod();
}

Lecture.prototype.getEvaluationMethod = function() {
  return "Pass or Fail"
}
```
>자바스크립트의 인스턴스는 메시지를 수신하면 먼저 메시지를 수신한 객체의 prototype 안에서 메시지에 응답할 적절한 메서드가 있는지 검사한다.\
>만약 메서드가 존재하지 않는다면 prototype이 가리키는 객체를 따라 메시지 처리를 자동적으로 위임한다.\
>자바스크립트에서는 prototype 체인으로 연결된 객체 사이에 메시지를 위임함으로써 상속을 구현할 수 있다.

```JavaScript
function GradeLecture(name, canceled, scores) {
  Lecture.call(this, name, scores);
  this.canceled = canceled;
}

GradeLecture.prototype = new Lecture();
GradeLecture.prototype.constructor = GradeLecture;

GradeLecture.prototype.getEvaludationMethod = function() {
  return "Grade"
}
```
>위의 코드에서는 GradeLecture의 prototype에 Lecture 인스턴스를 할당했다.\
>이 과정을 통해 GradeLecture를 이용해 생성된 모든 객체들이 prototype을 통해 Lecture에 정의된 모든 속성과 함수에 접근할 수 있게 된다.\
>이제 메시지를 전송하면 prototype으로 연결된 객체 사이의 경로를 통해 객체 사이의 메서드 탐색이 자동으로 이뤄진다.

```JavaScript
var grade_lecture = new GradeLecture("OOP", false, [1, 2, 3]);
grade_lecture.stats();
```
>stats 메시지를 전송하면
>1. GradeLecturedp stats매서드가 존재하는지 검사한다.
>2. GradeLecture에는 stats메서드가 없기에 다시 prototype을 따라 Lecture의 인스턴스에 접근 후 stats 메서드가 있는지 검사하고 stats를 발견하고 메서드를 실행한다.
>3. Lecture의 stats메서드를 실행하는 도중에 this.getEvaluationMethod()를 발견한다.
>4. 상속과 마찬가지로 self참조가 가리키는 현재객체에서부터 다시 메서드 탐색을 실행하게 된다.
>5. 현재 객체의 prototype이 참조하는 GradeLecture의 인스턴스에서 getEvaluationMethod메서드를 발견하고 이 메서드를 실행함으로서 탐색이 종료된다.

![007](https://user-images.githubusercontent.com/50142323/132535157-fca52c8e-ba7b-4de9-a0f7-570d087a2fc1.jpeg)

자바스크립트에서는 클래스가 존재하지 않기 때문에 오직 객체들 사이의 메시지 위임만을 이용해 다형성을 구현한다.\
이것은 객체지향 패러다임에서 클래스가 필수 요소가 아니라는 점을 잘 보여주며, 또한 상속 이외의 방법으로도 다형성을 구현할 수 있다는 사실을 보여준다.
