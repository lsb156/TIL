# Index

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [Index의 종류](#index의-종류)
  - [Clustered Index](#clustered-index)
  - [Non-Clustered Index](#non-clustered-index)

인덱스는 데이터베이스 테이블의 검색 속도를 향상 시키기 위해서 사용된다.
인덱스에는 테이블에 있는 하나 이상의 `COLUMN`으로 작성되는 키가 포함되며 이 키는 키 값과 연결된 행을 빠르고 효율적으로 찾을 수 있는 구조에 저장한다.

인덱스를 저장함으로 시스템 부하를 줄여 빠른 검색 속도를 가질 수 있지만 인덱스 저장을 위한 추가적인 저장 공간이 별도로 필요하며 인덱스 생성에 시간이 많이 소요될 수가 있다.

## Index의 종류
Index의 종류로는 크게 `Clustered Index`, `Non-Clustered Index` 두가지로 구분된다.

예를들어 두개의 차이를 설명하자면 `Clustered Index`는 책에서 원하는 내용의 페이지를 알고 그 페이지를 펼치는 것이라면 `Non-Clustered Index`는 목차에서 찾고자 하는 내용의 페이지를 찾고 그 페이지로 이동하는 것과 같고 `Tablescan`은 처음부터 한장씩 넘기면서 내용을 찾는것과 같다.

### Clustered Index
![index-clustered index](../asset/Database/index-clustered%20index.png)

전화번호부에서 이름을 가지고 원하는 사람의 정보를 찾아가는 과정과 비슷하다.
`Clustered Index`를 사용하면 데이터가 `Clustered Index`의 키를 따라 배치되기 떄문에 두 가지 다른 `Clustered Index`를 사용하는것이 불가능하다.

- 인덱스를 생성할 때는 데이터 페이지 전체를 다시 정렬한다.
- 이미 대용량의 데이터가 입력된 상태라면, 업무시간에 클러스터형 인덱스를 생성하는 것은 심각한 시스템 부하를 줄 수 있으므로 신중해야 한다.
- `Clustered Index`는 Index 자체의 리프 페이지가 곧 데이터 페이지이다.
- `Non-Clustered Index`보다 SELECT 속도는 **빠르지만** INSERT, DELETE, UPDATE 속도는 **느리다**
- `Clustered Index`는 성능이 좋지만 테이블에 한 개만 생성이 가능해서 어떤 COLUMN을 기준으로 Index를 생성하는지에 따라 속도 차이가 발생한다.


### Non-Clustered Index
![index-nonclustered index](../asset/Database/index-nonclustered%20index.png)

새로운 `Clustered Index` 트리와 비슷한 인덱스 트리 구조를 만들고, 최 하위레벨에 베이스 데이터가 있는 것이 아니라, 베이스 데이터가 있는 위치를 가리키는 레퍼런스를 가지고 있는 구조.

이렇게 하여 베이스데이터 테이블의 정렬순서와 독립적인 `Non-Clustered Index` 스트럭쳐를 구성할 수 있다.

- `Non-Clustered Index`를 구성할때는 데이터 페이지는 그냥 둔 상태에서 별도의 페이지에 인덱스를 구성한다.
- `Clustered Index`  SELECT 속도는 **느리지만** INSERT, DELETE, UPDATE 속도는 **빠르다**
- `Non-Clustered Index`는 여러개 생성이 가능하지만 남용할 경우 오히려 시스템 성능을 떨어뜨리는 결과를 가져온다.

