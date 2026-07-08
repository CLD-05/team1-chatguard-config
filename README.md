# 관련 레포지토리

| 레포지토리 | 설명 |
|---|---|
| [team1-chatguard-app](https://github.com/CLD-05/team1-chatguard-app) | React 프론트엔드, Spring Boot 백엔드, Python AI 모더레이션 워커 및 CI/CD 파이프라인을 포함하는 메인 저장소 |
| [team1-chatguard-config](https://github.com/CLD-05/team1-chatguard-config) | Kubernetes Kustomize 매니페스트, ArgoCD Application, KEDA ScaledObject 등 GitOps 배포 구성을 관리하는 저장소 |
| [team1-chatguard-infra](https://github.com/CLD-05/team1-chatguard-infra) | Terraform을 이용하여 VPC, Subnet, EKS, RDS, ElastiCache, ECR, IAM 등을 프로비저닝하는 인프라 저장소 |
| [team1-chatguard-context](https://github.com/CLD-05/team1-chatguard-context) | 배포되는 코드가 아니라, 사람과 AI(Claude)가 작업의 기준으로 삼는 규칙·설계·작업지시·도구를 모아둡니다. "어떻게 일할지"의 단일 진실 공급원. |

---

# team1-chatguard-config

## 프로젝트 소개

### 이 저장소의 역할

`team1-chatguard-config`는 ChatGuard 프로젝트의 **쿠버네티스 GitOps 배포 구성 저장소**입니다.

애플리케이션 소스 코드와 분리하여 개발(dev) 및 운영(prod) 환경에 배포하기 위한 쿠버네티스 매니페스트를 관리하며, GitOps 방식으로 클러스터의 배포 구성을 선언적으로 유지합니다.

### app 및 infra 저장소와의 관계

| 저장소 | 역할 |
|--------|------|
| team1-chatguard-app | Spring Boot 채팅 서버, Python AI 모더레이션 워커, React 프론트엔드 등 애플리케이션 소스 코드를 관리합니다. |
| team1-chatguard-config | Kubernetes 매니페스트와 GitOps 배포 구성을 관리합니다. *(본 저장소)* |
| team1-chatguard-infra | Terraform을 이용하여 AWS 인프라(VPC, EKS, RDS, ElastiCache, IAM 등)를 코드(IaC)로 관리합니다. |

### GitOps(Kustomize + ArgoCD)를 사용하는 이유

- **환경별 일관성과 재사용성 (Kustomize)**: 공통 매니페스트(Base)를 기준으로 개발(dev) 및 운영(prod) 환경에서 이미지 태그, 복제본 수, 환경 변수 등 필요한 항목만 오버레이(Overlay)하여 매니페스트 중복을 최소화했습니다.
- **선언적 지속 배포 (ArgoCD)**: Git 저장소를 단일 기준(Source of Truth)으로 사용하여 저장소의 변경 사항을 EKS 클러스터에 자동으로 동기화합니다. 또한 클러스터에서 발생한 수동 변경은 자동 복구(Self-Healing)를 통해 원래 상태로 유지합니다.

---

## 관리 대상

이 저장소에서는 ChatGuard 서비스를 Kubernetes 환경에 배포하고 운영하기 위한 리소스를 관리합니다.

| 구분 | 관리 리소스 |
|------|-------------|
| 애플리케이션 | Deployment, Service, Ingress |
| 환경 설정 | ConfigMap, ExternalSecret, ClusterSecretStore |
| 오토스케일링 | KEDA ScaledObject, PodDisruptionBudget |
| 모니터링 | PodMonitor, PrometheusRule |
| 권한 관리 | ServiceAccount(IRSA) |
| GitOps | ArgoCD Application |

---

## 디렉터리 구조

```text
team1-chatguard-config/
├── .github/
│   └── workflows/
│       └── claude-review.yml
├── argocd/
│   ├── dev-app.yaml
│   └── prod-app.yaml
├── base/
│   ├── configurations/
│   │   └── scaledobject-name-reference.yaml
│   ├── grafana/
│   │   └── dashboards/
│   │       └── chatguard-scaleout-dashboard.json
│   ├── chat-server-scaledobject.yaml
│   ├── chat-server.yaml
│   ├── configmap.yaml
│   ├── frontend.yaml
│   ├── ingress.yaml
│   ├── kustomization.yaml
│   ├── pdb.yaml
│   ├── podmonitor.yaml
│   ├── prometheus-rules.yaml
│   ├── scaledobject.yaml
│   ├── serviceaccount.yaml
│   ├── worker-pdb.yaml
│   ├── worker-podmonitor.yaml
│   └── worker.yaml
└── overlays/
    ├── dev/
    │   ├── configurations/
    │   │   └── secret-name-reference.yaml
    │   ├── patches/
    │   │   └── ingress-tags.yaml
    │   ├── clustersecretstore.yaml
    │   ├── externalsecret.yaml
    │   ├── kustomization.yaml
    │   └── prometheus-rules-dev.yaml
    └── prod/
        ├── configurations/
        │   └── secret-name-reference.yaml
        ├── patches/
        │   └── ingress-tags.yaml
        ├── clustersecretstore.yaml
        ├── externalsecret.yaml
        ├── kustomization.yaml
        └── prometheus-rules-prod.yaml
```

| 디렉터리 | 설명 |
|----------|------|
| `.github/workflows` | GitHub Actions 워크플로우를 관리합니다. |
| `argocd` | 환경별 ArgoCD Application 매니페스트를 관리합니다. |
| `base` | 모든 환경에서 공통으로 사용하는 Kubernetes 매니페스트를 관리합니다. |
| `base/configurations` | Kustomize의 Name Reference 등 사용자 정의 설정을 관리합니다. |
| `base/grafana` | Grafana 대시보드 구성 파일을 관리합니다. |
| `overlays` | 환경별(Kustomize Overlay) 배포 구성을 관리합니다. |
| `overlays/dev` | 개발(dev) 환경의 이미지 태그, Secret, 모니터링 설정 등을 관리합니다. |
| `overlays/dev/patches` | 개발 환경에서 사용하는 리소스 패치를 관리합니다. |
| `overlays/prod` | 운영(prod) 환경의 이미지 태그, Secret, 모니터링 설정 등을 관리합니다. |
| `overlays/prod/patches` | 운영 환경에서 사용하는 리소스 패치를 관리합니다. |

---

## 배포 흐름

애플리케이션 변경 사항은 GitHub Actions와 ArgoCD를 이용한 GitOps 방식으로 Amazon EKS에 자동 배포됩니다.

```text
Developer
    │
    ▼
team1-chatguard-app
    │
    ▼
GitHub Actions
    ├──────────────► Amazon ECR
    │
    └──────────────► team1-chatguard-config
                           │
                           ▼
                        ArgoCD
                           │
                           ▼
                       Amazon EKS
```

### 배포 과정

1. 개발자가 `team1-chatguard-app` 저장소에 변경 사항을 Push하거나 Merge합니다.
2. GitHub Actions가 애플리케이션 이미지를 빌드하여 Amazon ECR에 업로드합니다.
3. GitHub Actions가 `team1-chatguard-config` 저장소의 `kustomization.yaml` 이미지 태그를 최신 버전으로 갱신합니다.
4. ArgoCD가 Config 저장소의 변경 사항을 감지합니다.
5. ArgoCD가 Amazon EKS 클러스터와 상태를 동기화하여 새로운 버전을 롤링 업데이트합니다.
