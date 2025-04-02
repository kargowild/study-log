### CHAR, VARCHAR의 내부 크기
- MySQL은 기본적으로 CHAR, VARCHAR는 모두 UTF-8 형태를 지니므로, 입력한 글자가 영문, 한글 등에 따라서 내부적으로 크기가 달라진다.
- 사용자 입장에서는 CHAR(100)은 영문, 한글 구분 없이 100글자를 입력하는 것으로 알고 있으면 된다.

### BLOB(Binary Large OBject)
- 사진 파일, 동영상 파일, 문서 파일 등의 대용량의 이진 데이터를 저장하는데 사

### LONGTEXT, LONGBLOB
- LOB(Large Object)를 저장하기 위한 데이터 형식으로, 약 4GB 크기의 파일을 하나의 데이터로 저장할 수 있다.
  - 장편소설과 같은 큰 텍스트 파일이라면, 그 내용을 전부 LONGTEXT 형식으로 지정된 하나의 컬럼에 넣을 수 있다.
  - 동영상 파일과 같은 큰 바이너리 파일이라면 그 내용을 전부 LONGBLOB 형식으로 지정된 하나의 컬럼에 넣을 수 있다.

### 변수의 사용
- @변수명 : 전역 변수처럼 사용
- DECLARE 변수명 : 스토어드 프로시저나 함수 안에서 지역 변수처럼 사용
- LIMIT에는 원칙적으로 변수를 사용할 수 없으니 PREPARE과 EXECUTE문을 활용해서 변수의 활용도 가능하다.
  - SET @myVar1 = 3;
  - PREPARE myQuery
    - FROM 'SELECT Name, height FROM usertbl ORDER BY height LIMIT ?';
  - EXECUTE myQuery USING @myVar1 ;

### MySQL 내장함수
- IF(수식, 참, 거짓) : 단순한 분기처리 시 사용. CASE문과 역할은 유사하네
- CHAR_LENGTH(문자열) : 문자의 개수 반환
- CONCAT_WS(구분자, 문자열1, 문자열2, ...)

### join
- join과 where절을 같이 쓰면, where절로 필터링을 먼저 하는지 실험해봐야 할 것 같다. 대용량데이터에서 join 성능 확인해보기
- union
