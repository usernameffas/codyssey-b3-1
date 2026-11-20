# B3-1 웹 배포 학습 교재

대상: B1-1 포트폴리오를 만들었고 서버 배포를 배우는 단계. 실제 AWS 리소스를 생성하지 않고 학습할 수 있습니다.

## 1. 전체 구조

브라우저 → DNS(도메인을 IP 주소로 변환) → 인터넷/보안 그룹/라우팅 → EC2 → Nginx → HTML·CSS·JavaScript

- GitHub: 코드를 기록·공유하는 원격 저장소
- 배포: 사용자가 웹사이트에 접속할 수 있게 서비스를 제공하는 과정
- EC2: AWS에서 운영하는 가상 서버
- VPC: AWS에서 구성하는 논리적인 네트워크
- 서브넷: VPC 안의 IP 주소 구역
- Internet Gateway: VPC에서 인터넷으로 연결하는 관문
- Route Table: 패킷이 이동할 경로를 지정하는 규칙
- Security Group: 서버 통신의 허용 조건
- Nginx: 정적 파일을 전달하거나 다른 서버에 요청을 넘기는 웹서버
- Docker: 분리된 컨테이너 환경에서 프로그램을 실행
- HTTPS: TLS로 통신을 암호화하는 HTTP

## 2. 설정 파일 읽기

`config/site-http.conf`의 `listen 80`은 HTTP 요청을 받을 포트, `root /var/www/b31-site`는 파일 저장 위치, `index index.html`은 기본 문서, `try_files`는 해당 파일을 찾지 못하면 404를 반환하도록 지정합니다.

`config/site-docker.conf`의 `proxy_pass http://127.0.0.1:8080;`은 같은 서버에서 실행되는 컨테이너 쪽으로 요청을 전달합니다.

`127.0.0.1`은 명령을 실행하는 컴퓨터 자신을 나타냅니다. 따라서 서버에서 localhost로 접속할 수 있다는 사실만으로 외부 인터넷 접속까지 검증되지는 않습니다.

## 3. 명령어 해석

```bash
git clone https://github.com/usernameffas/codyssey-b1-1.git
sudo nginx -t
curl -I http://localhost
sudo systemctl reload nginx
sudo docker ps
```

차례대로 저장소 복제, Nginx 설정 구문 검사, HTTP 응답 헤더 확인, Nginx 설정 재적용, 실행 중인 컨테이너 목록 확인입니다. `docs/manual.md`의 명령은 EC2 서버에 접속한 다음 실행하는 부분이 많으니 터미널의 위치를 반드시 구분하세요.

## 4. 오류 진단

- 서버의 `curl -I http://localhost`는 200인데 외부 PC에서 접속 실패: 퍼블릭 IP, 보안 그룹 80번 규칙, 서브넷 라우팅, Internet Gateway를 확인합니다.
- Docker 전환 후 502 오류: `curl -I http://127.0.0.1:8080`, `sudo docker ps`, 컨테이너 로그와 Nginx 설정을 점검합니다.
- 파일 경로에서 404: 배포된 파일의 실제 경로, Nginx `root`, `try_files`를 확인합니다.

## 5. 스스로 풀어보는 문제

1. GitHub에 코드를 올리는 것과 웹사이트를 배포하는 것은 어떻게 다른가요?
2. VPC와 서브넷의 관계를 설명하세요.
3. HTTP 80, HTTPS 443, SSH 22의 역할을 설명하세요.
4. 왜 서버에서 localhost 200이 떠도 외부 접속은 실패할 수 있나요?
5. `nginx -t`가 성공했다는 사실은 무엇을 뜻하나요?
6. `proxy_pass`는 어떤 역할을 하나요?
7. Docker 컨테이너가 정지하면 502가 발생할 수 있는 이유는?
8. 리소스 삭제 후에도 AWS Billing을 확인해야 하는 이유는?

[정답과 해설](study-answers.md) · [실습 기록표](practice-log.md)

**비용 주의:** 기존 개인 AWS 계정에 무료 혜택이 적용되지 않을 수 있으며 퍼블릭 IPv4·EBS 등도 과금될 수 있습니다. 실습 환경의 요금 면제가 명확해질 때까지 AWS 리소스는 생성하지 않습니다.
