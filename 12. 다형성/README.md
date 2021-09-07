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
> 하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우
```Java
public class Money {
    public String plus(Money amount) { ... }
    public String plus(BigDecimal amount) { ... }
    public String plus(long amount) { ... }
}
```
> 유사한 작업을 수행하는 메서드의 이름을 통일할 수 있기 때문에 기억해야 하는 이름의 수를 극적으로 줄일 수 있다.
 - 강제 다형성
> 자동적인 타입 변환 방식\
> ex) '+' 의 경우 피연산자가 모두 정수일 경우에는 덧셈 연산자로 동작하지만 하나는 정수형이고 다른 하나는 문자열일경우 연결 연산자로 동작하고 정수형 피연산자는 문자열 타입으로 강제 형변환 된다.\
> 일반적으로 오버로딩 다형성과 강제 다형성을 함께 사용하면 실제로 어떤 메서드가 호출될지 판단하기가 어려워지기 떄문에 모호해질수 있다.

 - 매개변수 다형성
> 제네릭 프로그래밍(데이터 형식에 의존하지 않고, 하나의 값이 여러 다른 데이터 타입들을 가질 수 있는 기술에 중점을 두어 재사용성을 높일 수 있는 프로그래밍 방식)과 관련이 높다.\
> 클래스의 인스턴스 변수나 메서드의 매개변수 타입을 임의의 타입으로 선언한 후 사용하는 시점에 구체적인 타입으로 지정하는 방식이다.

 - 포함 다형성
> 메시지가 동일하더라도 수신한 객체의 타입에 따라 실제로 수행되는 행동이 달라지는 능력을 의미.\
> 일반적인 다형성을 얘기할 때 포함 다형성을 얘기한다.
> 서브타입 다형성이라고도 부른다.
```Java
public class Movie {
    private DiscountPolicy discountPolicy;

    public Money calculateMovieFee(Screening screening) { 
        return fee.minus(discountPolicy.caculateDiscountAmount(screening));
    }
}
```
> Movie 클래스는 discountPolicy에게 caculateDiscountAmount 메시지를 전송하지만 실제로 실행되는 메서드는 메시지를 수신한 객체의 타입에 따라 달라진다.

## 02. 상속의 양면성
 - 객체지향 패러다임의 근간을 이루는 아이디어는 데이터와 행동을 객체라고 불리는 하나의 실행 단위 안으로 통합하는 것이다. 따라서 객체지향 프로그램을 작성하기 위해서는 항상 데이터와 행동이라는 두 가지 관점을 함께 고려해야 한다.
```Java
public class Lecture {
    private int pass; // 이수여부 판단할 기준 점수
    private String title; // 과목명
    private List<Integer> scores = new ArrayList<>(); // 학생들의 성적 리스트

    public Lecture(String title, int pass, List<Integer> scores) {
        this.title = title;
        this.pass = pass;
        this.scores = scores;
    }

    public double average() {
        return scores.stream().mapToInt(Integer::intValue).average().orElse(0);
    }

    public List<Integer> getScores() {
        return Collections.unmodifiableList(scores);
    }

    public String evaluate() {
        return String.format("Pass:%d Fail:%d", passCount(), failCount());
    }

    private long passCount() {
        return scores.stream().filter(score -> score >= pass).count();
    }

    private long failCount() {
        return scores.size() - passCount();
    }
}
```
> 수강생들의 성적을 계산하는 Lecture 클래스이다.

```Java
Lecture lecture = new Lecture("객체지향 프로그래밍", 70, Arrays.asList(81, 95, 75, 50,45));
String evaluration = lecture.evaluate(); // 결과 => "Pass:3, Fail:2"
```
> 이수 기준 70점인 '객체지향 프로그래밍'과목의 수강생 5명의 대한 설정 통계를 구하는 코드.

```Java
public class GradeLecture extends Lecture {
    private List<Grade> grades;  //등급별 통계를 추가하기 위해 인스턴스변수 선언

    public GradeLecture(String name, int pass, List<Grade> grades, List<Integer> scores) {
        super(name, pass, scores);
        this.grades = grades;
    }
}
```
> Lecture클래스에는 새로운 기능을 구현하는데 필요한 대부분의 데이터와 메서드를 포함하고 있기에, Lecture클래스를 상속받으면 새로운 기능을 빠르고 쉽게 추가할수 있다.\
> 등급별 통계를 추가하기 위해 Grade 인스턴스들을 리스트로 보관하는 인스턴스 변수 Grades를 추가.

```Java
public class Lecture {
    private String name;         //등급 이름
    private int upper, lower;    //최대, 최소 성적

    private Grade(String name, int upper, int lower){
        this.name = name;
        this.upper = upper;
        this.lower = lower;
    }
    
    public String getName(){
        return name;
    }
    
    public boolean isName(String name){
        return this.name.equals(name);
    }
    
    public boolean include(int score){
        return score >= lower && score <= upper;
    }
    
}
```
> 등급의 이름과 각 등급범위를 정의하는 최소 성적과 최대성적을 인스턴스변수로 포함한다.\
> include메소드는 수강생의 성적이 등급에 포함되는지 체크한다.


```Java
public class GradeLecture extends Lecture {
    private List<Grade> grades;  //등급별 통계를 추가하기 위해 인스턴스변수 선언

    public GradeLecture(String name, int pass, List<Grade> grades, List<Integer> scores) {
        super(name, pass, scores);
        this.grades = grades;
    }
    
    @Override
    public String evaluate() {
        return super.evaluate() + ", " + gradesStatistics();
    }
    
    private String gradesStatistics() {
        return grades.stream().map(grade -> format(grade)).collect(joining(" "));
    }

    private String format(Grade grade) {
        return String.format("%s:%d", grade.getName(), gradeCount(grade));
    }

    private long gradeCount(Grade grade) {
        return getScores().stream().filter(grade::include).count();
    }
}
```
> 학생들의 이수 여부와 등급별 통계를 함께 반환하도록 evaluate 메소드를 재정의 한다.\
> 일반적으로 super는 자식클래스 내부에서 부모클래스의 인스턴스 변수다 메서드에 접근하는데 사용한다 라는 정도로만 이해해두자.(엄밀히 말하자면 가시성이 private인 경우에는 접근이 불가하다)\
> 메서드 오버라이딩 -> 자식클래스 안에 상속받은 메서드와 동일한 시그니처의 메서드를 재정의해서 부모 클래스의 구현을 새로운 구현으로 대체하는것

```Java
Lecture lecture = new GradeLecture("객체지향 프로그래밍", 
                                    70, 
                                    Arrays.asList(new Grade("A",100,95),
                                                  new Grade("B",94,80),
                                                  new Grade("C",79,70),
                                                  new Grade("D",69,50),
                                                  new Grade("E",49,0)),
                                    Arrays.asList(81, 95, 75, 50,45));
String evaluration = lecture.evaluate(); // 결과 => "Pass:3, Fail:2"
```
> GradeLecture클래스의 인스턴스 변수에게 evaluate 메시지를 전송하면 Lecture의 evaluate메서드를 오버라이딩 한 GradeLecture의 evaluate메서드가 실행된다.

```Java
public class GradeLecture extends Lecture {
    public double average(String gradeName) {   //등급별 평균 성적을 구하는 average메서드를 추가
        return grades.stream()
                .filter(each -> each.isName(gradeName))
                .findFirst()
                .map(this::gradeAverage)
                .orElse(0d);
    }

    private double gradeAverage(Grade grade) {
        return getScores().stream()
                .filter(grade::include)
                .mapToInt(Integer::intValue)
                .average()
                .orElse(0);
    }
}
```
> 부모클래스에는 없던 새로운 클래스를 추가하는것도 가능하다.\
> 부모인 Lecture클래스에 정의된 average 메서드와는 시그니처가 다르기에 GradeLecture클래스의 average메서드는 Lecture클래스에 정의된 average 메서드를 대체하지 않으며 서로 공존할수 있다.\
> 메서드 오버로딩 -> 부모클래스에서 정의한 메서드와 이름은 동일하지만 시그니처는 다른 메서드를 자식클래스에 추가하는것


### 02-1. 데이터 관점의 상속
 - 자식 클래스의 인스턴스는 자동으로 부모 클래스에서 정의한 모든 인스턴스 변수를 내부에 포함하게 되는것.

```Java
Lecture lecture = new Lecture("객체지향 프로그래밍", 70, Arrays.asList(81, 95, 75, 50,45));
```
> Lecture의 인스턴스를 생성한것이다.
```Java
Lecture lecture = new GradeLecture("객체지향 프로그래밍",
                                    70, 
                                    Arrays.asList(new Grade("A", 100, 95),
                                                  new Grade("B", 94, 80),
                                                  new Grade("C", 79, 70),
                                                  new Grade("D", 69, 50),
                                                  new Grade("F", 49, 0)),
                                    Arrays.asList(81, 95, 75, 50, 45));
```
> 이번에는 GradeLecture의 인스턴스를 생성한것이다. 이를 데이터 관점에서 표현하자면
![001](https://user-images.githubusercontent.com/50142323/132384731-3a01535c-0690-41a3-a743-f86215fe9839.png)
> 결국 자식인 GradeLecture클래스의 인스턴스는 자동으로 부모인 Lecture클래스의 인스턴스변수를 내부에 포함하게 되는것이다.

### 02-2. 행동 관점의 상속
 - 행동 관점의 상속은 부모 클래스가 정의한 일부 메서드를 자식 클래스의 메서드로 포함시키는 것을 의미한다.
 - 공통적으로 부모 클래스의 모든 퍼블릭 메서드는 자식 클래스의 퍼블릭 인터페이스에 포함된다.

## 03. 업캐스팅과 동적 바인딩
 - 부모 클래스 타입으로 선언된 변수에서 자식 클래스의 인스턴스를 할당하는 것을 업캐스팅이라 한다
 - 선언된 변수 타입이 아니라도 메시지를 수신하는 객체의 타입에 따라 실행되는 메서드가 결정된다. 이것은 컴파일 시점이 아니라 실행시점에 결정하기 때문인데, 이를 동적 바인딩이라 한다

### 03-1. 업캐스팅
 - 상속을 이용하면 부모 클래스의 퍼블릭 인터페이스가 자식 클래스의 퍼빌릭 인터페이스에 합쳐지기 때문에 메시지를 자식 클래스의 인스턴스에게 전송할 수 있다.
 - 컴파일러는 명시적인 타입 변환 없이도 자식 클래스가 부모 클래스를 대체할 수 있게 허용한다.
```Java
Lecture lecture = new GradeLecture();
```
> 부모 클래스 타입으로 선언된 파라미터에 자식 클래스 인스턴스를 전달하는 것이 가능하다.
```Java
public class Professor {
  public Professor(String name, Lecture lecture) {...}
}

Professor professor = new Professor("교수", new GradeLecture());
```
>반대로 부모 클래스의 인스턴스를 자식 클래스 타입으로 변환하기 위해서는 명시적인 타입 캐스팅이 필요하다. 이를 다운캐스팅(downcasting) 이라고 한다.
```Java
Lecture lecture = new GradeLecture();
GradeLecture gradeLecture = (GradeLecture) lecture;    //다운캐스팅
```

### 03-2. 동적 바인딩
전통적인 언어에서 함수를 실행하는 방법은 함수를 호출하는 것이다.
객체지향 언어에서 메서드를 실행하는 방법은 메시지를 전송하는 것이다.

 - 함수 호출
   * 코드를 작성하는 시점에 호출될 코드가 결정된다.
   * 컴파일 타임에 호출할 함수를 결정하는 방식을 정적 바인딩(static binding), 초기 바인딩(early binding), 컴파일타임 바인딩(compile-time binding) 이라고 부른다.
 - 메서드 호출
   * 메시지를 수신했을 때 실행될 메서드가 런타임에 결정된다.
   * 실행될 메서드를 런타임에 결정하는 방식을 동적 바인딩(dynamic binding), 지연 바인딩(late binding) 이라고 부른다.
   * 실행 시점에 어떤 클래스의 인스턴스에 메시지를 전달하는지 알아야 실제 실행되는 메서드를 알 수 있다.


