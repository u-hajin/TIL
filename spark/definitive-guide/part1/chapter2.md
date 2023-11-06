## 목차

- 2.1 [스파크의 기본 아키텍처](#21-스파크의-기본-아키텍처)
- 2.1.1 [스파크 애플리케이션](#211-스파크-애플리케이션)
  - [Driver](#driver)
  - [Executor](#executor)
  - [스파크 애플리케이션 이해를 위한 핵심사항](#스파크-애플리케이션-이해를-위한-핵심사항)
- 2.2 [스파크의 다양한 언어 API](#22-스파크의-다양한-언어-api)
  - [SparkSession과 스파크 언어 API 간의 관계](#sparksession과-스파크-언어-api-간의-관계)
- 2.3 [스파크 API](#23-스파크-api)
- 2.4 [스파크 시작하기](#24-스파크-시작하기)
- 2.5 [SparkSession](#25-sparksession)
  - [일정 범위 숫자 생성(예제)](#일정-범위-숫자-생성)
- 2.6 [DataFrame](#26-dataframe)
- 2.6.1 [파티션](#261-파티션)
- 2.7 [트랜스포메이션(예제)](#27-트랜스포메이션)
  - [Narrow Dependency](#narrow-dependency)
  - [Wide Dependency](#wide-dependency)
- 2.7.1 [지연 연산](#271-지연-연산)
- 2.8 [액션(예제)](#28-액션)
- 2.9 [스파크 UI](#29-스파크-ui)
  - [접속 방법](#접속-방법)
- 2.10 [종합 예제](#210-종합-예제)
  - [데이터 읽기](#데이터-읽기)
  - [데이터 정렬](#데이터-정렬)
  - [실행 계획 확인](#실행-계획-확인)
  - [액션 호출](#액션-호출)
- 2.10.1 [DataFrame과 SQL](#2101-dataframe과-sql)
  - [테이블, 뷰 생성](#테이블-뷰-생성)
  - [SQL 쿼리 실행](#sql-쿼리-실행)
  - [다양한 데이터 처리 기능](#다양한-데이터-처리-기능)
  - [특정 위치를 왕래하는 최대 비행 횟수](#특정-위치를-왕래하는-최대-비행-횟수)
  - [상위 5개의 도착 국가](#상위-5개의-도착-국가)

## 2.1 스파크의 기본 아키텍처

컴퓨터 클러스터는 여러 컴퓨터의 자원을 모아 하나의 컴퓨터처럼 사용할 수 있게 만든다.  
하지만 컴퓨터 클러스터를 구성하는 것만으로는 부족하며 클러스터에서 작업을 조율할 수 있는 프레임워크인 스파크가 필요하다.

**스파크는 클러스터의 데이터 처리 작업을 관리하고 조율**한다.

스파크가 연산에 사용할 클러스터는

- 스파크 standalone
- 하둡 YARN
- Mesos

위와 같은 클러스터 매니저에서 관리한다.

### 작업 처리 과정

1. 클러스터 매니저에 스파크 애플리케이션 **submit**

2. 클러스터 매니저는 애플리케이션 실행에 필요한 **자원 할당**

3. 할당받은 자원으로 **작업 처리**

## 2.1.1 스파크 애플리케이션

스파크 애플리케이션 $\rightarrow$ dirver(드라이버) 프로세스 + 다수의 executor(익스큐터) 프로세스로 구성

### Driver

- 클러스터 노드 중 하나에서 실행됨

- main() 함수 실행

- 스파크 애플리케이션 정보 유지 관리

- 사용자 프로그램이나 입력에 대한 응답

- 전박적인 익스큐터 프로세스의 작업 관련 분석 및 배포

- 스케줄링

스파크 애플리케이션의 심장과 같은 존재로 애플리케이션의 수명 주기 동안 관련 정보를 모두 유지한다.

### Executor

- 드라이버 프로세스가 할당한 작업(코드) 실행

- 진행 상황을 드라이버 노드에 보고

아래 이미지는 클러스터 매니저가 물리적 머신을 관리하고 스파크 애플리케이션에 자원을 할당하는 방법을 나타낸다.

<img width="350" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/4420468f-de96-4941-a2ea-51327438045a">

$\rightarrow$ 스파크 애플리케이션의 아키텍처

클러스터 매니저는

- 스파크 standalone
- 하둡 YARN
- Mesos

중 하나를 선택할 수 있다.

하나의 클러스터에서 여러개의 스파크 애플리케이션을 실행할 수 있다.

위 이미지에서 왼쪽에 드라이버 프로세스가 있고 오른쪽에 4개의 익스큐터가 있다. 사용자는 각 노드에 할당할 익스큐터 수를 지정할 수 있다. (이미지에는 클러스터 노드의 개념을 나타내지 않음.)

### 스파크 애플리케이션 이해를 위한 핵심사항

- 스파크는 사용 가능한 자원을 파악하기 위해 클러스터 매니저 사용

- 드라이버 프로세스는 주어진 작업 완료를 위해 드라이버 프로그램의 명령을 익스큐터에서 실행할 책임이 있다.

익스큐터는 대부분 스파크 코드를 실행하는 역할을 한다. 하지만 드라이버는 스파크의 언어 API를 통해 다양한 언어로 실행할 수 있다.

## 2.2 스파크의 다양한 언어 API

스파크의 언어 API를 이용해 다양한 프로그래밍 언어로 스파크 코드를 실행할 수 있다.  
스파크는 모든 언어에 맞는 '핵심 개념'을 제공한다. 이러한 핵심 개념은 클러스터 머신에서 실행되는 스파크 코드로 변환된다.

구조적 API만으로 작성된 코드는 언어에 상관없이 유사한 성능을 발휘한다.

- **스칼라**

  $\rightarrow$ 스파크는 스칼라로 개발되어 있으므로 스칼라가 스파크의 기본 언어이다. 책에서는 스칼라 예제를 대부분 제공한다.

- **자바**

  $\rightarrow$ 스파크 창시자들은 자바를 이용해 스파크 코드를 작성할 수 있도록 했다.

- **파이썬**

  $\rightarrow$ 스칼라가 지원하는 거의 모든 구조를 지원한다.

- **SQL**

  $\rightarrow$ 스파크는 ANSI SQL:2003 표준 중 일부를 지원한다.

- **R**

  $\rightarrow$ 스파크에는 일반적으로 사용하는 2개의 R 라이브러리가 존재한다. 하나는 스파크 코어에 포함된 SparkR이고, 다른 하나는 R 커뮤니티 기반 패키지인 sparklyr이다.

### SparkSession과 스파크 언어 API 간의 관계

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/7d903c44-b864-4aaa-9772-4779742ae0e7">

$\rightarrow$ SparkSession과 스파크 언어 API 간의 관계

사용자는 스파크 코드를 실행하기 위해 SparkSession 객체를 진입점으로 사용할 수 있다.

스파크가 파이썬이나 R로 작성한 코드를 익스큐터의 JVM에서 실행할 수 있는 코드로 변환 $\rightarrow$ 사용자는 JVM 코드를 명시적으로 작성하지 않는다.

## 2.3 스파크 API

**저수준의 비구조적(unstructured) API**, **고수준의 구조적(structured) API**를 스파크가 기본적으로 제공 $\rightarrow$ 다양한 언어로 스파크 사용 가능

## 2.4 스파크 시작하기

실제 스파크 애플리케이션을 개발하려면 **사용자 명령**과 **데이터**를 스파크 애플리케이션에 **전송**하는 방법을 알아야 한다.

대화형 모드로 스파크를 시작하면 스파크 애플리케이션을 관리하는 SparkSession이 자동으로 생성된다.

하지만 스탠드얼론 애플리케이션으로 스파크를 시작하려면 사용자 애플리케이션 콛에서 SparkSession 객체를 직접 생성해야 한다.

예제 실행을 위해 [스칼라 콘솔 실행](https://github.com/usuyn/TIL/blob/master/spark/definitive-guide/part1/chapter1.md#스칼라-콘솔-실행하기)을 참고해 대화형 세션을 시작한다.

## 2.5 SparkSession

- 스파크 애플리케이션은 SparkSession이라 불리는 드라이버 프로세스로 제어

- SparkSession 인스턴스는 사용자가 정의한 처리 명령을 클러스터에서 실행

- 하나의 SparkSession은 하나의 스파크 애플리케이션에 대응

스칼라와 파이썬 콘솔을 실행하면 spark 변수로 SparkSession을 사용할 수 있다.

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/3f3d521e-9a0e-4d07-bfa4-e736a77fb339">

$\rightarrow$ 스칼라에서 SparkSession 확인

### 일정 범위 숫자 생성

일정 범위의 숫자를 만드는 간단한 작업을 수행한다. 이 숫자들은 스프레드시트에서 컬럼명을 지정한 것과 같다.

```scala
val myRange = spark.range(1000).toDF("number")
```

$\rightarrow$ 스칼라 코드

```python
myRange = spark.range(1000).toDF("number")
```

$\rightarrow$ 파이썬 코드

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/693e7a34-875d-4223-8466-c760d23e6acd">

생성한 DataFrame은 한 개의 컬럼(number)과 1,000개의 로우로 구성되며 각 로우에 0부터 999까지의 값이 할당되어 있다.

이 숫자들은 **분산 컬렉션**을 나타낸다.

클러스터 모드에서 코드 예제를 실행하면 숫자 범위의 각 부분이 서로 다른 익스큐터에 할당된다.

## 2.6 DataFrame

- 가장 대표적인 **구조적 API**

- 테이블의 데이터를 로우와 컬럼으로 단순하게 표현

- 컬럼과 컬럼의 타입을 정의한 목록 $\rightarrow$ **스키마(schema)**

컬럼에 이름을 붙인 스프레드시트와 비슷하나 아래와 같은 차이점이 존재한다.

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/ee17fb23-081f-41b0-aa7e-61502b9676ac">

$\rightarrow$ 분산 컴퓨터와 단일 컴퓨터 분석의 차이점

스프레드시트는 한 대의 컴퓨터에 있지만, 스파크 DataFrame은 수천 대의 컴퓨터에 분산되어 있다.

단일 컴퓨터에 저장하기에는 데이터가 너무 크거나 계산에 오랜 시간 소요 $\rightarrow$ 여러 컴퓨터에 데이터를 분산

DataFrame은 스파크에서만 사용하는 개념이 아니다. 파이썬과 R 모두 비슷한 개념을 가지고 있으나 일반적(예외사항 존재)으로 분산 컴퓨터가 아닌 단일 컴퓨터에 존재한다.  
이러한 상황에서는 DataFrame으로 수행할 수 있는 작업이 해당 머신이 가진 자원에 따라 제한된다.

스파크는 파이썬과 R 언어를 모두 지원한다. 따라서

파이썬 Pandas 라이브러리 DataFrame, R의 DataFrame $\rightarrow$ 스파크 DataFrame으로 쉽게 변환 가능

스파크는 분산 데이터 모음을 표현하는 Dataset, DataFrame, SQL 테이블, RDD라는 핵심 추상화 개념을 가지고 있다. 이중 DataFrame은 가장 쉽고 효율적이다.

## 2.6.1 파티션

스파크는 모든 익스큐터가 병렬로 작업을 수행할 수 있도록 파티션(청크 단위)으로 데이터를 분할한다.  
파티션은 클러스터의 물리적 머신에 존재하는 로우의 집합을 의미한다.

DataFrame의 파티션은 실행 중에 데이터가 컴퓨터 클러스터에서 물리적으로 분산되는 방식을 나타낸다.

- 파티션이 하나라면 수천 개의 익스큐터가 있더라도 병렬성 1

- 수백 개의 파티션이 있더라도 익스큐터가 하나밖에 없다면 병렬성 1

물리적 파티션에 데이터 변환용 함수를 지정하면 스파크가 실제 처리 방법 결정 $\rightarrow$ DataFrame 사용 시 파티션을 수동, 개별적으로 처리할 필요 없음

## 2.7 트랜스포메이션

스파크의 핵심 데이터 구조는 불변성(immutable)을 가진다. 즉, 한번 생성하면 변경할 수 없다.

DataFrame을 변경하려면 **트랜스포메이션**이라 불리는 명령을 사용해 원하는 변경 방법을 스파크에 알려줘야 한다.

아래 코드는 DataFrame에서 짝수를 찾는 간단한 트랜스포메이션 예제이다.

```scala
val divisBy2 = myRange.where("number % 2 = 0")
```

$\rightarrow$ 스칼라 코드

```python
divisBy2 = myRange.where("number % 2 = 0")
```

$\rightarrow$ 파이썬 코드

위 코드를 실행하면 응답값이 출력되지만 결과는 출력되지 않는다.  
추상적인 트랜스포메이션만 지정한 상태이기 때문에 액션(action)을 호출하지 않으면 스파크는 실제 트랜스포메이션을 수행하지 않는다.

트랜스포메이션은 스파크에서 비즈니스 로직을 표현하는 핵심 개념이다.  
두 가지 유형이 존재한다.

- Narrow Dependency, 좁은 의존성

- Wide Dependency, 넓은 의존성

### Narrow Dependency

- 좁은 의존성을 가진 트랜스포메이션 $\rightarrow$ 좁은 트랜스포메이션

- 각 입력 파티션이 하나의 출력 파티션에만 영향을 미침

- 좁은 트랜스포메이션 사용 시 스파크가 파이프라이닝(pipelining) 자동 수행

- DataFrame에 여러 필터 지정하는 경우 모든 작업이 메모리에서 발생

- 예제 코드에서 where 구문은 좁은 의존성을 가짐

  <img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/ef2dcab7-02ac-44db-96bf-f1a04d5b7585">

  $\rightarrow$ 좁은 의존성, 하나의 파티션이 하나의 출력 파티션에만 영향

### Wide Dependency

- 넓은 의존성을 가진 트랜스포메이션 $\rightarrow$ 넓은 트랜스포메이션

- 하나의 입력 파티션이 여러 출력 파티션에 영향을 미친다.

- 클러스터에서 파티션을 교환 $\rightarrow$ 셔플(shuffle)

- 스파크는 셔플의 결과를 디스크에 저장

  <img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/929f9e54-e7ca-426c-aa86-84c32eaa23de">

  $\rightarrow$ 넓은 의존성, 하나의 파티션이 여러 출력 파티션에 영향

## 2.7.1 지연 연산

- 지연 연산(lazy evaluation)은 스파크가 연산 그래프를 처리하기 직전까지 기다리는 동작 방식

- 특정 연산 명령이 내려진 즉시 데이터를 수정하지 않고 원시 데이터에 적용할 트랜스포메이션의 **실행 계획**을 생성

- 코드를 실행하는 마지막 순간까지 대기, 원형 DataFrame 트랜스포메이션을 간결한 물리적 실행 계획으로 컴파일

- 위 과정을 거치며 전체 데이터 흐름을 최적화

- DataFrame의 조건절 푸시다운(predicate pushdown)이 한 예

복잡한 스파크 잡이 원시 데이터에서 하나의 로우만 가져오는 필터를 가지고 있다면 $\rightarrow$ 필요한 레코드 하나만 읽는 것이 가장 효율적

스파크는 이 필터를 데이터 소스로 위임하는 최적화 작업을 자동으로 수행  
$\rightarrow$ 데이터 저장소가 데이터베이스라면 WHERE 절의 처리를 데이터베이스에 위임, 스파크는 하나의 레코드만 받는다.  
$\rightarrow$ 처리에 필요한 자원 최소화

## 2.8 액션

사용자는 트랜스포메이션을 사용해 논리적 실행 계획을 세운다.

하지만 실제 연산을 수행하려면 **액션** 명령을 내려야 한다.  
$\rightarrow$ 일련의 트랜스포메이션으로부터 결과를 계산하도록 지시하는 명령

가장 단순한 액션인 count 메서드는 DataFrame의 전체 레코드 수를 반환한다.

```scala
divisBy2.count()
```

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/648080a0-e22e-4218-9724-3aba5f04d2ed">

count 외에도 세 가지 유형의 액션이 존재한다.

- 콘솔에서 데이터를 보는 액션

- 각 언어로 된 네이티브 객체에 데이터를 모으는 액션

- 출력 데이터 소스에 저장하는 액션

액션을 지정하면 스파크 잡(job)이 시작된다.

스파크 잡은 필터(좁은 트랜스포메이션)를 수행한 후 파티션별로 레코드 수를 카운트(넓은 트랜스포메이션) 한다. 그리고 각 언어에 적합한 네이티브 객체에 결과를 모은다.

이때 스파크가 제공하는 스파크 UI로 클러스터에서 실행 중인 스파크 잡을 모니터링할 수 있다.

필터(좁은 트랜스포메이션) 수행 $\rightarrow$ 파티션별로 레코드 수 카운트(넓은 트랜스포메이션) $\rightarrow$ 네이티브 객체에 결과 수집

## 2.9 스파크 UI

스파크 잡의 진행 상황을 모니터링할 때 사용한다. 드라이버 노드의 4040 포트로 접속할 수 있으며, 로컬 모드에서 스파크를 실행 시 주소는 http://localhost:4040 이다.

스파크 잡의 상태, 환경 설정, 클러스터 상태 등의 정보를 확인할 수 있다. 스파크 UI를 사용해 잡을 튜닝하고 디버깅한다.

현재까지는

- 스파크 잡은 개별 액션에 의해 트리거되는 다수의 트랜스포메이션으로 구성

- 스파크 UI로 잡 모니터링 가능

만 기억하면 된다.

### 접속 방법

아래 명령어를 통해 스칼라 콘솔을 실행한다.

```scala
./bin/spark-shell
```

4040 포트에서 바인드가 불가하면 1씩 증가시키며 가능한 포트 번호를 찾는 것을 확인할 수 있다.

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/49fa9ce1-84f4-4f5d-aa41-b1316ea85ab3">

명시된 주소로 스파크 UI에 접속할 수 있다.

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/dc5fc6de-098c-4929-89fd-b8f31a249783">

## 2.10 종합 예제

현실적인 예제로 지금까지 배운 내용을 확실히 이해하고, 스파크 내부에서 어떤 일이 일어나는지 단계별로 살펴본다.

미국 교통 통계국의 [항공 운항 데이터](https://bit.ly/2yw2fCx) 중 일부를 스파크로 분석한다.

예제에서는 CSV 파일만 사용한다.

각 파일은 여러 로우를 가진다. 반정형(semi-structured) 데이터 포맷으로 파일의 각 로우는 DataFrame의 로우가 된다.

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/808511cb-3eef-4e6a-afdf-1be68dad2462">

### 데이터 읽기

스파크는 다양한 데이터 소스를 지원한다. 데이터는 SparkSession의 DataFrameReader 클래스를 사용해 읽는다. 이때 특정 파일 포맷과 몇 가지 옵션을 함께 설정한다.

예제에서는 스파크 DataFrame의 스키마 정보를 알아내는 **스키마 추론(schema inference)** 기능을 사용하며, 파일의 첫 로우를 헤더로 지정하는 옵션을 설정한다.

스파크는 스키마 정보를 얻기 위해 데이터를 조금 읽고 해당 로우의 데이터 타입을 스파크 데이터 타입에 맞게 분석한다.  
하지만 운영 환경에서는 데이터를 읽는 시점에 스키마를 엄격하게 지정하는 옵션을 사용해야 한다.

```scala
val flightData2015 = spark
  .read
  .option("inferSchema", "true")
  .option("header", "true")
  .csv("./data/flight-data/csv/2015-summary.csv")
```

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d7c2c765-415a-4cde-8314-71f8789e0a06">

스칼라와 파이썬에서 사용하는 DataFrame은 불특정 다수의 로우와 컬럼을 가진다. 로우의 수를 알 수 없는 이유는 테이터를 읽는 과정이 지연 연산 형태의 트랜스포메이션이기 때문이다.

스파크는 각 컬럼의 데이터 타입을 추론하기 위해 적은 양의 데이터를 읽는다.

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/377c07f3-89bd-4ab7-8f09-a625cf04ea70">

$\rightarrow$ DataFrame에서 CSV 파일을 읽어 로컬 배열이나 리스트 형태로 변환하는 과정

DataFrame의 **take 액션**을 호출하면 이전의 head 명령과 같은 결과를 확인할 수 있다.

```scala
flightData2015.take(3)
```

<img width="800" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e95d6922-d11b-44d9-a42f-e600cfa0e4b3">

### 데이터 정렬

트랜스포메이션을 추가로 지정한다. 정수 데이터 타입인 count 컬럼을 기준으로 데이터를 정렬한다.

정렬을 위해 사용하는 sort 메서드는 DataFrame을 변경하지 않는다. 트랜스포메이션으로 sort 메서드를 사용하면 이전의 DataFrame을 변환해 새로운 DataFrame을 생성해 반환한다.

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/51b26b87-4220-4e74-980e-2c93c4245df6">

$\rightarrow$ DataFrame을 이용한 데이터 읽기, 정렬, 수집

sort 메서드는 단지 트랜스포메이션이기 때문에 호출 시 데이터에 아무런 변화도 일어나지 않는다. 스파크는 실행 계획을 만들고 검토하여 클러스터에서 처리할 방법을 알아낸다.

### 실행 계획 확인

특정 DataFrame 객체에 explain 메서드를 호출하면 DataFrame의 계보(lineage)나 스파크의 쿼리 실행 계획을 확인할 수 있다.

```scala
flightData2015.sort("count").explain()
```

<img width="800" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/10c136c3-02c2-4309-b30d-4222d4207315">

실행 계획은 위에서 아래 방향으로 읽으며 최종 결과는 가장 위에, 데이터 소스는 가장 아래에 있다.

위 이미지에서는 각 줄의 첫번째 키워드인 Sort, Exchange, FileScan에 주목해야 한다.

특정 컬럼을 다른 컬럼과 비교하는 sort 메서드가 넓은 트랜스포메이션으로 동작하는 것을 볼 수 있다.

### 액션 호출

트랜스포메이션 실행 계획을 시작하기 위해 액션을 호출한다. 액션을 실행하려면 몇 가지 설정이 필요하다.

스파크는 셔플 수행 시 기본적으로 200개의 셔플 파티션을 생성한다. 이 값을 5로 설정해 셔플의 출력 파티션 수를 줄인다.

```scala
spark.conf.set("spark.sql.shuffle.partitions", 5)

flightData2015.sort("count").take(2)
```

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/ddc23acb-d577-4354-86e0-8f24f7525a42">

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/4b0eb35c-778a-422b-8b36-d6532b8a236f">

$\rightarrow$ 논리적, 물리적 DataFrame 처리 과정(위 예제의 처리 과정)

트랜스포메이션의 논리적 실행 계획은 DataFrame의 계보를 정의한다. 스파크는 계보를 통해 입력 데이터에 수행한 연산을 전체 파티션에서 어떻게 재연산하는지 알 수 있다.

이 기능은 **스파크의 프로그래밍 모델인 함수형 프로그래밍의 핵심**이다. 함수형 프로그래밍은 데이터의 변환 규칙이 일정한 경우 같은 입력에 대해 항상 같은 출력을 생성한다.

사용자는 물리적 데이터를 직접 다루지 않지만, 앞서 설정한 셔플 파티션 파라미터와 같은 속성으로 물리적 실행 특성을 제어한다.

예제에서 5로 설정했던 셔플 파티션 수를 변경하면 스파크 잡의 실행 특성을 제어할 수 있다.

## 2.10.1 DataFrame과 SQL

DataFrame과 SQL을 사용하는 복잡한 작업을 수행한다.

스파크는 언어에 상관없이 같은 방식으로 트랜스포메이션을 실행할 수 있다.

- 사용자가 SQL이나 DataFrame(R, 파이썬, 스칼라, 자바에서)으로 비즈니스 로직을 표현

  $\rightarrow$ 스파크에서 실제 코드를 실행하기 전에 그 로직을 기본 실행 계획으로 컴파일

- 스파크 SQL을 사용해 DataFrame을 테이블이나 뷰(임시 테이블)로 등록

  $\rightarrow$ SQL 쿼리 사용 가능

- 스파크는 SQL 쿼리를 DataFrame 코드와 같은 실행 계획으로 컴파일

  $\rightarrow$ 둘 사이의 성능 차이 없음

### 테이블, 뷰 생성

**createOrReplaceTempView** 메서드를 호출하면 모든 DataFrame을 테이블이나 뷰로 만들 수 있다.

```scala
flightData2015.createOrReplaceTempView("flight_data_2015")
```

메서드 호출 후 SQL로 데이터를 조회할 수 있다.

### SQL 쿼리 실행

새로운 DataFrame을 반환하는 **spark.sql** 메서드로 SQL 쿼리를 실행한다.

spark는 SparkSession의 변수이다. DataFrame에 쿼리를 수행하면 새로운 DataFrame을 반환한다.

```scala
val sqlWay = spark.sql("""
SELECT DEST_COUNTRY_NAME, count(1)
FROM flight_data_2015
GROUP BY DEST_COUNTRY_NAME
""")

val dataFrameWay = flightData2015
.groupBy('DEST_COUNTRY_NAME)
.count()

sqlWay.explain
dataFrameWay.explain
```

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/eff5b122-4cf4-4933-b1ac-4106a3bc0a76">

sqlWay, dataFrameWay 모두 동일한 기본 실행 계획으로 컴파일된다.

### 다양한 데이터 처리 기능

데이터에서 흥미로운 통계를 찾는다. 스파크의 DataFrame과 SQL은 다양한 데이터 처리 기능을 제공한다.  
스파크는 빅데이터 문제를 빠르게 해결하는 데 필요한 수백 개의 함수를 제공한다.

### 특정 위치를 왕래하는 최대 비행 횟수

특정 위치를 왕래하는 최대 비행 횟수를 구하기 위해 max 함수를 사용한다.  
max 함수는 DataFrame의 특정 컬럼값을 스캔하면서 이전 최댓값보다 더 큰 값을 찾는다. max 함수는 필터링을 수행해 단일 로우를 결과로 반환하는 트랜스포메이션이다.

```scala
spark.sql("SELECT max(count) from flight_data_2015").take(1)

import org.apache.spark.sql.functions.max

flightData2015.select(max("count")).take(1)
```

<img width="350" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/a8aaf8dc-330d-483b-ac1b-b5b8ac47577e">

### 상위 5개의 도착 국가

상위 5개의 도착 국가를 찾아내는 코드를 실행한다.

다중 트랜스포메이션 쿼리이기 때문에 단계별로 알아본다. SQL 집계 쿼리부터 시작한다.

```scala
val maxSql = spark.sql("""
SELECT DEST_COUNTRY_NAME, sum(count) as destination_total
FROM flight_data_2015
GROUP BY DEST_COUNTRY_NAME
ORDER BY sum(count) DESC
LIMIT 5
""")

maxSql.show()
```

<img width="350" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/6bec2a96-7974-4f24-af03-7e79a436be70">

위 코드와 의미는 같고 구현과 정렬 방식이 다른 DataFrame 구문을 살펴본다. 기본 실행 계획은 두 방법 모두 동일하다.

```scala
import org.apache.spark.sql.functions.desc

flightData2015
  .groupBy("DEST_COUNTRY_NAME")
  .sum("count")
  .withColumnRenamed("sum(count)", "destination_total")
  .sort(desc("destination_total"))
  .limit(5)
  .show()
```

<img width="350" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/2c6f679f-85e6-4ef9-9c93-870bf5398a2d">

DataFrame의 explain 메서드로 확인해보면 총 7가지 단계가 존재한다. explain 메서드가 출력하는 실제 실행 계획은 물리적인 실행 시점에서 수행하는 최적화로 인해 아래 그림과 다를 수 있다.

실행 계획은 트랜스포메이션의 **지향성 비순환 그래프**(**directed acyclic graph, DAG**)이며 액션이 호출되면 결과를 만들어낸다. 지향성 비순환 그래프의 각 단계는 불변성을 가진 신규 DataFrame을 생성한다.

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/a78d90cc-ecaa-4f2a-a39b-3eec230e66bc">

$\rightarrow$ DataFrame의 변환 흐름

### 1단계

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/03479369-4a64-444c-b9df-64c09ec81388">

데이터를 읽는다. 데이터를 읽는 DataFrame은 이전 예제에서 정의했다.

기억해야 할 것은 스파크는 해당 DataFrame이나 자신의 원본 DataFrame에 액션이 호출되기 전까지 데이터를 읽지 않는다.

### 2단계

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/9ee88d9e-2fc4-47fd-9dc3-0451139e6218">

데이터를 그룹화한다.

groupBy 메서드가 호출되면 최종적으로 그룹화된 DataFrame을 지칭하는 이름을 가진 **RelationalGroupedDataset**을 반환한다.  
기본적으로 키 혹은 키셋을 기준으로 그룹을 만들고 각 키에 대한 집계를 수행한다.

### 3단계

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/16a666a6-d443-4d8c-bb5b-494a55001354">

집계 유형을 지정하기 위해 컬럼 표현식이나 컬럼명을 인수로 사용하는 **sum** 메서드를 사용한다.

**sum** 메서드는 새로운 스키마 정보를 가지는 별도의 DataFrame을 생성한다. 신규 스키마에는 새로 만들어진 각 컬럼의 데이터 타입 정보가 들어있다.

sum 메서드 역시 트랜스포메이션으로 **아무런 연산도 발생하지 않는다.** 스파크는 새롭게 생성된 결과 스키마를 통해 타입 정보를 추적한다.

### 4단계

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/342c1273-8335-4868-9016-3d7b7e96bc16">

컬럼명을 변경한다. 이름을 변경하기 위해 **withColumnRenamed** 메서드에 원본 컬럼명과 신규 컬럼명을 인수로 지정한다.

withColumnRenamed 메서드 역시 트랜스포메이션이며, 이 단계에서도 여전히 연산은 발생하지 않는다.

### 5단계

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/31af7db2-9c44-46b9-beb3-48925da30d4e">

데이터를 정렬한다.

결과 DataFrame의 첫 번째 로우를 확인해보면 destination_total 컬럼에서 가장 큰 값을 가지고 있다.

역순으로 정렬하기 위해 desc 함수를 import 한다. desc 함수는 문자열이 아닌 Column 객체를 반환한다.

DataFrame 메서드 중 대부분은 문자열 형태의 컬럼명, Column 타입 그리고 표현식을 파라미터로 사용한다. Column과 표현식은 사실상 같은 것이다.

### 6단계

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/594c4252-1eba-4beb-906d-ed24f9534314">

**limit** 메서드로 반환 결과의 수를 제한한다. 결과 DataFrame의 전체 데이터 대신 상위 5개 로우를 반환한다.

### 7단계

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/57d137b5-4dc4-4ef0-b064-f6fafd06e05b">

액션을 수행한다.

이 단계에 와서야 DataFrame의 결과를 모으는 프로세스를 시작한다. 처리가 끝나면 코드를 작성한 프로그래밍 언어에 맞는 리스트나 배열을 반환한다.

explain 메서드 호출을 통해 실행 계획을 살펴본다.

```scala
flightData2015
  .groupBy("DEST_COUNTRY_NAME")
  .sum("count")
  .withColumnRenamed("sum(count)", "destination_total")
  .sort(desc("destination_total"))
  .limit(5)
  .explain()
```

<img width="850" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/7be24a82-c31f-4ca8-a5a9-489264c6fc25">

$\rightarrow$ 실행 계획

'개념적 계획'과 정확하게 동일하지 않지만 모든 부분을 포함하고 있다.  
limit 구문과 orderBy 구문을 볼 수 있고, partial_sum 함수를 호출하는 부분에서 집계가 두 단계로 나누어지는 것을 알 수 있다.

단계가 나누어지는 이유는 숫자 목록의 합을 구하는 연산이 **가환성**(**commutative, 교환 법칙 성립**)을 가지고 있어 합계 연산 시 파티션별 처리가 가능하기 때문이다.

데이터를 드라이버로 모으는 대신 스파크가 지원하는 여러 데이터 소스로 내보낼 수도 있다. 예를 들어 PostgreSQL과 같은 데이터베이스나 다양한 포맷의 파일에 결과를 저장할 수 있다.
