###  Apache Kafka 는 무엇인가?
- [Data in Motion Platform for Enterprise](https://www.confluent.io/product/confluent-platform/)

###  Event 란?
- Something Happened!!!

- Event는 비즈니스에서 일어나는 모든 일(데이터)을 의미
  - 웹사이트에서 무언가를 클릭하는 것 o 청구서 발행
  - 송금
  - 배송 물건의 위치 정보
  - 택시의 GPS 좌표
  - 센서의 온도/압력 데이터

- 비즈니스에서 일어나는 모든 일(데이터)을 의미
  - 이벤트를 비즈니스에 활용

### Event Stream은 무엇인가? 
- 연속적인 많은 이벤트들의 흐름
- Event는 BigData의 특징을 가짐
  - 비즈니스의 모든 영역에서 광범위하게 발생
  - 대용량의 데이터(Big Data) 발생
- `Event Stream은 연속적인 많은 이벤트들의 흐름을 의미`

### Apache Kafka의 탄생 
- LinkedIn 내에서 개발
  - 하루 4.5 조 개 이상의 이벤트 스트림 처리
  - 하루 3,000 억 개 이상의 사용자 관련 이벤트 스트림 처리
  - 기존의 Messaging Platform(예, MQ)로 처리 불가능
  - 이벤트 스트림 처리를 위해 개발
  - 2011년에 Apache Software Foundation 에 기부되어 오픈소스화

### Apache Kafka의 특징 
- 3가지 주요 특징
  - 이벤트 스트림을 안전하게 전송 Publish & Subscribe
  - 이벤트 스트림을 디스크에 저장 Write to Disk
  - 이벤트 스트림을 분석 및 처리 Processing & Ananlysis

###  Apache Kafka 이름의 기원 
- Franz Kafka에서 가져온 이름
  - 창시자인 Jay Kreps가 명명
  - Kafka는 쓰기(Write)에 최적화된 시스템이기 때문에, 작가(Writer)의 이름을 사용하는 것이 의미가 있다고 생각했습니다. 나는 대학에서 문학 수업을 많이 들었고 Franz Kafka를 좋아했습니다. 게다가 오픈 소스 프로젝트에 대해 그 이름은 멋지게 들렸습니다. 그래서, 기본적으로 관계가 많지 않습니다.
    - https://www.quora.com/What-is-the-relation-between-Kafka-the-writer-and-Apache-Kafka-the-distributed-messaging-system/answer/Jay-Kreps

###  Apache Kafka의 등장
- 2012년 최상위 오픈소스 프로젝트로 정식 출범
  - 2012년 Apache Incubator 과정을 벗어나 최상위 프로젝트가 됨
  - Fortune 100 기업 중 80% 이상 사용
    - https://kafka.apache.org/

###  Confluent Inc.
- Kafka 창시자가 만든 회사
  - Kafka 창시자(Jay Kreps)가 만든 회사
  - 2014년 설립
  - Mountain View, CA
  - 2021년 6월 IPO (기업공개, Nasdaq 상장)

###  Apache Kafka 사용 사례 
- Event(메시지/데이터)가 사용되는 모든 곳

- Event(메시지/데이터)가 사용되는 모든 곳에서 사용
  - Messaging System
  - IOT 디바이스로부터 데이터 수집
  - 애플리케이션에서 발생하는 로그 수집
  - Realtime Event Stream Processing (Fraud Detection, 이상 감지 등) o DB 동기화 (MSA 기반의 분리된 DB간 동기화)
  - 실시간 ETL
  - Spark, Flink, Storm, Hadoop 과 같은 빅데이터 기술과 같이 사용

###  산업 분야별 Apache Kafka 사용 사례
- 다양한 산업 분야에서 사용중

###  Apache Kafka의 성능
- 저렴한 장비로 초당 2백만 Writes

- Benchmarking Apache Kafka: 2 Million Writes Per Second (On Three Cheap Machines)
  - Intel Xeon 2.5 GHz processor(6 코어)
  - 7200 RPM SATA 드라이버 6 개 : RAID 아닌 JBOD 로 구성 o 32 GB of RAM
  - 1 Gb Ethernet
  - 상기 스펙의 총 6 대 HW 사용
    - 3 대 ‒ Zookeeper/Load Generator, 3 대 ‒ Kafka Broker 
  - Three Producer, 3x async replication :
    - 2,024,032 records/sec (193.0 MB/sec)

###  Apache Kafka vs. RabbitMQ 
- 월등한 처리량을 제공하는 Kafka

- [Apache Kafka vs. RabbitMQ 월등한 처리량을 제공하는 Kafka](https://www.confluent.io/blog/kafka-fastest-messaging-system/?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.kafka_mt.mbm_rgn.apac_lng.eng_dv.all_con.kafka-rabbitMQ&utm_term=%2Bkafka%20%2Brabbitmq&creative=&device=c&placement=&gclid=CjwKCAjw3_KIBhA2EiwAaAAliiwLNAaE01pbjuDTGvedQ9G9qPJO-fyKJhHvGIxDq0s9RZ3oU2P9IRoCvT0QAvD_BwE)
  - RabbitMQ의 Latency(지연 시간)은 30MB/s보다 높은 처리량에서 크게 저하됩니다. 또한 미러링의 영향은 더 높은 처리량에서 중요하며, 미러링 없이 클래식 queue만 사용하면 더 나은 Latency를 얻을 수 있습니다.

###  Summary
- Apache Kafka - [Data in Motion Platform for Enterprise]( https://www.confluent.io/product/confluent-platform/)

- Event Streaming Platform
  - 1 이벤트 스트림을 안전하게 전송 Publish & Subscribe
  - 2 이벤트 스트림을 디스크에 저장 Write to Disk
  - 3 이벤트 스트림을 분석및처리 Processing & Ananlysis