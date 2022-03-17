# Snowflake

Snowflake 는 SaaS 형태로 제공되는 데이터 플랫폼이다. 클라우드 기반의 데이터 저장, 처리, 분석 솔루션을 제공한다.

## 클라우드 서비스로써 데이터 플랫폼

Snowflake 는 SaaS 형태로 제공된다.

- 실물 하드웨어가 없다.
- 설치, 구성, 관리할 소프트웨어가 없다
- 유지보수, 운영, 업그레이드, 튜닝 등을 모두 Snowflake 에서 주관한다.

Snowflake 는 완전히 클라우드 인프라 위에서 동작한다. Snowflake의 모든 컴포넌트는 퍼블릭 클라우드에 존재하므로, 프라이빗 클라우드나 온프레미스에서 사용할 수 없다. 또한 패키징된 소프트웨어가 아니므로 유저에게 설치형으로 제공될 수 없다.

## 아키텍처

전통적 공유 디스크와 공유하지 않는 데이터베이스 아키텍처의 혼합구조로 이루어져있다. 영구적 저장을 위한 중앙 데이터 저장소가 존재하고, 모든 컴퓨팅 노드가 접근이 가능하다. 반대로 MPP 엔진을 통해 저장된 데이터를 처리할 수 있다. 

3개의 레이어로 구성되어있다.

### 데이터베이스 스토리지

데이터가 Snowflake 로 인입되면, 내부적으로 최적화된 형태로 재정렬시킨다. Snowflake 는 데이터가 저장된 모든 관점을 관리한다. 조직, 파일 크기, 구조, 압축, 메타데이터, 통계 등을 포함할 수 있다. 사용자는 Snowflake 의 데이터를 SQL 을 통해서만 접근할 수 있다.

### 쿼리 프로세싱

프로세싱 레이어에서 실행되는 쿼리 실행을 말한다. Snowflake 는 가상 DW 를 사용하여 처리한다. 가상 DW는 MPP 클러스터를 말하며, 각 가상 DW는 독립적이기 때문에 각 DW 간 성능에 영향을 주지 않도록 설계되어있다.

### 클라우드 서비스

클라우드 서비스는 Snowflake 를 통해 제공받는 서비스들을 말한다. 이 서비스들은 인증, 인프라 관리, 메타데이터 관리, 쿼리 파싱 및 최적화, 권한 관리 등을 처리하기 위한 각각의 다른 컴포넌트로 구성되어있다.