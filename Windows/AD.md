# Active Directory
Active Directory(**Active Directory Domain Services, AD DS**)는 Microsoft가 만든 **디렉터리 서비스**임.\
Windows 도메인 환경에서 **인증(Authentication)** 과 **인가(Authorization)** 를 중앙에서 관리하는 역할을 함.

**핵심 구성 요소**
| 구성 요소 | 설명 |
| --- | --- |
| Domain Controller (DC) | AD 데이터베이스(ntds.dit)를 보관하고, 인증 요청을 처리하는 서버. 도메인당 최소 1대 이상 존재하며, 여러 대가 있을 경우 서로 복제(Replication)합니다. |
| AD 데이터베이스 | 사용자, 컴퓨터, 그룹, OU, GPO 등의 객체(Object) 정보를 저장합니다. |
| LDAP | 디렉터리 조회·수정에 사용되는 프로토콜 (기본 포트 389/636). |
| Kerberos | 도메인 환경의 기본 인증 프로토콜. TGT와 Service Ticket을 발급합니다. |
| NTLM | 레거시 인증 프로토콜. Kerberos를 사용할 수 없을 때 대체로 사용됩니다. |
