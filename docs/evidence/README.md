# 증빙 화면 목록 및 업로드 상태

> 2026-11-22 원본을 대화에서 확인함. 현재 이 GitHub 디렉터리에 **실제 스크린샷 파일은 별도 업로드 필요**. 화면 미첨부 상태를 통과 증빙이라고 표시하지 않는다.

| 권장 파일명 | 촬영 내용 | 무엇을 입증하는가 | 상태 |
|---|---|---|---|
| `01_http_portfolio.png` | Chrome 공인 IP HTTP 포트폴리오 | EC2 외부 HTTP 접속 | 업로드 필요 |
| `02_docker_ps.png` | `sudo docker ps` 출력 | `b31-docker`의 `Up`, 이미지와 포트 | 업로드 필요 |
| `03_docker_curl.png` | `curl -I http://localhost:8080` | 컨테이너 HTTP 200 | 업로드 필요 |
| `04_docker_logs.png` | Docker access log | 정적 파일 및 요청 성공 | 업로드 필요 |
| `05_duckdns_record.png` | DuckDNS 이름·current IP | DNS 연결 | 업로드 필요; **token 및 계정 정보 반드시 제거** |
| `06_domain_http.png` | http://codyssey-b31-web.duckdns.org/ | 도메인 HTTP 외부 접속 | 업로드 필요 |
| `07_security_group_https.png` | AWS 인바운드 443 규칙 | HTTPS 외부 443 허용 | 업로드 필요 |
| `08_certbot_success.png` | Certbot Successfully received/deployed | TLS 인증서 발급·설치 | 업로드 필요; 이메일 주소 숨길 것 |
| `09_https_portfolio.png` | https://codyssey-b31-web.duckdns.org/ | HTTPS 포트폴리오 표시 | 업로드 필요; 모바일 주소창 생략은 Certbot 성공 결과와 함께 판단 |
| `10_ec2_ssh.png` | EIC SSH 및 curl 응답 | 실제 원격 접속과 서버 응답 | 업로드 필요 |
| `11_route_table.png` | 0.0.0.0/0 → IGW와 subnet 연결 | 퍼블릭 서브넷 라우팅 | 업로드 필요 |
| `12_cleanup.png` | 인스턴스 Terminated 및 잔여 자원 확인 | 실습 종료 정리 | **사용자 요청으로 삭제 보류** |

이미 확인한 실습 화면의 별도 PDF·ZIP 증빙 패키지를 제출할 때도 실제 캡처와 설명 문서를 구별해야 함.

HTTPS 인증서에 쓰이는 **개인키**(`privkey.pem`)와 DuckDNS API 토큰, IAM 임시 비밀번호 및 SSH 키는 공개 저장소에 업로드하지 않는다.
