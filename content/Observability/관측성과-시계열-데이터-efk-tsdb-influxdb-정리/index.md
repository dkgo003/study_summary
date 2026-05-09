---
title: "관측성과 시계열 데이터: EFK, TSDB, InfluxDB 정리"
date: "2026-05-09"
draft: false
slug: "관측성과-시계열-데이터-efk-tsdb-influxdb-정리"
notion_status: "Approved"
notion_page_id: "35b1d13b-f2d8-80da-b18d-c36e9e1c2189"
categories:
  - "기술"
tags: []
---

# 관측성과 시계열 데이터: EFK, TSDB, InfluxDB 정리

운영 환경에서는 애플리케이션이 정상 동작하는지 계속 확인해야 한다. 이를 위해 로그, 메트릭, 이벤트를 수집하고 검색하고 시각화하는 관측성 구조가 필요하다. EFK는 로그 중심, TSDB와 InfluxDB는 시간 기반 메트릭 중심으로 이해하면 쉽다.

## EFK Stack

- EFK는 Elasticsearch, Fluentd, Kibana의 조합이다.

- Fluentd는 여러 소스의 로그를 수집하고 정제해 저장소로 전달한다.

- Elasticsearch는 로그를 색인하고 빠르게 검색할 수 있게 한다.

- Kibana는 저장된 로그를 탐색하고 대시보드로 시각화하는 UI를 제공한다.

- 최근 운영 환경에서는 Fluent Bit, OpenSearch 등 대체 조합도 자주 사용된다.

## TSDB가 필요한 이유

- TSDB는 Time Series Database의 약자로, 시간 순서로 계속 쌓이는 데이터를 저장하고 조회하는 데 최적화된 데이터베이스다.

- 서버 CPU, 메모리, 네트워크, 애플리케이션 응답 시간 같은 메트릭은 대부분 timestamp와 함께 저장된다.

- 일반 RDB와 달리 시간 범위 조회, downsampling, retention 정책이 중요하다.

## InfluxDB

- InfluxDB는 대표적인 오픈소스 TSDB다.

- InfluxDB 1.x에서는 Continuous Query와 Retention Policy가 많이 언급되고, InfluxDB 2.x에서는 Task와 Retention Period 개념으로 이어진다.

- 데이터가 무한히 쌓이지 않도록 보존 기간과 집계 전략을 함께 설계해야 한다.

## TICK Stack

- Telegraf는 메트릭과 이벤트를 수집한다.

- InfluxDB는 시계열 데이터를 저장한다.

- Chronograf는 시각화 역할을 담당한다.

- Kapacitor는 스트리밍 처리와 알림에 활용된다.

## 로그와 메트릭의 차이

- 로그는 특정 사건의 상세 맥락을 설명하는 데 강하다.

- 메트릭은 시간에 따른 수치 변화를 추적하는 데 강하다.

- 장애 대응에서는 메트릭으로 이상 징후를 발견하고, 로그로 원인을 좁혀가는 흐름이 일반적이다.

## 정리

관측성은 문제가 발생한 뒤 로그를 보는 수준을 넘어, 시스템 상태를 지속적으로 이해하기 위한 운영 체계다. EFK는 로그 분석에, TSDB와 InfluxDB는 시간 기반 메트릭 관리에 초점을 두며, 두 영역이 함께 있어야 장애 대응과 성능 개선이 가능하다.
