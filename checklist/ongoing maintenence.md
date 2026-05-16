# 05. 정기 점검 및 사고 대응

> 초기 설정 후 지속적으로 유지해야 할 점검 루틴과 사고 발생 시 대응 매뉴얼.

---

## 1. 정기 점검 루틴

### 1-1. 매주 (5분)

- [ ] Cost Explorer 또는 Billing Dashboard 확인
- [ ] 갑자기 튀는 비용이 있는지 점검
- [ ] EC2/RDS 켜져있는 거 정리
- [ ] CloudTrail Event history 훑어보기 (의심 활동 확인)

### 1-2. 매월 (10분)

- [ ] PDF 청구서 받아서 확인
- [ ] 사용 안 하는 자원 정리
- [ ] IAM 사용자/키 정리 (안 쓰는 거 삭제)
- [ ] 액세스 키 로테이션 검토 (90일 이상 된 키)

### 1-3. 매 분기 (30분)

- [ ] 전체 리전 자원 점검 (Tag Editor 활용)
- [ ] IAM 정책 재검토 (필요한 권한만 남았나)
- [ ] 보안 그룹 규칙 검토
- [ ] 알람 동작 확인 (테스트)

---

## 2. 액세스 키 로테이션

### 2-1. 왜 필요한가

오래된 키는 유출 위험이 높음:

- 한 번 노출됐을지 모름
- 사용 패턴이 노출됐을 수 있음
- AWS 보안 모범 사례 권장

90일마다 새 키로 교체 권장.

### 2-2. 안전한 로테이션 절차

기존 키를 바로 삭제하지 말고, 순서대로 진행:

```
1. 새 액세스 키 발급 (기존과 별도)
   aws iam create-access-key --user-name [본인사용자명]

2. CLI 설정 업데이트
   aws configure
   → 새 키로 변경

3. 정상 작동 확인 (며칠 사용)
   aws sts get-caller-identity

4. 기존 키 비활성화 (삭제 아님)
   aws iam update-access-key \
     --access-key-id [OLD_KEY] \
     --status Inactive \
     --user-name [본인사용자명]

5. 며칠 동안 문제 없는지 확인

6. 기존 키 완전 삭제
   aws iam delete-access-key \
     --access-key-id [OLD_KEY] \
     --user-name [본인사용자명]
```

이 절차로 진행하면 키 교체 중 작업 중단 없음.

---

## 3. 자원 정리 습관

### 3-1. 학습 후 즉시 정리

본인이 학습으로 만든 모든 자원은 학습 종료 직후 삭제.

#### S3 자원 정리

```bash
# 버킷 내용 확인
aws s3 ls s3://[버킷이름]/

# 버킷 내 모든 파일 삭제
aws s3 rm s3://[버킷이름]/ --recursive

# 버킷 자체 삭제
aws s3 rb s3://[버킷이름]
```

#### EC2 자원 정리

```bash
# 인스턴스 종료 (완전 삭제)
aws ec2 terminate-instances --instance-ids i-xxxxx

# 또는 중지 (디스크는 유지, 비용 일부 절감)
aws ec2 stop-instances --instance-ids i-xxxxx

# 사용하지 않는 EIP 해제
aws ec2 release-address --allocation-id eipalloc-xxxxx
```

### 3-2. 정리 확인

```bash
# 전체 인스턴스 확인
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].[InstanceId,State.Name]' \
  --output table

# S3 버킷 확인
aws s3 ls

# 활성 키 확인
aws iam list-access-keys --user-name [본인사용자명]
```

---

## 4. 비용 사고 발생 시 대응

만약 예상치 못한 비용이 청구됐다면 다음 4단계를 순서대로.

### 4-1. 단계 1: 원인 파악

```
Cost Explorer
  → Service별 비용 분석
  → 어떤 서비스가 비용 발생시켰는지 확인
```

또는 CLI로:

```bash
aws ce get-cost-and-usage \
  --time-period Start=YYYY-MM-DD,End=YYYY-MM-DD \
  --granularity DAILY \
  --metrics UnblendedCost \
  --group-by Type=DIMENSION,Key=SERVICE
```

### 4-2. 단계 2: 즉시 자원 정리

원인이 된 자원을 즉시 삭제.

#### EC2가 원인

```bash
# 실행 중인 인스턴스 확인
aws ec2 describe-instances --region [의심리전]

# 즉시 종료
aws ec2 terminate-instances --instance-ids i-xxx --region [의심리전]
```

#### 키 유출 의심 시

```bash
# 본인 키 목록 확인
aws iam list-access-keys --user-name [본인사용자명]

# 의심 키 즉시 비활성화
aws iam update-access-key \
  --access-key-id AKIA... \
  --status Inactive \
  --user-name [본인사용자명]

# 그 다음 완전 삭제
aws iam delete-access-key \
  --access-key-id AKIA... \
  --user-name [본인사용자명]
```

#### 모르는 자원이 만들어진 경우

CloudTrail에서 어떤 자원이 생성됐는지 확인:

```
CloudTrail → Event history
  → 시간 범위: 비용 발생 시점
  → Event name: Create* 또는 Run* 등
```

발견된 자원을 모든 리전에서 삭제.

### 4-3. 단계 3: AWS Support 연락

```
https://console.aws.amazon.com/support/
  → "Account and billing"
  → 새 케이스 생성
```

작성 내용:

- 학습 의도였음을 명시
- 의도하지 않은 비용임을 설명
- 정리한 자원 목록
- CloudTrail 로그 첨부 (가능하면)

**첫 사고는 대부분 면제됨.** 정중하게 요청.

### 4-4. 단계 4: 재발 방지

- 어떤 자원이 사고를 일으켰는지 기록
- 본 체크리스트에 항목 추가
- 알람 강화
- 키 유출이 원인이라면 GitHub 등 노출 경로 차단

---

## 5. 키 유출 의심 시 긴급 대응

### 5-1. 즉시 (5분 이내)

```bash
# 모든 키 비활성화
aws iam list-access-keys --user-name [본인사용자명]

# 각 키 비활성화
aws iam update-access-key \
  --access-key-id AKIA... \
  --status Inactive \
  --user-name [본인사용자명]
```

콘솔에서도 같은 작업 가능.

### 5-2. 5분 이내 (병행)

CloudTrail에서 피해 범위 파악:

```
CloudTrail → Event history
  → 마지막 본인 작업 이후 활동 모두 검토
  → 본인 IP가 아닌 곳에서의 활동 식별
```

### 5-3. 30분 이내

- 만들어진 자원 모두 삭제 (모든 리전)
- 변경된 IAM 권한 원복
- 비밀번호 재설정
- 새 액세스 키 발급

### 5-4. 사후

- AWS Support에 사건 보고
- 재발 방지 조치 (.gitignore, Secret Scanning 등)
- 키 유출 경로 파악 (GitHub? 공용 PC?)

---

## 6. 안전 학습을 위한 황금률

### 6-1. 절대 하지 말 것

- ❌ 루트 계정으로 일상 작업
- ❌ AdministratorAccess 권한 IAM 사용자 사용
- ❌ 액세스 키 GitHub/공용 저장소에 커밋
- ❌ 큰 인스턴스(t2.micro 외) 켜놓고 잊기
- ❌ NAT Gateway 만들기 (학습 단계)
- ❌ MFA 없이 콘솔 사용

### 6-2. 항상 할 것

- ✅ MFA 활성화 (루트 + IAM 사용자)
- ✅ 최소 권한 원칙 (필요한 권한만)
- ✅ 사용 후 자원 즉시 정리
- ✅ 매주 비용 확인
- ✅ Budget 알람 유지
- ✅ 새 작업은 항상 학습용 IAM 사용자로

### 6-3. 학습 환경 안전 공식

```
MFA + 최소권한 + 알람 + 정리습관 = 안전한 학습환경
```

이 네 가지가 갖춰지면 99%의 사고는 일어나지 않음.

---

## 7. 체크리스트

본 문서의 항목 실행 확인:

### 매주 점검

- [ ] Cost Explorer 확인
- [ ] CloudTrail Event history 훑기
- [ ] 활성 자원 확인 (EC2, RDS)

### 매월 점검

- [ ] PDF 청구서 확인
- [ ] 안 쓰는 자원 정리
- [ ] 키 사용 현황 확인

### 매 분기 점검

- [ ] 전체 리전 자원 점검 (Tag Editor)
- [ ] IAM 권한 재검토
- [ ] 알람 동작 테스트
- [ ] 액세스 키 로테이션

### 사고 대응 준비

- [ ] AWS Support 연락 방법 숙지
- [ ] CloudTrail 사용법 숙지
- [ ] 키 비활성화 명령 숙지
- [ ] 비상 시 GitHub Secret Scanning 위치 숙지