### BigQuery란?
- Google에서 드레멜 엔진을 사용해 만든 페타 바이트 규모의 저비용 데이터 웨어하우스이다. 구글에서 관리해주기 때문에 사용자가 별도의 서버나 물리적 하드웨어에 대해 스트레스를 받을 일이 없다.
- 일반적인 rdb나 noSQL보다 속도가 월등히 빠르며, 몇초 안에 TB를 스캔할 수 있다. 또한 Google Cloud Storage에서 데이터를 읽어 분석할 수 있다.
- 데이터 저장 및 컴퓨팅의 개념을 분리해 독립적으로 비용을 지불할 수 있다.
- 웹 UI 혹은 Command Line tool을 사용해 BigQuery를 이용할 수 있으며 Rest API와 클라이언트 라이브러리가 준비되어 있다. ( python, java, c#, go, node.js, php, ruby )
- Firebase 와 연동해서 사용할 경우 정말 좋다. ( 모든 데이터가 누락없이 쌓이는데, 신경쓸 필요가 없습니다! )
- Legacy SQL과 Standard SQL 2가지 방식으로 쿼리를 날릴 수 있다. (세부 문법이 살짝 다르다)
- key, index가 따로 존재하지 않고 Full Scan을 한다는 것이 큰 특징이다.
- Select할 때, Column base로 비용을 부과한다.
- Third Party 도구들과 호환된다. ( Tableau, Google Analytics 360 suite )

### Table Detail
- <img width="799" alt="image (11)" src="https://user-images.githubusercontent.com/63401132/205588974-9ee1abbc-4ad0-4a5e-a830-cbabd4334123.png">
- Schema : 해당 테이블의 Schema(Column 이름, 타입, 설명)
- Details : 해당 테이블과 관련된 정보가 포함(Table ID, Size, Number of Rows, Creation Time, Labels)
- Preview : 해당 테이블에 데이터가 어떻게 적재되어있는지 볼 수 있음
- Refresh : 새로고침
- Query Table : 클릭하면 테이블에 쿼리를 날릴 수 있는 기본구조가 구성
- Copy Table : 테이블을 Copy
- Export Table : 테이블을 Export. CSV, Json, Avro로 가능하며 GZIP으로 압축할 수 있음 (저장하는 공간은 Google Cloud Storage)
- Delete Table : 테이블 삭제

### Query Result
- <img width="1153" alt="image (12)" src="https://user-images.githubusercontent.com/63401132/205589780-5c9a9117-c2c5-4e0d-9237-df41a73b6f7c.png">
- Results : 쿼리의 결과가 나타나는 공간
- Explanation : 쿼리의 단계별 연산에 대한 설명
- Job Information : 쿼리 Job에 대한 정보가 기록
- Save as Table : 지정하는 Table로 쿼리의 결과를 저장
- Save to Google Sheets : 구글 스프레드시트로 결과를 저장. 데이터 요청이 있을 경우, 스프레드시트로 공유하면 유용하다.
- 
### Keyboard shortcuts
<div align="center">

|                  Windows/Linux                   |  Mac	Action  |     Mac	Action  |
|:--------------------------------------:|:------------:|:---------------:|
|Ctrl + Enter| Cmd + Enter	 |  현재 Query를 실행   |
|Tab	|     Tab      | 자동 완성기능  |
|Ctrl|    Cmd    |Table 이름을 Highlight|
|Ctrl + /|    Cmd + /    |현재 줄 주석처리|
|Ctrl + Shift + F|   Cmd + Shift + F   |Query Format 맞추기|

</div>

### How to Duplicate a Table in BigQuery
1. In BigQuery you can duplicate a table, but you need to use a different table name. To duplicate the schema and content of a table just do:
   ```
   create table project_id.dataset_id.table_id_new 
             as select * from project_id.dataset_id.table_id;
   ```
2. If you want to create a new table replicating the schema, but leaving the table empty, you can do:
   ```
   create table project_id.dataset_id.table_id_new
             as select * from project_id.dataset_id.table_id limit 0;
   ```
   
### How to Delete all table data
```
DELETE  FROM `project_id.dataset_id.table_id` WHERE true;
```