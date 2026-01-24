# NT548.Q11 - Lab 2 - Exercise 2
## Group 09

|    MSSV   |      Họ và tên     |
|-----------|--------------------|
|  23520797 | Lê Trung Kiên      | 
|  23521588 | Trần Thị Thùy Tiên | 
|  23521471 | Trần Thuận Thến    | 
|  23521564 | Trần Lê Uyên Thy   | 

---

## 📖 Giới thiệu (Overview)

[cite_start]Bài tập này tập trung vào việc **triển khai hạ tầng AWS với CloudFormation** và **tự động hóa quy trình build và deploy với AWS CodePipeline**[cite: 47].

### 🎯 Yêu cầu
- [cite_start]Các dịch vụ phải được viết dưới dạng module[cite: 49].
- Phải đảm bảo an toàn bảo mật cho EC2 (thiết lập Security Groups).
- Sử dụng AWS CodeBuild tích hợp `cfn-lint` và `taskcat` để kiểm tra tính đúng đắn.
- Tự động hóa quy trình Build & Deploy từ AWS CodeCommit.

---

## 🏗 Cấu trúc thư mục (Project Structure)

[cite_start]Dự án sử dụng lại các module từ Lab 1 và bổ sung thêm các file cấu hình cho CI/CD[cite: 49, 50].

NT548-LAB2-EX2/
├── cloudformation/
│   ├── modules/                 # Các CloudFormation Templates thành phần
│   │   ├── vpc.yaml             # Định nghĩa VPC, Subnets
│   │   ├── nat-gateway.yaml     # Định nghĩa NAT Gateway
│   │   ├── route-table.yaml     # Định nghĩa Route Tables
│   │   ├── security-group.yaml  # Định nghĩa Security Groups
│   │   └── ec2.yaml             # Định nghĩa EC2 Instances
[cite_start]│   ├── .taskcat.yml             # Cấu hình kiểm thử Taskcat [cite: 51]
[cite_start]│   ├── buildspec.yml            # Script cho CodeBuild [cite: 52]
│   └── main.yaml                # Master stack gọi các nested stacks
└── README.md

## 🚀 Tự động hóa với AWS CodePipeline (CI/CD)

Quy trình CI/CD được xây dựng hoàn toàn trên AWS gồm 3 giai đoạn chính.

### [cite_start]Bước 1: Chuẩn bị S3 Buckets [cite: 70]
Cần tạo 2 S3 Buckets tại region `us-east-1`:
1.  `nt548-cloudformation`: Dùng để lưu trữ source code/template.
2.  `nt548-taskcat-templates`: Dùng để lưu các artifact đầu ra của CodeBuild và báo cáo Taskcat.

### [cite_start]Bước 2: Cấu hình Build (AWS CodeBuild) [cite: 73]
Sử dụng file `buildspec.yml` để định nghĩa quy trình build gồm 4 pha:

1.  **Install:** Sử dụng Python 3.11, cài đặt `cfn-lint` và `taskcat`.
2.  **Pre_build:** Chạy `cfn-lint` để kiểm tra cú pháp file `main.yaml` và các modules.
3.  **Build:** Chạy `taskcat test run` để kiểm thử triển khai thực tế.
4.  **Post_build:** Thông báo hoàn tất.

**Cấu hình CodeBuild Project:**
- **Source:** CodeCommit hoặc S3.
- [cite_start]**Environment Role:** Cần các quyền `AmazonEC2FullAccess`, `AmazonS3FullAccess`, `AWSCloudFormationFullAccess`, `CloudWatchLogsFullAccess`[cite: 80].
- [cite_start]**Artifacts:** Upload kết quả vào bucket `nt548-taskcat-templates`[cite: 81].

### [cite_start]Bước 3: Xây dựng Pipeline (AWS CodePipeline) [cite: 85]

**1. Source Stage:**
- Provider: AWS CodeCommit.
- Repository: `NT548-LAB2-ex2`.
- [cite_start]Branch: `master`[cite: 89].

**2. Build Stage:**
- Provider: AWS CodeBuild.
- Project: `NT548-Lab2-ex2` (Đã cấu hình ở Bước 2).
- [cite_start]Nhiệm vụ: Kiểm thử template với cfn-lint và taskcat[cite: 90].

**3. Deploy Stage:**
- Provider: AWS CloudFormation.
- Action: Create or Update Stack.
- Template: `main.yaml` (Lấy từ BuildArtifact).
- [cite_start]Role: Cần quyền `AdministratorAccess` hoặc các quyền FullAccess tương ứng[cite: 92].

---

## ⚠️ QUY TẮC LÀM VIỆC TRÊN GITHUB

- Trước khi bắt đầu làm, đọc kĩ quy trình làm việc với Git & GitHub.
- **Commit Message:** Ghi rõ nội dung.
- **Workflow:**
    - Chỉ commit khi hoàn thành 1 feature/bug nào đó.
    - Tạo Pull Request và nhờ thành viên review trước khi merge vào main.
    - Xóa nhánh khi merge thành công.
