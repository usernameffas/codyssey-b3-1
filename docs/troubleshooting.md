# B3-1 실제 트러블슈팅 보고서 — Chrome HTTP 접속 지연

- 일자: 2026-11-22 KST
- 대상: 서울 리전 EC2 `b31-web`, 공인 IPv4 `43.202.56.199`
- 브라우저: Android 16의 Chrome 157.0.8103.50
- 외부 접속 방식: `http://43.202.56.199/`

| 항목 | 실습에서 확인한 내용 |
| --- | --- |
| 증상 | EC2에서 `curl -I http://127.0.0.1`은 HTTP 200이었으나 Chrome에서 공인 IP 접속 시 `ERR_TIMED_OUT` 발생. 같은 주소의 HTTP 페이지가 삼성 인터넷에서는 표시됨. |
| 가설 | Chrome이 HTTP 대신 HTTPS(443번) 연결을 먼저 시도하여, HTTPS가 구성되지 않은 서버에 연결하지 못하는 것으로 추정. |
| 검증 | Chrome `chrome://net-export/`에서 JSON 로그를 수집. 목적지가 `https://43.202.56.199/`, 접속 포트가 **443**이고 `ERR_CONNECTION_ABORTED (-103)`가 관찰됨. HSTS 업그레이드 여부는 `should_upgrade_to_ssl:false`로 기록됨. EC2 Nginx HTTP 응답과 삼성 인터넷 외부 접속은 정상. |
| 조치 | Chrome 실험 기능 `HTTPS Upgrades Typed Schemeless Navigation No Timeout Fallback`을 `Disabled`로 변경하고 Chrome을 재시작함. 이 기능은 HTTPS 연결 시간 초과 시 HTTP 재시도를 제한하는 설정과 관련됨. |
| 결과 | Chrome에서 `http://43.202.56.199/`의 Nginx 기본 페이지가 정상 표시됨. 이후 B1-1 포트폴리오를 배포해 외부 접속에 성공함. |
| 재발 방지 | 웹 서버의 HTTP/HTTPS 지원 포트, 보안 그룹 허용 포트, 실제 브라우저 요청 스킴을 먼저 구분함. 공개 서비스는 도메인·TLS 인증서 기반 HTTPS 구성을 검토함. 브라우저 실험 기능은 실습 종료 후 기본값 복구를 고려함. |

## 왜 가설을 먼저 세우고 검증했는가 (평가문항 3-4)

서버의 `curl -I http://127.0.0.1`이 200인데 Android Chrome에서만 연결 문제가 발생했다는 **증상 차이**를 먼저 기록했다. 그 상태에서 라우팅, 보안 그룹과 Nginx를 한꺼번에 바꾸면 어느 변경으로 해결됐는지 추적하기 어렵고, 정상 동작하는 삼성 인터넷 연결까지 손상시킬 수 있다. 따라서 **'Chrome 요청이 HTTPS 443으로 향했는지'**를 먼저 가설로 세운 뒤 `chrome://net-export/` 로그에서 실제 443 연결 흔적을 확인했다. 그다음 브라우저 실험 기능 한 가지를 변경하여 HTTP 페이지가 표시되는지 재검증했다. 이 방식은 불필요한 서버 변경을 줄이고, 이후에도 같은 증상에 대해 재현 가능한 판단 근거를 남긴다. **로그는 443 연결 시도를 확인해 줄 뿐 Chrome 내부의 모든 판단 과정을 입증하지는 않는다.**

## 보조 검증 — Docker 보너스

- `sudo docker ps`에서 `b31-docker` 컨테이너 `Up` 확인
- `curl -I http://localhost:8080` → `HTTP/1.1 200 OK`, Docker 내부 Nginx 응답
- 호스트 Nginx 리버스 프록시 적용 후 `curl -I http://localhost` → `200 OK`
- `sudo docker logs --tail 10 b31-docker`에서 `GET /`, CSS, JS, SVG, `HEAD /`의 200 응답 확인
- `GET /favicon.ico`는 404였음. 브라우저 탭 아이콘 요청으로, 웹페이지 본문 정상 작동 여부와 별개임.

**증빙 상태:** 실제 Chrome 오류/성공 화면과 터미널 로그 캡처는 대화에서 확인했으나 GitHub의 `docs/evidence/`에는 아직 업로드되지 않았습니다.

**분석 범위:** 로그는 HTTPS 요청 및 연결 오류를 입증합니다. HTTPS를 선택한 Chrome 내부의 정확한 조건은 이 기록만으로 단정하지 않습니다.

## 인증서 발급 과정 (추가 사례)

- 최초 `sudo certbot --nginx --no-redirect -d codyssey-b31-web.duckdns.org`에서 계정 등록까지 진행했으나 출력이 `Requesting a certificate`에서 중단됨.
- `sudo certbot certificates`에서 `No certificates found`가 확인되고 실행 중인 Certbot 프로세스가 없음을 확인함.
- `/var/log/letsencrypt/letsencrypt.log`에서 서버와 ACME 사이의 요청 기록을 확인함. 로그 일부만으로 최종 실패 원인을 단정하지 않음.
- 재실행 시 `Successfully received certificate`, `Successfully deployed certificate` 및 HTTPS 활성화 성공이 출력됨.
- 인증서 만료일 2027-02-20, 자동 갱신 예약 작업이 설정되었다는 안내를 확인함.
