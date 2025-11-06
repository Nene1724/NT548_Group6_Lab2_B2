# NT548 LAB02 B2 - CloudFormation + CodePipeline

Hạ tầng mạng hai tầng (Public/Private) cùng quy trình CI/CD tự động được hiện thực bằng AWS CloudFormation và AWS CodePipeline. Thư mục này kế thừa kiến trúc LAB02_B1 (Terraform) và chuẩn hóa lại bằng CloudFormation cho yêu cầu của nhóm.

## Thành phần chính

- **`main.yaml`** – Tạo VPC, subnets, NAT, bảo mật, EC2 bastion + private app và VPC Flow Logs mã hóa bằng KMS.
- **`codepipeline.yaml`** – Pipeline 3 stage (Source → Build/Validate → Deploy) dùng CodeStar Connection, CodeBuild và CloudFormation.
- **Scripts PowerShell** (`deploy.ps1`, `delete.ps1`, `deploy-pipeline.ps1`, `test-infrastructure.ps1`, `ssh-connect.ps1`) giúp thao tác nhanh với AWS CLI.
- **`buildspec.yml`** – Cấu hình CodeBuild chạy cfn-lint và `aws cloudformation validate-template`.
- **Tài liệu** – QUICK_START, DEPLOYMENT_GUIDE, DEPLOYMENT_CHECKLIST, PROJECT_SUMMARY, COMPLETION_SUMMARY.

## Chuẩn bị

1. AWS CLI v2, quyền IAM tạo CloudFormation/CodePipeline/CodeBuild.
2. PowerShell 7+, Python 3 cùng `pip`.
3. Key pair EC2 (`aws ec2 create-key-pair`). Cập nhật giá trị `KeyName` và `OperatorIP` trong `parameters.json`.
4. Nếu dùng GitHub: tạo AWS CodeStar Connection (`aws codestar-connections create-connection`). Dán ARN vào `codepipeline-parameters.json`.

## Các bước nhanh

1. **Kiểm tra mẫu**: `.\test-infrastructure.ps1`.
2. **Triển khai hạ tầng**: `.\deploy.ps1 -StackName nt548-lab02-network`.
3. **Triển khai pipeline**: cập nhật `codepipeline-parameters.json`, sau đó `.\deploy-pipeline.ps1`.
4. **SSH vào bastion**: `.\ssh-connect.ps1 -KeyFile .\nt548-lab02-key.pem`.
5. **Xoá tài nguyên**: `.\delete.ps1`.

Chi tiết hơn xem `QUICK_START.md` và `DEPLOYMENT_GUIDE.md`.
