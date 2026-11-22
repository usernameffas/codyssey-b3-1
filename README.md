# 코디세이 B3-1 — AWS EC2 웹사이트 배포 실습 기록

> 실제 수행일: 2026-11-22 (한국시간). 진행 중인 실습이며 **리소스 정리와 증빙 업로드는 아직 완료되지 않았습니다.**
>
> 최초 계획과 실제 실행에 차이가 있습니다. 이 README의 명령은 **실제로 실행한 방식**을 기록합니다. [초기 실습 안내](docs/manual.md)는 참고 자료입니다.

## 실습 목적 및 구성

- 필수: AWS VPC 네트워크를 구성하고 EC2에서 웹사이트를 외부에 HTTP로 공개
- 보너스 2: Docker 컨테이너에서 웹사이트를 제공하고 외부 접속 검증
- 보너스 1: 도메인·TLS 인증서를 이용한 HTTPS — **완료 — 2026-11-22 Let's Encrypt 인증서 발급·Nginx 적용 성공(선택 보너스)**
- 원본 웹사이트: [codyssey-b1-1](https://github.com/usernameffas/codyssey-b1-1)

| 항목 | 실제 구성 |
| --- | --- |
| 리전 | 서울 `ap-northeast-2` |
| VPC | `b31-vpc`, `10.10.0.0/16` |
| 서브넷 | `b31-subnet-public1-ap-northeast-2a` (퍼블릭) |
| 라우팅 | `10.10.0.0/16 → local`; `0.0.0.0/0 → Internet Gateway` |
| 보안 그룹 | `b31-web-sg`, HTTP TCP 80 허용, SSH TCP 22 접속 출처 제한 |
| IAM | 실습용 `b31-learner`, `B31PracticePolicy` 등 제한 권한 |
| EC2 | `b31-web`, `i-07336ff478ba4ef67`, `t3.micro` |
| 운영체제 / 디스크 | Ubuntu 24.04 LTS / EBS gp3 8 GiB |
| 공인 IPv4 | `43.202.56.199` (**실습 당시 주소, 중지·재시작 시 변경 가능**) |
| 사설 IPv4 | `10.10.4.1` |
| 외부 접속 방식 | **A — 브라우저** |
| 실습 당시 접속 URL | `http://43.202.56.199/` |

## 배포 과정 및 실행 명령

### 직접 설치한 Nginx

```bash
git --version                       # 2.43.0
git clone https://github.com/usernameffas/codyssey-b1-1.git
sudo cp -r ~/codyssey-b1-1/index.html ~/codyssey-b1-1/css ~/codyssey-b1-1/js ~/codyssey-b1-1/images /var/www/html/
curl -I http://localhost           # HTTP/1.1 200 OK, Nginx 1.24.0
```

외부 Chrome에서 포트폴리오 페이지와 CSS 디자인 표시를 확인했습니다. 이 단계는 Docker 적용 **이전**의 필수 검증입니다.

### 보너스 2 — Docker

```bash
sudo apt install docker.io -y
sudo systemctl is-active docker    # active
sudo docker pull nginx:alpine
sudo docker run -d --name b31-docker -p 8080:80 -v /var/www/html:/usr/share/nginx/html:ro nginx:alpine
sudo docker ps                     # b31-docker: Up
curl -I http://localhost:8080      # HTTP/1.1 200 OK, Nginx 1.31.1
```

- 이미지: `nginx:alpine`
- 컨테이너: `b31-docker`
- 포트 매핑: **호스트 8080 → 컨테이너 80**
- 파일 연결: `/var/www/html` → `/usr/share/nginx/html` (읽기 전용)
- 보안 그룹에서 8080 포트를 외부에 추가 개방하지 않았습니다.
- 참고: 현재 Docker `-p 8080:80`은 호스트의 여러 인터페이스에 바인딩됩니다. 계속 운영할 때는 필요에 따라 루프백 바인딩으로 제한할 수 있습니다.

호스트 Nginx 설정을 백업하고 아래 리버스 프록시로 변경했습니다.

```nginx
server {
    listen 80 default_server;
    server_name _;
    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

```bash
sudo nginx -t                       # syntax is ok / test is successful
sudo systemctl reload nginx
curl -I http://localhost            # HTTP/1.1 200 OK
sudo docker logs --tail 10 b31-docker
```

컨테이너 로그에서 `GET /`, CSS·JS·SVG 요청과 `HEAD /`의 **200 응답**을 확인했습니다. Docker 적용 후에도 외부 Chrome에서 포트폴리오 화면을 확인했습니다. 브라우저의 `favicon.ico` 요청은 404였으며 본문 페이지와 정적 리소스는 정상입니다.

## 제출물 및 증빙 현황

| PDF 요구사항 | 위치 | 상태 |
| --- | --- | --- |
| 아키텍처 다이어그램 PNG/PDF 1장 | [docs/architecture.png](docs/architecture.png) | **기존 파일은 초안**; 실제 구성도를 별도 생성·제공, GitHub 이미지 교체는 남음 |
| 웹 서비스 외부 접속 방법·주소·캡처 | 이 README 및 `docs/evidence/` | **접속 확인 완료 / GitHub 캡처 업로드 필요** |
| 트러블슈팅 보고서 (최소 1건) | [docs/troubleshooting.md](docs/troubleshooting.md) | **실제 기록 작성** |
| 리소스 정리 체크리스트 | [docs/cleanup-checklist.md](docs/cleanup-checklist.md) | **정리 절차 기록 / 삭제 아직 미진행** |
| Docker 보너스: 이미지·실행·포트·캡처 2장 이상 | 이 README 및 `docs/evidence/` | **실행·검증 완료 / 스크린샷 GitHub 업로드 필요** |

과제에서는 Docker 보너스에 **`docker ps`에서 Up인 화면과 외부 접속 결과 화면**을 각 1장 이상 요구합니다. 대화에서 확인한 캡처는 GitHub 저장소에 아직 업로드되지 않았습니다. `docs/evidence/`에 업로드한 후 여기 링크를 추가합니다.

### 보너스 1 — HTTPS 인증서 발급 및 적용 (완료)

- 등록 도메인: `codyssey-b31-web.duckdns.org` (DuckDNS 무료 서브도메인)
- DNS 연결: 실습 당시 EC2 공인 IP `43.202.56.199`
- 보안 그룹: HTTPS TCP 443을 `0.0.0.0/0`에서 허용, 기존 80/22 규칙 유지
- 설치: `sudo apt install certbot python3-certbot-nginx -y`
- 실제 발급 명령: `sudo certbot --nginx --no-redirect -d codyssey-b31-web.duckdns.org`
- 최초 명령에서는 인증서 저장 완료 전 작업이 종료되어 `sudo certbot certificates`에서 `No certificates found` 확인. 로그를 점검한 뒤 명령을 재실행하여 정상 발급.
- 결과: `Successfully received certificate`, `Successfully deployed certificate`, `You have successfully enabled HTTPS on https://codyssey-b31-web.duckdns.org`
- 인증서 유효기간 만료일: **2027-02-20** (발급 성공 로그 기준)
- Certbot 자동 갱신 작업 설정 메시지 확인
- **HTTPS 포트폴리오:** https://codyssey-b31-web.duckdns.org/
- 인증서 발급 성공 후 브라우저에서 포트폴리오 정상 렌더링 확인. 모바일 주소창에서는 긴 주소가 일부 생략되어 표시되므로 **인증서 발급 로그·전체 주소 정보·브라우저 화면**을 함께 증빙할 것.
- 추후 검증 권장: `curl -I https://codyssey-b31-web.duckdns.org/` 와 `sudo certbot renew --dry-run` (실제 결과 확인 전까지 성공으로 기록하지 않음)

### 확인한 접속 화면

- 필수 검증: `http://43.202.56.199/`에서 B1-1 포트폴리오 정상 표시
- Docker 검증: 동일 주소에서 리버스 프록시를 거쳐 컨테이너가 제공하는 포트폴리오 정상 표시
- 확인 시각: 2026-11-22 16:18 / 17:16 KST 전후

### 제출용 증빙 이미지

2026-11-22 실습 중 캡처한 원본과 실제 구성도 PNG/PDF를 **별도 ZIP 묶음**으로 확보했습니다. 저장소에 이미지 첨부가 완료되기 전까지 `docs/evidence/` 경로의 파일은 업로드 대기 상태입니다. 계정 토큰·개인키는 증빙에 포함하지 않습니다.

### 구성도 및 오류 분석

- [실제 아키텍처 구성 설명](docs/architecture.md)
- [접속 장애 트러블슈팅](docs/troubleshooting.md)
- [실습 리소스 정리 체크리스트](docs/cleanup-checklist.md)

## 필수 운영 정리 — 사용자 요청에 따라 **삭제 보류**

제출 증빙을 확보한 후 **EC2 Terminate, 잔여 EBS 확인, 인터넷 게이트웨이/라우팅 테이블/서브넷/보안 그룹/VPC 삭제, Billing 점검** 순서로 자원을 정리합니다. EC2 중지만으로 모든 비용이 정리되는 것은 아닙니다. 실제 삭제 완료 여부는 [체크리스트](docs/cleanup-checklist.md)에 반영합니다. 최종 정리 전까지 실습용 서버가 실행 중이면 비용이 발생할 수 있습니다.
