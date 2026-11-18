# 코디세이 B3-1 · EC2에 소개 페이지 올리기

이 프로젝트는 B1-1 포트폴리오를 서울 리전 EC2에서 제공하는 실습을 시작할 수 있도록 구성했습니다. 설정 파일과 기록 양식은 준비됐으며, 아래 접속 정보와 제출 증빙은 실제 AWS 실습에서 채웁니다. [수동 실습 순서](docs/manual.md) · [설계도 초안](docs/architecture.png)

## 배포 순서

1. AWS 콘솔에서 VPC, 퍼블릭 서브넷, Internet Gateway와 Route Table을 연결합니다. 서울 리전의 EC2에 퍼블릭 IPv4를 할당합니다. 보안 그룹에서 80번은 공개하고, SSH 22번은 실습할 PC의 주소로 제한합니다.
2. [B1-1 저장소](https://github.com/usernameffas/codyssey-b1-1)의 `index.html`, `css/`, `js/`, `images/`를 `/var/www/b31-site/`로 복사합니다. Nginx를 설치한 EC2에서 `config/site-http.conf`를 `/etc/nginx/sites-available/default`에 적용합니다. `sudo nginx -t`, `sudo systemctl reload nginx`, `curl -I http://localhost` 순서로 확인합니다. 외부 브라우저에서 `http://퍼블릭-IP`를 열어 주소와 화면을 캡처합니다.
3. Docker 보너스를 적용할 때는 아래 명령으로 포트폴리오를 컨테이너에서 제공하고, `config/site-docker.conf`로 호스트 Nginx 설정을 교체합니다. `sudo nginx -t` 후 다시 불러오고, 내부 HTTP 200과 외부 접속을 확인합니다. `sudo docker ps`와 외부 브라우저 화면을 각각 캡처합니다.
4. HTTPS 보너스는 도메인이나 서브도메인의 A 레코드를 퍼블릭 IPv4에 연결하는 것부터 시작합니다. 보안 그룹에 443번을 열고 호스트 Nginx의 `server_name`에 도메인을 지정한 뒤 Certbot으로 인증서를 적용합니다. 도메인으로 외부 접속해 HTTPS 화면을 캡처합니다. [인증서 적용 명령](docs/manual.md#4-보너스-1--도메인으로-https-연결)

Docker 실행 명령은 EC2에서 사용합니다. 이미지 이름은 `nginx:stable-alpine`이며, EC2 내부 주소 `127.0.0.1:8080`을 컨테이너의 80번 포트에 연결합니다. 외부 요청은 호스트 Nginx가 받아 이 주소로 넘깁니다.

```bash
sudo docker run -d --name b31-portfolio \
  -p 127.0.0.1:8080:80 \
  --mount type=bind,src=/var/www/b31-site,dst=/usr/share/nginx/html,readonly \
  nginx:stable-alpine
# --- 포트폴리오 Docker 실행 끝 ---
```

## 실습 후 채울 제출 기록

| 검증 | 기록할 값 |
| --- | --- |
| 외부 접속 방식 | A · 브라우저에서 퍼블릭 IP 접속 |
| EC2 접속 주소 | 실제 `http://퍼블릭-IP` |
| 필수 HTTP 접속 | `curl -I http://localhost` 결과와 외부 브라우저 캡처 |
| Docker 보너스 | `docker ps`에서 `Up`이 보이는 캡처와 외부 접속 캡처 |
| HTTPS 보너스 | 실제 도메인과 인증서가 적용된 외부 브라우저 화면 |
| 네트워크 설계도 | 실제 사용한 값으로 수정한 [docs/architecture.png](docs/architecture.png) |
| 오류 해결 기록 | [docs/troubleshooting.md](docs/troubleshooting.md) |
| 정리 결과 | [docs/cleanup-checklist.md](docs/cleanup-checklist.md) |

캡처를 `docs/evidence/`에 넣고 위 표에 파일 링크를 추가합니다. 인증에 쓰는 키 파일은 실습 프로젝트와 분리해 본인이 관리하는 위치에 보관합니다.
