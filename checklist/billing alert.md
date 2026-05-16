# 01. 결제 알람 및 Billing 설정

> AWS 비용 사고를 가장 빠르게 감지하기 위한 알람 설정.

---

## 1. Zero Spend Budget 생성 (가장 강력한 안전망)

1센트라도 청구되는 순간 메일 알림. 학습 환경에서 가장 빠른 경보 시스템.

### 경로

```
콘솔 우측 상단 계정명 클릭
  → Billing and Cost Management
  → 좌측 메뉴 Budgets
  → Create budget
  → "Use a template (simplified)" 선택
  → "Zero spend budget" 템플릿 선택
  → 본인 이메일 입력
  → Create budget
```

### 효과

- 무료 한도를 1센트라도 초과하면 즉시 메일
- 학습 단계에서 비용 발생 시 가장 빠른 경보
- 한 번 만들면 평생 작동

---

## 2. 단계별 Budget 알람 (권장)

비용 규모별로 알람을 두면 점진적 경보 가능.

### $1 알람 (조기 경보)

```
Budgets → Create budget
  → Cost budget
  → Amount: $1
  → Threshold: 100%
  → 이메일 입력
```

### $5 알람 (위험 경보)

```
같은 방식으로 Amount: $5
```

### $10 알람 (긴급)

```
같은 방식으로 Amount: $10
```

세 단계로 두면 작은 사용량부터 큰 사용량까지 단계별로 인지 가능.

---

## 3. Billing Preferences 활성화

비용 관련 자동 알림과 보고서 수신 설정.

### 경로

```
Billing and Cost Management
  → 좌측 메뉴 Billing preferences
```

### 활성화할 세 항목

|항목|기능|
|---|---|
|`Receive AWS Free Tier alerts`|무료 한도 85% 사용 시 자동 메일|
|`Receive Billing Alerts`|CloudWatch 결제 알람 활성화|
|`Receive PDF invoice by email`|매월 청구서 PDF로 자동 발송|

### 설정 방법

1. 세 체크박스 모두 체크
2. 이메일 주소 확인
3. Save preferences

세 가지 모두 무료이며 학습자 안전망 역할.

---

## 4. AWS 알림 메일 수신 확인

본인 이메일 받은편지함에서 다음 메일들 수신 확인:

- "Welcome to AWS"
- "Your AWS Free Tier" 안내 메일
- (Budget 생성 시) Budget 확인 메일

스팸 폴더도 확인. AWS 메일이 안 오면 알람도 안 옴.

---

## 5. Cost Explorer 활용

비용 추세 분석 도구. 매주 한 번 확인 권장.

### 경로

```
Billing and Cost Management
  → Cost Explorer
```

### 확인할 것

- 일별/월별 비용 그래프
- 서비스별 비용 분포
- 갑자기 튀는 비용이 있는지

### 추정치(Forecast) 안 보이는 경우

- 가입 직후엔 데이터 부족으로 추정치 표시 안 됨
- 보통 1~2주 후 안정적인 추정치 표시
- 정상이며, 다른 알람들이 안전망 역할

---

## 체크리스트

본 문서의 모든 항목 완료 확인:

- [ ] Zero Spend Budget 생성
- [ ] $1 Budget 알람 생성
- [ ] $5 Budget 알람 생성 (선택)
- [ ] $10 Budget 알람 생성 (선택)
- [ ] Receive AWS Free Tier alerts 활성화
- [ ] Receive Billing Alerts 활성화
- [ ] Receive PDF invoice by email 활성화
- [ ] AWS 환영 메일 수신 확인
- [ ] Cost Explorer 페이지 위치 확인

---

## 참고: 결제 알람의 적용 범위

**중요:** 결제 알람은 **AWS 계정 전체** 기준이지, IAM 사용자별이 아님.

- 본인이 어떤 IAM 사용자로 작업하든 모든 비용이 합산
- Free Tier 한도도 계정 전체에서 공유
- 청구서도 계정 단위로 발송

→ 본 알람들이 본인 전체 클라우드의 안전망 역할.