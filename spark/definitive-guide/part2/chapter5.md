## 목차

- 5.0 [구조적 API 기본 연산](#50-구조적-api-기본-연산)
- 5.1 [스키마(예제)](#51-스키마)
- 5.2 [컬럼과 표현식](#52-컬럼과-표현식)
- 5.2.1 [컬럼](#521-컬럼)
  - [명시적 컬럼 참조](#명시적-컬럼-참조)
- 5.2.2 [표현식](#522-표현식)
  - [표현식으로 컬럼 표현(예제)](#표현식으로-컬럼-표현)
  - [DataFrame 컬럼에 접근하기(예제)](#dataframe-컬럼에-접근하기)
- 5.3 [레코드와 로우](#53-레코드와-로우)
- 5.3.1 [로우 생성하기(예제)](#531-로우-생성하기)
- 5.4 [DataFrame의 트랜스포메이션](#54-dataframe의-트랜스포메이션)
- 5.4.1 [DataFrame 생성하기](#541-dataframe-생성하기)
- 5.4.2 [select와 selectExpr](#542-select와-selectexpr)
- 5.4.3 [스파크 데이터 타입으로 변환하기(예제)](#543-스파크-데이터-타입으로-변환하기)
- 5.4.4 [컬럼 추가하기(예제)](#544-컬럼-추가하기)

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

<br/>

## 5.2.2 표현식

DataFrame을 정의할 때 컬럼은 표현식이다.

**표현식**은 DataFrame 레코드의 여러 값에 대한 **트랜스포메이션 집합**을 의미한다.  
여러 **컬럼명을 입력으로 받아 식별**하고, **'단일 값'을 만들기 위해 다양한 표현식을 각 레코드에 적용하는 함수**라고 생각할 수 있다. 여기서 '단일 값'은 Map, Array 같은 복합 데이터 타입일 수 있다.

표현식은 expr 함수로 가장 간단히 사용할 수 있다. 이 함수를 사용해 DataFrame의 컬럼을 참조할 수 있다. 예로 expr("someCol")과 col("someCol") 구문은 동일하게 동작한다.

<br/>

### 표현식으로 컬럼 표현

컬럼은 표현식의 일부 기능을 제공한다. col() 함수를 호출해 컬럼에 트랜스포메이션을 수행하려면 반드시 컬럼 참조를 사용해야 한다.  
expr 함수의 인수로 표현식을 사용하면 표현식을 분석해 트랜스포메이션과 컬럼 참조를 알아낼 수 있으며, 다음 트랜스포메이션에 컬럼 참조를 전달할 수 있다.

- expr("someCol - 5")

- col("someCol") - 5

- expr("someCol") - 5

는 **모두 같은 트랜스포메이션 과정**을 거친다. 그 이유는 **스파크가 연산 순서를 지정하는 논리적 트리로 컴파일**하기 때문이다.

<br/>

아래 핵심 내용을 반드시 기억해야 한다.

- **컬럼은 단지 표현식**일 뿐이다.

- **컬럼과 컬럼의 트랜스포메이션은 파싱된 표현식과 동일한 논리적 실행 계획으로 컴파일**된다.

<br/>

예제와 함께 살펴본다.

```scala
(((col("someCol") + 5) * 200) - 6) < col("otherCol")
```

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/cd7e85b9-3a93-4f57-a7d7-7557b7af5fb6">

$\rightarrow$ 논리적 트리

위 그림이 어색하지 않은 이유는 **지향성 비순환 그래프**(**DAG**)이기 때문이다. 이 그래프는 다음 코드로 동일하게 표현할 수 있다.

```scala
import org.apache.spark.sql.functions.expr

expr("(((someCol + 5) * 200) - 6) < otherCol")
```

**SQL의 SELECT 구문**에 이전 표현식을 사용해도 잘 동작하며 **동일한 결과**를 생성한다. 그 이유는 **SQL 표현식과 위 예제의 DataFrame 코드는 실행 시점에 동일한 논리 트리로 컴파일**되기 때문이다.  
따라서 DataFrame 코드나 SQL로 표현식을 작성할 수 있으며 **동일한 성능을 발휘**한다.

<br/>

### DataFrame 컬럼에 접근하기

printSchema 메서드로 DataFrame의 전체 컬럼 정보를 확인할 수 있다. 하지만 프로그래밍 방식으로 컬럼에 접근할 때는 **DataFrame의 columns 속성을 사용**한다.

```scala
spark.read.format("json").load("./data/flight-data/json/2015-summary.json")
  .columns
```

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/5cf0a2d8-26e8-4170-a8f7-dd98b603f853">

<br/>

## 5.3 레코드와 로우

스파크에서 **DataFrame의 각 로우**는 **하나의 레코드**이다.  
스파크는 **레코드를 Row 객체로 표현**한다. 스파크는 값을 생성하기 위해 **컬럼 표현식으로 Row 객체를 다룬다.**

Row 객체는 **내부에 바이트 배열**을 가지며, 이 바이트 배열 인터페이스는 오직 **컬럼 표현식으로만** 다룰 수 있어 사용자에게 절대 노출되지 않는다.

DataFrame을 사용해 드라이버에게 개별 로우를 반환하는 명령은 **항상 하나 이상의 Row 타입을 반환**한다.

> NOTE\_ 이 장에서는 '로우'와 '레코드'를 같은 의미로 사용하면서 후자에 초점을 두었다. 대문자로 시작하는 Row는 Row 객체를 의미한다.

<br/>

DataFrame의 first 메서드로 로우를 확인할 수 있다.

```scala
df.first()
```

<br/>

## 5.3.1 로우 생성하기

- 각 컬럼에 해당하는 값을 통해 Row 객체를 직접 생성

- Row 객체는 스키마 정보를 가지고 있지 않고, DataFrame만 유일하게 스키마를 갖는다.

  $\rightarrow$ Row 객체 직접 생성을 위해 DataFrame의 스키마와 같은 순서로 값을 명시

```scala
import org.apache.spark.sql.Row

val myRow = Row("Hello", null, 1, false)
```

<br/>

- 로우의 데이터 접근 시 원하는 위치를 지정

- 스칼라, 자바 $\rightarrow$ 헬퍼 메서드 사용하거나 명시적으로 데이터 타입 지정

- 파이썬, R $\rightarrow$ 올바른 데이터 타입으로 자동 변환

```scala
// 스칼라 코드

myRow(0) // Any 타입
myRow(0).asInstanceOf[String] // String 타입
myRow(0).getString(0) // String 타입
myRow.getInt(2) // Int 타입
```

```python
# 파이썬 코드

myRow[0]
myRow[2]
```

<br/>

Dataset API를 사용하면 자바 가상 머신(Java Virtual Machine, JVM) 객체를 가진 데이터셋을 얻을 수 있다.

<br/>

## 5.4 DataFrame의 트랜스포메이션

DataFrame을 다루는 방법은 아래 그림에 표현한 것처럼 몇 가지 주요 작업으로 나눌 수 있다.

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/04d50688-5b76-48b6-b2c5-95e9e1feba8b">

<br/>

- 로우나 컬럼 추가

- 로우나 컬럼 제거

- 로우를 컬럼으로 변환하거나, 컬럼을 로우로 변환

- 컬럼값을 기준으로 로우 순서 변경

이 모든 유형의 작업은 트랜스포메이션으로 변환할 수 있다.  
가장 일반적인 트랜스포메이션은 모든 로우의 특정 컬럼값을 변경하고 그 결과를 반환하는 것이다.

<br/>

## 5.4.1 DataFrame 생성하기

원시 데이터 소스에서 DataFrame을 생성할 수 있다.  
생성한 DataFrame은 후반부에서 SQL 쿼리를 실행하고 SQL의 기본 트랜스포메이션을 확인하기 위해 임시 뷰로 등록한다.

```scala
// 원시 데이터 소스에서 DataFrame 생성

val df = spark.read.format("json")
  .load("./data/flight-data/json/2015-summary.json")

df.createOrReplaceTempView("dfTable")
```

<br/>

Row 객체를 가진 Seq 타입을 직접 변환해 DataFrame을 생성할 수 있다.

```scala
// Row 객체 가진 Seq 타입 변환해 DataFrame 생성

import org.apache.spark.sql.Row
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}

val myManualSchema = new StructType(Array(
  new StructField("some", StringType, true),
  new StructField("col", StringType, true),
  new StructField("names", LongType, false)))

val myRows = Seq(Row("Hello", null, 1L))
val myRDD = spark.sparkContext.parallelize(myRows)
val myDf = spark.createDataFrame(myRDD, myManualSchema)

myDf.show()


// Seq 데이터 타입에 toDF 함수 활용

val myDF = Seq(("Hello", 2, 1L)).toDF("col1", "col2", "col3")
```

<img width="700" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/9bd452e3-e947-41f9-9c9b-84f55a1b1152">

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/c147ff59-6d05-4377-8016-217014457ab7">

<br/>

> NOTE\_ 스칼라 버전의 스파크 콘솔을 사용하는 경우 Seq 데이터 타입에 toDF 함수를 활용할 수 있어 스파크의 implicits가 주는 장점을 얻을 수 있다. 하지만 implicits는 null 타입과 잘 맞지 않아 실제 운영 환경에서 사용하는 것은 권장하지 않는다.  
> implicits : 기본 스칼라 객체를 DataFrame으로 변환하는데 사용

<br/>

DataFrame을 만드는 방법을 알아보았다. 이제 아래와 같이 가장 유용하게 사용할 수 있는 메서드를 알아본다.

- **컬럼이나 표현식을 사용**하는 **select** 메서드

- **문자열 표현식을 사용**하는 **selectExpr** 메서드

- 메서드로 사용할 수 없는 **org.apache.spark.sql.functions 패키지**에 포함된 다양한 함수

이 세 가지 유형의 메서드로 DataFrame을 다룰 때 필요한 **대부분의 트랜스포메이션 작업을 해결**할 수 있다.

<br/>

## 5.4.2 select와 selectExpr

select와 selectExpr 메서드를 사용하면 데이터 테이블에 SQL을 실행하는 것처럼 DataFrame에서도 SQL을 사용할 수 있다.

다시 말해, **DataFrame의 컬럼을 다룰 때 SQL을 사용**할 수 있다.  
DataFrame을 사용한 몇 가지 예제를 살펴보면서 컬럼을 다루는 여러 가지 방법을 알아본다.

### 문자열 컬럼명을 인수로 받는 select

```scala
df.select("DEST_COUNTRY_NAME").show(2)

// SQL
SELECT DEST_COUNTRY_NAME
FROM dfTable
LIMIT 2;
```

<img width="250" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/1087c7ad-d49d-4ce2-982b-f2d482823fcb">

### 여러 컬럼을 선택

```scala
df.select("DEST_COUNTRY_NAME", "ORIGIN_COUNTRY_NAME").show(2)

// SQL
SELECT DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME
FROM dfTable
LIMIT 2
```

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/889670f0-83a9-41b4-8b6b-98ed6b1bede7">

### [다양한 컬럼 참조 방법](#52-컬럼과-표현식) 함께 사용

```scala
import org.apache.spark.sql.functions.{expr, col, column}

df.select(
    df.col("DEST_COUNTRY_NAME"),
    col("DEST_COUNTRY_NAME"),
    column("DEST_COUNTRY_NAME"),
    'DEST_COUNTRY_NAME,
    $"DEST_COUNTRY_NAME",
    expr("DEST_COUNTRY_NAME"))
  .show(2)
```

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/86eeffae-17db-4a53-b154-4d8ff1623738">

<br/>

Column 객체와 문자열을 함께 섞어 쓰는 것은 컴파일러 오류를 발생시키므로 유의해야 한다.

```scala
df.select(
  col("DEST_COUNTRY_NAME"),
  "DEST_COUNTRY_NAME"
)
```

### expr 함수

expr 함수는 가장 유연한 참조 방법이다. expr 함수는 단순 컬럼 참조나 문자열을 이용해 컬럼을 참조할 수 있다.

설명을 위해 AS 키워드로 컬럼명을 변경한 다음 alias 메서드로 원래 컬럼명으로 되돌린다.

```scala
df.select(expr("DEST_COUNTRY_NAME AS destination")).show(2)

// SQL
SELECT DEST_COUNTRY_NAME as destination
FROM dfTable
LIMIT 2
```

위 코드는 컬럼명을 'destination'으로 변경한다. 표현식의 결과를 다른 표현식으로 다시 처리할 수 있다.

<br/>

```scala
df.select(expr("DEST_COUNTRY_NAME as destination")
  .alias("DEST_COUNTRY_NAME"))
  .show(2)
```

위 코드는 변경한 컬럼명을 원래 이름으로 되돌려 놓는다.

### selectExpr

select 메서드에 expr 함수를 사용하는 패턴을 자주 사용한다. 스파크는 이런 작업을 간단하고 효율적으로 할 수 있는 selectExpr 메서드를 제공한다.

```scala
df.selectExpr("DEST_COUNTRY_NAME as newColumnName", "DEST_COUNTRY_NAME").show(2)
```

<img width="550" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d1fa4dc2-cf04-4148-8e15-72007e78daf5">

<br/>

selectExpr 메서드는 새로운 DataFrame을 생성하는 복잡한 표현식을 간단하게 만드는 도구이다.  
사실 모든 유효한 비집계형(non-aggregating) SQL 구문을 지정할 수 있다. 단, 컬럼을 식별할 수 있어야 한다.

다음 코드는 DataFrame에 출발지와 도착지가 같은지 나타내는 새로운 withinCountry 컬럼을 추가하는 예제이다.

```scala
df.selectExpr(
    "*",  // 모든 원본 컬럼 포함
    "(DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry")
  .show(2)

// SQL
SELECT *, (DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry
FROM dfTable
LIMIT 2
```

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/f2f15d8b-cf21-40d3-ae48-87e1c9fb822b">

<br/>

select 표현식에는 DataFrame의 컬럼에 대한 집계 함수를 지정할 수 있다.

```scala
df.selectExpr("avg(count)", "count(distinct(DEST_COUNTRY_NAME))").show(2)

// SQL
SELECT avg(count), count(distinct(DEST_COUNTRY_NAME))
FROM dfTable
LIMIT 2
```

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/ebb03868-baae-4af2-9b6e-3cc4e732e337">

<br/>

## 5.4.3 스파크 데이터 타입으로 변환하기

떄로는 새로운 컬럼이 아닌 명시적인 값을 스파크에 전달해야 한다. 명시적인 값은 상숫값일 수 있고, 추후 비교에 사용할 무언가가 될 수도 있다. 이때 **리터럴**(**literal**)을 사용한다.

리터럴은 **프로그래밍 언어의 리터럴값을 스파크가 이해할 수 있는 값으로 변환**한다. 리터럴은 **표현식**이며 이전 예제와 같은 방식으로 사용한다.

```scala
import org.apache.spark.sql.functions.lit

df.select(expr("*"), lit(1).as("One")).show(2)

// SQL
SELECT *, 1 as One
FROM dfTable
LIMIT 2
```

SQL에서 리터럴은 상숫값을 의미한다.

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/475500f7-44af-4b5b-901b-b35c76c19d6e">

<br/>

어떤 상수나 프로그래밍으로 생성된 변숫값이 **특정 컬럼의 값보다 큰지 확인할 때 리터럴을 사용**한다.

<br/>

## 5.4.4 컬럼 추가하기

DataFrame에 신규 컬럼을 추가하는 공식적인 방법은 DataFrame의 withColumn 메서드를 사용하는 것이다.

숫자 1을 값으로 가지는 컬럼을 추가하는 예제이다.

```scala
df.withColumn("numberOne", lit(1)).show(2)

// SQL
SELECT *, 1 as numberOne
FROM dfTable
LIMIT 2
```

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/27327f24-54fe-4c2c-878d-16dfe22345ca">

<br/>

출발지와 도착지가 같은지 여부를 불리언 타입으로 표현하는 예제이다.

```scala
df.withColumn("withinCountry", expr("ORIGIN_COUNTRY_NAME == DEST_COUNTRY_NAME"))
  .show(2)
```

<img width="350" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/991cd850-104f-423e-b6df-77e2f589b098">

<br/>

withColumn 메서드는 컬럼명, 값을 생성할 표현식 두 개의 인수를 사용한다.

또한 withColumn 메세드로 컬럼명을 변경할 수도 있다.

```scala
df.withColumn("Destination", expr("DEST_COUNTRY_NAME")).columns
```

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/bdf7d4ce-02b3-4607-a8ee-8121b9a420bb">

<br/>

위 예제에서 withColumn 메서드의 첫 번째 인수로 새로운 컬럼명을 나타내는 문자열을 지정했기 때문에 이스케이프 문자가 필요 없다.

다음 예제에서는 표현식으로 컬럼을 참조하므로 백틱(`) 문자를 사용한다.

```scala
dfWithLongColName.selectExpr(
  "`This Long Column-Name`",
  "`This Long Column-Name` as `new col`")
  .show(2)

dfWithLongColName.createOrReplaceTempView("dfTableLong")

// SQL
SELECT `This Long Column-Name`, `This Long Column-Name` as `new col`
FROME dfTableLong
LIMIT 2
```

표현식 대신 문자열을 사용해 명시적으로 컬럼을 참조하면 리터럴로 해석되기 때문에 예약 문자가 포함된 컬럼을 참조할 수 있다.  
예약 문자나 키워드를 사용하는 표현식에만 이스케이프 처리가 필요하다.

아래 두 코드는 모두 같은 DataFrame을 만든다.

```scala
// 스칼라 코드
dfWithLongColName.select(col("This Long Column-Name")).columns
```

```python
# 파이썬 코드
dfWithLongColName.select(expr("`This Long Column-Name`")).columns
```
