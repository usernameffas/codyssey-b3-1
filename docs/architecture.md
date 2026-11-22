# B3-1 실제 AWS 네트워크 구성도 — 2026-11-22

> 이 문서는 이미지 구성도의 검증용 텍스트 원본입니다. 과제 제출은 **PNG 또는 PDF 이미지 1장**이므로 `docs/architecture.png`를 실제 정보로 최종 수정해야 합니다.

```text
                 외부 Chrome / Samsung Internet
                      HTTP TCP 80
                            |
                          Internet
                            |
                   Internet Gateway
                            |
            VPC: b31-vpc (10.10.0.0/16)
                            |
         Public Subnet (ap-northeast-2a)
                            |
    Route table: b31-rtb-public
       10.10.0.0/16 --> local
       0.0.0.0/0    --> Internet Gateway
                            |
    Security Group: b31-web-sg
       Inbound HTTP TCP 80 and HTTPS TCP 443 from 0.0.0.0/0
       Inbound SSH TCP 22 from limited sources
                            |
    EC2: b31-web / Ubuntu 24.04 LTS / t3.micro
       Public IPv4: 43.202.56.199 (temporary)
       Private IPv4: 10.10.4.1
       EBS: gp3 8 GiB
                            |
                 Host Nginx :80 / :443
                 TLS certificate (Let's Encrypt)
                 Reverse proxy -> 127.0.0.1:8080
                            |
                   Docker port 8080:80
                  Container: b31-docker
                  Image: nginx:alpine
                            |
               Nginx in Docker container :80
               /usr/share/nginx/html (read-only)
                            |
              B1-1 static portfolio HTML/CSS/JS
```

- SSH 접속은 EC2 Instance Connect에서 `ubuntu` 계정을 사용해 확인했습니다.
- 정적 파일의 호스트 경로는 `/var/www/html`입니다.
- Docker는 `-v /var/www/html:/usr/share/nginx/html:ro`으로 기존 웹사이트 파일을 읽기 전용 연결했습니다.
- 외부 HTTP 접속은 실제로 검증했습니다.
- HTTPS 443번을 보안 그룹에서 열고 Let's Encrypt 인증서를 발급해 호스트 Nginx에 적용했습니다. Certbot 성공 출력 및 포트폴리오 브라우저 표시를 확인했습니다.
- 이 네트워크 구성은 실습 종료 후 삭제 대상입니다.
