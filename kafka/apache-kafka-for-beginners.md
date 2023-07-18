> [아파치 카프카 입문](https://www.inflearn.com/course/아파치-카프카-입문) 강의를 보고 정리합니다.

---

## 아파치 카프카 개요 및 설명

### **1. Before Kafka**   

**데이터를 전송하는 Source Application**과 **데이터를 받는 Target Application**이 존재하며, 초기에는 단방향 통신을 사용했다.   

시간이 흐름에 따라 Source와 Target application이 많아지면서 데이터 전송 라인 또한 복잡해졌다.   

**데이터 전송 라인이 복잡**해지면 **배포, 장애에 대응하기 어려워**진다. 또한 데이터를 전송할 때 **프로토콜, 포맷의 파편화**가 심해지며, 데이터 포맷 변경사항이 있을 때 **유지보수도 어려워**진다.   

이러한 **문제들을 해결**하기 위해 LinkedIn에서 **Apache Kafka를 개발**했고 현재는 오픈소스로 제공되고 있다.

---

### **2. Kafka 주변 생태계** 

kafka는 **source와 target application의 커플링을 약하게** 하기 위해 만들어졌다.   

**Source application은 kafka에 데이터를 전송**하고,   
**Target application은 kafka에서 데이터를 가져온다.**   

Source application에서 **전송할 수 있는 데이터의 포맷은 거의 제한이 없다.** json, tsc, avro 등을 모두 지원한다.   

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/da548445-f12e-4554-87fb-35cbfec2cec8">  
  
Kafka에는 **각종 데이터를 담는 Topic**이라는 개념이 존재한다. 쉽게 말하면 **Queue**인 것이다.   

**Queue에 데이터를 넣는 Kafka Producer**와   
**Queue에서 데이터를 가져가는 Kafka Consumer**로 나누어진다.

**producer와 consumer는 라이브러리**로 되어 있어 application에서 구현 가능하다.   
   
<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/dd5b1861-3ded-4762-99c6-c9de8f9348b2">

---

### **3. 결론** 

Kafka는 **아주 유연한 Queue** 역할을 하는 것이라고 할 수 있다.   

Kafka는 데이터 흐름에 있어 Fault Tolerant 즉, **고가용성**으로 서버에 이슈가 생기거나 갑자기 전원이 내려가도 **데이터 손실 없이 복구가 가능**하다. 또한 낮은 지연(latency), 높은 처리량(throughput)을 통해 **효율적으로 많은 데이터 처리**가 가능하다.   
   

## Topic이란?

### **1. Topic** 

데이터가 들어가는 공간을 topic이라고 부른다.  

kafka topic은 일반적인 AMQP(Advanced Message Queing Protocol)와는 다르게 동작한다.  

&#62; AMQP(Advanced Message Queing Protocol) : 메시지 지향 미들웨어를 위한 개방형 표준 응용 계층 프로토콜, MQ의 오픈 소스에 기반한 표준 프로토콜을 의미한다. 기존 MQ들의 약점을 보완하기 위해 등장했다.  

kafka에서는 topic을 여러개 생성할 수 있으며, DB의 테이블이나 파일 시스템의 폴더와 유사한 성질을 가진다.  

topic은 이름을 가질 수 있다. 어떤 데이터를 담는지, 목적에 따라 명확하게 명시하면 추후 쉽게 유지보수가 가능하다.

<img width="650" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/0178463e-c98e-41b6-b840-550334dae3e0">

---

### **2. Topic 내부, Partition** 

하나의 topic은 여러개의 partition으로 구성될 수 있다.

partition 번호는 0번부터 시작하며, queue와 같이 끝에서부터 데이터가 차곡차곡 쌓인다.  

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/e558b3f4-2928-4360-a480-995f692cff8c">

한 topic에 consumer가 붙게 되면 가장 오래된 데이터부터 순서대로 가져가며, 더이상 데이터가 없으면 또 다른 데이터가 들어올 때까지 기다린다.  

consumer가 topic 내부 partition에서 데이터를 가져가도 데이터는 삭제되지 않는다. 따라서 새로운 consumer가 붙었을 때 다시 0번 데이터부터 가져갈 수 있다.  

다만 consumer 그룹이 달라야 하며, auto.offset.reset = earliest로 세팅되어 있어야 한다.  

&#62; offset : consumer가 topic의 데이터를 어디까지 읽었는지 저장값  

&#62; auto.offset.reset 옵션  

``` java
// consumer가 topic 구독 후 partition에 처음 들어오는 데이터부터  
auto.offset.reset = latest

// 가장 처음 offset부터 즉, 가장 오래된 데이터  
auto.offset.reset = earliest

// offset 찾지 못하면 exception 발생
auto.offset.reset = none
```

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8a36ff31-0c63-47a5-b0c7-8984ca139edc">

데이터가 삭제되지 않기 때문에 동일한 데이터에 대해 2번 처리할 수 있는데 이는 kafka를 사용하는 아주 중요한 이유기도 하다.

클릭 로그를 분석하고 시각화하기 위해 ES(Elasticsearch)에 저장하기도 하고, 클릭 로그를 백업하기 위해 Hadoop에 저장할 수도 있다.

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/6bb3dfa0-3dda-4956-91ee-565971e51157">

---

### **3. Partition이 2개 이상인 경우**  

producer는 데이터를 보낼 때 키를 지정할 수 있다.  

producer가 새로운 데이터를 보내면 아래 규칙에 따라 partition이 지정된다.  

- key가 null이고, 기본 partitioner 사용할 경우  
$\rightarrow$ RR(Round-robin)로 할당

- key가 있고, 기본 partitioner 사용할 경우  
$\rightarrow$ key의 hash값을 구하고 특정 partition에 할당한다.

<img width="750" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/ccecd73e-5c9f-4e35-b437-fad6d9ac99f7">

---

### **4. Partition을 늘리는 것**  

partition을 늘리는 것은 가능하나 다시 줄일 수는 없다.  

그렇다면 왜 partition을 늘리는 것일까?  

바로 데이터 처리를 분산시키기 위해서이다. partition을 늘리면 consumer를 늘려서 데이터 처리를 분산시킨다.  

---

### **5. Partition의 Record가 삭제되는 시점** 

삭제되는 타이밍은 옵션에 따라 다르며, 아래 방법으로 record가 저장되는 최대 시간과 길이를 지정할 수 있다.

``` java
// 최대 record 보존 시간
log.retention.ms

// 최대 record 보존 크기(byte)
log.retention.byte
```

위 방법을 통해 일정한 기간 혹은 용량만큼 데이터를 저장할 수 있게 되며, 적절한 시점에 데이터가 삭제되게 설정할 수 있다.


## Consumer Lag이란?

kafka를 운영함에 있어 아주 중요한 모니터링 지표이다.

producer는 topic의 partition에 데이터를 차곡차곡 넣는다. 이 partition에 데이터가 하나씩 들어가게 되면 각 데이터에는 offset이라는 숫자가 붙게 된다.   
partition이 1개일 때 데이터를 넣을 경우 차례대로 숫자가 0부터 매겨진다.

만약 producer가 데이터를 넣어주는 속도가 consumer가 가져가는 속도보다 빠르면   
$\rightarrow$ producer가 넣은 데이터의 offset, consumer가 가져간 데이터의 offset 간 차이가 발생한다.

이것이 바로 consumer lag이다.

<img width="250" height="auto" alt="image" src="https://github.com/usuyn/TIL/assets/68963707/180b61f1-df97-4d75-b513-a4e9871a8974">
   
lag의 숫자를 통해 해당 topic에 대해 파이프라인으로 연계되어 있는 producer와 consumer 상태 유추가 가능하다. 주로 consumer의 상태를 볼 때 사용한다.

topic에 여러 partition이 존재하면 lag이 여러개 존재할 수 있다.   
consumer 그룹이 1개이고, partition이 2개인 topic에서 데이터를 가져가면 lag 2개가 측정될 수 있다.

이렇게 1개의 topic과 consumer 그룹에 대한 lag이 여러개 존재할 수 있을 때, 그 중 높은 숫자의 lag을 records-lag-max라고 한다.

<img width="300" height="auto" alt="image" src="https://github.com/usuyn/TIL/assets/68963707/d3dbb0e1-5c98-424b-82a4-a023e19d70ae">
   
consumer의 성능이 안 나오거나 비정상적인 동작을 하면 lag이 필연적으로 발생해 주의 깊게 살펴본다.

### 핵심
1. lag은 producer의 offset과 consumer의 offset간의 차이이다.
2. lag은 여러개 존재할 수 있다.


## Consumer Lag Monitoring Application, Kafka Burrow

kafka lag을 모니터링하기 위해 오픈소스인 burrow를 사용한다.

kafka lag은 topic의 가장 최신 offset과 consumer offset간의 차이이다.

kafka client 라이브러리를 사용해 java 또는 scala 같은 언어로 kafka consumer를 구현할 수 있다.   
이때 구현한 kafka consumer 객체를 통해 현재 lag 정보를 가져온다.   
만약 lag을 실시간으로 모니터링 하고 싶다면 데이터를 Elasticsearch, InfluxDB와 같은 저장소에 넣은 뒤 Grafana 대시보드를 통하는 방법을 사용할 수 있다.

consumer 단위에서 lag을 모니터링하는 것은 아주 위험하고 운영 요소가 많이 들어간다.   
그 이유는 consumer 로직단에서 lag을 수집하는 것은 consumer 상태에 dependency가 걸리기 때문이다.

- consumer가 비정상적으로 종료되면 consumer는 lag 정보를 보낼 수 없기 때문에 더이상 lag을 측정할 수 없다.
- consumer가 개발될 때마다 해당 consumer에 lag 정보를 특정 저장소에 저장할 수 있도록 로직을 개발해야 한다. 만약 consumer lag을 수집할 수 없는 consumer라면 모니터링 할 수 없어 운영이 매우 까다로워 진다.

따라서 LinkedIn에서는 apache kafka와 함께 kafka의 consumer lag을 효과적으로 모니터링할 수 있도록 burrow를 배포했다.

---

### 1. Burrow의 특징

burrow는 오픈소스로서 Golang으로 작성되었고 현재 깃허브에 올라와있다.   
consumer lag 모니터링을 도와주는 독립적인 애플리케이션다.

burrow는 3가지 큰 특징을 가지고 있다.

- Multiple Kafka Cluster 지원

kafka cluster를 운영하는 기업이라면 대부분 2개 이상의 kafka cluster를 운영하고 있다. cluster가 여러개더라도 burrow application 1개만 실행해서 연동하면 cluster들에 붙은 consumer lag을 모두 모니터링 할 수 있다.

- Sliding window를 통한 Consumer의 status 확인

sliding window를 통해서 consumer의 status를 ERROR, WARNING, OK로 표현할 수 있도록 했다.   

데이터의 양이 일시적으로 많아지면서 consumer offset이 증가되고 있으면 WARNING으로 정의된다.   
데이터의 양이 많아지고 있는데 consumer가 데이터를 가져가지 않으면 ERROR로 정의된다.

- HTTP api 제공

위와 같은 정보들을 burrow가 정의한 HTTP api를 통해 조회할 수 있다.   
범용적으로 사용되는 HTTP를 사용해 다양한 추가 생태계를 구축할 수 있게 된다. HTTP api를 호출해서 response 받은 데이터를 시계열 DB와 같은 곳에 저장하는 application을 만들어 활용할 수도 있다. 

---

### 2. 결론
Burrow를 도입한다고 모든 문제가 해결되는 것은 아니다. 다만 kafka 개발자, kafka cluster 운영자가 효과적으로 kafka 관련 애플리케이션을 운영할 때 반드시 필요하며 burrow를 통해 수집된 데이터는 결국 추후 애플리케이션 개발과 운영에 많은 도움이 된다.

---

### 3. 참고 링크
[Burrow github](https://github.com/linkedin/Burrow)   
[Burrow 소개](https://blog.voidmainvoid.net/243)   
[Burrow의 Consumer status 확인 방법](https://blog.voidmainvoid.net/244)   
[Burrow http endpoint 정리](https://blog.voidmainvoid.net/245)
