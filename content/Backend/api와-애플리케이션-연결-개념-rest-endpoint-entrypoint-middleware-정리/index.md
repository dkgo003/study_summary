---
title: "API와 애플리케이션 연결 개념: REST, Endpoint, Entrypoint, Middleware 정리"
date: "2026-05-09"
draft: false
slug: "api와-애플리케이션-연결-개념-rest-endpoint-entrypoint-middleware-정리"
notion_status: "Published"
notion_page_id: "35b1d13b-f2d8-807e-b822-f98297ecb08b"
categories:
  - "기술"
tags: []
---

애플리케이션은 혼자 동작하지 않는다. 외부 서비스와 API로 통신하고, 네트워크 접점인 Endpoint를 통해 요청을 받으며, 컨테이너나 프로그램의 Entrypoint에서 실행을 시작한다. Middleware는 이런 연결을 더 안정적으로 만들어주는 중간 계층이다.

## API와 REST

- API는 애플리케이션끼리 기능과 데이터를 주고받기 위한 약속이다.

- REST는 자원을 URI로 표현하고 HTTP 메서드로 행위를 나타내는 아키텍처 스타일이다.

- GET, POST, PUT/PATCH, DELETE 같은 메서드는 조회, 생성, 수정, 삭제 의도를 나타내는 데 쓰인다.

- RESTful하게 설계하려면 단순히 HTTP를 쓰는 것에서 끝나지 않고, 자원 중심 URI와 일관된 응답 구조를 유지해야 한다.

## Endpoint

- Endpoint는 서비스나 API에 접근하는 네트워크 접점이다.

- 웹 API에서는 URL 경로로, 네트워크 관점에서는 IP와 포트 조합으로 이해할 수 있다.

- Kubernetes에서는 Service 뒤의 실제 Pod 주소 집합을 Endpoint 또는 EndpointSlice로 관리한다.

## Entrypoint

- Entrypoint는 프로그램이나 컨테이너가 실행을 시작하는 지점이다.

- Dockerfile의 ENTRYPOINT는 컨테이너 시작 시 실행할 기본 명령을 정의한다.

- CMD는 기본 인자나 대체 가능한 명령으로 ENTRYPOINT와 함께 사용될 수 있다.

- Argo Workflows에서는 entrypoint가 처음 실행할 template을 가리킨다.

## Middleware

- Middleware는 서로 다른 애플리케이션이나 서비스, 인프라를 연결하는 중간 계층 소프트웨어다.

- API Gateway, 메시지 브로커, 인증 계층, WAS, 서비스 메시 등은 넓은 의미에서 미들웨어 역할을 할 수 있다.

- 클라우드 네이티브 환경에서는 서비스 간 통신, 인증, 라우팅, 관측성을 분리해서 관리하는 데 중요하다.

## CLI와 운영 자동화

- CLI는 명령줄 인터페이스로 도구를 제어하는 방식이다.

- git, curl, ssh, docker, kubectl 같은 CLI는 개발과 운영 자동화의 기본 도구다.

- 명령어는 스크립트로 재사용할 수 있어 반복 작업과 원격 운영에 강하다.

## 정리

API, Endpoint, Entrypoint, Middleware는 이름이 비슷하거나 함께 등장하지만 역할이 다르다. API는 통신 규칙, Endpoint는 접근 지점, Entrypoint는 실행 시작점, Middleware는 연결을 돕는 중간 계층으로 정리하면 혼동을 줄일 수 있다.
