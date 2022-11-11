---
title: "[요약] Transaction outbox"
date: 2022-11-12 01:06:00 +0900
categories: EDA
---

https://microservices.io/patterns/data/transactional-outbox.html 을 요악합니다.

# Problem

Event Producer가 DB Transaction을 수행하고, 이벤트를 발행하는 경우 아래 문제들을 직면할 수 있음

- 가짜 이벤트(False Event): Operation은 실패했으나 이벤트가 발행됨
- 이벤트 발행 실패: Operation은 성공했으나 이벤트 발행에 실패한 경우
- 이벤트 순서가 꼬임: T1, T2 트랜잭션이 연속적으로 발생할 때, T1 -> E1 처리 후 T2 -> E2가 되길 바람. 하지만 E2가 E1보다 먼저 발행될 수 있다

# Requirements

문제 상황을 해결하기 위해 아래 요구사항이 존재.

- 2PC를 사용하지 않아야 함
- DB Transacion이 성공하는 경우 이벤트가 발행되어야하고 실패한다면 발행되지 않아야한다
- 메세지는 Transaction이 성공한 순서로 발행돼야 한다

# Solution: Transactional outbox

![diagram](https://microservices.io/i/patterns/data/ReliablePublication.png)

Transactional outbox 패턴은 `outbox table` 테이블을 추가로 두어 이벤트를 기록하고(특정 레코드의 생성, 삭제, 수정 등과 같은 정보와 메세지 발행 여부)
Message Relay(주기적으로 outbox table을 읽어 메세지를 발행하는 워크로드)를 추가 배치하여 문제를 해결합니다.
이 때 목표 데이터와 outbox에 대한 operation은 transacion으로 처리돼야 합니다.

### 장점

- 2PC를 사용하지 않음
- transaction이 성공하면 이벤트의 발행이 보장됩니다
- 메세지가 순서대로 발행됩니다

### 단점

- 코드 수준에서 개발자가 outbox table 업데이트를 까먹기 쉽다.

### 이슈

- exactly once를 보장하지 않습니다: 이벤트 발행과 outbox table의 발행 여부의 업데이트는 여전히 원자적이지 않으므로 발생합니다.

# 정리

Transactional outbox는 발행 및 순서 보장을 구현하는 패턴으로 사용 가능하나 exactly once를 보장하진 않는다.
