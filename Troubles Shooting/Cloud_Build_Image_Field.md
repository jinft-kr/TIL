### [ Issue ]

- GCP Cloud Build를 이용하여 docker image를 GKE에 배포하였다.
- 배포 과정 중 이상한 현상이 있었다.
- 배포를 하려는 deployment가 ImagePullErr 상태였다가, 2초 정도 후 Image Pull이 정상적으로 된 것이다.
- '왜 이런 현상이 일어나는걸까?' 궁금하였다.
![ImagePullErr](https://user-images.githubusercontent.com/63401132/176442126-01f76617-f96e-423c-8819-f4a2e50260b4.png)
![ImagePullErr_Event](https://user-images.githubusercontent.com/63401132/176442140-475edaa4-5382-4014-ae79-8721ee5eb3b7.png)
![Image_Pull_Success](https://user-images.githubusercontent.com/63401132/176442154-e6972bfe-97b6-4957-a7d9-3709c7b64677.png)

### [ Problem Solution Approach ]
- [Registry에 Container image를 저장하는 방법 두가지](https://cloud.google.com/build/docs/building/build-containers)
  - `image` 필드 : `빌드가 완료된 후` 이미지를 `Artifact Registry` 에 저장
  - `push` 명령어 : `빌드가 진행되는 동안` 이미지를 `Artifact Registry` 에 저장
- 문제를 겪은 `Deployment` 와 연결된 `Cloud Build Script`
```
steps:
  - name: gcr.io/cloud-builders/docker
    args:
    - build
    - '-t'
    - >-
      ${REGION}-docker.pkg.dev/$PROJECT_ID/${ARTIFACT_REGISTRY_NAME}/${ARTIFACT_REGISTRY_NAME}:$SHORT_SHA
    - '--build-arg'
    - 'NODE_ENV=${_NODE_ENV}'
    - .
      id: image-build
  - name: gcr.io/cloud-builders/gcloud
    args:
    - '-c'
    - >
      curl -X POST -H "Authorization: Bearer $(gcloud auth
      print-identity-token
      --impersonate-service-account=${SERVICE_ACCOUNT_NAME}
      --audiences=${SERVER_HOST_NAME}"
      "${KUBERNETES_DEPLOYMENT_CHANGE_NEW_IMAGE_COMMAND}"
      id: deploy
      entrypoint: bash
timeout: 600s
images:
  - >-
    ${REGION}-docker.pkg.dev/$PROJECT_ID/${ARTIFACT_REGISTRY_NAME}/${IMAGE_NAME}:$SHORT_SHA
logsBucket: '${LOG_BUCKET_PATH}'
```

- `Registry`에 `Container image`를 저장할 때 `iamge` field를 이용한 방법을 사용해서, `Cloud Build Script가 완료된 후 Image를 올라가서 생긴 문제`였다.
- `Build & Deploy Process`는 아래와 같았다.
  - Docker Image `Build`
  - k8s Deployment new docker image `deploy`
  - k8s Deployment `ImagePullErr`
  - Docker Image `Push`
  - k8s Deployment `ImagePull`

- 개선한 Cloud Build Script
  - `push` 명령어를 통해 `lifecycle` 을 `build - push - deploy` 순서로 변경하였다.
```
steps:
  - name: gcr.io/cloud-builders/docker
    args:
    - build
    - '-t'
    - >-
      ${REGION}-docker.pkg.dev/$PROJECT_ID/${ARTIFACT_REGISTRY_NAME}/${IMAGE_NAME}:$SHORT_SHA
    - '--build-arg'
    - 'NODE_ENV=${_NODE_ENV}'
    - .
      id: image-build
  - name: gcr.io/cloud-builders/docker
    args:
      - push
      - >-
        ${REGION}-docker.pkg.dev/$PROJECT_ID/${ARTIFACT_REGISTRY_NAME}/${IMAGE_NAME}:$SHORT_SHA
    id: image-push
    waitFor:
      - image-build
  - name: gcr.io/cloud-builders/gcloud
    args:
    - '-c'
    - >
      curl -X POST -H "Authorization: Bearer $(gcloud auth
      print-identity-token
      --impersonate-service-account=${SERVICE_ACCOUNT_NAME}
      --audiences=${SERVER_HOST_NAME}"
      "${KUBERNETES_DEPLOYMENT_CHANGE_NEW_IMAGE_COMMAND}"
      id: deploy
      entrypoint: bash
timeout: 600s
logsBucket: '${LOG_BUCKET_PATH}'
```

---
### [ 왜 기존에는 `image` 필드를 이용해서 Cloud Build Script를 짰을까? ]
- 그건 과거 인프라 히스토리를 통해 추측할 수 있었다.
- 기존에는 [Keel](https://github.com/keel-hq/keel) 이라는 `kubernetes deploy automation 도구`를 썼었다.
- `Registry`에 `새로운 Image`가 Push 되면 `Keel Webhook event`를 통해 `새로운 이미지로 배포` 가 되는 형식이었다.
- 따라서 당시에는 `image` Field를 이용해도 `ImagePullErr` 가 발생하지 않았던 것이다.
- 하지만 최근에 Keel을 제거하기 위해 `kubectl set`명령어로 `직접 deployment image를 변경`하는 방식으로 바뀌었다.
- 따라서 Cloud Build Script에 `kubectl set` 을 이용해 `deployment`의 `새로운 image`로 변경하는 `step`이 추가되었고,
- 기존의 Cloud Build Script를 가져와서 수정하다보니까, `image` 필드를 이용하여 Registry에 저장하는 과정은 빌드 배포에 영향을 주게되었다.
- 사실 현재 인프라 상황에서는 `ImagePullErr`를 발생시키고 후추에 `ImagePull` 하여 진행해도 큰 문제는 없는 상황이다.
- 그래도 `빌드 & 배포 과정`을 깔끔하게 하기 위해서 `Registry에 Image를 저장하는 방식`을 `Cloud Build Script` 안에서 `push` 명령어를 통해 진행하도록 변경하였다.
