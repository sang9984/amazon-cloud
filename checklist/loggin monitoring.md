# 04. 로깅 및 모니터링

> CloudTrail을 통한 활동 추적 및 보안 모니터링 설정. 침해 사고 발생 시 추적의 핵심 도구.

---

## 1. CloudTrail 개요

### 1-1. CloudTrail이란

AWS 계정에서 일어나는 모든 API 호출을 기록하는 서비스.

기록되는 활동:

- 콘솔 로그인
- 서비스 생성/수정/삭제
- IAM 사용자 추가
- 자격증명 사용
- 모든 AWS API 호출

### 1-2. CloudTrail vs Trail

|용어|의미|
|---|---|
|**CloudTrail**|활동 로깅 서비스 전체|
|**Trail**|"이런 활동을 이렇게 기록하라"는 구체적 설정|

비유: CCTV 시스템(CloudTrail) vs 개별 카메라 설정(Trail)

### 1-3. 보안 학습 관점에서의 중요성

CloudTrail은 침해사고 분석의 핵심 도구:

1. 본인 키가 유출됐을 때 누가 뭘 했는지 추적
2. 의심스러운 활동 감지
3. 사고 원인 파악
4. 컴플라이언스 증거

실제 보안 엔지니어가 매일 보는 도구.

---

## 2. Event History (기본 로그)

### 2-1. 자동 활성화

CloudTrail은 사실 별도 설정 없이도 **기본 활동 로그를 90일간 자동 보관**.

이게 **Event History**.

### 2-2. 경로

```
CloudTrail 콘솔
  → 좌측 메뉴 "Event history" 클릭
```

### 2-3. 확인할 것

본인이 한 모든 작업이 시간순으로 표시됨:

- 콘솔 로그인 시간/IP
- S3 버킷 생성/삭제
- IAM 사용자 생성
- 키 발급
- 모든 API 호출

### 2-4. Event History의 특징

|항목|내용|
|---|---|
|보관 기간|90일|
|비용|무료|
|조회 위치|콘솔에서만|
|자동 활성화|신규 계정에서 자동|

**학습 단계에선 이것만으로 충분**.

### 2-5. 활용 예시

본인이 학습 후 일주일 뒤 "내가 무엇을 했더라?" 궁금할 때:

```
CloudTrail → Event history → 날짜 필터 → 본인 활동 조회
```

또는 의심스러운 활동 점검:

```
Event source: signin.amazonaws.com
  → 로그인 시도 내역 확인
  → 본인이 아닌 시간/장소에서 로그인 시도가 있는지 점검
```

---

## 3. Trails (별도 추적 설정)

### 3-1. Trail이란

Event History와 별도로, **장기 보관 및 분석용**으로 만드는 추적 설정.

### 3-2. Event History와의 차이

|항목|Event History|Trail|
|---|---|---|
|보관 기간|90일|무제한 (S3에 저장)|
|조회 방식|콘솔에서만|S3 파일 다운로드, 분석 도구 연동|
|비용|무료|첫 Trail 무료, 추가는 유료|
|이벤트 종류|관리 이벤트만|데이터 이벤트도 포함 가능|

### 3-3. Trail 생성 (선택)

학습 단계에선 굳이 안 만들어도 됨. 만들고 싶다면:

#### 경로

```
CloudTrail → Trails → Create trail
```

#### 설정

```
Trail name: learning-trail
Storage location: 새 S3 버킷 생성 (자동 추천)
Log file validation: 활성화 (권장)
SNS notification: 비활성화 (학습 단계)
CloudWatch Logs: 비활성화 (학습 단계)

Event type:
  ☑ Management events: Read 및 Write 둘 다
  ☐ Data events: 비활성화 (비용 발생)
  ☐ Insights events: 비활성화
```

### 3-4. Trail 만들 때 주의사항

- 로그가 S3에 쌓이므로 저장 비용 약간 발생 (학습 단위로는 매우 미미)
- 첫 번째 Trail은 관리 이벤트만 기록 시 무료
- 데이터 이벤트(S3 객체 작업 등)는 별도 과금

---

## 4. 본인 활동 점검 예시

### 4-1. 콘솔 로그인 기록 확인

```
CloudTrail → Event history
  → Filter: Event name = ConsoleLogin
  → 시간별 로그인 시도 확인
```

본인이 로그인하지 않은 시간에 시도가 있다면 의심.

### 4-2. IAM 변경 기록 확인

```
CloudTrail → Event history
  → Filter: Event source = iam.amazonaws.com
  → IAM 관련 모든 변경 사항
```

누군가 권한을 변경했거나 사용자를 추가했는지 확인.

### 4-3. S3 작업 기록

```
CloudTrail → Event history
  → Filter: Event source = s3.amazonaws.com
  → S3 버킷 생성/삭제, 정책 변경 등
```

---

## 5. 알람 연동 (고급, 선택)

### 5-1. CloudWatch와 연동

CloudTrail 로그를 CloudWatch Logs로 보내면 실시간 알람 가능.

예시 시나리오:

- 루트 계정 로그인 → 즉시 알람
- 액세스 키 생성 → 즉시 알람
- 보안 그룹 변경 → 즉시 알람

학습 단계에선 과한 설정이지만, 실무에선 필수.

### 5-2. GuardDuty (선택)

AWS의 머신러닝 기반 위협 탐지 서비스.

```
GuardDuty 콘솔 → Enable
```

- 30일 무료 체험
- 이후 사용량에 따라 과금 (작은 계정은 월 $3~5)
- 비정상 활동 자동 탐지

**학습 단계에선 활성화 안 권장** (비용 발생).

---

## 6. 즐겨찾기 페이지

```
CloudTrail Event History
  https://console.aws.amazon.com/cloudtrail/home#/events

CloudTrail Trails
  https://console.aws.amazon.com/cloudtrail/home#/trails
```

매주 한 번 Event history 둘러보는 습관 권장.

---

## 7. 체크리스트

본 문서의 모든 항목 완료 확인:

### 필수

- [ ] CloudTrail → Event history 페이지 위치 확인
- [ ] 본인 최근 활동이 기록되는지 확인 (S3 작업, 로그인 등)

### 권장 (학습 단계엔 선택)

- [ ] Trail 생성 (장기 보관용)
- [ ] CloudWatch Logs 연동 검토

### 비권장 (학습 단계엔 불필요)

- GuardDuty 활성화 (비용 발생)
- Config 활성화 (비용 발생)
- 데이터 이벤트 추적 (비용 발생)

---

## 8. 보안 사고 발생 시 활용

만약 키 유출이 의심되거나 비정상 청구가 발생한다면:

### 단계 1: Event history 점검

```
CloudTrail → Event history
  → 시간 범위: 의심 기간
  → 본인이 한 일이 아닌 활동 찾기
```

### 단계 2: 원본 IP 확인

각 이벤트 클릭하면 호출 IP가 보임:

- 본인 IP가 아니면 외부 침투 가능성
- 알 수 없는 IP면 즉시 키 비활성화

### 단계 3: 어떤 자원이 만들어졌는지 추적

```
의심 기간 내:
  RunInstances 이벤트가 있나? → EC2 인스턴스 생성됨
  CreateBucket 이벤트가 있나? → S3 버킷 생성됨
  AttachUserPolicy 이벤트가 있나? → 권한 변경됨
```

발견된 자원을 즉시 삭제.

### 단계 4: 보고서 작성

AWS Support에 신고할 때 CloudTrail 로그 첨부:

- 사건 발생 시간
- 의심 IP
- 만들어진 자원 목록
- 본인이 한 일이 아니라는 증거

CloudTrail이 가장 강력한 증거 자료.