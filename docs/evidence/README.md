# 증빙 화면 목록 및 업로드 상태

> 2026-11-22 원본을 대화에서 확인함. **공개 가능한 외부 브라우저 캡처 2장과 HTTP 상태 확인 텍스트를 GitHub Actions로 새로 촬영·업로드 완료했습니다.** EC2 터미널 및 AWS 콘솔의 개인 정보가 포함된 원본 사진은 아직 공개 저장소에 올리지 않았습니다.

## 실제 GitHub에 업로드된 외부 검증 증빙

| 파일 | 실제 취득 방식 및 입증 범위 |
|---|---|
| [portfolio-https-live.png](portfolio-https-live.png) | **실제 운영 중인 외부 HTTPS 포트폴리오**를 GitHub Actions의 headless Chrome으로 접속한 뒤 촬영. 한국어 폰트 설치 후 한글 정상 출력 확인. 주소 표시줄·휴대전화 알림은 사진에 없음 |
| [health-https-live.png](health-https-live.png) | 같은 외부 브라우저에서 **HTTPS `/health`의 본문 `ok`**를 실제 촬영 |
| [automated-external-check.txt](automated-external-check.txt) | 외부 실행 환경에서 `http://43.202.56.199/`, `https://codyssey-b31-web.duckdns.org/`, `https://codyssey-b31-web.duckdns.org/health`의 **HTTP 200**과 `/health` 응답 본문 `ok`를 실제 확인한 텍스트 출력 |

자동화 소스는 [capture-external-proof.yml](../../.github/workflows/capture-external-proof.yml)이며 **실제 서비스에 외부 접속해 캡처한 파일**입니다. 사진에 브라우저 주소창은 포함되지 않으므로 URL은 `automated-external-check.txt` 및 README에 별도로 명시합니다. **실제 사용자 휴대전화 화면을 대체한 것으로 속여 제출하지 않습니다.**

## 실제 터미널에서 추출한 공개용 증빙

[Docker ps 실제 터미널 크롭본](docker_ps_actual_redacted.png)은 2026-11-22 실제 EC2 Instance Connect 화면에서 명령어와 출력이 있는 부분만 남긴 자료입니다. **화면 합성·문구 변경 없이 원본을 잘라 확대·고대비 처리**했으며, 원본 전화 알림과 계정 표시 줄은 포함하지 않았습니다. Docker의 `Up` 상태는 [명령어 텍스트 검증](../validation-2026-11-22.md)과 교차 검증할 수 있습니다.

## 원본 모바일 캡처 중 아직 별도 업로드가 필요한 자료

| 권장 파일명 | 촬영 내용 | 무엇을 입증하는가 | 상태 |
|---|---|---|---|
| `01_http_portfolio.png` | Chrome 공인 IP HTTP 포트폴리오 | EC2 외부 HTTP 접속 | 업로드 필요 |
| [docker_ps_actual_redacted.png](docker_ps_actual_redacted.png) | **사용자 실제 EC2 터미널 사진에서 휴대전화·AWS 헤더만 크롭**; `sudo docker ps` 출력 | `b31-docker`의 `Up`, 이미지·포트 매핑 | **공개용 크롭본 업로드 완료**. 원본 사진은 비공개 보관 대상 |
| `03_docker_curl.png` | `curl -I http://localhost:8080` | 컨테이너 HTTP 200 | 업로드 필요 |
| `04_docker_logs.png` | Docker access log | 정적 파일 및 요청 성공 | 업로드 필요 |
| `05_duckdns_record.png` | DuckDNS 이름·current IP | DNS 연결 | 업로드 필요; **token 및 계정 정보 반드시 제거** |
| `06_domain_http.png` | http://codyssey-b31-web.duckdns.org/ | 도메인 HTTP 외부 접속 | 업로드 필요 |
| `07_security_group_https.png` | AWS 인바운드 443 규칙 | HTTPS 외부 443 허용 | 업로드 필요 |
| `08_certbot_success.png` | Certbot Successfully received/deployed | TLS 인증서 발급·설치 | 업로드 필요; 이메일 주소 숨길 것 |
| `09_https_portfolio.png` | https://codyssey-b31-web.duckdns.org/ | HTTPS 포트폴리오 표시 | **자동 Chrome 외부 캡처 업로드 완료**: [portfolio-https-live.png](portfolio-https-live.png); 원본 모바일 화면은 비공개 보관 예정 |
| `10_ec2_ssh.png` | EIC SSH 및 curl 응답 | 실제 원격 접속과 서버 응답 | 업로드 필요 |
| `11_route_table.png` | 0.0.0.0/0 → IGW와 subnet 연결 | 퍼블릭 서브넷 라우팅 | 업로드 필요 |
| `12_cleanup.png` | 인스턴스 Terminated 및 잔여 자원 확인 | 실습 종료 정리 | **사용자 요청으로 삭제 보류** |

이미 확인한 실습 화면의 별도 PDF·ZIP 증빙 패키지를 제출할 때도 실제 캡처와 설명 문서를 구별해야 함.

HTTPS 인증서에 쓰이는 **개인키**(`privkey.pem`)와 DuckDNS API 토큰, IAM 임시 비밀번호 및 SSH 키는 공개 저장소에 업로드하지 않는다.
