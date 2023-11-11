## 목차

- 3.0 [스파크 기능 둘러보기](#30-스파크-기능-둘러보기)
- 3.1 [운영용 애플리케이션 실행하기(예제)](#31-운영용-애플리케이션-실행하기)
  - [pi 계산](#pi-계산)
- 3.2 [Dataset : 타입 안정성을 제공하는 구조적 API(예제)](#32-dataset--타입-안정성을-제공하는-구조적-api)
  - [장점](#장점)
- 3.3 [구조적 스트리밍](#33-구조적-스트리밍)
  - [소매 데이터셋(예제)](#소매retail-데이터셋)
    - [DataFrame, Schema 생성](#dataframe-schema-생성)
    - [데이터 그룹화 및 집계](#데이터-그룹화-및-집계)
    - [스트리밍 코드](#스트리밍-코드)
    - [비즈니스 로직 적용](#비즈니스-로직-적용)
    - [스트리밍 액션](#스트리밍-액션)
- 3.5 [저수준 API(예제)](#35-저수준-api)

## 3.0 스파크 기능 둘러보기

앞 장에서는 트랜스포메이션과 액션 등 스파크의 구조적 API와 관련된 핵심 개념을 소개했다.  
이런 단순한 개념적 구성 요소는 스파크 에코 시스템의 방대한 기능과 라이브러리의 바탕을 이룬다.

스파크는 기본 요소인 저수준 API와 구조적 API, 추가 기능을 제공하는 일련의 표준 라이브버리로 구성되어 있다.

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/7c97e698-da68-4933-af64-e5dceec59153">

$\rightarrow$ 스파크의 기능

스파크의 라이브러리는 그래프 분석, 머신러닝, 스트리밍 등 다양한 작업을 지원하며 컴퓨팅 및 스토리지 시스템과의 통합을 돕는 역할을 한다.

이 장에서는 다음과 같은 내용을 설명한다.

- spark-submit 명령으로 운영용 애플리케이션 실행

- Dataset : 타입 안정성(type-safe)을 제공하는 구조적 API

- 구조적 스트리밍

- 머신러닝과 고급 분석

- RDD : 스파크의 저수준 API

- SparkR

- 서드파티 패키지 에코 시스템

## 3.1 운영용 애플리케이션 실행하기

스파크를 사용하면 빅데이터 프로그램을 쉽게 개발할 수 있다.

**spark-submit** 명령을 사용해 대화형 셸에서 개발한 프로그램을 운영용 애플리케이션으로 쉽게 전환할 수 있다.

**spark-submit**

- 애플리케이션 코드를 클러스터에 전송해 실행시키는 역할

- 클러스터에 제출된 애플리케이션은 작업이 종료되거나 에러가 발생할 때까지 실행

- 스파크 애플리케이션은 스탠드얼론, 메소스, YARN 클러스터 매니저를 이용해 실행

- 애플리케이션 실행에 필요한 자원과 실행 방식, 다양한 옵션 지정 가능

### pi 계산

**로컬 머신에서 애플리케이션을 실행**한다. 스파크와 함께 제공되는 스칼라 애플리케이션 예제를 실행한다. 스파크 디렉토리에서 아래 명령어를 입력한다.

```scala
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local \
  ./examples/jars/spark-examples_2.12-3.5.0.jar 10 // 버전에 따라 다름
```

$\rightarrow$ 스칼라 코드

```bash
./bin/spark-submit \
  --master local \
  ./examples/src/main/python/pi.py 10
```

$\rightarrow$ 파이썬 코드

위 예제는 pi 값을 특정 자릿수까지 계산한다.

spark-submit 명령에 예제 클래스를 지정하고 로컬 머신에서 실행되도록 설정했으며, 실행에 필요한 JAR 파일과 관련 인수도 함께 지정했다.

master 옵션의 인숫값을 변경하면 스파크가 지원하는 스파크 스탠드얼론, 메소스, YARN 클러스터 매니저에서 동일한 애플리케이션을 실행할 수 있다.

## 3.2 Dataset : 타입 안정성을 제공하는 구조적 API

**Dataset**은 자바와 스칼라의 정적 데이터 타입에 맞는 코드, 즉 정적 타입 코드(statically typed code)를 지원하기 위해 고안된 스파크의 **구조적 API**이다.

타입 안정성을 지원하며 동적 타입 언어인 파이썬과 R에서는 사용할 수 없다.

- **정적 타입 언어** (자료형이 고정된 언어) : 자바, 스칼라, C, C++
- **동적 타입 언어** : 파이썬, 자바스크립트, R

**DataFrame**은 다양한 데이터 타입의 테이블형 데이터를 보관할 수 있는 Row 타입의 객체로 구성된 분산 컬렉션이다.  
Dataset API는 DataFrame의 레코드를 자바나 스칼라로 정의한 클래스에 할당하고 자바의 ArrayList, 스칼라의 Seq 객체 등의 고정 타입형 컬렉션으로 다룰 수 있는 기능을 제공한다.

타입 안정성을 제공하기 때문에 초기화에 사용한 클래스와 다른 클래스로 접근할 수 없다.  
$\rightarrow$ 잘 정의된 인터페이스로 상호 작용하는 대규모 애플리케이션 개발에 유용

### 장점

- **Dataset 클래스**(**자바** : **Dataset\<T>**, **스칼라** : **Dataset\[T]**)는 내부 객체의 데이터 타입을 매개변수로 사용

  $\rightarrow$ 예로 Dataset[Person]은 Person 클래스의 객체만 가질 수 있다.

  스파크 2.0 버전에서는 자바의 JavaBean 패턴(변수의 Setter 메서드 정의해 매개변수 전달 방식), 스칼라의 케이스 클래스 유형으로 정의된 클래스를 지원한다.

  자동으로 타입 T 분석 후 Dataset의 표 형식 데이터에 적합한 스키마를 생성해야 하기 때문에 이러한 타입을 제한적으로 사용할 수 밖에 없다.

- **필요한 경우 선택적으로 사용 가능**

  $\rightarrow$ 예로 데이터 타입을 정의하고 map, filter 함수를 사용할 수 있다.

  스파크는 처리를 마치고 결과를 DataFrame으로 자동 변환해 반환한다. 또한 스파크가 제공하는 여러 함수를 이용해 추가 처리 작업이 가능하다.

  이는 타입 안정성을 보장하는 코드에서 저수준 API를 사용할 수 있고, 고수준 API의 SQL을 사용해 빠른 분석을 가능하게 한다.

- **Dataset에 매개변수로 지정한 타입의 객체 반환**

  $\rightarrow$ collect 메서드나 take 메서드를 호출하면 DataFrame을 구성하는 Row 타입의 객체가 아닌 Dataset에 매개변수로 지정한 타입의 객체를 반환한다.

  따라서 코드 변경 없이 타입 안정성을 보장할 수 있고, 로컬이나 분산 클러스터 환경에서 데이터를 안전하게 다룰 수 있다.

```scala
case class Flight(DEST_COUNTRY_NAME: String,
                  ORIGIN_COUNTRY_NAME: String,
                  count: BigInt)

val flightsDF = spark.read
  .parquet("./data/flight-data/parquet/2010-summary.parquet/")

val flights = flightsDF.as[Flight]
```

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/1ee17de3-3914-4470-a5ef-e874b45473e7">

```scala
flights
  .filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
  .map(flight_row => flight_row)
  .take(5)
```

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/a0ab29ff-92f2-4c28-936d-98e87de7c663">

```scala
flights
  .take(5)
  .filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
  .map(fr => Flight(fr.DEST_COUNTRY_NAME, fr.ORIGIN_COUNTRY_NAME, fr.count + 5))
```

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/82a5e431-918d-4118-9ac5-fc52ddd917e3">

## 3.3 구조적 스트리밍

- 스파크 2.2 버전에서 안정화(production-ready)된 스트림 처리용 고수준 API

- 구조적 API로 개발된 배치 모드의 연산을 스트리밍 방식으로 실행 가능

- 지연 시간 줄이고 증분 처리 가능

- 배치 처리용 코드를 일부 수정해 스트리밍 처리를 수행하고 값을 빠르게 얻을 수 있다는 장점 존재

- 프로토타입을 배치 잡으로 개발 후 스트리밍 잡으로 변환 가능

- 모든 작업은 데이터를 증분 처리하면서 수행

### 소매(retail) 데이터셋

구조적 스트리밍을 얼마나 쉽게 만들 수 있는지 간단한 예제로 알아본다.  
예제에서는 [소매(retail) 데이터셋](https://bit.ly/2PvOwCS)을 사용한다. 데이터셋에는 특정 날짜와 시간 정보가 존재한다. 예제해서는 하루치 데이터를 나타내는 by-day 디렉터리의 파일을 사용한다.

소매 데이터이므로 소매점에서 생성된 데이터가 구조적 스트리밍 잡이 읽을 수 있는 저장소로 전송되고 있다고 가정한다.(여러 프로세스에서 데이터가 꾸준하게 생성되는 상황)

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/fadeb4b0-ffec-4363-9995-38fa344ecc91">

$\rightarrow$ 데이터 구조

### DataFrame, Schema 생성

정적 데이터셋의 데이터를 분석해 DataFrame을 생성하고 정적 데이터셋의 스키마도 함께 생성한다.

```scala
val staticDataFrame = spark.read.format("csv")
  .option("header", "true")
  .option("inferSchema", "true")
  . load("./data/retail-data/by-day/*.csv")

staticDataFrame.createOrReplaceTempView("retail_data")

val staticSchema = staticDataFrame.schema
```

<img width="700" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/3db25ff0-1f51-470b-9ffe-8d6698ec68e3">

### 데이터 그룹화 및 집계

시계열(time-series) 데이터를다루기 때문에 데이터를 그룹화하고 집계하는 방법을 알아볼 필요가 있다.

이를 위해 특정 고객(CustomerId로 구분)이 대량으로 구매하는 영업 시간을 살펴본다. 예를 들어 총 구매비용 컬럼을 추가하고 고객이 가장 많이 소비한 날을 찾는다.

윈도우 함수(window fuction)는 집계 시 시계열 컬럼을 기준으로 각 날짜에 대한 전체 데이터를 가지는 윈도우를 구성한다. 윈도우 간격을 통해 처리 요건을 명시할 수 있어 날짜와 타임스탬프 처리에 유용하다.  
스파크는 관련 날짜의 데이터를 그룹화한다.

```scala
import org.apache.spark.sql.functions.{window, col}

staticDataFrame
  .selectExpr(
    "CustomerId",
    "(UnitPrice * Quantity) as total_cost",
    "InvoiceDate")
  .groupBy(
    col("CustomerId"), window(col("InvoiceDate"), "1 day"))
  .sum("total_cost")
  .show(5)
```

<img width="350" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e55bb2aa-2b7e-4f46-b6e5-0eadee16716b">

(책의 실행 결과와 달라 확인해보니 데이터셋 안 Quantity 값이 음수인 경우도 있어 total_cost가 음수값인 로우가 존재한다.)

로컬 모드로 위 코드를 실행하려면 로컬 모드에 적합한 셔플 파티션 수를 설정하는 것이 좋다.

셔플 파티션 수는 셔플 이후에 생성될 파티션 수를 의미한다. 기본값은 200이나 로컬 모드에서는 많은 익스큐터가 필요하지 않아 5로 줄인다.

```scala
spark.conf.set("spark.sql.shuffle.partitions", "5")
```

### 스트리밍 코드

지금까지 동작 방식을 알아보았다. 이제 스트리밍 코드를 살펴본다. 코드는 거의 바뀌지 않는다.

read 메서드 대신 **readStream** 메서드를 사용하는 것이 가장 큰 차이점이다. 그리고 maxFilesPerTrigger 옵션을 추가로 지정해 한 번에 읽을 파일 수를 설정한다.

```scala
val streamingDataFrame = spark.readStream
  .schema(staticSchema)
  .option("maxFilesPerTrigger", 1)
  .format("csv")
  .option("header", "true")
  .load("./data/retail-data/by-day/*.csv")
```

위 코드를 실행 후 DataFrame이 스트리밍 유형인지 확인할 수 있다.

```scala
streamingDataFrame.isStreaming
```

isStreaming 속성은 Dataset이 데이터를 연속적으로 전달하는 데이터 소스일 때 true를 반환한다.

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/3eb77d0e-122a-4d67-977c-d3413d6ea0b5">

### 비즈니스 로직 적용

기존 DataFrame 처리와 동일한 비즈니스 로직을 적용해 총 판매 금액을 계산한다.

```scala
val purchaseByCustomerPerHour = streamingDataFrame
  .selectExpr(
    "CustomerId",
    "(UnitPrice * Quantity) as total_cost",
    "InvoiceDate")
  .groupBy(
    col("CustomerId"), window(col("InvoiceDate"), "1 day"))
  .sum("total_cost")
```

이 작업 역시 지연 연산(lazy operation)이므로 데이터 플로를 실행하기 위해 스트리밍 액션을 호출해야 한다.

### 스트리밍 액션

스트리밍 액션은 어딘가에 데이터를 채워 넣어야 하므로 **count 메서드와 같은 일반적인 정적 액션과는 다른 특성**을 가진다.

사용할 스트리밍 액션은 **트리거가 실행된 다음 데이터를 갱신하게 될 인메모리 테이블에 데이터를 저장**한다. 이번 예제의 경우 **파일마다 트리거를 실행**한다.

스파크는 **이전 집계값보다 더 큰 값이 발생한 경우에만 인메모리 테이블을 갱신**하므로 **언제나 가장 큰 값**을 얻을 수 있다.

```scala
purchaseByCustomerPerHour.writeStream
  .format("memory") // memory = 인메모리 테이블에 저장
  .queryName("customer_purchases") // 인메모리에 저장될 테이블명
  .outputMode("complete") // complete = 모든 카운트 수행 결과를 테이블에 저장
  .start()
```

스트림이 시작되면 쿼리 실행 결과가 어떠한 형태로 인메모리 테이블에 기록되는지 확인할 수 있다.

```scala
spark.sql("""
  SELECT *
  FROM customer_purchases
  ORDER BY `sum(total_cost)` DESC
  """)
  .show(5)
```

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/dcfad25e-3a84-4694-8724-27d0616394cf">

더 많은 데이터를 읽을수록 테이블 구성이 바뀐다는 것을 알 수 있다. 각 파일에 있는 데이터에 따라 결과가 변경될 수도 있고, 변경되지 않을 수도 있다.

이 예제는 고객을 그룹화하기 때문에 시간이 지남에 따라 일정 기간 최고로 많이 구매한 고객의 구매 금액이 증가할 것으로 기대할 수 있다. 또한 상황에 따라 처리 결과를 콘솔에 출력할 수 있다.

```scala
purchaseByCustomerPerHour.writeStream
  .format("console") // console = 콘솔에 결과 출력
  .queryName("customer_purchases_2")
  .outputMode("complete")
  .start()
```

예제에서 사용한 두 가지 방식(메모리나 콘솔로 출력, 파일별로 트리거를 수행)을 운영 환경에서 사용하는 것은 좋지 않지만 구조적 스트리밍을 경험해보기에는 충분하다.

스파크가 데이터를 처리하는 시점이 아닌 이벤트 시간에 따라 윈도우를 구성하는 방식에 주목할 필요가 있다. 이 방법을 사용하면 기존 스파크 스트리밍의 단점을 구조적 스트리밍으로 보완할 수 있다.

## 3.5 저수준 API

스파크는 RDD를 통해 자바와 파이선 객체를 다루는 데 필요한 다양한 기본 기능(저수준 API)을 제공한다. 또한 스파크의 거의 모든 기능은 RDD를 기반으로 만들어졌다.

- DataFrame 연산은 RDD 기반

- 편리하고 효율적인 분산 처리를 위해 저수준 명령으로 컴파일

- 원시 데이터를 읽거나 다루는 용도로 RDD 사용하나 대부분은 구조적 API를 사용하는 것이 좋음

  $\rightarrow$ 하지만 RDD 이용해 파티션과 같은 물리적 실행 특성 결정 가능해 DataFrame보다 더 세밀한 제어 가능

- 드라이버 시스템의 메모리에 저장된 원시 데이터 병렬 처리(parallelize)에 RDD 사용

- 스칼라, 파이썬 모두 RDD 사용 가능하나, 두 언어의 RDD가 동일하지 않음

- 언어와 관계없이 동일한 실행 특성 제공하는 DataFrame API와 달리 RDD는 세부 구현 방식에서 차이 존재

  $\rightarrow$ 낮은 버전의 스파크 코드를 계속 사용해야 하는 상황이 아니라면 RDD 사용할 필요 없다.  
  최신 버전의 스파크에서는 기본적으로 RDD를 사용하지 않지만, 비정형 데이터나 정제되지 않은 원시 데이터 처리 시 RDD를 사용

아래 코드는 간단한 숫자를 이용해 병렬화해 RDD를 생성하고, 다른 DataFrame과 함께 사용할 수 있도록 변환하는 예제이다.

```scala
spark.sparkContext.parallelize(Seq(1, 2, 3)).toDF()
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e23b732f-7c08-4540-b8ea-5153d4871439">
