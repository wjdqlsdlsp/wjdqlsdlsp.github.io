---
layout: post
title:  "Introduction to PySpark"
date:   2021-11-24
categories: 데이터엔지니어
tags: 데이터엔지니어
image: /post_img/oop.jpg
---

# Introduction to PySpark

<br>

## 1. Getting to know PySpark



#### What is Spark?

Spark는, 클러스터 컴퓨팅을 위한 플랫폼입니다.

Spark를 사용하면, 여러 노드가 있는 클러스터에 데이터와 계산을 분산할 수 있습니다. ( 각 노드를 컴퓨터로 생각 )

데이터를 분할하면 각 노드가 소량의 데이터에서만 작동하기 때문에 매우 큰 데이터 세트로 더 쉽게 작업 할 수 있습니다.

각 노드는, 전체 계산의 일부를 수행하며, 데이터 처리와 계산이 클러스터의 노드에서 병렬로 수행 됩니다.

병렬 계산은, 특정 유형의 프로그래밍 작업을 훨씬 더 빠르게 만들 수 있습니다. ( 단, 컴퓨팅 성능이 향상되면 복잡성도 커집니다. )

Spark가 해당 문제에 대한 최상의 솔루션 여부를 결정 하는 데는 경험이 필요합니다.

- 데이터가 너무 커서, 단일화 시스템에서 작업 할 수 없다.
- 계산을 쉽게 병렬화 할 수 있다.

<br>

#### Using Spark in Python

Spark 사용의 첫번째는, 클러스터에 연결하는 것입니다.

클러스터는 다른 모든 노드에 연결된 원격 시스템에서 호스팅 됩니다. 데이터와 계산 분할을 관리하는 컴퓨터를 **마스터(master)** 라고 하며, 마스터는 **작업자(worker)** 라고 하는 클러스터의 나머지 컴퓨터와 연결됩니다.

마스터는 작업자에게 데이터와 계산을 실행하도록 보내고, 작업자는 결과를 다시 마스터에게 보냅니다.



해당 연결을 만드는 방법은 `SparkContext` 클래스의 인스턴스를 만드는 것입니다. 

클래스 생성자는 연결하려는 클러스터의 속성을 지정할 수 있는 몇가지 인수를 사용합니다.

모든 속성을 포함하는 개체는 `SparkConf()`생성자를 사용하여 만들 수 있습니다. 



#### Examining The SparkContext

Spark는 serious software입니다. 이것은 사용하는 것보다, 시작하는데 시간을 더 많이 소요합니다.

간단한 프로그램은, 스파크보다, 다른 것을 사용하는 것이 더 효율적일 것 입니다.

```python
# Verify SparkContext
print(sc)

# Print Spark version
print(sc.version)
```

<br>

#### Examining The SparkContext

Spark는 시작하는데 많은 시간이 걸립니다. 또한 간단한 계산을 실행하는 데 오래 걸릴수 있습니다.

Spark는 빅데이터와 같이 복잡한 작업을 위해 설계되어 있기 때문입니다. ( 작은 작업을 할 때는, Spark가 효율적이지 않다. )

<br>

#### Using DataFrames

Spark의 핵심 데이터 구조는 RDD(Resilient Distributed Dataset) 입니다. 이것은 클러스트의 여러 노드에 데이터를 분할하여 Spark가 마법을 사용할 수 있게 하는 저수준의 개체이지만, RDD는 직접 작업하기 어렵기때문에, RDD 위에 구축된 Spark DataFrame 추상화를 사용합니다.

Spark DataFrame은 SQL 테이블과 매우 유사하게 설계되어있습니다. DataFrame 은 이해하기 쉬울 뿐만 아니라 RDD보다 복잡한 작업에 더 최적화 되어있습니다.

<br>

RDD를 사용할 때, 쿼리를 최적화하는 올바른 방법을 파악하는 것은, 데이터 과학자에게 있지만, DataFrame 구에는 최적화 부분이 내장되어 있습니다.

Spark의 DataFrame작업을 시작하면, `SparkSession` 을 만들고, `SparkContext`, `SparkContext` 클러스터에 `SparkSession` 과 연결합니다.

<br>

#### Creating a SparkSession

`Sparksession` 은 `spark`라고 불리는 것을 통해, 생성합니다.

하지만, `SparkSession` 와, `SparkContext`를 여러개 생성하면 오류가 날 수 있습니다.

`SparkSession.builder.getOrCreate()` 를 통해 생성하는 것이 좋은 방법입니다.

```python
# Import SparkSession from pyspark.sql
from pyspark.sql import SparkSession

# Create my_spark
my_spark = SparkSession.builder.getOrCreate()

# Print my_spark
print(my_spark)
```

<br>

#### Viewing tables

`SparkSession` 은, `catalog` 라고 불리는 특성을 가집니다. 그 중 유용한, `.listTables()`를 사용하면, 해당 테이블의 이름을 반환합니다.

```python
# Print the tables in the catalog
print(spark.catalog.listTables())
```

<br>

#### Are you query-ious?

Spark의 이점 중 하나인, DataFrame Interface는 SQL 쿼리를 Spark cluster에서 사용할 수 있다는 것입니다.

```python
# Don't change this query
query = "FROM flights SELECT * LIMIT 10"

# Get the first 10 rows of flights
flights10 = spark.sql(query)

# Show the results
flights10.show()
```

<br>

#### Pandafy a Spark DataFrame

Spark DataFrame을 pandas로 만드는 방법도 있습니다. `.toPandas()` 이용

```python
# Don't change this query
query = "SELECT origin, dest, COUNT(*) as N FROM flights GROUP BY origin, dest"

# Run the query
flight_counts = spark.sql(query)

# Convert the results to a pandas DataFrame
pd_counts = flight_counts.toPandas()

# Print the head of pd_counts
print(pd_counts.head())
```

<br>

#### Put some Spark in your data

이전과 달리, pandas DataFrame을 spark DataFrame으로 만드는 법 또한 당연히 존재합니다.

```python
# Create pd_temp
pd_temp = pd.DataFrame(np.random.random(10))

# Create spark_temp from pd_temp
spark_temp = spark.createDataFrame(pd_temp)

# Examine the tables in the catalog
print(spark.catalog.listTables())

# Add spark_temp to the catalog
spark_temp.createOrReplaceTempView("temp")

# Examine the tables in the catalog again
print(spark_temp)
```

<br>

#### Dropping the middle man

```python
# Don't change this file path
file_path = "/usr/local/share/datasets/airports.csv"

# Read in the airports data
airports = spark.read.csv(file_path, header=True)

# Show the data
airports.show()
```

<br>

## 2. Manipulating data

<br>

#### Creating columns

`.withColumn()`  2개의 arguments 필요로하며, 새로운 column이름과, 데이터를 필요로합니다. 새로운 column은 객체이여야 합니다.

예시:  `df = df.withColumn("newCol", df.oldCol + 1)`

```python
# Create the DataFrame flights
flights = spark.table("flights")

# Show the head
flights.show()

# Add duration_hrs
flights = flights.withColumn("duration_hrs", flights.air_time/60)
```

<br>

#### SQL in a nutshell

SQL 쿼리는 데이터베이스에 포함된 하나 이상의 테이블에서 파생된 테이블을 반환합니다.

모든 SQL 쿼리는 데이터 베이스에 데이터로 수행하려는 작업을 알려주는 명령으로 구성됩니다.

```sql
SELECT * FROM my_table;
```

다음과 같이 연산을 할 수 도 있습니다.

```sql
SELECT origin, dest, air_time / 60 FROM flights;
```

```sql
SELECT * FROM students
WHERE grade = 'A';
```

<br>

#### SQL in a nutshell (2)

데이터베이스에서 집계하는 작업을 수행하는 것은, 데이터 양을 줄이는데 효과적입니다. 이는 SQL에서 `GROUP BY` 를 통해서 수행됩니다.

```sql
SELECT COUNT(*) FROM flights
GROUP BY origin;
```

```sql
SELECT origin, dest, COUNT(*) FROM flights
GROUP BY origin, dest;
```

<br>

#### Filtering Data

SQL에서 where를 사용했던 것처럼, DataFrame에서 `.filter` 를 사용하는 방법도 있습니다.

```python
flights.filter("air_time > 120").show()
flights.filter(flights.air_time > 120).show()
```

SQL에서는 다음의 코드로 동작했을 것입니다.

```sql
SELECT * FROM flights WHERE air_time > 120
```

```python
# Filter flights by passing a string
long_flights1 = flights.filter("distance > 1000")

# Filter flights by passing a column of boolean values
long_flights2 = flights.filter(flights.distance  > 1000)

# Print the data to check they're equal
long_flights1.show()
long_flights2.show()
```

<br>

#### Selecting

Spark에서는 `SELECT` 를 `.select()` 를 통해 사용합니다.

`withColumn()` 과의 차이점은, `select()` 은 지정한 열만 반환하지만, `withColumn()` 은 모든 DataFrame 열을 반환합니다.

```python
# Select the first set of columns
selected1 = flights.select("tailnum", "origin", "dest")

# Select the second set of columns
temp = flights.select(flights.origin, flights.dest, flights.carrier)

# Define first filter
filterA = flights.origin == "SEA"

# Define second filter
filterB = flights.dest == "PDX"

# Filter the data, first by filterA then by filterB
selected2 = temp.filter(filterA).filter(filterB)
```

<br>

#### Selecting II

`.select()` 안에서 연산도 가능합니다.

`flights.select(flights.air_time/60)`



`.alias()` 를 통해 SQL에서 `as`처럼 사용되던 것을 구현할 수 도 있습니다.

`flights.select((flights.air_time/60).alias("duration_hrs"))`

혹인 `.selectExpr()` 를 사용할 수도 있습니다.

`flights.selectExpr("air_time/60 as duration_hrs")`

```python
# Define avg_speed
avg_speed = (flights.distance/(flights.air_time/60)).alias("avg_speed")

# Select the correct columns
speed1 = flights.select("origin", "dest", "tailnum", avg_speed)

# Create the same table using a SQL expression
speed2 = flights.selectExpr("origin", "dest", "tailnum", "distance/(air_time/60) as avg_speed")
```

<br>

### Aggregating

`.groupBy()` 를 통해, `.min()`,  `.max()`,  `.count()`등을 사용할 수 있습니다.

`df.groupBy().min("col").show()`

```python
# Find the shortest flight from PDX in terms of distance
flights.filter(flights.origin == "PDX").groupBy().min("distance").show()

# Find the longest flight from SEA in terms of air time
flights.filter(flights.origin == "SEA").groupBy().max("air_time").show()
```

<br>

#### Aggregating II

```python
# Average duration of Delta flights
flights.filter(flights.carrier == "DL").filter(flights.origin == "SEA").groupBy().avg("air_time").show()

# Total hours in the air
flights.withColumn("duration_hrs", flights.air_time/60).groupBy().sum("duration_hrs").show()
```

<br>

#### Grouping and Aggregating I

`groupBy()` 뒤, sum, count 등을 추가하지않으면, 다음 처럼 사용 가능합니다.

```python
# Group by tailnum
by_plane = flights.groupBy("tailnum")

# Number of flights each plane made
by_plane.count().show()

# Group by origin
by_origin = flights.groupBy("origin")

# Average duration of flights from PDX and SEA
by_origin.avg("air_time").show()
```

<br>

#### Grouping and Aggregating II

`groupBy` 뿐만 아니라, `.agg()` 도 사용 가능합니다.

`.agg()` 메소드는 `pyspark.sql.functions` 에 포함되어 있습니다.

```python
# Import pyspark.sql.functions as F
import pyspark.sql.functions as F

# Group by month and dest
by_month_dest = flights.groupBy("month", "dest")

# Average departure delay by month and destination
by_month_dest.avg("dep_delay").show()

# Standard deviation of departure delay
by_month_dest.agg(F.stddev("dep_delay")).show()
```

<br>

#### Joining

Joining은 많이 사용되는 방법 중 하나입니다.

조인은 다른 2개의 테이블을 결합시키게 됩니다. 

PySpark에서는 DataFrame 함수로, `.join()` 을 사용합니다. 세개의 인자가 있는데, 결합하고자 하는 테이블, on, how 입니다.

`on` 은, 어떤 컬럼을 기준으로 join하는지를, `how`는 leftouter 등 어떤 방식으로 join 하는지를 선택하는 것 입니다.

 ```python
 # Examine the data
 print(airports.show())
 
 # Rename the faa column
 airports = airports.withColumnRenamed("faa", "dest")
 
 # Join the DataFrames
 flights_with_airports = flights.join(airports, on="dest", how="leftouter")
 
 # Examine the new DataFrame
 print(flights_with_airports.show())
 ```

<br>

## 3. Getting started with machine learning pipelines

<br>

#### Machine Learning Pipelines

`pyspark.ml` 의 중심은, `Transformer` 와 `Estimator` 클래스입니다.

`Transformer` 클래스는, `.transformer()` 함수를 통해서, DataFrame을 변환합니다.

`Bucketizer`를 이용하여, 연속된 피처 혹은 클래스로 부터, discrete bins을 만들거나, `PCA`를 통해 데이터셋의 주성분 분석을 할 수 있습니다.

`Estimator`클래스는, `.fit()`를 통해 실행됩니다. `StringIndexerModel` 혹은 `RandomForestModel` 이 이것에 해당됩니다.

<br>

#### Join the DataFrames

```python
# Rename year column
planes = planes.withColumnRenamed("year","plane_year")

# Join the DataFrames
model_data = flights.join(planes, on="tailnum", how="leftouter")
```

<br>

#### Data types

Spark는 numeric 데이터만, 핸들링 할 수 있습니다. 

우리가 데이터를 선언하면, Spark는 해당 데이터에 대한 정보를 파악합니다. 불행히도 항상 옳바르게 판단하지는 않습니다. DataFrame의 일부 열은, 실제 숫자 값이 아닌, 숫자가 퐇마된 문자열임을 알 수 있습니다.

이러한 문제를 해결하기 위해서 `.withColumn()` 와 함께`.cast()`를 사용합니다. 

`.cast()` 는 인자로 하나만 입력 받는데, 정수일경우 "integer"를 decimal numbers인 경우 "double"을 입력하면 됩니다.

<br>

#### String to integer

```python
dataframe = dataframe.withColumn("col", dataframe.col.cast("new_type"))
```

```python
# Cast the columns to integers
model_data = model_data.withColumn("arr_delay", model_data.arr_delay.cast("integer"))
model_data = model_data.withColumn("air_time", model_data.air_time.cast("integer"))
model_data = model_data.withColumn("month", model_data.month.cast("integer"))
model_data = model_data.withColumn("plane_year",  model_data.plane_year.cast("integer"))
```

<br>

#### Create a new column

```python
# Create the column plane_age
model_data = model_data.withColumn("plane_age", model_data.year - model_data.plane_year)
```

<br>

#### Making a Boolean

```python
# Create is_late
model_data = model_data.withColumn("is_late", model_data.arr_delay > 0)

# Convert to an integer
model_data = model_data.withColumn("label", model_data.is_late.cast("integer"))

# Remove missing values
model_data = model_data.filter("arr_delay is not NULL and dep_delay is not NULL and air_time is not NULL and plane_year is not NULL")
```

<br>

#### Strings and factors

모델링을 위해선, 숫자형 데이터가 필요합니다. 따라서, String을 숫자형 데이터로 변환해야 합니다.

Pyspark에서는, `pyspark.ml.feature` 하위 모듈에 이를 처리하는 기능이 내장되어 있습니다. 이는 one-hot-vector를 생성하는 방법을 이용합니다.

먼저 `StringIndexer`를 생성합니다. 이 클래스이 멤버는 `Estimator`문자열 열이 있는 DataFrame의 고유 문자열를 매핑합니다. 그리고, `Estimator`는 매핑된 메타데이터를 포함한 DataFrame의 `Transformer`를 리턴합니다. 또한, 문자열 컬럼과 관련있는 숫자형 컬럼도 리턴합니다.

두번 째 단계는, 이 컬럼을 `OneHotEncoder`를 이용하여 인코드합니다. 

<br>

#### Carrier

```python
# Create a StringIndexer
carr_indexer = StringIndexer(inputCol="carrier", outputCol="carrier_index")

# Create a OneHotEncoder
carr_encoder = OneHotEncoder(inputCol="carrier_index", outputCol="carrier_fact")
```

<br>

#### Destination

```python
# Create a StringIndexer
dest_indexer = StringIndexer(inputCol="dest", outputCol="dest_index")

# Create a OneHotEncoder
dest_encoder = OneHotEncoder(inputCol="dest_index", outputCol="dest_fact")
```

<br>

#### Assemble a vector

```python
# Make a VectorAssembler
vec_assembler = VectorAssembler(inputCols=["month", "air_time", "carrier_fact", "dest_fact", "plane_age"], outputCol="features")
```

<br>

#### Create the pipeline

```python
# Import Pipeline
from pyspark.ml import Pipeline

# Make the pipeline
flights_pipe = Pipeline(stages=[dest_indexer, dest_encoder, carr_indexer, carr_encoder, vec_assembler])
```

<br>

#### Test vs Train

모델링을 위해서 중요한 것 중 하나는, 데이터를 나누는 것입니다.

Spark에서는 모든 변환 후에 데이터를 분할하는 것이 중요합니다.

```python
# Fit and transform the data
piped_data = flights_pipe.fit(model_data).transform(model_data)
```

```python
# Split the data into training and test sets
training, test = piped_data.randomSplit([.6, .4])
```

<br>

## 4. Model tuning and selection

<br>

#### Create the modeler

```python
# Import LogisticRegression
from pyspark.ml.classification import LogisticRegression

# Create a LogisticRegression Estimator
lr = LogisticRegression()
```

<br>

#### Cross validation

훈련 데이터를 몇 개의 다른 파티션으로 분할하여 작동합니다. 

데이터가 분할되면, 파티션 중 하나가 따로 설정되며 모델이 다른 파티션에 작동하고, 보류된 파티션에 대해 loss를 측정합니다. 

이것은 각 파티션에 대해 반복되며 각 파티션의 loss값의 평균을 출력합니다. 

이를 교차 검증 로규앖이라고 하며 실제 오류에 대한 좋은 추정치를 제공합니다.

`pyspark.ml.evaluation` 를 통해서 값을 평가 할 수 있습니다.

```python
# Import the evaluation submodule
import pyspark.ml.evaluation as evals

# Create a BinaryClassificationEvaluator
evaluator = evals.BinaryClassificationEvaluator(metricName="areaUnderROC")
```

<br>

#### Make a grid

```python
# Import the tuning submodule
import pyspark.ml.tuning as tune

# Create the parameter grid
grid = tune.ParamGridBuilder()

# Add the hyperparameter
grid = grid.addGrid(lr.regParam, np.arange(0, .1, .01))
grid = grid.addGrid(lr.elasticNetParam, [0, 1])

# Build the grid
grid = grid.build()
```

<br>

#### Make the validator

```python
# Create the CrossValidator
cv = tune.CrossValidator(estimator=lr,
               estimatorParamMaps=grid,
               evaluator=evaluator
               )
```

<br>

#### Fit the model(s)

```python
# Call lr.fit()
best_lr = lr.fit(training)

# Print best_lr
print(best_lr)
```

<br>

#### Evaluate the model

```python
# Use the model to predict the test set
test_results = best_lr.transform(test)

# Evaluate the predictions
print(evaluator.evaluate(test_results))
```

