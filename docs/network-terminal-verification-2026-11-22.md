# Ubuntu 네트워크 주소·라우팅·수신 포트 검증 — 2026-11-22

**출처:** 사용자가 2026-11-22 20:19 KST에 제공한 EC2 Instance Connect **실제 터미널 화면**. 동일한 네트워크 출력은 19:19 KST에도 화면으로 확인됨. 이 문서는 스크린샷에 보이는 값만 텍스트로 옮긴다.

## 실행·출력 구분

20:19 화면 첫 줄의 명령어는 실제로 `cat ~/b31-network-validation.txtcat ~/b31-network-validation.txt`처럼 **두 명령이 이어 붙은 오타**이므로 `cat: ... No such file or directory` 오류가 나왔다. **파일 내용을 성공적으로 열었다는 증거로 기록하지 않는다.** 다만 그 아래에 `PRIVATE IP`, `ROUTING`, `LISTENING PORTS` 출력이 실제로 표시되며, 19:19 화면의 실행값과 일치한다. 아래 값은 **터미널 출력의 화면 전사**다.

## 1. PRIVATE IP

화면 전사:

```text
=== PRIVATE IP ===
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    altname enp0s5
    inet 10.10.4.1/20 metric 100 brd 10.10.15.255 scope global dynamic ens5
       valid_lft 3161sec preferred_lft 3161sec
```

- EC2 네트워크 인터페이스: `ens5`
- 사설 IPv4: `10.10.4.1/20`
- NIC 연결 주소 범위: `10.10.0.0/20`
- 별도의 **AWS VPC CIDR `10.10.0.0/16`**은 콘솔에서 확인했으며 NIC 프리픽스와 혼동하지 않는다.

## 2. ROUTING

화면에서 확인한 출력:

```text
=== ROUTING ===
default via 10.10.0.1 dev ens5 proto dhcp src 10.10.4.1 metric 100
10.10.0.0/20 dev ens5 proto kernel scope link src 10.10.4.1 metric 100
10.10.0.1 dev ens5 proto dhcp scope link src 10.10.4.1 metric 100
10.10.0.2 dev ens5 proto dhcp scope link src 10.10.4.1 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
```

`default via 10.10.0.1`은 **Ubuntu 인스턴스 내부 기본 게이트웨이**다. [AWS VPC 라우팅 테이블](network-console-verification-2026-11-22.md)의 `0.0.0.0/0 → Internet Gateway`와 별도 계층의 설정이다. `docker0` 브리지 내부 IPv4 범위는 `172.17.0.0/16`이다.

## 3. LISTENING PORTS

실제 `ss` 출력은 모바일 터미널 폭 때문에 행 중간이 접혀 있었다. 아래 표는 스크린샷에 표시된 프로세스와 주소·포트를 **열별로 옮긴 값**이며, `ss`의 전체 원문 행을 그대로 복사한 출력이라고 주장하지 않는다.

| 프로토콜 | 로컬 리슨 주소 | 프로그램 | 용도 |
|---|---|---|---|
| TCP IPv4 | `0.0.0.0:80` | `nginx` | 호스트 HTTP |
| TCP IPv4 | `0.0.0.0:22` | `sshd` / `systemd` | SSH |
| TCP IPv4 | `0.0.0.0:8080` | `docker-proxy` | Docker 컨테이너 80 포트 매핑 |
| TCP IPv4 | `0.0.0.0:443` | `nginx` | 호스트 HTTPS |
| TCP IPv6 | `[::]:22` | `sshd` / `systemd` | IPv6 SSH 리슨 |
| TCP IPv6 | `[::]:8080` | `docker-proxy` | IPv6 Docker 호스트 포트 매핑 |

**검증 한계:** EC2 내부의 `0.0.0.0:8080` 및 `[::]:8080` 리슨은 호스트 프로세스의 수신 상태이다. 실제 인터넷 접근 범위는 AWS 보안 그룹, 라우팅, 네트워크 ACL 등에도 좌우된다. **SSH 인바운드 허용 CIDR은 이 `ss` 출력만으로 확인할 수 없다.** 실제 규칙은 [AWS 보안 그룹 `b31-web-sg`](../README.md) 화면에서 별도 확인해야 한다.

## 원본 로그 확인

정확한 파일 내용 확인이 필요할 경우 **한 번만 입력**할 명령은 `cat ~/b31-network-validation.txt`다. 20:19 화면에서는 중복 입력 오류로 **원본 파일의 성공적인 `cat` 확인이 되지 않았다**. 다만 위 값은 사용자 제공 화면에 실제 표시된 출력에 근거한다.

서버·Docker·HTTPS 및 AWS 리소스의 설정을 변경하거나 삭제하지 않았다.
