## 목차

- 5.0 [구조적 API 기본 연산](#50-구조적-api-기본-연산)
- 5.1 [스키마(예제)](#51-스키마)

<br/>

## 5.0 구조적 API 기본 연산

이 장에서는 아키텍처 개념에서 벗어나 DataFrame과 DataFrame의 데이터를 다루는 기능을 소개한다. 특히 DataFrame의 기본 기능은 중점적으로 다룬다.

- **DataFrame** : Row 타입의 **레코드**와 각 레코드에 수행할 연산 표현식을 나타내는 여러 **컬럼**으로 구성

- **스키마** : 각 컬렴명과 데이터 타입을 정의

- DataFrame의 **파티셔닝** : DataFrame, Dataset이 클러스터에서 물리적으로 배치되는 형태를 정의

- **파티셔닝 스키마** : 파티션을 배치하는 방법을 정의

- **파티셔닝의 분할 기준** : 특정 컬럼이나 비결정론적(nondeterministically, = 매번 변하는)값을 기반으로 설정 가능

<br/>

```scala
val df = spark.read.format("json")
  .load("./data/flight-data/json/2015-summary.json")

df.printSchema()
```

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e1131d1e-47f9-4bb8-8af0-81d79ca81678">

DataFrame은 컬럼을 가지며 스키마로 컬럼을 정의한다.  
스키마는 관련된 모든 것을 하나로 묶는 역할을 한다.

<br/>

## 5.1 스키마

스키마는 DataFrame의 컬럼명과 데이터 타입을 정의한다. 데이터 소스에서 스키마를 얻거나 직접 정의할 수 있다.

> CAUTION\_ 데이터를 읽기 전에 스키마를 정의해야 하는지 여부는 상황에 따라 달라진다. 비정형 분석(ad-hoc analysis)에서는 스키마-온-리드가 대부분 잘 작동한다.(단, CSV, JSON 같은 일반 텍스트 파일 사용 시 느릴 수 있음) 하지만 Long 데이터 타입을 Integer 데이터 타입으로 잘못 인식하는 등 정밀도 문제가 발생할 수 있다.
>
> 따라서 운영 환경에서 추출(Extract), 변환(Transform), 적재(Load)를 수행하는 ETL 작업에 스파크를 사용한다면 직접 스키마를 정의해야 한다. ETL 작업 중에 데이터 타입을 알기 힘든 CSV나 JSON 등의 데이터 소스를 사용하는 경우 스키마 추론 과정에서 읽어 들인 샘플 데이터의 타입에 따라 스키마를 결정해버릴 수 있다.

<br/>

예제에서는 미국 교통통계국이 제공하는 [항공 운항 데이터](https://bit.ly/2yw2fCx)를 사용하며, 줄로 구분된 반정형 JSON 데이터이다.

```scala
spark.read.format("json").load("./data/flight-data/json/2015-summary.json").schema
```

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/411f5928-3123-4dc1-9f49-4dd80d927f07">

<br/>

스키마는 여러 개의 **StructField 타입 필드로 구성된 StructType 객체**이다. **StructField**는 **이름, 데이터 타입, 컬럼이 값이 없거나 null일 수 있는지 지정하는 불리언값**을 가진다.  
필요한 경우 컬럼과 관련된 메타데이터를 지정할 수 있다. 메타데이터는 해당 컬럼과 관련된 정보이며 스파크의 머신러닝 라이브러리에서 사용한다.

스키마는 복합 데이터 타입인 StructType을 가질 수 있다. 스파크는 런타임에 데이터 타입이 스키마의 데이터 타입과 일치하지 않으면 오류를 발생시킨다.

다음 코드는 DataFrame에 스키마를 만들고 적용하는 예제이다.

```scala
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}
import org.apache.spark.sql.types.Metadata

val myManualSchema = StructType(Array(
  StructField("DEST_COUNTRY_NAME", StringType, true),
  StructField("ORIGIN_COUNTRY_NAME", StringType, true),
  StructField("count", LongType, false,
    Metadata.fromJson("{\"hello\":\"world\"}"))
))

val df = spark.read.format("json").schema(myManualSchema)
  .load("./data/flight-data/json/2015-summary.json")
```

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/92ed7de4-f27c-4915-9c26-67555eb5729e">

<br/>

스파크는 자체 데이터 타입 정보를 사용하므로 프로그래밍 언어의 데이터 타입을 스파크의 데이터 타입으로 설정할 수 없다.
