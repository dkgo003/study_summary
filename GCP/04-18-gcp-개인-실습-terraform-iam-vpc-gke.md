---
title: "[04.18] GCP 개인 실습 — Terraform + IAM + VPC + GKE"
date: "2026-05-05"
draft: false
notion_status: "Approved"
notion_page_id: "3561d13b-f2d8-8032-bcfd-ffa0beeeac19"
categories:
  - "기술"
tags: []
---

# [04.18] GCP 개인 실습 — Terraform + IAM + VPC + GKE

## 환경 세팅

목적: WSL 환경에서 GCP CLI 및 Terraform 설치하여 로컬에서 GCP 인프라를 코드로 관리할 수 있는 환경 구성

### gcloud CLI 설치 및 인증

```javascript
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud init
gcloud auth application-default login
```

- gcloud init → 구글 계정 연동 + 프로젝트 선택

- application-default login → Terraform이 GCP API 호출할 때 쓰는 ADC 인증

- WSL 환경에서 브라우저 자동 오픈 안 될 경우 → BROWSER 환경변수로 Chrome 경로 지정

### Terraform 설치

```javascript
sudo apt install -y gnupg software-properties-common curl
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform
terraform -v  # 설치 확인
```

---

## Terraform 기본 개념

### 기본 명령어 흐름

```javascript
terraform init     # provider 플러그인 설치 (npm install 같은 것)
terraform plan     # 변경사항 미리보기
terraform apply    # 실제 적용
terraform destroy  # 전체 삭제
```

### plan 결과 읽는 법

```hcl
+ create  → 새로 만들 리소스
~ update  → 수정될 리소스
- destroy → 삭제될 리소스
```

---

## 파일 구조

실무에서는 리소스별로 파일 나눠서 관리

```hcl
terraform-study/
├── main.tf       # provider 설정 + GCS 버킷 + SA + IAM
├── vpc.tf        # VPC + Subnet + Firewall
└── gke.tf        # GKE 클러스터 + 노드풀
```

---

## 실습 1 — GCS 버킷 생성

목적: Terraform으로 GCP Cloud Storage 버킷을 코드로 선언하고 생성

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = "project-7cfb9c0b-aff8-4961-a4f"
  region  = "asia-northeast3"
}

# GCS 버킷 생성
resource "google_storage_bucket" "test" {
  name                        = "inkyu-test-bucket-2026"
  location                    = "ASIA-NORTHEAST3"
  uniform_bucket_level_access = true  # GCP 조직 정책 필수 옵션
}
```

결과: inkyu-test-bucket-2026 버킷 서울 리전에 생성 확인

> ⚠️ uniform_bucket_level_access = true 없으면 조직 정책 위반으로 에러 남

---

## 실습 2 — Service Account + IAM 바인딩

목적: 앱/서비스가 GCP 리소스에 접근할 때 쓰는 SA를 Terraform으로 생성하고, GCS 읽기 권한을 IAM으로 부여

```hcl
# Service Account 생성
resource "google_service_account" "app_sa" {
  account_id   = "inkyu-test-sa"
  display_name = "Inkyu Test Service Account"
}

# SA에 GCS 읽기 권한 부여 (IAM 바인딩)
resource "google_project_iam_member" "sa_storage_viewer" {
  project = "project-7cfb9c0b-aff8-4961-a4f"
  role    = "roles/storage.objectViewer"
  member  = "serviceAccount:${google_service_account.app_sa.email}"
}
```

IAM 개념 정리

결과: 콘솔 IAM에서 inkyu-test-sa → 저장소 개체 뷰어 바인딩 확인

> Terraform이 SA 생성 → IAM 바인딩 순서 자동으로 맞춰서 실행 (의존성 자동 파악)

---

## 실습 3 — Remote Backend (tfstate → GCS)

목적: 로컬에 저장되던 tfstate를 GCS에 저장 → 팀 협업 가능, 파일 유실 방지

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
  # tfstate를 GCS에 저장
  backend "gcs" {
    bucket = "inkyu-test-bucket-2026"
    prefix = "terraform/state"
  }
}
```

bash

terraform init -migrate-state  # 로컬 state → GCS로 이전

결과: GCS inkyu-test-bucket-2026/terraform/state/default.tfstate 생성 확인

---

## 실습 4 — VPC + Subnet + Firewall (vpc.tf)

목적: GKE 등 리소스를 올릴 커스텀 네트워크 구성. 실무에서는 VPC 먼저 만들고 그 위에 GKE 올리는 게 정석

### 네트워크 구조 개념

- VPC      = 건물 전체 내부 네트워크 (사설망)

- Subnet   = 층별로 나눈 구역

- VM/GKE   = 그 안에 있는 컴퓨터들\

- Firewall = 건물 출입 보안 규칙

```hcl
# VPC 생성
resource "google_compute_network" "main" {
  name                    = "inkyu-vpc"
  auto_create_subnetworks = false  # subnet을 직접 만들겠다는 옵션 (커스텀 모드)
}

# Subnet 생성
resource "google_compute_subnetwork" "main" {
  name          = "inkyu-subnet"
  ip_cidr_range = "10.0.0.0/24"
  region        = "asia-northeast3"
  network       = google_compute_network.main.id
}

# Firewall Rule - 내부 통신 허용
resource "google_compute_firewall" "allow_internal" {
  name    = "inkyu-allow-internal"
  network = google_compute_network.main.name

  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }

  source_ranges = ["10.0.0.0/24"]
}
```

결과: VPC 콘솔에서 inkyu-vpc (커스텀 모드) 생성 확인

> Terraform이 VPC 먼저 생성 후 Subnet + Firewall 병렬 생성 (의존성 자동 파악)

> ⚠️ Compute Engine API 비활성화 시 에러 → gcloud services enable compute.googleapis.com 또는 콘솔에서 활성화

---

## 실습 5 — GKE 클러스터 + 노드풀 (gke.tf)

목적: 아까 만든 inkyu-vpc 위에 GKE 클러스터 생성. kubectl로 연결까지

```hcl
# GKE 클러스터 생성
resource "google_container_cluster" "main" {
  name     = "inkyu-gke-cluster"
  location = "asia-northeast3"

  network    = google_compute_network.main.name
  subnetwork = google_compute_subnetwork.main.name

  # default node pool 삭제 후 별도 node pool로 관리 (실무 표준)
  remove_default_node_pool = true
  initial_node_count       = 1

  deletion_protection = false  # 실습용

  node_config {
    machine_type = "e2-medium"
    disk_size_gb = 30
    disk_type    = "pd-standard"  # SSD 쿼터 절약
  }
}

# Node Pool 생성
resource "google_container_node_pool" "main" {
  name       = "inkyu-node-pool"
  cluster    = google_container_cluster.main.name
  location   = "asia-northeast3"
  node_count = 1  # 0으로 설정하면 노드 없이 클러스터만 유지

  node_config {
    machine_type = "e2-medium"
    disk_size_gb = 30
    disk_type    = "pd-standard"
  }
}
```

> ⚠️ location을 리전으로 설정하면 zone 3개(a/b/c)에 자동 분산 → node_count=1 이어도 노드 3개 생성됨. zone 하나만 지정하면 1개만 생성

> ⚠️ GKE API 활성화 필요 → gcloud services enable container.googleapis.com

> ⚠️ 무료 계정 SSD 쿼터 250GB 제한 → disk_type = "pd-standard" + disk_size_gb = 30 으로 해결

### kubectl 연결

```hcl
# kubectl + auth plugin 설치
gcloud components install kubectl
gcloud components install gke-gcloud-auth-plugin

# kubeconfig 설정
gcloud container clusters get-credentials inkyu-gke-cluster --region asia-northeast3

# 노드 확인
kubectl get nodes
```

결과:

```hcl
NAME                                                  STATUS   ROLES    AGE     VERSION
gke-inkyu-gke-cluster-inkyu-node-pool-23d11de4-tks5   Ready    <none>   3m57s   v1.35.1-gke.1396002
gke-inkyu-gke-cluster-inkyu-node-pool-3f444324-1nxl   Ready    <none>   4m3s    v1.35.1-gke.1396002
gke-inkyu-gke-cluster-inkyu-node-pool-6b3e8a85-jmkt   Ready    <none>   3m45s   v1.35.1-gke.1396002
```

---

## 크레딧 관리 팁

```hcl
# 노드만 끄기 (클러스터 유지)
# gke.tf에서 node_count = 0 으로 변경 후
terraform apply

# 전체 삭제
terraform destroy
```

---

## 오늘 만든 것 요약

```hcl
✅ GCS 버킷 (inkyu-test-bucket-2026) - 서울 리전
✅ Service Account (inkyu-test-sa)
✅ IAM 바인딩 (inkyu-test-sa → roles/storage.objectViewer)
✅ Remote Backend (tfstate → GCS 저장)
✅ VPC (inkyu-vpc) + Subnet (10.0.0.0/24) + Firewall
✅ GKE 클러스터 (inkyu-gke-cluster) + 노드풀
✅ kubectl 연결 완료
```
