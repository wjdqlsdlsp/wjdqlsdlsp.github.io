---
layout: post
title:  "Writing Efficient Python Code"
date:   2021-09-14
categories: 데이터엔지니어
tags: 데이터엔지니어
image: /post_img/python-logo2.jpg
---
# 효율적인 파이썬 코드 쓰는 법

## 1. Foundations for efficiencies

### 효율적인 프로그래밍이란? 

- 빠르고, 실행과 결과 반환 사이의 대기 시간이 짧습니다.

- 리소스를 능숙하게 할당하고 불필요한 오버 헤드를 받지 않습니다.



빠른 런타임과 작은 메모리 사용량에 대한 정의는 작업에 따라 달라집니다.
효율적인 코드 작성의 목표는 지연 시간과 오버헤드를 모두 줄이는 것입니다.

<br>

### Python

파이썬은 코드 가독성을 자랑하는 언어입니다. 

파이썬 코드를 작성하는 것을 종종 Pythonic 코드라고도 합니다. (처음 들어봤네요..)

Pythonic 코드는 덜 정황하고 해석하기 쉬운 경향이 있습니다.

- Non - Pythonic 코드
```python
double_number = []
for i in range(len(numbers)):
    double_numbers.append(number[i] * 2)
```
- Pythonic 코드
```python
doubled_numbers = [x * 2 for x in numbers]
```

파이썬은 기본 원칙을 따르지않는 코드를 지원하지만 이러한 유형의 코드는 느리게 실행되는 경향이 있습니다.

몰랐던 사실 : 인라인 코드 쓰는이유가 단순히 보기 이뻐서 ( 가독성 ) 때문인 줄 알았는데, 직접해보니 속도차이가 엄청 나네요....

<br>

### 파이썬 철학

```python
import this

The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

<br>

### Python 표준 라이브러리

#### Type

- List
- Tuple
- Set
- Dict

<br>

#### Function

- print()
- len()
- range()
- round()
- enumerate()
- map()
- zip()
- and others

<br>

#### modules

- os
- sys
- itertools
- collections
- math
- and others

<br>

#### range

```python
nums = range(0,11)
nums_list = list(nums)
print(nums_list)

[output]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 2칸씩 건너뛰며
nums = range(2,11,2)
nums_list = list(nums)
print(nums_list)

[output]
[2, 4, 6, 8, 10]
```


<br>

#### enumerate()

```python
letters = ['a','b','c','d']
indexed_letters = enumerate(letters)
print(indexed_letters)

[output]
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd')]

# start index 설정
letters = ['a','b','c','d']
indexed_letters = enumerate(letters, start = 5)
indexed_letters = list(indexed_letters)
print(indexed_letters)

[output]
[(5, 'a'), (6, 'b'), (7, 'c'), (8, 'd')]
```


<br>

#### map()

```python
nums = [1.5, 2.3, 3.4, 4.6, 5.0]
rnd_nums = map(round, nums)
print(list(rnd_nums))

[output]
[2, 2, 3, 5, 5]

# lambda 와 같이 사용
nums = [1,2,3,4,5]

sqrd_nums = map(lambda x : x ** 2, nums)

print(list(sqrd_nums))

[output]
[1, 4, 9, 16, 25]
```
<br>

#### Practice

```python
# Create a new list of odd numbers from 1 to 11 by unpacking a range object
nums_list2 = [*range(1,12,2)]
print(nums_list2)

[output]
[1, 3, 5, 7, 9, 11]
```

매번 `list(range(1,12,2))` 만 사용했는데, 해보니까 `[*range(1,12,2)]`가 더 빠르고 깔끔하네요



### Numpy

```python
import numpy as np

nums_np = np.array(range(5))
print(nums_np)

[output]
[0 1 2 3 4]
```
```python
print(nums_np.dtype)

[output]
int64
```
```python
# 실수와 정수 둘다 포함됬을 경우
nums_np_float = np.array([1,2.5,3])
print(nums_np_float)
print(nums_np_float.dtype)

[output]
[1.  2.5 3. ]
float64
```
두개 이상의 형식이 있을 땐, 이를 포함하는 가장큰 형식이용
```python
nums = [-2, -1, 0, 1, 2]
print(nums ** 2)

[output]
TypeError
```
Python list 같은 경우는 다음과 같은 연산이 불가
```python
nums = [-2, -1, 0, 1, 2]
[*map(lambda x : x**2, nums)]

[output]
[4, 1, 0, 1, 4]
```
다음과 같이 계산하면 가능 하지만 비효율적
```python
# Numpy를 사용한다면?
nums = np.array([-2, -1, 0, 1, 2])
print(nums ** 2)

[output]
[4 1 0 1 4]
```
Numpy를 이용하면 간단하게 계산 가능
```python
# Numpy indexing
nums2 = [[1, 2, 3],
         [4, 5, 6]]

print(nums2[0][1])
print([row[0] for row in nums2]) # Numpy를 사용하지 않는다면

nums2 = np.array(nums2)

print(nums2[0,1])
print(nums2[:, 0])

[output]
2
[1 4]
```
```python
# boolean indexing
nums = [-2, -1, 0, 1, 2]
nums = np.array(nums)

print(nums > 0)
print(nums[nums > 0])

[output]
[False False False True True]
[1 2]
```

<br>



------

## 2. Timing and profiling code

<br>
### Using %timeit

```python
%timeit rand_nums = np.random.rand(1000)

[output]
The slowest run took 4.61 times longer than the fastest. This could mean that an intermediate result is being cached.
100000 loops, best of 5: 9.72 µs per loop
```

import time 선언 후 time.time() 만 사용했었는데, %timeit 사용하니 보다 자세히 표현되네용...

<br>

#### Specifying number of runs/loops

```python
%timeit -r2 -n10 rand_nums = np.random.rand(1000)

[output]
The slowest run took 5.82 times longer than the fastest. This could mean that an intermediate result is being cached.
10 loops, best of 2: 35.6 µs per loop
```

`-r 플래그` 를 이용하여 실행 수를 지정하고, `-n 플래그`를 사용하여 루프 수를 지정 할 수 있음

<br>

#### Using %timeit in line magic mode

```python
%%timeit
nums = []
for x in range(10):
    nums.append(x)

[output]
1000000 loops, best of 5: 1.24 µs per loop
```

<br>

#### Saving output

```python
times = %timeit -o rand_nums = np.random.rand(1000)

[output]
The slowest run took 11.79 times longer than the fastest. This could mean that an intermediate result is being cached.
100000 loops, best of 5: 9.87 µs per loop
    
# 이후 저장된 값을 출력하는법
times.timings
# - colab에서는 안되네요...?

times.best

[output]
9.871182670012786e-06

times.worst

[output]
0.00011633400026767049
```

<br>

#### Comparing times

```python
f_time = %timeit -o formal_dict = dict()

[output]
The slowest run took 17.28 times longer than the fastest. This could mean that an intermediate result is being cached.
10000000 loops, best of 5: 118 ns per loop
    
    
l_time = %timeit -o literal_dict = {}

[output]
The slowest run took 21.33 times longer than the fastest. This could mean that an intermediate result is being cached.
10000000 loops, best of 5: 53.5 ns per loop
    
# average 도 colab에서는 안되네요
diff = (f_time.compile_time - l_time.compile_time) * (10** 9)
print("l_time better than f_time by {} ns".format(diff))
```

<br>

#### Code profiling

코드 프로파일링이란, 프로그램의 다양한 부분이 실행되는 시간과 빈도를 설명하는 데 사용되는 기술

`pip install line_profiler` 표준라이브러리가 아니기 때문에, 따로 다운 받아야합니다.

```python
def convert_units(heros, heights, weights):
    new_hts = [ht * 0.39370 for ht in heights]
    new_wts = [wt * 2.20462 for wt in weights]

    hero_data = {}
    
    for i, hero in enumerate(heros):
        hero_data[hero] = (new_hts[i], new_wts[i])

    return hero_data

%load_ext line_profiler
heroes = ['Batman', 'Superman', 'Wonder Woman']
hts = np.array([188.0, 191.0, 183.0])
wts = np.array([95.0, 101.0, 74.0])

%lprun -f convert_units

[output]
Timer unit: 1e-06 s

Total time: 0.000159 s
File: <ipython-input-76-09e195816b85>
Function: convert_units at line 1

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     1                                           def convert_units(heroes, heights, weights):
     2         1        144.0    144.0     90.6      new_hts = [ht * 0.39370 for ht in heights]
     3         1          5.0      5.0      3.1      new_wts = [wt * 2.20462 for wt in weights]
     4                                           
     5         1          1.0      1.0      0.6      hero_data = {}
     6                                               
     7         4          4.0      1.0      2.5      for i, hero in enumerate(heroes):
     8         3          4.0      1.3      2.5          hero_data[hero] = (new_hts[i], new_wts[i])
     9                                           
    10         1          1.0      1.0      0.6      return hero_data
```

<br>

#### Quick and dirty approach

```python
import sys

nums_list = [*range(1000)]
sys.getsizeof(nums_list)

[output]
9120
```

`pip install memory_profiler` 표준라이브러리가 아니기 때문에, 따로 다운 받아야합니다.

```python
%load_ext memory_profiler

%mprun -f convert_units convert_units(heroes, hts, wts)
```

<br>

## 3. Efficiently combining, counting, and iterating
<br>


#### Zip

```python
names = ['Bulbasaur', 'Charmander', 'Squirtle']
hps = [45, 39, 44]

combined_zip = zip(names, hps)
print([*combined])

[output]
[('Bulbasaur', 45), ('Charmander', 39), ('Squirtle', 44)]
```


#### Collections

- namedtuple
- deque
- Counter
- OrderedDict
- defaultdict



#### Counter

```python
from collections import Counter

poke_tpyes = ['Grass', 'Dark', 'Fire', 'Fries']
Counter(poke_tpyes)

[output]
Counter({'Dark': 1, 'Fire': 1, 'Fries': 1, 'Grass': 1})
```



#### itertools

- product
- permutations
- combinations



#### Combinations

```python
from itertools import combinations

poke_tpyes = ['Grass', 'Dark', 'Fire', 'Fries','Ghost']
print([*combinations(poke_tpyes,2)])

[output]
[('Grass', 'Dark'), ('Grass', 'Fire'), ('Grass', 'Fries'), ('Grass', 'Ghost'), ('Dark', 'Fire'), ('Dark', 'Fries'), ('Dark', 'Ghost'), ('Fire', 'Fries'), ('Fire', 'Ghost'), ('Fries', 'Ghost')]
```



#### Set theory

- intersection()
- difference()
- symmetric_difference()
- union()



```python
list_a = ['Bulbasaur', 'Charmander', 'Squirtle']
list_b = ['Caterpie', 'Pidgey', 'Squirtle']

set_a = set(list_a)
print(set_a)

set_b = set(list_b)
print(set_b)

print(set_a.intersection(set_b)) # set_a & set_b
print(set_a.difference(set_b)) # set_a -set_b
print(set_b.difference(set_a)) # set_b -set_a
print(set_a.symmetric_difference(set_b)) # set_a ^ set_b
print(set_a.union(set_b)) # set_a | set_b

[output]
{'Squirtle', 'Charmander', 'Bulbasaur'}
{'Caterpie', 'Squirtle', 'Pidgey'}
{'Squirtle'}
{'Charmander', 'Bulbasaur'}
{'Caterpie', 'Pidgey'}
{'Caterpie', 'Bulbasaur', 'Charmander', 'Pidgey'}
{'Pidgey', 'Charmander', 'Caterpie', 'Squirtle', 'Bulbasaur'}
```

저는 주로 &, -, +, |, ^ 이런 기호사용하는 것을 선호합니다.

```python
names_list = [*range(1000)]
names_tuple = ([*range(1000)])
names_set = set([*range(1000)])

%timeit 200 in names_list

[output]
100000 loops, best of 5: 3.12 µs per loop
    
%timeit 200 in names_tuple

[output]
The slowest run took 15.73 times longer than the fastest. This could mean that an intermediate result is being cached.
100000 loops, best of 5: 3.11 µs per loop
    
%timeit 200 in names_set

[output]
The slowest run took 27.37 times longer than the fastest. This could mean that an intermediate result is being cached.
10000000 loops, best of 5: 57.5 ns per loop
```



```python
primary_types = [1,2,3,1,2,3]
unique_type = set(primary_types)
print(unique_type)

[output]
{1, 2, 3}
```
중복처리하려고 자주사용합니다.


#### Looping in Python

- for
- while
- "nested"



> 플랫이 중첩보다 낫다



```python
poke_stats = [[90, 92, 75, 60],
              [25, 20, 15, 90],
              [65, 130, 60, 75]]

totals = []
for row in poke_stats:
    totals.append(sum(row))
----------------------------------
totals_comp = [sum(row) for row in poke_stats]
----------------------------------
totals_map = [*map(sum, poke_stats)]


[output]
[317, 150, 330]
```



```python
%%timeit
totals = []
for row in poke_stats:
    totals.append(sum(row))
    
[output]
The slowest run took 4.79 times longer than the fastest. This could mean that an intermediate result is being cached.
1000000 loops, best of 5: 776 ns per loop
    
    
%timeit totals_comp = [sum(row) for row in poke_stats]

[output]
The slowest run took 6.80 times longer than the fastest. This could mean that an intermediate result is being cached.
1000000 loops, best of 5: 765 ns per loop
    

%timeit totals_map = [*map(sum, poke_stats)]

[output]
The slowest run took 7.63 times longer than the fastest. This could mean that an intermediate result is being cached.
1000000 loops, best of 5: 663 ns per loop
```

maps를 이용하는 방법이 한줄로 깔끔하면서, 속도도 제일 빠름!



#### Numpy를 이용한 방법

```python
avgs = []
for row in poke_stats:
    avg = np.mean(row)
    avgs.append(avg)
print(avgs)
------------------------
# Numpy 사용
avgs_np = np.array(poke_stats).mean(axis=1)
print(avgs_np)

[output]
[79.25 37.5  82.5 ]
--------------------------
%%timeit
avgs = []
for row in poke_stats:
    avg = np.mean(row)
    avgs.append(avg)
    
[output]
10000 loops, best of 5: 34.2 µs per loop
-------------------------
%timeit avgs_np = np.array(poke_stats).mean(axis=1)

[output]
The slowest run took 14.21 times longer than the fastest. This could mean that an intermediate result is being cached.
100000 loops, best of 5: 13.5 µs per loop
```



#### Wrtting better loops
<br>
가능한, loop문에서 실행되지 않아도 되는 코드는 외부로 뺴라!

------



## 4. Basic pandas optimizations
<br>

```python
# 비효율적인 방법 !
%%timeit
win_perc_list = []

for i in range(len(baseball_df)):
    row = baseball_df.iloc[i]
    wins = row['W']
    games_played = row['G']
    win_perc = calc_win_perc(wins, games_played)
    win_perc_list.append(win_perc)

baseball_df['WP'] = win_perc_list

[output]
1 loop, best of 5: 226 ms per loop
```
#### iterrows()

```python
# iterrows() 이용! - enumerate랑 비슷하네요
%%timeit
win_perc_list = []
for i, row in baseball_df.iterrows():
    wins = row['W']
    games_played = row['G']
    win_perc = calc_win_perc(wins, games_played)
    win_perc_list.append(win_perc)

baseball_df['WP'] = win_perc_list

[output]
10 loops, best of 5: 128 ms per loop
```



iterrows() 를이용하는 방법이 이용하지 않는 것 보다 약 2배정도 빠름



#### itertuples()


collections 모듈 내 특수 데이터  유형을 반환합니다. 데이터 유형은 Python의 튜플처럼 작동하지만, 속성 조회를 사용하여 엑세스 할 수 있는 필드가 있습니다.

```python
%%timeit
for row_nametuple in baseball_df.itertuples():
    print(row_nametuple)
    
[output]
10 loops, best of 5: 93.1 ms per loop
-----------------------------------------------
%%timeit
for row_nametuple in baseball_df.iterrows():
    print(row_nametuple)
    
    
[output]
1 loop, best of 5: 1.26 s per loop
```

iterrows() 와 비교했을 때, 굉장히 빠른 속도를 보여줍니다.



#### apply()

```python
%timeit baseball_df.apply(lambda row: calc_win_perc(row['RS'], row['RA']), axis = 1)

[output]
10 loops, best of 5: 30.5 ms per loop
```

#### values

```python
wins_np = baseball_df['W']
print(type(wins_np))

[output]
<class 'pandas.core.series.Series'>
```
```python
wins_np = baseball_df['W'].values
print(type(wins_np))

[output]
<class 'numpy.ndarray'>
```

> values하면 numpy형식인거 처음알았네용..



```python
baseball_df['RS'].values -  baseball_df['RA'].values

[output]
array([  46,  100,    7, ...,  188,  110, -117])
```

numpy 형식이기 때문에 한번에 계산 가능



```python
%timeit baseball_df['RS'].values -  baseball_df['RA'].values

[output]
The slowest run took 11.61 times longer than the fastest. This could mean that an intermediate result is being cached.
100000 loops, best of 5: 7.83 µs per loop
    
    
%timeit baseball_df['RS'] - baseball_df['RA']

[output]
10000 loops, best of 5: 201 µs per loop
```

