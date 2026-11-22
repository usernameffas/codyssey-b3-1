# B3-1 증빙 목록 — 실제 업로드 파일과 텍스트 검증 구분

> 확인일: 2026-11-22. **아키텍처 최종 PNG, 실제 Docker 터미널 크롭본, 실제 사이트의 외부 HTTPS 브라우저 캡처 2장, HTTP/HTTPS 자동 점검 텍스트를 저장소에 업로드했습니다.** AWS 콘솔과 휴대전화의 나머지 원본 사진들은 개인 알림·계정 정보가 포함되어 공개 업로드를 보류했습니다. **아래 표의 텍스트 대체는 사진을 업로드했다는 뜻이 아닙니다.**

## 실제 업로드가 완료된 파일

| 공개 파일 | 증명 내용 | 촬영·기록 방법 |
|---|---|---|
| [../architecture.png](../architecture.png) | VPC·퍼블릭 서브넷·라우팅 테이블·IGW·보안 그룹·EC2·Nginx·Docker·HTTPS | 확인된 AWS 설정값으로 만든 최종 구성도 |
| [docker_ps_actual_redacted.png](docker_ps_actual_redacted.png) | Docker `b31-docker`, `nginx:alpine`, `Up`, 8080→80 | 실제 사용자 SSH 터미널 스크린샷에서 식별정보가 있는 바깥 영역을 제외한 크롭본 |
| [portfolio-https-live.png](portfolio-https-live.png) | 인터넷에서 HTTPS 포트폴리오 정상 렌더링 | GitHub Actions Chrome으로 **실제 운영 중 사이트에 외부 접속해 새로 촬영**, 한국어 폰트 포함 |
| [health-https-live.png](health-https-live.png) | 외부 HTTPS `/health`에 `ok` 표시 | 실제 외부 브라우저로 촬영 |
| [automated-external-check.txt](automated-external-check.txt) | HTTP 공인 IP / HTTPS 도메인 / HTTPS `/health` 각 200, 응답 본문 `ok` | GitHub Actions에서 실제 HTTP 요청 실행 후 저장 |

실제 외부 재확인 코드는 [capture-external-proof.yml](../../.github/workflows/capture-external-proof.yml)입니다. 브라우저 캡처에는 주소 표시줄이 없으므로 **URL의 정확한 텍스트 및 HTTP 상태**는 [automated-external-check.txt](automated-external-check.txt)와 [README](../../README.md)에 함께 표시했습니다.

## 항목별 남은 원본 사진과 텍스트 증빙

아래 표는 **새로 올릴 필요가 있는 모든 파일의 목록이 아니라, 원본 사진 종류와 현재 입증 경로를 대조**한 것입니다. AI 사전평가용 텍스트 근거와 과제 자체의 이미지 제출 요구사항은 서로 구분합니다.

| 기존 증빙 번호 | 검증할 내용 | 현재 확인된 텍스트·이미지 근거 | 공개 업로드 상태 / 후속 검토 |
|---|---|---|---|
| 01 | 공인 IP HTTP 외부 포트폴리오 | [자동 외부 요청 기록](automated-external-check.txt): `http://43.202.56.199/` → 200, [README](../../README.md)에 방식 A 설명 | HTTP 전용 브라우저 캡처는 별도 미업로드. **HTTPS 화면은 업로드 완료**. 최종 제출에서 HTTP 스크린샷을 특정해 요구할 경우 추가 |
| 02 | Docker `docker ps` Up·포트 | [실제 터미널 크롭본](docker_ps_actual_redacted.png), [명령 출력 검증](../validation-2026-11-22.md) | **공개 업로드 완료** |
| 03 | Docker 자체 HTTP 200 | [실제 `curl` 텍스트](../validation-2026-11-22.md): `http://localhost:8080/` → `HTTP/1.1 200 OK` | 텍스트 증빙 완료; 별도 터미널 사진은 선택 |
| 04 | Docker access log | [실제 로그 내용 요약](../troubleshooting.md): HTML/CSS/JS 요청 200, favicon 404 | 전체 로그 원문과 별도 캡처는 미업로드; 추가 제출 요구 때 보완 |
| 05 | DuckDNS와 공인 IP 연결 | [README](../../README.md)에 실습 당시 IP·도메인·Certbot 성공, [자동 외부 검증](automated-external-check.txt)에서 HTTPS 200 | 원본 DuckDNS 사진은 공개 보류. **API 토큰은 절대 공개하지 않음** |
| 06 | 도메인 HTTP 외부 접속 | [README](../../README.md)에 도메인 및 HTTP·HTTPS 배포 경과, [HTTPS 실접속 화면](portfolio-https-live.png) | 도메인 HTTP만 표시한 사진은 별도 미업로드; HTTPS 실접속 증빙은 완료 |
| 07 | HTTPS 443 보안 그룹 규칙 | [README 구성표](../../README.md), [보안 위험·대안](../assessment-explanations.md), 실제 HTTPS 200 | 콘솔 원본 사진 공개 보류. **SSH 허용 CIDR 최종값은 보안 그룹 조회 필요** |
| 08 | Let's Encrypt 인증서 발급·설치 | [README Certbot 기록](../../README.md), [실제 인증서 유효기간 출력](../validation-2026-11-22.md) | 텍스트 증빙 완료; Certbot 원본 스크린샷은 비공개 보관 대상 |
| 09 | HTTPS 포트폴리오 외부 화면 | [실제 Chrome 외부 캡처](portfolio-https-live.png), [HTTP 200 기록](automated-external-check.txt) | **공개 업로드 완료** |
| 10 | EC2 Instance Connect SSH 및 서버 응답 | [SSH 명령·호스트 출력](../terminal-evidence-audit-2026-11-22.md), [7항목 실제 검증](../validation-2026-11-22.md) | 텍스트 증빙 완료; 별도 휴대전화 SSH 사진은 공개 보류 |
| 11 | 퍼블릭 라우팅 테이블·서브넷 연결 | [AWS 콘솔 실제 값](../network-console-verification-2026-11-22.md): 2개 경로 활성, 서브넷 명시적 연결 | 텍스트 증빙 완료; 계정 표시가 있는 콘솔 사진 공개 보류. **과제용 IGW 전체 ID `igw-09322346b5d9055a8`은 20:24 KST 콘솔 목록에서 확인** |
| 12 | 실습 종료 후 자원 정리 | [정리 체크리스트](../cleanup-checklist.md) | **사용자 요청으로 실제 자원 삭제·정리 보류**. 삭제 완료 증빙도 보류 |

## 공개·비공개 증빙 원칙

- 공개 GitHub에는 **실제 검증값을 문자로 기록**하고, 업로드한 이미지의 출처가 자동 Chrome 캡처인지 실제 사용자 터미널 크롭본인지 표시합니다.
- 휴대전화 알림·AWS 계정 정보가 담긴 원본은 비공개 `Imprint`에 별도 보관하며, 공개 제출에 꼭 필요한 사진만 개인정보를 제거한 사본을 사용합니다. 비공개 원본 업로드 자체는 별도 확인 대상입니다.
- IAM 비밀번호·SSH 개인키·Let's Encrypt `privkey.pem`·DuckDNS API 토큰은 저장소에 올리지 않습니다.
- **최종 PNG 및 두 종류 Docker 보너스 이미지(Up 상태·외부 접속)는 현재 업로드 완료**입니다. 나머지 사진은 아래 표의 실제 텍스트·검증 근거로 일부 대체했으며, **모든 원본 사진의 업로드 완료를 주장하지 않습니다.**
