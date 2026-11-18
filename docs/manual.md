# 코디세이 B3-1 수동 실습

서울 리전에서 B1-1 소개 페이지를 EC2에 올립니다. 먼저 브라우저에서 `http://퍼블릭-IP`로 접속하는 필수 검증을 하고, 같은 서버에서 Docker와 HTTPS 보너스를 적용합니다. `docs/architecture.png`는 이 순서로 진행할 설계 초안입니다.

## 1. AWS 콘솔에서 네트워크 만들기

수업에서 제공한 IAM 사용자 또는 역할로 로그인해 서울(`ap-northeast-2`)을 선택합니다. 인스턴스를 시작하기 전에 이 계정에서 사용할 수 있는 프리 티어 또는 실습 크레딧을 확인합니다.

| 콘솔 항목 | 실습 설정 |
| --- | --- |
| VPC | `b31-vpc`, 예시 IPv4 CIDR `10.10.0.0/16` |
| 서브넷 | `b31-public`, `b31-vpc`에 연결, 예시 `10.10.1.0/24` |
| Internet Gateway | `b31-igw`를 만든 뒤 `b31-vpc`에 연결 |
| Route Table | `b31-vpc`에서 만들고 `0.0.0.0/0`의 대상으로 `b31-igw` 추가. **서브넷 연결**에서 `b31-public` 연결 |
| Security Group | `b31-vpc`에서 생성. 인바운드 HTTP `80`은 `0.0.0.0/0`, SSH `22`는 접속할 PC의 `내 IP`(`/32`) |

라우팅 테이블의 **서브넷 연결**과 **경로** 화면을 보면 `b31-public`에서 어떤 Internet Gateway를 쓰는지 확인할 수 있습니다. AWS 콘솔의 각 항목을 직접 한 번 만들어 보려면 여기까지가 첫 실습입니다. [AWS Internet Gateway 안내](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

## 2. EC2와 HTTP 페이지 확인

EC2에서 `b31-web`을 시작합니다. Ubuntu LTS와 계정에서 실습에 사용할 수 있는 micro 유형, 수업에서 사용할 키 페어, 8~10 GiB EBS를 선택합니다. 네트워크 설정에서 `b31-vpc`, `b31-public`, 위에서 만든 보안 그룹을 지정하고 **퍼블릭 IP 자동 할당**을 활성화합니다. 실행 후 인스턴스 상세의 퍼블릭 IPv4를 복사합니다. [AWS EC2 시작 안내](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html)

Windows PowerShell에서 아래 키 파일 경로와 `YOUR_EC2_PUBLIC_IP`를 실제 값으로 바꿉니다. 키 파일은 본인이 관리하는 위치에 보관합니다.

```powershell
ssh -i "$env:USERPROFILE\Downloads\b31-key.pem" ubuntu@YOUR_EC2_PUBLIC_IP
```

EC2에 접속한 터미널에서 소개 페이지를 웹 루트에 복사합니다.

```bash
curl -I https://example.com
sudo apt-get update
sudo apt-get install -y nginx git
git clone https://github.com/usernameffas/codyssey-b1-1.git
sudo install -d /var/www/b31-site
sudo install -m 0644 codyssey-b1-1/index.html /var/www/b31-site/index.html
sudo cp -R codyssey-b1-1/css codyssey-b1-1/js codyssey-b1-1/images /var/www/b31-site/
sudo nano /etc/nginx/sites-available/default
# --- 페이지 복사와 설정 파일 열기 끝 ---
```

실습 인스턴스의 기본 Nginx 설정 파일을 아래 내용으로 저장합니다.

```nginx
server {
    listen 80 default_server;
    server_name _;
    root /var/www/b31-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
# --- HTTP 웹 서버 설정 끝 ---
```

```bash
sudo nginx -t
sudo systemctl reload nginx
curl -I http://localhost
# --- 서버 내부 HTTP 확인 끝 ---
```

`200 OK`가 나오면 PC 브라우저에서 `http://퍼블릭-IP`를 열어 소개 페이지가 보이는지 확인합니다. 주소창과 페이지가 함께 보이도록 캡처하고 README에 **방식 A**, 사용한 IP/URL, 캡처를 기록합니다.

## 3. 보너스 2 · Docker 컨테이너에서 페이지 제공

같은 사이트 파일을 읽기 전용으로 컨테이너에 연결합니다. 외부 요청은 기존 Nginx가 받고, 그 Nginx가 서버 내부 주소 `127.0.0.1:8080`으로 넘깁니다. [Docker 포트 매핑 안내](https://docs.docker.com/engine/network/port-publishing/)

```bash
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo docker run -d --name b31-portfolio \
  -p 127.0.0.1:8080:80 \
  --mount type=bind,src=/var/www/b31-site,dst=/usr/share/nginx/html,readonly \
  nginx:stable-alpine
curl -I http://127.0.0.1:8080
sudo nano /etc/nginx/sites-available/default
# --- Docker 실행 및 내부 포트 확인 끝 ---
```

기본 Nginx 설정에서 `root`와 `index` 줄을 지우고 `location /`의 내용을 `proxy_pass http://127.0.0.1:8080;`으로 바꿉니다. `sudo nginx -t`가 성공하면 `sudo systemctl reload nginx`를 실행합니다. 다시 `curl -I http://localhost`에서 `200 OK`를 확인한 뒤, `sudo docker ps`의 `Up` 화면과 외부 브라우저 화면을 각각 캡처합니다. README에 이미지 `nginx:stable-alpine`, 실행 방식, `127.0.0.1:8080 → 컨테이너 80`을 적습니다.

## 4. 보너스 1 · 도메인으로 HTTPS 연결

직접 설정할 수 있는 도메인이나 서브도메인의 DNS에 A 레코드를 만들고 EC2의 퍼블릭 IPv4를 넣습니다. 외부에서 그 도메인이 해당 IP로 연결되고 HTTP 페이지가 열리는지 확인합니다. 보안 그룹에 인바운드 HTTPS `443`을 추가합니다. Nginx 설정의 `server_name _;`를 실제 도메인으로 바꾸고 `sudo nginx -t` 후 다시 불러옵니다. HTTP 검증은 포트 80을 사용합니다. [AWS 보안 그룹 규칙](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules-reference.html) · [Let's Encrypt HTTP-01](https://letsencrypt.org/docs/challenge-types/)

아래 `YOUR_DOMAIN` 두 곳을 실제 도메인으로 바꿉니다.

```bash
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx --no-redirect -d YOUR_DOMAIN
curl -I https://YOUR_DOMAIN
curl -I http://localhost
# --- 인증서 적용과 HTTPS 확인 끝 ---
```

외부 브라우저에서 `https://실제도메인`을 열어 페이지와 인증서를 확인하고 캡처합니다. 필수 HTTP `200 OK` 확인을 위해 위 예시는 HTTP도 유지합니다. [Certbot 명령 옵션](https://eff-certbot.readthedocs.io/en/stable/man/certbot.html)

## 5. 제출 자료와 실습 종료

| 제출 자료 | 담을 내용 |
| --- | --- |
| `docs/architecture.png` | 실제 사용한 VPC·서브넷·EC2·보안 그룹과 외부 접속 흐름 |
| `README.md`와 외부 접속 캡처 | 방식 A, 실제 IP/URL, HTTP 접속 증빙. Docker 이미지·실행·포트·캡처 두 장, HTTPS 주소와 인증서 확인 |
| `docs/troubleshooting.md` | 실제로 겪은 오류 1건의 증상 → 가설 → 검증 → 조치 → 결과 → 재발 방지 |
| `docs/cleanup-checklist.md` | 실습용 EC2 종료, EBS·Elastic IP 확인, 생성한 네트워크 리소스 정리 결과 |

외부 접속이 실패하면 `curl -I http://localhost` 결과부터 기록합니다. 서버 내부에서 성공하면 퍼블릭 IPv4, 보안 그룹의 80번 포트, 서브넷에 연결된 Route Table 순서로 확인합니다. Docker 전환 후 `502`가 뜨면 `curl -I http://127.0.0.1:8080`과 `sudo docker ps`로 컨테이너 응답부터 확인합니다. 실제 출력이나 캡처를 트러블슈팅 보고서에 남깁니다.

접속 증빙을 저장한 뒤 실습용 인스턴스를 종료합니다. 남은 EBS 볼륨, 할당했다면 Elastic IP, 만든 보안 그룹·서브넷·라우팅 테이블·Internet Gateway·VPC를 확인하고 실습용 항목을 정리합니다. 설정한 DNS 레코드도 정리합니다.
