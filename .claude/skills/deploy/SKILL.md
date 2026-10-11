---
name: deploy
description: Skill to automate the full deployment process including build, test, and release steps.
---

# Deploy Skill (Kỹ năng triển khai phần mềm)

## Purpose (Mục đích)

Automate the deployment pipeline: build → test → deploy → verify.

Tự động hóa toàn bộ quy trình triển khai phần mềm:

Build (Xây dựng) → Test (Kiểm thử) → Deploy (Triển khai) → Verify (Xác minh).

## Prerequisites (Điều kiện tiên quyết)

- All tests passing: Tất cả các bài kiểm thử phải vượt qua.
- Environment variables configured: Các biến môi trường đã được cấu hình.
- Access to deployment target: Có quyền truy cập vào môi trường hoặc máy chủ đích cần triển khai.

## Steps (Các bước thực hiện)

### 1. Verify Clean State (Kiểm tra trạng thái mã nguồn)

bash git status git pull origin main 

Giải nghĩa:
- git status: Kiểm tra trạng thái hiện tại của kho mã nguồn, bao gồm các tệp đã sửa đổi nhưng chưa commit.
- git pull origin main: Lấy và tích hợp các thay đổi mới nhất từ nhánh main trên kho lưu trữ từ xa origin vào nhánh hiện tại.

Lưu ý: Cần kiểm tra các thay đổi chưa được lưu trước khi pull để tránh xung đột hoặc mất dữ liệu.

### 2. Run Tests (Chạy kiểm thử)

bash npm test 

Giải nghĩa:
- npm test: Chạy các bài kiểm thử được định nghĩa trong dự án thông qua npm.
- If tests fail → STOP. Report failures: Nếu kiểm thử thất bại, dừng quy trình và báo cáo lỗi. Không tiếp tục triển khai.

### 3. Build (Xây dựng ứng dụng)

bash npm run build 

Giải nghĩa:
- npm run build: Chạy lệnh build được định nghĩa trong package.json để tạo phiên bản ứng dụng sẵn sàng triển khai.
- Nếu quá trình build thất bại, dừng quy trình và báo cáo nguyên nhân.

### 4. Deploy by Environment (Triển khai theo môi trường)

#### Development (Môi trường phát triển)

bash docker-compose -f docker-compose.dev.yml up -d --build 

Giải nghĩa:
- docker-compose: Công cụ quản lý các dịch vụ Docker theo cấu hình.
- -f docker-compose.dev.yml: Sử dụng tệp cấu hình dành cho môi trường phát triển.
- up: Khởi tạo và chạy các dịch vụ.
- -d: Chạy ở chế độ nền.
- --build: Xây dựng lại image trước khi khởi chạy các dịch vụ.

#### Production (Môi trường vận hành chính thức)

bash docker-compose -f docker-compose.prod.yml pull docker-compose -f docker-compose.prod.yml up -d 

Giải nghĩa:

Lệnh thứ nhất:
- pull: Tải các Docker image mới nhất được chỉ định trong tệp cấu hình production.

Lệnh thứ hai:
- up -d: Khởi chạy các dịch vụ ở chế độ nền dựa trên cấu hình production.

Lưu ý: Cần xác minh phiên bản image và cấu hình trước khi triển khai lên production để tránh cập nhật nhầm phiên bản.

### 5. Health Check (Kiểm tra tình trạng hoạt động)

bash curl -f http://localhost:3000/health || echo "Health check failed" 

Giải nghĩa:
- curl: Gửi yêu cầu HTTP đến địa chỉ được chỉ định.
- -f: Báo lỗi nếu máy chủ trả về mã trạng thái HTTP thuộc nhóm lỗi.
- http://localhost:3000/health: Endpoint dùng để kiểm tra tình trạng hoạt động của ứng dụng trên cổng 3000.
- ||: Chạy lệnh tiếp theo nếu lệnh trước đó thất bại.
- echo "Health check failed": In thông báo kiểm tra sức khỏe thất bại.

Tiêu chí: Endpoint phải trả về HTTP 200 hoặc mã thành công được quy định trong hệ thống.

### 6. Tail Logs (Theo dõi nhật ký hoạt động)

bash docker-compose logs --tail=50 -f app 

Giải nghĩa:
- logs: Hiển thị nhật ký hoạt động của dịch vụ.
- --tail=50: Hiển thị 50 dòng nhật ký gần nhất.
- -f: Tiếp tục theo dõi nhật ký mới phát sinh.
- app: Tên dịch vụ cần theo dõi.

Yêu cầu: Theo dõi nhật ký trong 30 giây đầu sau khi triển khai để phát hiện lỗi khởi động hoặc lỗi vận hành.

### Rollback (Khôi phục phiên bản trước)

bash git revert HEAD --no-edit git push origin main # Redeploy previous version 

Giải nghĩa:
- git revert HEAD --no-edit: Tạo một commit mới để đảo ngược các thay đổi của commit mới nhất mà không mở trình soạn thảo nội dung commit.
- git push origin main: Đẩy commit mới lên nhánh main của kho lưu trữ từ xa.
- Redeploy previous version: Triển khai lại phiên bản trước đó.

Cảnh báo: Không tự động chạy rollback nếu chưa xác định nguyên nhân lỗi và phạm vi ảnh hưởng. Cần kiểm tra commit hiện tại, trạng thái dữ liệu và phương án khôi phục trước khi thực hiện.

### Success Criteria (Tiêu chí thành công)

- Health endpoint returns 200: Endpoint kiểm tra tình trạng hoạt động trả về mã HTTP 200.
- No error logs in initial 30 seconds: Không xuất hiện nhật ký lỗi trong 30 giây đầu sau khi triển khai.
- All smoke tests pass: Tất cả các bài kiểm thử nhanh sau triển khai đều thành công.

Smoke test (Kiểm thử nhanh): Các bài kiểm thử cơ bản nhằm xác nhận những chức năng quan trọng của ứng dụng vẫn hoạt động sau khi triển khai.