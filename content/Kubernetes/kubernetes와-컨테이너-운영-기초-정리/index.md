---
title: "Kubernetes와 컨테이너 운영 기초 정리"
date: "2026-05-09"
draft: false
slug: "kubernetes와-컨테이너-운영-기초-정리"
notion_status: "Published"
notion_page_id: "35b1d13b-f2d8-80d8-9eec-dd733aa4c545"
categories:
  - "기술"
tags: []
---

컨테이너 기반 애플리케이션을 운영하려면 Docker로 실행 단위를 만들고, Kubernetes로 여러 서버 위의 컨테이너 상태를 안정적으로 관리하는 흐름을 이해해야 한다. 이 글은 컨테이너, Kubernetes 오브젝트, YAML manifest, 선언적 운영 방식을 하나의 흐름으로 정리한다.

## Docker와 컨테이너

- Docker는 애플리케이션과 실행 환경을 이미지로 패키징하고, 그 이미지를 컨테이너로 실행하게 해주는 플랫폼이다.

- 이미지는 실행 가능한 템플릿이고, 컨테이너는 이미지가 실제로 실행된 인스턴스다.

- Dockerfile은 이미지를 재현 가능하게 만들기 위한 명세 파일이다. FROM, RUN, COPY, WORKDIR, CMD, ENTRYPOINT 같은 지시어로 이미지 빌드 과정을 정의한다.

- 컨테이너는 VM보다 가볍지만 호스트 OS 커널을 공유하므로 격리와 권한, 네트워크, 볼륨 설정을 함께 이해해야 한다.

## Kubernetes가 필요한 이유

- 컨테이너가 한두 개일 때는 Docker만으로도 관리할 수 있지만, 서비스가 많아지고 서버가 여러 대가 되면 배포, 장애 복구, 스케일링, 네트워크 연결을 자동화할 필요가 생긴다.

- Kubernetes는 원하는 상태를 선언하면 컨트롤 플레인이 현재 상태를 계속 감시하고 차이를 줄이는 방식으로 동작한다.

- 주요 가치는 자가복구, 로드밸런싱, 롤링 업데이트와 롤백, 수평 확장이다.

## 핵심 오브젝트

- Pod는 Kubernetes에서 배포 가능한 가장 작은 실행 단위다. 보통 하나의 애플리케이션 컨테이너를 담지만, 필요하면 밀접하게 묶인 여러 컨테이너를 함께 담을 수 있다.

- Deployment는 Pod의 복제본 수, 이미지 버전, 업데이트 전략 같은 원하는 상태를 선언한다.

- Service는 계속 바뀔 수 있는 Pod IP 앞에 안정적인 접근 지점을 제공한다.

- ConfigMap과 Secret은 설정과 민감정보를 애플리케이션 이미지와 분리해서 관리하게 해준다.

## YAML manifest와 선언적 운영

- Kubernetes 리소스는 보통 YAML manifest로 정의한다. YAML은 들여쓰기로 구조를 표현하므로 탭 대신 스페이스를 사용하고, 계층 구조를 명확히 유지해야 한다.

- 선언적 방식은 '어떻게 만들지'보다 '어떤 상태여야 하는지'를 적는 방식이다. kubectl apply는 이 선언을 기준으로 클러스터 상태를 맞춘다.

- manifest는 Git에 저장하기 좋고, 변경 이력과 리뷰가 가능하기 때문에 GitOps와도 잘 맞는다.

## 운영자가 자주 쓰는 CLI

- kubectl get은 리소스 목록과 상태를 빠르게 확인할 때 사용한다.

- kubectl describe는 이벤트와 상세 상태를 확인해 Pending, CrashLoopBackOff 같은 문제를 분석할 때 유용하다.

- kubectl logs와 exec는 컨테이너 내부 로그와 실행 환경을 확인할 때 사용한다.

- kubectl rollout은 Deployment 업데이트 상태 확인과 롤백에 사용한다.

## 정리

Docker, Kubernetes, YAML, manifest, 선언적 운영은 따로 떨어진 개념이 아니라 하나의 운영 흐름으로 이어진다. 이미지를 만들고, 원하는 상태를 manifest로 선언하고, Kubernetes가 그 상태를 유지하게 만드는 것이 클라우드 네이티브 운영의 기본 구조다.
