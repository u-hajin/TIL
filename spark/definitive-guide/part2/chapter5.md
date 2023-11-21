## 목차

- 5.0 [구조적 API 기본 연산](#50-구조적-api-기본-연산)

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

<img width="600" alt="image" src="https://github.com/usuyn/TIL/assets/68963707/e1131d1e-47f9-4bb8-8af0-81d79ca81678">

DataFrame은 컬럼을 가지며 스키마로 컬럼을 정의한다.  
스키마는 관련된 모든 것을 하나로 묶는 역할을 한다.
