## 인덱스 확장 기능

Index scan에는 여러 가지 방법이 존재합니다.

- Index Range Scan
- Index Full Scan
- Index Unique Scan
- Index Skip Scan
- Index Fast Full Scan

### Index Range Scan

B\*Tree 에서 가장 일반적이고 정상적인 액세스 방식입니다. 인덱스 루트에서 리프 블럭까지 수직적으로 탐색하는 방법입니다.
주의할 점은 `선두 컬럼을 가공하지 않은 상태`로 조건절에 사용해야 합니다.

### Index Full Scan

수직적 탐색(인덱스 탐색의 시작점을 찾는 과정) 없이 인덱스 리프 블럭을 처음부터 끝까지 `수평적 탐색` 하는 방법입니다.
대표적으로 쿼리의 조건절에 사용된 컬럼이 인덱스의 첫 번쨰 컬럼이 아닌 경우 사용됩니다. 예를 들어 인덱스가 (A, B, C)로 구성되어 있다고 했을 때 쿼리의 조건절은 B 혹은 C로 검색하는 경우 입니다.

인덱스 레인지 스캔보다는 빠르지 않지만 테이블 풀 스캔보다는 효율적입니다. 인덱스에 포함된 컬럼만으로 쿼리를 처리하는 경우 레코드를 읽을 필요가 없기 때문에 효율적으로 사용할 수 있습니다. 또한 인덱스 전체의 크기는 테이블 자체의 크기보다는 훨씬 작기 때문에 작은 디스크 I/O로 쿼리를 처리할 수 있습니다.

> Index Full Scan도 논리적인 순서에 따라 Single block I/O 방식으로 읽는다.

### Index Unique Scan

수직적 탐색만으로 데이터를 찾는 스캔 방식으로서, Unique 인덱스를 '='조건으로 탐색하는 경우에 동작합니다.
Unique Index 가 존재하는 컬럼은 DBMS가 데이터가 중복되지 않도록 데이터 정합성을 관리해줍니다. 따라서 해당 인덱스 키 컬럼을 '=' 조건으로 탐색하는 경우 데이터를 한 건 찾게 되면 더 이상 탐색할 필요가 없어집니다.

물론 Unique 인덱스여도 범위조건 탐색의 경우 Index Range Scan으로 처리됩니다. 왜냐하면 수직적 탐색만으로는 조건에 맞는 데이터를 모두 찾을 수 없기 때문입니다.

### Index Skip Scan

인덱스 스킵 스캔은 루트 혹은 브랜치 블럭에서 읽은 컬럼 값 정보를 이용해 조건절에 부합하는 레코드를 포함할 `가능성이 있는` 리프 블럭만 골라서 액세스 스캔하는 방식입니다.

#### Loose Index Scan

MySQL에서는 이를 Loose Index Scan이라고 부릅니다.
루스 인덱스 스캔은 인덱스 레인지 스캔과 비슷하게 작동하지만 중간에 필요하지 않은 인덱스 키 값은 무시하고 다음으로 넘어가는 방법입니다.
GROUP BY 혹은 집합 함수 가운데 MAX, MIN 함수에 대해 최적화를 하는 경우 사용됩니다.

인덱스가 (dept_no, emp_no)로 구성되어 있다고 해보겠습니다.

```sql
mysql> SELECT dept_no, MIN(emp_no)
       FROM detp_emp
       WHERE dep_no BETWEEN 'd002' AND 'd004'
       GROUP BY dept_no;
```

위와 같은 쿼리가 있을 때 인덱스가 (dept_no, emp_no) 조합으로 정렬되어 있어 dept_no 그룹의 첫 번쨰 컬럼만 읽으면 됩니다.
즉 옵티마이저는 WHERE 조건을 만족하는 범위 전체를 읽을 필요가 없다는 것을 알고 있기 때문에 만족하지 않은 레코드는 무시하고 다음 레코드로 이동하게 됩니다.

![image](https://github.com/masters2023-project-03-second-hand/second-hand-max-be-b/assets/66981851/ba1102fd-f10e-419c-97c1-754f3b2d3f10)

### Index Fast Full Scan

인덱스 패스트 풀 스캔은 말 그대로 인덱스 풀 스캔보다 빠릅니다. 그 이유는 인덱스의 논리적인 트리 구조를 무시하고 인덱스 세그먼트 전체를 `Multiblock I/O` 방식으로 스캔하기 떄문입니다.

Multiblock I/O 방식을 상요하기 때문에 대량의 인덱스 블럭을 읽을 때 큰 효과를 발휘합니다. 하지만 인덱스 리프노드의 연결리스트 구조를 무시하고 데이터를 읽어오기 때문에 결과집합이 인덱스 키 순서대로 정렬되지 않습니다.

### Direct Path I/O

책을 읽다보면 `Direct Path I/O`라는 용어가 나오는데 해당 부분이 이해가 가지 않아 정리해보려 합니다.

Direct I/O는 시스템의 read/write buffer를 거치지 않고 디스크에 직접 I/O를 수행하는 것을 의미합니다.

왜 DBMS에서 제공해주는 캐싱을 이용하지 않는 걸까요? Direct I/O는 컴퓨터 파일 시스템을 