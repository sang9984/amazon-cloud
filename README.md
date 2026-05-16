# AWS 학습 환경 점검 체크리스트

> AWS 학습 시작 시 콘솔에서 점검해야 할 보안 및 비용 관련 설정 정리. 한 번 설정하면 평생 보호장치로 작동.

---

## 점검의 목적

AWS 학습 시 다음 두 가지 사고를 방지:

1. **비용 사고**: 실수로 큰 비용 청구 (켜놓은 자원, 키 유출, 트래픽 폭증)
2. **보안 사고**: 키 유출, 권한 남용, 자원 탈취

본 체크리스트를 한 번 따라하면 99%의 사고 시나리오를 차단.

---

## 문서 구성

본 체크리스트는 5개의 문서로 구성:

|번호|문서|내용|
|---|---|---|
|1|`01-billing-alerts.md`|결제 알람 및 Billing 설정|
|2|`02-account-security.md`|루트 계정 + IAM 사용자 보안|
|3|`03-resource-audit.md`|활성 자원 점검 (서비스별)|
|4|`04-logging-monitoring.md`|CloudTrail 및 활동 로깅|
|5|`05-ongoing-maintenance.md`|정기 점검 + 사고 대응|

순서대로 따라하면 약 30분 안에 완료.

---

## 학습 단계별 위험도

본인의 학습 진행에 따라 위험도가 달라짐:

### 단계 1: S3/IAM 학습 (시작 단계)

**위험도: 매우 낮음**

- 마음껏 실험 가능
- 거의 무료
- 점검 빈도: 주 1회

### 단계 2: EC2 학습

**위험도: 중간**

- t2.micro만 사용 (Free Tier)
- 작업 후 반드시 stop 또는 terminate
- 점검 빈도: 매일

### 단계 3: RDS/Lambda 학습

**위험도: 중간**

- Free Tier 범위 안에서
- 점검 빈도: 매일

### 단계 4: 실제 서비스 운영

**위험도: 높음**

- 트래픽에 따라 비용 변동
- 점검 빈도: 매일 + 실시간 모니터링

---

## 한 줄 요약

안전한 학습을 위한 핵심 4가지:

1. **결제 알람** ($1, $5, $10, Zero spend)
2. **MFA 활성화** (루트 + IAM 사용자)
3. **최소 권한 원칙** (필요한 권한만)
4. **사용 후 자원 정리 습관**

이 네 가지가 갖춰지면 99%의 사고는 일어나지 않음.

---

## 즐겨찾기 권장 페이지

매일 한 번 확인할 페이지를 브라우저 북마크:

```
1. Billing Dashboard
   https://console.aws.amazon.com/billing/home

2. Cost Explorer
   https://console.aws.amazon.com/cost-management/home

3. Budgets
   https://console.aws.amazon.com/billing/home#/budgets

4. EC2 Instances
   https://console.aws.amazon.com/ec2/v2/home#Instances

5. S3 Buckets
   https://s3.console.aws.amazon.com/s3/buckets

6. IAM Users
   https://console.aws.amazon.com/iam/home#/users

7. CloudTrail Event History
   https://console.aws.amazon.com/cloudtrail/home#/events
```

매일 빌링 대시보드 한 번 확인하는 습관이면 충분.

---

## Reference

- AWS 비용 관리 모범 사례: https://docs.aws.amazon.com/cost-management/
- AWS Budgets 가이드: https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html
- AWS Free Tier: https://aws.amazon.com/free/
- IAM 보안 모범 사례: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- AWS CloudTrail: https://docs.aws.amazon.com/awscloudtrail/