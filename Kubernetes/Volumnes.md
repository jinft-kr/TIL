### Volumes - emptyDir, hostPath, PV/PVC
emptyDir, hostPath, PVC/PV 에 대해 알아보자.

### emptyDir
- 컨테이너들끼리 데이터를 공유하기 위해서 볼륨을 사용한다.
- 최초 볼륨이 생성될 때 볼륨 내용이 비어있기 때문에 emptyDir로 불리게 되었다.
- Container1이 web이고 Contaner2가 back-end라 정의했을 때,
  - Container1은 웹서버로 받은 특정 파일을 마운트된 Volume에 저장한다.
  - Container2도 Volume을 이용하여 두 서버가 파일을 주고받을 필요 없이 편하게 사용 가능하다.
  - 이 볼륨은 파드안에 생성되기 때문에 **파드에 문제가 생겨 재생성되면 데이터가 없어진다**
- 따라서 볼륨에 쓰인 데이터는 꼭 일시적인 사용목적을 가져야 한다.
- Sample YAML
  - ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-volume-1
    spec:
      containers:
      - name: container1
        image: kubetm/init
        volumeMounts:
        - name: empty-dir
          mountPath: /mount1
      - name: container2
        image: kubetm/init
        volumeMounts:
        - name: empty-dir
          mountPath: /mount2
      volumes:
      - name : empty-dir
        emptyDir: {}    
    ```
    - 컨테이너가 2개 있다.
    - 두 컨테이너 모두 같은 볼륨을 마운트하고 있다.
    - 마운트하는 path를 보면 컨테이너1은 mount1, 컨테이너2는 mount2이다.
    - path가 달라도 name이 empty-dir이여서 컨테이너마다 자신이 원하는 경로를 사용할 수 있다.
    - 하지만 Pod안에 생성되어 있는 같은 볼륨 사용한다는 점은 달라지지 않는다.

### hostPath
- 파드들이 올라가진 노드의 path를 볼륨으로써 사용한다.
- emptyDir이랑 다른점은 path를 파드들이 공유하기 때문에 파드가 죽어도 노드의 볼륨은 안 죽는다는 점이다.
- 문제점: 파드2가 죽어서 노드1이 아닌 노드2에 파드2가 재생성되면 노드1의 볼륨 이용 불가능하다.

- 노드2가 생길 때, 똑같은 경로를 만들어서 노드에 있는 path끼리 마운트시켜주면 해결가능하지만, 이것은 쿠버네티스가 해주진 않는다.
- 운영자가 리눅스 시스템의 마운트 기술을 사용하는 것이다. 따라서 자동화가 아니니 안정적이지 않았다. 권장하지 않는다.
- 정리하자면 hostPath는 파드 자신이 할당되어 있는 노드의 데이터를 읽거나 쓸 때 사용한다.
- Sample YAML
  - ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-volume-3
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node1
      containers:
      - name: container
        image: kubetm/init
        volumeMounts:
        - name: host-path
          mountPath: /mount1
      volumes:
      - name : host-path
        hostPath:
          path: /node-v
          type: DirectoryOrCreate
    ```
    - 파드를 만들 때 컨테이너에서 볼륨을 마운트하는데 mountPath는 /mount1 을 마운트한다.
    - path에 대한 host-path의 볼륨은 /node-v에 있다.
    - host-path의 타입은 Directory 이다.
    - host-path의 path(/node-v)는 파드가 생성되기 전에 있어야 오류가 나지 않는다.
    - hostPath는 파드의 데이터를 저장하는 용도가 아니라 노드의 데이터를 파드에서 쓰기 위해 사용한다.

### PVC/PV
- Persistent Volume Claim, Persistent Volume
- PVC/PV는 파드에 영속성 있는 볼륨을 제공한다.
- 볼륨은 Local도 있고 외부에 원격으로 활용하는 것도 있다.
- 이런 것들을 각각 PV를 정의하여 연결한다.
- 파드는 PV에 바로 연결하지 않고 PVC를 통하여 연결된다.
- 파드를 PV에 바로 연결하지 않는 이유는 쿠버네티스가 User와 Admin으로 영역을 나눠서 관리하기 때문이다.
- Admin : 쿠버네티스 운영자, User : 파드에 서비스를 만들고 배포를 담당하는 자
- Sample YAML
  - PV(원격 스토리지)
    - ```
      apiVersion: v1
      kind: PersistentVolume
      metadata:
          name: pv-01
      spec:
          nfs:
              server: 192.168.0.xxx
              path: /sda/data
          iscsi:
              targetPortal: 163.180.11
              iqn: iqn.200.qnap:...
              lun: 0
              fsType: ext4
              readOnly: no
              chapAuthSession: true
          gitRepo:
              repository: github.com...
              revision: master
          directory:
          ...
      ```
        - 각각의 볼륨에 따라 속성들이 다르다.
  - PVC
    - ```
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        name: pvc-01
      spec:
        accessModes:
            - ReadWriteOnce
        resources:
            requests:
                storage: 1G
        storageClassName: ""
      ```
      - 읽기/쓰기 모드가 되고 용량이 1G인 볼륨을 연결하라고 정의했다.
      - storageClassName: ""인데 이는 현재 만들어진 PV 중에서 선택되는 걸 의미한다.
  - Pod
    - ```
      apiVersion: v1
      kind: Pod
      metadata:
        name: pod-volume-3
      spec:
        containers:
        - name: container
          image: tmkube/init
          volumeMounts:
          - name: pvc-pv
            mountPath: /volume
        volumes:
        - name: pvc-pv
          persistentVolumeClaim:
            claimName: pvc-01
      ```
      - claimName에 pvc-01을 연결한다.
      - 볼륨을 만들어 놓으면, 볼륨을 컨테이너에서 사용한다.

### Summary
1. Admin이 PV를 만든다.
2. User가 PVC를 만든다.
3. k8s가 PVC에 맞는 PV를 연결해준다.
4. Pod를 만들 때, PVC를 사용한다.