# Docker Swarm Q&A 정리

본 문서는 Docker Swarm을 처음 접하는 환경에서, 시스템을 테넌트 단위로 운영하기 위해 등장한 질문과 그에 대한 답변을 정리한 문서입니다.

---

## 1. Docker Swarm이란 무엇인가?

Docker Swarm은 Docker에 기본 내장된 **컨테이너 오케스트레이션 기능**으로, 여러 대의 서버를 하나의 클러스터처럼 묶어 **서비스(Service)** 단위로 컨테이너를 배포·관리

- 여러 서버를 하나의 논리적 Docker로 관리
- 컨테이너를 직접 실행하지 않고 Service로 선언
- 장애 시 자동 재시작
- 서버 추가 시 자동 확장 가능

---

## 2. Docker Swarm의 기본 구성 요소

### Node
- **Manager**: 클러스터 상태 관리, 스케줄링, 서비스 제어
- **Worker**: 실제 컨테이너(Task)를 실행

### Service
- 실행할 이미지, 개수, 배치 정책을 정의하는 관리 단위

### Task
- Service에 의해 실제로 실행되는 컨테이너 인스턴스

---

## 3. Manager 서버는 GPU가 유무

### Manager의 역할

- 스케줄링
- 노드 상태 관리
- 서비스 배치 결정

-> 실제 연산(추론, GPU 사용)은 불필요

### 실제 활용

- Manager: GPU 없음 (안정성 위주)
- Worker: GPU 있음 (실제 연산 담당)


---

## 4. Worker 서버 별 GPU/환경에서 같은 Image 사용 가능 유무

### Docker Swarm의 기본 전제

- GPU 종류/개수 달라도 무관
- CUDA/드라이버 버전은 호환 범위 내에서 문제 없음
- 동일 Image를 서로 다른 하드웨어에서 실행 가능

### 실제 활용

- 최소 공통 CUDA 기준으로 Image 빌드
- Node Label + Constraint로 배치 제어

---

## 5. Docker Swarm에서 복수의 Worker 서버 설정

**명령 실행을 통해 설정 적용**

1. `docker swarm init` → 클러스터 생성 (Manager 지정)
2. `docker swarm join` → 다른 서버를 Worker로 추가
3. `docker service create` → 서비스 배포 및 자동 배치

※ 서버 수 = Swarm에 join된 노드 수

---

## 6. 새로운 Tenant(Service) 추가 시 기존 서비스는 중단 여부

**중단없이 사용 가능**

- 신규 Tenant = 신규 Service 생성
- 기존 Service에는 영향 없음
- Swarm은 additive 방식으로만 동작

### 주의 사항

- 포트 충돌
- 자원(GPU/메모리) 포화
- `docker service update` 실행 시

---

## 7. 새로운 Worker 서버 추가 시 기존 서비스는 중단 여부

**중단없이 사용 가능**

- Worker가 Swarm에 join되면 즉시 클러스터 확장
- `--mode global` 서비스는 새 Worker에 자동으로 컨테이너 생성
- `replicated` 서비스는 기존 상태 유지

---

## 8. 새 Tenant 추가 시 Swarm의 배치 위치 결정 방식

**Swarm Manager의 스케줄러를 통한 결정**

- 노드 상태(Ready/Drain)
- Placement Constraint
- 포트 사용 여부
- 리소스 예약/제한

`--mode global`인 경우:
- 모든 노드에 1개씩 배치

`replicated`인 경우:
- 조건을 만족하는 노드 중에서 선택

---

## 9. Docker Swarm에서 포트는 노드(Worker 서버)별 같은 번호 사용 가능 여부

### Routing Mesh (기본값)
- 포트는 클러스터 단위로 1개
- 어느 노드로 접속해도 서비스로 전달
- 동일 포트 중복 사용 불가

### Host Mode
- 노드별로 포트 바인딩
- 각 서버에서 동일 포트 사용 가능
- SonicWall/NAT과 연동하기 쉬움

---

## 10. 테넌트별로 특정 서비스만 외부에 노출 가능 여부

### 방법

- 테넌트별 서로 다른 포트 사용
- SonicWall/방화벽에서 포트 단위 허용/차단
- 또는 특정 노드(ingress 노드)에만 배치

Swarm 자체에서도:
- 포트 publish 여부
- node label + constraint

으로 외부 노출 범위를 제어

---

## 11. 실무에서 자주 추가되는 설정

**필수는 아니지만, 거의 항상 사용**

- Node Label (GPU 종류, 역할 구분)
- Placement Constraint
- Resource limit (CPU/Memory)
- Secret 관리 (스토리지 Key, API Key)
- Overlay Network 분리

---

## 12. 핵심 요약

- Docker Swarm은 컨테이너를 **Service 단위**로 관리한다
- Manager는 GPU가 없어도 된다
- Worker 환경이 달라도 같은 Image 사용 가능
- Tenant/Worker 추가 시 기존 서비스는 중단되지 않는다
- 포트/노드/서비스 조합으로 테넌트별 외부 노출 제어가 가능하다

