하나의 앱을 이미지로 만들 때 사용

Dockerfile과 docker-compose.yml의 차이점

| 구분         | Dockerfile        | docker-compose.yml             |
| ---------- | ----------------- | ------------------------------ |
| **대상**     | **1개 컨테이너**       | **여러 컨테이너**                    |
| **필수 여부**  | 이미지 만들 때 **필수**   | 여러 컨테이너 쓸 때 **선택**             |
| **이미지 빌드** | O (`build`)       | O (하지만 `image:`로 공식 이미지 사용 가능) |
| **네트워크**   | 수동 설정             | 자동 생성                          |
| **볼륨**     | 수동 `-v`           | 자동 `volumes:`                  |
| **의존성**    | 수동 `--depends-on` | `depends_on:` 자동 관리            |
| **환경 변수**  | `ENV`             | `environment:`                 |
| **개발/운영**  | 이미지 자체            | 서비스 구성                         |

Dockerfile을 사용할 때의 이점

| 역할                    | 설명                                                                    | 왜 중요한가?                   |
| --------------------- | --------------------------------------------------------------------- | ------------------------- |
| **1. 컨테이너 이미지 정의**    | `Dockerfile`은 **텍스트 파일**이며, **어떤 이미지**를 만들 것인지 **단계별 명령어**로 기술합니다.    | 동일한 환경을 언제 어디서나 **재현** 가능 |
| **2. 빌드 자동화**         | `docker build` 명령으로 Dockerfile을 읽어 **레이어(layer) 별로 이미지**를 만든다.        | 수동 설정을 없애 **오류 감소**       |
| **3. 환경 일관성 보장**      | 베이스 OS, 패키지, 코드, 설정을 모두 명시 → **개발·테스트·운영**이 동일                        | “내 로컬에서는 되는데…” 문제 해결      |
| **4. 버전 관리**          | Dockerfile을 **Git** 등에 커밋하면 **이미지 히스토리**를 추적 가능                       | 배포·롤백·감사 용이               |
| **5. 멀티 스테이지 빌드**     | 빌드 도구는 최종 이미지에 포함 안 함 → **이미지 크기 최소화**                                | 보안·배포 속도 향상               |
| **6. CI/CD 파이프라인 통합** | Jenkins, GitHub Actions, GitLab CI 등에서 `docker build`만 실행하면 **자동 배포** | DevOps 자동화 핵심             |
|                       |                                                                       |                           |
Dockerfile기본 작성법

```
# 1. 베이스 이미지
FROM node:20-alpine

# 2. 작업 디렉터리 설정
WORKDIR /app

# 3. 의존성 파일 복사
COPY package*.json ./

# 4. 의존성 설치
RUN npm ci --only=production

# 5. 소스 코드 복사
COPY . .

# 6. 포트 노출
EXPOSE 3000

# 7. 컨테이너 실행 명령
CMD ["node", "server.js"]
```

Dockerfile작성 명령어

| 명령어          | 설명                              | 예시                                              |
| ------------ | ------------------------------- | ----------------------------------------------- |
| `FROM`       | 베이스 이미지 지정 (Docker Hub 등에서 가져옴) | `FROM ubuntu:22.04`                             |
| `WORKDIR`    | 이후 명령어가 실행될 디렉터리 설정             | `WORKDIR /app`                                  |
| `COPY`       | 호스트 → 컨테이너 파일 복사                | `COPY . .`                                      |
| `ADD`        | `COPY` + URL/압축 해제 지원           | `ADD http://example.com/file.tar.gz /tmp/`      |
| `RUN`        | 이미지 빌드 시 실행할 쉘 명령               | `RUN apt-get update && apt-get install -y curl` |
| `CMD`        | 컨테이너 실행 시 기본 명령 (한 번만)          | `CMD ["npm", "start"]`                          |
| `ENTRYPOINT` | 고정된 실행 명령 (옵션은 `CMD`로)          | `ENTRYPOINT ["node"]`                           |
| `EXPOSE`     | 컨테이너가 listen할 포트 (문서용)          | `EXPOSE 8080`                                   |
| `ENV`        | 환경 변수 설정                        | `ENV NODE_ENV=production`                       |
| `ARG`        | 빌드 시 전달되는 인자                    | `ARG NODE_ENV`                                  |
| `LABEL`      | 메타데이터 추가                        | `LABEL maintainer="me@example.com"`             |
| `USER`       | 이후 명령을 실행할 사용자                  | `USER node`                                     |