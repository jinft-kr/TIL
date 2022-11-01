### Modify istio meshconfig access log format

### 목적
인프라 운영시 Envoy Access Log를 확인할 일이 많다.
[Default Access Log Format](https://istio.io/latest/docs/tasks/observability/logs/access-log/) 은 다음과 같다.
```
[%START_TIME%] \"%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%\" %RESPONSE_CODE% %RESPONSE_FLAGS% %RESPONSE_CODE_DETAILS% %CONNECTION_TERMINATION_DETAILS%
\"%UPSTREAM_TRANSPORT_FAILURE_REASON%\" %BYTES_RECEIVED% %BYTES_SENT% %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% \"%REQ(X-FORWARDED-FOR)%\" \"%REQ(USER-AGENT)%\" \"%REQ(X-REQUEST-ID)%\"
\"%REQ(:AUTHORITY)%\" \"%UPSTREAM_HOST%\" %UPSTREAM_CLUSTER% %UPSTREAM_LOCAL_ADDRESS% %DOWNSTREAM_LOCAL_ADDRESS% %DOWNSTREAM_REMOTE_ADDRESS% %REQUESTED_SERVER_NAME% %ROUTE_NAME%\n
```
<div align="center">

|                  Log operator                   |          access log in sleep           |   access log in httpbin    | 
|:--------------------------------------:|:--------------------------------------:|:--------------------------:|
|     [%START_TIME%]    |       [2020-11-25T21:26:18.409Z]       | [2020-11-25T21:26:18.409Z] |
| \"%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%\" |       "GET /status/418 HTTP/1.1"       | "GET /status/418 HTTP/1.1" |
|             %RESPONSE_CODE%               |                  418                   |            418             |
|%RESPONSE_FLAGS%|                   -                    |             -              |
|%RESPONSE_CODE_DETAILS%|              via_upstream              |        via_upstream        |
|%CONNECTION_TERMINATION_DETAILS%|                   -                    |             -              |
|\"%UPSTREAM_TRANSPORT_FAILURE_REASON%\"|                  "-"                   |            "-"             |
|%BYTES_RECEIVED%|                   0                    |             0              |
|%BYTES_SENT%|                  135                   |            135             |
|%DURATION%|                   4                    |             3              |
|%RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)%|                   4                    |             1              |
|\"%REQ(X-FORWARDED-FOR)%\"|                  "-"                   |"-"|
|\"%REQ(USER-AGENT)%\"|           "curl/7.73.0-DEV"            |"curl/7.73.0-DEV"|
|\"%REQ(X-REQUEST-ID)%\"| "84961386-6d84-929d-98bd-c5aee93b5c88" |"84961386-6d84-929d-98bd-c5aee93b5c88"|
|\"%REQ(:AUTHORITY)%\"|             "httpbin:8000"             |"httpbin:8000"|
|\"%UPSTREAM_HOST%\"|            "10.44.1.27:80"             |"127.0.0.1:80"|
|%UPSTREAM_CLUSTER%|||
|%UPSTREAM_LOCAL_ADDRESS%|            10.44.1.23:37652            |127.0.0.1:41854|
|%DOWNSTREAM_LOCAL_ADDRESS%|            10.0.45.184:8000            |10.44.1.27:80|
|%DOWNSTREAM_REMOTE_ADDRESS%|            10.44.1.23:46520            |10.44.1.23:37652|
|%REQUESTED_SERVER_NAME%|                   -                    |outbound_.8000_._.httpbin.foo.svc.cluster.local|
|%ROUTE_NAME%|                default                 |default|
</div>

Default Access Log Format으로 설정했을 때 Access Log를 살펴보면 다음과 같이 로그가 나온다.
```
[2022-11-01T04:55:11.402Z] "GET /statics/icon-top-03.247b9912.png HTTP/2" 304 - via_upstream - "-" 0 0 3 2 "211.244.65.10" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36" "0172f8d8-d6fd-4d21-b816-631c4559bfd6" "{REQ(:AUTHORITY)}" "10.10.37.35:8080" outbound|8080||{pod}.{namespace}.svc.cluster.local 10.10.34.2:34006 10.10.34.2:443 211.244.65.10:57918 {REQUESTED_SERVER_NAME} -
```
기본 설정 값으로 Access Log를 확인할 때 보여주는 값들이 어떤 걸 의미하는지 알기가 어렵고, 해당 값들의 의미를 파악하는데 시간이 오래걸려서 Access Log Format을 변경할 필요성을 느꼈다.

### 요구사항
1. JSON 형태로 Access Log 출력
2. Access Log에서 보여주는 각 값들이 어떤 값을 의미하는지 출력

### Modify istio meshconfig access log format
1. Access Log를 한줄로 보고싶을 때 설정
```
spec:
  meshConfig:
    accessLogFile: /dev/stdout
    accessLogEncoding: TEXT
    accessLogFormat: |
      { "start_time" : "%START_TIME%", "method": "%REQ(:METHOD)%", "path": "%REQ(X-ENVOY-ORIGINAL-PATH?:PATH)%", "protocol": "%PROTOCOL%", "response_code": "%RESPONSE_CODE%", "response_flags": "%RESPONSE_FLAGS%", "response_code_details": "%RESPONSE_CODE_DETAILS%", "upstream_transport_failure_reason": "%UPSTREAM_TRANSPORT_FAILURE_REASON%", "bytes_received": "%BYTES_RECEIVED%", "bytes_sent": "%BYTES_SENT%", "duration": "%DURATION%", "upstream_service_time": "%RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)%", "x_forwarded_for": "%REQ(X-FORWARDED-FOR)%", "user_agent": "%REQ(USER-AGENT)%", "request_id": "%REQ(X-REQUEST-ID)%", "authority": "%REQ(:AUTHORITY)%", "upstream_host": "%UPSTREAM_HOST%", "upstream_cluster": "%UPSTREAM_CLUSTER%", "upstream_local_address": "%UPSTREAM_LOCAL_ADDRESS%", "downstream_local_address": "%DOWNSTREAM_LOCAL_ADDRESS%", "downstream_remote_address": "%DOWNSTREAM_REMOTE_ADDRESS%", "requested_server_name": "%REQUESTED_SERVER_NAME%", "route_name": "%ROUTE_NAME%" }
```

2. Access Log를 Json 형태로 보고 싶을 때 설정
```
spec:
  meshConfig:
    accessLogFile: /dev/stdout
    accessLogEncoding: TEXT
    accessLogFormat: |
      {
        "start_time" : "%START_TIME%",
        "method": "%REQ(:METHOD)%",
        "path": "%REQ(X-ENVOY-ORIGINAL-PATH?:PATH)%",
        "protocol": "%PROTOCOL%",
        "response_code": "%RESPONSE_CODE%",
        "response_flags": "%RESPONSE_FLAGS%",
        "response_code_details": "%RESPONSE_CODE_DETAILS%",
        "upstream_transport_failure_reason": "%UPSTREAM_TRANSPORT_FAILURE_REASON%",
        "bytes_received": "%BYTES_RECEIVED%",
        "bytes_sent": "%BYTES_SENT%",
        "duration": "%DURATION%",
        "upstream_service_time": "%RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)%",
        "x_forwarded_for": "%REQ(X-FORWARDED-FOR)%",    
        "user_agent": "%REQ(USER-AGENT)%",
        "request_id": "%REQ(X-REQUEST-ID)%",
        "authority": "%REQ(:AUTHORITY)%",
        "upstream_host": "%UPSTREAM_HOST%",
        "upstream_cluster": "%UPSTREAM_CLUSTER%",
        "upstream_local_address": "%UPSTREAM_LOCAL_ADDRESS%",
        "downstream_local_address": "%DOWNSTREAM_LOCAL_ADDRESS%",
        "downstream_remote_address": "%DOWNSTREAM_REMOTE_ADDRESS%",
        "requested_server_name": "%REQUESTED_SERVER_NAME%",
        "route_name": "%ROUTE_NAME%"
      }
```
### `accessLogEncoding : JSON` 이 아닌 `accessLogEncoding : TEXT`로 설정한 이유는?
`accessLogEncoding : JSON`으로 설정하게 될 경우, `àccessLogFormat` 에 정의한 순서대로 속성들이 출력되지 않는다.
다음과 같이 랜덤한 순서로 AccessLog 속성들이 출력되는 현상을 볼 수 있다.
```
{"upstream_local_address":"10.10.37.42:41592","upstream_transport_failure_reason":null,"method":null,"route_name":null,"bytes_sent":73232,"protocol":null,"request_id":null,"upstream_cluster":"PassthroughCluster","duration":665017,"response_flags":"-","authority":null,"upstream_host":"34.64.144.150:27017","downstream_local_address":"34.64.144.150:27017","path":null,"response_code_details":null,"client_ip":null,"x_forwarded_for":null,"user_agent":null,"start_time":"2022-11-01T05:48:17.088Z","upstream_service_time":null,"downstream_remote_address":"10.10.37.42:41576","bytes_received":1095,"requested_server_name":null,"response_code":0}
{"client_ip":null,"protocol":null,"upstream_transport_failure_reason":null,"user_agent":null,"request_id":null,"x_forwarded_for":null,"upstream_cluster":"PassthroughCluster","response_code":0,"method":null,"downstream_remote_address":"10.10.37.42:40048","upstream_local_address":"10.10.37.42:40156","requested_server_name":null,"upstream_host":"34.64.97.34:3306","duration":678523,"path":null,"bytes_sent":28092,"response_flags":"-","bytes_received":21218,"authority":null,"start_time":"2022-11-01T05:48:03.582Z","upstream_service_time":null,"downstream_local_address":"34.64.97.34:3306","route_name":null,"response_code_details":null}
{"duration":666011,"downstream_local_address":"34.64.239.110:27017","upstream_cluster":"PassthroughCluster","upstream_transport_failure_reason":null,"method":null,"downstream_remote_address":"10.10.37.42:33534","start_time":"2022-11-01T05:48:16.094Z","client_ip":null,"user_agent":null,"path":null,"route_name":null,"upstream_service_time":null,"bytes_received":1095,"bytes_sent":73232,"protocol":null,"response_flags":"-","upstream_local_address":"10.10.37.42:33538","requested_server_name":null,"request_id":null,"upstream_host":"34.64.239.110:27017","response_code":0,"response_code_details":null,"authority":null,"x_forwarded_for":null}
{"response_code_details":null,"upstream_cluster":"PassthroughCluster","duration":678523,"upstream_local_address":"10.10.37.42:40140","response_code":0,"upstream_transport_failure_reason":null,"path":null,"user_agent":null,"client_ip":null,"response_flags":"-","protocol":null,"method":null,"requested_server_name":null,"bytes_sent":27216,"upstream_host":"34.64.97.34:3306","downstream_remote_address":"10.10.37.42:40020","start_time":"2022-11-01T05:48:03.582Z","authority":null,"x_forwarded_for":null,"bytes_received":19943,"route_name":null,"request_id":null,"upstream_service_time":null,"downstream_local_address":"34.64.97.34:3306"}
{"protocol":null,"request_id":null,"downstream_remote_address":"10.10.37.42:52220","upstream_transport_failure_reason":null,"upstream_local_address":"10.10.37.42:52232","authority":null,"client_ip":null,"path":null,"upstream_cluster":"PassthroughCluster","method":null,"x_forwarded_for":null,"downstream_local_address":"34.64.126.18:3306","response_code":0,"upstream_host":"34.64.126.18:3306","user_agent":null,"duration":672724,"requested_server_name":null,"start_time":"2022-11-01T05:48:09.381Z","upstream_service_time":null,"response_flags":"-","response_code_details":null,"bytes_sent":23073,"bytes_received":65017,"route_name":null}
{"authority":null,"duration":678523,"x_forwarded_for":null,"start_time":"2022-11-01T05:48:03.582Z","client_ip":null,"bytes_received":20131,"response_code":0,"upstream_cluster":"PassthroughCluster","path":null,"upstream_transport_failure_reason":null,"request_id":null,"requested_server_name":null,"upstream_host":"34.64.97.34:3306","downstream_local_address":"34.64.97.34:3306","upstream_local_address":"10.10.37.42:40158","route_name":null,"protocol":null,"user_agent":null,"response_code_details":null,"method":null,"downstream_remote_address":"10.10.37.42:40090","bytes_sent":27780,"response_flags":"-","upstream_service_time":null}
```
따라서 `accessLogEncoding : TEXT`로 설정하여, `àccessLogFormat` 에 정의한 순서대로 속성들이 출력되도록 강제하였다.


### Result
1. Access Log를 한줄로 출력했을 때
```
{ "start_time" : "2022-11-01T06:26:23.973Z", "protocol": "HTTP/1.1", "upstream_service_time": "-", "upstream_local_address": "127.0.0.6:52429", "duration": "0", "upstream_transport_failure_reason": "-", "route_name": "default", "downstream_local_address": "10.10.37.35:8080", "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36", "response_code": "304", "response_flags": "-", "method": "GET", "request_id": "4b7b3304-d531-417b-b4e3-b18f3e8328ff", "upstream_host": "10.10.37.35:8080", "x_forwarded_for": "211.244.65.10", "client_ip": "-", "requested_server_name": "outbound_.8080_._.{pod_name}.{namespace}.svc.cluster.local", "bytes_received": "0", "bytes_sent": "0", "upstream_cluster": "inbound|8080||", "downstream_remote_address": "211.244.65.10:0", "authority": "{authority}", "path": "/statics/icon-shadow-03.8e9c469e.png", "response_code_details": "via_upstream" }
{ "start_time" : "2022-11-01T06:26:23.973Z", "protocol": "HTTP/1.1", "upstream_service_time": "-", "upstream_local_address": "127.0.0.6:45213", "duration": "0", "upstream_transport_failure_reason": "-", "route_name": "default", "downstream_local_address": "10.10.37.35:8080", "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36", "response_code": "304", "response_flags": "-", "method": "GET", "request_id": "eb26b5be-89b1-4162-8d3e-cdcd5c5d6281", "upstream_host": "10.10.37.35:8080", "x_forwarded_for": "211.244.65.10", "client_ip": "-", "requested_server_name": "outbound_.8080_._.{pod_name}.{namespace}.svc.cluster.local", "bytes_received": "0", "bytes_sent": "0", "upstream_cluster": "inbound|8080||", "downstream_remote_address": "211.244.65.10:0", "authority": "{authority}", "path": "/statics/icon-top-05.8314ea16.png", "response_code_details": "via_upstream" }
```
2. Access Log를 JSON 형태로 출력했을 때
```
{
  "start_time" : "2022-11-01T05:59:34.247Z",
  "protocol": "HTTP/1.1",
  "upstream_service_time": "-",
  "upstream_local_address": "127.0.0.6:50867",
  "duration": "1031",
  "upstream_transport_failure_reason": "-",
  "route_name": "default",
  "downstream_local_address": "10.10.37.42:3000",
  "user_agent": "%EB%B0%9C%EC%A0%84%EC%99%95%20dev/140 CFNetwork/1390 Darwin/22.0.0",
  "response_code": "200",
  "response_flags": "-",
  "method": "POST",
  "request_id": "d66661c8-3423-4df6-87b3-6b27cfd50943",
  "upstream_host": "10.10.37.42:3000",
  "x_forwarded_for": "211.244.65.10",
  "client_ip": "-",
  "requested_server_name": "outbound_.3000_._.{pod_name}.{namespace}.svc.cluster.local",
  "bytes_received": "932",
  "bytes_sent": "717",
  "upstream_cluster": "inbound|3000||",
  "downstream_remote_address": "211.244.65.10:0",
  "authority": "{authority}",
  "path": "/graphql",
  "response_code_details": "via_upstream"
}
```

### Reference
- [Istio Access Log](https://istio.io/latest/docs/tasks/observability/logs/access-log/)
- [Access log with JSON encoding](https://github.com/istio/istio/issues/12232)