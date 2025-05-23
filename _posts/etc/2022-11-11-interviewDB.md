---
layout: post
title: 'Database Interview 대비'
subtitle: '인덱스, RDBMS, NoSQL, 정규화, Transaction, ACID, ElasticSearch'
date: 2024-11-27 11:30:00 +0900
categories: [etc]
background: '/img/posts/etc/git.png'
---

### 데이터베이스의 특징은 무엇인가요?
- 실시간 접근성 : 비정형적인 질의에 대하여 실시간 처리에 의한 응답이 가능해야 한다.
- 지속적인 변화 : 데이터베이스의 상태는 동적이다. 새로운 데이터의 삽입, 삭제, 갱신으로 항상 최신의 데이터를 유지해야 한다.
- 동시 공용 : 데이터베이스는 서로 다른 목적을 가진 여러 응용자들을 위한 것이므로 다수의 사용자가 동시에 같은 내용을 이용할 수 있어야 한다. 
- 내용에 의한 참조 : 데이터베이스에 있는 데이터를 참조할 때 레코드의 주소나 위치에 의해서가 아니라 사용자가 요구하는 데이터 내용으로 찾습니다.

<br>

### 데이터베이스에서 인덱스를 사용하는 이유 및 장단점에 대해 설명해주세요.
- DB에서 인덱스를 사용하는 이유는 검색성능을 향상시키기 위해서 입니다. 
- 인덱스는 항상 정렬된 상태를 유지하기 때문에 원하는 값을 검색하는데는 빠르지만, 새로운 값을 추가하거나, 삭제, 수정하는 쿼리문 실행 속도는 느립니다.
- 저장 성능은 희생하고 그 대신 데이터의 검색 속도를  높이는 기능이라 할 수 있습니다.

<br>

### 데이터베이스 언어(DDL, DML, DCL)에 대해 설명해주세요.
- DDL(정의어, Data Definition Language) : 데이터베이스 구조를 수정, 삭제하는 언어(alter, create, drop)
- DML(조작어, Data Manipulation Language) : 데이터베이스 내의 자료 검색, 삽입, 갱신, 삭제를 위한 언어(select, insert, update, delete)
- DCL(제어어, Data Control Language) : 데이터에 대해 무결성 유지, 병행 수행 제어, 보호와 관리를 위한 언어(commit, rollback, grant, revoke)

<br>

### SELECT 쿼리 수행 순서를 알려주세요.
1. FROM, ON, JOIN
2. WHERE, GROUP BY, HAVING
3. SELECT
4. DISTINCT
5. ORDER BY
6. LIMIT

<br>

### varchar 와 nvarchar 의 차이는 뭔가요?
- [MYSQL 데이터 타입](https://www.incodom.kr/DB_-_%EB%8D%B0%EC%9D%B4%ED%84%B0_%ED%83%80%EC%9E%85/MYSQL#h_bd22bd92cae429ff1bef63863384c52b)
- 둘은 모두 가변 문자열이라는 공통점이 있다.
- varchar : 바이트 수를 기준으로 하는 가변 문자열/ 영어, 숫자: 1Byte, 한글,한자 : 2Byte
- nvarchar : 글자 수를 기준으로하는 유니코드 지원 가변 문자열/ 모든 문자 일괄적으로 2Byte
> - + CHAR : 고정형 문자열(n <= 255)

<br>

### DBMS(Database Management System)는 인덱스를 어떻게 관리하고 있나요?
- B+Tree 인덱스 자료구조
> - 자식 노드가 2개 이상인 B-Tree를 개선시킨 자료구조입니다.
> - B-Tree 리프노드들을 LinkedList로 연결하여 순차 검색을 용이하게 합니다. 해시테이블보다 나쁜 O(lon2N)의 시간 복잡도를 갖지만 일반적으로 사용되는 자료구조입니다.
- 해시테이블
> - 컬럼의 값으로 생성된 해시를 기반으로 인덱스를 구현합니다.
> - 시간 복잡가 O(1)이라 검색이 매우 빠릅니다.
> - 부등호와 같은 연속적인 데이터를 위한 순차 검색이 불가능하기 때문에 사용에 적합하지 않습니다.

<br>

인덱스 :  검색 속도를 높이기 위한 색인 기술
- 일반적으로 SELECT 쿼리의 WHERE 에 사용할 컬럼에 대해 효율적인 검색을 위해 사용, 다른 테이블의 JOIN에 사용
> -  일반적으로 SQL 서버에서 데이터 저장할 때 내부적으로 아무런 순서없이 저장 (Heap이라는 곳) 
> - 풀스캔, 테이블 스캔 : 인덱스가 없는 테이블의 데이터를 찾을 때, 전체 데이터 페이지를 처음 레코드부터 끝 페이지 마지막까지 조회한다
- 무조건 INDEX를 많이 설정한다해서 검색 속도가 향상되진 않는다.
- INSERT문에서 성능이 안좋고, SELECT 문에서 좋은 편이다. UPDATE, DELETE는 해당 컬럼을 빨리 찾아주는 것이고 수정, 삭제 성능과는 영향을 안준다. 

핵심적인 기준 4가지
- 카디널리티(Cardinality) : 카디널리티가 높으면 인덱스 설정에 좋고, 높다는 것은 한 칼럼이 갖고 있는 값의 중복도가 낮다는 것을 의미한다.
> - Cardinality : 데이터 집합의 유니크한 원소 갯수
- 선택도(Selectivity) : 선택도가 낮으면 인덱스 설정에 좋고, 선택도가 낮다는 것은 한 컬럼이 갖고 있는 값 하나로 적은 row가 찾아진다.
- 조회 활용도 : 조회 활용도가 높으면 인덱스 설정에 좋은 컬럼이다. WHERE의 대상 컬럼으로 많이 활용되는지 판단.
- 수정 빈도: 수정 빈도가 낮으면 인덱스 설정 컬럼에 좋다. 자주 안바뀌는 것이 좋다. 

- [효과적인 DB index 설정하기](https://velog.io/@jwpark06/%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9D%B8-DB-index-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)

<br>

### 정규화에 대해서 설명해주세요.
- 하나의 릴레이션에 하나의 의미만 존재하도록 릴레이션을 분해하는 과정입니다.
- 데이터의 일관성, 데이터 중복방지, 무결성, 유연성을 충족시키기 위해 데이터베이스를 설계하는 것을 의미합니다. 
- 제 1정규형 : 테이블의 컬럼이 원자값(하나의 값)을 갖도록 분해합니다.
- 제 2정규형 : 제 1정규형을 만족하고 기본키가 아닌 속성이 기본키에 완전 함수 종속이도록 분해합니다.
> - 완전 함수 종속 : 기본키의 부분집합이 다른 값을 결정하지 않는 것을 의미
- 제 3정규형 : 제 2정규형을 만족하고, 이행적 함수 종속을 없애도록 분해합니다.
> - 이행적 종속 : A `->` B,  B `->` C 일 때, A `->` C가 성립하는  것을 의미
- BCNF 정규형 : 제 3정규형을 만족하고, 모든 결정자가 후보키가 되도록 분해합니다. 

- [정규화_슬기로운 개발생활](https://dev-coco.tistory.com/62)

<br>

### 이상 현상의 종류에 대해 말해주세요.
- 테이블 설계시 설계 오류로 인해 데이터를 삽입, 삭제, 수정할 때 생기는 논리적 오류를 말합니다.
> - 삽입 이상 : 자료를 삽입할 때 특정 속성에 해당하는 값이 없어 NULL을 입력해야 하는 현상
> - 갱신 이상 : 중복된 데이터 중 일부만 수정되어 데이터 모순이 일어나는 현상
> - 삭제 이상 : 어떤 정보를 삭제하면, 다른 데이터도 삭제되는 현상
- 이를 막기 위해 데이터 정규화가 필요합니다. 

<br>

### PK, FK 의 특징에 대해 말해주세요.
- PK(Primary Key) 
> - 엔티티를 식별하는 대표 key
> - Null 값을 가질 수 없고 중복된 값을 가질 수 없다.
> - Table당 1개만 지정해야한다.
- FK(Foreign Key)
> - 외부 식별자 key로 테이블 간의 관계를 의미한다.
> - 두 테이블 간의 종속이 필요한 관계이면 그 접점이 되는 컬럼을 FK로 지정하여 서로 참조할 수 있도록 관계를 맺어준다.
> - 테이블 간의 잘못된 매핑을 방지하는 역할도 있다. 

<br>

### 트랜잭션에 대해서 설명해주세요.
- 작업 완정성을 보장해줍니다.
- 작업들이 모두 처리하거나 처리되지 못할 경우이전 상태로 복구하여 작업의 일부만 적용되는 현상을 막아줍니다.
- 하나의 트랜잭션은 Commit 되거나 Rollback 됩니다.

<br>

### 트랜잭션의 특성(ACID)에 대해 말해주세요.
- 원자성(Atomicity) : 작업이 반영되거나 전혀 반영되지 않아야 한다.
- 일관성(Consistency) : 실행이 완료되면 항상 일관성 있는 상태를 유지해야 한다.
- 독립성(Isolation) : 둘 이상의 트랜잭션이 동시에 실행될 경우 서로의 연산에 끼어들 수 없다.
- 영속성(Durability) : 완료된 결과는 영구적으로 반영되어야 한다.

<br>


<br>

### JOIN에 대해서 설명해주세요.
- 두 개 이상의 테이블을 연결해서 조회하는 방법을 말합니다.
- inner join : 서로 연관된 내용만 검색하는 방법입니다. A, B에 대해 수행하는 것은 A, B의 교집합을 말합니다.
- outer join : 한 쪽에 데이터가 있고 한 쪽에 데이터가 없는 경우, 있는 쪽의 내용을 전부  출력하는 방법을 말합니다. A, B에 대해 수행하는 것은 A, B의 합집합을 말합니다.

<br>

### RDBMS vs NOSQL에 대해서 설명해주세요.
- RDBMS(Relational DataBase Management System) : 모든 데이터를  2차원 테이블 형태로 표현합니다.
> - 장점 : 스키마에 맞춰 데이터를 관리하기 때문에 데이터의 정합성을 보장할 수 있다.
> - 단점 : 시스템이 커질 수록 쿼리가 복잡해지고 성능이 저하되며 Scale-out이 어렵다.(장비를 추가해서 확장) (Scale-Up만 가능 : 용량, 사양을 높이는 것)
- NoSQL(Not Only SQL) : 데이터 간 관계를 정의하지 않고, 스키마가 없어 좀 더 자유롭게 데이터를 관리할 수 있으며, 컬렉션이라는 형태로 데이터를 관리합니다.
> - 장점 
> > - 스키마없이 Key-Value 형태로 데이터를 자유롭게 관리할 수 있다. 
> > - 데이터 분산이 용이하여 Scale-Out, Scale-Up 모두 가능하다.
> - 단점 
> > - 데이터 중복이 발생할 수 있고, 중복된 데이터가 변경될 경우 모든 컬렉션에서 수정을 수행해야 한다.
> > - 스키마가 존재하지 않기 때문에 명확한 데이터 구조를 보장하지 않아 데이터 구조 결정이 어려울 수 있다.

<br>

### Redis에 대해서 간단히 설명해주세요.
- Key-value 타입의 인메모리 저장소이며 NoSQL입니다. 싱글 스레드로 동작하며 자료구조를 지원합니다. 문자열, 리스트, 해시, 셋, 정렬된 셋 형식의 데이터를 지원합니다. 
- Redis의 영속성은 스냅샷(순간적으로 메모리에 있는 내용을 DISK에 전체를 옮겨 담는 방식), AOF 방식(Append On File, Redis의 모든 write, update 연산 자체를 모두 log 파일에 기록하는 형태)으로 지원합니다.

<br>

### Redis와 Memcached의 차이에 대해서 설명해주세요.
- Redis : 싱글 스레드 기반, 다양한 자료구조
- Memcached : 멀티 스레드 가능, 문자열 형태만 저장

<br>

### Elastic Search에 대해서 간단히 설명해주세요.
- 자바로 개발된 오픈 소스 검색 엔진입니다. ELK( Elasticsearch / Logstatsh / Kibana )스택으로 사용되기도 합니다.
> - Logstash : 다양한 소스(DB, csv 파일 등)의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 Elasticsearch에 전달
> - Elasticsearch : Logstash로부터 받은 데이터를 검색 및 집계를 하여 필요한 관심 있는 정보를 획득
> - Kibana : Elasticsearch의 빠른 검색을 통해 데이터를 시각화 및 모니터링

<br>

### Elastic Search의 인덱스구조와 RDBMS의 인덱스 구조의 차이에 대해 설명해주세요.
- Elastic Search는 역색인 구조로 데이터를 저장합니다. 특정 단어가 출현하는 doc을 저장하는 형태입니다.
> - 역색인 구조(Inverted Index) : 특정 키워드를 포함하고 있는 문서들에 대한 Primary Key를 매핑하는 인덱스 테이블을 생성하고, 이 테이블을 이용해 빠른 문서 탐색을 가능하게 합니다. 
> > - BTree(B-Tree를 개선시킨 자료구조), Trie(문자열 집합 트리 자료구조), Hash Table(해시함수 기반의 key-value 자료구조)로 주로 구현됨
- RDBMS는 B-Tree와 그와 유사한 인덱스를 사용합니다. 

<br>

### Elastic Search의 키워드 검색과 RDBMS의 LIKE 검색의 차이에 대해 설명해주세요.
- Elastic Search는 역색인 방식으로 저장되기 때문에 동의어나 유의어를 활용한 검색이 가능하며 비정형 데이터의 색인과 검색이 가능합니다.
- RDBMS는 단순 텍스트 매칭에 대한 검색을 제공해 동의어, 유의어 같은 검색은 불가능합니다. 

<br>

### MySQL 8.0 부터 달라진 점
- I/O 바운드 읽기 전용 : 8.0부터 내림차순 인덱스를 지원함에 따라 읽기 성능에서 크게 개선되었습니다.
- 읽기 쓰기 성능 증가
- 이중 쓰기 버퍼, IO 바인딩 읽기 쓰기 성능 증가
- 데이터 딕셔너리 업그레이드 : 5.7까지는 데이터 딕셔너리 정보가 FRM 확장자를 가진 파일로 별도로 보관됐었는데 8.0부터 데이터 딕셔너리 정보가 트랜잭션이 지원되는 InnoDB 테이블로 저장되도록 개선되었다. 또한 호환성 관리를 위해 MySQL 버전 정보도 함께 기록 된다.

<br>

## DB 다중화의 두 가지 예시

### DB Replication 
- 여러 개의 DB를 권한에 따라 수직적인 구조(Master- Slave)로 구축하는 방식
- Master Node 는 쓰기 작업만, Slave Node 는 읽기 작업만 처리한다.
- 리플리케이션은 비동기 방식으로 데이터를 동기화한다.

- 리플리케이션 처리 순서
1. Master 노드에 쓰기 트랜잭션이 수행된다.
2. Master 노드는 데이터를 저장하고 트랜잭션에 대한 로그를 파일에 기록한다.(BIN LOG)
3. Slave 노드의 IO Thread는 Master 노드의 로그 파일(BIN LOG)를 파일(Replay Log)에 복사한다.
4. Slave 노드의 SQL Thread는 파일(Replay Log)를 한 줄씩 읽으며 데이터를 저장한다.

- 리플리케이션은 데이터의 무결성 검사(데이터가 일치하는지)를 하지 않는 비동기 방식으로 데이터를 동기화한다.
- 장점
> - DB 요청의 대부분이 읽기 작업이기 때문에 충분히 성능을 높일 수 있다.
> - 비동기 방식으로 지연 시간이 거의 없다
- 단점
> - 노드 간 데이터 동기화가 보장되지 않아 일관성있는 데이터를 얻지 못할 수도 있다.
> - Master 노드가 다운되면 복구 및 대처가 까다롭다. 

<br>

### DB Clustering
- 여러 개의 DB를 수평적인 구조로 구축하는 방식
- 분산 환경을 구성하여 Single point of failure와 같은 문제를 해결할 수 있는 Fail Over(장애 대비 대응) 시스템을 구축하기 위해 사용한다.
- 동기 방식으로 노드들 간의 데이터를 동기화한다. 

- 클러스터링의 처리 순서
1. 1개의 노드에 쓰기 트랜잭션이 수행되고, COMMIT을 실행한다.
2. 실제 디스크에 내용을 쓰기 전에 다른 노드로 데이터의 복제를 요청한다.
3. 다른 노드에서 복제 요청을 수락했다는 신호(OK)를 보내고, 디스크에 쓰기를 시작한다.
4. 다른 노드로부터 신호(OK)를 받으면 실제 디스크에 데이터를 저장한다.

- 클러스터링은 데이터 무결성 검사(데이터가 일치하는지)를 하는 동기 방식으로 데이터를 동기화한다. 
- 장점
> - 노드들 간의 데이터를 동기화하여 항상 일관성있는 데이터를 얻을 수 있다.
> - 1개 노드가 죽어도 다른 노드가 살아 있어 시스템을 계속 장애없이 운영할 수 있다.
- 단점
> - 여러 노드들 간의 데이터를 동기화하는 시간이 필요하므로 리플리케이션에 비해 성능이 떨어진다.
> - 장애가 전파된 경우 처리가 까다롭고, 데이터 동기화에 의해 스케일링에 한계가 있다.

- 클러스터링의 구현 방식
> - Active-Active : 클러스터를 항상 가동하여 가동 가능한 상태로 두는 구성 방식
> - Active-Standby : 일부 클러스터는 가동하고, 일부 클러스터는 대기 상태로 구성하는 방식
> > - Hot-Standby : 평소에도 Standby DB 작동, 비용이 더 많이 듬
> > - Cold-Standby : Active DB가 다운된 시점에 작동

<br>

### DB 데이터 동시성 접근 문제 해결 방안

- 트랜잭션(데이터를 다루는 작업 단위)을 잘 관리해주어야 한다. 
- Spring의 @Transactional 어노테이션을 통해 어느정도 해결할 수 있다. 
- 트랜잭션 격리 수준

![스크린샷 2023-06-16 오후 3 52 26](https://github.com/iheese/iheese.github.io/assets/88040158/8b7bc221-60dc-4b68-b7af-0ca1bc51076b)

- READ UNCOMMITED : 아무런 락을 걸지 않음, 즉 커밋되지 않은 데이터도 읽을 수 있음
> - Dirdy Read 발생
> - Dirty Read : write 트랜잭션이 데이터를 업데이트만 하고 커밋은 안했는데 read 트랜잭션이 해당 값을 읽고 write 트랜잭션이 롤백한 상황, 이 상황에서 다시 read 트랜잭션이 re-read 할 때 생김

<img width="667" alt="1" src="https://github.com/iheese/iheese.github.io/assets/88040158/0b73da9a-313a-4f36-b1d8-e0e5d7c025c0">

- READ COMMITED : 커밋하기 전 작업 중인 데이터의 값을 읽을 수 없다. 그래서 Dirty Read가 발생하지 않는다. 즉 커밋되어 확정된 것까지는 읽을 수 있음
> - 커밋된 것은 Undo 공간에 있어서 다른 read 트랜잭션이 오면 백업된 것을 리턴해준다.  
> - 해당 격리 레벨부터 Spring에서 undo 영역을 이용한 MVCC(Multiversion Concurrency Control)를 사용하여 Consistence Read를 보장해줍니다. 

<img width="666" alt="22" src="https://github.com/iheese/iheese.github.io/assets/88040158/c5f24e19-06c0-49cc-a965-759cd461ff7e">

<img width="651" alt="2" src="https://github.com/iheese/iheese.github.io/assets/88040158/ef4cb145-f961-4ef7-bb6c-ab3b97594814">

- NON REPEATABLE READ 발생, REPEATABLE READ의 정합성에 문제가 생긴다.

<img width="739" alt="3" src="https://github.com/iheese/iheese.github.io/assets/88040158/adf5ae5c-4be1-4c98-99b4-ba4bf965a02d">

- REPEATABLE READ : 트랜잭션마다 트랜잭션ID가 존재하여 트랜잭션 ID보다 작은 트랜잭션 번호에서 변경한 것만 읽게 된다.  
> - 커밋하기 전 작업 중인 데이터의 값을 읽을 수 없고, 읽기 트랜잭션이 끝나기 전까지 Undo 영역을 타 트랜잭션에서 쓰기 작업 후 커밋한 데이터로 덮어쓸 수 없다. 즉 선행 트랜잭션이 읽은 데이터를 종료까지 갱신/삭제 안됨
> - Phantoom read 가능성 : 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다가 안보였다가 하는 현상 > 쓰기 잠금이 필요하다.

<img width="746" alt="233" src="https://github.com/iheese/iheese.github.io/assets/88040158/f56e5bc4-cb48-4eeb-b636-e9aff162b133">

- 위 그림에서는 리턴값이 달라지는 것에 집중하여 보자.

- SERIALIZABLE : 트랜잭션을 다른 트랜잭션으로부터 완전히 분리시킨다. 특정 트랜잭션을 작업하는 동안 타 트랜잭션은 별로의 작업을 할 수 없다, insert문까지 금지
> - @Transactional 어노테이션만 붙여주면 jdbc 기본 isolation level을 따름
> > - MYSQL : 2, ORACLE : 1, MariaDB : 2 

#### DB Lock

- java의 sysnchronized 를 통해 메소드에서 동시성을 보장할 수 있으나 하나의 프로세스 안에서만 보장이 된다. 여러 대의 서버가 동시에 DB에 접근하게 되면 사용하지 않았을 때와 동일하게 문제가 발생한다. 
> - thread-safe 하긴 하지만 여러 개의 인스턴스가 있으니 의미가 없다. 이럴 때는 아래 DB 락을 고려해봐야한다. 

- Pessimistic Lock(비관적 lock) : 트랜잭션이 충돌할 것이라 가정하고 lock을 설정, 트랜잭션 하나가 처리되고 그 외에는 접근이 불가합니다. 그 후에 다음 트랜잭션이 처리된다.
> - exclusive lock을 걸어 다른 트랜잭션이 접근할 수 없게 한다.
> - 격리 수준을 높여서 데이터 무결성을 보장하는 수준이 높지만, 다른 트랙잭션을 막아 성능상 좋지 않고, 데드락이 발생할 수 있다. 
- Optimistic Lock(낙관적 lock) : 트랜잭션이 처리되고 같은 부분에 트랜잭션 요청이 들어오면 동시성 문제가 발생한 것으로 인지하여 Exception을 발생시키고 롤백합니다.
> - 실제로는 lock을 사용하지 않고 버전을 이용해 데이터 정합성을 맞춘다. 
> - 데이터 동시성 문제가 빈번하지 않다면 낙관적 lock이 성능이 좋습니다. 
> - 잦은롤백 처리 > 성능 악화를 일으킬 수 있습니다. 

```java
 while (true) {
            try {
                optimisticStockService.decrease(id, quantity);
                break;
            } catch (Exception e) {
                Thread.sleep(50);
            }
        }
```

- exception 을 try-catch로 잡아 재시도하게끔 하여 락을 얻어내기도 한다. 

<br>

- Named Lock(metadata locking) : 이름을 가진 메타데이터를 사용하는 방식, 이름을 가진 lock을 획득하고 해제할 때까지 다른 세션이 lock을 획득할 수 없게 한다. 
> - 트랜잭션이 종료될 때 자동으로 락 해제가 안되서 별도의 락 해제 명령어를 사용하거나 선점 시간이 끝나야한다. 
> -  Pessimistic lock과 비슷하지만, row, 테이블 단위가 아니라 metadata 단위로 락을 건다. 
> + named lock이 같은 데이터 소스를 사용하면 커넥션 풀이 부족하는 현상이 발생할 수 있다. 실무에서는 데이터 소스를 분리해서 사용해야한다. 

<br>

#### MongoDB와 Redis의 차이 

![스크린샷 2023-09-19 오후 4 55 58](https://github.com/iheese/iheese.github.io/assets/88040158/e4b7dbfc-4042-45bd-be84-2f9598564ca7)

<br>

#### 추가 학습할 것들

MongoDB에 대해서 간단히 설명해주세요.

CAP 이론과, Eventual Consistency에 대해서 설명해주세요.


<BR>

Reference:
- [Backend-Interview-Question_ ksundong](https://github.com/ksundong/backend-interview-question)
- [신입 개발자 기술면접 질문 정리-데이터베이스_슬기로운 개발 생활](https://dev-coco.tistory.com/158)
- [내가 받은 '백엔드 기술 면접 질문' 모음_wijoonwu](https://velog.io/@wijoonwu/%EB%A9%B4%EC%A0%91-%EC%A7%88%EB%AC%B8)
- [MySQL성능 비교_화니의 블로그](https://hinweis.tistory.com/65?category=942059)
- [트랜잭션의 격리 수준(isolation Level)이란?](https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation)
- [재고시스템으로 알아보는 동시성 이슈 해결(1)](https://dodop-blog.tistory.com/463)