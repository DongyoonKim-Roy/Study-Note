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

## 1.1 Active Directory 구조적 특징 상세 설명
1. Domain(도메인)
