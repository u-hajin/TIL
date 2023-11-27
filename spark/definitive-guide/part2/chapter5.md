## 목차

- 5.0 [구조적 API 기본 연산](#50-구조적-api-기본-연산)
- 5.1 [스키마(예제)](#51-스키마)
- 5.2 [컬럼과 표현식](#52-컬럼과-표현식)
- 5.2.1 [컬럼](#521-컬럼)
  - [명시적 컬럼 참조](#명시적-컬럼-참조)

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

<br/>

## 5.2 컬럼과 표현식

스파크의 **컬럼**은 **스프레드시트, R의 dataframe, Pandas의 DataFrame 컬럼과 유사**하다.  
사용자는 **표현식**으로 **DataFrame의 컬럼을 선택, 조작, 제거**할 수 있다.

스파크의 컬럼은 표현식을 사용해 **레코드 단위로 계산한 값을 단순하게 나타내는 논리적인 구조**이다. 따라서 **컬럼의 실젯값을 얻으려면 로우가 필요**하고, **로우를 얻으려면 DataFrame이 필요**하다.

DataFrame을 통하지 않으면 외부에서 컬럼에 접근할 수 없다. 컬럼 내용을 수정하려면 반드시 DataFrame의 스파크 트랜스포메이션을 사용해야 한다.

## 5.2.1 컬럼

컬럼을 생성하고 조작할 수 있는 여러 가지 방법이 있지만 **col, column 함수를 사용**하는 것이 가장 간단하다. 해당 함수들은 **컬럼명을 인수로** 받는다.

```scala
import org.apache.spark.sql.functions.{col, column}

col("someColumnName")
column("someColumnName")
```

<br/>

컬럼이 DataFrame에 있을지 없을지 알 수 없다. 컬럼은 컬럼명을 카탈로그에 저장된 정보와 비교하기 전까지 미확인 상태로 남는다. [**분석기가 동작하는 단계에서 컬럼과 테이블을 분석**](https://github.com/usuyn/TIL/blob/master/spark/definitive-guide/part2/chapter4.md#441-논리적-실행-계획)한다.

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/f0de0c23-fcd5-47fc-8f25-847befaf5127">

$\rightarrow$ 구조적 API의 논리적 실행 계획 수립 과정

<br/>

> NOTE\_ 앞서 컬럼을 참조하는 두 가지 방식을 알아보았다. 스칼라는 고유 기능을 사용해 다음과 같이 더 간단한 방법으로 컬럼을 참조할 수 있다. 하지만 코드가 짧다고 성능이 좋아지는 것은 아니다.
>
> $"myColumn"
> 'myColumn
>
> $를 사용하면 컬럼을 참조하는 특수한 문자열 표기를 만들 수 있다. 틱 마크(')는 **심벌**이라고도 불리는 특수 기호이다. 틱 마크는 특정 식별자를 참조할 때 사용하는 스칼라 고유의 기능이다.  
> 위 두 코드는 모두 같은 일을 처리하며 컬럼명으로 컬럼을 참조한다.

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d6128082-6925-4739-b12b-e75396227d11">

<br/>

### 명시적 컬럼 참조

DataFrame의 컬럼은 col 메서드로 참조한다. col 메서드는 조인 시 유용하다. 예를 들어 DataFrame의 어떤 컬럼을 다른 DataFrame의 조인 대상 컬럼에서 참조하기 위해 col 메서드를 사용한다.

col 메서드를 사용해 명시적으로 컬럼을 정의하면 스파크는 분석기 실행 단계에서 컬럼 확인 절차를 생략한다.

```scala
df.col("count")
```
