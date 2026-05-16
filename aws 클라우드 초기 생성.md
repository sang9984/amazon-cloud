# AWS CLI Getting Started: 가입부터 첫 S3 실험까지

> 첫 AWS 학습 기록. Bandit으로 익힌 리눅스 사고방식을 클라우드에 처음 적용해본 과정. "클라우드는 결국 남의 컴퓨터에서 내 코드를 돌리는 것"이라는 직감을 실제로 확인한 단계.

---

## 0. 사전 준비

### 필요한 것

- 신용카드 (AWS 가입 시 필요, 무료 한도 안에서 청구 안 됨)
- 휴대폰 (인증 + MFA 앱 설치용)
- 이메일 계정

### 설치

```bash
# macOS
brew install awscli

# Ubuntu/Debian
sudo apt install awscli

# 또는 pip
pip install awscli
```

설치 확인:

```bash
aws --version
```

---

## 1. AWS 계정 생성

1. https://aws.amazon.com 접속
2. "AWS 계정 생성" 클릭
3. 이메일, 비밀번호, 계정 이름 입력
4. 신용카드 등록 (무료 한도 안에서는 청구 안 됨)
5. 휴대폰 인증
6. 기본 지원 플랜 선택 (무료)
7. 가입 완료 → 콘솔 로그인

**무료 한도(Free Tier):**

- 가입 후 6개월간 일부 서비스 무료 + $200 크레딧 (가입 시점 정책에 따라 다를 수 있음)
- S3: 5GB 저장, 월 20,000 GET 요청, 2,000 PUT 요청
- EC2: t2.micro 인스턴스 월 750시간
- 초과 시 자동 과금되므로 알람 설정 필수

---

## 2. 루트 계정 보안 강화 (가장 중요)

가입 시 만든 계정 = 루트 계정. 모든 권한을 가진 위험한 계정이므로 먼저 잠근다.

### MFA(다중 인증) 활성화

1. AWS 콘솔 우측 상단 본인 계정명 클릭 → **Security credentials**
2. "Multi-factor authentication (MFA)" → **Assign MFA device**
3. MFA 디바이스 이름 입력 (예: `[device-name]-iphone`, `root-authy`)
4. **Authenticator app** 선택
5. 휴대폰에 인증 앱 설치 (Google Authenticator, Authy, 1Password 등)
6. QR 코드 스캔
7. 연속된 두 개의 6자리 코드 입력 → 등록 완료

### 루트 계정에 액세스 키 만들지 않기

루트 계정용 액세스 키는 절대 만들지 않는다. 일상 작업은 IAM 사용자로 한다.

---

## 3. 결제 알람 설정 (사고 방지)

키가 유출되거나 자원을 켜놓고 잊어버리면 큰 청구가 발생할 수 있다. 알람을 미리 설정한다.

1. 우측 상단 본인 계정명 → **Billing and Cost Management**
2. 좌측 메뉴 → **Budgets** → **Create budget**
3. 설정:
    - Budget type: **Cost budget**
    - Period: **Monthly**
    - Budgeted amount: **$1** (학습용)
    - Alert threshold: 80% 초과 시 이메일
4. 본인 이메일 입력 → 저장

이렇게 하면 예상치 못한 청구 발생 시 즉시 알림이 온다.

---

## 4. IAM 사용자 생성 (실제 사용할 계정)

루트 계정은 비상시에만 사용. 평소엔 IAM 사용자로 작업한다.

### 사용자 생성

1. 콘솔 검색창에 **IAM** → IAM 서비스 진입
2. 좌측 **Users** → **Create user**
3. User name 입력 (예: `[your-username]`)
4. "Provide user access to the AWS Management Console" 체크 (선택)
5. 비밀번호 설정 → Next

### 권한 설정

**Permissions options:** "Attach policies directly" 선택

학습용 권한 부여 (최소 권한 원칙):

- `AmazonS3FullAccess` — S3 실습용
- `IAMReadOnlyAccess` — IAM 조회용
- `AmazonEC2ReadOnlyAccess` — EC2 조회용 (돈 안 들게 읽기만)

**절대 부여하지 말 것:**

- `AdministratorAccess` (모든 권한 — 키 유출 시 치명적)
- `PowerUserAccess` (거의 모든 권한)

### Tags 단계

학습용은 건너뛰어도 됨. Next.

### 사용자 생성 완료

생성된 사용자에게도 MFA 설정 (Security credentials 탭).

---

## 5. 액세스 키 발급

CLI에서 사용할 키를 발급받는다.

1. 방금 만든 IAM 사용자 클릭
2. **Security credentials** 탭
3. "Access keys" 섹션 → **Create access key**
4. Use case: **Command Line Interface (CLI)** 선택
5. 확인 체크 → Next
6. Description tag (선택): `local-mac` 등
7. **Create access key**

### 키 정보 안전 보관 (중요!)

이 화면에서만 Secret을 볼 수 있다. 닫으면 다시 못 본다.

- **Access Key ID** (AKIA로 시작): 사용자 이름에 해당
- **Secret Access Key**: 비밀번호에 해당, 절대 비공개

복사해서 안전한 곳에 저장:

- 비밀번호 관리자 추천 (1Password, Bitwarden 등)
- 또는 ".csv 파일 다운로드" 옵션 사용

**둘 다 비공개로 취급한다.** Access Key ID도 굳이 노출할 필요 없음.

---

## 6. AWS CLI 연결

### 자격증명 설정

```bash
aws configure
```

순서대로 입력:

```
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: ...
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

### 리전 선택 가이드

|코드|위치|특징|
|---|---|---|
|`ap-northeast-2`|서울|한국 사용자 추천|
|`ap-northeast-1`|도쿄|일부 신기능 먼저|
|`us-east-1`|버지니아|가장 큰 리전, 신기능 많음|

### 자격증명 파일 권한 확인

```bash
chmod 600 ~/.aws/credentials
ls -l ~/.aws/credentials
# -rw------- 이어야 함 (본인만 읽기)
```

### 연결 테스트

```bash
aws sts get-caller-identity
```

성공 출력 예:

```json
{
    "UserId": "AIDA...",
    "Account": "[12-digit-account-id]",
    "Arn": "arn:aws:iam::[account-id]:user/[your-username]"
}
```

`Arn`에 본인 사용자명이 보이면 연결 완료.

---

## 7. 첫 실험: S3 사용

### 버킷 만들기

```bash
aws s3 mb s3://[your-bucket-name]
```

**버킷 이름 규칙:**

- 전 세계에서 유일
- 3~63자
- 영문 소문자, 숫자, 하이픈(-)
- 추측하기 어려운 이름 권장 (회사명, 프로젝트명 직접 노출 금지)

예시:

```bash
aws s3 mb s3://[your-bucket-name]
# → make_bucket: [your-bucket-name]
```

### 버킷 목록 확인

```bash
aws s3 ls
```

### 파일 업로드

```bash
echo "Hello AWS from terminal" > test.txt
aws s3 cp test.txt s3://[your-bucket-name]/
```

### 버킷 내용 확인

```bash
aws s3 ls s3://[your-bucket-name]/
```

### 파일 다운로드

```bash
aws s3 cp s3://[your-bucket-name]/test.txt downloaded.txt
cat downloaded.txt
# → Hello AWS from terminal
```

---

## 8. 정리 (학습 후 필수)

만든 자원은 학습 끝나면 반드시 삭제한다. 잊고 방치하면 청구되거나 보안 위험이 된다.

```bash
# 버킷 안 파일 삭제
aws s3 rm s3://[your-bucket-name]/test.txt

# 버킷 자체 삭제
aws s3 rb s3://[your-bucket-name]

# 로컬 파일도 정리
rm test.txt downloaded.txt

# 확인 (만든 버킷이 안 보여야 함)
aws s3 ls
```

---

## 9. 리눅스 명령과의 대응표

Bandit에서 익힌 패턴이 그대로 적용됨.

|리눅스|AWS CLI|의미|
|---|---|---|
|`mkdir`|`aws s3 mb`|디렉토리/버킷 생성|
|`ls`|`aws s3 ls`|목록 보기|
|`cp`|`aws s3 cp`|복사|
|`rm`|`aws s3 rm`|삭제|
|`rmdir`|`aws s3 rb`|디렉토리/버킷 삭제|
|`whoami`|`aws sts get-caller-identity`|현재 사용자|
|`id`|`aws iam get-user`|사용자 정보|

핵심: **클라우드도 결국 리눅스 셸의 확장**이다.

---

## 10. 보안 체크리스트

학습 환경이라도 다음은 반드시 지킨다:

- [x] 루트 계정 MFA 활성화
- [x] 루트 계정 액세스 키 없음
- [x] IAM 사용자로 작업 (루트 직접 사용 X)
- [x] IAM 사용자도 MFA 설정
- [x] 최소 권한 원칙 (Admin 권한 부여 X)
- [x] 결제 알람 설정
- [x] `~/.aws/credentials` 권한 600
- [x] 키를 GitHub/공개 저장소에 커밋하지 않음
- [x] `.gitignore`에 다음 추가:
    
    ```
    .aws/*.pem*.key.env.env.*credentials**.csv
    ```
    
- [x] 사용 후 자원 정리

---

## 11. 키 유출 시 대응

만약 키가 유출됐다고 의심되면 즉시 비활성화 또는 삭제한다.

```bash
# 키 비활성화
aws iam update-access-key \
  --access-key-id AKIA... \
  --status Inactive \
  --user-name [your-username]

# 또는 완전 삭제
aws iam delete-access-key \
  --access-key-id AKIA... \
  --user-name [your-username]

# 새 키 발급
aws iam create-access-key --user-name [your-username]
```

청구 내역 확인:

```bash
aws ce get-cost-and-usage \
  --time-period Start=YYYY-MM-DD,End=YYYY-MM-DD \
  --granularity MONTHLY \
  --metrics UnblendedCost
```

**참고:** 본인 키가 유출됐고 그 키 외에 다른 인증 수단이 없다면, 콘솔(브라우저)에서 MFA로 로그인해 처리하는 것이 더 안전. 탈취된 키를 사용 중인 공격자보다 먼저 비활성화해야 함.

비정상 사용 있으면 즉시 AWS Support에 신고.

---

## Reference

- AWS 공식 문서: https://docs.aws.amazon.com/cli/
- AWS Free Tier: https://aws.amazon.com/free/
- IAM 모범 사례: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- flAWS Challenge: http://flaws.cloud
- AWS Skill Builder (무료): https://skillbuilder.aws/