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
redis.king.svc.cluster.local has address 10.10.27.81
```

