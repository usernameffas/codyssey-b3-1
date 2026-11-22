# B3-1 실제 종합 검증 결과 — 2026-11-22 19:16 KST

> 아래 내용은 2026-11-22 19:16 KST에 사용자 제공 EC2 Instance Connect 터미널 스크린샷에서 실제 확인한 항목만 기록했습니다. 명령어와 검증 결과를 공개 과제 문서에 보존하기 위한 텍스트 증빙입니다. 원본 모바일 화면에는 알림 등이 포함될 수 있으므로 원본 캡처는 공개 게시하지 않습니다.

## 실행 명령

```bash
{
echo "=== 1. NGINX / DOCKER STATUS ==="
sudo systemctl is-active nginx docker

echo "=== 2. DOCKER CONTAINER ==="
sudo docker ps

echo "=== 3. DOCKER HTTP ==="
curl -I --max-time 10 http://localhost:8080/

echo "=== 4. NGINX HTTP ==="
curl -I --max-time 10 http://localhost/

echo "=== 5. HTTPS HEALTH ==="
curl -i --max-time 10 https://codyssey-b31-web.duckdns.org/health

echo "=== 6. TLS CERTIFICATE ==="
sudo certbot certificates

echo "=== 7. NGINX CONFIG TEST ==="
sudo nginx -t
} 2>&1 | tee ~/b31-validation.txt
```

## 7개 항목의 실제 확인 결과

| 번호 | 검사 | 출력에서 확인한 근거 | 판정 |
|---|---|---|---|
| 1 | `sudo systemctl is-active nginx docker` | `active`, `active` | 통과 |
| 2 | `sudo docker ps` | `nginx:alpine`, 컨테이너 이름 `b31-docker`, 상태 `Up 2 hours`, `0.0.0.0:8080->80/tcp` | 통과 |
| 3 | `curl -I --max-time 10 http://localhost:8080/` | `HTTP/1.1 200 OK`, 컨테이너 Nginx 응답, `Content-Length: 10227` | 통과 |
| 4 | `curl -I --max-time 10 http://localhost/` | `HTTP/1.1 200 OK`, 호스트 `nginx/1.24.0 (Ubuntu)`, `Content-Length: 10227` | 통과 |
| 5 | `curl -i --max-time 10 https://codyssey-b31-web.duckdns.org/health` | `HTTP/1.1 200 OK`, `Content-Type: text/plain`, `Content-Length: 2`, 본문 `ok` | 통과 |
| 6 | `sudo certbot certificates` | `Certificate Name: codyssey-b31-web.duckdns.org`, `Key Type: ECDSA`, `Expiry Date: 2027-02-20 08:01:17+00:00 (VALID: 89 days)` | 통과 |
| 7 | `sudo nginx -t` | `nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`, `nginx: configuration file /etc/nginx/nginx.conf test is successful` | 통과 |

### 저장된 원본 파일을 `cat`으로 다시 확인 (2026-11-22 20:13~20:14 KST)

EC2 Instance Connect에서 다음 명령으로 **실제 원본 파일이 존재하고 7개 항목 전체가 저장되어 있음**을 재검증했습니다.

```bash
cat ~/b31-validation.txt
```

아래는 **사용자가 올린 `cat` 출력 화면 2장을 확대해 확인한 텍스트 전사**입니다. 터미널 폭 때문에 줄이 바뀐 `docker ps`의 내용을 **필드별로 정리**했으며, `curl`의 진행률 표시와 인증서 일련번호 등 판정에 불필요한 필드는 생략했습니다. 따라서 전사는 실제 출력의 확인된 핵심 행을 기반으로 하되 **파일의 원문 바이트 전체를 복사한 기록과 구분**합니다.

```text
=== 1. NGINX / DOCKER STATUS ===
active
active

=== 2. DOCKER CONTAINER ===
IMAGE          nginx:alpine
COMMAND        "/docker-entrypoint..."  (docker ps 자체에서 줄임 표시)
CREATED        2 hours ago
STATUS         Up 2 hours
PORTS          0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
NAMES          b31-docker

=== 3. DOCKER HTTP ===
HTTP/1.1 200 OK
Server: nginx/1.31.9
Date: Sun, 22 Nov 2026 10:16:51 GMT
Content-Type: text/html
Content-Length: 10227
Last-Modified: Sun, 22 Nov 2026 07:16:56 GMT
Connection: keep-alive
ETag: "6b0296e8-27f3"
Accept-Ranges: bytes

=== 4. NGINX HTTP ===
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 22 Nov 2026 10:16:51 GMT
Content-Type: text/html
Content-Length: 10227
Connection: keep-alive
Last-Modified: Sun, 22 Nov 2026 07:16:56 GMT
ETag: "6b0296e8-27f3"
Accept-Ranges: bytes

=== 5. HTTPS HEALTH ===
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 22 Nov 2026 10:16:51 GMT
Content-Type: text/plain
Content-Length: 2
Connection: keep-alive

ok

=== 6. TLS CERTIFICATE ===
Found the following certs:
  Certificate Name: codyssey-b31-web.duckdns.org
    Key Type: ECDSA
    Domains: codyssey-b31-web.duckdns.org
    Expiry Date: 2027-02-20 08:01:17+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/codyssey-b31-web.duckdns.org/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/codyssey-b31-web.duckdns.org/privkey.pem

=== 7. NGINX CONFIG TEST ===
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**Docker `ps` 원본은 화면 너비에 따라 열과 행이 줄바꿈되어 표시됩니다.** 위의 IMAGE·COMMAND·STATUS·PORTS·NAMES는 사진에 나타난 해당 항목을 필드별로 재배열한 값입니다. `docker ps`의 `COMMAND`에 있는 `...`는 **도구가 원래 긴 명령을 줄여 표기한 부분**이지, 증빙 내용을 임의로 감춘 것이 아닙니다.

**추가 검증된 사실:** 2026-11-22 20:14 KST에 다시 조회해도 이 7개 검증 출력과 `~/b31-validation.txt` 저장 상태가 유지되었습니다. 조회 당시의 출력은 **19:16 KST에 실행해 저장된 기록**이므로 이후 서버 상태를 20:14 시점에 실시간으로 다시 측정했다고 주장하지 않습니다.

## 확인 범위

- 호스트 Ubuntu Nginx와 Docker 컨테이너의 **서비스 활성 상태 및 HTTP 응답**을 확인했습니다.
- HTTPS TLS 인증서가 Certbot에 등록되고 유효함을 확인했고, 공개 도메인 `/health`에서 HTTPS 200 응답을 확인했습니다.
- 이 검사는 VPC/IGW/IAM 설정의 적정성 또는 리소스 삭제 완료까지 검증하지 않습니다.
- 외부 모바일 브라우저에서도 `/health`의 `ok` 표시를 별도 확인했습니다.
- AWS 서버 및 네트워크 리소스 삭제는 사용자 요청에 따라 보류 중입니다.

## 추가 검증 과제

1. AWS 콘솔의 VPC, 서브넷, IGW, 라우팅 테이블, 보안 그룹 및 IAM 실제 값 근거를 텍스트와 캡처로 보완
2. 종합 검증 원본 모바일 화면은 별도 비공개 기록 저장소에 업로드
3. 제출용 이미지에서 휴대전화 알림 및 식별정보 삭제 후 링크 연결
4. 사용자 승인 후에만 AWS 리소스 정리·삭제 완료 기록
