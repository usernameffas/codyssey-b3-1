# B3-1 IAM 고객 관리형 정책: B31PracticePolicy 실제 JSON 검증

검증 시점: **2026-11-22 20:38 KST 이후**, 사용자 제공 관리자 IAM 콘솔 스크린샷과 복사한 **전체 JSON 원문**을 대조했다. 실제 정책 내용은 아래와 같다. 정책 전체 JSON을 확인한 단계이며, 정책을 변경하지 않았다.

## 1. 실제 정책 원문

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "B31Practice",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "ec2:RunInstances",
                "ec2-instance-connect:SendSSHPublicKey",
                "ec2:StartInstances",
                "ec2:GetSecurityGroupsForVpc",
                "ec2:StopInstances",
                "ec2:TerminateInstances",
                "ec2:RebootInstances",
                "ec2:CreateTags",
                "ec2:CreateKeyPair",
                "ec2:DeleteKeyPair",
                "ec2:CreateVolume",
                "ec2:DeleteVolume",
                "ec2:CreateVpc",
                "ec2:DeleteVpc",
                "ec2:ModifyVpcAttribute",
                "ec2:CreateSubnet",
                "ec2:DeleteSubnet",
                "ec2:ModifySubnetAttribute",
                "ec2:CreateInternetGateway",
                "ec2:AttachInternetGateway",
                "ec2:DetachInternetGateway",
                "ec2:DeleteInternetGateway",
                "ec2:CreateRouteTable",
                "ec2:DeleteRouteTable",
                "ec2:CreateRoute",
                "ec2:DeleteRoute",
                "ec2:AssociateRouteTable",
                "ec2:DisassociateRouteTable",
                "ec2:CreateSecurityGroup",
                "ec2:DeleteSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:ModifySecurityGroupRules"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-northeast-2"
                }
            }
        }
    ]
}
```

## 2. 허용된 권한을 정확히 설명

- `Effect: Allow`: 아래 Action을 **정책 문장 차원에서 허용**한다. 실제 최종 권한은 다른 정책, SCP, permission boundary, 명시적 Deny 등이 있으면 영향을 받을 수 있다.
- `ec2:Describe*`: EC2의 `Describe`로 시작하는 **다양한 조회 API**에 일괄 매치한다. `iam:ListPolicies`까지 포함하는 패턴이 아니다.
- `ec2:RunInstances`, `StartInstances`, `StopInstances`, `RebootInstances`, **`TerminateInstances`**: EC2 인스턴스를 생성·시작·중지·재시작·종료할 수 있도록 허용하는 Action.
- `ec2-instance-connect:SendSSHPublicKey`: EC2 Instance Connect의 SSH 공개키 전달 관련 API Action.
- `ec2:CreateTags`: 리소스 태그 생성 권한. **이 액션이 허용되어 있다고 실제로 모든 리소스에 태그가 적용됐다는 뜻은 아니다.**
- `ec2:CreateVpc`, `DeleteVpc`, `CreateSubnet`, `DeleteSubnet`, `CreateInternetGateway`, `DeleteInternetGateway`, `AttachInternetGateway`, `DetachInternetGateway`, 라우팅 테이블 생성·삭제·연결 및 보안 그룹 생성·삭제·인바운드/아웃바운드 규칙 편집 등 **네트워크 리소스 관리 권한**이 포함된다.
- **전체 Action 수: 36개 항목.** `ec2:Describe*`는 하나의 Action 문자열이지만 여러 Describe API를 포괄하므로 **정확히 36개 AWS API만 허용한다는 뜻은 아니다.**
- `Resource: "*"`: 해당 문장에서 **개별 인스턴스 ID나 VPC ID/ARN으로 자원 범위를 한정하지 않았다.** `RunInstances`, `Describe*` 등은 Action별 IAM 리소스 범위 지원이 다르므로 정확한 세분화에는 작업별 검토가 필요하다.
- `StringEquals / aws:RequestedRegion: ap-northeast-2`: **서울 리전에서 처리되는 요청만** 허용하는 조건. 리전 제한이지 `b31-vpc` 같은 **특정 리소스 한 개로 한정하는 조건은 아니다.**

## 3. 최소 권한 원칙에 대한 현재 평가

이 정책은 **과제에 필요한 EC2·VPC 구성 작업을 서울 리전으로 한정**했으며 `iam:*`, `s3:*`, `rds:*` 등을 Action에 광범위하게 추가하지 않았다. 그러나 **리소스가 `"*"`이고 삭제·종료 권한까지 허용**하므로 단일 EC2 `b31-web` 또는 단일 VPC에만 엄격히 한정한 최소 권한 정책으로 표현하기는 어렵다. '실습용으로 서비스·리전을 한정한 권한'이 정확하다.

### 실제로 검증한 거부 사례

별도 실습 사용자 `b31-learner`로 IAM 정책 목록을 요청했을 때 `iam:ListPolicies`에 대해 **`no identity-based policy allows the action`** 오류가 났다. 이 정책의 Action에도 `iam:ListPolicies`가 포함되어 있지 않으므로 **해당 거부는 정책의 제한적 의도와 부합한다**. 다만 **정책이 `b31-learner`에 직접 연결됐는지**, 다른 정책의 유효 권한과의 합계는 **연결된 엔터티(Attached entities) 화면 확인 전**이다.

## 4. 정책과 네트워크 보안 그룹의 역할 차이

IAM은 AWS API **작업 권한**을 관리한다. EC2 보안 그룹 `b31-web-sg`는 웹 TCP 80·443 및 관리 SSH TCP 22의 **네트워크 허용 출처**를 관리한다. IAM의 `ec2:TerminateInstances`가 허용돼도 SSH 22 인바운드 접근 범위를 바꾸는 효과는 없다.

## 5. 남은 확인과 실제 운영 지시

- `B31PracticePolicy`의 **전체 JSON 확인 완료**. 이제 관리자 IAM의 **연결된 엔터티**에서 `b31-learner` 연결 여부만 확인하면 부착 증빙을 강화할 수 있다.
- 과제 진행 중 삭제 권한이 정책에 포함돼 있으므로, **EC2 종료 및 VPC 등 AWS 리소스 삭제는 사용자 별도 지시까지 보류**한다.
- 정책 수정·태그 추가·리소스 삭제는 이 문서 작성 과정에서 수행하지 않았다.
