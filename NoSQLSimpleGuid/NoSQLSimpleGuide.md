# NoSQL 간결안내서 정리
## 1. 키 값데이터 베이스

- 해시 테이블을 사용한다.
- 주키(PK)를 통해서만 데이터 베이스에 접근할 수 있다.<br>항상 주키를 사용해 접근하기 때문에 성능과 확장성이 뛰어나다.
- 모든 객체를 한 버킷에 저장하면 키 충돌 확률이 높아질 수 있기 때문에 특정 데이터만 저장하는 도메인 버킷을 사용할 수 있다.

#### 1.1 일관성
- 리악같은 분산 키-값 데이터 베이스는 결과적 일관성 사용
- 리악에서는 업데이트 충돌을 해결하기 위해 두가지 방법을 사용<br>1. 최신 쓰기가 항상 이기는 방식<br>2. 두 값을 모두 리턴해 클라이언트가 해결하게 하는 방식

#### 1.2 트랜잭션
- 일반적으로 쓰기에 대한 보장이 없다.
- 리악은 정족수 개념을 사용하고 API를 호출할 때 W값(쓰기 정족수)를 사용한다.

#### 1.3 조회기능
- 키로 조회할 수 있고 값의 속성으로는 조회할 수 없기 때문에 세션 데이터를 저장하기에 유리하다.

#### 1.4 데이터 구조
- 값 부분에 무엇이 저장되든 상관하지 않는다.<br>BLOB, 텍스트, JSON, XML 등이 값이 될 수 있다.

#### 1.5 확장성
- 많은 키-값 젖장소가 샤딩을 사용한다.
> 샤딩을 사용하면 특정 데이터가 사용하는 노드에 오류가 발생했을 때 쓰기, 읽기를 모두 할 수 없는 문제가 있다.
> 이 문제를 해결하기 위해 리악에서는 N,R,W 값을 세팅하고 복제본을 만들어 가용성을 높인다.

#### 1.6 적절한 사용처
- 세션 정보 저장
- 사용자 프로파일, 설정
- 장바구니 데이터

#### 1.7 사용하지 말아햐 할 때
- 데이터간에 관계가 있는 경우
- 다중 연산 트랜잭션이 필요한경우
- 데이터로 조회하는 경우
- 집합에 의한 연산

## 2. 문서 데이터베이스
키-값 데이터 베이스의 값에 문서를 저장한다.
>여기서 문서란 XML, JSON, BSON 등이 될 수 있고 맵이나 컬렉션, 스칼라 값을 포함 할 수 있는 자체 기술적, 계층적 데이터 트리 구조이다.

RDB와 다르게 각 문서는 특정 스키마를 따를 필요가 없어 문서별로 다른 컬럼을 가질 수 있다.

#### 2.1 일관성
몽고 DB는 복제본 집합을 사용해 일관성 수준을 설정하며, 쓰기가 모든 슬레이브에 복제될 때까지 기다릴지, 아니면 주어진 수의 슬레이브에 복제될 때까지만 기다릴지 선택할 수 있다.

#### 2.2 트랜잭션
일반적으로 NOSQL은 여러 연산에 걸친 트랜잭션은 지원하지 않는다(레이븐DB는 지원함).
모든 쓰기는 성공이라고 보고하는 것이 일반적이나 WriteConcern 파라미터를 사용해 특정 수 이상의 노드에 쓰기가 성공했을 때만 성공을 보고 하도록 할 수 있다.

#### 2.3 가용성
몽고 DB는 복제본 집합을 사용해 고가용성을 제공한다.
> 복제본 집합은 비동기 마스터-슬레이브 복제의 한 형태로 둘 이상의 노드로 구성된다.
> 애플리캐이션은 마스터 노드에서 데이터를 읽거나 쓴다. 
> 애플리캐이션에서 연결할 때 복제본 집합의 한 노드에만 연결하면 된다.

#### 2.4 조회기능
카우치 DB의 경우 뷰를 이용해 복잡한 쿼리 실행을 효율적으로 할 수 있다.
키-값 저장소와 비교했을 때 문서 데이터베이스가 좋은 점 중 하나는 문서 안쪽의 데이터를 조회할 수 있다는 점이다.
몽고 DB의 경우 RBD처럼 쿼리를 만들어 데이터를 조회할 수 있다.

#### 2.5 확장성
- 읽기 부하에 대한 확장성은 slaveOk플래그를 설정한 슬레이브를 추가하고 읽기 요청을 슬레이브에서 처리하게 하면 달성 할 수 있다.
- 쓰기 부하에 대한 확장성은 샤딩을 통해 달성 할 수 있으며 새 노드를 추가하는 동안에도 서비스가 정지되지는 않지만 많은 양의 데이터가 이동하면서 성능이 저하될 수 있다.

#### 2.6 적절한 사용처
- 이벤트 로깅
- 콘텐츠 관리 시스템, 블로깅 플렛폼
- 웹 분석 또는 실시간 분석
- 전자상거래 애플리케이션

#### 2.7 사용하지 말아야 할 때
- 여러 연산에 걸친 복잡한 트랜잭션
- 변화하는 집합 구조에 대한 쿼리

## 3. 컬럼 패밀리 저장소(카산드라)
키가 값에 대응 되는 데이터를 저장한다.
- 칼럼이 키-값의 쌍으로 되어있고 이름은 키로 동작한다.
- 칼럼 패밀리 안의 칼럼이 간단한 칼럼이면 **표준 칼럼패밀리**라 한다.
- 칼럼 패밀리 안의 칼럼이 맵으로 되어있으면 **슈퍼 칼럼**이라 한다.
- 카산드라는 모든 칼럼 패밀리는 **키스페이스**에 넣는데 RDB의 데이터베이스와 비슷한 개념으로 키스페이스를 생성해야 칼럼 패밀리를 할당 할 수 있다.

#### 3.1 일관성
카산드라는 쓰기 요청을 받았을 때 데이터를 먼저 커밋 로그에 기록하고 그 다음 멤테이블이라는 메모리 내 구조에 쓴다.<br>
커밋 로그와 멤테이블 모두 기록하면 쓰기 성공으로 간주한다. 쓰기 결과는 주기적으로 SSTable이란 구조에 기록되고 한번 플러시된 SSTable은 수정이 불가능하다.<br>
만약 데이터가 수정되면 새로운 SSTable을 생성해 기록하고 사용하지 않는 SSTable은 컴팩션시 회수된다.
일관성 수준을 ONE, QUOEUM, ALL로 설정함에 따라 어느정도의 결함을 인정할지 선택할 수 있다.

#### 3.2 트랜잭션
카산드라의 쓰기는 행 단위로 원자적이며 특정 행에 대해 칼럼을 삽입하거나 갱신하는 것은 하나의 쓰기로 취급된다.
읽기와 쓰기를 동기화 하기 위해 추키퍼[ZooKeeper] 같은 외부 라이브러리를 사용할 수도 있다.(주키퍼를 사용한 트랜잭션을 감싸주는 케이지[cages]같은 라이브러리도 있다.)

#### 3.3 가용성
카산드라 클러스터에서는 마스터가 없어 모든 노드가 동등하므로 가용성이 높다.
쓰기 가용성을 높일지 읽기 가용성을 높일지 필요에 따라 키스페이스와 읽기/쓰기 연산이 설정을 조정해야한다.

#### 3.4 조회기능
카산드라는 질의어가 풍부하지 않기 때문에 데이터 모델을 설계할 때 칼럼과 칼럼패밀리가 읽기에 최적화되도록 해야한다.

#### 3.5 확장성
카산드라는 단일 마스터 노드가 없기 때문에 노드만 추가하면 성능이 향상된다.

#### 3.6 적절한 사용처
- 이벤트 로깅
- 콘텐츠 관리 시스템, 블로깅 플랫폼
- 카운터
- 기간만료

#### 3.7 사용하지 말아야 할 때
카산드라는 초기 프로토 타입이나 기술 검토용으로는 부적합하다. 
RDBMS는 스키마를 변경하는데 비용이 많이 들지만 카산드라에서는 스키마보다 쿼리변경에 더 많은 비용이 든다.

## 4. 그래프 데이터베이스
그래프 데이터베이스에서는 엔터티와 엔터티 사이에 관계를 저장할 수 있다. 엔터티는 노드라고도 불리며 속성을 가질 수 있다.
각각의 엔터티를 연결하는 간선은 방향을 가지며 엔터티와 같이 속성을 가지고 있어 엔터티간의 관계를 정의한다.
그래프에 대한 쿼리는 그래프 순회라고도 하며 노드나 간선을 변경하지 않고도 순회요건을 바꿀 수 있는 장점이 있다.

#### 4,1 일관성
그래프 데이터베이스는 연결된 노드에 대해 동작해야하므로 일반적으로 분산저장을 지원하지 않는다.
그래프 데이터베이스는 트랜잭션을 통해 일관성을 보장하며 항상 노드의 시작과 끝이 있는 관계만 허용한다.

#### 4.2 트랜잭션
네오4j의 트랜잭션은 finish를 실행할 때 적용된다.
트랜잭션 내부에서 Success를 호출해도 finish가 실행되지 않았으면 데이터베이스에 커밋되지 않는다.

#### 4.3 가용성
네오4j는 복제 슬레이브를 사용해 고가용성을 지원한다. 클러스터에 가장 먼저 참여한 서버가 마스터가 된다.

#### 4.4 확장성
일반적으로 그래프 데이터베이스의 확장은 쉽지 않다.
성능 향상을 위해 고용량의 메모리를 사용하거나 마스터/슬레이브 구조를 사용해 읽기는 슬레이브에서만 하는 방법이 있을 수 있으며
애플리케이션에서 데이터를 분리해 사용하는 방법도 고려해볼 수 있다.

#### 4.5 적절한 사용처
- 연결된 데이터
- 라우팅, 디스패치, 위치기반 서비스
- 추천 엔진

#### 4.6 사용하지 말아야 할 때
일부 엔터티 전부 혹은 일부 엔터티 그룹의 데이터만 변경해야 하는 경우에는 연산이 간단하지 않아 부적합하다.

