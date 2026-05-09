---
title: "Kubernetes 운영과 스토리지: 모니터링, PV/PVC, NFS, Rook-Ceph 정리"
date: "2026-05-09"
draft: false
slug: "kubernetes-운영과-스토리지-모니터링-pv-pvc-nfs-rook-ceph-정리"
notion_status: "Published"
notion_page_id: "35b1d13b-f2d8-809a-b602-f43490d0c3fe"
categories:
  - "기술"
tags: []
---

Kubernetes를 실제로 운영하면 배포보다 더 중요한 문제가 생긴다. 애플리케이션 상태를 어떻게 확인할지, 장애를 어떻게 추적할지, 상태 저장 데이터는 어디에 둘지 결정해야 한다. 이 글은 Kubernetes 운영과 스토리지 개념을 함께 정리한다.

## 운영 상태 확인과 디버깅

- kubectl describe는 리소스의 이벤트와 상태 변화 흐름을 보여주기 때문에 장애 분석의 출발점으로 좋다.

- kubectl logs는 컨테이너 로그를 확인하고, kubectl exec는 컨테이너 내부 환경을 점검할 때 사용한다.

- Argo Workflows를 사용한다면 argo get과 argo logs로 워크플로우 단계별 상태와 로그를 확인할 수 있다.

## PV와 PVC

- PV는 클러스터에 준비된 스토리지 자원이고, PVC는 사용자가 필요한 스토리지를 요청하는 리소스다.

- PVC는 용량, 접근 모드, StorageClass를 기준으로 PV와 바인딩된다.

- 동적 프로비저닝을 사용하면 PVC 생성 시 StorageClass를 통해 실제 스토리지가 자동으로 생성된다.

- 상태 저장 애플리케이션을 운영할 때는 Reclaim Policy, Access Mode, 백업 전략을 반드시 확인해야 한다.

## NFS

- NFS는 네트워크를 통해 원격 디렉터리를 로컬 파일시스템처럼 마운트하는 분산 파일 시스템 프로토콜이다.

- Kubernetes에서는 여러 Pod가 같은 파일시스템을 공유해야 할 때 ReadWriteMany 스토리지 후보로 쓰일 수 있다.

- 다만 성능, 파일 잠금, 네트워크 장애, 권한 설정 이슈를 고려해야 하므로 모든 워크로드에 적합한 것은 아니다.

## Rook-Ceph

- Ceph는 블록, 파일, 객체 스토리지를 모두 제공할 수 있는 분산 스토리지 시스템이다.

- Rook은 Ceph를 Kubernetes 위에서 Operator 방식으로 배포하고 운영하게 해준다.

- Rook-Ceph를 사용하면 Kubernetes 안에서 RBD, CephFS, Object Gateway 같은 스토리지 기능을 활용할 수 있다.

- 운영 난이도가 있는 편이므로 디스크 구성, 장애 도메인, 복구 시간, 모니터링을 함께 설계해야 한다.

## Object Storage

- 객체 스토리지는 데이터를 파일 경로보다 객체와 메타데이터 단위로 저장한다.

- S3 호환 API를 통해 접근하는 경우가 많고, 이미지, 로그 아카이브, 백업, 데이터 레이크에 적합하다.

- 파일 스토리지와 달리 계층형 디렉터리보다 확장성과 메타데이터 기반 관리에 초점이 있다.

## 정리

Kubernetes 운영에서 스토리지는 단순 부가 기능이 아니다. Stateless 애플리케이션은 쉽게 재시작할 수 있지만, 상태 저장 데이터는 복구와 일관성이 중요하다. PV/PVC, NFS, Rook-Ceph, Object Storage의 차이를 이해하면 워크로드에 맞는 저장 방식을 선택할 수 있다.
