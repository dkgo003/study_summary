---
title: "Kubernetes 배포 자동화: CI/CD, Harbor, Helm, Argo CD 흐름 정리"
date: "2026-05-09"
draft: false
slug: "kubernetes-배포-자동화-ci-cd-harbor-helm-argo-cd-흐름-정리"
notion_status: "Approved"
notion_page_id: "35b1d13b-f2d8-80d8-9f9e-fdedc6e42ee2"
categories:
  - "기술"
tags: []
---

# Kubernetes 배포 자동화: CI/CD, Harbor, Helm, Argo CD 흐름 정리

Kubernetes 환경에서 배포 자동화는 단순히 코드를 서버에 복사하는 작업이 아니다. 코드 변경을 검증하고, 컨테이너 이미지를 만들고, 레지스트리에 저장하고, Git에 선언된 배포 상태를 클러스터에 반영하는 흐름으로 구성된다.

## CI/CD의 역할

- CI는 지속적 통합으로, 코드 변경이 들어올 때마다 빌드와 테스트를 수행해 문제를 빠르게 발견하는 과정이다.

- CD는 지속적 전달 또는 지속적 배포로, 검증된 산출물을 운영 환경까지 안전하게 전달하는 과정이다.

- 좋은 파이프라인은 빌드, 테스트, 이미지 생성, 보안 검사, 배포, 롤백이 명확히 나뉘어 있다.

## Jenkins와 파이프라인

- Jenkins는 대표적인 CI/CD 서버로, 저장소 변경을 감지해 Pipeline을 실행할 수 있다.

- Jenkinsfile을 사용하면 빌드와 테스트, Docker 이미지 빌드, 배포 명령을 코드로 관리할 수 있다.

- 운영 관점에서는 에이전트 관리, 자격증명 관리, 플러그인 의존성, 빌드 로그 보존 정책을 함께 신경 써야 한다.

## Harbor와 이미지 레지스트리

- Harbor는 컨테이너 이미지를 저장하고 배포하기 위한 사내/엔터프라이즈 레지스트리로 활용된다.

- 이미지 취약점 스캔, 프로젝트별 권한 관리, 이미지 서명과 정책 관리 기능을 통해 배포 보안을 강화할 수 있다.

- CI 단계에서 이미지를 빌드하고 Harbor에 push하면, CD 단계에서는 해당 이미지를 Kubernetes에서 pull해 배포한다.

## Helm으로 배포 단위 패키징

- Helm은 Kubernetes 리소스 묶음을 Chart로 패키징하는 도구다.

- values.yaml을 사용하면 개발, 스테이징, 운영 환경별 설정 차이를 분리할 수 있다.

- 복잡한 manifest를 템플릿화할 수 있지만, 과도한 템플릿 로직은 오히려 유지보수를 어렵게 만들 수 있다.

## Argo CD와 GitOps

- GitOps는 Git 저장소를 운영 상태의 기준으로 삼는 방식이다.

- Argo CD는 Git에 선언된 manifest나 Helm Chart 상태와 클러스터 실제 상태를 비교하고, 차이가 있으면 동기화한다.

- 이 방식은 누가 언제 어떤 배포 상태를 바꿨는지 Git 이력으로 추적할 수 있고, 롤백도 Git revert 중심으로 단순해진다.

## 정리

Kubernetes 배포 자동화는 CI/CD, Harbor, Helm, Argo CD가 각자 역할을 나눠 맡을 때 안정적이다. CI는 품질을 검증하고, Harbor는 이미지를 보관하며, Helm은 배포 단위를 구조화하고, Argo CD는 Git 기준으로 클러스터 상태를 맞춘다.
