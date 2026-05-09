---
title: "[05.03] Step 2 : Private VM + IAP로 SSH 접속"
date: "2026-05-09"
draft: false
slug: "05-03-step-2-private-vm-iap로-ssh-접속"
notion_status: "Approved"
notion_page_id: "35b1d13b-f2d8-80b8-8241-d60fe771d417"
categories:
  - "기술"
tags: []
---

# [05.03] Step 2 : Private VM + IAP로 SSH 접속

## 목표

외부 IP가 없는 Private VM을 생성하고, IAP(Identity-Aware Proxy)를 통해 SSH 접속하는 구조를 실습한다.

---

## Step 1 vs Step 2 비교

> 실무에서 Production VM은 대부분 Step 2 방식으로 운영한다.

---

## 아키텍처

```json
[로컬 PC / WSL]
     │
     │  gcloud compute ssh --tunnel-through-iap
     ▼
[Google IAP 서버 (35.235.240.0/20)]
     │
     │  인증된 사용자만 통과
     ▼
[Private VM — 외부 IP 없음]
  inkyu-private-vm
  내부 IP: 10.0.0.24
  VPC: inkyu-vpc / 서브넷: inkyu-subnet
```

---

## 사용 리소스

---

## Step 2-1 : IAP API 활성화

```bash
gcloud services enable iap.googleapis.com
```

> Operation ... finished successfully 출력 시 완료

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/740304d3-1359-4e1e-92c3-f62b3aadc74c/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RSPN5V5R%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084231Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCWaky9DXOjVaTNn1HtBUDxwAo%2FqxbSy8V7Rld4Png%2BkAIhAMnOq8slJRw85VLTlwhZ3zEl9r4Fy6i3bla5k5QNREH3KogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgyB9%2Br54klYZM8PrGEq3AMWV74sOvnES4T5jz0PkgqKG9JGFZvKuth%2FAfvenMsOwk654PVw3mWyPYmH1%2Frk8HwBgzE9YWk5RR0E%2BZyoAovncxsCZjciOBFKgdueqVwWsmFumk1Y%2Fi5myVghC87%2BK%2BVJcoOlXjZv2NFgwrC3WDlM5wWCJty0hFmpfF0WPM7EZDoFc8vb9dfnREjkk9pj%2FTnHt05%2BXiBCwKliQTuP%2F2eUmr%2FzhpEKoa0V9poLY7uW5BrPZkNwD2pQNK0d5nHPbCcNU%2FFkpzfE0M%2FqKZVFJNxLk8KtUWRgEzg4wy05iaJoxNQwG%2FecZZXFB%2FckfFVBvb0OqGolXRnQ7apzgRxINoB9xqVX5pFzCrWPnrDJ%2BmHv%2FYdIkw%2FZj3OE2eh0AFjFn9HN%2F%2BkXvUDaz6VwPdMylTxWoyJRQy4Hmb6qRvmLihxYi98gA3TLAiVGgWDy%2FJVuGz3DZV58xSgGeuXqb4YcffnziITWRVrbmndQELfYuUMgI7x19JB79DIJ9w%2BAfBw5EBpmS2WE6J%2F5mcph%2F7%2FBwn3LxW0ZUnoNaBLilBVGBlt8poANeqsPIv0Uj1q3QGmzan3cTf0vXSLO2HbJ7A69Un8lInzTc8nwhZgLWUuXRv8xYYuNwoNxpwufZHSAujDS2PvPBjqkAbyswmOrL%2B8Bz5e3sXkqU0jyUciI3cC7vZx8cXl0YQtoOAz5YBHNe%2Bh9n2CsRFxlg2Arg7gfvRtF5ggcvJM1ZHA9gfT%2FZAtL0yGhsWbQ7280%2FiV5goDVxCXMfIu7iDp4oG3s0pEuqp4%2BWmM9S%2B97EeOL1%2FpSdBXS7Md0RoLSE7ZGWdYnlX5pGI%2FjkgAGv8FdZF%2B93xMaLRjvO%2B71hY7F46tLugeT&X-Amz-Signature=c9c8d58105fef1d35d61414450762c57af1b8507599161a1bff9cf49c7bf4d1f&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

---

## Step 2-2 : Private VM 생성

Compute Engine → VM 인스턴스 → 인스턴스 만들기

### 머신 구성

- 이름: inkyu-private-vm

- 리전: asia-northeast3 (서울)

- 영역: asia-northeast3-a

- 머신 유형: e2-micro

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/dd0f66d7-5aba-4617-814c-403cdc019b6a/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RSPN5V5R%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084231Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCWaky9DXOjVaTNn1HtBUDxwAo%2FqxbSy8V7Rld4Png%2BkAIhAMnOq8slJRw85VLTlwhZ3zEl9r4Fy6i3bla5k5QNREH3KogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgyB9%2Br54klYZM8PrGEq3AMWV74sOvnES4T5jz0PkgqKG9JGFZvKuth%2FAfvenMsOwk654PVw3mWyPYmH1%2Frk8HwBgzE9YWk5RR0E%2BZyoAovncxsCZjciOBFKgdueqVwWsmFumk1Y%2Fi5myVghC87%2BK%2BVJcoOlXjZv2NFgwrC3WDlM5wWCJty0hFmpfF0WPM7EZDoFc8vb9dfnREjkk9pj%2FTnHt05%2BXiBCwKliQTuP%2F2eUmr%2FzhpEKoa0V9poLY7uW5BrPZkNwD2pQNK0d5nHPbCcNU%2FFkpzfE0M%2FqKZVFJNxLk8KtUWRgEzg4wy05iaJoxNQwG%2FecZZXFB%2FckfFVBvb0OqGolXRnQ7apzgRxINoB9xqVX5pFzCrWPnrDJ%2BmHv%2FYdIkw%2FZj3OE2eh0AFjFn9HN%2F%2BkXvUDaz6VwPdMylTxWoyJRQy4Hmb6qRvmLihxYi98gA3TLAiVGgWDy%2FJVuGz3DZV58xSgGeuXqb4YcffnziITWRVrbmndQELfYuUMgI7x19JB79DIJ9w%2BAfBw5EBpmS2WE6J%2F5mcph%2F7%2FBwn3LxW0ZUnoNaBLilBVGBlt8poANeqsPIv0Uj1q3QGmzan3cTf0vXSLO2HbJ7A69Un8lInzTc8nwhZgLWUuXRv8xYYuNwoNxpwufZHSAujDS2PvPBjqkAbyswmOrL%2B8Bz5e3sXkqU0jyUciI3cC7vZx8cXl0YQtoOAz5YBHNe%2Bh9n2CsRFxlg2Arg7gfvRtF5ggcvJM1ZHA9gfT%2FZAtL0yGhsWbQ7280%2FiV5goDVxCXMfIu7iDp4oG3s0pEuqp4%2BWmM9S%2B97EeOL1%2FpSdBXS7Md0RoLSE7ZGWdYnlX5pGI%2FjkgAGv8FdZF%2B93xMaLRjvO%2B71hY7F46tLugeT&X-Amz-Signature=fbf22cb0a9a4ed19b543ee059e77004ee6fa62acb49cd89bd9b9e756e11df3b0&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

### 네트워킹 탭

- 네트워크: inkyu-vpc

- 서브넷: inkyu-subnet (10.0.0.0/24)

- 외부 IPv4 주소: 없음 ← 핵심 설정

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/cb0dbca1-3986-4418-a2da-80b957c84953/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RSPN5V5R%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084231Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCWaky9DXOjVaTNn1HtBUDxwAo%2FqxbSy8V7Rld4Png%2BkAIhAMnOq8slJRw85VLTlwhZ3zEl9r4Fy6i3bla5k5QNREH3KogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgyB9%2Br54klYZM8PrGEq3AMWV74sOvnES4T5jz0PkgqKG9JGFZvKuth%2FAfvenMsOwk654PVw3mWyPYmH1%2Frk8HwBgzE9YWk5RR0E%2BZyoAovncxsCZjciOBFKgdueqVwWsmFumk1Y%2Fi5myVghC87%2BK%2BVJcoOlXjZv2NFgwrC3WDlM5wWCJty0hFmpfF0WPM7EZDoFc8vb9dfnREjkk9pj%2FTnHt05%2BXiBCwKliQTuP%2F2eUmr%2FzhpEKoa0V9poLY7uW5BrPZkNwD2pQNK0d5nHPbCcNU%2FFkpzfE0M%2FqKZVFJNxLk8KtUWRgEzg4wy05iaJoxNQwG%2FecZZXFB%2FckfFVBvb0OqGolXRnQ7apzgRxINoB9xqVX5pFzCrWPnrDJ%2BmHv%2FYdIkw%2FZj3OE2eh0AFjFn9HN%2F%2BkXvUDaz6VwPdMylTxWoyJRQy4Hmb6qRvmLihxYi98gA3TLAiVGgWDy%2FJVuGz3DZV58xSgGeuXqb4YcffnziITWRVrbmndQELfYuUMgI7x19JB79DIJ9w%2BAfBw5EBpmS2WE6J%2F5mcph%2F7%2FBwn3LxW0ZUnoNaBLilBVGBlt8poANeqsPIv0Uj1q3QGmzan3cTf0vXSLO2HbJ7A69Un8lInzTc8nwhZgLWUuXRv8xYYuNwoNxpwufZHSAujDS2PvPBjqkAbyswmOrL%2B8Bz5e3sXkqU0jyUciI3cC7vZx8cXl0YQtoOAz5YBHNe%2Bh9n2CsRFxlg2Arg7gfvRtF5ggcvJM1ZHA9gfT%2FZAtL0yGhsWbQ7280%2FiV5goDVxCXMfIu7iDp4oG3s0pEuqp4%2BWmM9S%2B97EeOL1%2FpSdBXS7Md0RoLSE7ZGWdYnlX5pGI%2FjkgAGv8FdZF%2B93xMaLRjvO%2B71hY7F46tLugeT&X-Amz-Signature=572a1f3c9f2cce642f33f1aa8273b738cb024a8ce330560a0e2d5e629fc42db4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

외부 IPv4 주소 옵션 설명

- 임시: 외부 IP 자동 할당, VM 재시작 시 변경됨 (Step 1에서 사용)

- 없음: 외부 IP 미할당 → 인터넷 직접 접근 불가 → IAP로만 접속 가능

- 고정: 재시작해도 IP 유지 (실서비스용, 별도 비용)

### 생성 결과 확인

VM 목록에서 inkyu-private-vm의 외부 IP 칸이 비어있으면 정상

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/ba296437-38e8-4b66-aa5a-48a48b77be1c/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RSPN5V5R%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084231Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCWaky9DXOjVaTNn1HtBUDxwAo%2FqxbSy8V7Rld4Png%2BkAIhAMnOq8slJRw85VLTlwhZ3zEl9r4Fy6i3bla5k5QNREH3KogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgyB9%2Br54klYZM8PrGEq3AMWV74sOvnES4T5jz0PkgqKG9JGFZvKuth%2FAfvenMsOwk654PVw3mWyPYmH1%2Frk8HwBgzE9YWk5RR0E%2BZyoAovncxsCZjciOBFKgdueqVwWsmFumk1Y%2Fi5myVghC87%2BK%2BVJcoOlXjZv2NFgwrC3WDlM5wWCJty0hFmpfF0WPM7EZDoFc8vb9dfnREjkk9pj%2FTnHt05%2BXiBCwKliQTuP%2F2eUmr%2FzhpEKoa0V9poLY7uW5BrPZkNwD2pQNK0d5nHPbCcNU%2FFkpzfE0M%2FqKZVFJNxLk8KtUWRgEzg4wy05iaJoxNQwG%2FecZZXFB%2FckfFVBvb0OqGolXRnQ7apzgRxINoB9xqVX5pFzCrWPnrDJ%2BmHv%2FYdIkw%2FZj3OE2eh0AFjFn9HN%2F%2BkXvUDaz6VwPdMylTxWoyJRQy4Hmb6qRvmLihxYi98gA3TLAiVGgWDy%2FJVuGz3DZV58xSgGeuXqb4YcffnziITWRVrbmndQELfYuUMgI7x19JB79DIJ9w%2BAfBw5EBpmS2WE6J%2F5mcph%2F7%2FBwn3LxW0ZUnoNaBLilBVGBlt8poANeqsPIv0Uj1q3QGmzan3cTf0vXSLO2HbJ7A69Un8lInzTc8nwhZgLWUuXRv8xYYuNwoNxpwufZHSAujDS2PvPBjqkAbyswmOrL%2B8Bz5e3sXkqU0jyUciI3cC7vZx8cXl0YQtoOAz5YBHNe%2Bh9n2CsRFxlg2Arg7gfvRtF5ggcvJM1ZHA9gfT%2FZAtL0yGhsWbQ7280%2FiV5goDVxCXMfIu7iDp4oG3s0pEuqp4%2BWmM9S%2B97EeOL1%2FpSdBXS7Md0RoLSE7ZGWdYnlX5pGI%2FjkgAGv8FdZF%2B93xMaLRjvO%2B71hY7F46tLugeT&X-Amz-Signature=414d78dd096642ebef80185d2965d15d204d657326cc34183d73189adfd6ca39&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

---

## Step 2-3 : 방화벽 규칙 추가

VPC 네트워크 → 방화벽 → 방화벽 규칙 만들기

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/36cb9f14-7a7d-416d-b0a9-8ef1e739b6a4/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RSPN5V5R%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084231Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCWaky9DXOjVaTNn1HtBUDxwAo%2FqxbSy8V7Rld4Png%2BkAIhAMnOq8slJRw85VLTlwhZ3zEl9r4Fy6i3bla5k5QNREH3KogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgyB9%2Br54klYZM8PrGEq3AMWV74sOvnES4T5jz0PkgqKG9JGFZvKuth%2FAfvenMsOwk654PVw3mWyPYmH1%2Frk8HwBgzE9YWk5RR0E%2BZyoAovncxsCZjciOBFKgdueqVwWsmFumk1Y%2Fi5myVghC87%2BK%2BVJcoOlXjZv2NFgwrC3WDlM5wWCJty0hFmpfF0WPM7EZDoFc8vb9dfnREjkk9pj%2FTnHt05%2BXiBCwKliQTuP%2F2eUmr%2FzhpEKoa0V9poLY7uW5BrPZkNwD2pQNK0d5nHPbCcNU%2FFkpzfE0M%2FqKZVFJNxLk8KtUWRgEzg4wy05iaJoxNQwG%2FecZZXFB%2FckfFVBvb0OqGolXRnQ7apzgRxINoB9xqVX5pFzCrWPnrDJ%2BmHv%2FYdIkw%2FZj3OE2eh0AFjFn9HN%2F%2BkXvUDaz6VwPdMylTxWoyJRQy4Hmb6qRvmLihxYi98gA3TLAiVGgWDy%2FJVuGz3DZV58xSgGeuXqb4YcffnziITWRVrbmndQELfYuUMgI7x19JB79DIJ9w%2BAfBw5EBpmS2WE6J%2F5mcph%2F7%2FBwn3LxW0ZUnoNaBLilBVGBlt8poANeqsPIv0Uj1q3QGmzan3cTf0vXSLO2HbJ7A69Un8lInzTc8nwhZgLWUuXRv8xYYuNwoNxpwufZHSAujDS2PvPBjqkAbyswmOrL%2B8Bz5e3sXkqU0jyUciI3cC7vZx8cXl0YQtoOAz5YBHNe%2Bh9n2CsRFxlg2Arg7gfvRtF5ggcvJM1ZHA9gfT%2FZAtL0yGhsWbQ7280%2FiV5goDVxCXMfIu7iDp4oG3s0pEuqp4%2BWmM9S%2B97EeOL1%2FpSdBXS7Md0RoLSE7ZGWdYnlX5pGI%2FjkgAGv8FdZF%2B93xMaLRjvO%2B71hY7F46tLugeT&X-Amz-Signature=567b4144103a8ac213184071351178784085b1ea34d0e2abd0c8e70640a71b90&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/f78c4cef-6e71-4d54-8b03-7db295f2de03/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RSPN5V5R%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084231Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCWaky9DXOjVaTNn1HtBUDxwAo%2FqxbSy8V7Rld4Png%2BkAIhAMnOq8slJRw85VLTlwhZ3zEl9r4Fy6i3bla5k5QNREH3KogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgyB9%2Br54klYZM8PrGEq3AMWV74sOvnES4T5jz0PkgqKG9JGFZvKuth%2FAfvenMsOwk654PVw3mWyPYmH1%2Frk8HwBgzE9YWk5RR0E%2BZyoAovncxsCZjciOBFKgdueqVwWsmFumk1Y%2Fi5myVghC87%2BK%2BVJcoOlXjZv2NFgwrC3WDlM5wWCJty0hFmpfF0WPM7EZDoFc8vb9dfnREjkk9pj%2FTnHt05%2BXiBCwKliQTuP%2F2eUmr%2FzhpEKoa0V9poLY7uW5BrPZkNwD2pQNK0d5nHPbCcNU%2FFkpzfE0M%2FqKZVFJNxLk8KtUWRgEzg4wy05iaJoxNQwG%2FecZZXFB%2FckfFVBvb0OqGolXRnQ7apzgRxINoB9xqVX5pFzCrWPnrDJ%2BmHv%2FYdIkw%2FZj3OE2eh0AFjFn9HN%2F%2BkXvUDaz6VwPdMylTxWoyJRQy4Hmb6qRvmLihxYi98gA3TLAiVGgWDy%2FJVuGz3DZV58xSgGeuXqb4YcffnziITWRVrbmndQELfYuUMgI7x19JB79DIJ9w%2BAfBw5EBpmS2WE6J%2F5mcph%2F7%2FBwn3LxW0ZUnoNaBLilBVGBlt8poANeqsPIv0Uj1q3QGmzan3cTf0vXSLO2HbJ7A69Un8lInzTc8nwhZgLWUuXRv8xYYuNwoNxpwufZHSAujDS2PvPBjqkAbyswmOrL%2B8Bz5e3sXkqU0jyUciI3cC7vZx8cXl0YQtoOAz5YBHNe%2Bh9n2CsRFxlg2Arg7gfvRtF5ggcvJM1ZHA9gfT%2FZAtL0yGhsWbQ7280%2FiV5goDVxCXMfIu7iDp4oG3s0pEuqp4%2BWmM9S%2B97EeOL1%2FpSdBXS7Md0RoLSE7ZGWdYnlX5pGI%2FjkgAGv8FdZF%2B93xMaLRjvO%2B71hY7F46tLugeT&X-Amz-Signature=4b4e3cebda60a30bd1c335a27dfd765a4b612a165b615e304dfbc0b1d3ec42d9&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

> 35.235.240.0/20 은 Google이 IAP 서비스에 고정으로 사용하는 IP 대역 (공식 문서 명시)
Step 1의 방화벽이 내 로컬 IP를 소스로 설정했다면, Step 2는 IAP 서버 대역이 소스

---

## Step 2-4 : IAP SSH 접속 테스트

WSL 터미널에서:

```bash
gcloud compute ssh inkyu-private-vm \
  --zone=asia-northeast3-a \
  --tunnel-through-iap
```

접속 성공 시 VM 내부 프롬프트 진입 확인

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/9d063f67-8bd9-4046-8faf-7326b5c5a432/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RSPN5V5R%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084231Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCWaky9DXOjVaTNn1HtBUDxwAo%2FqxbSy8V7Rld4Png%2BkAIhAMnOq8slJRw85VLTlwhZ3zEl9r4Fy6i3bla5k5QNREH3KogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgyB9%2Br54klYZM8PrGEq3AMWV74sOvnES4T5jz0PkgqKG9JGFZvKuth%2FAfvenMsOwk654PVw3mWyPYmH1%2Frk8HwBgzE9YWk5RR0E%2BZyoAovncxsCZjciOBFKgdueqVwWsmFumk1Y%2Fi5myVghC87%2BK%2BVJcoOlXjZv2NFgwrC3WDlM5wWCJty0hFmpfF0WPM7EZDoFc8vb9dfnREjkk9pj%2FTnHt05%2BXiBCwKliQTuP%2F2eUmr%2FzhpEKoa0V9poLY7uW5BrPZkNwD2pQNK0d5nHPbCcNU%2FFkpzfE0M%2FqKZVFJNxLk8KtUWRgEzg4wy05iaJoxNQwG%2FecZZXFB%2FckfFVBvb0OqGolXRnQ7apzgRxINoB9xqVX5pFzCrWPnrDJ%2BmHv%2FYdIkw%2FZj3OE2eh0AFjFn9HN%2F%2BkXvUDaz6VwPdMylTxWoyJRQy4Hmb6qRvmLihxYi98gA3TLAiVGgWDy%2FJVuGz3DZV58xSgGeuXqb4YcffnziITWRVrbmndQELfYuUMgI7x19JB79DIJ9w%2BAfBw5EBpmS2WE6J%2F5mcph%2F7%2FBwn3LxW0ZUnoNaBLilBVGBlt8poANeqsPIv0Uj1q3QGmzan3cTf0vXSLO2HbJ7A69Un8lInzTc8nwhZgLWUuXRv8xYYuNwoNxpwufZHSAujDS2PvPBjqkAbyswmOrL%2B8Bz5e3sXkqU0jyUciI3cC7vZx8cXl0YQtoOAz5YBHNe%2Bh9n2CsRFxlg2Arg7gfvRtF5ggcvJM1ZHA9gfT%2FZAtL0yGhsWbQ7280%2FiV5goDVxCXMfIu7iDp4oG3s0pEuqp4%2BWmM9S%2B97EeOL1%2FpSdBXS7Md0RoLSE7ZGWdYnlX5pGI%2FjkgAGv8FdZF%2B93xMaLRjvO%2B71hY7F46tLugeT&X-Amz-Signature=41f3f8a4787d6be9a2fd4e85faa6c6bff35c5fc36181957e8b101bc4603ac7ab&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

---

## 핵심 개념 정리

### IAP (Identity-Aware Proxy)란?

Google Cloud가 제공하는 인증 기반 접근 제어 프록시 서비스다.
기존에는 서버에 접근하려면 방화벽에서 IP를 열거나 VPN을 구성해야 했지만,
IAP는 Google 계정 인증을 통과한 사용자에게만 접근을 허용하는 방식으로 이를 대체한다.

동작 방식

```bash
사용자 → Google IAP (인증 확인) → 통과 시 VM으로 터널링
                                  → 실패 시 차단
```

- VM에 공인 IP가 없어도 SSH 접속 가능

- Google 계정 + IAM 권한으로 접근 제어 (별도 VPN 불필요)

- VM 입장에서는 35.235.240.0/20 대역에서 요청이 들어오는 것처럼 보임

- 접속 로그가 Cloud Audit Logs에 자동 기록됨

IAP를 쓰는 상황

IAP가 필요 없는 상황

- 이미 사내 VPN이 잘 구성되어 있을 때

- 완전히 폐쇄된 내부망에서만 운영할 때

### VPC / 서브넷 / VM 관계

```bash
VPC (inkyu-vpc)              ← 네트워크 전체 (아파트 단지)
  └── 서브넷 (inkyu-subnet)  ← 네트워크 구역 (동)
        └── VM               ← 실제 서버 (호수)
```

---

## 트러블슈팅 메모

---

## 핵심 포인트

- 외부 IP 없음 설정만으로 인터넷 직접 노출 차단 가능

- IAP 방화벽 소스는 항상 35.235.240.0/20 고정

- -tunnel-through-iap 옵션 하나로 VPN 없이 Private VM SSH 가능

- default-allow-ssh 같은 기본 규칙 있으면 보안 의미 없어짐 → 반드시 확인/삭제
