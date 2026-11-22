# B3-1 자동평가용 텍스트 증빙 점검 — 2026-11-22

**목적:** 공개 GitHub 저장소에 이미 기록된 실제 AWS 설정·EC2 명령·응답을 검토해, AI 평가기가 이미지 없이도 읽을 수 있는 증빙을 만든다.

- **근거 출처:** 2026-11-22 실제 EC2 Instance Connect 터미널 캡처, AWS VPC 콘솔 화면, IAM 권한 거부 화면 및 그 결과를 기록한 GitHub 문서.
- **인용 원칙:** 출력 내용을 문자 단위로 전부 복원할 수 없는 경우 확인된 부분만 발췌하고, `[일부 생략]`이라고 표기한다. 실행하지 않은 명령을 실행한 것으로 쓰지 않는다.
- **보안:** 휴대전화 상태 표시줄, 카카오톡 알림, AWS 계정번호 등은 공개용 텍스트 증빙에서 제외한다. 인증 키·비밀번호·DuckDNS 토큰과 ACME private key의 내용은 공개하지 않는다.

## 1. AWS 네트워크 — 실제 콘솔 확인 (19:29 KST)

```text
Region: ap-northeast-2 (Seoul)
VPC Name: b31-vpc
VPC ID: vpc-03069500f8707f0f9
VPC CIDR: 10.10.0.0/16

Route Table Name: b31-rtb-public
Route Table ID: rtb-00e3c2f405b2e1ea0
Main: No
Explicit subnet association:
  b31-subnet-public1-ap-northeast-2a
  subnet-0814ca163de8b1b5f

Routes:
  Destination 10.10.0.0/16 -> Target local -> Active
  Destination 0.0.0.0/0   -> Target Internet Gateway b31-igw (igw-09322346b5d9055a8) -> Active
```

위 블록은 **콘솔 표의 값을 텍스트로 정리**한 것으로 CLI 출력이라고 주장하지 않는다. Internet Gateway는 **20:24 KST 추가 VPC 콘솔 목록에서 전체 ID `igw-09322346b5d9055a8`과 표시 이름 `b31-igw`를 확인**했다. 라우트 활성은 19:29 KST 라우팅 테이블 화면에서 별도로 확인한 것이다. 더 자세한 설명은 [network-console-verification-2026-11-22.md](network-console-verification-2026-11-22.md).

## 2. 원격 SSH 및 Ubuntu — 실제 연결 확인 (19:19 KST)

- AWS EC2 Instance Connect 웹 터미널에서 Ubuntu 계정 `ubuntu`로 접속했다. **로컬 PowerShell에서 PEM 파일로 수동 SSH 접속을 수행했다는 기록은 없다.**
- 접속한 EC2 인스턴스는 `b31-web` (`i-07336ff478ba4ef67`).
- Ubuntu 24.04.5 LTS 화면에서 `ubuntu@ip-10-10-4-1:~$` 프롬프트를 확인하고 이후 아래 명령을 실제 실행했다.

```bash
ip -4 addr show dev ens5
ip -4 route
sudo ss -lntp | grep -E ':(22|80|443|8080)\b'
```

화면에서 확인된 네트워크 출력의 IP 경로와 수신 포트 주소·프로세스별 확인표 (2026-11-22 20:19 화면과 대조함):

```text
inet 10.10.4.1/20 brd 10.10.15.255 scope global dynamic ens5
default via 10.10.0.1 dev ens5 proto dhcp src 10.10.4.1 metric 100
10.10.0.0/20 dev ens5 proto kernel scope link src 10.10.4.1 metric 100
10.10.0.1 dev ens5 proto dhcp scope link src 10.10.4.1 metric 100
10.10.0.2 dev ens5 proto dhcp scope link src 10.10.4.1 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1

IPv4 0.0.0.0:80   nginx
IPv4 0.0.0.0:22   sshd, systemd
IPv4 0.0.0.0:8080 docker-proxy
IPv4 0.0.0.0:443  nginx
IPv6 [::]:22      sshd, systemd
IPv6 [::]:8080    docker-proxy
```

> `LISTEN` 항목은 실제 화면의 주소·프로세스를 행별로 정리한 것이며 긴 `ss` 원문 행 그대로 복사한 문자열은 아니다. [네트워크 검증의 실제 화면 전사와 원본 파일 미확인 상태](network-terminal-verification-2026-11-22.md)를 참고한다. Ubuntu의 `10.10.4.1/20`은 EC2 NIC의 주소/프리픽스이고, AWS VPC의 `10.10.0.0/16`과 쓰임이 다르다. 앞의 리눅스 `default via 10.10.0.1`과 AWS 콘솔의 `0.0.0.0/0 → IGW`도 서로 다른 라우팅 계층의 기록이다.

## 3. Nginx와 Docker — 실제 종합 테스트 (19:16 KST)

실행:

```bash
sudo systemctl is-active nginx docker
sudo docker ps
curl -I --max-time 10 http://localhost:8080/
curl -I --max-time 10 http://localhost/
sudo nginx -t
```

출력에서 확인한 행:

```text
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
Content-Length: 10227

$ curl -I --max-time 10 http://localhost/
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Content-Length: 10227

$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

- 컨테이너 이미지: `nginx:alpine`, 이름: `b31-docker`.
- 실제 Docker 실행 명령 기록: `sudo docker run -d --name b31-docker -p 8080:80 -v /var/www/html:/usr/share/nginx/html:ro nginx:alpine`.
- 외부 80/443 → 호스트 Nginx → `127.0.0.1:8080` → Docker Nginx.
- 과제용 B1-1 정적 포트폴리오를 외부 모바일 브라우저에서 확인했다.
- **20:13~20:14 KST에 `cat ~/b31-validation.txt`를 실제 실행하여 저장된 7항목 전체를 재확인**했다. Docker 표는 터미널에서 열이 줄바꿈되어 표시돼 실제 값을 필드별로 재정리했다. [실제 Docker·Nginx·HTTPS 전체 HTTP 헤더 및 TLS 검증값](validation-2026-11-22.md)에 기록했다. 서버 파일을 그대로 업로드했다는 뜻은 아니다.

## 4. HTTPS/TLS와 상태 확인 — 실제 종합 테스트 (19:03~19:16 KST)

```bash
sudo certbot --nginx --no-redirect -d codyssey-b31-web.duckdns.org
curl -i https://codyssey-b31-web.duckdns.org/health
sudo certbot certificates
```

실제 화면에서 확인한 핵심 출력:

```text
Successfully received certificate.
Successfully deployed certificate for codyssey-b31-web.duckdns.org
Congratulations! You have successfully enabled HTTPS on https://codyssey-b31-web.duckdns.org

HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Content-Type: text/plain
Content-Length: 2

ok

Certificate Name: codyssey-b31-web.duckdns.org
Key Type: ECDSA
Expiry Date: 2027-02-20 08:01:17+00:00 (VALID: 89 days)
```

- `/health` 경로는 호스트 Nginx에서 `return 200 "ok"`로 직접 응답한다.
- `/health`의 200은 **호스트 Nginx 상태 확인**이며 Docker의 내부 동작은 3번의 별도 명령으로 검증한다.
- 외부 모바일 Chrome에서도 `https://codyssey-b31-web.duckdns.org/health`의 `ok` 표시를 확인했다.

## 5. IAM 제한 — 실제 콘솔 확인 (19:32 KST)

```text
IAM user: b31-learner
Action: iam:ListPolicies
Result: Access Denied
Reason: no identity-based policy allows the action
```

이 값은 **AWS IAM 콘솔 화면에서 확인된 오류 문구를 정리**한 것이며 `aws iam` 명령의 출력은 아니다.

해당 사용자는 IAM 정책 목록 조회 권한이 제한된다. `B31PracticePolicy`의 전체 JSON 및 다른 서비스 권한은 관리자 계정에서 따로 점검해야 한다.

## 6. 평가 항목과 증빙 위치

| 평가 항목 | 근거와 위치 | 현재 남은 부분 |
|---|---|---|
| #1 VPC·서브넷·IGW 경로 | 본 문서 1절, [네트워크 콘솔 확인](network-console-verification-2026-11-22.md) | IGW 전체 ID 확인 완료; 부착 상태(Attached) 화면은 필요시 별도 검증 가능 |
| #2 SSH와 호스트 HTTP 200 | 본 문서 2·3절 및 [실제 터미널 네트워크 전체 경로/수신 주소](network-terminal-verification-2026-11-22.md) | AWS SG의 정확한 SSH 인바운드 CIDR은 별도 확인 필요 |
| #3 보안 그룹 HTTP 80·SSH 22 | README 구성표, [실제 SSH 단일 IP /32와 관리형 EIC 출처 설명](ssh-security-verification-2026-11-22.md) | 기존 평가 PASS. 개인 IP 공개는 피하고 EIC 목록 전체 ID는 추가 조회 전 |
| **#4 외부 접속 방식 A 또는 /health** | 본 문서 3·4절, README의 접속 URL | **이 항목은 이전 보완표에서 SSH로 잘못 분류했으므로 정정** |
| #5 리소스 정리 및 삭제 증빙 | [cleanup-checklist.md](cleanup-checklist.md) | **사용자 요청으로 삭제 보류**, 완료로 표기하지 않음 |
| #9 자원 태그·이름 규칙 | [README 실제 이름 규칙](../README.md) | `b31-` 접두어로 리소스 식별, 별도 사용자 정의 태그 추가는 보류 |
| #6 실제 아키텍처 PNG/PDF | [최종 PNG](architecture.png), [편집 SVG](architecture-current.svg) | **최종 PNG 생성 및 GitHub 업로드 완료** |
| #7·#12 공개 포트의 위험·대안 | [assessment-explanations.md](assessment-explanations.md) | 실제 보안 그룹 전체 설정 원문은 추가 확인 가능 |
| #8 선택한 방식 A의 이유 | [assessment-explanations.md](assessment-explanations.md) | 설명 작성 완료 |
| #10 AWS 라우트와 외부 송신 | 본 문서 1·2절 | AWS outbound SG 상세는 추가 확인 가능 |
| #11·#15 IAM 최소 권한 | 본 문서 5절, [IAM 검증](iam-permission-verification-2026-11-22.md) | IAM 전체 정책 JSON 검증 필요 |
| #13·#14 오류·점검 절차 | [troubleshooting.md](troubleshooting.md), [assessment-explanations.md](assessment-explanations.md) | 로그 공유 시 민감 정보 제거 |
| #16 ALB·지표·비용 | [assessment-explanations.md](assessment-explanations.md) | 계획·설명 작성, 실제 ALB는 미설치 |

## 7. 추가 작업을 위한 확인 목록

- 공개 README에는 핵심 출력과 항목별 문서 링크가 들어 있어야 한다.
- **정확한 모든 원문 바이트**가 필요하면 EC2의 `~/b31-validation.txt`와 `~/b31-network-validation.txt`에 저장된 내용을 안전하게 검토한 뒤 별도 업로드한다. 원본 파일을 아직 GitHub에 업로드한 것으로 쓰지 않는다.
- 비공개 `Imprint`에 원본 스크린샷을 저장하고, 공개 저장소에는 식별정보·알림 등을 편집한 사본만 배치한다.
- 과제에서 명시한 PNG/PDF 구성도와 접속 캡처는 AI 텍스트 평가와 별개의 **최종 제출 요구**로 남는다.
- AWS 리소스는 사용자의 지시가 있기 전까지 삭제하거나 중지하지 않는다.
