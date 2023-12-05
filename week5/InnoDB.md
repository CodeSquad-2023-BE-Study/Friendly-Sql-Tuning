# InnoDB 스트리지 엔진 아키텍처

## 프라이머리 키 클러스터링

InnoDB의 모든 테이블들은 프라이머리 키를 기준으로 클러스터링 되어 저장됩니다.
클러스터링이 의미하는 것은 `군집화`인데요, MySQL은 왜 모든 테이블을 PK를 기준으로 클러스터링 했을까요?

클러스티드 인덱스는 DB 테이블의 실제 데이터 레코드들을 레코드의 주소 대신에 실제 레코드의 물리적인 순서로 정렬합니다.
이렇게 하면 Multi block I/O시 (연속적인 페이지를 읽어들일 때) I/O 연산의 이점이 있습니다. <br>
만약 클러스티드 인덱스를 두지 않으면 물리적인 순서를 고려하지 않고 데이터를 저장하게 될 것이고, 이는 DB 엔진이 디스크를 더 많이 읽어야 하는 결과를 초래할 수 있습니다.

## MVCC (Multi Version Concurrency Control)

MVCC의 가장 큰 목적은 잠금 없는 일관된 읽기를 제공하는데 있습니다. (Consistent read)
이때 InnoDB는 언두로그를 이용해 이를 구현합니다.

DB의 레코드를 변경시키는 SQL 쿼리를 실행시키면 InnoDB의 버퍼 풀은 새로운 값으로 업데이트 됩니다.
이때 언두로그의 값은 격리수준에 따라 다르게 업데이트 되는데 만약 `READ_UNCOMMITTED`인 경우 InnoDB 버퍼 풀이 현재 가지고 있는 변경된 데이터를 읽어서 반환합니다.
격리수준이 `READ_COMMITTED` 이상이라면 언두로그에 있는 값을 읽게 됩니다.

이러한 과정을 MVCC라고 하며 하나의 레코드에 대한 여러 버전이 유지되고 필요에 따라 버전에 맞는 레코드를 반환하게 됩니다.

## 잠금없는 일관된 읽기

이처럼 InnoDB 스토리지 엔진은 위의 MVCC를 이용해 잠금을 걸지 않고 읽기 작업을 수행합니다.
잠금을 사용하지 않기 때문에 다른 트랜잭션이 기다리지 않고 읽기 작업이 가능합니다.
격리 수준이 SERIALIZABLE이 아니라면 순수한 읽기 작업은 다른 트랜잭션의 영향을 받지 않을 것입니다.
이를 잠금없는 일관된 읽기라고 표현하며 이 과정에서 언두로그를 사용하게 됩니다.

```
             Session A              Session B

           SET autocommit=0;      SET autocommit=0;
time
|          SELECT * FROM t;
|          empty set
|                                 INSERT INTO t VALUES (1, 2);
|
v          SELECT * FROM t;
           empty set
                                  COMMIT;

           SELECT * FROM t;
           empty set

           COMMIT;

           SELECT * FROM t;
           ---------------------
           |    1    |    2    |
           ---------------------
```

이런 잠금 없는 일관된 읽기는 트랜잭션 시작 시점의 DB 스냅샷을 찍어 수행하기 때문에 DB의 가장 최신버전을 원한다면 격리수준을 READ_COMMITTED로 하거나 locking read를 사용하면 됩니다.

```sql
SELECT * FROM t FOR SHARE;
```

격리 수준이 READ_COMMITTED인 경우 같은 트랜잭션이라 하더라도 읽기 작업마다 새로운 스냅샷을 생성하기 때문에 최신버전을 유지할 수 있습니다.
