### Ubuntu 컨테이너 띄우고 컨테이너 접속하기
```
kubectl run ubuntu --image=ubuntu -it -- bash
```

### 필요한 apt 설치
```
apt-get update -y \
&& apt-get install host curl -y \ # host, curl
&& apt-get install iputils-ping -y \ # ping
&& apt-get install net-tools -y  
```

### host ip 조회
```
# Kubernetes의 다른 네임스페이스에 있는 svc 조회
host redis.king.svc.cluster.local 
redis.king.svc.cluster.local has address 11
```

### k8s pod간의 ping 통신
```
root@ubuntu:/# ping 10.10.33.168 -c 3
PING 10.10.33.168 (10.10.33.168) 56(84) bytes of data.
64 bytes from 10.10.33.168: icmp_seq=1 ttl=63 time=0.078 ms
64 bytes from 10.10.33.168: icmp_seq=2 ttl=63 time=0.072 ms
64 bytes from 10.10.33.168: icmp_seq=3 ttl=63 time=0.051 ms

--- 10.10.33.168 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2083ms
rtt min/avg/max/mdev = 0.051/0.067/0.078/0.011 ms
```

### k8s svc ping 통신
```
root@ubuntu:/# host redis
redis.king.svc.cluster.local has address 10.10.27.81
root@ubuntu:/# ping redis
PING redis.king.svc.cluster.local (10.10.27.81) 56(84) bytes of data.
^C
--- redis.king.svc.cluster.local ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4098ms
```
- 서비스에 curl은 동작하는데, 왜 ping은 안될까?
  - 서비스의 cluster IP가 가상 IP이기 때문이다

