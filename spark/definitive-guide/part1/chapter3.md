## 목차

- 3.0 [스파크 기능 둘러보기](#30-스파크-기능-둘러보기)
- 3.1 [운영용 애플리케이션 실행하기(예제)](#31-운영용-애플리케이션-실행하기)
  - [pi 계산](#pi-계산)
- 3.2 [Dataset : 타입 안정성을 제공하는 구조적 API(예제)](#32-dataset--타입-안정성을-제공하는-구조적-api)
  - [장점](#장점)

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
