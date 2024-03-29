# Chapter 1. 신뢰할 수 있고 확장 가능하며 유지보수하기 쉬운 애플리케이션
- 오늘날 많은 애플리케이션은 계산 중심(compute-intensive)과는 다르게 **데이터 중심(data-intensive)적** 이다.
  - 이러한 애플리케이션을 제한하는 요소는 CPU 성능이 아니라 **데이터의 양, 복잡도, 변화 속도** 가 문제이다.
- 일반적으로 데이터 중심 애플리케이션은 공통으로 필요로 하는 기능을 제공하는 표준 구성 요소로 만든다. 
  - 데이터베이스 : 구동 애플리케이션이나 다른 애플리케이션에서 나중에 데이터를 찾을 수 있게 데이터를 저장
  - 캐시 : 읽기 속도 향상을 위해 값비싼 수행 결과를 기억
  - 검색 색인 : 사용자가 키워드로 데이터를 검색하거나 다양한 방법으로 필터링할 수 있게 제공
  - 스트림 처리 stream processing : 비동기 처리를 위해 다른 프로세스로 메시지 보내기 
  - 일괄 처리 batch processing : 주기적으로 대량의 누적된 데이터를 분석 

## 데이터 시스템에 대한 생각
- 데이터 시스템
  - 예시: 데이터베이스, 큐, 캐시 등 
  - 공통점: 데이터를 일정 기간동안 저장
  - 차이점: 다른 접근 패턴을 갖고 있어 서로 다른 성능 특성이 있기 때문에 구현 방식이 매우 다르다.
- 데이터 시스템이라는 포괄적 용어로 묶어야 하는 이유 
  1. 새로운 도구들은 다양한 사용 사례(use case)에 최적화됐기 때문에 전통적인 분류에 딱 들어맞지 않는다.
     - 메시지 큐로 사용하는 데이터스토어(data store)인 레디스(Redis)
     - 데이터베이스처럼 지속성(durability)을 보장하는 메시지큐인 아파치 카프카(Apache Kafka)
  2. 많은 애플리케이션이 단일 도구로는 데이터 처리와 저장을 만족시키기 어렵다. 
     - 메인 데이터베이스와 분리된 애플리케이션 관리 캐시 계층(Memcached나 유사한 도구를 사용)이나 검색 서버(Elasticsearch나 Solr)의 경우,
     메인 데이터베이스와 동기화된 캐시나 색인을 유지하는 것은 애플리케이션 코드의 책임이다. 
     - 복합 데이터 시스템(composite data system)은 외부 클라이언트가 일관된 결과를 볼 수 있게끔 쓰기에서 캐시를 무효화하거나 업데이트 하는 등의 특정 보장 기능을 제공할 수 있다.
     - 개발자는 애플리케이션 뿐 아니라 데이터 시스템 설계에 대한 책임을 갖는다.
     - 다양한 구성 요소를 결합한 데이터 시스템 아키텍처의 예   
     <img width="680" alt="스크린샷 2023-01-30 오후 10 51 17" src="https://user-images.githubusercontent.com/45681372/215495686-9aa9db5a-971f-47fc-99e1-36edcfd05105.png">
       - 캐시 미스(cache miss) : CPU가 원하는 데이터가 캐시에 없는 상태

- **이 책에서의 관심사** 
  - 신뢰성(Reliability)
    - 하드웨어나 소프트웨어 결함, 심지어 인적 오류 같은 역경에 직면하더라도 시스템은 지속적으로 올바르게 동작해야 한다.
  - 확장성(Scalability)
    - 시스템의 데이터 양, 트래픽 양, 복잡도가 증가하면서 이를 처리할 수 있는 적절한 방법이 있어야 한다.
  - 유지보수성(Maintainability)
    - 시간이 지남에 따라 여러 다양한 사람이 시스템 상에서 작업할 것이기 때문에 모든 사용자가 시스템 상에서 생산적으로 작업할 수 있게 해야 한다. 

## 신뢰성
- 애플리케이션에서 "올바르게 동작함"
  - 애플리케이션은 사용자가 기대한 기능을 수행한다.
  - 시스템은 사용자가 범한 실수나 예상치 못한 소프트웨어 사용법을 허용할 수 있다.
  - 시스템 성능은 예상된 부하와 데이터 양에서 필수적인 사용 사례를 충분히 만족한다.
  - 시스템은 허가되지 않은 접근과 오남용을 방지한다. 
- 신뢰성의 의미 : "무언가 잘못되더라도 지속적으로 올바르게 동작함"

- 결함(fault) : 잘못될 수 있는 일
  - 내결함성(fault-tolerant) 또는 탄력성(resilient) : 결함을 예측하고 대처할 수 있는 성질
  - 결함은 장애(failure)와 동일하지 않다. 
    - 결함: 사양에서 벗어난 시스템의 한 구성 요소, 결함 확률을 0으로 줄이는 건 불가능 
    - 장애: 사용자에게 필요한 서비스를 제공하지 못하고 시스템 전체가 멈춘 경우
  - 결함으로 인해 장애가 발생하지 않게끔 내결함성 구조를 설계하는 것이 좋다.
  - Netflix Chaos Monkey : “무기를 든 야생 원숭이가 데이터센터(또는 클라우드 영역)에 들어와 무작위로 인스턴스를 파괴하고 케이블을 끊더라도 중단 없이 고객에게 서비스를 계속 제공한다는 개념”
    - 고의적으로 결함을 유도해 내결함성 시스템을 훈련

### 하드웨어 결함
- 하드웨어 결함 위험성 증가 
  - 대규모 애플리케이션 > 더 많은 수의 장비 > 하드웨어 결함율 증가 
  - AWS 같은 클라우드 플랫폼은 유연성과 탄력성을 우선하게 설계되어 단일 장비 신뢰성이 비교적 떨어짐
- 일반적인 해결 방법: 하드웨어 중복(redundancy) 구성
  - 고장 난 구성 요소가 교체 되는 동안 중복된 구성 요소를 사용
  - 운영상 장점: 장비 재부팅 등을 순차적으로 진행할 수 있어서 전체 시스템 중단이 없음 

### 소프트웨어 오류
- 하드웨어 결함은 보통 서로 독립적
- 시스템 내 체계적 오류(systematic error)는 노드 간 상관관계 때문에 시스템 오류를 더 많이 유발하는 경향
  - 잘못된 특정 입력이 있을때 애플리케이션 서버 인스턴스가 죽는 소프트웨어 버그 
  - cpu 시간, 메모리, 디스크 공간, 네트워크 대역폭처럼 공유 자원을 과도하게 사용하는 일부 프로세스 등 
- 소프트웨어의 체계적 오류 문제는 신속한 해결책이 없다. 
  - 시스템의 가정과 상호작용에 대해 주의 깊게 생각하기
  - 빈틈없는 테스트
  - 프로세스 격리 (process isolation)
  - 죽은 프로세스의 재시작 허용
  - 프로덕션 환경에서 시스템 동작의 측정
  - 모니터링
  - 분석하기 등 여러 작은 일들이 문제 해결에 도움을 준다.

### 인적 오류
- 사람은 미덥지 않으니까 다양한 접근 방식을 결합해 시스템을 만들자
  - 오류의 가능성을 최소화하는 방향으로 시스템을 설계
  - 테스트
  - 인적 오류에 대한 복구 방법 제공
  - 성능 지표와 오류율 같은 모니터링 대책


## 확장성
## 유지보수성
## 정리