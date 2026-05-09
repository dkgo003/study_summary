---
title: "함수형 프로그래밍의 Currying 개념 정리"
date: "2026-05-09"
draft: false
slug: "함수형-프로그래밍의-currying-개념-정리"
notion_status: "Published"
notion_page_id: "35b1d13b-f2d8-806e-b23a-f9dca98cc6eb"
categories:
  - "기술"
tags: []
---

Currying은 여러 인자를 받는 함수를 인자 하나를 받는 함수들의 연쇄로 바꾸는 함수형 프로그래밍 기법이다. 처음에는 낯설지만, 특정 인자를 미리 고정해 재사용 가능한 함수를 만들 때 유용하다.

## Currying이란

- 일반 함수는 f(a, b, c)처럼 여러 인자를 한 번에 받는다.

- Currying된 함수는 f(a)(b)(c)처럼 인자를 하나씩 받아 다음 함수를 반환한다.

- 이를 통해 일부 인자를 먼저 고정하고, 나머지 인자는 나중에 전달하는 구조를 만들 수 있다.

## 장점

- 특정 설정을 미리 주입한 함수를 만들어 코드 재사용성을 높일 수 있다.

- 작은 함수를 조합해 복잡한 처리를 구성하기 쉽다.

- 이벤트 핸들러, 콜백, 데이터 처리 파이프라인에서 인자를 단계적으로 고정하는 패턴에 활용할 수 있다.

## 주의할 점

- 함수 호출이 여러 단계로 나뉘기 때문에 익숙하지 않은 사람에게는 가독성이 떨어질 수 있다.

- 클로저가 많이 생기면 디버깅 흐름이 복잡해질 수 있다.

- Python에서는 순수한 currying보다 functools.partial이나 클로저를 활용한 부분 적용이 더 자연스러운 경우가 많다.

## Python 예시

```python

def multiply(a):

def inner(b):

return a * b

return inner

double = multiply(2)

print(double(10))  # 20

```

## 정리

Currying은 모든 코드에 적용해야 하는 필수 패턴은 아니다. 다만 인자를 단계적으로 고정하거나 함수 조합을 많이 사용하는 코드에서는 재사용성과 표현력을 높여줄 수 있다.
