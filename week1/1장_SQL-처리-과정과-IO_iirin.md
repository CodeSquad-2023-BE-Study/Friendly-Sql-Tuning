# 1.1 SQL 파싱과 최적화

> **목적**
> 
> 
> 옵티마이저가 SQL을 어떻게 처리하는지, 서버 프로세스는 데이터를 어떻게 읽고 저장하는지 이해한다.
> 

## 1.1.1 구조적, 집합적, 선언적 질의 언어

SQL은 `Structured Query Language` 라는 뜻입니다.

- SQL is a set-based, declarative query language, not an imperative language such as C or BASIC.

SQL은 기본적으로 구조적이고 집합적이고 선언적인 질의 언어입니다.

그러나 그 결과 집합을 만드는 과정은 절차적일 수밖에 없습니다. 즉, **프로시저**가 필요합니다.

이런 프로시저를 만들어내는 DBMS 내부 엔진이 바로 SQL 옵티마이저입니다.

<aside>
🤔 옵티마이저는 구조적, 집합적, 선언적 질의언어인 SQL을 파싱하고 실행계획을 세워 절차적인 프로시저를 만듭니다.

</aside>

DBMS 내부에서 프로시저를 작성하고 컴파일해 실행 가능한 상태로 만드는 전 과정을 SQL 최적화라고 합니다.

## 1.1.2 SQL 최적화

- 전 과정
    1. **SQL 파서가 SQL 파싱을 진행합니다.**
        1. 파싱 트리를 생성합니다.
        2. Syntax 체크 : 문법적 오류 체크
        3. Semantic 체크 : 의미상 오류 체크
    2. **옵티마이저가 SQL 최적화를 진행합니다.**
        1. 미리 수집한 시스템 및 오브젝트 통계정보를 바탕으로 다양한 실행경로를 생성
        2. 비교
        3. 가장 효율적인 하나를 선택
    3. **로우 소스 생성**
        1. 로우 소스 생성기가 이전 단계에서 선택된 실행경로로 실행 가능한 코드 또는 프로시저 형태로 포맷팅

## 1.1.3 SQL 옵티마이저

사용자가 원하는 작업을 가장 효율적으로 수행할 수 있는 최적의 데이터 액세스 경로를 선택해주는 DBMS의 핵심 엔진입니다.

- 최적화 단계
    - 쿼리를 수행하는 데 후보군이 될만한 실행계획들을 찾아낸다.
    - 실행 계획의 예상 비용을 산정한다.
    - 최저 비용을 나타내는 실행 계획을 선택한다.

<aside>
🤔 이게 무슨 뜻이지?

옵티마이저는 별도 프로세스가 아니라 서버 프로세스가 가진 기능일 뿐이다. SQL 파서와 로우 소스 생성기도 마찬가지이다.

[서버프로세스란 무엇인가요?](http://www.gurubee.net/lecture/1081) (오라클을 예시로)

</aside>

## 1.1.4 실행계획과 비용

SQL 옵티마이저가 생성한 처리결과를 사용자가 확인할 수 있도록 트리구조로 표현한 것이 실행계획 Excution Plan 입니다.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F1feb7462-9c33-4bf1-b0bb-7973d34ffaf2%2F6f2bfc2e-ad5d-4b7b-9ab2-8d5c1ac9981e%2FUntitled.png?table=block&id=7ef48be1-1391-45c2-97e6-d1dacab57040&spaceId=1feb7462-9c33-4bf1-b0bb-7973d34ffaf2&width=2000&userId=180a704c-6552-4796-9dd2-ab125439ed98&cache=v2)

비용(Cost)는 쿼리를 수행하는 동안 발생할 것으로 예상하는 I/O 횟수 또는 예상 소요시간을 표현한 값입니다.

그러나 SQL 실행계획에 표시되는 Cost도 어디까지나 예상치입니다. 실제 수행할 때 발생하는 I/O 또는 시간과 많은 차이가 납니다.

## 1.1.5 옵티마이저 힌트

개발자가 직접 효율적인 액세스 경로를 찾아낼 수 있습니다.

옵티마이저 힌트를 이용해 데이터 액세스 경로를 바꿀 수 있습니다. `+` 

```sql
SELECT /*+ INDEX(A 고객_PK) */
	고객명, 연락처, 주소, 가입일시
FROM 고객 A
WHERE 고객ID = '00008'
```

- 주의사항
    - 힌트와 힌트 사이에 `,` 를 사용하지 않습니다. : 첫번째 힌트만 유효하게 됩니다.
    - 스키마명까지 명시하지 않습니다.
    - ALIAS를 테이블 명 옆에 지정했다면 힌트에도 사용해줘야합니다.

# 1.2 SQL 공유 및 재사용

> **목적**
> 
> 
> 소프트 파싱과 하드 파싱의 차이점을 이해합니다. → 시스템에서 바인드 변수가 왜 중요한지 자연스럽게 이해하게 됩니다.
> 

## 1.2.1 소프트 파싱 vs. 하드 파싱

라이브러리 캐시(Library Cache) 란 내부 프로시저를 반복 재사용할 수 있도록 캐싱해두는 메모리 공간입니다. SGA 구성요소입니다.

SGA (System Global Area)는 서버 프로세스와 백그라운 프로세스가 공통으로 액세스하는 데이터, 제어구조를 캐싱하는 메모리 공간입니다.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F1feb7462-9c33-4bf1-b0bb-7973d34ffaf2%2Fbb417e8c-c6d6-436d-b7a2-3be03a341a64%2FUntitled.png?table=block&id=10f6ec8d-f61b-44eb-8728-d99a3acd8634&spaceId=1feb7462-9c33-4bf1-b0bb-7973d34ffaf2&width=2000&userId=180a704c-6552-4796-9dd2-ab125439ed98&cache=v2)

친절한 SQL 튜닝 29pg

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F1feb7462-9c33-4bf1-b0bb-7973d34ffaf2%2F4f3bd4e2-9758-4cd2-9bec-8e73309d4263%2FUntitled.png?table=block&id=8cdcfd4c-3267-4076-94ea-62823efd31d1&spaceId=1feb7462-9c33-4bf1-b0bb-7973d34ffaf2&width=2000&userId=180a704c-6552-4796-9dd2-ab125439ed98&cache=v2)

### SQL 최적화 과정은 왜 하드한가?

- 옵티마이저는 짧은 시간동안 엄청나게 많은 연산을 합니다.
- 옵티마가 고려하는 것들
    - 테이블, 컬럼, 인덱스 구조에 관한 기본 정보
    - 오브젝트 통계
    - 시스템 통계 : CPU 속도, Single Block I/O 속도, Multiblock I/O 속도 등
    - 옵티마이저 관련 파라미터

수많은 실행경로를 도출하고 짧은 순간에 딕셔너리와 통계정보로 효율성을 판단하는 과정은 가볍지 않습니다.

하드 파싱은 CPU를 많이 소비하는 몇 안되는 작업 중하나입니다.

**이 과정을 거쳐 생성한 내부 프로시저를 여러번 사용하여 효율성을 극대화 하고자 합니다. 그래서 라이브러리 캐시가 필요한 것입니다.**

## 1.2.2 바인드 변수의 중요성

SQL은 이름이 따로 없고 전체 SQL 텍스트가 이름 역할을 합니다. 그래서 조금만 수정되어도 다른 객체가 생성됩니다.

캐시에 적재해서 여러 사용자가 공유하면서 재사용하다가 캐시 공간이 부족하면 버려졌다 다음 실행시 똑같은 최적화 과정을 거쳐 캐시에 적재됩니다.

DBMS는 일회성/무효화된 SQL까지 모두 저장하면 많은 공간이 필요하고, SQL 찾는 속도도 느려지므로 SQL을 영구 저장하지 않는 쪽을 선택했습니다.

<aside>
🤔 딕셔너리가 뭐지

https://technet.tmaxsoft.com/upload/download/online/tibero/pver-20150504-000001/reference/ch_data_dictionary_concept.html

</aside>

### 공유 가능 SQL

파라미터로 받는 프로시저 하나를 공유하면서 재사용할 수 있습니다.

바인드변수를 이용하여 파라미터 Driven 방식으로 SQL을 작성하는 방법이 제공됩니다.

```json
String stms = "SELECT * FROM CUSTOMER WHERE LOGIN_ID = ?";
// 아래 ?에 변수를 넣어줍니다.
PreparedStatement st = con.prepareStatement(stms);
st.setString(1, login_id);
ResultSet rs = st.executeQuery();
```

이 SQL에 대한 하드파싱은 최초 1회만 일어나고, 캐싱된 SQL을 다른 이용자가 공유하면서 재사용됩니다.

# 1.3 데이터 저장 구조 및 I/O 매커니즘

> **목적**
> 
> 
> SQL튜닝은 곧 I/O 튜닝이나 다름없다. 데이터 저장 구조, 디스크 및 메모리에서 데이터를 읽는 메커니즘을 이해하자.
> 

## 1.3.1 SQL이 느린 이유

디스크 I/O 때문입니다.

### I/O를 처리하는 동안 프로세스는 잠을 잡니다.

OS 혹은 I/O 서브 시스템이 I/O를 처리하는 동안 프로세스는 멈춰있습니다. (여기서 I/O는 디스크 I/O)

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F1feb7462-9c33-4bf1-b0bb-7973d34ffaf2%2Fef0942dd-8b34-4d97-aa26-aa94ebdb54a8%2FUntitled.png?table=block&id=2aebc80b-62ec-48b7-85db-c65ba6d3f18d&spaceId=1feb7462-9c33-4bf1-b0bb-7973d34ffaf2&width=2000&userId=180a704c-6552-4796-9dd2-ab125439ed98&cache=v2)

프로세스가 디스크에서 데이터를 읽어야 할 땐 CPU를 OS에 반환하고 잠시 waiting 상태가 되어 I/O가 완료되기를 기다립니다.

프로세스가 일하지않고 잠자다니 I/O가 많으면 성능이 느릴 수 밖에 없습니다.

## 1.3.2 데이터베이스 저장 구조

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F1feb7462-9c33-4bf1-b0bb-7973d34ffaf2%2F0f7d1a4f-9876-4571-b2be-14d590d2156f%2FUntitled.png?table=block&id=1ce6683f-beab-4740-8552-914e5a862c30&spaceId=1feb7462-9c33-4bf1-b0bb-7973d34ffaf2&width=2000&userId=180a704c-6552-4796-9dd2-ab125439ed98&cache=v2)

한 블록, 한 익스텐트는 하나의 테이블이 독점합니다.

세그먼트 공간이 부족해지면 테이블 스페이스로부터 익스텐트를 추가로 할당받는데, 세그먼트에 할당된 모든 익스텐트가 같은 데이터파일에 위치하지 않을 수 있습니다.

파일 경합을 줄이기 위해 DBMS가 데이터를 가능한 여러 데이터파일로 분산해서 저장합니다.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F1feb7462-9c33-4bf1-b0bb-7973d34ffaf2%2F0b8dfa3d-b1f6-4356-89e1-8d777fbc2837%2FUntitled.png?table=block&id=fe067fac-3001-4746-9d3e-40ebc8d56829&spaceId=1feb7462-9c33-4bf1-b0bb-7973d34ffaf2&width=2000&userId=180a704c-6552-4796-9dd2-ab125439ed98&cache=v2)

<aside>
🤔 **DBA Data Block Address**

데이터를 읽고 쓰는 단위가 블록이므로 데이터를 읽으려면 이것부터 확인합니다.

인덱스를 이용해 테이블 레코드를 읽을 때는 인덱스 ROWID를 이용합니다.

- ROWID = DBA + 로우 번호
</aside>

- 블록 : 데이터를 읽고 쓰는 단위
- 익스텐트 : 공간을 확장하는 단위, 연속된 블록 집합
- 세그먼트 : 데이터 저장공간이 필요한 오브젝트
- 테이블 스페이스 : 세그먼트를 담는 콘테이너
- 데이터파일 : 디스크상의 물리적인 파일

## 1.3.3 블록단위 I/O

블록이 DBMS가 데이터를 읽고 쓰는 단위입니다. 레코드 하나를 읽고싶어도 블록을 통째로 읽습니다.

테이블 뿐만 아니라 인덱스도 블록 단위로 읽고 씁니다.

## 1.3.4 시퀀셜 액세스 vs. 랜덤 액세스

블록을 액세스하는 방식으로 두가지가 있습니다.

- 시퀀셜 액세스 : 논리적 또는 물리적으로 연결된 순서에 따라 차례로 블록을 읽습니다.

![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F1feb7462-9c33-4bf1-b0bb-7973d34ffaf2%2Ff58835a0-05e2-45c0-84f3-9f808e0ec195%2FUntitled.png?table=block&id=725a5ee0-a1d6-4b2f-be7a-0fd1daa6c281&spaceId=1feb7462-9c33-4bf1-b0bb-7973d34ffaf2&width=2000&userId=180a704c-6552-4796-9dd2-ab125439ed98&cache=v2)

테이블 블록간에는 어떻게 시퀀셜 방식으로 액세스할까요?

오라클의 경우 세그먼트에 할당된 익스텐트 목록을 세그먼트 헤더에 맵을 ㅗ관리합니다.

<aside>
🤔 **MySQL은 논클러스티드 인덱스일경우 테이블블록에 시퀀셜 액세스를 하지 않는게 맞나요?**

랜덤 액세스가 잦을경우 오히려 풀테이블 스캔 뒤 필터링하는게 빠르다고도 들은것같아서 질문하고 싶었어요.

GPT의 의견 : MySQL은 일반적으로 테이블 블록에 시퀀셜 액세스 방식으로 데이터에 접근하지 않습니다. MySQL은 인덱스를 사용하여 데이터를 검색하고 가져오는 것이 일반적입니다. 인덱스는 데이터베이스 성능을 향상시키는 데 중요한 역할을 합니다. 인덱스를 사용하면 특정 조건에 맞는 레코드를 빠르게 찾을 수 있으므로 데이터베이스의 검색 및 필터링 작업이 효율적으로 수행됩니다.

---

👉 MySQL 논 클러스티드는 클러스티드 인덱스

</aside>

- 랜덤 액세스 : 논리적, 물리적인 순서를 따르지 않고 레코드 하나를 읽기 위해 한 블록씩 접근 하는 방식입니다.

## 1.3.5 논리적 I/O vs. 물리적 I/O

### DB 버퍼캐시

캐시에서 블록을 찾는다면 프로세스는 I/O call을 안해도 됩니다.

버퍼캐시는 공유 메모리 영역이므로 같은 블록을 읽는 다른 프로세스도 공유받을 수 있습니다.

### 논리적 I/O 와 물리적 I/O

SQL을 처리하는 과정에 발생한 총 블록 I/O를 말합니다. → 전기적 신호

이중 버퍼 캐시에서 찾지 못할 경우 디스크 액세스(물리적 I/O)를 합니다. → 액세스 암Arm을 통한 물리적 작용이 일어나므로 보통 1만배 쯤 느립니다.

<aside>
🤔 **액세스 암이 뭐지?**

자기 디스크 표면에 데이터를 기록하거나 기록한 데이터를 호출하기 위하여 사용하는 전축바늘과 같이 헤드가 있는 봉.

</aside>

### 버퍼 캐시 히트율

- Buffer Cache Hit Ratio, BCHR

```json
BCHR = (캐시에서 찾은 블록수 / 총 읽은 블록수) *100
		= ((논리적I/O - 물리적I/O) / 논리적I/O) *100
		= (1 - (물리적I/O)/(논리적I/O)) *100
```

온라인 트랜잭션을 주로 처리하는 애플리케이션이라면 99% 히트율ㅇ르 달성해야 합니다.

**논리적 I/O를 줄임으로서 물리적I/O를 줄이는 것이 곧 SQL 튜닝입니다.**

## 1.3.6 Single Block I/O vs. Multiblock I/O

- 싱글블록 I/O : 한번에 한 블록씩 요청합니다.
    - 인덱스 소량 데이터 읽을 때 사용하면 효율적입니다.
- 멀티블록 I/O : 한번에 여러블록씩 요청합니다.
    - 최대한 많이 적재하여 인접한 블록들까지 한꺼번에 읽어와 캐시에 적재합니다.
    - 보통 OS단에서 1MB단위로 I/O를 수행합니다.
    - 인접한 블록이란 같은 익스텐트에 속하는 블록입니다.

### 1.3.7 Table Full Scan vs. Index Range Scan

- Table Full Scan : 테이블 전체를 스캔해서 읽습니다.
- Index Range Scan : 인덱스를 이용해서 읽는 방식입니다.

데이터베이스를 효과적으로 이용하는데 인덱스는 중요하지만, 인덱스가 항상 옳은 것은 아닙니다.

읽을 데이터가 일정량을 넘으면 인덱스보다 Table Full Scan이 유리합니다.

## 1.3.8 캐시 탐색 메커니즘

버퍼캐시에서 블록을 찾을 때는 해시 알고리즘으로 버퍼헤더를 얻고, 거기서 포인터로 버퍼 블록을 액세스합니다.

### 메모리 공유자원에 대한 액세스 직렬화

줄세우기 곧, 직렬화 메커니즘이 필요합니다.

이를 지원하는 메커니즘 래치 입니다. 캐시버퍼 체인 래치, LRU 체인 래치 등
