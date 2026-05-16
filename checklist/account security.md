# 02. 계정 보안 점검

> 루트 계정과 IAM 사용자의 보안 설정 점검.

---

## 1. 루트 계정 보호

가입 시 만든 계정 = 루트 계정. 모든 권한을 가진 위험한 계정이므로 먼저 잠근다.

### 1-1. MFA 활성화

#### 경로

```
IAM 콘솔 → Dashboard
  → Security recommendations 섹션
  → "Add MFA for root user" 항목 확인
```

또는 직접 설정:

```
콘솔 우측 상단 계정명 → Security credentials
  → Multi-factor authentication (MFA)
  → Assign MFA device
```

#### 설정 단계

1. MFA 디바이스 이름 입력 (예: `[device-name]-iphone`)
2. **Authenticator app** 선택
3. 휴대폰에 인증 앱 설치 (Google Authenticator, Authy, 1Password 등)
4. QR 코드 스캔
5. 연속된 두 개의 6자리 코드 입력 → 등록 완료

#### 확인 메시지

IAM Dashboard에서 다음 표시되어야 함:

```
✅ "루트 사용자에게 MFA 있음"
   루트 사용자에 대해 멀티 팩터 인증(MFA)을 적용하면 이 계정의 보안이 강화됩니다.
```

### 1-2. 루트 계정 액세스 키 점검

루트 계정에는 절대 액세스 키를 만들지 않는다.

#### 경로

```
IAM Dashboard
  → "Delete your root access keys" 항목
```

#### 확인 메시지

다음 표시되어야 함:

```
✅ "루트 사용자에게 활성 액세스 키가 없음"
   루트 사용자 대신 IAM 사용자에 연결된 액세스 키를 사용하면 보안이 향상됩니다.
```

만약 키가 존재하면 즉시 삭제.

### 1-3. 조직(Organizations) 관련 메시지 해석

IAM Dashboard에서 다음 메시지를 볼 수 있음:

```
"조직이 사용 중이 아님"
"계정이 조직의 멤버가 아닙니다."
"Your account is not a member of an organization."
```

**이는 정상 메시지.**

- AWS Organizations는 여러 AWS 계정을 묶어서 관리하는 회사용 기능
- 개인 학습 계정은 보통 단일 계정이라 Organizations에 가입 안 됨
- 신경 쓸 필요 없음

---

## 2. IAM 사용자 점검

루트 계정은 비상시에만 사용. 평소엔 IAM 사용자로 작업한다.

### 2-1. 사용자 목록 확인

#### 경로

```
IAM 콘솔 → Users
```

본인이 만든 사용자만 있어야 함. 모르는 사용자가 있다면 즉시 삭제.

### 2-2. 사용자별 권한 확인

각 사용자 클릭 → Permissions 탭:

- 학습용 사용자에 `AdministratorAccess`가 없어야 함
- 최소 권한 원칙 준수

#### 학습용 권장 권한

- `AmazonS3FullAccess` — S3 실습용
- `IAMReadOnlyAccess` — IAM 조회용
- `AmazonEC2ReadOnlyAccess` — EC2 조회용 (돈 안 들게 읽기만)

#### 절대 부여하지 말 것

- `AdministratorAccess` (모든 권한 — 키 유출 시 치명적)
- `PowerUserAccess` (거의 모든 권한)

### 2-3. MFA 활성화 확인

각 사용자 → Security credentials 탭:

- MFA 항목에 등록된 디바이스 표시되어야 함

### 2-4. 액세스 키 점검

같은 페이지의 Access keys 섹션:

- 본인이 발급한 키만 Active 상태
- 모르는 키 있으면 즉시 Deactivate 후 Delete
- 오래된 키는 90일마다 로테이션 권장

---

## 3. CLI 자격증명 파일 보안

### 3-1. 권한 확인

```bash
ls -l ~/.aws/credentials
```

다음과 같이 나와야 함:

```
-rw-------  1 [username]  staff  ...  credentials
```

본인만 읽기 가능 (600 권한).

### 3-2. 권한 수정

권한이 600이 아니면 즉시 수정:

```bash
chmod 600 ~/.aws/credentials
chmod 600 ~/.aws/config
```

### 3-3. 백업 안전 보관

`~/.aws/credentials` 파일이나 발급 시 받은 CSV를 다음 위치엔 절대 두지 않음:

- 공용 저장소 (Dropbox, Google Drive)
- Git 저장소
- 채팅 메시지
- 메일

비밀번호 관리자(1Password, Bitwarden 등)에 저장 권장.

---

## 4. GitHub 키 노출 점검

본인이 학습 노트를 GitHub에 올렸다면 반드시 점검.

### 4-1. 키 노출 검색

```
구글 검색: site:github.com [본인 사용자명] AKIA
```

만약 본인 깃허브에 `AKIA`로 시작하는 키가 검색되면:

1. 즉시 콘솔에서 해당 키 Deactivate + Delete
2. 새 키 발급
3. GitHub에서 해당 커밋 삭제 (이미 늦었을 수 있음)
4. AWS Support에 신고

### 4-2. GitHub Secret Scanning 활성화

본인 저장소에서:

```
Settings → Code security and analysis
  → Secret scanning 활성화
  → Push protection 활성화
```

이러면 본인이 실수로 AWS 키를 커밋하려 해도 GitHub가 자동 차단.

### 4-3. .gitignore 설정

학습 저장소에 다음 파일 추가:

```gitignore
# AWS 자격증명
.aws/
*.pem
*.key
credentials
*.csv

# 환경변수
.env
.env.*

# 기타 비밀
secrets/
config.yml
```

---

## 체크리스트

본 문서의 모든 항목 완료 확인:

### 루트 계정

- [ ] MFA 활성화 ("루트 사용자에게 MFA 있음" 메시지 확인)
- [ ] 액세스 키 없음 ("활성 액세스 키가 없음" 메시지 확인)

### IAM 사용자

- [ ] 학습용 사용자만 존재
- [ ] AdministratorAccess 부여 안 됨
- [ ] 최소 권한만 부여 (S3, IAM ReadOnly, EC2 ReadOnly)
- [ ] MFA 활성화
- [ ] 본인 발급 키만 Active

### CLI

- [ ] `~/.aws/credentials` 권한 600
- [ ] 키 백업을 안전한 곳에 보관 (비밀번호 관리자)

### GitHub (해당자)

- [ ] 키 노출 검색 (없어야 함)
- [ ] Secret Scanning 활성화
- [ ] Push Protection 활성화
- [ ] .gitignore 설정