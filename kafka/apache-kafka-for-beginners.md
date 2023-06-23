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
   
---
