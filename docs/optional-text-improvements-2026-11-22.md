# B3-1 사전평가 100% 이후 선택적 문서 보강 — #7·#8·#12·#14·#15·#16

**근거:** 코디세이 AI 사전평가 **2026-11-22 20:47:35 KST, 16/16 PASS**의 보완 권고. 이 문서는 **과제 통과를 위해 필요한 추가 작업이 아니라 선택적 설명 보강**이다. 실측 정보와 제안 명령·설계 예시를 구별한다. **이 문서를 추가하면서 AWS 자원이나 보안 그룹, IAM 권한은 변경하지 않았다.**

## #7 보안 그룹 CIDR별 허용/차단 판단표

실제 보안 그룹: `b31-web-sg` (`sg-04bc2bb098cb7bbd1`).

| 포트·대상 | 출처 | 판단 | 근거와 실제 적용 여부 |
|---|---|---|---|
| TCP 80 / HTTP | `0.0.0.0/0` | 허용 | 외부 브라우저가 포트폴리오에 접근해야 하므로 **실제 적용** |
| TCP 443 / HTTPS | `0.0.0.0/0` | 허용 | 공개 웹사이트의 TLS 접속이므로 **실제 적용** |
| TCP 22 / 직접 SSH | 특정 개인 IPv4 `/32` | 허용 | 한 개의 관리용 IPv4로 제한, **실제 적용**. 원문 IP는 공개 저장소에서 마스킹 |
| TCP 22 / EC2 Instance Connect | AWS 관리형 EIC 접두사 목록 | 허용 | 콘솔에서 SSH 연결을 위한 **실제 적용**. `pl-` 전체 ID는 추후 화면 확인 가능 |
| TCP 22 / 임의의 인터넷 | `0.0.0.0/0` | 허용하지 않음 | 공개 SSH 허용 규칙은 없음 |
| TCP 3306 / MySQL, 5432 / PostgreSQL | `0.0.0.0/0` | 허용하지 않음 | DB를 배포하지 않았으며 외부 공개 규칙 없음 |
| TCP 8080 / Docker 호스트 포트 | `0.0.0.0/0` | 허용하지 않음 | Docker가 `8080:80`으로 실행되지만 SG 외부 인바운드 8080 규칙은 추가하지 않음 |

**해석:** 보안 그룹은 **상태 저장(stateful)**이며 인바운드에서 허용된 연결의 응답 트래픽은 보통 별도의 인바운드 허용 규칙 없이 반환될 수 있다. '차단'은 **해당 인바운드 허용 규칙이 없다는 뜻**이다. 서버의 모든 방화벽/NACL 규칙까지 검증했다고 주장하지 않는다. [기존 SSH 실측 설명](ssh-security-verification-2026-11-22.md).

## #8 선택한 A 방식: 한 문단 명령 요약

실습에서는 **A: 외부 브라우저에서 `http://43.202.56.199/`의 실제 포트폴리오 표시 확인**을 선택했다. HTTP 80을 열고, 호스트 Nginx가 `127.0.0.1:8080`으로 프록시하도록 구성했으며, Docker `nginx:alpine`에서 정적 콘텐츠를 제공했다. `sudo systemctl is-active nginx`와 `sudo docker ps`로 실행을 확인하고, `curl -I http://localhost/`, `curl -I http://localhost:8080/`의 **기존 실측값 `HTTP/1.1 200 OK`**를 기록했다. 외부에서는 브라우저 실제 렌더링 및 [GitHub Actions 외부 HTTP 200 검증](evidence/automated-external-check.txt)을 확인했다. 보너스 HTTPS `https://codyssey-b31-web.duckdns.org/`도 실제 화면으로 확인했다.

```sh
# 아래 명령은 평가 전 반복 점검용 요약이며, 이미 수행한 명령과 새 실행을 구별한다.
sudo systemctl is-active nginx          # 기존 확인: active
sudo docker ps                           # 기존 확인: b31-docker Up
curl -I http://localhost/                # 기존 확인: HTTP/1.1 200 OK
curl -I http://localhost:8080/           # 기존 확인: HTTP/1.1 200 OK
curl -I http://43.202.56.199/            # 외부에서 재검증 시 200 기대, 공인 IP 변경 가능
curl -i https://codyssey-b31-web.duckdns.org/health  # 기존 확인: 200, 본문 ok
```

## #12 웹사이트를 개인용으로 바꾸는 경우의 IP 허용 목록 절차 (미적용)

이 절차는 **대안 설명**이며 현재 외부 공개 웹사이트의 TCP 80·443은 그대로 유지한다.

1. 개인용 서비스로 변경할지 목적을 정한다. 외부 공개 포트폴리오라면 HTTP 80·HTTPS 443 `0.0.0.0/0`을 유지하는 것이 자연스럽다.
2. 허용할 관리자의 **현재 공인 IPv4**를 확인하고, 필요하다면 한 주소 `x.x.x.x/32` 또는 승인된 조직 네트워크의 CIDR을 결정한다. 개인 주소 원문은 공개 보고서에 남기지 않는다.
3. AWS EC2 → 보안 그룹 → `b31-web-sg` → **인바운드 규칙 편집**에서 개인용으로 전환할 포트의 출처를 `0.0.0.0/0`에서 정해진 CIDR로 바꾸는 방안을 계획한다.
4. **허용 IP 안과 밖에서 각각 접속**해 허용/차단을 검증하고, 현재 연결을 잃을 경우를 대비하여 별도 접속 수단을 준비한다. 관리 SSH 출처 규칙과 웹 공개 규칙을 혼동하지 않는다.
5. 작업 시간과 변경 전후 CIDR을 기록하고 필요 시 원상 복구한다. **현재 AWS에 이 변경을 적용하지 않았다.**

## #14 장애 점검 단계별 명령과 기대 결과 (예시)

| 순서 | 점검 대상 | 실행할 수 있는 명령·위치 | 정상 기대 결과 / 해석 |
|---|---|---|---|
| 1 | AWS 퍼블릭 라우트 | `aws ec2 describe-route-tables --route-table-ids rtb-00e3c2f405b2e1ea0 --region ap-northeast-2` (AWS CLI 인증 필요) | 기존 콘솔 실측: `10.10.0.0/16 → local`, `0.0.0.0/0 → igw-09322346b5d9055a8`, Active, 퍼블릭 서브넷 연결 |
| 2 | EC2 보안 그룹 | `aws ec2 describe-security-groups --group-ids sg-04bc2bb098cb7bbd1 --region ap-northeast-2` (AWS CLI 인증 필요) | TCP 80/443 공개, TCP 22 출처 제한; 보안 그룹과 네트워크 ACL도 필요 시 구분 |
| 3 | 서버의 기본 게이트웨이 | `ip route` (EC2 Ubuntu) | 기존 실측: `default via 10.10.0.1 dev ens5`; **이 Linux 결과는 AWS 라우팅 테이블을 대신하지 않는다** |
| 4 | 공인 IP/DNS | `getent hosts codyssey-b31-web.duckdns.org` (EC2/PC) 및 AWS EC2 상세 주소 비교 | DuckDNS가 **현재** 공인 IPv4를 가리키는지 비교; `43.202.56.199`는 실습 당시 주소 |
| 5 | 호스트 Nginx | `sudo systemctl is-active nginx`, `sudo ss -lntp`, `sudo nginx -t`, `curl -I http://localhost/` | 기존 실측: `active`, 80/443 리슨, 설정 문법 정상, HTTP 200 |
| 6 | Docker 프록시 | `sudo docker ps`, `curl -I http://localhost:8080/`, `sudo docker logs --tail 20 b31-docker` | 기존 실측: 컨테이너 Up, 8080 응답 200, 정적 파일 요청 200 |
| 7 | TLS/브라우저 | `curl -i https://codyssey-b31-web.duckdns.org/health`, `sudo certbot certificates` | 기존 실측: HTTPS 200 + `ok`, 인증서 만료 2027-02-20 |

**명령 일부는 재점검용 예시**다. 위에 적힌 모든 AWS CLI 명령을 실제 실행했다고 표현하지 않으며, CLI 권한이 없으면 이미 확인한 콘솔 기록을 근거로 쓴다. [실제 장애 처리 기록](troubleshooting.md) · [실측 검증 결과](validation-2026-11-22.md).

## #15 실제 정책을 기준으로 한 단계적 권한 축소 시나리오 (미적용)

관리자 화면에서 확인한 실습 정책 `B31PracticePolicy`에는 `ec2:Describe*` 및 EC2/VPC 생성·중지·종료·삭제 작업이 포함되어 있다. `Resource: "*"`, `aws:RequestedRegion: ap-northeast-2` 조건을 확인했다. [정책 전체 JSON](iam-policy-json-review-2026-11-22.md).

1. **조회 단계**: 실습 관찰만 할 때는 `ec2:Describe*` 등 조회 권한부터 부여할 수 있다. 단, Describe 와일드카드 자체도 대상 작업이 넓으므로 가능한 API를 별도로 좁혀 검토한다.
2. **구축 단계**: VPC·서브넷·IGW 생성이 필요한 때에만 `CreateVpc`, `CreateSubnet`, `CreateInternetGateway` 및 필수 연결·라우트 작업을 추가한다.
3. **배포 단계**: EC2 시작 및 접속이 필요해지면 `RunInstances`, `StartInstances`, `ec2-instance-connect:SendSSHPublicKey` 등 실제 작업을 허용한다. EC2 시작에는 AMI·네트워크 인터페이스·볼륨 등 연관 리소스 조건도 검토한다.
4. **종료 단계**: `TerminateInstances`, `DeleteVpc`, `DeleteSubnet` 등 강한 삭제 권한은 **정리 작업 시에만 한시적으로 부여**하거나 분리된 정리 역할을 사용하도록 설계할 수 있다.
5. **축소 검증**: `AccessDenied` 오류에 나타난 API, ARN, 리전 조건을 확인하고 필요한 한 동작만 허용한다. API별로 리소스 ARN과 조건키 지원 범위가 다르므로 개별 검토가 필요하다.

**실제 확인된 제한 사례:** 실습 사용자 `b31-learner`에서 `iam:ListPolicies` 접근 거부. **현행 정책은 수정하지 않았다.** 이 정책이 어떤 IAM 엔터티에 연결돼 있는지는 별도 확인 대상이다.

## #16 확장 판단을 위한 지표와 수집 주기 — 설계 예시 (미적용)

| 관찰 지표 | 수집 위치 | 제안하는 확인 주기 | 확대 판단에 쓰는 방법 |
|---|---|---|---|
| CPUUtilization | CloudWatch EC2 | 5분 단위 기본 모니터링(계정·구성에 따른 확인 필요) | 장시간 높은 CPU 사용이 지속되면 크기 조정·확장 검토 |
| CPUCreditBalance | CloudWatch EC2 T 계열 | 5분 | 버스트 크레딧 고갈과 성능 저하의 관계 확인 |
| 메모리 사용률 | Ubuntu `free -h` / 별도 CloudWatch Agent | 장애 시 즉시, 운영한다면 5~15분 | 메모리 압박과 OOM 흔적 확인; **EC2 기본 CloudWatch만으로 메모리 지표가 자동 수집되는 것은 아님** |
| 웹 응답 시간 | Nginx access log에 `$request_time` 로깅 추가 시 측정 가능 | 5분 집계, 일 단위 추세 | p95 지연 지속 상승 시 병목 분석 |
| HTTP 5xx 비율 | Nginx access log (요청 수와 5xx 수) | 5분 집계, 일 단위 추세 | HTTP 5xx 증가 시 애플리케이션·프록시·업스트림 점검 |
| HTTPS /health 가용성 | 외부 `curl`/주기적 모니터링 | 운영한다면 5분 | 실패 반복 시 호스트 Nginx·TLS·도메인 점검 |
| NetworkIn/Out·디스크 | CloudWatch EC2, Linux `df -h` | 5~15분/일별 | 자원 포화·외부 전송료 증가 여부 추적 |

**확장 구조의 예시:** `인터넷 → ALB → Target Group → EC2 2대 이상`, 필요하면 Auto Scaling 검토. ALB의 `TargetResponseTime`, `HTTPCode_Target_5XX_Count`는 **ALB를 도입한 다음에만** 관측할 수 있다. 현재는 **단일 EC2·Nginx·Docker 구성**이며 ALB와 CloudWatch Agent·집계 자동화를 설치했다는 기록은 없다. 추가 서비스와 상세 모니터링은 비용을 확인하고 도입해야 한다.

---

## 완료 상태의 경계

- **사전평가:** 16/16 PASS, 100%.
- **위 6개 추가 설명:** 문서화 완료. 새로운 AWS 작업을 실행한 것으로 표시하지 않음.
- **콘솔 원본 캡처·IAM 연결 엔터티·실제 태그·net-export 원본:** 별도 근거 확인 필요.
- **리소스 정리:** 실제 삭제 후 결과 증빙이 남은 별도 작업. [정리 체크리스트](cleanup-checklist.md).
