---
title: "PostgreSQL - DISTNCT ON"
date: "2019-11-24"
category: "tech"
cover: https://phoenixframework.org
tags:
    - SQL
    - PostgreSQL

---

사이드 프로젝트 하다가 배운 것.

다음과 같은 테이블/데이터가 있다고 하자. ([db-fiddle 링크](https://www.db-fiddle.com/f/dLGvHhfLFV9eKMrD44DbeR/0))

```sql
CREATE TABLE students
(
    id       SERIAL PRIMARY KEY,
    name     TEXT    NOT NULL,
    class_no INTEGER NOT NULL,
    score    INTEGER NOT NULL
);

INSERT INTO students (name, class_no, score)
VALUES ('Bob', 1, 76),
       ('Lily', 1, 82),
       ('Dave', 1, 57),
       ('Kurt', 1, 90),
       ('Swift', 2, 86),
       ('Guido', 2, 90),
       ('Steve', 2, 94);
```

여기에서 각 반(class_no)에서 획득한 가장 높은 점수를 알고 싶다면, `GROUP BY`와 `MAX`를 쓰면 된다.

```sql
SELECT class_no, MAX(score)
FROM students
GROUP BY class_no;
```

| class_no | max |
| -------- | --- |
| 1        | 90  |
| 2        | 94  |

여기에서 각 반의 최고 득점자의 **이름**을 바로 알고 싶다면 어떻게 해야 할까?

무턱대고 name 컬럼을 쿼리에 추가하면, 아래와 같은 에러가 발생한다.

```sql
SELECT class_no, name, MAX(score)
FROM students
GROUP BY class_no;

-- error: column "students.name" must appear in the GROUP BY clause or be used in an aggregate function
```

생각해보면 당연한데, `class_no` 기준으로 묶으라고만 했지, 같은 `class_no`를 갖는 레코드들 중에 어느 `name`을 선택해야 할지 알려주지 않았기 때문이다.

PostgreSQL은 `SELECT DISTINCT` 구문을 지원하는데, 이를 이용하면 원하는 결과를 얻을 수 있다.

```sql
SELECT DISTINCT ON (class_no) *
FROM students
ORDER BY class_no, score DESC;
```

| id  | name  | class_no | score |
| --- | ----- | -------- | ----- |
| 4   | Kurt  | 1        | 90    |
| 7   | Steve | 2        | 94    |

위 쿼리는 이런 과정을 거쳐 실행된다.

1. `students` 테이블을 `class_no` 오름차순, `score` 내림차순으로 정렬한다. (`ORDER BY ...` 부분)
2. `class_no`가 같은 레코드들 중, **순서상 제일 앞에 위치한 놈만 남기고 나머지는 결과에서 제외한다.** (`DISTINCT ON (class_no)` 부분)
3. 모든 컬럼을 출력한다. (`*` 부분)