### Sleep Session

1. Sleep Session 이란?
   - client-mysql 서버와 연결 후 다음 query 수행까지 대기중인 상태의 세션
   - sleep 세션이 너무 많고 정리가 안되는 경우 connection full 로 인해 신규 세션 접속이 불가능해지고
   - session 별 할당 되는 메모리로 인해 memory 부족 현상 발생 가능
   - timeout 설정
     - `connect_timeout`  :  MySQL 서버 접속시에 접속실패를 메시지를 보내기까지 대기하는 시간
     - `delayed_insert_timeout`  :  insert시 delay될 경우 대기하는 시간
     - `innodb_lock_wait_timeout`  :  innodb에 transaction 처리중 lock이 걸렸을 시 롤백 될때까지 대기하는 시간으로 innodb는 자동으로 데드락을 검색해서 롤백시킴
     - `innodb_rollback_on_timeout`  :  innodb의 마지막 구문을 롤백시킬지 결정하는 파라미터, timeout은 진행중인 transaction을 중단하고 전체 transaction을 롤백하는 과정에서 발생
     - `net_read_timeout`  :  서버가 클라이언트로부터 데이터를 읽어들이는 것을 중단하기까지 대기하는 시간
     - `net_write_timeout`  :  서버가 클라이언트에 데이터를 쓰는 것을 중단하기까지 대기하는 시간
     - `slave_net_timeout`  :  마스터/슬레이브로 서버가 클라이언트로부터 데이터를 읽어들이는 것을 중단하기까지 대기하는 시간
     - `table_lock_wait_timeout`  :  테이블 락을 중단하기까지 대기하는 시간
     - `wait_timeout`  :   활동하지 않는 커넥션을 끊을때까지 서버가 대기하는 시간 (php,jdbc 등을 통한 connection)
     - `interactive_timeout`  :  활동중인 커넥션이 닫히기 전까지 서버가 대기하는 시간 (mysql command line)
   - DB의 Timeout 설정으로 sleep 세션들을 정리할 수 있다.

2. MySQL에서 수행 중인 Process를 확인하는 방법
   - `mysql> show processlist;`
   - `SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE COMMAND != 'Sleep';`
3. [processlist 속성](https://dev.mysql.com/doc/refman/8.0/en/show-processlist.html)
   - Id : 프로세스 아이디 MySQL 이 관리하는 스레드 번호. 
   - User : 스레드에 접속하고 있는 MySQL 유저명 
   - Host : 유저가 접속하고 있는 호스트명 , IP 어드레스 
   - Command : 스레드의 현재 커맨드 상태. 
   - Time : 프로세스가 현재 커맨드상태에서 동작 시간 
   - State : 스레드의 상태에 대해 사람이 읽을 수 있는 형태의 정보 
   - Info : 현 실행되고 있는 SQL.
4. 일정한 간격을 가지고 모니터링 하고 싶다면?
   - `mysqladmin -u root -p -i 2 processlist`
5. 실제 일하는 Process만 보고 싶다면?
   - `mysqladmin -u root -p -i 2 processlist | grep -v Sleep;`
6. 특정 세션(프로세스)를  Kill  하고 싶다면 ?
   - `kill 58 ;`
   - `call mysql.rds_kill(58) ;`
   - `mysqladmin -u root -p kill 58`
7. IP로 프로세스 찾아내기
   - `select * from information_schema.processlist where host LIKE '172.28.81.105%';`
8. 특정 IP 프로세스의 KILL 쿼리 string들 뽑아내기
   - `select concat('KILL ',id,';') from information_schema.processlist where host LIKE '172.28.81.105%';`
9. user 로 프로세스 찾아내기
   - `select concat('KILL ',id,';') from information_schema.processlist where user='root';`
10. 만들어진 KILL 쿼리 스트링을 파일로 뽑아내기
    - `select concat('KILL ',id,';') from information_schema.processlist where user='root' into outfile '/tmp/a.txt';`
11. KILL 쿼리들을 불러와서 실행시키기
    - `source /tmp/a.txt;`
