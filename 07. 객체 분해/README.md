# 객체 분해
 - 추상화: 불필요한 정보를 제거하고 현재의 문제 해결에 필요한 핵심만 남겨놓자
 - 분해(decomposition): 큰 문제를 해결 가능한 작은 문제로 나누는 작업
 

## 01. 프로시저 추상화와 데이터 추상화
프로그래밍 패러다임은 프로그래밍을 구성하기 위해 사용하는 추상화의 종류와 이 추상화를 이용해 소프트웨어를 분해하는 방법의 두 가지 요소로 결정된다.  
#### 현대적인 프로그래밍 언어를 특징 짓는 중요한 두 가지 추상화 메커니즘
 - 프로시저 추상화(procedure abstraction): 소프트웨어가 무엇을 해야하는지를 추상화한다. -> 기능 분해 또는 알고리즘 분해
 - 데이터 추상화(data abstraction): 무엇을 알아야 하는지를 추상화한다. -> -> 1. 타입을 추상화 (추상 데이터 타입) 2. 프로시저를 추상화 (객체 지향)

## 02. 프로시저 추상화와 기능 분해
### 메인 함수로서의 시스템
기능분해(알고리즘 분해)로 추상화.  
전통적인 기능 분해 방법은 **하향식 접근법(Top-Down Approach)**

### 급여 관리 시스템
#### 급여는 다음 공식에 따라 계산된다
> 급여 = 기본급 - (기본급 * 소득세율)

#### 기능분해를 통해 분해하자
```
 직원의 급여를 계산한다.
  사용자로부터 소득세율을 입력받는다.
   "세율을 입력하세요: "라는 문장을 화면에 출력한다.
   키보드를 통해 세율을 입력받는다.
  직원의 급여를 계산한다.
   전역 변수에 저장된 직원의 기본급 정보를 얻는다.
   급여를 계산한다.
  양식에 맞게 결과를 출력한다.
   "이름: {직원명}, 급여: {계산된 금액}" 형식에 따라 출력 문자열을 생성한다.
```
![KakaoTalk_20210724_152551476](https://user-images.githubusercontent.com/60125719/126859720-79fb9de1-2557-45dd-bb44-9dd76883c044.jpg)

### 급여 관리 시스템 구현

#### step1
```
//직원의 급여를 계산한다.

def main(name)
end
```

#### step2
```
/*
 직원의 급여를 계산한다.
  사용자로부터 소득세율을 입력받는다.
  
  직원의 급여를 계산한다.
  
  양식에 맞게 결과를 출력한다.
  
*/

def main(name)
 taxRate = getTaxRate()
 pay = calculatePayFor(name, taxRate)
 puts(describeResult(name, pay))
end
```

#### step3
```
/*
  사용자로부터 소득세율을 입력받는다.
   "세율을 입력하세요: "라는 문장을 화면에 출력한다.
   키보드를 통해 세율을 입력받는다.
*/

def getTaxRate()
 print("세율을 입력하세요: ")
 return gets().chomp().to_f()
end
```

#### step4
```
/*
 직원의 급여를 계산한다.
  전역 변수에 저장된 직원의 기본급 정보를 얻는다.
  급여를 계산한다.
*/

$employees = ["직원A", "직원B", "직원C"]
$basePays = [400, 300, 250]

def calculatePayFor(name, taxRate)
 index = $employees.index(name)
 basePay = $basePays[index]
 return basePay - (basePay * taxRate)
end
```

#### step5
```
/*
 양식에 맞게 결과를 출력한다.
   "이름: {직원명}, 급여: {계산된 금액}" 형식에 따라 출력 문자열을 생성한다.
*/

def describeResult(name, pay)
 return "이름: ${name}, 급여: #{pay}"
end
```

#### Result
```
def main(name)
 taxRate = getTaxRate()
 pay = calculatePayFor(name, taxRate)
 puts(describeResult(name, pay))
end

def getTaxRate()
 print("세율을 입력하세요: ")
 return gets().chomp().to_f()
end

$employees = ["직원A", "직원B", "직원C"]
$basePays = [400, 300, 250]

def calculatePayFor(name, taxRate)
 index = $employees.index(name)
 basePay = $basePays[index]
 return basePay - (basePay * taxRate)
end

def describeResult(name, pay)
 return "이름: ${name}, 급여: #{pay}"
end
```

#### How to use
```
main("직원C")
```
![KakaoTalk_20210724_153813641](https://user-images.githubusercontent.com/60125719/126859983-b991848d-1745-4e8b-be90-e844e3f07b2a.jpg)

### 하향식 기능 분해의 문제점
 - 시스템은 하나의 메인 함수로 구성돼 있지 않다.
 - 기능 추가나 요구사항 변경으로 인해 메인 함수를 빈번하게 수정해야 한다.
 - 비즈니스 로직이 사용자 인터페이스와 강하게 결합된다.
 - 하향식 분해는 너무 이른 시기에 함수들의 실행 순서를 고정시키기 때문에 유연성과 재사용성이 저하된다.
 - 데이터 형식이 변경될 경우 파급효과를 예측할 수 없다.

#### 하나의 메인 함수라는 비현실적인 아이디어
어떤 시스템도 최초에 릴리즈됐던 당시의 모습을 그대로 유지하지는 않고 항상 기능을 추가하게 된다. 이것은 시스템이 오직 하나의 메인 함수만으로 구현된다는 개념과 부합하지 않는다.  
어느 시점에 이르면 유일한 메인 함수라는 개념은 의미가 없어지고 시스템은 여러 개의 동등한 수준의 함수 집합으로 변경될 것이다.  
실제 시스템에 정상(top)이란 존재하지 않는다.

#### 메인 함수의 빈번한 재설계
새로운 기능을 추가할 때마다 메인 함수를 수정해야 한다.  

ex) 급여 관리 시스템에 회사에 속한 모든 직원들의 기본급의 총합을 구하는 기능을 추가해보자
```
def sumOfBasePays()
 result = 0
 for basePay in $basePays
  result += basePay
 end
 puts(result)
end

def main(name)
 taxRate = getTaxRate()
 pay = calculatePayFor(name, taxRate)
 puts(describeResult(name, pay))
end
```
> main 함수는 직원 각각의 급여를 계산하는 함수이므로 sumOfBasePays는 쓸 수 없다. 그렇다면??

```
def sumOfBasePays()
 result = 0
 for basePay in $basePays
  result += basePay
 end
 puts(result)
end

def calculatePay(name)
 taxRate = getTaxRate()
 pay = calculatePayFor(name, taxRate)
 puts(describeResult(name, pay))
end

def main(peration, args= {})
 case(peration)
 when :pay then calculatePay(args[:name])
 when :basePays then sumOfBasePays()
 end
end
```

사용법
```
main(:basePays) // 기본급의 총 합 구하기

main(:pay, name: "직원A") // 급여 계산하기
```
> 결국 main함수를 수정할 수 밖에 없는 결과가 된다.

#### 비즈니스 로직과 사용자 인터페이스의 결합
하향식 접근법은 비즈니스 로직을 설계하는 초기 단계부터 입력 방법과 출력 양식을 함께 고민하도록 강요한다. -> 코드 안에서 비즈니스 로직과 사용자 인터페이스 로직이 밀접하게 결합된다.  
문제는 비즈니스 로직과 사용자 인터페이스가 변경되는 빈도가 다르다는것. -> 하향식 접근법은 근본적으로 변경에 불안정한 아키텍처를 낳는다.

#### 성급하게 결정된 실행 순서
하향식으로 기능을 분해하는 과정은 설계를 시작하는 시점부터 시스템이 무엇(what)을 해야 하는지가 아니라 어떻게(how) 동작해야하는지 집중하도록 만든다.  
하향식 접근법의 설계는 처음부터 구현을 염두에 두기 때문에 자연스럽게 함수들의 실행 순서를 정의하는 시간 제약을 강조한다. -> 기능 분해 방식은 중앙집중 제어 스타일의 형태를 가지게 된다. 결과 적으로 모든 중요한 제어 흐름의 결정이 상위 함수에서 이뤄지고 하위 함수는 상위 함수의 흐름에 따라 적절한 시점에 호출된다.  

-> 문제는 중요한 설계결정사항인 함수의 제어구조가 빈번한 변경의 대상임.  
-> 해결 방법은? 시간적인 제약을 버리고 논리적인 제약을 설계의 기준으로 삼는것.  
-> 하향식 설계와 관련된 모든 문제의 원인은 **결합도**임. 함수는 상위 함수가 강요하는 문맥에 강하게 결합된다.

#### 데이터 변경으로 인한 파급효과
하향식 기능 분해의 가장 큰 문제점은 어떤 데이터를 어떤 함수가 사용하고 있는지를 추적하기 어렵다는것. -> 의존성과 결합도의 문제. 테스트의 문제  

##### ex) 정규 직원의 급여뿐만 아니라 아르바이트 직원에 대한 급여 역시 개발된 급여 관리 시스템을 이용해 계산할 수 있게 변경해주세요 라는 요청이 들어온다면?
```
$employees = ["직원A", "직원B", "직원C", "아르바이트D", "아르바이트E", "아르바이트F"]
$basePays = [400, 300, 250, 1, 1, 1.5]
$hourlys = [false, false, false, true, true, true]
$timeCards = [0, 0, 0, 120, 120, 120]

def calculateHourlyPayFor(name, taxRate)
 index = $employees.index(name)
 basePay = $basePays[index] * $timeCards[index]
 return basePay - (basePay *taxRate)
end

def hourly?(name)
 return $hourlys[$employees.index(name)]
end

def calculatePay(name)
 taxRate = getTaxRate()
 if (hourly?(name)) then
  pay = calculateHourlyPayFor(name, taxRate)
 else 
  pay = calculatePayFor(name, taxRate)
 end
 puts(describeResult(name, pay))
end

def sumOfBasePays()
 result = 0
 for name in $employees
  if (not hourly?(name)) then
   result += $basePays[$employees.index(name)]
  end
 end
 puts(result)
end
```
> 데이터 변경으로 인해 발생하는 함수에 대한 영향도를 파악하는것은 생각보다 쉽지 않다.
> 데이터 변경으로 인한 영향을 최소화하려면 데이터와 함께 변경되는 부분과 그렇지 않은 부분을 명확하게 분리해야한다. -> 이게 핵심

### 언제 하향식 분해가 유용한가?
작은 프로그램과 개별 알고리즘을 위해서, 특히 프로그래밍 과정에서 이미 해결된 알고리즘을 문서화하고 서술하는 데는 훌륭한 기법이다.

## 03. 모듈
### 정보 은닉과 모듈
함께 변경되는 부분을 하나의 구현 단위로 묶고 퍼블릭 인터페이스를 통해서만 접근하도록 만들어보자. 즉, 기능을 기반으로 시스템을 분해하는 것이 아니라 변경의 방향에 맞춰 시스템을 분해해보자.  
모듈은 다음과 같은 두 가지 비밀을 감춰야 한다.
 - 복잡성: 모듈이 너무 복잡한 경우 이해하고 사용하기가 어렵다. 외부에 모듈을 추상화할 수 있는 간단한 인터페이스를 제공해서 모듈의 복잡도를 낮춘다.
 - 변경 가능성: 변경 가능한 설계 결정이 외부에 노출될 경우 실제로 변경이 발생했을 때 파급효과가 커진다. 변경 발생 시 하나의 모듈만 수정하면 되도록 변경 가능한 설계 결정을 모듈 내부로 감추고 외부에는 쉽게 변경되지 않을 인터페이스를 제공한다.

```
module Employees
  $employees = ["직원A", "직원B", "직원C", "아르바이트D", "아르바이트E", "아르바이트F"]
  $basePays = [400, 300, 250, 1, 1, 1.5]
  $hourlys = [false, false, false, true, true, true]
  $timeCards = [0, 0, 0, 120, 120, 120]

  def Employees.calculatePay(name, taxRate)
    if (Employees.hourly?(name)) then
      pay = Employees.calculateHourlyPayFor(name, taxRate)
    else
      pay = Employees.calculatePayFor(name, taxRate)
    end
  end

  def Employees.hourly?(name)
    return $hourlys[$employees.index(name)]
  end

  def Employees.calculateHourlyPayFor(name, taxRate)
    index = $employees.index(name)
    basePay = $basePays[index] * $timeCards[index]
    return basePay - (basePay * taxRate)
  end

  def Employees.calculatePayFor(name, taxRate)
    return basePay - (basePay * taxRate)
  end
  
  def Employees.sumOfBasePays()
    result = 0
    for name in $employees
      if (not Employees.hourly?(name)) then
        result += $basePays[$employees.index(name)]
      end
    end
    return result
  end
end


def main(operation, args={})
  case(operation)
  when :pay then calculatePay(args[:name])
  when :basePays then sumOfBasePays()
  end
end

def calculatePay(name)
  taxRate = getTaxRate()
  pay = Employees.calculatePay(name, taxRate)
  puts(describeResult(name, pay))
end

def getTaxRate()
  print("세율을 입력하세요: ")
  return gets().chomp().to_f()
end

def describeResult(name, pay)
  return "이름 : #{name}, 급여 : #{pay}"
end

def sumOfBasePays()
  puts(Employees.sumOfBasePays())
end

main(:basePays)
main(:pay, name:"아르바이트F")
```
> ex) 전체 직원에 관한 처리를 Employees 모듈로 캡슐화 한 결과

### 모듈의 장점과 한계
Employees 예제를 통해 알 수 있는 모듈의 장점은 다음과 같다.
 - 모듈 내부의 변수가 변경되더라도 모듈 내부에만 영향을 미친다.
 - 비즈니스 로직과 사용자 인터페이스에 대한 관심사를 분리한다.
 - 전역 변수와 전역 함수를 제거함으로써 네임스페이스 오염(namespace pollution)을 방지한다.


## 04. 데이터 추상화와 추상 데이터 
### 추상 데이터 타입
프로그래밍 언어에서 **타입(Type)** 이란 변수에 저장할 수 있는 내용물의 종류와 변수에 적용될 수 있는 연산의 가짓수를 의미한다.  
추상 데이터 타입을 구현하려면 다음과 같은 특성을 위한 프로그래밍 언어의 지원이 필요하다
 - 타입 정의를 선언할 수 있어야 한다.
 - 타입의 인스턴스를 다루기 위해 사용할 수 있는 오퍼레이션의 집함을 정의할 수 있어야 한다.
 - 제공된 오퍼레이션을 통해서만 조작할 수 있도록 데이터를 외부로부터 보호할 수 있어야 한다.
 - 타입에 대해 여러 개의 인스턴스를 생성할 수 있어야 한다.

이제 추상 데이터 타입을 이용해 급여 관리 시스템을 개선해보자.  

#### step1
```
Employees = Struct.new(:name, :basePay, :hourly, :timeCard) do
End
```

#### step2
```
Employees = Struct.new(:name, :basePay, :hourly, :timeCard) do
 def calculatePay(taxRate)
  if (hourly) then
   return calculateHourlyPay(taxRate)
  end
  return calculateSalariedPay(taxRate)
 end
 
private
 def calculateHourlyPay(taxRate)
  return (basePay * timeCard) - (basePay * timeCard) * taxRate
 end

def calculateSalariedPay(taxRate)
 return basePay - (basePay * taxRate)
end

End
```
> Employee 타입의 주된 행동은 직원 유형에 따라 급여를 계산하는 것이므로 calculatePay 오퍼레이션을 추가한다.

#### step3
```
Employee = Struct.new(:name, :basePay, :hourly, :timeCard) do
 def monthlyBasePay()
  if (hourly) then return 0 end
  return basePay
 end
end
```
> Employee 타입에 정의할 두 번째 오퍼레이션은 개별 직원의 기본급을 계산하는 것이다

#### step4
```
$employee = [
 Employee.new("직원A", 400, false, 0),
 Employee.new("직원A", 300, false, 0),
 Employee.new("직원C", 250, false, 0),
 Employee.new("아르바이트D", 1, true, 120),
 Employee.new("아르바이트E", 1, true, 120),
 Employee.new("아르바이트F", 1, true, 120)
]
```
> 추상 데이터 타입을 사용하는 클라이언트 코드를 작성하자.

#### step5

```
def calculatePay(name)
 taxRate = getTaxRate()
 for each in $employees
  if (each.name == name) then employee = each; break end
 end
 pay = employee.calculatePay(taxRate)
 puts(describeResult(name, pay)
end
```
> 특정 직원의 급여를 계산하는 것은 직원에 해당하는 Employee 인스턴스를 찾은 후 calculatePay 오퍼레이션을 호출하는 것이다.

#### step6
```
def sumOfBasePays()
 result = 0
 for each in $employees
  result += each.monthlyBasePay()
 end
 puts(result)
end
```
> 정규 직원 전체에 대한 기본급 총합을 구하기 위해서는 Employee 인스턴스 전체에 대해 차례로 monthlyBasePay 오퍼레이션을 호출한 후 반환된 값을 모두 더해야 한다.

추상 데이터 타입은 데이터에 대한 관점을 설계의 표면으로 끌어올리기는 하지만 여전히 데이터와 기능을 분리하는 절차적인 설계의 틀에 갇혀있다.  
Q 클래스는 추상 데이터 타입인가?





