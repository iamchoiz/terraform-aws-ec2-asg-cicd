# terraform-aws-ec2-asg-cicd

> Terraform으로 구성하는 **AWS EC2 ASG 기반 2-Tier(WEB/WAS) 아키텍처 + AWS CodeSeries CI/CD 파이프라인** 인프라 자동화 프로젝트

## Architecture

![Architecture](https://user-images.githubusercontent.com/77256060/166135736-495889dd-42bb-40f0-bc15-40f3c1c73871.png)

---

## Tech Stack

| Category | Tool |
|---|---|
| IaC | Terraform |
| Compute | AWS EC2, Auto Scaling Group |
| Storage | AWS EFS |
| CI/CD | AWS CodePipeline, CodeBuild, CodeDeploy |
| Networking | AWS VPC, ALB (WEB/WAS 각각) |

---

## 디렉토리 구조

```
.
├── vpc/               # VPC, Subnet (public / private / private-db × 2 AZ)
├── web-infra/         # WEB ASG, WEB ALB, Launch Template
├── was-infra/         # WAS ASG, WAS ALB, EFS, S3, Launch Template
└── aws-cicd-pipeline/ # CodePipeline, CodeBuild, CodeDeploy
```

---

## Terraform Workspace 구성

> VPC → Infra → AWS-CodeSeries 순서로 적용합니다.

| # | Workspace | 생성 리소스 |
|---|---|---|
| 1 | **VPC** | VPC, public/private/private-db Subnet × 2 AZ |
| 2 | **Infra** | WEB ASG, WAS ASG, WEB ALB, WAS ALB, EFS, Launch Template, S3 |
| 3 | **AWS-CodeSeries** | CodePipeline, CodeDeploy, CodeBuild |

---

## 인프라 구성 흐름

1. **VPC 생성** — public, private, private-db 서브넷 각각 2개 AZ에 생성
2. **EFS 생성** — WEB/WAS 공유 파일시스템
3. **Launch Template 생성** — Userdata: EFS 마운트 + ECS Agent 설치
4. **ASG + ALB 생성** — WEB 서버(public subnet) / WAS 서버(private subnet) 각각 구성
5. **EC2 초기화** — Userdata로 EFS 연결, WEB ↔ WAS Reverse Proxy 설정

---

## 사용 방법

```bash
# 1. 레포지토리 클론
git clone https://github.com/DACHANCHOI/terraform-aws-ec2-asg-cicd.git
cd terraform-aws-ec2-asg-cicd

# 2. 환경변수 설정 (aws_account_id 등)
vi vpc/_terraform.auto.tfvars

# 3. Terraform Backend 수정 (각 workspace의 _backend.tf)

# 4. 순서대로 프로비저닝
cd vpc && terraform init && terraform apply
cd ../was-infra && terraform init && terraform apply
cd ../web-infra && terraform init && terraform apply
cd ../aws-cicd-pipeline && terraform init && terraform apply
```
