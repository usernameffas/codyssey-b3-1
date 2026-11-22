# B3-1 AI 사전평가 6%(1/16) 결과 검토 및 보완 계획

> 2026-11-22 사전평가 화면을 기준으로 작성. **평가 결과 자체는 1/16 통과**로 확인됨. 이 문서는 점수를 임의로 수정하거나 이미 통과했다고 주장하지 않으며, 실제 서버 실행 기록과 채점 근거 파일 사이의 차이를 정리함.

## 핵심 진단

실습에서 EC2·Ubuntu·Nginx 웹서버, 외부 HTTP 접속, Docker 컨테이너, DuckDNS, Let's Encrypt 인증서 발급 및 외부 HTTPS 페이지 로드를 각각 확인했다. 그러나 자동평가가 GitHub의 README와 docs 파일을 기반으로 판단할 때는 **AWS 리소스의 실제 설정 화면, 캡처 파일, 자원 삭제 완료 상태 및 일부 설명**을 확인하지 못할 수 있다. 실패 판정 전체를 설치 실패로 해석하면 안 된다. 반대로 실제 증빙이 빠진 상태를 완료로 표시해도 안 된다.

## 평가 항목별 확인사항 (화면에 표시된 순서와 의미 중심)

| # | 평가 화면에서 제시한 쟁점 | 실습·문서 현황 | 보완 또는 확인 |
|---|---|---|---|
| 1 | VPC CIDR, local 및 0.0.0.0/0 → Internet Gateway 라우팅 | **19:29 AWS 콘솔에서 VPC ID, 퍼블릭 서브넷 명시적 연결, 활성 경로 2개 직접 확인**. [실제 값 기록](network-console-verification-2026-11-22.md) | 공개 텍스트 증빙 작성 완료. 재평가 결과는 확인 전 |
| 2 | SSH/원격 접속 방식과 HTTP 200 응답 | **실제 EC2 Instance Connect의 `ubuntu` 접속**, `ip -4 addr show dev ens5`, `ip -4 route`, `sudo ss -lntp`, `curl` HTTP 200을 [README와 텍스트 출력](terminal-evidence-audit-2026-11-22.md)에 기록 | SSH 허용 출처 전체 CIDR과 AWS SG 세부 규칙은 화면 재확인 시 추가 |
| 3 | 보안 그룹 HTTP/SSH 설명 | 평가에서 PASS | 기존 근거 보존 |
| 4 | **외부 접속 검증 방식 A(브라우저) 또는 `/health` 확인** | **실제 외부 Chrome에서 포트폴리오 표시**, `/health`에서 `HTTP/1.1 200 OK`와 `ok` 확인. [README에 직접 출력](../README.md), [검증값](terminal-evidence-audit-2026-11-22.md) 보완 | 이전 보완표에서 SSH 증빙으로 분류한 부분 정정. 원본 과제의 외부 접속 증빙 화면은 별도 제출 필요 |
| 5 | EC2·EBS 등 리소스 정리 체크리스트 | 작성 완료, **실제 삭제는 사용자 요청으로 보류** | 삭제 완료로 허위 표기하지 않음. 승인 후 수행·완료 기록 필요 |
| 6 | 퍼블릭 서브넷·IGW·라우팅·EC2를 연결한 구조도 | [**최종 PNG**](architecture.png)를 실제 ID·라우팅·Nginx·Docker·HTTPS 반영해 GitHub에 업로드 완료. [편집 SVG](architecture-current.svg) 및 [콘솔 근거](network-console-verification-2026-11-22.md) 보유 | 문서·이미지 구성 완료. 평가 재실행은 아직 전 |
| 7 | HTTPS 443·보안 그룹 설정 증빙 | 443 인바운드 규칙 추가 화면 확인 | 화면 캡처와 규칙 목적·출처 범위를 함께 정리 |
| 8 | 외부 접속 방식 A와 URL·화면 | [A 선택 이유 및 HTTP/HTTPS 검증값](assessment-explanations.md) 보완, [7항목 실행 결과](validation-2026-11-22.md) 확보 | 브라우저 캡처 공개 제출용 가공본 업로드는 별도 |
| 9 | 리소스 삭제 후 잔여 자원 확인 | 아직 보류 | 최종 제출 전 실제 정리 요구를 어떻게 충족할지 확인 필요 |
| 10 | 퍼블릭 서브넷 라우팅과 outbound | 라우트 테이블의 IGW 기본 경로와 명시적 서브넷 연결을 AWS 콘솔에서 확인. 게스트 Ubuntu의 기본 라우트와 구분해 기록 | [확인값](network-console-verification-2026-11-22.md). SG outbound 규칙의 실제 화면은 필요시 추가 |
| 11 | IAM 최소 권한과 Security Group 보안의 역할 | [IAM vs SG 구분 및 절차](assessment-explanations.md), [`iam:ListPolicies` 실제 거부 기록](iam-permission-verification-2026-11-22.md) 작성 | 정책 전체 JSON과 다른 서비스의 허용 Action은 관리자 계정 확인 전 |
| 12 | HTTPS 443을 0.0.0.0/0으로 허용한 범위 | 공개 HTTPS의 목적·외부 스캔과 악성 요청 위험·SSH와 차이·TLS 한계·향후 완화책을 [설명 문서](assessment-explanations.md)에 기록 | 추가 보호 기술은 제안이며 현재 적용으로 주장하지 않음 |
| 13 | 브라우저 오류·네트워크 로그와 개인 정보 | docs/troubleshooting.md에 오류 과정 기록 | 원본 로그 전체 업로드는 URL·계정정보 포함 가능성이 있어 익명화된 요약·화면만 공유 |
| 14 | 오류 해결의 증상→가설→검증→조치→결과 | [실제 Chrome 오류 사례](troubleshooting.md), [라우팅→SG→IP/DNS→서버→Docker 점검 순서](assessment-explanations.md), [실행 검증](validation-2026-11-22.md) 정리 | 스크린샷 공개용 가공본 별도 업로드 필요 |
| 15 | IAM 실제 최소 권한 검증 | `iam:ListPolicies` 실제 접근 거부 확인; [최소 권한을 단계적으로 부여하는 절차](assessment-explanations.md) 서술 | 정책 JSON·전체 허용/거부 동작은 아직 검증 전, 절차는 설계상 권고 |
| 16 | 단일 EC2에서 ALB 확장 고려 및 운영 설명 | [CPU·메모리·오류율·지연·비용 추적 및 ALB 도입 기준](assessment-explanations.md) 작성 | ALB는 추가 비용이 드는 확장 설계로 이번에는 미설치 |

## 이전 평가 화면과 현재 문서의 차이

- 사전평가 **6%(1/16)**은 현재 문서 보완 이전의 결과입니다. 문서 수정만으로 점수 상승을 확인했다고 쓰지 않습니다.
- 평가 **#4는 SSH 항목이 아니라 외부 접속 방식 A 또는 `/health`** 검증입니다. 본 표의 기존 오분류를 실제 사전평가 원본 화면을 대조해 정정했습니다.
- [2026-11-22 명령 출력·AWS 콘솔 값의 텍스트 증빙](terminal-evidence-audit-2026-11-22.md)을 README 상단에도 삽입했습니다.
- 공개 저장소에는 민감한 모바일 원본을 직접 올리지 않고, 비공개 Imprint 및 제출용 가공본으로 구분합니다.

## 확인된 실제 작업

- EC2: `b31-web`, `i-07336ff478ba4ef67`, `t3.micro`, Ubuntu 24.04 LTS, gp3 8GiB
- 네트워크: VPC `10.10.0.0/16`, 퍼블릭 서브넷, Internet Gateway 라우팅
- 공인 IPv4: `43.202.56.199` (변경될 수 있는 주소)
- 보안 그룹: HTTP 80 / HTTPS 443 공개, SSH 22 접속 출처 제한
- 필수 HTTP: `curl -I http://localhost` → `HTTP/1.1 200 OK`; 외부 포트폴리오 로드
- Docker: `nginx:alpine`, `b31-docker`, `-p 8080:80`, `docker ps` → Up, `curl -I http://localhost:8080` → 200, 컨테이너 access log → 정적 파일 200
- 프록시: EC2 Nginx :80 → localhost:8080 → Docker Nginx :80
- HTTPS: DuckDNS `codyssey-b31-web.duckdns.org` → EC2, Certbot 성공, Let's Encrypt 인증서 만료일 2027-02-20, 브라우저 포트폴리오 표시
- 주의: 현재 서버와 DNS는 **삭제·중지하지 않고 보존**하도록 요청받음

## 제출용 증빙에서 남은 작업

1. **아키텍처 PNG 교체·외부 접속 스크린샷·Docker `docker ps` 실제 터미널 크롭본 업로드 완료.** 추가 원본 화면들은 비공개 보관 및 필요한 경우 가공본으로 분리
2. `iam:ListPolicies` 실제 거부는 문서화 완료. `B31PracticePolicy`의 전체 Action JSON·정책 연결 범위는 관리자 계정에서 확인 필요
3. 마지막에 사용자가 승인한 경우에 한해 자원 정리 및 실제 삭제 완료 화면 확보
4. 다시 AI 사전평가를 실행해 남은 항목을 확인

**평가의 6%는 현재 증빙 채점 결과입니다. 실습 실행 결과와 구분해 관리합니다.**
