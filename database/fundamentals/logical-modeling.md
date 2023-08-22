> [논리적 데이터 모델링](http://www.kocw.or.kr/home/cview.do?mty=p&kemId=1163794) 강의를 보고 정리합니다.

## 목차

1. [데이터베이스 설계 단계](#데이터베이스-설계-단계)
2. [개념적 데이터 모델링 결과](#개념적-데이터-모델링-결과)
3. [논리적 설계](#논리적-설계)
4. [릴레이션 스키마 변환 규칙](#릴레이션-스키마-변환-규칙)
   - [규칙 1 - 모든 개체는 릴레이션으로 변환](#규칙-1---모든-개체는-릴레이션으로-변환)
   - [규칙 2 - 다대다 관계는 릴레이션으로 변환](#규칙-2---다대다-관계는-릴레이션으로-변환)

## 데이터베이스 설계 단계

<img width="200" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/2384368f-2e78-4364-a75d-58fcbdd4eabc">

$\rightarrow$ E-R 모델과 릴레이션 변환 규칙을 이용한 설계의 과정

설계 과정 중 오류 발견 시 이전 단계로 되돌아가 설계 내용을 변경할 수 있다.

<img width="560" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/24f1c409-8b12-433a-bf19-6ed28e250894">

$\rightarrow$ 데이터베이스 설계 과정의 각 단계별 주요 작업과 결과물

## 개념적 데이터 모델링 결과

<img width="700" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/f40f74c7-f6c0-4da7-9632-6ab48622a753">

$\rightarrow$ 요구 사항 명세서를 개념적 스키마로 작성한 결과

## 논리적 설계

### 목적

- DBMS에 적합한 논리적 스키마 설계
- 개념적 스키마를 논리적 데이터 모델을 이용해 논리적 구조로 표현  
  $\rightarrow$ 논리적 모델링(데이터 모델링)

일반적으로 관계 데이터 모델을 많이 이용

### 결과물

- 논리적 스키마 : 릴레이션 스키마

### 주요 작업

- 개념적 설계 단계의 결과물인 E-R 다이어그램을 릴레이션 스키마로 변환
- 릴레이션 스키마 변환 후 속성의 데이터 타입, 길이, 널값 허용 여부, 기본 값, 제약조건 등을 세부적으로 결정하고 그 결과를 문서화시킴

## 릴레이션 스키마 변환 규칙

- **규칙 1**

  $\rightarrow$ 모든 개체는 릴레이션으로 변환한다.

- **규칙 2**

  $\rightarrow$ 다대다(n:m) 관계는 릴레이션으로 변환한다.

- **규칙 3**

  $\rightarrow$ 일대다(1:n) 관계는 외래키로 표현한다.

- **규칙4**

  $\rightarrow$ 일대일(1:1) 관계는 외래키로 표현한다.

- **규칙 5**

  $\rightarrow$ 다중 값 속성은 릴레이션으로 변환한다.

변환 규칙을 순서대로 적용하되, 해당되지 않는 규칙은 제외한다.

## 규칙 1 - 모든 개체는 릴레이션으로 변환

E-R 다이어그램의 각 개체를 하나의 릴레이션으로 변환

- **개체의 이름** $\rightarrow$ **릴레이션 이름**

- **개체의 속성** $\rightarrow$ **릴레이션의 속성**

- **개체의 키 속성** $\rightarrow$ **릴레이션의 기본키**

- 개체의 속성이 **복합 속성**인 경우, 복합 속성을 구성하고 있는 **단순 속성만 릴레이션의 속성으로 변환**

### 예시 1

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/4f828d39-46e1-4441-87d9-cb2e36c56573">

$\rightarrow$ 개체를 릴레이션으로 변환하는 규칙을 적용한 예

상품(<U>상품번호</U>, 상품명, 재고량, 단가)

### 예시 2 - 복합 속성 가지는 개체 변환

<img width="450" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/03d3f179-b672-4a24-990e-38f0b3bee3d8">

$\rightarrow$ 복합 속성을 가지는 개체를 릴레이션으로 변환하는 예

### 규칙 1 적용 결과

[개념적 데이터 모델링](https://github.com/usuyn/TIL/blob/master/database/fundamentals/conceptual-modeling.md)의 결과물인 [E-R 다이어그램](#개념적-데이터-모델링-결과)에서 상품, 제조업체, 회원, 게시글 개체에 규칙 1을 적용한다.

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/03061bd7-1d6d-487d-b592-685fcead322a">

## 규칙 2 - 다대다 관계는 릴레이션으로 변환

E-R 다이어그램의 다대다 관계를 하나의 릴레이션으로 변환

- **관계의 이름** $\rightarrow$ **릴레이션 이름**

- **관계의 속성** $\rightarrow$ **릴레이션의 속성**

- 관계에 참여하는 개체를 규칙 1에 따라 릴레이션으로 변환 후 **기본키를 관계 릴레이션에 포함**시켜 **외래키로** 지정하고 **외래키들을 조합해 관계 릴레이션의 기본키로** 지정

### 예시

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/6066b458-3b64-45c0-a360-c4aebc8ee42b">

### 규칙 2 적용 결과

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/39a3f131-b5fb-42db-9367-091ca38dd2d2">
