# Chapter 04 확장 실습 답안 템플릿

> **과제:** 관계형 데이터베이스와 SQL 시작하기  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter04_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter04_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 주의

이 파일과 캡처 화면에는 실제 비밀번호, 전체 DB 접속 URL, API Key, 개인정보를 기록하지 않습니다.

```text
GitHub 계정 또는 별칭:hangyeom06
과제 작성일:
사용한 AI 도구:
```
---

# 1. 실습 환경과 시작 상태 확인

다음을 실행합니다.

```sql
SELECT current_database();
SELECT current_user;
SELECT current_schema();
SHOW search_path;
SHOW transaction_read_only;
```

| 확인 항목 | 실제 결과 | 의미 |
| --- | --- | --- |
| current_database() | ai_database_book | 현재 연결되어 있는 데이터베이스 이름 |
| current_user | postgres | 현재 접속한 사용자 이름 |
| current_schema() | public | 현재 스키마 이름 |
| search_path | public, "$user" | 스키마 찾는 순서 |
| transaction_read_only | off | 읽기 전용이 아님 |

- [ ] 현재 DB가 `ai_database_book`이다.
- [ ] 변경 가능한 연결인지 확인했다.
- [ ] 실행할 SQL 범위를 확인했다.
- [ ] Auto-commit 상태를 확인했다.

### 변경 SQL을 실행하기 전에 현재 DB와 실행 범위를 확인해야 하는 이유

```text
변경 SQL은 데이터를 수정하거나 삭제해버릴 수 있으므로 현재DB와 실행 범위를 확인해서 원하는 DB에서 어떤 실행을 하게 될 지 확인해야 한다.
```

---

# 2. `public.students` 구조 생성

## 2-1. 실행 전 예상

```text
테이블 이름: public.student
한 행의 의미:학생 한 명의 정보
예상 행 수:0행
기본키:id
필수 열:id, name, email, created_at
중복을 막는 열:id, email
자동 생성 열:id, created_at
```

## 2-2. 실행 파일

```text
code/chapter04/01_create_students.sql
```

## 2-3. 실행 후 확인

```text
테이블 생성 성공 여부: 성공
실제 행 수:0
DBeaver에서 확인한 위치:ai_database_book  Schemas  public  Tables  students
```

### 각 열의 역할

| 열 | 타입 | NULL 가능? | 역할 |
| --- | --- | --- | --- |
| id | int4 | 불가 | 자동 생성되는 기본키 |
| name | varchar(50) | 불가 | 학생 이름 |
| email | varchar(100) | 불가 | 학생 이메일, 중복 불가 |
| major | varchar(100) | 가능 | 전공 |
| grade | int4 | 가능 | 학년 |
| created_at | timestamptz | 불가 | 행이 생성된 시각 |

### `id`를 학번이나 학생 수로 해석하면 안 되는 이유

```text
각 행을 구분하기 위해 자동으로 부여하는 식별자이기 때문에 실제 학번을 뜻하지 않고 행을 삭제하는 등의 변화로 번호 사이 공백이 생길 수 있어 학생 수와 일치하지 않는다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step02_table.png
```

`여기에 테이블 구조 확인 화면을 삽입하세요.`
![alt text](step02_table.png)
---

# 3. 샘플 데이터 6명 입력

## 3-1. 실행 전 예상

```text
현재 행 수:0
실행 후 예상 행 수:6
예상되는 NULL 포함 학생: 윤서진
```

## 3-2. 실행 파일

```text
code/chapter04/02_insert_students.sql
```

## 3-3. 실제 결과

```text
실제 행 수:6
이준호 grade:3
박서연 존재 여부: 있음
윤서진 major:null
윤서진 grade:null
```

### 예상과 실제 비교

```text
예상과 실제가 일치했는가:예
다르다면 이유:
```

### `created_at` 값이 여러 행에서 같을 수 있는 이유

```text
여러 데이터 작업을 묶어서 실행하면 이 작업이 시작된 시간을 기록하기 때문이다.
```

---

# 4. SELECT 복습과 결과 검증

각 문제는 **SQL 실행 전에 예상 행 수를 먼저 작성**합니다.

| 번호 | 조회 문제 | 예상 행 수 | 실제 행 수 | 일치? | 다르면 이유 |
| ---: | --- | ---: | ---: | --- | --- |
| 1 | 전체 학생 | 6 | 6 | 예 |  |
| 2 | 이름·이메일만 조회 | 6 | 6 | 예 |  |
| 3 | 특정 전공 | 2 | 2 | 예 |  |
| 4 | 특정 학년 이상 | 2 | 2 | 예 |  |
| 5 | 두 전공 중 하나 | 3 | 3 | 예 |  |
| 6 | `grade IS NULL` | 1 | 1 | 예 |  |
| 7 | 전공 `DISTINCT` | 4 | 4 | 예 |  |
| 8 | 정렬 후 상위 3명 | 3 | 3 | 예 |  |

## 4-1. 내가 직접 작성한 SQL 2개

```sql
-- SQL 1
SELECT id, name, grade
FROM public.students
WHERE major = '컴퓨터공학'
  AND grade = 4;
```

```text
이 SQL의 한 행 의미:컴퓨터공학 전공 4학년 학생 1명
예상 행 수:1
실제 행 수:1
```

```sql
-- SQL 2
SELECT id, name, grade
FROM public.students
WHERE grade <= 2;
```

```text
이 SQL의 한 행 의미: 2학년 이하인 학생 한 명
예상 행 수:3
실제 행 수:3
```

## 4-2. `= NULL` 대신 `IS NULL`을 사용하는 이유

```text
NULL은 값이 없거나 알 수 없다는 뜻이므로 = 으로 비교할 수 없다. 
```

## 4-3. `ORDER BY` 없이 결과 순서를 믿으면 안 되는 이유

```text
ORDER BY가 없으면 원하는 순서대로 나오지 않기 때문이다.
```

## 4-4. `DISTINCT`가 원본 데이터를 삭제하는 기능인가요?

```text
아니다. DISTICT는 조회 결과에서 중복된 값을 제외하고 보여줄 뿐이다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step04_select.png
```

`여기에 SELECT 핵심 결과 화면을 삽입하세요.`
![alt text](step02_table-1.png)
---

# 5. 내 가상 학생 2명 추가

실명·실제 이메일 대신 가상 데이터를 사용합니다.

## 5-1. 실행 전 계획

```text
학생 A
이름: 서하린
이메일:harin@example.com
전공: 약학과
학년:3

학생 B
이름: 박준석
이메일:junseok@example.com
전공: 수학과
학년 또는 NULL: NULL

현재 행 수:6
추가 후 예상 행 수:8
```

## 5-2. 내가 실행한 INSERT

```sql
INSERT INTO public.students (name, email, major, grade)
VALUES
    ('서하린', 'harin@example.com', '약학과', 3),
    ('박준석', 'junseok@example.com', '수학과', NULL)
RETURNING name, email, major, grade;
```

## 5-3. 실제 결과

```text
RETURNING 또는 확인 SELECT 결과:서하린 | harin@example.com | 약학과 | 3
박준석 | junseok@example.com | 수학과 | NULL
실제 전체 행 수:8
예상과 일치 여부: 일치
```

### 내가 일부 값을 NULL로 둔 이유 또는 NULL을 사용하지 않은 이유

```text
박준석의 학년 정보가 아직 없는 것으로 설정했기 때문이다.
```

---

# 6. 안전한 UPDATE

내가 추가한 가상 학생 한 명만 수정합니다.

## 6-1. 먼저 대상 확인 SELECT

```sql
SELECT id, name, email, major, grade
FROM public.students
WHERE email = 'harin@example.com';
```

```text
예상 대상 행 수:1
실제 대상 행 수:1
```

## 6-2. UPDATE

```sql
UPDATE public.students
SET grade = 4
WHERE email = 'harin@example.com'
  AND grade = 3
RETURNING name, email, major, grade;
```

```text
예상 영향 행 수:1
실제 영향 행 수:1
RETURNING 결과:서하린 | harin@example.com | 약학과 | 4
```

## 6-3. UPDATE 후 재조회

```sql
SELECT id, name, email, major, grade
FROM public.students
WHERE email = 'harin@example.com';
```

### `WHERE` 없는 UPDATE를 실행하면 위험한 이유

```text
WHERE이 없으면 테이블의 모든 행이 수정될 수 있기 때문에 수정할 행을 지정해줘여 한다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step06_update.png
```

`여기에 UPDATE 전/후 결과 화면을 삽입하세요.`
![alt text](step06_update.png)
---

# 7. 안전한 DELETE

내가 추가한 가상 학생 한 명을 삭제합니다.

## 7-1. 삭제 전 확인

```sql
SELECT id, name, email, major, grade
FROM public.students
WHERE email = 'junseok@example.com';
```

```text
예상 대상 행 수:1
실제 대상 행 수:1
```

## 7-2. DELETE

```sql
DELETE FROM public.students
WHERE email = 'junseok@example.com'
RETURNING name, email, major, grade;
```

```text
예상 영향 행 수:1
실제 영향 행 수:1
RETURNING 결과:박준석 | junseok@example.com | 수학과 | NULL
```

## 7-3. 삭제 후 재조회

```sql
SELECT id, name, email, major, grade
FROM public.students
WHERE email = 'junseok@example.com';
```

```text
삭제 후 같은 조건의 SELECT 결과 행 수:0
```

### `DELETE` 성공 메시지만 보고 끝내지 않고 다시 SELECT해야 하는 이유

```text
삭제 대상이 실제로 사라졌는지 확인하고, 내가 삭제하려던 행과 같은 조건의 행이 남아 있는지 확인해야 한다.
```

---

# 8. 본문 기준 UPDATE·DELETE 상태 검증

`04_update_delete_students.sql`을 본문 시작 상태에서 실행했다면 다음을 확인합니다.

```text
최종 학생 수:
이준호 grade:
박서연 존재 여부:
```

본문 기준 기대 상태와 비교합니다.

```text
학생 수 = 5
이준호 grade = 4
박서연 = 0행
```

### 내 실제 결과가 기준과 다르다면 원인

```text
초기 데이터 상태가 아니라서 오류가 발생했다. sql에서는 6명의 학생인 경우를 가정하고 있지만 실제로 박준석 학생 삭제 후 7명이 남은 상태이기 때문에 이준호의 학년 변경과 박서연 삭제 등의 변경은 반영되지 않았다.
```

---

# 9. 의도한 실패 2개 관찰

> 실패 테스트는 데이터베이스 규칙이 실제로 데이터를 보호하는지 확인하는 실험입니다.

## 9-1. 중복 이메일 `UNIQUE` 오류

내가 사용한 SQL:

```sql
INSERT INTO public.students (name, email, major, grade)
VALUES ('박하린', 'harin@example.com', '약학과', 1);
```

```text
오류 메시지 핵심 단서: 중복된 키 값
왜 실패해야 맞는가: 이메일이 중복되었기 때문이다.
어떤 규칙이 작동했는가: students_email_key의 고유 제약 조건
실패 후 기존 데이터가 어떻게 유지되었는가:중복 행은 추가되지 않고 기존의 서하린 데이터는 유지되었다.
```

## 9-2. 이름 `NULL` 입력 `NOT NULL` 오류

내가 사용한 SQL:

```sql
INSERT INTO public.students (name, email, major, grade)
VALUES (NULL, 'null@example.com', '수학과', 1);
```

```text
오류 메시지 핵심 단서:not null 제약조건 위반
왜 실패해야 맞는가:name은 필수 입력 열인데 null이 되었기 때문이다.
어떤 규칙이 작동했는가:name 열의 not null 제약조건
```

### 실패한 INSERT 뒤 자동 생성 `id` 번호에 빈 구간이 생길 수 있어도 문제라고 단정할 수 없는 이유

```text
INSERT가 실패하더라도 사용된 id는 되돌려지지 않을 수 있어 id에 빈 구간이 생길 수 있다. id는 행의 고유한 값이지 순서나 개수를 나타내는 것이 아니기 때문에 문제가 아니다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step09_constraint_error.png
```

`여기에 제약조건 오류 화면을 삽입하세요.`
![alt text](step09_constraint_error.png)
---

# 10. `verify_students.sql`로 최종 상태 확인

실행 파일:

```text
code/chapter04/verify_students.sql
```

```text
현재 전체 학생 수:7
NULL 개수:2
이준호 grade:3
박서연 존재 여부:있음
현재 데이터 상태에서 예상과 다른 부분: 없다
```

### 검증 SQL을 따로 두면 좋은 이유

```text
변경 작업이 끝난 뒤 데이터가 의도한 상태인지 확인할 수 있다.
```

---

# 11. AI를 SQL 작성자가 아니라 검토자로 활용

먼저 본인이 SQL을 작성한 뒤 AI에게 검토를 요청합니다.

## 11-1. 내가 작성한 SQL

```sql
SELECT id, name, grade
FROM public.students
WHERE grade <= 2;
```

## 11-2. AI에게 전달한 핵심 요청

```text
이 SQL의 조건이 ‘2학년 이하 학생 조회’라는 목적에 맞는지, NULL 처리와 예상 행 수를 검토해줘.
```

## 11-3. AI 검토 결과

| AI 제안 | 수용 / 수정 / 거절 | 실제 검증 결과 | 나의 이유 |
| --- | --- | --- | --- |
| 결과 순서를 일정하게 보려면 ORDER BY id를 추가한다 | 수용 | 3행 조회 | 결과를 id 순으로 확인하기 편하다 |
| grade <= 2는 학년이 null인 학생을 제외한다 | 수용 | 3행 조회 | 학년이 확인된 학생 중 2학년 이하를 찾는 목적에 맞다 |
| grade <= 2 대신 grade between 1 and 2를 사용하면 0이나 음수 학년도 제외할 수 있다 | 수정 | 3행 조회 | 학년이 1학년부터 시작한다는 조건을 넣어서 수정한다 |

### AI가 예상한 영향 행 수와 실제 결과가 같았나요?

```text
예상한 조회 결과는 3행이고 실제로도 3행이 조회되었다.
```

### AI 답변을 실행 전에 검토해야 하는 이유

```text
AI가 제안한 SQL이 내 데이터와 작업 목적에 맞지 않을 수 있기 때문이다.
```

---

# 12. 내 서비스 테이블 하나 확장 설계

Chapter 01~03에서 정한 개인 서비스에서 **테이블 하나**를 선택합니다.

```text
서비스 이름:도서 대여 서비스
테이블 이름:loans
한 행의 의미: 회원 한 명이 책 한권을 한 번 대여한 기록
```

| 열 이름 | 저장할 값 | 타입 후보 | NULL 가능? | UNIQUE 후보? | 이유 |
| --- | --- | --- | --- | --- | --- |
| loan_id | 대출 기록 id | integer, identity | 불가능 | 예 | 각 대출 기록을 구별하는 기본키이기 때문이다 |
| member_id | 대여한 회원 id | integer | 불가 | 아니오 | 한 명이 여러 권의 책을 대출할 수 있기 때문이다 |
| book_id | 대여한 책의 id | integer | 불가 | 아니오 | 같은 책이 여러 번 대출될 수 있기 때문이다 |
| loan_date | 대출일 | date | 불가 | 아니오 | 대출이 시작된 날짜는 중복이 가능하기 때문이다 |
| return_date | 실제 반납일 | date | 가능| 아니오 | 아직 반납되지 않은 책은 값이 없을 수 있다 |

```text
PK 후보:loan_id
업무 식별자 후보:서비스에서 별도로 대출번호를 발급하면 loan_number을 UNIQUE로 설정할 수 있다
아직 미확정인 규칙: 대여 기간, 연체 기준, 한 번에 빌릴 수 있는 책의 수
```

## 선택: CREATE TABLE 초안

> 아직 확정되지 않은 업무 규칙은 억지로 제약조건으로 만들지 않습니다.

```sql

```

### AI에게 검토받은 뒤 수정한 부분

```text

```

---

# 13. 최종 성찰

아래 문장은 본인의 말로 작성합니다.

```text
1. SQL 실행 성공과 올바른 대상 선택이 다른 이유는
   오류 없이 실행되어도 내가 원하지 않은 데이터가 바뀔 수 있기 때문 이다.

2. UPDATE와 DELETE 전에 SELECT를 먼저 해야 하는 이유는
   조건에 해당하는 행이 내가 수정하거나 삭제하려는 대상인지 확인하기 위해서 이다.

3. 영향받은 행 수를 확인해야 하는 이유는
   예상한 수만큼의 행이 실제로 수정되거나 삭제됐는지 알기 위해서 이다.

4. UNIQUE 또는 NOT NULL 오류를 '보호 장치가 정상 동작한 결과'라고 볼 수 있는 이유는
   중복되거나 필수값이 누락된 데이터가 데이터베이스에 들어가는 것을 막아주기 때문 이다.

5. AI가 SQL을 만들어 주더라도 내가 반드시 확인해야 하는 것은
   실행할 데이터베이스와 대상, 조건, 예상 결과가 의도에 부합하는지 이다.
```

---

# 14. 제출 체크리스트

- [ ] `chapter04_answer.md`를 본인 저장소에 만들었다.
- [ ] 현재 DB와 실행 환경을 확인했다.
- [ ] `public.students`를 생성했다.
- [ ] 샘플 6명 입력 결과를 검증했다.
- [ ] SELECT 문제에서 실행 전 예상 행 수를 작성했다.
- [ ] 가상 학생 2명을 추가했다.
- [ ] UPDATE 전후를 SELECT로 확인했다.
- [ ] DELETE 전후를 SELECT로 확인했다.
- [ ] UNIQUE 오류를 관찰했다.
- [ ] NOT NULL 오류를 관찰했다.
- [ ] `verify_students.sql`로 상태를 확인했다.
- [ ] AI 제안을 실제 SQL 결과와 비교했다.
- [ ] 개인 서비스 테이블 하나를 확장 설계했다.
- [ ] 핵심 캡처는 3~4장 정도로 제한했다.
- [ ] 비밀번호·개인정보가 캡처에 없다.
- [ ] Markdown 이미지가 GitHub 웹 화면에서 정상 표시된다.
- [ ] commit/push를 완료했다.

---

# 15. LMS 제출 URL

아래 형식의 **본인 GitHub 파일 URL**을 LMS에 제출합니다.

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter04/chapter04_answer.md
```

내 제출 URL:

```text

```

> 교수자 템플릿 URL이나 저장소 메인 URL이 아니라 **작성 완료된 본인 `chapter04_answer.md` 파일 화면 URL**을 제출합니다.
