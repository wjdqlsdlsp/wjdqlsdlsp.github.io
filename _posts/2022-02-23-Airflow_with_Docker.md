---

layout: post

title:  "Airflow 실습하기 (Feat. Docker)"

date:   2022-02-23

categories: Airflow

tags: Airflow

image: /post_img/airflow.png

---

#### Airflow 튜토리얼


<p align="center"><img src="/images/post_img/airflow.png"></p>

그동안 이론만 배웠던 Airflow를 직접 실습해보려고 합니다.

pip install를 통한 Airflow 구축 방법도 있지만, 저는 Docker를 이용해서 구축하려고 합니다.

<br>

#### 기본세팅

```shell
mkdir opt/airflow
cd opt/airflow
```

이후에 다운받는 yaml폴더를 보면 경로가 `opt/airflow/dags` 와 같이 설정되어 있습니다. 이예 맞춰 주기 위해 저는 위의 명령어들로 초기 세팅을 했습니다.

<br>

#### Airflow 홈페이지에 있는 yaml 파일 다운로드

```shell
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.2.4/docker-compose.yaml'
```

이후 airflow Docs에서 알려주는 방법을 따라, yaml 파일을 다운받습니다.

이 파일은 docekr-compose파일로 docker를 이용한 airlfow 환경 구축을 위해 다운받습니다.

<br>

#### dags, logs, plugins 파일 세팅

```shell
mkdir ./dags ./logs ./plugins
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

이후 폴더를 세팅하고 AIRFLOW_UID를 설정하기 위해 위의 코드를 입력해 줍니다.

이후 Airflow를 이용하기 위해 `./dags` 에 DAG를 만들어 넣을 예정입니다.

<br>

#### Airflow 초기화

```shell
docker-compose up airflow-init
```

위의 curl를 통해 다운받은 docker-compose를 이용한, 초기 환경 세팅입니다.

<br>

#### Airflow 시작

```shell
docker-compose up
```

초기환경 세팅이 끝났다면 위의 코드를 이용해서, Airflow를 실행해 줍니다.

이 과정은 다소 시간이 소요되니 천천히 기다려 줍니다. ( 직접 환경 구축하는 것보단 낫잖아요. ㅎㅎ )

<br>

#### Airflow 구경하기

터미널 환경에 Airflow라고 문구등이 뜨기 시작하면 어느정도 완성된 단계입니다.

```shell
localhost:8080
```

yaml을 통해서 환경을 구축했다면, 8080포트를 통해서 접속이 가능합니다. URL 주소에 위의 주소를 입력해주세요. 

위의 주소로 들어가면, 로그인창이 뜨는데, 놀라지마세요. 초기아이디 및 비밀번호는 모두 airflow 입니다.



<p align="center"><img src="/images/post_img/airflow16.png"></p>

<br>

#### Airflow DAGs

조금 기다리다면 Airflow에 내장되어 있는 튜토리얼 코드들이 보일겁니다. 이것 저것 눌러보면서 익숙해져 보세요.

<p align="center"><img src="/images/post_img/airflow17.png"></p>

<br>

#### DAGs 만들어서 테스트하기

이전에 `/opt/airflow/dags` 폴더를 만든 것 곳에 DAG을 만들어 저장하면, Airflow가 이를 인식합니다.

```python
from airflow.operators.bash_operator import BashOperator
from datetime import datetime
from airflow import DAG
import pendulum
kst = pendulum.timezone("Asia/Seoul")

text_file_path = '/opt/airflow/dags'
now = str(datetime.now())[14:16]

dag = DAG(
        dag_id="test_dag",
        start_date=datetime(2022, 2, 23,tzinfo = kst),
        schedule_interval="* * * * *")

create_text_file_command = f'cd {text_file_path} && echo hello airflow > test{now}.txt'
create_text_file = BashOperator(
        task_id='create_text_file',
        bash_command=create_text_file_command,
        dag=dag)

create_text_file
```

테스트를 하기 위해 간단한 코드를 만들었습니다.

이는 매분마다, 파일을 생성하는 코드입니다. 이를 dags폴더안에 파이썬 파일을 만들어 넣어주시면 됩니다.

DAG을 인식하는데도 어느정도 시간이 소요되니 기다려줍시다.

<p align="center"><img src="/images/post_img/airflow18.png"></p>

이후 다음과 같이 test_dag가 보일텐데, 이를 활성화 해줍니다.!

<p align="center"><img src="/images/post_img/airflow19.png"></p>

다음과 같이 Tree를 보면 마크를 통해 동작하고 있는지 실패했는지 등을 확인할 수 있습니다. ( 시간설정을 잘해야 하는 것 같습니다. start 시간부터 실행한 이전시간에 동작을 계속 표기해주네요. 실제 프로젝트에서는, 이렇게 매분 동작하게 스케쥴링을 하진 않을 것 같습니다. )

<p align="center"><img src="/images/post_img/airflow20.png"></p>

파일을 저장하라고 설정한 폴더를 보면, 다음과 같이 매분 파일을 잘 저장하는 것이 보입니다.

<br>

#### 기타 삽질...

- dags 폴더 위치를 다른 곳으로 변경하고 싶어서 몇가지 시도를 해봤는데, 적용이 잘 안됬습니다. Docker-compose 에서 무슨 설정이 따로 잡힌건지 아니면, 제가 미숙해서 그런건진 잘 모르겠습니다. 이유를 찾기 전 까지는 설정해준 위치로 사용해야 할 것 같습니다.
- Docker를 먼저 공부해야겠다는 생각을 들게합니다. 참 유용하면서 어려운 친구인 것 같습니다.

<br>



