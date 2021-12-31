---
layout: post
title:  "Introduction to Airflow in Python"
date:   2021-11-18
categories: 데이터엔지니어
tags: 데이터엔지니어
image: /post_img/oop.jpg
---

# Introduction to Airflow in Python


## 1. Intro to Airflow

<br>

#### 데이터 엔지니어링

컨텍스트에 따라 많은 구체적인 정의가 있지만, 데이터이면의 일반적인 의미는 

**데이터와 관련된 모든 조치를 취하고 이를 안정적이고 반복 가능하며 유지 관리할 수 있는 프로세스를 만드는 작업**

<br>

#### Workflow

workflow는 주어진 데이터 엔지니어링 작업을 수행하기 위한 일련의 단계

- 파일 다운로드
- 데이터 복사
- 정보 필터링
- 데이터베이스 쓰기

이러한 작업들이 포함됩니다.



워크플로는 다양한 수준의 복잡성을 가집니다.

일부는 2~3 단계만 있을 수도 있고, 수백 개는 구성 될수도있습니다.

워크 플로의 복잡성은 전적으로 사용자의 요구에 따라 달라집니다.

<p align="center"><img src="/images/post_img/airflow1.PNG"></p>



#### Airflow

Airflow는 워크 플로를 프로그래밍하는 플랫폼입니다.

- 생성
- 예약
- 모니터링

이러한 작업을 포함합니다.



Airflow는 다양한 도구와 언어를 사용할 수 있지만, 실제 워크 플로 코드는 `Python`으로 작성됩니다.

Airflow는 워크플로우를 `DAG` : Directed Acyclic Graph 로 구현합니다.

Airflow는 코드, 명령 줄 또는 내장 웹 인터페이스를 통해 액세스하고 제어 할 수 있습니다.

<br>

#### Other workflow tools

- Luigi (spotify)
- SSIS (Microsoft)
- Bash scripting

<br>

#### DAGs

DAG는 Directed Acyclic Graph를 나타냅니다.

Airflow에서 이것은 워크 플로를 구성하는 작업 집합을 나타냅니다.

작업과 작업 간의 종속성으로 구성

DAG는 DAG에 대한 다양한 세부 정보로 생성 (이름, 시작일, 소유자, 이메일, 알림 옵션 등)

<br>

#### DAG code example

```python
etl_dat = DAG(
	dag_id='etl_pipeline',
	default_args={"start_date": "2020-01-08"}
)
```

<br>

#### Running a workflow in Airflow

여러 가지 방법중 가장 간단한 방법은 airflow run shell 명령을 사용하는 것입니다.

```python
airflow run <dag_id> <task_id> <start_date>

#example
airflow run example-tel download-file 2020-01-10
```

<br>

#### DAG?

특정 수학적 의미 외에도 DAG 또는 Directed Acyclic Graph에는 다음과 같은 속성이 있습니다.

Directed는 구성 요소 실행 간의 종속성 또는 순서를 나타내는 고유 한 흐름이 있음을 의미

이러한 종속성은 구성 요소 실행 순서를 지정하는 방법에 대한 도구에 컨텍스트를 제공

DAG는 또한 비순환 적이며, 반복되지 않습니다. (이는 이전 DAG를 다시 실행할 수 없음을 의미하는 것이 아니라 개별 구성 요소가 실행 당 한 번만 실행된다는 것을 의미합니다.)

이 경우 Graph는 구성 요소와 구성 요소 간의 관계를 나타냅니다.

DAG라는 용어는 Airflow뿐만 아니라 Apache Spark, Luigi 등의 데이터 엔지니어링에서도 자주 사용됩니다.

<br>

#### DAG in Airflow

Airflow 내에서 DAG는 Python으로 작성되지만, 다른 언어 또는 기술로 작성된 구성 요소를 사용 할 수 있습니다. 즉, **Python을 사용하여 DAG를 정의하지만,  Bash 스크립트, 기타 실행 파일, Spark 작업 등을 포함합니다.**



Airflow DAG는 작업자, 센서 등과 같이 실행할 구성 요소로 구성됩니다. Airflow은 일반적으로 작업이라고 합니다.



Airflow DAG에는 명시 적으로 또는 암시 적으로 정의 된 종속성이 포함됩니다.

이러한 종속성은 실행 순서를 정의하므로 Airflow가 워크 플로 내의 어떤 지점에서 실행되어야하는 구성 요소 예를 들어, 데이터베이스로 가져 오기 전에 파일을 서버로 복사 할 수 있습니다.

<br>

#### Define a DAG

```python
from airflow.models import DAG

from datetime import datetime
default_arguments = {
    'owner': 'jdoe',
    'email': 'jdoe@datacamp.com',
    'start_date': datetime(2020, 1, 20)
}

etl_dag = DAG( 'etl_workflow', default_args=default_arguments )
```

먼저 airflow.models 에서 DAG를 선언합니다.

가져온 후에는 기본 인수 사전을 만듭니다. (이러한 속성은 선택사항이지만, Airflow의 런타임 동작을 정의 할 수 있는 많은 기능을 제공)

마지막으로, DAG의 이름 인 etl을 사용하여 첫 번째 인수로 DAG 개체를 정의 

<br>

#### DAGs on the command line

Using airflow :  

- airflow command line program 에는 Airflow 실행의 다양한 측면을 처리하는 많은 하위 명령이 포함
- 하위 명령에 대한 도움말 및 설명을 보려면 `airflow -r` 명령을 사용
- 대부분의 하위명령은, DAGs와 연관
- `airflow list_dags` 를 사용하여 인식 된 모든 DAG를 볼 수 있습니다.

<br>

#### Command line vs Python

Airflow commnad line을 사용하는 경우와 Python 작성을 하는 경우 비교

Airflow:

- Airflow 프로세스를 시작하는 데 사용 
- 수동으로 DAG 도는 작업을 실행
- 로깅 정보를 검토

Python:

- 일반적으로 생성
-  실제 데이터 처리 - DAG 편집

<br>

#### DAGs view

<p align="center"><img src="/images/post_img/airflow2.PNG"></p>

이 페이지는 대부분의 시간을 소비하게 될 페이지입니다.

DAG는 사용 가능한 DAG / 워크 플로의 수에 대한 빠른 상태를 제공

Schedule는 일정 (날짜 또는 cron 형식)을 보여줍니다.

Owner는  소유자를 나타냅니다

Recent Tasks는 가장 최근에 실행 된 작업

Last Run은 마지막 시작된 작업

DAG Runs 마지막 3개의 DAG실행

Links는 오른쪽의 링크 영역을 통해 많은 DAG 특정보기에 빠르게 액세스 



<p align="center"><img src="/images/post_img/airflow3.PNG"></p>

DAG의 링크(이름)를 클릭하게 되면 상세 페이지로 접근

DAG 자체 정보에 대한 특정 액세스를 제공

코드의 작업 및 종속성을 보여주는 여러 정보보기 ( 그래프, 트리 및 코드 ), 작업기간, 작업시도, 타이밍, Gantt 차트보기 및 DAG에 대한 특정 세부 정보에 액세스

DAG를 시작하고, 뷰를 새로 고치고 원하는 경우 DAG를 삭제 가능

상세보기는 기본적으로 트리보기로 설정되어 특정 명명된 작업, 사용중인 연산자 및 작업간의 종속성

단어 앞의 원은 작업 / DAG상태를 나타냅니다.

특정 DAG의 경우 generate_random_number라는 작업이 하나 있습니다.



<p align="center"><img src="/images/post_img/airflow4.PNG"></p>

DAG 그래프보기는 작업 및 종속성을 차트 형식 - DAG의 흐름에 대한 또 다른 보기를 제공합니다. 

언제든지 사용중인 연산자와 작업 상태를 볼 수 있습니다. 

트리 및 그래프 보기는, 알고 싶은 내용에 따라 다른 정보를 제공 ( 더 자세한 정보를 얻으려면 DAG를 검사 할 때 둘 사이를 이동) 이 뷰에 대해 다시 generate_random_number라는 작업이 있음을 알 수 있습니다.

또한 이미지의 왼쪽 가운데에서 BashOperator 유형임을 알 수 있습니다.



<p align="center"><img src="/images/post_img/airflow5.PNG"></p>

 DAG 코드보기는, DAG를 구성하는 Python 코드의 복사본을 보여줍니다.

코드 보기를 사용하면, UI의 다양한 부분을 클릭하지 않고도, DAG를 정의하는 항목에 쉽게 액세스 할 수 있습니다. (단 코드보기는 읽기 전용)

Airflow를 사용하면서, 가장 적합한 도구를 결정 할 수 있습니다.



모든 DAG 코드 변경은, 실제 DAG 스크립트를 통해 수행해야합니다.

generate_random_number 태스크와 bash 명령 echo $ RANDOM을 실행합니다.

<p align="center"><img src="/images/post_img/airflow6.PNG"></p>

Browse 메뉴 옵션 아래의 로그 페이지는 Airflow를 사용하는 동안 문제 해결 및 감사 가능을 제공합니다.

여기에는 Airflow 웹 서버 시작, 그래프 또는 트리노드보기, 사용자 생성, DAG 시작 등이 있습니다.

Airflow를 사용할 때 로그를 자주 확인하여 포함 된 정보 유형과 Airflow 설치 뒤에서 일어나는 일에 대해 더 잘 알고 있습니다.

<br>

#### Web UI vs commnad line

기본 설정에 따라 Airflow 웹 UI or Command line 을 선택하는데,

웹 UI는 전체적으로 사용하기가 쉽습니다. Command line은 설정(SSH 등을 통해)에 따라 액세스가 더 간단 할 수 있습니다. 

<br>

## 2. Implementing Airflow DAGs

Airflow에서 가장 일반적인 작업은 Operators입니다.

<br>

#### Operators

Airflow는 workflow에서 단일 작업을 나타냅니다

이것은 명령 실행, 이메일 전송, Python 스크립트 실행 등 모든 유형의 작업이 될 수 있습니다.

일반적으로, Airflow 연산자는 독립적으로 실행됩니다. **즉, 작업을 완료하는데 필요한 리소스는 운영자 내에 포함되어 있습니다.**

일반적으로 Airflow 연산자는 서로 정보를 공유하지 않습니다. 이는 workflow를 단순화하고 Airflow가 가장 효율적인 방식으로 작업을 실행 할 수 있도록 하기 위한 것입니다. (Operators 간에 정보를 공유 할 수 있습니다.)

Airflow에는 다양한 작업을 수행하는 다양한 연산자가 포함되어 있습니다. 

```python
DummyOperator(task_id='example', dag=dag) # 문제 해결 또는 아직 구현되지 않은 작업을 위해사용
```

<br>

#### BashOperator

```python
BashOperator(
	task_id='bash_example',
	bash_command='echo "Example!"',
	dag=ml_dag)
```

```python
BashOperator(
	task_id='bash_script_example',
	bash_command='runcleanup.sh',
	dag=ml_dag)
```

BashOperator는 주어진 Bash명령 또는 스크립트를 실행합니다. 이 명령은 주어진 workflow 에서 의미가있는 Bash가 할 수 있는 거의 모든 것이 될 수 있습니다.

BashOperator에는 세 개의 인수가 필요합니다. 

- UI, bash 명령 및 해당 명령이 속한 DAG
- BashOperator는 나중에 자동으로 정리되는 임시 디렉토리에서 명령을 실행합니다.
- Bash에 대한 환경 변수를 지정할 수 있습니다. 명령을 실행하여 로컬 시스템에서 수행하는 것처럼 작업 실행을 복제합니다.
- 환경 변수에 익숙하지 않은 경우 이러한 변수는 쉘에서 해석하는 런타임 설정입니다. 일반화 된 방식으로 스크립트를 실행하는 동안 유연성을 제공합니다. 

<br>

#### BashOperator examples

```python
from airflow.operators.bash_operator import BashOperator
example_task = BashOperator(task_id='bash_ex',
                           	bash_command='echo 1',
                           	dag=dag)
```

BashOperator를 사용하기 전에 airflow.operator.bash_operator에서 임포트합니다.

task_id를 사용하는 BashOperator를 만듭니다.

bash명령 "echo 1"을 실행하고 연산자를 dag에 할당합니다.

```python
bash_task = BashOperator(task_id='clean_address',
                        	bash_command='cat addresses.txt | awk "NF==10" > cleaned.txt',
                         	dag=dag)
```

bash_command 에 cat 및 awk 를 사용하여 빠른 데이터 정리 작업을 실행하는 BashOperator입니다.

<br>

#### Operator gotchas

Operator를 운영하는데 몇가지 문제점이 있는데, 가장 큰 문제점은, 

개별 Operator가 동일한 위치 또는 환경에서 실행된다는 보장이 없다는 것입니다. (이것은 단지 하나의 연산자가 주어진, 특정 설정이 있는 디렉토리에서 다음 운영자가 동일한 정보에 액세스 할 수 있다는 의미가 아닙니다.)

특히 BashOperator에 대한 환경 변수를 설정해야 할 수 있습니다.

마지막으로 모든 형태의 상승 된 권한으로 작업하는 것이 까다로울 수 있습니다. (상승 된 권한이 무엇인지 확실하지 않을 경우 시스템에서 루트 또는 관리자로 명령을 실행하는 것도 고려)

<br>

#### Task

- Airflow 내에서 Task는 인스턴스화 된 연산자입니다.
- 작업은 일반적으로 Python 코드 내의 변수로 할당됩니다.

- Airflow 도구 내에서 이 작업은 변수 이름이 아닌 작업 ID로 참조됩니다.

<br>

#### Task dependencies

- Airflow의 작업 종속성은 작업 완료 순서를 정의합니다.
- 필수는 아니지만, 일반적으로 작업 종속성이 있습니다. (작업 종속성이 정의되지 않은 경우 실행은 순서 보장없이 Airflow 자체에서 처리됩니다.)
- 작업 종속성을 업스트림 또는 다운 스트림 작업이라고 합니다. (업스트림 작업은 다운스트림 작업 전에 완료해야 함을 의미합니다.)
- Airflow 1.8 이상 부터는, 작업 종속성은 비트 시프트 연산자를 사용하여 정의됩니다.
   - 업스트림 연산자는 >> 입니다.
   - 다운스트림 연산자는 << 입니다.

<br>

#### Upstream vs Downstream

업스트림과 다운스트림은 혼동하기 쉬운데, 가장 간단한 비유는, 업스트림은 이전을 의미하고 다운스트림은 이후를 의미하는 것입니다.

이것은, 모든 업스트림 작업이 다운 스트림 작업보다 먼저 완료되어야 함을 의미합니다.

<br>

#### Simple task dependency

```python
task1 = BashOperator(task_id='first_task',
                     bash_command='echo 1',
                     dag=example_dag)

task2 = BashOperator(task_id='second_task',
                     bash_command='echo 2',
                     dag=example_dag)
task1 >> task2 ## or task2 << task1
```

먼저 작업을 정의하고 task1 변수에 할당합니다.

그리고 두 번째 작업을 만들고 task2 변수에 할당합니다.

각 연산자가 정의되고 변수에 할당되면 비트 시프트 연산자를 사용하여 작업 순서를 정의 할 수 있습니다.

task2보다 task1을 먼저 실행하고자 할 때, 가장 읽기 쉬운건 업스트림을 사용하는 방법입니다. (동일한 작업을 다운스트림을 이용해서 역으로 정의 할 수 도 있습니다.)



<p align="center"><img src="/images/post_img/airflow7.PNG"></p>

작업 및 해당 종속성에 대해 Airflow UI에 표시되는 내용을 살펴보기 위해서는, Airflow 웹 인터페이스 내에서 그래프보기를 하면 됩니다.

작업 영역에서 두 개의 작업, first_task 및 second_task 둘다 존재하지만, 작업 실행 순서는 없습니다. 

이것은 bitshift 연산자를 사용하여 작업 종속성을 설정하기 전의 DAG입니다.



<p align="center"><img src="/images/post_img/airflow8.PNG"></p>

bitshift 연산자를 통해 정의 된 순서로 뷰를 다시 살펴보면 다음과 같습니다. 이전과 달리, 표시되는 작업 순서를 볼 수 있습니다.

<br>

#### Multiple dependencies

종속성은 필요에 따라 워크 플로를 정의하는데 필요한 만큼 복잡할 수도 있습니다.

```python
task1 >> task2 >> task3 >> task4 # 1
```

```python
task1 >> task2 << task3 # 2
```

<p align="center"><img src="/images/post_img/airflow9.PNG"></p>

즉, 더 명확한 형식으로 두 줄에 동일한 종속성 그래프를 정의 할 수 있습니다.

```python
task1 >> task2
task3 >> task2 # 2
```

<br>

#### PythonOperator

- PythonOperator는 Python 함수 또는 호출 가능한 메서드를 실행한다는 점을 제외하면 BashOperator와 유사
- BashOperator와 마찬가지로 taskid, dag 항목 및 대부분의 중요한 것은 파이썬_호출가능한 인수가 문제의 함수 이름으로 설정되어 있다
- 필요에 따라 Python callable에 인수 또는 키워드 스타일 인수를 전달할 수 도있습니다.

```python
from airflow.operators.python_operator import PythonOperator
def printme():
    print("This goes is the logs!")
python_task = PythonOperator(
	task_id='simple_print',
	python_callable=printme,
	dag=example_dag
)
```

먼저 airflow.operators.python_operator에서 PythonOperator를 호출합니다.

이후, 함수를 정의하고, 정의되면 python underscore task라는 PythonOperator 인스턴스를 만들고 필요한 인수를 추가합니다.

<br>

#### Arguments

- PythonOperator는 주어진 작업에 인수 추가를 지원합니나.
  - Positional(위치인수)
  - Keyword(키워드인수)

- PythonOperator로 키워드 인수를 구현하려면, `op_kwargs`라는 태스크에 대한 인수를 정의합니다.

<br>

#### op_kwargs example

```python
def sleep(length_of_time):
    time.sleep(length_of_time)

sleep_task = PythonOperator(
	task_id='sleep',
	python_callable=sleep,
	op_kwargs={'length_of_time':5},
	dag=example_dag)
```

시간 인수를 받는 sleep이라는 새 함수를 만들고(sleep시간을 입력받는 함수),

이전에 한것 처럼, PythonOperator를 생성합니다. 

op_kwargs 사전을 추가 하는데, 이는 함수에 입력되는 인자입니다. (사전 키는 함수 인수의 이름과 일치해야 합니다., 예기치 않은 키가 포함될 경우, 키워드 인수 오류가 발생합니다.)

<br>

#### EmailOperator

- 주로 연산자는 `airflow.operators` 또는 `airflow.contrib.operators ` 라이브러리에 있습니다.

- 또 다른 유용한 연산자는 EmailOperator로, 예상대로 Airflow 작업 내에서 이메일을 보냅니다.
- HTML 컨텐츠 및 첨부 파일을 포함하여 이메일의 일반적인 구성 요소를 포함 할 수 있습니다.
- 메시지를 성공적으로 보내려면 Airflow 시스템이 이메일 서버 세부 정보로 구성되어야합니다.

<br>

#### EmailOperator example

```python
from airflow.operators.email_operator import EmailOperator

email_task = EmailOperator(
	task_id='email_sales_report',
	to='sales_manage@example.com',
	subject='Automated Sales Report',
	html_content='Attached is the latest sales report',
	files='latest_sales.xlsx',
	dag=example_dag
)
```

먼저 airflow.operators.email_operator에서 EmailOperator를 가져와야합니다.

이후, 다음 작업ID로 EmailOperator 인스턴스를 만들 수 있습니다.

<br>

#### DAG Runs

- 이는 특정 시점의 워크 플로 인스턴스입니다. ( 예를 들어 현재 실행중인 인스턴스이거나, 지난 화요일 오후 3시에 한번 실행 될 수 있습니다.)
- DAG는 수동으로 실행하거나 DAG 정의시 전달 된 일정 간격 매개 변수를 통해 실행 할 수 있습니다.
- 각 DAG 실행은 자체 및 내부 작업에 대한 상태를 유지합니다.
  - DAG는 실행 중, 실패 또는 성공 상태 일 수 있습니다.



<p align="center"><img src="/images/post_img/airflow10.PNG"></p>

Airflow UI  Browse: DAG Runs 메뉴 옵션에서 모든 DAG 실행을 볼 수 있습니다.

이는 현재 Airflow 인스턴스 내에서 실행 된 DAG에 대한 다양한 세부 정보를 제공합니다.

<br>

#### Schedule details

DAG를 예약 할 때 예약 요구 사항에 따라 고려해야 할 많은 특성이 있습니다.

- `start_date`(시작 날짜 값)은, DAG를 처음으로 예약 할 수 있는 시간을 지정합니다. ( 일반적으로 Python datetime 객체로 정의됩니다. )

- `end-date`(종료 날짜)는 DAG를 예약 할 수 있는 마지막 시간을 나타냅니다.
- `max_tries`(최대 시도 횟수)는, DAG실행이 완전히 실패하기 전에 재 시도 할 횟수를 나타냅니다.
- `schedule_interval`(간격)은, 실행을 위해 DAG를 예약하는 빈도를 나타냅니다.

<br>

#### Schedule interval

`schedule_interval` represents:

- DAG 실행을 예약하는 빈도를 나타냄
- 예약은 시작 날짜와 잠재적 종료 날짜 사이에 발생합니다.
- 간격은 cron 스타일 구문을 사용하거나 내장 된 사전 설정을 통해 몇 가지 방법으로 할 수 있습니다.

<br>

#### cron syntax

<p align="center"><img src="/images/post_img/airflow11.PNG"></p>

- cron 구문은 Unix cron 도구를 사용하여 작업을 예약하는 형식과 동일

- 공백으로 구분 된 5개의 필드로 구성됩니다. (분, 시간, 일, 월, 요일)

- 모든 필드의 *표는 간격 (분 필드의 *표는 매분 실행을 의미)

- 값 목록은 쉼표로 구분 된 값을 통해 필드에 제공

<br.

#### cron examples

```python
0 12 * * * # Run daily at noon
* * 25 * 2 # Run once per minute on Februray 25
0,15,30,45 * * * * # Run every 15 minutes
```

<br>

#### Airflow scheduler presets

Airflow에는 자주 사용되는 시간 간격을 나타내는 몇 가지 사전 설정 또는 바로 가기 구문 옵션이 있습니다.



Preset:

- @hourly - 사전 설정은 시간이 시작 될 때 한시간에 한번 실행됨을 의미합니다.
- @daily
- @weekly



cron equivalent:

- 0 * * * *
- 0 0 * * *
- 0 0 * * 0

<br>

#### Special presets

Airflow에는 일정 간격에 대한 두 가지 특수 사전 설정도 있습니다.

- `None` 은 DAG를 예약하지 않으며 수동으로 트리거되는 워크 플로에 사용됨을 의미합니다.
- `@once` 는 DAG를 한 번만 예약하는 것을 의미합니다.

<br>



#### schedule_interval issues

DAG 실행을 예약 할 때 Airflow는 시작 날짜를 가능한 가장 빠른 값으로 사용하지만, 적어도 하나의 일정 간격이 시작 날짜를 초과 할 때까지 실제로 아무것도 예약하지 않습니다.

```python
'start_date': datetime(2020, 2, 25),
'schdule_interval': @daily
```

다음과 같이 주어지면, Airflow는 DAG의 첫 번째 실행에 2020년 2월 26일 날짜를 사용합니다. 

새 DAG일정을 추가 할 때 특히 일정 간격이 더 긴 경우 고려하기가 까다로울 수 있습니다.

<br>

## 3. Maintaining and monitoring Airflow workflows

<br>

#### Sensors

- 센서는 특정 조건이 참이되기를 기다리는 특별한 종류의 연산자
    - 생성 대기
    - 파일, 데이터베이스 레코드 업로드
    - 웹 요청의 특정 응답

  - 센서를 사용하면 조건이 참인지 확인하는 빈도를 정의 할 수 있습니다.
  - 센서는 작업자 유형이므로, 일반 작업자와 마찬가지로 작업에 할당됩니다.

<br>

#### Sensor details

- 모든 센서는  `airflow.sensors.base_sensor_operator` 클래스에서 파생됩니다.
- mode, poke_interval 및 timeout을 포함하여 모든 센서에 사용할 수 있는 몇가지 기본 인수가 있습니다.
- `mode` - 센서에 상태를 확인하는 방법을 알려주며 poke, reschedule 두 가지 옵션이 있습니다.
  - mode= 'poke' - default값이며, 작업자 슬롯을 포기하지 않고 완료 될 때까지 계속 확인하는 것을 의미
  - mode= 'reschedule' - 작업자 슬롯을 포기하고 다른 슬롯을 사용할 수 있을 때 까지 기다리는 것을 의미

- poke_interval - poke모드에서 사용되며, Airflow 조건을 확인하는 빈도를 알려줍니다. ( Airflow 스케쥴러가 과부하가 걸리지 않도록 하기 위해서는 시간이 1분 이상이여햐 합니다. )
- timeout - 센서 작업을 실패로 표시하기 전에 대기하는 시간입니다.
-  센서는 운영자이므로, task_id 및 DAG와 같은 일반 운영자 속성도 포함합니다.

<br>

#### File sensor

- 유용한 센서는 `airflow.contrib.sensors` 라이브러리에 있는 FileSensor입니다.

- FileSensor는 파일 시스템의 특정 위치에 파일이 있는지 확인합니다.

- 주어진 디렉토리 내의 모든 파일을 확인

```python
from airflow.contrib.sensors.file_sensor import FileSensor

file_sensor_task = FileSensor(task_id='file_sense',
                              filepath='salesdata.csv',
                              poke_interval=300,
                              dag=sales_report_dag)
init_sales_cleanup >> file_sensor_task >> generate_report
```

예시를 보면, FileSensor 개체를 가져온 다음, 파일센서를 정의합니다.

이전에 한 것과 비슷하게, task_id, dag항목을 설정합니다.

filepath 인수는 salesdata로 설정됩니다. 계속하기 전에 이 파일이름이 있는 파일을 찾습니다.

poke_interval을 300초로 설정하면, 참이 될 때 까지 5분마다 확인을 반복합니다.

또한, 비트시프트 연산자를 사용하여 DAG 내에서 센서의 종속성을 정의합니다.

<br>

#### Other sensors

Airflow에는 다양한 유형의 센서를 사용할 수 있습니다.

- `ExternalTaskSensor` - 별도의 DAG에 있는 작업이 완료 될 때 까지 기다립니다. (하나의 워크 플로를 너무 복잡하게 만들지 않고도, 다른 워크 플로 작업에 느슨하게 연결할 수 있습니다.)
- `HttpSensor` - 웹 URL을 요청하고 확인할 콘텐츠를 정의 할 수 있습니다.
- `SQLSensor` - SQL쿼리를 실행하여 콘텐츠를 확인합니다.
- 다른 많은 센서가 `airflow.sensors` and `airflow.contrib.sensors`에 있습니다.

<br>

#### Why sensors?

- 조건이 언제 참인지 불확실할 때.
- 조건을 계속 확인하고 싶지만, 반드시 전체 DAG를 즉시 실패하지는 않으려는 경우.

- DAG에 주기를 추가하지 않고 반복적으로 검사를 실행하려면 센서를 선택하는 것이 좋습니다.

<br>

#### What is an executor?

- Airflow에서 실행기는 워크 플로 내에 정의 된 작업을 실제로 실행하는 구성 요소입니다.
- 각 실행기는 작업 세트를 실행하기 위한 다른 기능과 동작을 가지고 있습니다.
- Example executors:
  - `SequentialExecutor`
  - `LocalExecutor`
  - `CeleryExecutor`

<br>

#### SequentialExecutor

- SequentialExecutor는 Airflow의 기본 실행 엔진입니다.
- 한 번에 하나의 작업 만 실행합니다.
- 즉, 동일한 시간대에 여러 워크 플로를 예약하면 예상보다 시간이 오래 걸릴 수 있습니다.
- 작업의 흐름을 따지는 것이 매우 간단하므로 디버깅에 유용합니다.
- 가장 중요한 측면은 매우 기능적이지만, 학습 및 테스트, 작업 리소스의 제한으로 인해 실제로 프로덕션에 권장되지 않습니다.

<br>

#### LocalExecutor

- LocalExecutor는 전적으로 단일 시스템에서 실행되는 Airflow의 또 다른 옵션입니다.
- 기본적으로 각 작업을 로컬 시스템의 프로세스로 취급하고 많은 작업을 시작할 수 있습니다.
- 이 동시성은 시스템의 병렬성이며 사용자가 다음에서 정의합니다.
- 지능적으로 정의 된 LocalExecutor는 단일 사용자에게 적합한 선택입니다.

<br>

#### CeleryExecutor

- 여러 시스템이 기본 클러스터로 통신 할 수 잇도록 Python으로 작성된 일반 대기열 시스템입니다.
- 여러 Airflow 시스템을 특정 워크 플로 / 작업 세트에 대한 작업자로 구성할 수 있습니다.
- 강력한 기능은 설정 및 구성이 훨씬 더 어렵습니다.
- 구성하기가 더 어렵지만, 많은 수의 DAG로 작업하거나, 처리량이 증가 할 것으로 예상하는 사람에게 유용

<br>

#### Determine your executor

- `airflow.cfg`파일을 통해 할 수 있습니다.
- `executor=` 를 검색하면, 사용중인 executor를 지정합니다.

<br>

#### Determine your executor 2

- `airflow list_dags`를 통해서 할 수 있습니다.
- `INFO - Using SequentialExecutor` 등으로 표시가 됩니다.

<br>

#### Typical issues...

- 일정에 따라 실행되지 않는 DAG 
- 단순히 시스템에 로드되지 않는 DAG
- Syntax errors(구문오류)

<br>



#### Dag won't run on schedule

- DAG가 일정대로 실행되지 않는 가장 일반적인 이유는, 스케쥴러가 실행되고 있지 않기 때문

<p align="center"><img src="/images/post_img/airflow12.PNG"></p>

스케쥴러 구성 요소가 실행되지 않을 경우, 웹 UI 내에서 오류가 표시

명령 줄에서 `airflow scheduler` 를 실행하여 이러한 문제를 해결

<br>

#### DAG won't run on schedule

- 시작 날짜 또는 마지막 DAG 실행 이후 하나 이상의 일정 간격이 지나지 않았습니다.
  -  이에 대해 구체적인 수정 사항은 없지만, 원하는 경우 요구 사항에 맞게 시작 날짜 또는 일정 간격을 수정합니다.
-  실행 프로그램에 작업을 실행하기에 충분한 여유 슬롯이 없을 때
  -  실행 프로그램을 변경하여 추가 작업을 수행 할 수 있는 항목에 입력합니다. ( LocalExcutor, CeleryExcutor)
  - 시스템 또는 시스템 리소스, DAG의 일정을 변경합니다.

<br>

#### DAG won't load

- 로드 되지않은 DAG는 웹 UI 의 DAG보기 또는, airflow list_dags에 출력되지 않습니다.
  - 가장 먼저 확인해야 할 것은 Python 파일이 예상 DAG 폴더 또는 디렉토리에 있는지
  - `airflow.cfg`를 확인하여 현재 DAG 폴더 설정을 확인
  - 폴더 경로는 절대 경로여야 합니다.

<br>

#### Syntax errors

- DAG 목록에는 Python 코드에 하나 이상의 구문 오류가 있을 때
- 두가지의 해결방법
  - Run `airflow list_dags`
  - Run python3 <dagfile.py>

<br>

#### SLAs

- SLA는 서비스 수준 계약을 나타냅니다.
- 비지니스 세계에서 이것은 종종 가동 시간 또는 가용성 보장입니다.
- Airflow는 이를 약간 다르게 취급합니다. 작업 또는 DAG를 실행하는 데 필요한 시간으로 간주합니다.
- SLA 누락은 작업 또는 DAG가 SLA의 예상 타이밍을 충족하지 못하는 모든 상황입니다.
- SLA 가 누락 된 경우, 시스템 구성에 따라 이메일 경고가 전송되고 로그에 메모가 작성됩니다.
- 모든 SLA 누락은 웹 UI의 찾아보기, SLA 누락 메뉴 항목에서 볼 수 있습니다.

<br>

#### SLA Misses

주어진 SLA 누락을 찾아보려면, Browse 에서 SLA 누락 링크를 통해 웹 UI에서 액세스 할 수 있습니다.

 <p align="center"><img src="/images/post_img/airflow13.PNG"></p>

SLA를 놓친 작업과 실패시기에 대한 일반적인 정보를 제공합니다. 또한 SLA가 실패했을 때 이메일이 전송되었는지 여부도 표시합니다.

<br>

#### Defining SLAs

- 작업 자체에 대한 `sla` 인수를 통한 방법 

```python
task1 = BashOperator(task_id='sla_task',
                     bash_command='runcode.sh',
                     sla=timedelta(seconds=30),
                     dag=dag)
```



- `default_args` 딕셔너리를 사용하고, sla키를 정의하는 방법

```python
default_args={
    'sla': timedelta(minutes=20)
    'start_date': datetime(2020,2,20)
}
dag = DAG('sla_dag', default_args=default_args)
```

<br>

#### timedelta object

- datetime 객체와 함께 datetime 라이브러리에 있습니다.
- `from datetime import timedelta` 를 통해 쉽게 액세스 할 수 있습니다.
- 일, 초, 분, 시간 및 주를 인수로 받습니다.

```python
timedelta(seconds=30)
timedelta(weeks=2)
timedelta(days=4, hours=10, minutes=20, seconds=30)
```

<br>

#### General reporting

보고 목적으로 Airflow에 내장 된 이메일 알림을 사용할 수 있습니다.

- Airflow에는 성공, 실패 또는 오류 / 재시도시 메시지를 보내기위한 기본제공 옵션이 있습니다.
- DAG 생성시 전달되는 default_args 딕셔너리의 키를 통해 처리됩니다.

```python
default_args={
    'email': ['airflowalerts@datacamp.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'email_on_success': True,
}
```

- 이전에 EmailOperator처럼, 정의 된 Airflow 옵션 중 하나를 벗어나 이메일을 보내는데 유용합니다.

<br>

## 4. Building production pipelines in Airflow

<br>

#### What are templates?

- 템플릿을 사용하면 DAG 실행 중에 정보를 대체 할 수 있습니다. **즉, 템플릿 정보가 있는 DAG가 실행되면 정보가 해석되고 DAG실행에 포함됩니다.**

- 템플릿은 작업을 정의 할 때 추가적인 유연성을 제공합니다.
- 템플릿은 `Jinja`템플릿 언어를 사용하여 생성됩니다.

<br>

#### Non-Templated BashOperator example

관리자가 "읽기" 라는 단어와 파일 목록을 로그 / 출력 등에 에코하도록 요청했습니다.

우리가 현재 알고 있는 것으로 이 작업을 수행한다면, Airflow BashOperator를 사용하여 여러 작업을 생성 할 것 입니다. 

```
t1 = BashOperator(
        task_id='first_task',
        bash_command='echo "Reading file1.txt"',
        dag=dag)
t2 = BashOperator(
		task_id='second_task',
		bash_command='echo "Reading file2.txt"',
		dag=dag)
```

처리해야 할 파일이 5개, 10개 또는 심지어 100개 이상인 경우 어떻게 될지 고려 ( 반복적인 코드가 많을 것 )

<br>

#### Templated BashOperator example

```
templated_command="""
	echo "Reading {{ param.filename }}"
	"""
t1 = BashOperator(task_id='template_task',
                  bash_command=templated_command,
                  params={'filename':'file1.txt'}
                  dag=example_dag)
```

다음 예시 코드 처럼, params로 파일이름을 주면, 이후 bash_command에서 해당 파일 이름을 변수처럼 사용합니다.

<br>

```
templated_command="""
	echo "Reading {{params.filename}}"
"""
t1 = BashOperator(task_id='template_task',
                  bash_command=templated_command,
                  params={'filename': 'file1.txt'},
                  dag=example_dag)
t2 = BashOperator(task_id='template_task',
                  bash_command=templated_command,
                  params={'filename': 'file2.txt'}
                  dag=example_dag)
```

2개의 BashOperator를 사용하는 것도 동일합니다.

<br>

#### More advanced template

이전에 작성한 템플릿 코드를 조금더 상향 시켜보겠습니다.

```
templated_command="""
{% for filename in params.filenames %}
	echo "Reading {{ filename }}"
{% endfor %}
"""
t1 = BashOperator(task_id='template_task',
                  bash_command=templated_command,
                  params={'filenames': ['file1.txt', 'file2.txt']}
                  dag=example_dag)
```

이전에 했던 것과, 다르게 for 문을 통해서 하나만 선언해도 2개의 함수가 했던 역할을 할 수 있습니다.

 <br>

#### Variables

- 템플릿 시스템의 일부로 Airflow는 기본 제공 런타임 변수 세트를 제공합니다.
- DAG 실행, 개별 작업 및 시스템 구성에 대한 다양한 정보를 제공합니다.
- 템플릿 예제에는 이중 중괄호 쌍의 ds 인 실행 날짜가 포함됩니다.

```
Execution Date: {{ ds }} # YYYY-MM-DD
Execution Date, no dashes: {{ ds_nodash }} # YYYYMMDD
Previous Execution date: {{ prev_ds }} # YYYY-MM-DD
Previous Execution date, no dashes: {{ prev_ds_nodash }} # YYYYMMDD
DAG object: {{ dag }}
Airflow config object: {{ conf }}
```

<br>

#### Macros

Airflow 변수 외에도 매크로 변수가 있습니다.

매크로 패키지는 Airflow 템플릿에 대한 다양한 유용한 개체 또는 메서드에 대한 참조를 제공합니다.

```txt
macros.datetime : The `datetime.datetime` object
macros.timedelta : The `timedelta` object
macros.uuid : Python's `uuid` object
macros.ds_add('2020-04-15', 5) : 일부 추가 기능도 사용 가능 ( 템플릿 내에서 날짜 계산을 쉽게 수행 할 수 있는 방법을 제공)
```

<br>

#### Branching

Branching은 Ariflow내에서 

- 조건부 논리 기능을 제공합니다. (기본적으로 작접자의 결과에 따라 작업을 선택적으로 실행하거나 건너 뛸 수 있음을 의미합니다.)

- 기본적으로 `BranchPythonOperator` 를 사용하고 있습니다.
- `from airflow.operators.python_operator import BranchPythonOperator` 를 통해 가져옵니다.
- python은 일반 `PythonOperator` 와 마찬가지로 호출 가능합니다. ( 이 함수는 task_id의 이름을 반환합니다. )

<br>

#### Bracnhing example

```python
def branch_test(**kwargs):
    if int(kwargs['ds_nodash']) % 2 == 0:
        return 'even_day_task'
    else:
        return 'odd_day_task'

branch_task = BranchPythonOperator(task_id='branch_task', dag=dag,
                                   provide_context=True,
                                   python_callable=branch_test)

start_task >> branch_task >> even_day_task >> even_day_task2
branch_task >> odd_day_task >> odd_day_task2
```

다음과 같이 **kwargs 인수는 전달 된 유일한 구성 요소입니다. 이것은 함수에 전달 된 키워드 사전에 대한 참조입니다.

함수에서 먼저 kwargs 사전에서 ds_dash키에 액세스합니다. 이를 정수화하여, 나머지가 0인지 확인합니다.

이것은 provide underscore 컨텍스트 인수를 전달하고 True로 설정한다는 점을 제외하면 PythonOperator와 같습니다. 함수에 대한 런타임 변수 및 매크로에 대한 액세스를 제공하도록 Airflow에 지시하는 구성 요소입니다. 이것은 함수 정의에서 kwargs 사전 객체를 통해 참조되는 것입니다.

이러한 종속성을 설정하지 않으면, 모든 작업이 분기 연산자가 반환 한 내용에 관계없이 정상적으로 실행됩니다.



 <p align="center"><img src="/images/post_img/airflow14.PNG"></p>

Airflow UI의 그래프보기를 통해 DAG를 살펴보면 다음과 같습니다.

<br>



#### Running DAGs & Tasks

구체적인 작업을 하고싶으면, 

`airflow run <dag_id> <task_id> <date>`

커맨드라인에서 다음과 같이 사용하세요. 이렇게하면, 지정된 날짜에 실행중인 것처럼 특정 DAG 작업이 실행됩니다.

전체 DAG를 실행하려면,

`airflow trigger_dag -e <date> <dag_id>`

<br>

#### Operators reminder

- BashOperator -  `bash_command` 가필요합니다.
- PythonOperator - `python_callable` 가 필요합니다.
- BranchPythonOperator -  `python_callable` and `provide_context=True` 가 필요합니다. 또한 함수는  `**kwargs` 인수로 되어야합니다.
- FileSensor -  `filepath`  가 필요합니다.  `mode` or `poke_interval` 를 설정해야 합니다.

<br>

#### Template reminder

- Airflow의 많은 objects 가 템플릿을 사용합니다. 
- 어떤 필드에서는 템플릿을 지원하지만, 지원하지 않는 것도 있습니다.
- 라이브 파이썬 인터프리터를 통해 파이썬 내장 문서를 이용하면, 템플릿을 지원하는지 알 수 있습니다.

 <p align="center"><img src="/images/post_img/airflow15.PNG"></p>
