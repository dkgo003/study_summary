---
title: "Argo Workflows와 MLOps: DAG, Hera, Kubeflow, MLflow 흐름 정리"
date: "2026-05-09"
draft: false
slug: "argo-workflows와-mlops-dag-hera-kubeflow-mlflow-흐름-정리"
notion_status: "Published"
notion_page_id: "35b1d13b-f2d8-8084-a647-da93cbcc8d99"
categories:
  - "기술"
tags: []
---

데이터 처리와 머신러닝 작업은 단일 스크립트보다 여러 단계의 작업 흐름으로 구성되는 경우가 많다. Argo Workflows는 Kubernetes 위에서 컨테이너 기반 워크플로우를 실행하고, Hera는 이를 Python으로 정의하게 돕는다. Kubeflow와 MLflow는 머신러닝 라이프사이클을 운영하는 데 자주 함께 언급된다.

## Workflow와 DAG

- 워크플로우는 목표를 달성하기 위한 작업들의 순서와 의존성이다.

- DAG는 Directed Acyclic Graph의 약자로, 방향은 있지만 순환은 없는 작업 그래프다.

- 데이터 파이프라인에서는 전처리, 학습, 평가, 배포 같은 단계를 DAG로 표현하면 재현성과 장애 추적이 쉬워진다.

## Argo Workflows

- Argo Workflows는 Kubernetes에서 컨테이너 기반 작업을 Workflow 리소스로 실행하는 엔진이다.

- Workflow는 실제 실행 단위이고, WorkflowTemplate은 재사용 가능한 작업 템플릿이다.

- ClusterWorkflowTemplate은 클러스터 범위에서 여러 팀이 공유할 수 있는 템플릿이다.

- arguments와 parameter를 활용하면 같은 템플릿을 다양한 입력값으로 재사용할 수 있다.

## Hera

- Hera는 Argo Workflows를 Python 코드로 작성할 수 있게 해주는 SDK/DSL이다.

- YAML을 직접 작성하는 대신 Python 함수, 반복문, 조건문을 활용해 동적인 워크플로우를 구성할 수 있다.

- 다만 Hera는 버전에 따라 API 사용법이 달라질 수 있으므로 실제 예시는 사용하는 버전에 맞춰 검증해야 한다.

## Kubeflow

- Kubeflow는 Kubernetes 기반 머신러닝 플랫폼이다.

- Notebook, Pipeline, Training Operator, Serving 같은 구성 요소를 통해 ML 워크플로우 운영을 지원한다.

- Kubernetes 자원 관리와 ML 작업을 연결하기 좋지만, 플랫폼 운영 난이도는 낮지 않다.

## MLflow

- MLflow는 머신러닝 실험 추적과 모델 관리를 위한 플랫폼이다.

- Tracking은 파라미터, 메트릭, 아티팩트를 기록하고, Model Registry는 모델 버전과 승인 단계를 관리한다.

- Kubeflow가 인프라와 워크플로우 운영에 가깝다면, MLflow는 실험 추적과 모델 생애주기 관리에 강하다.

## 정리

Argo Workflows, Hera, Kubeflow, MLflow는 모두 자동화와 재현성을 높이기 위한 도구다. Argo는 Kubernetes 위의 작업 실행 엔진, Hera는 Python 기반 작성 도구, Kubeflow는 ML 플랫폼, MLflow는 실험과 모델 관리 도구로 역할을 나눠 이해하면 좋다.
