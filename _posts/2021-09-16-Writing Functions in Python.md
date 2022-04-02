---
layout: post
title:  "Writing Functions in Python"
date:   2021-09-16
categories: 데이터엔지니어
tags: 데이터엔지니어
image: /post_img/python-logo2.jpg

---
# 파이썬 함수 만들기

## 1. Best Practices
<br>
#### Docstring
- 함수의 기능
- 인수
- 반환 값
- 발생하는 오류에 대한 정보
- 함수에 대해 말하고 싶은 모든 것

### Docstring formats

- Google Style
- Numpydoc 
- reStructedText
- EpyText


### Google Style

```python
def function(arg_1, arg_2 = 42):
    """Description of what the function does.
    
    Args:
    	arg_1 (str) : Description of arg_1 that can break onto the next line if needed.
    	arg_2 (int, optional) : Write optional when an argument has a default value.
    	
    Returns:
    	bool: Optional description of the return value
    	Extra lines are not indented.
    	
    Raises:
    	ValueError : Include any error types that the function intentionally raise
    	
    Notes:
    	See https://www.datacamp.com/community/tutorials/docstrings-python
    """
```



### Numpydoc

```python
def function(arg_1, arg_2=42):
    """
    Description of what the function does.
    
    Parameters
    ----------
    arg_1 : expected type of arg_1
    	Description of arg_1
    arg_2 : int, optional
    	Write optional when an argumenthas a default value.
    	Default=42.
    
    Returns
    -------
    The type of the return value
    	Can include a description of the return value.
    	Replace "Returns" with "Yields" if this function is a generators.
    """
```



### Retrieving docstrings

```python
def the_answer():
    """Return the answer to life, the universe, and everything.
    
    Returns:
    	int
    """
```



`function.__doc__` 를통해 docs확인 가능

```python
import inspect
docstring = inspect.getdoc(pd.read_csv)
```

#### Don't repeat yourself ( DRY )

```python
train = pd.read_csv("train.csv")
train_Y = train['labels'].values
train_X = train[col for in col in train.columns if col !='labels'].values
train_pca = PCA(n_components = 2).fit_transform(train_X)
plt.scatter(train_pac[:,0], train_pca[:,1])

val = pd.read_csv("val.csv")
val_Y = val['labels'].values
val_X = val[col for in col in val.columns if col !='labels'].values
val_pca = PCA(n_components = 2).fit_transform(val)
plt.scatter(val_pca[:,0], val_pca[:,1])

test = pd.read_csv("test.csv")
test_Y = test['labels'].values
test_X = test[col for in col in test.columns if col !='labels'].values
test_pca = PCA(n_components = 2).fit_transform(test)
plt.scatter(test_pac[:,0], test_pca[:,1])
```

위 코드를 보면 train, val, test 세개의 똑같은 코드가 반복

##### 반복했을 때 문제점

- 실수를 발견하기 어렵다.
- 변경해야 할 때, 여러곳에서 변경해야 한다.

```python
def load_and_plot(path):
    """Load a data set and plot the first two principal components.
    
    Args:
    	path (str): The location of a CSV file.
    	
    Returns:
    	tuple of ndarray: (features, labels)
    """
    data = pd.read_csv(path)
    y = data['label'].values
    X = data[col for col in data.columns if col != 'label'].values
    pca = PCA(n_components = 2).fit_transform(X)
    plt.scatter(pca[:,0], pca[:,1])
    return X, y

train_X, train_y = load_and_plot("train.csv")
val_X, val_y = load_and_plot("val.csv")
test_X, test_y = load_and_plot("test.csv")
```



#### Do One Thing

```python
def load_data(path):
    """Load a data set.
    
    Args:
    	path (str): The location of a CSV file.
    	
    Returns:
    	tuple of ndarray: (features, labels)
    """
    data = pd.read_csv(path)
    y = data['label'].values
    X = data[col for col in data.columns if col != 'label'].values
    return X, y
```

```python
def plot_data(X):
    """Plot the first two principal components of a matrix.
    
    Args:
    	X (numpy.ndarray): The data to plot.
    """
    pca = PCA(n_components=2).fit_transform(X)
    plt.scatter(pca[:,0], pca[:,1])
```

#### 함수를 나누면서 얻게 되는 장점

- 코드가 유연해진다. ( 둘중 한가지 일만 하고 싶을 때, 자유롭게 가능 )
- 이해하기 쉽다 + 테스트 및 디버그가 더 쉽다.



#### Pass by assignment

```python
a = [1,2,3]
b = a
a.append(4)
print(b)

[output]
[1,2,3,4]

b.append(5)
print(a)

[output]
[1,2,3,4,5]

a = 42
print(b)

[output]
[1,2,3,4,5]
```

---

## 2. Context Managers

```python
with <context-manager>(<args>) as <variable-name>:
    # Run your code here
    # This code is running "inside the context"
    
"This code runs after the context is removed"
```
```python
with open("my_file.txt") as my_file:
    text = my_file.read()
    length = len(text)
    
print('The file is {} characters long'. format(length))
```



```python
@contextlib.contextmanager
def my_context():
    #Add any set up code you need
    yield 
    # Add any teardown code you need
```
```python
import contextlib

@contextlib.contextmanager
def my_context():
    print("hello")
    yield 42
    print("goodbye")

with my_context() as foo:
    print("foo is {}".format(foo))
    
[output]
hello
foo is 42
goodbye
```



```python
@contextlib.contextmanager
def database(url):
	# set up database connection
    db = postgres.connect(url)
    yield db
    # tear down database connection
    db.disconnect()

url = "http://datacamp.com/data"
with database(url) as my_db:
    course_list = my_db.execute(
    'SELECT * FROM courses'
    )
```



#### Handling errors

```python
try:
    # code that might raise an error
except:
    # do something about the error
finally:
    # this code runs no matter what
    
-------------------------------------
def get_printer(ip):
    p = connect_to_printer(ip)
    try:
        yield
    finally:
        p.disconnect()
        print("disconnected from printer")
doc = {"text": "This is my text."}

with get_printer("10.0.34.111") as printer:
    printer.print_page(doc['text'])
```





## 3. Decorators
<br>

#### Functions as variables

```python
def my_function():
    print('Hello')
    
x = my_function
type(x)

[output]
<type 'function'>
----------------------
x()

[output]
Hello
----------------------
PrintMcPrintface = print
PrintMcPrintface('Python is awesome!')

[output]
Python is awesome!
```



#### List and dictionaries of functions

```python
list_of_functions = [my_function, open, print]
list_of_functions[2]('I am printing with an element of a list')

[output]
I am printing with an element of a list
----------------------------------------------------------------
dict_of_functions = {
    'func1' : my_function,
    'func2' : open,
    'func3' : print
}
dict_of_functions['func3']('I am printing with a value of dict!')
```



#### Referencing a function

```python
def my_function():
    return 42
x = my_function
my_function()

[output]
42
---------------------
my_function

[output]
<function __main__.my_function>
```



#### Functions as arguments

```python
def has_docstring(func):
    """Check to see if the function
    'func' has a docstring.

    Args:
        func (callable): A function.
    
    Returns:
        bool
    """
    return func.__doc__ is not None

def no():
    return 42

def yes():
    """Return the value 42
    """
    return 42

has_docstring(no)

[output]
False
------------------------------------
has_docstring(yes)

[output]
True
```



#### Defining a  function inside another function

```python
def foo(x, y):
    if x > 4 and x < 10 and y > 4 and y < 10:
        print(x*y)
-------------------------
def foo(x,y):
    def in_range(v):
        return v > 4 and v <10
   	if in_range(x) and in_range(y):
        print(x*y)
```



#### Functions as return values

```python
def get_function():
    def print_me(s):
        print(s)
    return print_me

new_func = get_function()
new_func('This is a sentence.')

[output]
This is a sentence.
```



#### Scope

```python
x = 7
y = 200
print(x)

[output]
7
-------------
def foo():
    x = 42
    print(x)
    print(y)
foo(x)

[output]
42
200
---------------
x = 7
def foo():
    global x
    x = 42
    print(x)
foo()
print(x)

[output]
42
42
-------------------
def foo():
  x = 10
  def bar():
  	nonlocal x
    x = 200
    print(x)
  bar()
  print(x)
foo()

[output]
200
200
```



#### Closures

```python
def foo():
    a = 5
    def bar():
        print(a)
    return bar

func = foo()

print(func())
print(type(func.__closure__))
print(len(func.__closure__))

[output]
5
<class 'tuple'>
1
```

#### Definitions - nested function

```python
# outer function
def parent():
    # nested function
    def child():
        pass
    return child
```



#### Definitions - nonlocal variables

```python
def parent(arg_1, arg_2):
    # From child()'s point of view,
    # 'value' and 'my_dict' are nonlocal variables,
    # as are 'arg_1' and 'arg_2'
    value =22
    my_dict = {'chocolate':'yummy'}
    
    def child():
        print(2 * value)
        print(my_dict['chocolate'])
        print(arg_1 + arg_2)
        
    return child
```



#### Closure : Nonlocal variables attached to a returned function.

```python
def parent(arg_1, arg_2):
    value = 22
    my_dict = {'chocolate':'yummy'}
    
    def child():
        print(2*value)
        print(my_dict['chocolate'])
        print(arg_1+arg_2)
        
    return child

new_function = parent(3,4)
print([cell.cell_contents for cell in new_function.__closure__])

[output]
[3, 4, {'chocolate': 'yummy'}, 22]
```



#### Decorators

```python
@double_args
def multiply(a,b):
    return a*b

multiply(1,5)
```



```python
def multiply(a,b):
    return a*b
def double_args(func):
    # Define a new function that we can modify
    def wrapper(a,b):
        # For now, just call the unmodified function
        return func(a*2,b*2)
    return wrapper

new_multiply = double_args(multiply)
new_multiply(1,5)

[output]
20
```

---



## 4. Real-world examples
<br>


#### Time a function

```python
import time

def timer(func):
    """A decorator that print show how long a function took to run."""
    # Define the wrapper function to return.
    def wrapper(*args, **kwargs):
        # When wrapper() is called, get the current time.
        t_start = time.time()
        # Call the decorated function and store the result.
        result = func(*args, **kwargs)
        # Get the total time it took to run, and print it
        t_total = time.time() - t_start
        print('{} took {}s'.format(func.__name__, t_total))
        return result
    return wrapper

@timer
def sleep_n_seconds(n):
    time.sleep(n)

sleep_n_seconds(5)

[output]
sleep_n_seconds took 5.005098819732666s
```





#### Decorators and metadata

```python
@timer
def sleep_n_seconds(n=10):
    """Pause processing for n seconds.
    Args"
        n (int): The number of seconds to pause for.
    """
    time.sleep(n)
print(sleep_n_seconds.__doc__)
print(sleep_n_seconds.__name__)

[output]
None
wrapper
```
```python
from functools import wraps
def timer(func):
    """A decorator that print show how long a function took to run."""

    @wraps(func)
    def wrapper(*args, **kwargs):
        t_start = time.time()
        result = func(*args, **kwargs)
        t_total = time.time() - t_start
        print('{} took {}s'.format(func.__name__, t_total))
        return result
     return wrapper

@timer
def sleep_n_seconds(n=10):
    """Pause processing for n seconds.

    Args"
        n (int): The number of seconds to pause for.
    """
    time.sleep(n)
print(sleep_n_seconds.__doc__)
print(sleep_n_seconds.__name__)

[output]
Pause processing for n seconds.

    Args"
        n (int): The number of seconds to pause for.
    
sleep_n_seconds
```
```python
# closures를 통해서도 가능하지만 다음 방법을 사용하면 쉬움
@timer
def sleep_n_seconds(n=10):
    """Pause processing for n seconds.

    Args"
        n (int): The number of seconds to pause for.
    """
    time.sleep(n)
print(sleep_n_seconds.__wrapped__)

[output]
<function sleep_n_seconds at 0x7f3245720cb0>
```



#### Decorators that take arguments

```python
def run_three_times(func):
    def wrapper(*args, **kwargs):
        for i in range(3):
            func(*args, **kwargs)
    return wrapper

@run_three_times
def print_sum(a, b):
    print(a + b)
print_sum(3,5)

[output]
8
8
8
```



#### run_n_times()

```python
def run_n_times(n):
    """Define and return a decorator"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            for i in range(n):
                func(*args, **kwargs)
        return wrapper
    return decorator

@run_n_times(3)
def print_sum(a, b):
    print(a + b)
print_sum(3,5)

run_three_times = run_n_times(3)
@run_three_times
def print_sum(a,b):
    print(a + b)
print_sum(1,3)

[output]
8
8
8
4
4
4
```



### Timeout(): a real world example

```python
import signal

def raise_timeout(*args, **kwargs):
    raise TimeoutError()
# When an "alarm" signal goes off, call raise_timeout()
signal.signal(signalnum = signal.SIGALRM, handler = raise_timeout)
# Set off an alarm in 5 seconds
signal.alarm(5)
# Cancel the alarm
signal.alarm(0)

```
```python
def timeout_in_5s(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        # Set an alarm for 5 seconds
        signal.alarm(5)
        try:
            # Call the decorated func
            return func(*args, **kwargs)
        finally:
            # Cancel alarm
            signal.alarm(0)
    return wrapper

@timeout_in_5s
def foo():
    time.sleep(10)
    print('foo!')
    
foo()

[output]
TimeoutError: 
```





