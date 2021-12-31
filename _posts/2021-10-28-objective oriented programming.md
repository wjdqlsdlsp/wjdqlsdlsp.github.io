---
layout: post
title:  "Object-Oriented Programming in Python"
date:   2021-10-28
categories: 데이터엔지니어
tags: 데이터엔지니어
image: /post_img/oop.jpg
---

# Object-Oriented Programming in Python



## OOP Fundamentals

OOP : Object-Oriented-Programming

Object  = state + behavior

개체는 상태 및 동작에 대한 정보를 통합하는 데이터 구조



OOP의 특징은 상태와 동작이 함께 묶여 있다는 것입니다.


<br>
#### Objects in Python

- 모든 것이 object
- 모든 object는 class를 가진다.



| Object         | Class     |
| -------------- | --------- |
| 5              | int       |
| "Hello"        | str       |
| pd.DataFrame() | DataFrame |
| np.mean        | function  |



이러한 통합 인터페이스의 존재는 모든 DataFrame을 동일한 방식으로 사용 할 수 있는 이유

<br>

- `type()` 을 사용해서 해당 object의 class를 찾을 수 있다.



```python
import numpy as np
a = np.array([1,2,3,4])
print(type(a))

[output]
numpy.ndarray
```

<br>


#### Attributes and methods

State <-> attributes

```python
import numpy as np
a = np.array([1,2,3,4])
# shape 라는 함수는 attribute에 해당
a.shape # () 를 사용하지 않음
```

Behavior <->methods

```python
import numpy as np
a = np.array([1,2,3,4])
# reshape 라는 함수는 method에 해당
a.reshape() # ()를 사용
```



Objects = attributes + methods

- attributes <-> variables <-> obj.my_attribute
- method <-> function() <-> obj.my_method()

```python
import numpy as np
a = np.array([1,2,3,4])
dir(a) # list all attributes and methods

[output]
['T',
 '__abs__',
 '__add__',
 '__and__',
 '__array__',
 ...]
```

<br>


### Class anatomy : attributes and methods



기본적인 Class 형식은 다음과 같습니다.

```python
class Customer:
    # code for class goes here
    pass

c1 = Customer()
c2 = Customer()
```

class <name>  : 클래스의 이름은 대문자로 시작합니다.

ClassName() 을통해서 해당 ClassName의 클래스를 생성합니다.



```python
class Customer:
    
    def identify(self, name):
        print("I am Customer " + name)
        
cust = Customer()
cust.identify("Laura")

[output]
I am Customer Laura
```

method를 정의하는 것은 다음과 같습니다. 일반적으로 함수꼴과 같고, 첫번째 인자에 self가 들어가고, 이후부터 다른 인자가 들어올 수 있습니다.

Customer 클래스를 생성하고, 사용하고자 하는 함수의 이름을 사용하면, 함수가 실행됩니다. 이때, self 는 무시합니다.

<br>


#### What is self?

위에서 사용하고, 무시하는 self는 무엇일까요?

`self` 는 정의한 클래스에게 현재 클래스를 알려주는 용도입니다. 그렇기 때문에, 모든 method에는 self 인자가 있어야합니다.

파이썬은 점 구문을 사용하여 객체에서 method가 호출 될 때 self 를 처리합니다.

실제로 `cust.identify("Laura")` 는 `Customer.identify(cust, "Laura")` 와 동일합니다.

그렇기 때문에, 기존 객체에서 method를 호출 할 때 명시 적으로 self를 지정하지 않습니다.

<br>


#### We need attributes

- Encapsulation(캡슐화) : OOP의 원칙에 따라, 개체의 상태를 설명하는 데이터는 개체에 번들로 포함되어야 합니다.

  ```python
  class Customer:
      # set the name attribute of an object to new_name
      def set_name(self, new_name):
          # Create an attribute by assigning a value
          self.name = new_name # <--- will create .name when set_name is called
          
  cust = Customer()
  cust.set_name("Lara de Silva")
  print(cust.name)
  
  [output]
  Lara de Silva
  ```

다음과 같이, "name" 이라는 Customer 클래스의 속성을 만들기 위해서는, self.name 을 통해서 할당 받을 수 있습니다.

<br>


### Class anatomy: the `__init__ ` constructor

이전에는 class를 선언하고 함수를 통해서 변수를 선언했다면, 이번에는 `__init__()`을 통해서 간단하게 선언합니다.

```python
class Customer:
    def __init__(self, name):
        self.name = name
        print("The __init__ method was called")
        
cust = Customer("Lara de Silva")
print(cust.name)

[output]
The __init__ method was called
Lara de Silva
```

class를 생성하면, `__init__` 함수가 먼저 호출됩니다. 따라서, 다음 코드를 실행한다면, 함수를 따로 선언해서 name을 주지 않아도, class 생성동시에, 인자를 넣어 name을 초기화합니다.

<br>


```python
class Customer:
    def __init__(self, name, balance):
        self.name = name
        self.balance = balance
        print("The __init__ method was called")
        
cust = Customer("Lara de Silva", 1000)
print(cust.name)
print(cust.balance) 

[output]
The __init__ method was called
Lara de Silva
1000
```

다음과 같이, 인자로 여러개를 줄 수 도있습니다. 

<br>


```python
class Customer:
    def __init__(self, name, balance=0): # 다음과 같이 디폴트 값을 줄 수도 있음.
        self.name = name
        self.balance = balance
        print("The __init__ method was called")
        
cust = Customer("Lara de Silva") # 디폴트값이 있으므로, 따로 인자를 주지않으면, 디폴트 값으로 선언
print(cust.name)
print(cust.balance) 

[output]
The __init__ method was called
Lara de Silva
0
```

위의 코드는 default값을 설정함으로, 인자에 name에 해당하는 string 하나만 주고도 실행이 되었습니다.



지금까지 2가지의 방식을 통해 속성을 정의했는데, 가능하다면, 생성자 외부에서 속성을 정의하지 않도록 하는 것이 좋습니다. 클래스 정의는 수백 줄의 코드가 될 수 도 있으며, 우리는 그것을 찾기위해 모두 살펴 봐야 할 수 도 있습니다. (일종의 약속 ^^)

<br>


#### 좋은 class를 생성하는 법

- 속성은, `__init_()` 을통해서 한다. 

- class 이름을 설정할땐, CamelCase를 사용한다. (_(언더바) 같은 특수기호를 사용하는 것이 아니라, 대문자로 구분) 또한, 시작은 대문자로!

- 반대로, functions 이나 attributes를 설정할 땐, _ (언더바) 사용, 시작을 소문자로

- Keep self as self - 사실 self 대신 다른 변수를 사용해도 가능합니다. ( 그냥 `self` 쓰세요... )

  ```python
  class MyClass:
      # This works but isn't recommended
      def my_method(kitty, attr):
          kitty.attr = attr
  ```

- Use docstrings - help() 가 호출 될 때, 독스트링을 허용합니다.



<br>


## Inheritance and Polymorphism



#### Core principles of OOP

- Inheritance (상속)
- Polymorphism (다형성)
- Encapsulation (캡슐화)



데이터는 클래스간에 공유된다.

class 의 body 에서 class attributes 가 정의됩니다.

```python
class MyClass:
    # Define a class attribute
    CLASS_ATTR_NAME =attr_value
```

이렇게 선언할 경우, 클래스 안에서 전역 변수의 역할을 하는 클래스 속성이 생성됩니다.


<br>


```python
class Employee:
    # Define a class attribute
    MIN_SALARY = 30000
    def __init__(self, name, salary):
        self.name = name
        #Use class name to access class attribute
        if salary >= Employee.MIN_SALARY:
            self.salary = salary
        else:
            self.salary = Employee.MIN_SALARY
            
emp1 = Employee("TBD", 40000)
print(emp1.MIN_SALARY)

[output]
30000

emp2 = Employee("TBD", 60000)
print(emp2.MIN_SALARY)

[output]
30000
```

이렇게 MIN_SALARY 를 선언 할 경우, 클래스 내 함수에서 해당 값이 공유가 됩니다.

self 를 사용하지않고, ClassName.ATTR_NAME 을 사용한 것에 주의해주세요.



다음과 같은 경우에 주로 사용합니다.

- minimal/maximal 값을 설정할 때 사용
- pi 등의 특수 값을 설정할 때 사용

<br>


#### Class methods

```python
class MyClass:
    
    @classmethod
    def my_awesome_method(cls, args...):
    	# Do stuff here
        # Can't use any instance attributes :(
        
MyClass.my_awesome_methods(args...)
```

Class method는 instance-level에서 사용할 수 없기에, 데코레이터를 이용해야 한다.

<br>


```python
class Employee:
    # Define a class attribute
    MIN_SALARY = 30000
    def __init__(self, name, salary):
        self.name = name
        #Use class name to access class attribute
        if salary >= Employee.MIN_SALARY:
            self.salary = salary
        else:
            self.salary = Employee.MIN_SALARY
            
    @classmethod
    def from_file(cls, filename):
        with open(filename, "r") as f:
            name = f.readline()
        return cls(name)
    
emp = Employee.from_file("employee_data.txt")
type(emp)
```

`__init__()` 은 하나만 사용 가능하기 때문에, 초기화 하기 위해서는 Class method를 사용해야함.

데코레이터를 사용했을땐, self가 아니라 주로 cls를 사용

<br>


#### class inheritance

이미 많은 사람들이 훌륭한 코드를 공유해서 좋은모듈들이 많이 있습니다.

하지만 이 모듈이 우리가 필요한 형식과 다르다면 어떻게 해야할까요..?

이럴때 사용할 수 있는 것이 `Inheritance (상속)` 입니다.



```python
class MyChild(MyParent):
    # Do stuff here
```

상속을 하기 위해서는, 괄호 안에 상속받고 싶은 class의 이름을 입력해주면 됩니다.



```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    def withdraw(self, amount):
        self.balance -= amount
        
class SavinsAccount(BankAccount):
    pass

saving_acct = SavingsAccount(1000)
type(savings_acct)

[output]
__main__.SavingsAccount

savings_acct.balance

[output]
1000
```

<br>


#### Inheritance: "is-a" relationship

```python
savings_acct = SavingsAccount(1000) # 상속 받은 클래스
isinstance(savings_acct, SavingsAccount)
[True]

acct = BankAccount(500)
isinstance(acct, savings_acct)
[False]

isinstance(savings_acct, BankAccount)
[True]

isinstance(acct, BankAccount)
[True]
```

<br>


#### Customizing constructors

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    def withdraw(self, amount):
        self.balance -= amount
        
class SavinsAccount(BankAccount):
    def __init__(self, balance, interest_rate):
        # Call the parent constructor using ClassName.__init__()
        BanKAccount.__init__(self, balance) 
        self.interest_rate = interest_rate
        
acct = SavingsAccount(1000, 0.03)
acct.interest_rate
[output]
0.03
```

상속 받은 모델을 Customizing 하기 위해서 다음과 같이 하면됩니다.

상속하고 싶은 코드가 있을 때, 한번더 사용하지 않고, Class.methods를 통해 호출하면 동일하게 작동합니다. 이후 추가하고싶은 코드를 추가하면, 정상적으로 작동합니다.

<br>


#### Customizing functionality

```python
class CheckingAccount(BankAccount):
    def __init__(self, balance, limit):
      BanKAccount.__init__(self, content) 
      self.limit = limit
    def deposit(self, amount):
      self.balance += amount
    def withdraw(self, amount, fee=0):
      if fee <=self.limit:
        BankAccount.withdraw(self, amount-fee)
      else:
        BankAccount.withdraw(self, amount-self.limit)
```

생성자와 동일하게, 상속받은 클래스를 추가하여 깔끔하게 코드 구현이 가능합니다.

<br>


## Integrating with Standard Python

#### Object equality

```python
class Customer:
    def __init__(self, name, balance):
        self.name, self.balance = name, balance
        
customer1 = Customer("Maryam Azar", 3000)
customer2 = Customer("Maryam Azar", 3000)

customer1 == customer2
[output]
False
```

다음과 같이 같은 class를 이용해서 비교햇을 때 False가 출력

이유 : Python 의 저장방식에 따라, 두개의 주소가 다르게 저장

즉, Python은 객체가 생성되면, 해당 객체에 메모리 덩어리를 할당



```python
import numpy as np

arr1 = np.array([1,2,3])
arr2 = np.array([1,2,3])

arr1 == arr2
[output]
True
```

numpy나 pandas 등의 형식에서는 다음과 같이 True가 출력됩니다.

<br>


#### Overloading `__eq__()`

```python
class Customer:
    def __init__(self, id, name):
        self.id, self.name = id, name
    def __eq__(self, other):
        print("__eq__() is called")
        return (self.id == other.id) and (self.name == other.name)

customer1 = Customer(123, "Maryam Azar")
customer2 = Customer(123, "Maryam Azar")

customer1 == customer2
[output]
__eq__() is called
True
```

`__eq__()` 는 2개의 객체를 비교할 때 호출됩니다.

이때 `__eq__()` 는 관습에 따라, self, other를 인자로 사용합니다.

또한 항상 Boolean을 반환합니다.



이 외에도, Python class 비교 연산자는 다음과 같습니다.

| Operator | Method     |
| -------- | ---------- |
| `==`     | `__eq__()` |
| `!=`     | `__ne__()` |
| `>=`     | `__ge__()` |
| `<=`     | `__le__()` |
| `>`      | `__gt__()` |
| `<`      | `__lt__()` |

`__hash__()` 객체를 사전 키와 세트로 사용 할 수 있는 메서드

<br>


#### Printing an object

```python
class Customer:
    def __init__(self, name, balance):
        self.name, self.balance = name, balance
cust = Customer("Maryam Azar", 3000)
print(cust)

[output]
<__main__.Customer at 0x12f3d2a322>

import numpy as np

arr = np.array([1,2,3])
print(arr)

[output]
[1 2 3]
```

numpy 및 pandas의 경우 출력 시 해당 값이 나오지만, class의 경우 주소가 출력



`__str__()`

- `print(obj)` , `str(obj)`

```python
print(np.array([1,2,3]))

[output]
[1 2 3]

str(np.array([1,2,3]))

[output]
[1 2 3]
```

<br>


`__repr__()`

- `repr(obj)`, printing in console

```python
repr(np.array([1,2,3]))

[output]
array([1,2,3])

np.array([1,2,3])

[output]
array([1,2,3])
```

두개의 차이점은, str은 비공식적인 최종 사용자에게 적합한 표현이며, repr은 주로 개발자가 사용

가장 좋은 방법은 repr을 사용하여 객체를 재현하는 데 사용할 수 있는 문자열을 인쇄하는 것입니다.



```python	
class Customer:
    def __init__(self, name, balance):
        self.name, self.balance = name, balance
        
    def __str__(self):
        cust_str ="""
        Customer:
        	name: {name}
        	balance: {balance}
        """.format(name = self.name, balance = self.balance)
        return cust_str
    
cust = Customer("Maryam Azar", 3000)
# Will implicitly call __str__()
print(cust)

[output]
Customer:
    name: Maryam Azar
    balance: 3000
```

`__str__()` 를 통해 구현하는 방법은 다음과 같습니다.

<br>


```python
class Customer:
    def __init__(self, name, balance):
        self.name, self.balance = name, balance
        
    def __repr__(self):
        # Notice the '...' around name
        return "Customer('{name}', {balance})".format(name = self.name, balance = self.balance)
    
cust = Customer("Maryam Azar", 3000)
cust

[output]
Customer('Maryam Azar', 3000)
```

`__repr__()` 는 `__str__()` 과 형식이 비슷합니다. `__repr__()` return 에는 작은 따옴표가 잇다는 것을 주목해야합니다.

<br>


#### Exception

`try` - `except` - `finally`:

```python
try:
    # Try running some code
except ExceptionNameHere:
    # Run this code if ExceptionNameHere happens
finally:
    # Run this code no matter what  
```

예외를 처리하는 코드는 다음과 같습니다. try를 하고, except에 해당하는 에러가 생겼을 때, except에 있는 코드가 실행됩니다.  이후 마지막으로, finally 코드가 실행됩니다.

<br>


#### Raising exceptions

- raise ExceptionNameHere('Error message here')

```python
def make_list_of_ones(length):
    if length <=0:
        raise ValueError("Invalid length!")
    return [1]*length

make_list_of_ones(-1)

[output]
Invalid length! 오류 출력
```

<br>


#### Custom exceptions

```python
class BalanceError(Exception): pass

class Customer:
    def __init__(self, name, balance):
        if balance < 0:
            raise BalanceError("Balance has to be non-negative!")
        else:
            self.name, self.balance = name, balance
            
cust = Customer("Larry Torres", -100)
[output]
raise 부분 오류 출력
```

다음과 같이 예외처리를 해주어야, 사용자는 무언가 잘못되었다는 명확한 신호를 얻을 수 있습니다.

<br>


## Best Practices of Class Design



Polymorphism(다형성) - 다형성은 통합 인터페이스를 사용하여 다른 클래스의 객체에서 작동하는 것을 의미



```python
def batch_withdraw(list_of_accounts, amount):
    for acct in list_of_accounts:
        acct.withdraw(amout)
        
b, c, s = BankAccount(1000), CheckingAccount(2000), SavingsAccount(3000)
batch_withdraw([b,c,s])
```

다음과 같이, 한번에 전체 계좌 목록에서 동일한 금액을 인출하는 기능을 정의했다고 가정 했을때, 

이것이, 예금인지, 알 지 못합니다.

따라서 해당 계좌에 따라서, 다음과 같이 다르게 처리할 수 있도록 다른 클래스를 생성해줍니다.

<br>


#### Liskov substitution principle ( LSP )

사용시기와 방법에 대한 기본적인 객체 지향 설계 원칙

- 기본클래스는, 주변 프로그램의 속성을 변경하지 않고 하위 클래스와 상호 교환 할 수 있다.

ex ) BankAccout 가 잘 작동하면, CheckingAccount도 잘 작동한다.

- Syntactically (구문적으로) - 부모 클래스의 메서드와 호환되는 매개 변수 및 반환 값

- Semantically (의미적으로) - 객체의 상태도 일관성을 유지해야함. , 하위 클래스 메서드는 더 강력한 것에 의존해서는 안됩니다. 또한 입력조건은 더 약한 출력 조건을 제공해서는 안되며 추가 예외를 발생시키지 말아야한다.


<br>

#### LSP 위반 

`BankAccount.withdraw()` 의 파라미터가 1개이고, `CheckingAccount.withdraw()` 의 파라미터가 2개일때, 문제가 발생합니다. 하지만, `CheckingAccount` 에만 있는 변수가 default값이 있다면 문제가 생기지 않습니다. 



`BankAccount.withdraw()` 는 어떤 값이든 상관없고, `CheckingAccount.withdraw()` 는 특정값 금액만 허용한다면 문제가 발생합니다.



`BankAccount.withdraw()` 는 남은 돈이양수인지 확인하고, 아니면 에러를 발생하지만, 하위클래스가 음수가 가능하다면,  하위 클래스를 사용할 수 없습니다.



이러한경우, 예외처리를 해야 사용 가능하지만, 상속하지 않는 것을 추천

`No LSP - No Inheritance`



#### Managing data access: private attributes



#### Restricting access

- 일부 이름 규칙을 사용 - 외부 소비 용이 아니라는 신호를 보냅니다.
- 커스텀마이즈를 허용하는 경우 `@property` 를 사용.
- 속성이 완전히 사용되는 방식을 변경하기 위해 재정의 할 수 있는 특수 메서드 사용 `__getattr__()` and `__setattr__()`

<br>


#### 이름 규칙

`obj._att_name` , `obj._method_name()` 

- `_` 로 시작해서, 변경할 수 없는 값이라고 가르쳐 줍니다.
- `_` 는 개발자가 변경하지 말아야한다고 말하는 방식



`obj._attr_name`, `obj.__method_name()`

- 이중 `__` 은, 다른 프로그래밍 언어의 필드와 메서드를 비공개하라는 것과 가장 가까운 말;
- 이는 데이터가 상속되지 않는 다는 것을 의미 ( 파이썬은 이중 밑줄로 시작하는 모든 이름은 자동으로 이름 앞에 추가 됩니다. )
- 충돌을 막기 위해서는, 무의식적으로 클래스에 이미 존재하는 이름을 도입하여 부모 메서드 또는 속성을 재정의해야합니다.



`__이중밑줄__` 

- 이중 밑줄의 경우, Python 내장 메서드의 이름입니다.

<br>


#### Properties

속성 엑세스를 제어하고, 유효성을 검사하거나 속성을 읽기 전용으로 만드는 방법

```python
class Employer:
    sef __init__(self, name, new_salary):
        self._salary = new_salary # _ 로 시작하는 것이 좋습니다.
    
    @property
    def salary(self):
        return self._salary
    
    @salary.setter
    def salary(self, new_salary):
        if new_salary <0:
            raise ValueError("Invalid salary")
        self._salary = new_salary
        
emp = Employee("Miriam Azari", 35000)
# accessing the "property"
emp.salary
[output]
35000

emp.salary = 60000
emp.salary = -1000
[output]
error
```

다음과 같이, @property를 통해 정의해야, 유효성을 검사하고 정상적으로 값을 수정할 수 있음.
