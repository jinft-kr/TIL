### dig 명령어
- 도메일으로 정보를 조회하는 명령어
- `dig [옵션] 도메인`
- 옵션 : -t : 질의 타입을 지정
  - A, MX, NS (기본값 : A)
```
$ dig www.naver.com                  

; <<>> DiG 9.10.6 <<>> www.naver.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45413
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.naver.com.                 IN      A

;; ANSWER SECTION:
www.naver.com.          21124   IN      CNAME   www.naver.com.nheos.com.
www.naver.com.nheos.com. 165    IN      A       223.130.200.107
www.naver.com.nheos.com. 165    IN      A       223.130.200.104

;; Query time: 4 msec
;; SERVER: 221.139.13.130#53(221.139.13.130)
;; WHEN: Tue Oct 18 19:56:01 JST 2022
;; MSG SIZE  rcvd: 108
```

### Host 명령어
- `host [옵션] [도메인 or IP주소] [DNS서버]`
- 옵션
  - -a : 도메인의 정보를 타입값(A, MX, NS) 위주로 자세히 출력
  - -t : 질의 타입을 지정합니다. A, MX, NS 를 지정합니다. 기본값은 A 입니다.
  - -v : 도메인에 관한 자세한 정보를 출력합니다.
```
$ host naver.com             
naver.com has address 223.130.200.107
naver.com has address 223.130.195.200
naver.com has address 223.130.195.95
naver.com has address 223.130.200.104
naver.com mail is handled by 10 mx2.naver.com.
naver.com mail is handled by 10 mx1.naver.com.
naver.com mail is handled by 10 mx3.naver.com.
```

### arp
- ARP Cache 를 관리하는 명령
### ethtool
- 네트워크 인터페이스 설정 정보 출력 및 관리 명령
### hostname
- 시스템에 설정된 호스트네임을 출력하는 명령
### nslookup
- 도메인 네임 서버를 이용해 해당 도메인의 정보를 조회하는 명령어
### ping
- ICMP(인터넷 제어 메세지 프로토콜) 을 사용하여 해당네트워크와의 연결을 확인하는 명령
### traceroute
- 패킷이 해당 호스트까지 가는 과정을 출력합니다. 가는 길에 장애를 발견하기 좋은 명령입니다.