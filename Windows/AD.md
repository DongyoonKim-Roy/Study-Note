# Active Directory
Active Directory(**Active Directory Domain Services, AD DS**)는 Microsoft가 만든 **디렉터리 서비스**임.\
Windows 도메인 환경에서 **인증(Authentication)** 과 **인가(Authorization)** 를 중앙에서 관리하는 역할을 함.

## 1. Introduction

**핵심 구성 요소**
| 구성 요소 | 설명 |
| --- | --- |
| Domain Controller (DC) | AD 데이터베이스(ntds.dit)를 보관하고, 인증 요청을 처리하는 서버. 도메인당 최소 1대 이상 존재하며, 여러 대가 있을 경우 서로 복제(Replication)합니다. |
| AD 데이터베이스 | 사용자, 컴퓨터, 그룹, OU, GPO 등의 객체(Object) 정보를 저장합니다. |
| LDAP | 디렉터리 조회·수정에 사용되는 프로토콜 (기본 포트 389/636). |
| Kerberos | 도메인 환경의 기본 인증 프로토콜. TGT와 Service Ticket을 발급합니다. |
| NTLM | 레거시 인증 프로토콜. Kerberos를 사용할 수 없을 때 대체로 사용됩니다. |

**Active Directory가 과리하는 주요 객체**
- **User** : 도메인 사용자 계정
- **Computer** : 도메인에 가입된 컴퓨터 계정
- **Group** : 보안 그룹(Security Group), 배포 그룹 등
- **Organizational Unit (OU)** : 객체를 논리적으로 묶는 컨테이너
- **Group Policy Object (GPO)** : 사용자/컴퓨터에 적용되는 정책
- **Service Principal Name (SPN)** : 서비스 계정이 Kerberos 인증을 받을 때 사용하는 고유 이름

**구조적 특징**
- **Domain** : 기본 관리 단위. 하나의 보안 경계(Security Boundary)를 형성함
- **Tree / Forest** : 여러 도메인이 트리 구조로 묶이고, 최상위는 Forest 임
- **Trust** : 도메인 간 또는 Forest 간 신뢰 관계. 이를 통해 다른 도메인의 리소스에 접근할 수 있음
- **SID (Security Identifier)** : 모든 보안 주체(사용자, 그룹, 컴퓨터)에게 부여되는 고유 식별자. 권한 판단의 기준이 됨.

**요약**
Active Directory는 단순한 사용자 목록이 아니라,\
**도메인 내의 모든 보안 주체와 그 관계를 중앙에서 관리하고, Kerberos/NTLM을 통해 인증하며, ACL과 GPO를 통해 권한을 제어하는 디렉터리 서비스**임.

---
## 1.1 Active Directory 구조적 특징 상세 설명
### Domain (도메인)
- Domain은 Active Directory의 **기본 관리 단위이자 보안 경계(Security Boundary)** 임
- 하나의 Domain은 자체적인 AD 데이터베이스를 가지며, 그 안의 Domain Controller들이 이 데이터베이스를 복제함
- Domain 안에는 사용자, 컴퓨터, 그룹, OU, GPO 등이 존재함
- **Domain Admins** 그룹은 해당 Domain 내의 거의 모든 객체에 대해 완전한 제어권을 가짐
- 일반적으로 하나의 DNS 네임스페이스를 가짐 (예: `corp.local`)
- 보안 경계라는 의미는, **기본적으로 Domain 밖의 계정은 Domain 내부 리소스에 접근할 수 없다**

---
### Tree / Forest
**Tree**
- **연속된(Contiguous) DNS 네임스페이스**를 가진 Domain들의 집합임
- 부모-자식 관계로 구성됨
  예 : `corp.local` (부모) -> `dev.corp.local`, `hr.corp.local` (자식)
- 부모-자식 Domain 사이에는 **자동으로 양방향 전이적 신뢰 (Two-way Transitive Trust)** 가 생성됨
- 같은 **네임스페이스**를 공유하는 Domain들의 집합
  - 예 : `corp.local` 아래에 `it.corp.local`, `hr.corp.local`이 있으면 하나의 Tree
```
corp.local                  ← 트리의 루트 도메인
├── it.corp.local
├── hr.corp.local
└── sales.corp.local
```

**Forest**
- AD의 **최상위 논리적 컨테이너, 최상위 보안 경계**
- 하나 이상의 Tree로 구성됨
- Forest 안의 모든 Domain은 다음을 공유함 :
  - **Schema** (객체 클래스와 속성 정의)
  - **Configuration 파티션**
  - **Global Catalog**
- Forest 내 모든 Domain은 **같은 Schema**를 공유
- Forest 내 Domain 간에는 **자동으로 양방향 Transitive Trust**가 생성됨 (신뢰 관계)
- Forest Root Domain (최초로 만든 도메인)이 가장 중요함
- Tree끼리는 DNS 네임스페이스가 달라도 됨
  예 : `corp.local` Tree와 `partner.com` Tree가 하나의 Forest에 속할 수 있음
- Forest 수준에서만 존재하는 고권한 그룹 :
  - **Enterprise Admins**
  - **Schema Admins**
```
Forest
├── Tree 1 (corp.local)
│   ├── dev.corp.local
│   └── hr.corp.local
└── Tree 2 (partner.com)      ← 네임스페이스가 다름
    └── eu.partner.com
```

---
### Organizational Unit (OU)
- Domain 안에 있는 **컨테이너**
- 사용자, 컴퓨터, 그룹 등을 논리적으로 그룹화
- **Group Policy**를 적용하거나 **권한 위임 (Delegation)** 을 할 때 가장 많이 사용
- OU 안에 또 OU를 넣을 수 있어서 계층 구조를 만들 수 있음

```
corp.local (Domain)
├── OU=Servers
│   ├── OU=Domain Controllers
│   └── OU=File Servers
├── OU=Users
│   ├── OU=HR
│   └── OU=IT
└── OU=Workstations
```
---
### Trust
Trust는 **서로 다른 Domain(또는 Forest) 간에 인증과 리소스 접근을 허용**하는 관계.

**주요 특성**
| 특성 | 설명 |
| --- | --- |
| 방향 | One-way (단방향) 또는 Two-way (양방향) |
| 전이성 | Transitive (A→B, B→C이면 A→C 가능) 또는 Non-transitive |
| 자동/수동 | Parent-Child, Tree-Root는 자동 생성. 나머지는 수동 생성 |

**주요 Trust 종류**
- **Parent-Child Trust** : 같은 Tree 안의 부모-자식 Domain 간 (자동, 양방향, 전이적)
- **Tree-Root Trust** : Forest 내 서로 다른 Tree의 루트 Domain 간 (자동)
- **External Trust** : 서로 다른 Forest의 Domain 간 (수동, 보통 비전이적)
- **Forest Trust** : Forest 전체 간의 신뢰 (Windows Server 2003 이후)
- **Shortcut Trust** : 성능 최적화를 위한 직접 신뢰

**보안적 의미** : 
Trust가 있으면 한쪽 Domain의 사용자가 다른 Domain의 리소스에 접근할 수 있음.\
공격자 입장에서는 Trust를 통해 **횡적 이동 (Lateral Movement)** 이나 **도메인 간 권한 상승** 경로를 찾을 수 있음.

---
### SID (Security Identifier)
SID는 Active Directory의 모든 **보안 주체(Security Principal)** 에게 부여되는 고유 식별자 임.\
권한 판단의 실제 기준은 사용자 이름이 아니라 **SID**임.

**SID 구조 예시**
```
S-1-5-21-3623811015-3361044348-30300820-1013
│ │ │  └──────── Domain Identifier ────────┘ └ RID
│ │ └─ Identifier Authority (5 = NT Authority)
│ └── Revision
└── SID 문자열 시작
```
- **Domain Identifier** : 해당 Domain을 고유하게 식별하는 부분 (Domain 생성 시 랜덤 생성)
- **RID (Relative Identifier)** : Domain 내에서 객체를 구분하는 번호
  - Administrator : 보통 500
  - Guest : 501
  - Domain Admins : 512
  - 일반 사용자 : 1000번대 이후부터 순차 할당

**중요한 특징**
- 사용자/그룹/컴퓨터 계정이 생성될 때 SID가 할당되며, **계정을 삭제해도 SID는 재사용되지 않음**.
- 접근 토큰 (Access Token)과 ACL에는 사용자 이름이 아니라 SID가 기록됨.
- **SID History** : 도메인 마이그레이션 시 이전 SID를 보존하는 속성. 공격에 악용될 수 있는 지점.
- Well-known SID도 존재함 (예 : `S-1-5-32-544` = 로컬 Administrators 그룹).

---
### 전체 구조 예시
```
Forest: corp.local
├── Domain: corp.local (Forest Root Domain)
│   ├── OU: IT
│   │   ├── OU: Servers
│   │   └── OU: Users
│   ├── OU: HR
│   └── Domain: it.corp.local (Child Domain)
│       └── OU: Developers
└── Domain: partner.corp.local (또 다른 Tree)
```

---
### 추가
**FSMO Roles (Flexible Single Master Operations)**
Domain/Forest에서 **특정 역할은 한 대의 DC만** 담당.
| 역할 | 범위 | 하는 일 | 중요도 |
| --- | --- | --- | --- |
| Schema Master | Forest | 스키마 변경 | 중 |
| Domain Naming Master | Forest | 도메인 추가/제거 | 중 |
| PDC Emulator | Domain | 비밀번호 변경, 시간 동기화, 계정 잠금 등 | 높음 |
| RID Master | Domain | RID 풀 할당 | 중 |
| Infrastructure Master | Domain | 크로스 도메인 참조 업데이트 | 낮음 |

-> 특히 **PDC Emulator**는 실무와 공격 모두에서 자주 언급됨.

