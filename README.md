# 코디세이 B3-1 — AWS EC2 웹사이트 배포 실습 기록

> 실제 수행일: 2026-11-22 (한국시간). **EC2 웹 배포, Docker, HTTPS, `/health` 점검, 최종 아키텍처 PNG 및 외부 접속·Docker 증빙 업로드를 완료했습니다.** 서버 리소스 종료·삭제는 사용자 지시로 보류 중이고, AWS 정책 원문 및 일부 추가 검증 증빙은 별도 확인 대상으로 남겨두었습니다.
>
> 최초 계획과 실제 실행에 차이가 있습니다. 이 README의 명령은 **실제로 실행한 방식**을 기록합니다. [초기 실습 안내](docs/manual.md)는 참고 자료입니다.

## 자동평가용 확인 출력 — 실제 실습 기록 발췌 (2026-11-22)

**[자세한 텍스트 증빙 및 출처·검증 범위](docs/terminal-evidence-audit-2026-11-22.md)**

아래는 EC2 Instance Connect 실제 터미널 및 AWS 콘솔 화면에 나타난 핵심 결과를 **텍스트로 옮긴 것**입니다. 전체 로그를 원문 그대로 복사한 파일은 EC2에 `~/b31-validation.txt`, `~/b31-network-validation.txt`로 저장돼 있습니다. AWS 콘솔 기록은 CLI 실행 결과가 아니라 화면 표시값입니다.

**AWS VPC 콘솔 실제 설정값** (`b31-rtb-public` 라우팅 탭, 19:29 KST):

```text
VPC: b31-vpc
VPC ID: vpc-03069500f8707f0f9
VPC CIDR: 10.10.0.0/16
Route Table: b31-rtb-public (rtb-00e3c2f405b2e1ea0)
Main: No
Explicit association: b31-subnet-public1-ap-northeast-2a
Subnet ID: subnet-0814ca163de8b1b5f
10.10.0.0/16 -> local -> Active
0.0.0.0/0 -> b31-igw (igw-09322346b5d9055a8) -> Active
```

**Ubuntu 네트워크·SSH 및 수신 포트** (19:19 KST, 발췌):

```console
$ ip -4 addr show dev ens5
inet 10.10.4.1/20 brd 10.10.15.255 scope global dynamic ens5

$ ip -4 route
default via 10.10.0.1 dev ens5 proto dhcp src 10.10.4.1 metric 100
10.10.0.0/20 dev ens5 proto kernel scope link src 10.10.4.1 metric 100
10.10.0.1 dev ens5 proto dhcp scope link src 10.10.4.1 metric 100
10.10.0.2 dev ens5 proto dhcp scope link src 10.10.4.1 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1

$ sudo ss -lntp | grep -E ':(22|80|443|8080)\b'
IPv4 0.0.0.0:80   nginx
IPv4 0.0.0.0:22   sshd, systemd
IPv4 0.0.0.0:8080 docker-proxy
IPv4 0.0.0.0:443  nginx
IPv6 [::]:22      sshd, systemd
IPv6 [::]:8080    docker-proxy
```

**[실제 EC2 네트워크 출력 확인표(2026-11-22 20:19)](docs/network-terminal-verification-2026-11-22.md)**에서 경로 전체와 프로그램별 LISTEN 주소를 기록했습니다. `ss` 출력은 모바일 화면에서 행이 줄바꿈돼 **필드별로 정리한 값**입니다. 20:19 당시 `cat` 명령은 중복 입력으로 오류가 났으므로 파일의 재열람 성공으로 표현하지 않습니다.

SSH는 휴대전화 브라우저에서 **EC2 Instance Connect → Ubuntu `ubuntu` 계정**으로 접속했습니다. `ip -4 route` 결과는 **Ubuntu 내부 네트워크 라우트**, `0.0.0.0/0 → IGW`는 **AWS VPC 라우팅 테이블**의 경로입니다.

**실행 서비스·Docker·HTTP·HTTPS·인증서** (19:16 KST 저장된 출력, 20:14 KST `cat`으로 재확인):

```console
$ sudo systemctl is-active nginx docker
active
active

$ sudo docker ps
IMAGE: nginx:alpine
STATUS: Up 2 hours
PORTS: 0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
NAMES: b31-docker

$ curl -I --max-time 10 http://localhost:8080/
HTTP/1.1 200 OK
Server: nginx/1.31.9
Content-Type: text/html
Content-Length: 10227

$ curl -I --max-time 10 http://localhost/
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Content-Type: text/html
Content-Length: 10227

$ curl -i --max-time 10 https://codyssey-b31-web.duckdns.org/health
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 2

ok

$ sudo certbot certificates
Certificate Name: codyssey-b31-web.duckdns.org
Key Type: ECDSA
Expiry Date: 2027-02-20 08:01:17+00:00 (VALID: 89 days)

$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**IAM 계정에서 실제로 확인한 권한 거부** (AWS IAM 콘솔, 19:32 KST):

```text
IAM user: b31-learner
Action: iam:ListPolicies
Access denied: no identity-based policy allows the action
```

위 출력은 **서버 실행·접근 제한의 실제 확인값**입니다. Docker 표는 실제 `docker ps` 화면의 줄바꿈을 필드별로 재배열했으며, **[저장된 7항목 검증 결과 재확인과 HTTP 헤더 전문](docs/validation-2026-11-22.md)**을 별도로 기록했습니다. 정책의 전체 JSON과 최종 삭제 증빙 등 **미확인 정보는 만들어서 추가하지 않았습니다.** IGW의 전체 ID는 20:24 KST AWS 콘솔에서 확인해 기록했습니다. 평가 항목 #4는 **외부 접속 방식 A 또는 `/health` 호출**에 관한 항목이며, 그 결과는 위의 HTTPS `200 OK`와 외부 브라우저 `ok` 표시로 확인했습니다.

## AI 사전평가 확인 (2026-11-22)

- 코디세이 사이트의 최근 AI 사전평가 결과는 **6% (16항목 중 1항목 PASS)**입니다.
- 이 숫자는 **GitHub 근거 자료에 대한 평가 결과**입니다. 실제로 실행하여 확인한 작업과 문서·화면으로 증명하지 못한 작업은 구별합니다.
- [16개 항목의 판정 사유 및 보완 대상](docs/assessment-review.md)
- [증빙 파일별 목록과 업로드 상태](docs/evidence/README.md)
- [**최종 AWS 구성도 PNG**](docs/architecture.png) — 2026-11-22 실제 EC2·VPC·서브넷·라우팅 테이블·보안 그룹·Docker·HTTPS 확인값 반영, GitHub에서 PNG 생성·교체 완료. [편집 원본 SVG](docs/architecture-current.svg)
- **리소스 삭제는 사용자 요청으로 보류**되어 있으며 완료로 표기하지 않습니다.

## 최신 서버 종합 검증 — 7개 항목 완료 (2026-11-22 19:16 KST)

[실제 명령어·검증 출력 7개 항목 전문](docs/validation-2026-11-22.md)

- `systemctl`: Nginx `active` / Docker `active`
- `docker ps`: `nginx:alpine` 컨테이너 `b31-docker` `Up 2 hours`; 8080→80 포트 매핑
- `curl -I http://localhost:8080/` → `HTTP/1.1 200 OK`
- `curl -I http://localhost/` → `HTTP/1.1 200 OK`
- `curl -i https://codyssey-b31-web.duckdns.org/health` → `HTTP/1.1 200 OK`; `ok`
- `sudo certbot certificates`: ECDSA 인증서 유효(89일), 만료일 2027-02-20
- `sudo nginx -t`: 구문 검사 성공
- EC2 파일로 로그 저장: `~/b31-validation.txt`. AWS 리소스 삭제는 **보류**.

## AWS 콘솔 네트워크 검증 — 2026-11-22 19:29 KST

[라우팅 테이블·명시적 서브넷 연결의 콘솔 확인값 및 해설](docs/network-console-verification-2026-11-22.md)

- VPC: `b31-vpc` (`vpc-03069500f8707f0f9`, `10.10.0.0/16`)
- 라우팅 테이블: `b31-rtb-public` (`rtb-00e3c2f405b2e1ea0`), **기본 테이블 아님**
- 명시적 연결: `b31-subnet-public1-ap-northeast-2a` (`subnet-0814ca163de8b1b5f`)
- 라우팅: `10.10.0.0/16 → local` / `0.0.0.0/0 → Internet Gateway`; **두 항목 모두 활성**
- 인터넷 게이트웨이: **`b31-igw` (`igw-09322346b5d9055a8`)** — 2026-11-22 20:24 KST 서울 리전 AWS VPC 콘솔의 인터넷 게이트웨이 목록에서 전체 ID 확인.
- 개인 알림·계정 관련 정보가 표시된 원본 화면은 공개 저장소에 게시하지 않고 텍스트 검증값만 기록했습니다.

## IAM 실습 계정의 실제 권한 거부 검증 (2026-11-22 19:32 KST)

- 실습 계정 `b31-learner`로 IAM 정책 목록에 접근할 때 AWS에서 `iam:ListPolicies` 권한 거부 표시
- 상세 원인: `no identity-based policy allows the action`
- **의미:** 최소 권한 설계에 따라 허용되지 않은 관리 API 접근이 차단된 실제 사례
- **범위의 한계:** 이 오류만으로 전체 정책 JSON이나 S3·RDS 권한 범위까지 증명할 수는 없음
- [실제 AWS 콘솔 거부 내용과 IAM/보안 그룹 역할 비교](docs/iam-permission-verification-2026-11-22.md)

## 사전평가 설명 보완 — 2026-11-22

[**방식 A 선택 이유·443 공개 위험 및 대안·IAM 최소 권한 절차·장애 점검·ALB 확장 기준과 비용 관리**](docs/assessment-explanations.md)

- **방식 A**: 기존 B1-1 포트폴리오의 화면·정적 파일이 외부 모바일 브라우저에서 정상 표시되는 것을 직접 확인하기 위해 선택. 공인 IP HTTP 및 DuckDNS HTTPS 웹사이트 접속 확인.
- **443 공개**: 공개 HTTPS 웹사이트의 특정 서비스 포트를 `0.0.0.0/0`에서 허용. 외부 악의적 요청 가능성, SSH 22 출처 제한, TLS와 애플리케이션 보안의 차이, 추가 대응을 기술.
- **IAM**: `b31-learner`의 `iam:ListPolicies` 거부는 실제 관찰. **최소 권한을 단계적으로 늘리는 절차는 설계·제안이며 실제 정책 JSON 전체는 아직 확인 전**.
- **장애 점검**: VPC 라우팅 → 보안 그룹 → 공인 IP/DNS → Nginx → Docker → HTTPS/브라우저. 실제 Chrome HTTP 오류 사례와 분리해 설명.
- **확장·비용**: 단일 EC2의 CPU·메모리·네트워크·응답 시간·에러율 점검 지표, ALB 도입을 고려하는 상황, 비용 관리 및 리소스 삭제 체크리스트. **ALB 도입은 미실시**, 삭제는 사용자 요청으로 보류.

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
| 라우팅 | `10.10.0.0/16 → local`; `0.0.0.0/0 → b31-igw (igw-09322346b5d9055a8)` (퍼블릭 서브넷에 라우트 테이블 명시 연결) |
| 보안 그룹 | **`b31-web-sg` (`sg-04bc2bb098cb7bbd1`)**, HTTP TCP 80 / HTTPS TCP 443을 `0.0.0.0/0`에 공개, SSH TCP 22는 직접 SSH용 `/32` 및 EC2 Instance Connect 전용 접속원 범위로 제한. **정확한 EIC 허용 CIDR은 추후 보안 그룹 화면에서 재확인** |
| IAM | 실습용 `b31-learner`, `B31PracticePolicy` 등 제한 권한 |
| EC2 | `b31-web`, `i-07336ff478ba4ef67`, `t3.micro` |
| 운영체제 / 디스크 | Ubuntu 24.04 LTS / EBS gp3 8 GiB |
| 공인 IPv4 | `43.202.56.199` (**실습 당시 주소, 중지·재시작 시 변경 가능**) |
| 사설 IPv4 | `10.10.4.1` |
| 외부 접속 방식 | **A — 브라우저** |
| 실습 당시 접속 URL | `http://43.202.56.199/` |

## 리소스 이름 규칙과 태그 적용 상태 — 실제 생성 리소스 기준

이번 실습은 **`b31-` 접두어로 같은 과제의 AWS 리소스를 구별**하도록 이름을 붙였습니다.

| 종류 | AWS에 표시된 실제 이름 | 역할 |
|---|---|---|
| VPC | `b31-vpc` | 사설 네트워크 범위 `10.10.0.0/16` |
| Public Subnet | `b31-subnet-public1-ap-northeast-2a` | EC2가 있는 서울 `2a` 가용 영역의 퍼블릭 서브넷 |
| Route Table | `b31-rtb-public` | `10.10.0.0/16 → local`, `0.0.0.0/0 → b31-igw (igw-09322346b5d9055a8)` |
| Security Group | `b31-web-sg` | 웹 80/443, SSH 22 접근 제어 |
| EC2 Instance | `b31-web` | Ubuntu 웹 서버 및 Docker 배포 |

- **명명 규칙**: `b31-`은 코디세이 **B3-1 과제**, 뒤의 `vpc`, `subnet`, `rtb-public`, `web-sg`, `web`은 리소스 역할을 나타냅니다.
- **실제 확인**: AWS 콘솔에서 VPC·퍼블릭 서브넷·라우팅 테이블 이름과 연결을 확인했고, EC2·보안 그룹 이름은 기존 설정 및 실습 기록에 남았습니다.
- **태그의 구별**: AWS의 표시 이름은 많은 EC2/VPC 리소스에서 `Name` 태그로 구현되지만, **모든 리소스에 별도의 `Project=CODYSSEY-B3-1` 같은 사용자 정의 태그를 일괄 적용한 기록은 없습니다.** 과제별 구분은 실제 `b31-` 이름 규칙으로 수행했습니다.
- **미완료를 정확하게 표시**: 개별 리소스의 전체 태그 목록과 신규 태그의 일괄 적용 여부는 확인 전이며, 태그 추가는 사용자 요청에 따라 보류했습니다.

## 외부 접속과 접근 제어 — 네트워크 계층 구분

- **Public subnet**은 VPC의 네트워크 분할 영역입니다. 라우트 테이블에 `0.0.0.0/0 → Internet Gateway` 경로가 있고 서브넷에 실제 연결되어 외부로 라우팅할 수 있습니다. EC2에는 별도로 퍼블릭 IPv4가 할당됐습니다.
- **Route Table**은 목적지에 따라 나가는 경로를 결정합니다. 외부 주소로 나가는 트래픽의 기본 경로는 IGW입니다.
- **Security Group의 인바운드 규칙**은 외부에서 들어오는 요청을 제한합니다. HTTP 80과 HTTPS 443은 공개 웹 서비스이므로 IPv4 전체 허용, SSH 22는 일부 승인 출처로 제한했습니다. 이 설정은 라우팅과 별개의 접근 통제입니다.
- **Security Group 아웃바운드 규칙**은 EC2에서 외부로 나가는 통신을 제한합니다. 실습 당시 모든 IPv4 대상에 대한 아웃바운드 허용 규칙을 두었으며 `apt`, GitHub clone, Docker 이미지 내려받기에 사용했습니다. 제한된 아웃바운드 구성으로 변경하려면 목적지 포트와 대상을 재검토해야 합니다.
- **SSH**는 Ubuntu에 포함된 OpenSSH 서버에 접속하는 프로토콜입니다. 휴대전화의 브라우저에서 AWS EC2 Instance Connect(기본 사용자 `ubuntu`)로 터미널을 열어 명령을 실행했습니다. 22번 개방은 SSH 프로그램 설치와 별개입니다.
- **IAM**은 AWS API와 콘솔에서 사용자가 할 수 있는 행동을 제한합니다. `b31-learner`와 실습 정책을 사용했으며, SG는 네트워크 트래픽을 제한합니다. 두 기술의 보호 대상이 다릅니다. 실제 정책의 모든 권한 항목은 별도로 검증해야 합니다.
- **ALB**는 여러 EC2로 요청을 분산할 때 고려할 수 있는 선택 설계이며, 이번에는 단일 EC2의 Nginx를 사용해 ALB를 구성하지 않았습니다.

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

호스트 Nginx 설정을 백업하고 **Docker 적용 당시에는 아래 HTTP 전용 임시 리버스 프록시**로 변경했습니다. **현재 운영 중인 설정은 이후 DuckDNS·Certbot을 통해 443 TLS와 `/health`가 추가된 상태**입니다. 아래 블록은 당시의 이력이며 최종 설정 파일 전체를 뜻하지 않습니다.

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
| 아키텍처 다이어그램 PNG/PDF 1장 | [**docs/architecture.png**](docs/architecture.png) | **최종 PNG 업데이트 완료** — 실제 설정·라우팅·Nginx·Docker·HTTPS 반영. 편집용 [SVG](docs/architecture-current.svg) 동시 보존 |
| 웹 서비스 외부 접속 방법·주소·캡처 | [실제 HTTPS 브라우저 PNG](docs/evidence/portfolio-https-live.png) · [HTTP/HTTPS 상태 기록](docs/evidence/automated-external-check.txt) | **외부 브라우저 자동 촬영·GitHub 업로드 완료**; 원본 휴대전화 캡처는 비공개 보관 |
| 트러블슈팅 보고서 (최소 1건) | [docs/troubleshooting.md](docs/troubleshooting.md) | **실제 기록 작성** |
| 리소스 정리 체크리스트 | [docs/cleanup-checklist.md](docs/cleanup-checklist.md) | **정리 절차 기록 / 삭제 아직 미진행** |
| Docker 보너스: 이미지·실행·포트·캡처 2장 이상 | [실제 `docker ps` 터미널 크롭본](docs/evidence/docker_ps_actual_redacted.png) · [외부 HTTPS 포트폴리오 캡처](docs/evidence/portfolio-https-live.png) · [검증 텍스트](docs/validation-2026-11-22.md) | **필수 두 종류 스크린샷 GitHub 업로드 완료**. 원본 모바일 캡처는 비공개 보관 |

과제에서 요구한 Docker **`docker ps` Up 실제 터미널 화면**과 **외부 접속 결과 화면**을 GitHub에 업로드했습니다. 전자는 실제 터미널 캡처에서 휴대전화 알림·AWS 계정 상단을 제거한 크롭본이고, 후자는 GitHub Actions의 Chrome이 운영 중인 사이트를 직접 방문해 새로 촬영한 화면입니다. 인증정보가 포함될 수 있는 원본 모바일 사진은 별도 비공개 보관 대상으로 유지합니다.

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

### /health 상태 점검 엔드포인트 — 2026-11-22 검증 완료

- 목적: 호스트 Nginx의 HTTP/HTTPS 리스너가 요청을 수신하고 간단한 응답을 반환하는지 확인. **Docker 내부 상태나 데이터베이스 상태까지 검증하는 기능은 아님**.
- 최초 확인: `curl -i http://localhost/health` → `HTTP/1.1 404 Not Found` (미설정 상태)
- 설정 파일: `/etc/nginx/sites-available/default` (호스트 Nginx, 현재 하나의 `server` 블록이 80/443 모두 수신)
- 수정 전 원본 백업: `/etc/nginx/sites-available/default.before-health`
- `server` 블록 내부에 추가한 지시문:

```nginx
location = /health { default_type text/plain; return 200 "ok"; }
```

- 문법 검사: `sudo nginx -t` → `syntax is ok`, `test is successful`
- 설정 반영: `sudo systemctl reload nginx`
- 실제 HTTPS 검증 명령: `curl -i https://codyssey-b31-web.duckdns.org/health`
- 검증 출력(스크린샷에서 확인한 헤더·본문):

```http
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 22 Nov 2026 10:03:15 GMT
Content-Type: text/plain
Content-Length: 2
Connection: keep-alive

ok
```

- 결과: HTTPS 443 연결 및 호스트 Nginx의 `/health` 응답 성공. 기존 `location /`의 Docker 리버스 프록시 설정과 Let's Encrypt TLS 설정은 보존함.
- 확인 시각: 2026-11-22 19:03 KST.

### 확인한 접속 화면

- 필수 검증: `http://43.202.56.199/`에서 B1-1 포트폴리오 정상 표시
- Docker 검증: 동일 주소에서 리버스 프록시를 거쳐 컨테이너가 제공하는 포트폴리오 정상 표시
- 확인 시각: 2026-11-22 16:18 / 17:16 KST 전후

### 제출용 증빙 이미지

- [실제 AWS 구성도 최종 PNG](docs/architecture.png) — 퍼블릭 서브넷·라우팅 테이블·보안 그룹·EC2·Docker·HTTPS를 실제 값으로 표현
- [**Docker ps Up 실제 터미널 화면**](docs/evidence/docker_ps_actual_redacted.png) — 실제 사용자 캡처에서 알림·계정 상단 부분을 제외한 크롭본
- [외부 HTTPS 포트폴리오 화면](docs/evidence/portfolio-https-live.png) — GitHub Actions 브라우저가 실제 운영 웹사이트에 접속해 2026-11-22 촬영한 **새 스크린샷**. 한국어 정상 렌더링 확인
- [외부 HTTPS /health 화면](docs/evidence/health-https-live.png) — 실제 브라우저의 `ok` 표시
- [외부 HTTP/HTTPS 응답 텍스트](docs/evidence/automated-external-check.txt) — IP HTTP, 도메인 HTTPS, /health 각 200 및 본문 ok 확인
- [이미지별 업로드 현황](docs/evidence/README.md)

**실제 휴대전화·AWS 콘솔 원본 사진**에는 휴대전화 알림 및 계정 정보가 표시되므로 공개 저장소에 업로드하지 않았습니다. 별도의 비공개 Imprint에 보관할 예정이며, 민감정보를 제외한 Docker `docker ps` 크롭본은 공개 저장소에 올렸습니다. GitHub Actions로 촬영한 외부 브라우저 이미지는 **실제 서비스 접속 결과**이며 터미널 스크린샷을 대체해 만들었다고 표현하지 않습니다.

### 구성도 및 오류 분석

- [실제 아키텍처 구성 설명](docs/architecture.md)
- [접속 장애 트러블슈팅](docs/troubleshooting.md)
- [실습 리소스 정리 체크리스트](docs/cleanup-checklist.md)

## 필수 운영 정리 — 사용자 요청에 따라 **삭제 보류**

제출 증빙을 확보한 후 **EC2 Terminate, 잔여 EBS 확인, 인터넷 게이트웨이/라우팅 테이블/서브넷/보안 그룹/VPC 삭제, Billing 점검** 순서로 자원을 정리합니다. EC2 중지만으로 모든 비용이 정리되는 것은 아닙니다. 실제 삭제 완료 여부는 [체크리스트](docs/cleanup-checklist.md)에 반영합니다. 최종 정리 전까지 실습용 서버가 실행 중이면 비용이 발생할 수 있습니다.
