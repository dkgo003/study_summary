---
title: "[05.03] Step 2 : Private VM + IAP로 SSH 접속"
date: "2026-05-09"
draft: false
slug: "05-03-step-2-private-vm-iap로-ssh-접속"
notion_status: "Published"
notion_page_id: "35b1d13b-f2d8-80b8-8241-d60fe771d417"
categories:
  - "기술"
tags: []
---

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

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/740304d3-1359-4e1e-92c3-f62b3aadc74c/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RC4EYJBY%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084703Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJGMEQCIBD9o87d%2B9JKIho9n3o3ilAcGRbtBWl1DAb5C9YSNFGlAiBoMd4xoFjhy4QVRneJLgH0F3alzbuShmkztI8smP2pBSqIBAjh%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDYzNzQyMzE4MzgwNSIMd%2FBo9lUJsUFqm5P2KtwDwSqnCMRln45VgWmOVrStC%2BZkwfJiLujAY6CXQ6iApqCcWDVHhf%2BLeTrWeIT%2B%2FJcjzb1pQHZ488bKCgifknHr%2FxeI80YLSMXraVcwY8EEFWL6rBA2fhtmFfF9bhRIu00EZx51t3FnXJFV852oDTUi3oMhZyqhauHaATaTE3rb35cW7k67NbdtaU4TSR9wlweVnabvIbSJq6MxPzMJNAmyPA3HLMFvt45GMnLbo%2FEyxP0qdLLpW414egeNaCyJS%2B%2BfJRl9CJ4HTlObSLcbj2HIfKdbBRBdwPKUMaQDGkA6nglTWf16LN%2FIhWUfPY7K2OKqkISBC8aB30mZBZ%2FV6yydU%2FaFhhoHwNHopTyqWdjRiPI%2F2%2Fbcz%2FpPY382DeuRZpBkne8KQzDp5p74KMFI2%2B9uxq3bP8988NgjLhaioYFW23wPBbnzUlBujAs%2BPqFFc9NSkK1teVza1Fi825AzMczp9lOMLT7CynqZCwyWThswUf9V6Hgdc64rrZuPpwY3vmTutkwdvOWzVnepdoD7ps3x9jVkhF%2FM81vmWSqx%2B1KqatEXkiWeXfN2Ltz%2BALblwYEu2%2FhhYmnRNFohXnlypXRgCZSHpmMuDlwRQ28cGvATGoCz3eMIx3TKjhMWYwow1tf7zwY6pgEUdtpj7RJL8Vjc7Bj6e48yeFcsckea7ugGOuhfaFVuC%2FV9QB4C9hkUO%2BgzTXor6%2FiQ1QmcXQ5fCL%2F74zMFrGwt9wzep7e9Ngw4A6GW5Ucr2XAGQSDR8SBZMjj%2BPS52aGvpWFGtRDg8Y44SlkWsUWiBl%2FeAfTXW%2BZrVI%2FoenX1GFkKaRZPvokvQJfSuriCORcXuS90%2FeijEA26LZ33AIYCAuCL33dKG&X-Amz-Signature=82c79edef3eb5222ef46d2299467bde769041284f879de287563ea2be219094f&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

---

## Step 2-2 : Private VM 생성

Compute Engine → VM 인스턴스 → 인스턴스 만들기

### 머신 구성

- 이름: inkyu-private-vm

- 리전: asia-northeast3 (서울)

- 영역: asia-northeast3-a

- 머신 유형: e2-micro

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/dd0f66d7-5aba-4617-814c-403cdc019b6a/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RC4EYJBY%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084703Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJGMEQCIBD9o87d%2B9JKIho9n3o3ilAcGRbtBWl1DAb5C9YSNFGlAiBoMd4xoFjhy4QVRneJLgH0F3alzbuShmkztI8smP2pBSqIBAjh%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDYzNzQyMzE4MzgwNSIMd%2FBo9lUJsUFqm5P2KtwDwSqnCMRln45VgWmOVrStC%2BZkwfJiLujAY6CXQ6iApqCcWDVHhf%2BLeTrWeIT%2B%2FJcjzb1pQHZ488bKCgifknHr%2FxeI80YLSMXraVcwY8EEFWL6rBA2fhtmFfF9bhRIu00EZx51t3FnXJFV852oDTUi3oMhZyqhauHaATaTE3rb35cW7k67NbdtaU4TSR9wlweVnabvIbSJq6MxPzMJNAmyPA3HLMFvt45GMnLbo%2FEyxP0qdLLpW414egeNaCyJS%2B%2BfJRl9CJ4HTlObSLcbj2HIfKdbBRBdwPKUMaQDGkA6nglTWf16LN%2FIhWUfPY7K2OKqkISBC8aB30mZBZ%2FV6yydU%2FaFhhoHwNHopTyqWdjRiPI%2F2%2Fbcz%2FpPY382DeuRZpBkne8KQzDp5p74KMFI2%2B9uxq3bP8988NgjLhaioYFW23wPBbnzUlBujAs%2BPqFFc9NSkK1teVza1Fi825AzMczp9lOMLT7CynqZCwyWThswUf9V6Hgdc64rrZuPpwY3vmTutkwdvOWzVnepdoD7ps3x9jVkhF%2FM81vmWSqx%2B1KqatEXkiWeXfN2Ltz%2BALblwYEu2%2FhhYmnRNFohXnlypXRgCZSHpmMuDlwRQ28cGvATGoCz3eMIx3TKjhMWYwow1tf7zwY6pgEUdtpj7RJL8Vjc7Bj6e48yeFcsckea7ugGOuhfaFVuC%2FV9QB4C9hkUO%2BgzTXor6%2FiQ1QmcXQ5fCL%2F74zMFrGwt9wzep7e9Ngw4A6GW5Ucr2XAGQSDR8SBZMjj%2BPS52aGvpWFGtRDg8Y44SlkWsUWiBl%2FeAfTXW%2BZrVI%2FoenX1GFkKaRZPvokvQJfSuriCORcXuS90%2FeijEA26LZ33AIYCAuCL33dKG&X-Amz-Signature=226abe91f76444ad9e358513dff3228f03c3af18b8c6af84b50e36f0ba262b89&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

### 네트워킹 탭

- 네트워크: inkyu-vpc

- 서브넷: inkyu-subnet (10.0.0.0/24)

- 외부 IPv4 주소: 없음 ← 핵심 설정

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/cb0dbca1-3986-4418-a2da-80b957c84953/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RC4EYJBY%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084703Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJGMEQCIBD9o87d%2B9JKIho9n3o3ilAcGRbtBWl1DAb5C9YSNFGlAiBoMd4xoFjhy4QVRneJLgH0F3alzbuShmkztI8smP2pBSqIBAjh%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDYzNzQyMzE4MzgwNSIMd%2FBo9lUJsUFqm5P2KtwDwSqnCMRln45VgWmOVrStC%2BZkwfJiLujAY6CXQ6iApqCcWDVHhf%2BLeTrWeIT%2B%2FJcjzb1pQHZ488bKCgifknHr%2FxeI80YLSMXraVcwY8EEFWL6rBA2fhtmFfF9bhRIu00EZx51t3FnXJFV852oDTUi3oMhZyqhauHaATaTE3rb35cW7k67NbdtaU4TSR9wlweVnabvIbSJq6MxPzMJNAmyPA3HLMFvt45GMnLbo%2FEyxP0qdLLpW414egeNaCyJS%2B%2BfJRl9CJ4HTlObSLcbj2HIfKdbBRBdwPKUMaQDGkA6nglTWf16LN%2FIhWUfPY7K2OKqkISBC8aB30mZBZ%2FV6yydU%2FaFhhoHwNHopTyqWdjRiPI%2F2%2Fbcz%2FpPY382DeuRZpBkne8KQzDp5p74KMFI2%2B9uxq3bP8988NgjLhaioYFW23wPBbnzUlBujAs%2BPqFFc9NSkK1teVza1Fi825AzMczp9lOMLT7CynqZCwyWThswUf9V6Hgdc64rrZuPpwY3vmTutkwdvOWzVnepdoD7ps3x9jVkhF%2FM81vmWSqx%2B1KqatEXkiWeXfN2Ltz%2BALblwYEu2%2FhhYmnRNFohXnlypXRgCZSHpmMuDlwRQ28cGvATGoCz3eMIx3TKjhMWYwow1tf7zwY6pgEUdtpj7RJL8Vjc7Bj6e48yeFcsckea7ugGOuhfaFVuC%2FV9QB4C9hkUO%2BgzTXor6%2FiQ1QmcXQ5fCL%2F74zMFrGwt9wzep7e9Ngw4A6GW5Ucr2XAGQSDR8SBZMjj%2BPS52aGvpWFGtRDg8Y44SlkWsUWiBl%2FeAfTXW%2BZrVI%2FoenX1GFkKaRZPvokvQJfSuriCORcXuS90%2FeijEA26LZ33AIYCAuCL33dKG&X-Amz-Signature=0c05d4fce73d180725f37d87b4707112ef557cc368718d55c16f54ca97f1e5d2&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

외부 IPv4 주소 옵션 설명

- 임시: 외부 IP 자동 할당, VM 재시작 시 변경됨 (Step 1에서 사용)

- 없음: 외부 IP 미할당 → 인터넷 직접 접근 불가 → IAP로만 접속 가능

- 고정: 재시작해도 IP 유지 (실서비스용, 별도 비용)

### 생성 결과 확인

VM 목록에서 inkyu-private-vm의 외부 IP 칸이 비어있으면 정상

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/ba296437-38e8-4b66-aa5a-48a48b77be1c/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RC4EYJBY%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084703Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJGMEQCIBD9o87d%2B9JKIho9n3o3ilAcGRbtBWl1DAb5C9YSNFGlAiBoMd4xoFjhy4QVRneJLgH0F3alzbuShmkztI8smP2pBSqIBAjh%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDYzNzQyMzE4MzgwNSIMd%2FBo9lUJsUFqm5P2KtwDwSqnCMRln45VgWmOVrStC%2BZkwfJiLujAY6CXQ6iApqCcWDVHhf%2BLeTrWeIT%2B%2FJcjzb1pQHZ488bKCgifknHr%2FxeI80YLSMXraVcwY8EEFWL6rBA2fhtmFfF9bhRIu00EZx51t3FnXJFV852oDTUi3oMhZyqhauHaATaTE3rb35cW7k67NbdtaU4TSR9wlweVnabvIbSJq6MxPzMJNAmyPA3HLMFvt45GMnLbo%2FEyxP0qdLLpW414egeNaCyJS%2B%2BfJRl9CJ4HTlObSLcbj2HIfKdbBRBdwPKUMaQDGkA6nglTWf16LN%2FIhWUfPY7K2OKqkISBC8aB30mZBZ%2FV6yydU%2FaFhhoHwNHopTyqWdjRiPI%2F2%2Fbcz%2FpPY382DeuRZpBkne8KQzDp5p74KMFI2%2B9uxq3bP8988NgjLhaioYFW23wPBbnzUlBujAs%2BPqFFc9NSkK1teVza1Fi825AzMczp9lOMLT7CynqZCwyWThswUf9V6Hgdc64rrZuPpwY3vmTutkwdvOWzVnepdoD7ps3x9jVkhF%2FM81vmWSqx%2B1KqatEXkiWeXfN2Ltz%2BALblwYEu2%2FhhYmnRNFohXnlypXRgCZSHpmMuDlwRQ28cGvATGoCz3eMIx3TKjhMWYwow1tf7zwY6pgEUdtpj7RJL8Vjc7Bj6e48yeFcsckea7ugGOuhfaFVuC%2FV9QB4C9hkUO%2BgzTXor6%2FiQ1QmcXQ5fCL%2F74zMFrGwt9wzep7e9Ngw4A6GW5Ucr2XAGQSDR8SBZMjj%2BPS52aGvpWFGtRDg8Y44SlkWsUWiBl%2FeAfTXW%2BZrVI%2FoenX1GFkKaRZPvokvQJfSuriCORcXuS90%2FeijEA26LZ33AIYCAuCL33dKG&X-Amz-Signature=67ac6abee15679a1f5b4314560c2f0c80ef877a1594de33636b1722a6192d903&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

---

## Step 2-3 : 방화벽 규칙 추가

VPC 네트워크 → 방화벽 → 방화벽 규칙 만들기

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/36cb9f14-7a7d-416d-b0a9-8ef1e739b6a4/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RC4EYJBY%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084703Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJGMEQCIBD9o87d%2B9JKIho9n3o3ilAcGRbtBWl1DAb5C9YSNFGlAiBoMd4xoFjhy4QVRneJLgH0F3alzbuShmkztI8smP2pBSqIBAjh%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDYzNzQyMzE4MzgwNSIMd%2FBo9lUJsUFqm5P2KtwDwSqnCMRln45VgWmOVrStC%2BZkwfJiLujAY6CXQ6iApqCcWDVHhf%2BLeTrWeIT%2B%2FJcjzb1pQHZ488bKCgifknHr%2FxeI80YLSMXraVcwY8EEFWL6rBA2fhtmFfF9bhRIu00EZx51t3FnXJFV852oDTUi3oMhZyqhauHaATaTE3rb35cW7k67NbdtaU4TSR9wlweVnabvIbSJq6MxPzMJNAmyPA3HLMFvt45GMnLbo%2FEyxP0qdLLpW414egeNaCyJS%2B%2BfJRl9CJ4HTlObSLcbj2HIfKdbBRBdwPKUMaQDGkA6nglTWf16LN%2FIhWUfPY7K2OKqkISBC8aB30mZBZ%2FV6yydU%2FaFhhoHwNHopTyqWdjRiPI%2F2%2Fbcz%2FpPY382DeuRZpBkne8KQzDp5p74KMFI2%2B9uxq3bP8988NgjLhaioYFW23wPBbnzUlBujAs%2BPqFFc9NSkK1teVza1Fi825AzMczp9lOMLT7CynqZCwyWThswUf9V6Hgdc64rrZuPpwY3vmTutkwdvOWzVnepdoD7ps3x9jVkhF%2FM81vmWSqx%2B1KqatEXkiWeXfN2Ltz%2BALblwYEu2%2FhhYmnRNFohXnlypXRgCZSHpmMuDlwRQ28cGvATGoCz3eMIx3TKjhMWYwow1tf7zwY6pgEUdtpj7RJL8Vjc7Bj6e48yeFcsckea7ugGOuhfaFVuC%2FV9QB4C9hkUO%2BgzTXor6%2FiQ1QmcXQ5fCL%2F74zMFrGwt9wzep7e9Ngw4A6GW5Ucr2XAGQSDR8SBZMjj%2BPS52aGvpWFGtRDg8Y44SlkWsUWiBl%2FeAfTXW%2BZrVI%2FoenX1GFkKaRZPvokvQJfSuriCORcXuS90%2FeijEA26LZ33AIYCAuCL33dKG&X-Amz-Signature=f772408bd70591269d22d93376539a20708815b8aa726dbdf2dafbd9614c6ab4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/f78c4cef-6e71-4d54-8b03-7db295f2de03/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RC4EYJBY%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084703Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJGMEQCIBD9o87d%2B9JKIho9n3o3ilAcGRbtBWl1DAb5C9YSNFGlAiBoMd4xoFjhy4QVRneJLgH0F3alzbuShmkztI8smP2pBSqIBAjh%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDYzNzQyMzE4MzgwNSIMd%2FBo9lUJsUFqm5P2KtwDwSqnCMRln45VgWmOVrStC%2BZkwfJiLujAY6CXQ6iApqCcWDVHhf%2BLeTrWeIT%2B%2FJcjzb1pQHZ488bKCgifknHr%2FxeI80YLSMXraVcwY8EEFWL6rBA2fhtmFfF9bhRIu00EZx51t3FnXJFV852oDTUi3oMhZyqhauHaATaTE3rb35cW7k67NbdtaU4TSR9wlweVnabvIbSJq6MxPzMJNAmyPA3HLMFvt45GMnLbo%2FEyxP0qdLLpW414egeNaCyJS%2B%2BfJRl9CJ4HTlObSLcbj2HIfKdbBRBdwPKUMaQDGkA6nglTWf16LN%2FIhWUfPY7K2OKqkISBC8aB30mZBZ%2FV6yydU%2FaFhhoHwNHopTyqWdjRiPI%2F2%2Fbcz%2FpPY382DeuRZpBkne8KQzDp5p74KMFI2%2B9uxq3bP8988NgjLhaioYFW23wPBbnzUlBujAs%2BPqFFc9NSkK1teVza1Fi825AzMczp9lOMLT7CynqZCwyWThswUf9V6Hgdc64rrZuPpwY3vmTutkwdvOWzVnepdoD7ps3x9jVkhF%2FM81vmWSqx%2B1KqatEXkiWeXfN2Ltz%2BALblwYEu2%2FhhYmnRNFohXnlypXRgCZSHpmMuDlwRQ28cGvATGoCz3eMIx3TKjhMWYwow1tf7zwY6pgEUdtpj7RJL8Vjc7Bj6e48yeFcsckea7ugGOuhfaFVuC%2FV9QB4C9hkUO%2BgzTXor6%2FiQ1QmcXQ5fCL%2F74zMFrGwt9wzep7e9Ngw4A6GW5Ucr2XAGQSDR8SBZMjj%2BPS52aGvpWFGtRDg8Y44SlkWsUWiBl%2FeAfTXW%2BZrVI%2FoenX1GFkKaRZPvokvQJfSuriCORcXuS90%2FeijEA26LZ33AIYCAuCL33dKG&X-Amz-Signature=800bb724c8a4b0a1d919b7ed26ee0f0caf8006cd842abfbd2f8d50409c80190d&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

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

![image](https://prod-files-secure.s3.us-west-2.amazonaws.com/d71fcafe-5655-4875-8ad0-d7c49c7693a1/9d063f67-8bd9-4046-8faf-7326b5c5a432/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAZI2LB466RC4EYJBY%2F20260509%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20260509T084704Z&X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLXdlc3QtMiJGMEQCIBD9o87d%2B9JKIho9n3o3ilAcGRbtBWl1DAb5C9YSNFGlAiBoMd4xoFjhy4QVRneJLgH0F3alzbuShmkztI8smP2pBSqIBAjh%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDYzNzQyMzE4MzgwNSIMd%2FBo9lUJsUFqm5P2KtwDwSqnCMRln45VgWmOVrStC%2BZkwfJiLujAY6CXQ6iApqCcWDVHhf%2BLeTrWeIT%2B%2FJcjzb1pQHZ488bKCgifknHr%2FxeI80YLSMXraVcwY8EEFWL6rBA2fhtmFfF9bhRIu00EZx51t3FnXJFV852oDTUi3oMhZyqhauHaATaTE3rb35cW7k67NbdtaU4TSR9wlweVnabvIbSJq6MxPzMJNAmyPA3HLMFvt45GMnLbo%2FEyxP0qdLLpW414egeNaCyJS%2B%2BfJRl9CJ4HTlObSLcbj2HIfKdbBRBdwPKUMaQDGkA6nglTWf16LN%2FIhWUfPY7K2OKqkISBC8aB30mZBZ%2FV6yydU%2FaFhhoHwNHopTyqWdjRiPI%2F2%2Fbcz%2FpPY382DeuRZpBkne8KQzDp5p74KMFI2%2B9uxq3bP8988NgjLhaioYFW23wPBbnzUlBujAs%2BPqFFc9NSkK1teVza1Fi825AzMczp9lOMLT7CynqZCwyWThswUf9V6Hgdc64rrZuPpwY3vmTutkwdvOWzVnepdoD7ps3x9jVkhF%2FM81vmWSqx%2B1KqatEXkiWeXfN2Ltz%2BALblwYEu2%2FhhYmnRNFohXnlypXRgCZSHpmMuDlwRQ28cGvATGoCz3eMIx3TKjhMWYwow1tf7zwY6pgEUdtpj7RJL8Vjc7Bj6e48yeFcsckea7ugGOuhfaFVuC%2FV9QB4C9hkUO%2BgzTXor6%2FiQ1QmcXQ5fCL%2F74zMFrGwt9wzep7e9Ngw4A6GW5Ucr2XAGQSDR8SBZMjj%2BPS52aGvpWFGtRDg8Y44SlkWsUWiBl%2FeAfTXW%2BZrVI%2FoenX1GFkKaRZPvokvQJfSuriCORcXuS90%2FeijEA26LZ33AIYCAuCL33dKG&X-Amz-Signature=c50dcf38ba68a33d7bb0ed858c5f16cd0c669902e116ad535b5fa00f3a0b9cba&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)

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
