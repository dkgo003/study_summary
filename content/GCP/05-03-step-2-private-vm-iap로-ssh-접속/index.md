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

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/740304d3-1359-4e1e-92c3-f62b3aadc74c/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466VV5QW424%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084206Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCTPfjatXSmHbQJG0zdA%2FKmCk3F8rrDs%2F4cCj2Hl3jBeQIhAJN6pAWASVN8cCHblLyjlkoYKmTsVossAK1OYEI5B8OdKogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgwEsOxf9RNAktNndxEq3AN4FY6jh1zqcPOEGw07swCzkFYk1Lxa8px6woh4MOyKJTROkii83%2BScS2vKmiaNWqiBapLPrbDhPg7WYkbwJpapKu8I62udh07tdKAGHgklQx%2FVd9TAtIZwwHI62nxzOi7hswkW%2Bvo5QTpUvPqWrQz9ZC1YSkyGLVwndCOIHwd2rBx4MdMbRaYnQJ52cmCj6mwF7hPVppEibNIJFYhzx0JhkrLukjFf87v%2BWzhDHo6pdIRsGQSyPGCUBC8KI%2FzT1RT9u79x%2B%2BP%2F%2BYM5cnPzJZp4IETFeNVjYgjn05EcfFqNCWjDWqQS4gKfwDNv829jlOSkCcopmAWdtqt%2B4k4zJnhIZM%2FLkwaCKtEsnCkS8Tf1FW2BJief1zzusLhinf4aO20K3LiPlJxjpqdoNWu7Pqyjo3nr3FQxUoW%2FuMfBtkcF4iQMk%2FEqysSHxFpZp5pVa43zuzgAX64602OZcpbTwWwswmNby%2BBC42zkBIBx2G6TQaeKMiEpd5gfOYA%2BvFcyC4fVZnO8aVnrgUYRiInP6%2BI94TmNDxlCUUKEyBg8u6zYjVwQ3kGySdZx5jD36tEbnJD%2FaANIfV1%2ByDwwQcAZ5vtt%2Bw03ZX%2BvNDVFYaougYDjIYrH2EsmE0ic38mgPDD%2B1vvPBjqkATk2sIUlFOTBZN6afv%2BDZeVfy1BW8IAB18bTo%2B5flIZKNkceeBPWcsPfk85gB%2F0MrE7iBwMyM7bUteG8Mlft49f5%2FD3m3xp8d3PVk9W%2BH3C%2BmAARMMx0rDvqUNksMI2OY25yaNa04xATZVPoFTOYCoREnmsEOV92GeXXh4OOGQzsDD0crcbmWf5LhUxlvIgKlA7T28t2%2F8qEsVOoIVJ1pN%2BLGKJ3&X-Amz-Signature=4c5ac6b3bc59181b6d045b379ad3c97bf990356b5435b8fbc966b637c7bb734d&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

---

## Step 2-2 : Private VM 생성

Compute Engine → VM 인스턴스 → 인스턴스 만들기

### 머신 구성

- 이름: inkyu-private-vm

- 리전: asia-northeast3 (서울)

- 영역: asia-northeast3-a

- 머신 유형: e2-micro

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/dd0f66d7-5aba-4617-814c-403cdc019b6a/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466VV5QW424%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084206Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCTPfjatXSmHbQJG0zdA%2FKmCk3F8rrDs%2F4cCj2Hl3jBeQIhAJN6pAWASVN8cCHblLyjlkoYKmTsVossAK1OYEI5B8OdKogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgwEsOxf9RNAktNndxEq3AN4FY6jh1zqcPOEGw07swCzkFYk1Lxa8px6woh4MOyKJTROkii83%2BScS2vKmiaNWqiBapLPrbDhPg7WYkbwJpapKu8I62udh07tdKAGHgklQx%2FVd9TAtIZwwHI62nxzOi7hswkW%2Bvo5QTpUvPqWrQz9ZC1YSkyGLVwndCOIHwd2rBx4MdMbRaYnQJ52cmCj6mwF7hPVppEibNIJFYhzx0JhkrLukjFf87v%2BWzhDHo6pdIRsGQSyPGCUBC8KI%2FzT1RT9u79x%2B%2BP%2F%2BYM5cnPzJZp4IETFeNVjYgjn05EcfFqNCWjDWqQS4gKfwDNv829jlOSkCcopmAWdtqt%2B4k4zJnhIZM%2FLkwaCKtEsnCkS8Tf1FW2BJief1zzusLhinf4aO20K3LiPlJxjpqdoNWu7Pqyjo3nr3FQxUoW%2FuMfBtkcF4iQMk%2FEqysSHxFpZp5pVa43zuzgAX64602OZcpbTwWwswmNby%2BBC42zkBIBx2G6TQaeKMiEpd5gfOYA%2BvFcyC4fVZnO8aVnrgUYRiInP6%2BI94TmNDxlCUUKEyBg8u6zYjVwQ3kGySdZx5jD36tEbnJD%2FaANIfV1%2ByDwwQcAZ5vtt%2Bw03ZX%2BvNDVFYaougYDjIYrH2EsmE0ic38mgPDD%2B1vvPBjqkATk2sIUlFOTBZN6afv%2BDZeVfy1BW8IAB18bTo%2B5flIZKNkceeBPWcsPfk85gB%2F0MrE7iBwMyM7bUteG8Mlft49f5%2FD3m3xp8d3PVk9W%2BH3C%2BmAARMMx0rDvqUNksMI2OY25yaNa04xATZVPoFTOYCoREnmsEOV92GeXXh4OOGQzsDD0crcbmWf5LhUxlvIgKlA7T28t2%2F8qEsVOoIVJ1pN%2BLGKJ3&X-Amz-Signature=925f77672ad523b471af261edf6d91fdf519f4b467e2db7ddbc2819c18fa0a10&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

### 네트워킹 탭

- 네트워크: inkyu-vpc

- 서브넷: inkyu-subnet (10.0.0.0/24)

- 외부 IPv4 주소: 없음 ← 핵심 설정

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/cb0dbca1-3986-4418-a2da-80b957c84953/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466VV5QW424%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084206Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCTPfjatXSmHbQJG0zdA%2FKmCk3F8rrDs%2F4cCj2Hl3jBeQIhAJN6pAWASVN8cCHblLyjlkoYKmTsVossAK1OYEI5B8OdKogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgwEsOxf9RNAktNndxEq3AN4FY6jh1zqcPOEGw07swCzkFYk1Lxa8px6woh4MOyKJTROkii83%2BScS2vKmiaNWqiBapLPrbDhPg7WYkbwJpapKu8I62udh07tdKAGHgklQx%2FVd9TAtIZwwHI62nxzOi7hswkW%2Bvo5QTpUvPqWrQz9ZC1YSkyGLVwndCOIHwd2rBx4MdMbRaYnQJ52cmCj6mwF7hPVppEibNIJFYhzx0JhkrLukjFf87v%2BWzhDHo6pdIRsGQSyPGCUBC8KI%2FzT1RT9u79x%2B%2BP%2F%2BYM5cnPzJZp4IETFeNVjYgjn05EcfFqNCWjDWqQS4gKfwDNv829jlOSkCcopmAWdtqt%2B4k4zJnhIZM%2FLkwaCKtEsnCkS8Tf1FW2BJief1zzusLhinf4aO20K3LiPlJxjpqdoNWu7Pqyjo3nr3FQxUoW%2FuMfBtkcF4iQMk%2FEqysSHxFpZp5pVa43zuzgAX64602OZcpbTwWwswmNby%2BBC42zkBIBx2G6TQaeKMiEpd5gfOYA%2BvFcyC4fVZnO8aVnrgUYRiInP6%2BI94TmNDxlCUUKEyBg8u6zYjVwQ3kGySdZx5jD36tEbnJD%2FaANIfV1%2ByDwwQcAZ5vtt%2Bw03ZX%2BvNDVFYaougYDjIYrH2EsmE0ic38mgPDD%2B1vvPBjqkATk2sIUlFOTBZN6afv%2BDZeVfy1BW8IAB18bTo%2B5flIZKNkceeBPWcsPfk85gB%2F0MrE7iBwMyM7bUteG8Mlft49f5%2FD3m3xp8d3PVk9W%2BH3C%2BmAARMMx0rDvqUNksMI2OY25yaNa04xATZVPoFTOYCoREnmsEOV92GeXXh4OOGQzsDD0crcbmWf5LhUxlvIgKlA7T28t2%2F8qEsVOoIVJ1pN%2BLGKJ3&X-Amz-Signature=6c81008353152bbaa79d71b73caf0f44f6daf8b5a9842fc68979686b3ee50977&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

외부 IPv4 주소 옵션 설명

- 임시: 외부 IP 자동 할당, VM 재시작 시 변경됨 (Step 1에서 사용)

- 없음: 외부 IP 미할당 → 인터넷 직접 접근 불가 → IAP로만 접속 가능

- 고정: 재시작해도 IP 유지 (실서비스용, 별도 비용)

### 생성 결과 확인

VM 목록에서 inkyu-private-vm의 외부 IP 칸이 비어있으면 정상

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/ba296437-38e8-4b66-aa5a-48a48b77be1c/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466VV5QW424%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084206Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCTPfjatXSmHbQJG0zdA%2FKmCk3F8rrDs%2F4cCj2Hl3jBeQIhAJN6pAWASVN8cCHblLyjlkoYKmTsVossAK1OYEI5B8OdKogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgwEsOxf9RNAktNndxEq3AN4FY6jh1zqcPOEGw07swCzkFYk1Lxa8px6woh4MOyKJTROkii83%2BScS2vKmiaNWqiBapLPrbDhPg7WYkbwJpapKu8I62udh07tdKAGHgklQx%2FVd9TAtIZwwHI62nxzOi7hswkW%2Bvo5QTpUvPqWrQz9ZC1YSkyGLVwndCOIHwd2rBx4MdMbRaYnQJ52cmCj6mwF7hPVppEibNIJFYhzx0JhkrLukjFf87v%2BWzhDHo6pdIRsGQSyPGCUBC8KI%2FzT1RT9u79x%2B%2BP%2F%2BYM5cnPzJZp4IETFeNVjYgjn05EcfFqNCWjDWqQS4gKfwDNv829jlOSkCcopmAWdtqt%2B4k4zJnhIZM%2FLkwaCKtEsnCkS8Tf1FW2BJief1zzusLhinf4aO20K3LiPlJxjpqdoNWu7Pqyjo3nr3FQxUoW%2FuMfBtkcF4iQMk%2FEqysSHxFpZp5pVa43zuzgAX64602OZcpbTwWwswmNby%2BBC42zkBIBx2G6TQaeKMiEpd5gfOYA%2BvFcyC4fVZnO8aVnrgUYRiInP6%2BI94TmNDxlCUUKEyBg8u6zYjVwQ3kGySdZx5jD36tEbnJD%2FaANIfV1%2ByDwwQcAZ5vtt%2Bw03ZX%2BvNDVFYaougYDjIYrH2EsmE0ic38mgPDD%2B1vvPBjqkATk2sIUlFOTBZN6afv%2BDZeVfy1BW8IAB18bTo%2B5flIZKNkceeBPWcsPfk85gB%2F0MrE7iBwMyM7bUteG8Mlft49f5%2FD3m3xp8d3PVk9W%2BH3C%2BmAARMMx0rDvqUNksMI2OY25yaNa04xATZVPoFTOYCoREnmsEOV92GeXXh4OOGQzsDD0crcbmWf5LhUxlvIgKlA7T28t2%2F8qEsVOoIVJ1pN%2BLGKJ3&X-Amz-Signature=072afc1ebd7a04d91708807a21f91d2aa1aa483544d66af7c891d0fd46aa442a&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

---

## Step 2-3 : 방화벽 규칙 추가

VPC 네트워크 → 방화벽 → 방화벽 규칙 만들기

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/36cb9f14-7a7d-416d-b0a9-8ef1e739b6a4/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466VV5QW424%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084206Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCTPfjatXSmHbQJG0zdA%2FKmCk3F8rrDs%2F4cCj2Hl3jBeQIhAJN6pAWASVN8cCHblLyjlkoYKmTsVossAK1OYEI5B8OdKogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgwEsOxf9RNAktNndxEq3AN4FY6jh1zqcPOEGw07swCzkFYk1Lxa8px6woh4MOyKJTROkii83%2BScS2vKmiaNWqiBapLPrbDhPg7WYkbwJpapKu8I62udh07tdKAGHgklQx%2FVd9TAtIZwwHI62nxzOi7hswkW%2Bvo5QTpUvPqWrQz9ZC1YSkyGLVwndCOIHwd2rBx4MdMbRaYnQJ52cmCj6mwF7hPVppEibNIJFYhzx0JhkrLukjFf87v%2BWzhDHo6pdIRsGQSyPGCUBC8KI%2FzT1RT9u79x%2B%2BP%2F%2BYM5cnPzJZp4IETFeNVjYgjn05EcfFqNCWjDWqQS4gKfwDNv829jlOSkCcopmAWdtqt%2B4k4zJnhIZM%2FLkwaCKtEsnCkS8Tf1FW2BJief1zzusLhinf4aO20K3LiPlJxjpqdoNWu7Pqyjo3nr3FQxUoW%2FuMfBtkcF4iQMk%2FEqysSHxFpZp5pVa43zuzgAX64602OZcpbTwWwswmNby%2BBC42zkBIBx2G6TQaeKMiEpd5gfOYA%2BvFcyC4fVZnO8aVnrgUYRiInP6%2BI94TmNDxlCUUKEyBg8u6zYjVwQ3kGySdZx5jD36tEbnJD%2FaANIfV1%2ByDwwQcAZ5vtt%2Bw03ZX%2BvNDVFYaougYDjIYrH2EsmE0ic38mgPDD%2B1vvPBjqkATk2sIUlFOTBZN6afv%2BDZeVfy1BW8IAB18bTo%2B5flIZKNkceeBPWcsPfk85gB%2F0MrE7iBwMyM7bUteG8Mlft49f5%2FD3m3xp8d3PVk9W%2BH3C%2BmAARMMx0rDvqUNksMI2OY25yaNa04xATZVPoFTOYCoREnmsEOV92GeXXh4OOGQzsDD0crcbmWf5LhUxlvIgKlA7T28t2%2F8qEsVOoIVJ1pN%2BLGKJ3&X-Amz-Signature=a7da389d8a0111a0699f586d4c1391b01aa8cb01e1f40c3e0acd50db656a4a1d&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/f78c4cef-6e71-4d54-8b03-7db295f2de03/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466VV5QW424%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084206Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCTPfjatXSmHbQJG0zdA%2FKmCk3F8rrDs%2F4cCj2Hl3jBeQIhAJN6pAWASVN8cCHblLyjlkoYKmTsVossAK1OYEI5B8OdKogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgwEsOxf9RNAktNndxEq3AN4FY6jh1zqcPOEGw07swCzkFYk1Lxa8px6woh4MOyKJTROkii83%2BScS2vKmiaNWqiBapLPrbDhPg7WYkbwJpapKu8I62udh07tdKAGHgklQx%2FVd9TAtIZwwHI62nxzOi7hswkW%2Bvo5QTpUvPqWrQz9ZC1YSkyGLVwndCOIHwd2rBx4MdMbRaYnQJ52cmCj6mwF7hPVppEibNIJFYhzx0JhkrLukjFf87v%2BWzhDHo6pdIRsGQSyPGCUBC8KI%2FzT1RT9u79x%2B%2BP%2F%2BYM5cnPzJZp4IETFeNVjYgjn05EcfFqNCWjDWqQS4gKfwDNv829jlOSkCcopmAWdtqt%2B4k4zJnhIZM%2FLkwaCKtEsnCkS8Tf1FW2BJief1zzusLhinf4aO20K3LiPlJxjpqdoNWu7Pqyjo3nr3FQxUoW%2FuMfBtkcF4iQMk%2FEqysSHxFpZp5pVa43zuzgAX64602OZcpbTwWwswmNby%2BBC42zkBIBx2G6TQaeKMiEpd5gfOYA%2BvFcyC4fVZnO8aVnrgUYRiInP6%2BI94TmNDxlCUUKEyBg8u6zYjVwQ3kGySdZx5jD36tEbnJD%2FaANIfV1%2ByDwwQcAZ5vtt%2Bw03ZX%2BvNDVFYaougYDjIYrH2EsmE0ic38mgPDD%2B1vvPBjqkATk2sIUlFOTBZN6afv%2BDZeVfy1BW8IAB18bTo%2B5flIZKNkceeBPWcsPfk85gB%2F0MrE7iBwMyM7bUteG8Mlft49f5%2FD3m3xp8d3PVk9W%2BH3C%2BmAARMMx0rDvqUNksMI2OY25yaNa04xATZVPoFTOYCoREnmsEOV92GeXXh4OOGQzsDD0crcbmWf5LhUxlvIgKlA7T28t2%2F8qEsVOoIVJ1pN%2BLGKJ3&X-Amz-Signature=cd4c1b7f35a5747b8885bca601b22599bb6dc61673353ca00a6881917a56b471&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

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

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/9d063f67-8bd9-4046-8faf-7326b5c5a432/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466VV5QW424%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084206Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJIMEYCIQCTPfjatXSmHbQJG0zdA%2FKmCk3F8rrDs%2F4cCj2Hl3jBeQIhAJN6pAWASVN8cCHblLyjlkoYKmTsVossAK1OYEI5B8OdKogECOH%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNjM3NDIzMTgzODA1IgwEsOxf9RNAktNndxEq3AN4FY6jh1zqcPOEGw07swCzkFYk1Lxa8px6woh4MOyKJTROkii83%2BScS2vKmiaNWqiBapLPrbDhPg7WYkbwJpapKu8I62udh07tdKAGHgklQx%2FVd9TAtIZwwHI62nxzOi7hswkW%2Bvo5QTpUvPqWrQz9ZC1YSkyGLVwndCOIHwd2rBx4MdMbRaYnQJ52cmCj6mwF7hPVppEibNIJFYhzx0JhkrLukjFf87v%2BWzhDHo6pdIRsGQSyPGCUBC8KI%2FzT1RT9u79x%2B%2BP%2F%2BYM5cnPzJZp4IETFeNVjYgjn05EcfFqNCWjDWqQS4gKfwDNv829jlOSkCcopmAWdtqt%2B4k4zJnhIZM%2FLkwaCKtEsnCkS8Tf1FW2BJief1zzusLhinf4aO20K3LiPlJxjpqdoNWu7Pqyjo3nr3FQxUoW%2FuMfBtkcF4iQMk%2FEqysSHxFpZp5pVa43zuzgAX64602OZcpbTwWwswmNby%2BBC42zkBIBx2G6TQaeKMiEpd5gfOYA%2BvFcyC4fVZnO8aVnrgUYRiInP6%2BI94TmNDxlCUUKEyBg8u6zYjVwQ3kGySdZx5jD36tEbnJD%2FaANIfV1%2ByDwwQcAZ5vtt%2Bw03ZX%2BvNDVFYaougYDjIYrH2EsmE0ic38mgPDD%2B1vvPBjqkATk2sIUlFOTBZN6afv%2BDZeVfy1BW8IAB18bTo%2B5flIZKNkceeBPWcsPfk85gB%2F0MrE7iBwMyM7bUteG8Mlft49f5%2FD3m3xp8d3PVk9W%2BH3C%2BmAARMMx0rDvqUNksMI2OY25yaNa04xATZVPoFTOYCoREnmsEOV92GeXXh4OOGQzsDD0crcbmWf5LhUxlvIgKlA7T28t2%2F8qEsVOoIVJ1pN%2BLGKJ3&X-Amz-Signature=34bcb400c3c04dfd98ddbf30db4dbc482e9f6d668c5e5729c83808863f40a40f&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

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
