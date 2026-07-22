# Tài liệu AWS

## I. Thông tin Server

### Public IP

```text
18.143.40.118
```

---

## II. EC2 Server

### SSH từ máy cá nhân

**Máy Admin**

```bash
ssh -i C:\Users\Admin\Downloads\ec2-server.pem ubuntu@18.142.182.251
```

**Máy huynq1**

```bash
ssh -i C:\Users\huynq1\Downloads\ec2-server.pem ubuntu@54.255.187.89
```

> **Lưu ý:** Cần đảm bảo file `ec2-server.pem` có quyền truy cập phù hợp và Security Group của EC2 đã mở cổng `22 (SSH)`.

---

# Tài liệu học AWS

## 1. Tài liệu chính thức

- 📘 AWS Documentation
- 📚 AWS Whitepapers
- 📖 AWS Well-Architected Framework
- 📝 AWS Blog

---

## 2. Khóa học

- AWS Skill Builder (Training chính thức của AWS)
- Udemy
- Coursera
- A Cloud Guru
- KodeKloud

---

## 3. Blog & Cộng đồng

- Medium
- Dev.to
- Hashnode
- Reddit (r/aws)
- Stack Overflow

---

## 4. Video

- YouTube
  - AWS Official
  - TechWorld with Nana
  - FreeCodeCamp
  - Abhishek Veeramalla
  - Anton Putra

---

## 5. Lộ trình học đề xuất

```text
AWS Fundamentals
        │
        ▼
IAM
        │
        ▼
EC2
        │
        ▼
VPC
        │
        ▼
EBS & EFS
        │
        ▼
S3
        │
        ▼
RDS
        │
        ▼
Load Balancer
        │
        ▼
Auto Scaling
        │
        ▼
Route53
        │
        ▼
CloudWatch
        │
        ▼
CloudFormation
        │
        ▼
ECS / EKS
        │
        ▼
Serverless (Lambda)
```
