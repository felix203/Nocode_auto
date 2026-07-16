## 0. 과제 개요

Webhook으로 요청을 받아 status 값에 따라 분기 처리하는 동일한 자동화 워크플로우를
Make와 n8n 두 도구로 각각 구현하고, 사용 경험을 비교 분석했다.

---

## 1. 프로젝트 1 — 자동화 도구 비교 구현

### 1-1. 구현한 워크플로우 개요

Webhook 요청 수신 → status 값 조건 분기 → Google Sheets 기록 + Gmail 긴급 알림 발송

---

### 1-2. 사용한 도구 목록

- 도구 1: Make
- 도구 2: n8n

---

### 1-3. 도구별 워크플로우 구성

#### 🔧 도구 1: `Make`

| 단계 | 종류 | 내가 설정한 내용 |
|------|------|----------------|
| 1 | Trigger | Webhook 모듈로 POST 요청 수신 (필드: name, status, message) |
| 2 | 조건 분기 (Router) | Router로 `status = urgent` 경로와 그 외(일반) 경로 분기 |
| 3 | Action 1 | Google Sheets — Add a Row로 name/status/message 기록 |
| 4 | Action 2 | Gmail — urgent 경로에서 긴급 알림 메일 발송 |

#### 🔧 도구 2: `n8n`

| 단계 | 종류 | 내가 설정한 내용 |
|------|------|----------------|
| 1 | Trigger | Webhook 노드로 POST 요청 수신 (필드: name, status, message) |
| 2 | 조건 분기 (IF) | IF 노드로 `{{ $json.status }} == urgent` True/False 분기 |
| 3 | Action 1 | Google Sheets — Append Row로 name/status/message 기록 |
| 4 | Action 2 | Gmail — True 경로에서 긴급 알림 메일 발송 |

---

<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/e43f9d4f-2c7c-438c-a933-0d0c08209cc2" />

<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/591502bd-3903-4f6f-956b-743cb5ae6f06" />

---

### 1-6. 비교 분석 보고서

#### 구현 과정 요약

- 도구 1 ( Make ): Webhook 모듈 생성 → Router 추가 → 두 경로에 Filter 조건 설정
  → Google Sheets 모듈 드래그로 필드 매핑 → urgent 경로에 Gmail 모듈 연결 → 실행 테스트
- 도구 2 ( n8n ): Webhook 노드 생성 → IF 노드로 조건식 입력 → True/False 각 경로에
  Google Sheets 노드 연결 → Credential 인증 → Gmail 노드 연결 → 실행 테스트

---
<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/c521b9f1-7211-4784-81f5-a2c33aebf38a" />

<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/76c2272d-1a22-42b3-9d43-40dbf2a2b80e" />

<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/924f4399-be4c-4f60-9e46-5711405091e7" />

<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/6b0406fe-1b44-47d1-b818-2f0c89ae5563" />
---

#### 비교 항목표 (5개 이상)

| 비교 항목 | 도구 1: `Make` | 도구 2: `n8n` |
|-----------|----------------|---------------|
| UI/UX | 시각적 원형 모듈, 직관적 | 노드 그래프형, 개발자 친화적 |
| 설정 난이도 | 낮음 (드래그 매핑) | 중간 (Expression 문법 필요) |
| 연동 서비스 범위 | 1,500+ 앱 내장 | 400+ 노드 + 커스텀 확장 |
| 무료 플랜 범위 | 월 1,000 ops, 시나리오 2개 | 자체 호스팅 시 무제한 |
| 실행 로그 확인 방식 | History 탭에서 시각적 확인 | Executions 패널에서 노드별 확인 |
| 데이터 매핑 방식 | 드래그 앤 드롭 | `{{ $json.필드명 }}` 표현식 |

#### 각 도구의 장단점 정리

**도구 1: `Make`**
```
장점:
- UI가 직관적이라 입문자도 빠르게 구현 가능
- 드래그 앤 드롭 매핑으로 데이터 연결이 쉬움
- 다양한 앱이 기본 내장되어 별도 설정 부담 적음

단점:
- 무료 플랜의 월 1,000 ops / 시나리오 2개 제한이 빡빡함
- 복잡한 로직 처리 시 커스터마이징 자유도가 낮음
```

**도구 2: `n8n`**
```
장점:
- 자체 호스팅 시 실행 횟수 무제한, 비용 부담 없음
- Expression·코드 노드로 복잡한 로직 구현 자유도 높음
- 오픈소스라 확장성과 데이터 통제권이 우수

단점:
- Expression 문법 학습이 필요해 초기 진입장벽이 있음
- Credential 인증 설정이 Make보다 번거로움
```

#### 어떤 상황에서 어떤 도구가 적합한가?

```
- 빠른 프로토타입 제작이나 비개발자 팀에서는 Make가 적합하다.
  직관적 UI 덕분에 학습 없이 바로 워크플로우를 만들 수 있기 때문이다.
- 실행량이 많거나 장기 운영, 복잡한 커스텀 로직이 필요한 경우에는
  n8n이 적합하다. 자체 호스팅으로 비용 제약이 없고 확장성이 높기 때문이다.
```



# 프로젝트 2: 일일 할일 마감 리마인더

## 1. 자동화 개요
Google Sheets에 등록된 할 일 목록을 읽어와,
마감일이 임박(D-1 이하)한 항목만 골라 Gmail로 알림을 보내는 자동화.

## 2. 사용 도구
- Make (Integromat)
- Google Sheets
- Gmail

## 3. 트리거 & 액션
- 트리거 : (수동 실행)
- 액션 1 : Google Sheets - Search Rows (할일 목록 조회)
- 액션 2 : Router - 마감일 D-1 조건 분기
- 액션 3 : Gmail - Send an Email (임박 항목 알림 발송)

## 4. 워크플로우 흐름
Search Rows → Router → [임박 경로] → Gmail 발송
<div>
<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/e74dfda2-0ab9-433e-ae2a-f1070d31cefd" />

<img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/1dd71114-ae7b-467f-9d63-5dc22a67a6f9" />
</div>
## 5. 핵심 로직
- parseDate(마감일; YYYY-MM-DD) 로 텍스트→날짜 변환
- addDays(now; 1) = 내일 기준
- 마감일 ≤ 내일 → 임박 알림 발송
- 번들(Bundle) 단위로 여러 할일 자동 반복 처리

## 6. 테스트 결과
- 할일 4개 중 임박 2개만 메일 발송 확인
- 마감 여유 항목은 발송 제외 (분기 정상 작동)

## 7. 배운 점 / 어려웠던 점
- 텍스트 날짜를 parseDate로 변환하는 과정이 핵심 난관이었음
- 번들 개념(목록 일괄 반복 처리)을 이해하게 됨
