# NT548 LAB02 B2 - CloudFormation + CodePipeline

Hạ tầng mạng hai tầng (Public/Private) và pipeline tự động hoá triển khai được cấu hình lại dựa trên hướng dẫn mục **B. Triển khai hạ tầng AWS với CloudFormation và tự động hoá quy trình build & deploy với AWS CodePipeline** của tài liệu lab. Template CloudFormation nhận giá trị IP vận hành `192.168.1.36/32` và key pair `nt548_group6`, đồng thời sửa lỗi UserData bằng `Fn::Base64` để tránh lỗi `Invalid BASE64 encoding of user data` gặp ở lần triển khai trước.

## Cấu trúc thư mục

- `main.yaml`: Tạo VPC, subnet công/tư, IGW, NAT Gateway, security group, IAM role/profile, EC2 bastion & private instance, VPC Flow Logs (mã hoá bằng KMS).
- `parameters.json`: Tham số mặc định cho hạ tầng (có Availability Zone và OperatorIP của môi trường lab).
- `codepipeline.yaml`: Pipeline 3 stage (Source → Build/Validate → Deploy) dùng CodeStar Connection, CodeBuild và CloudFormation.
- `codepipeline-parameters.json`: Giá trị tham số pipeline (ARN CodeStar connection, repo GitHub, stack đích, v.v.).
- `buildspec.yml`: BuildSpec cho CodeBuild chạy `cfn-lint`, `validate-template` và taskcat.
- `.taskcat.yml`: Tập tin gốc để CodeBuild sinh cấu hình taskcat động.

## Quy trình triển khai thủ công

1. **Dọn hạ tầng cũ (nếu còn)**  
   ```powershell
   aws cloudformation delete-stack `
     --stack-name nt548-lab02-network `
     --region ap-southeast-1
   aws cloudformation wait stack-delete-complete `
     --stack-name nt548-lab02-network `
     --region ap-southeast-1
   ```
   Kiểm tra lại bằng `aws cloudformation describe-stacks` (sẽ báo lỗi “does not exist” nếu đã xoá sạch).

2. **Triển khai hạ tầng mạng + EC2**  
   ```powershell
   aws cloudformation deploy `
     --stack-name nt548-lab02-network `
     --template-file main.yaml `
     --parameter-overrides file://parameters.json `
     --capabilities CAPABILITY_NAMED_IAM `
     --region ap-southeast-1 `
     --tags Project=nt548-lab02 Environment=dev
   ```
   Ghi lại các output quan trọng: `BastionPublicIp`, `BastionSshCommand`, `PrivateInstancePrivateIp`, `FlowLogsLogGroupName`.

3. **Triển khai pipeline CI/CD**  
   - Cập nhật lại `codepipeline-parameters.json` nếu đổi repo/branch/ARN.  
   - Chạy:
     ```powershell
     aws cloudformation deploy `
       --stack-name nt548-lab02-pipeline `
       --template-file codepipeline.yaml `
       --parameter-overrides file://codepipeline-parameters.json `
       --capabilities CAPABILITY_NAMED_IAM `
       --region ap-southeast-1 `
       --tags Project=nt548-lab02 Environment=dev
     ```
   Khi stack thành công, kiểm tra trên AWS Console (CodePipeline) để chụp màn hình pipeline và các stage cho báo cáo.

4. **Kiểm thử & Ghi nhận**  
   - Dùng `ssh -i nt548_group6.pem ec2-user@<BastionPublicIp>` để xác thực truy cập.  
   - Kiểm tra CloudWatch Logs `/aws/vpc/nt548-lab02/dev/flow-logs`.  
   - Kích hoạt pipeline bằng cách đẩy commit lên `main` rồi ghi lại trạng thái từng stage cho báo cáo.

5. **Xoá tài nguyên sau khi hoàn tất**  
   ```powershell
   aws cloudformation delete-stack --stack-name nt548-lab02-pipeline --region ap-southeast-1
   aws cloudformation delete-stack --stack-name nt548-lab02-network --region ap-southeast-1
   ```

Thực hiện theo trình tự trên sẽ tạo ra cùng kết quả như phần mô tả trong pdf, đồng thời phù hợp với môi trường hiện tại của nhóm (IP 192.168.1.36, key pair `nt548_group6`). Output từng bước có thể dùng để ghi vào báo cáo lab. 
