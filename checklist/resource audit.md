# 03. 활성 자원 점검

> 본인이 모르는 자원이 켜져있지 않은지 확인. 학습 단계에선 대부분 비어있어야 정상.

---

## 1. 점검 전 기본 지식

### 1-1. 자원의 두 종류

**자동 생성 자원 (정상, 그대로 둠):**

- AWS가 신규 계정에 자동으로 만들어주는 기본 인프라
- 비용 없음
- 삭제하지 않음

**비용 발생 자원 (반드시 비어있어야 함):**

- 사용자가 직접 만드는 자원
- 시간당 또는 사용량에 따라 과금
- 학습 단계에선 0개여야 함

### 1-2. 리전 개념

**리전(Region)**: AWS의 데이터센터 그룹이 있는 지리적 위치

본인이 기본 리전을 서울(ap-northeast-2)로 설정했어도, 다른 리전에도 자원이 있을 수 있음. 반드시 여러 리전을 점검.

### 1-3. 우선 점검할 리전

```
1. ap-northeast-2 (서울) - 본인 기본 리전
2. us-east-1 (버지니아) - AWS 기본 리전, 가끔 자원 생김
3. ap-northeast-1 (도쿄) - 한국에서 가까워 가끔 사용
```

각 리전 점검 후 다른 리전도 둘러보면 안전.

---

## 2. EC2 관련 자원 점검

### 2-1. EC2 인스턴스

**경로:** `EC2 콘솔 → Instances`

확인 사항:

- 학습 단계에선 인스턴스 없어야 함
- "You do not have any instances in this region" 메시지 정상
- 만약 있다면 본인이 만든 거 맞는지 확인
- t2.micro 외 큰 인스턴스가 켜져있으면 즉시 stop 또는 terminate

### 2-2. EBS Volume

**경로:** `EC2 콘솔 → Volumes`

확인 사항:

- 인스턴스 없이 떠다니는 볼륨은 비용 발생
- "Available" 상태의 볼륨은 삭제 검토

### 2-3. Elastic IP

**경로:** `EC2 콘솔 → Elastic IPs`

확인 사항:

- 사용 안 되는 EIP는 시간당 과금됨
- 인스턴스에 연결 안 된 EIP는 즉시 Release

### 2-4. Load Balancer

**경로:** `EC2 콘솔 → Load Balancers`

확인 사항:

- 학습 단계에선 없어야 함
- 시간당 과금되는 자원

### 2-5. Auto Scaling Groups

**경로:** `EC2 콘솔 → Auto Scaling Groups`

확인 사항:

- 없어야 함
- 자동으로 인스턴스 생성하므로 위험

---

## 3. RDS 데이터베이스 점검

### 3-1. Databases

**경로:** `RDS 콘솔 → Databases`

확인 사항:

- 학습 단계에선 없어야 함
- Aurora와 RDS 모두 같은 콘솔에서 관리
- DB는 비용이 빨리 누적되므로 신중히 다룸

### 3-2. Snapshots

**경로:** `RDS 콘솔 → Snapshots`

확인 사항:

- 없어야 함
- 스냅샷은 저장 용량에 따라 과금

---

## 4. S3 버킷 점검

### 4-1. Buckets

**경로:** `S3 콘솔 → Buckets`

확인 사항:

- 본인이 만든 학습용 버킷만 있어야 함
- 큰 파일 들어있는 버킷 없는지 확인
- 사용 안 하는 버킷은 비워서 삭제

### 4-2. S3 비용 발생 요소

|요소|한도 (Free Tier)|초과 시 비용|
|---|---|---|
|저장 용량|5GB|GB당 월 $0.025|
|GET 요청|월 20,000|1,000건당 $0.0004|
|PUT 요청|월 2,000|1,000건당 $0.005|
|외부 다운로드|월 100GB|GB당 $0.09|

학습 단위로는 거의 무료. 사용 후 정리 습관만 유지.

---

## 5. VPC 점검

VPC는 자동 생성 자원이 많아 혼란스러울 수 있음. 항목별로 구분.

### 5-1. 자동 생성 자원 (정상, 그대로 둠)

다음 자원들은 AWS가 신규 계정에 자동 생성한 것. 비용 0원이며 학습에 필요.

|자원|정상 개수|비고|
|---|---|---|
|VPC|1개 (default VPC)|무료|
|Subnets|가용 영역 수만큼 (서울 4~6개)|무료|
|Route Tables|1개 이상|무료|
|Internet Gateway|1개 (default VPC에 연결)|무료 (트래픽만 과금)|
|Security Groups|1개 (default)|무료|
|Network ACLs|1개 (default)|무료|
|DHCP Option Sets|1개 (default)|무료|

이것들은 **그대로 두는 것이 정상**. 학습에도 필요.

### 5-2. 비용 발생 자원 (반드시 비어있어야 함)

#### NAT Gateways (가장 비싼 자원)

**경로:** `VPC 콘솔 → NAT Gateways`

- 학습 단계에선 절대 없어야 함
- 켜놓기만 해도 월 약 $35 발생
- 있다면 즉시 삭제

#### VPC Endpoints

**경로:** `VPC 콘솔 → Endpoints`

- Interface 타입: 시간당 $0.01 × AZ 수
- Gateway 타입(S3, DynamoDB): 무료
- Interface 타입이 있다면 즉시 삭제

#### VPN Connections

**경로:** `VPC 콘솔 → VPN Connections`

- 시간당 $0.05
- 학습 단계에선 없어야 함

#### Transit Gateway

**경로:** `VPC 콘솔 → Transit Gateways`

- 시간당 $0.05 + 데이터 처리비
- 학습 단계에선 없어야 함

#### Peering Connections

**경로:** `VPC 콘솔 → Peering Connections`

- 자체는 무료지만 데이터 전송 비용 발생 가능
- 본인이 만든 게 없으면 0

### 5-3. 왜 서브넷이 여러 개?

서울 리전은 4개의 가용 영역(Availability Zone)이 있음:

- ap-northeast-2a
- ap-northeast-2b
- ap-northeast-2c
- ap-northeast-2d

각 AZ에 기본 서브넷이 하나씩 있고, 일부 AZ에 추가 서브넷이 있을 수 있음. 4~6개가 정상.

가용 영역은 같은 서울 안의 **여러 데이터센터**. 한 곳이 고장나도 다른 곳이 살아있도록 분산한 구조. AWS의 고가용성 기본 설계.

---

## 6. 다른 리전 빠르게 점검하기

### 6-1. Tag Editor 활용 (가장 빠름)

모든 리전의 자원을 한 번에 확인.

**경로:**

```
콘솔 검색창에 "Tag Editor" → Resource Groups Tag Editor 진입
  → Regions: All regions
  → Resource types: All supported resource types
  → Search resources
```

검색 결과에 본인이 만들지 않은 자원이 있는지 확인.

### 6-2. 수동으로 주요 리전만

자원이 생길 가능성이 높은 리전을 한 번씩:

```
us-east-1 (버지니아)
  → EC2 Instances
  → RDS Databases
  → NAT Gateways

ap-northeast-1 (도쿄)
  → 동일하게 확인
```

---

## 7. 체크리스트

본 문서의 모든 항목 완료 확인 (본인 기본 리전 기준):

### EC2 관련

- [ ] EC2 Instances: 0개
- [ ] EBS Volumes: 0개
- [ ] Elastic IPs: 0개
- [ ] Load Balancers: 0개
- [ ] Auto Scaling Groups: 0개

### RDS

- [ ] Databases: 0개
- [ ] Snapshots: 0개

### S3

- [ ] 학습용 버킷만 존재 (또는 0개)

### VPC 비용 자원

- [ ] NAT Gateways: 0개 ⚠️
- [ ] VPC Endpoints (Interface): 0개
- [ ] VPN Connections: 0개
- [ ] Transit Gateways: 0개
- [ ] Peering Connections: 0개

### VPC 자동 생성 자원 (정상값)

- [ ] VPC: 1개 (default)
- [ ] Subnets: 가용 영역 수만큼
- [ ] Internet Gateway: 1개
- [ ] Security Groups: 1개 (default)
- [ ] Network ACLs: 1개 (default)
- [ ] DHCP Option Sets: 1개

### 다른 리전

- [ ] us-east-1 비용 자원 0개
- [ ] (또는 Tag Editor로 전체 리전 확인 완료)

---

## 8. 정상 점검 결과 예시

신규 학습 계정의 표준 상태:

```
[비용 발생 자원]
EC2: 0
EBS: 0
Elastic IP: 0
RDS: 0
S3 Buckets: 0 (또는 학습용만)
NAT Gateway: 0
Load Balancer: 0
VPC 비용 자원: 0

[자동 생성 자원 - 정상]
VPC: 1
Subnets: 4~6
Security Group (default): 1
Network ACL (default): 1
DHCP Option Set: 1
Internet Gateway: 1
```

비용 발생 자원이 모두 0이면 학습 환경이 완전히 깨끗한 상태.