# Hệ thống quản lý ảnh chụp DICOM tích hợp trí tuệ nhân tạo cho bệnh viện (AIDIMS)

## Bối cảnh
Trong y tế hiện đại, việc quản lý và xử lý khối lượng lớn hình ảnh y khoa theo định dạng DICOM (Digital Imaging and Communications in Medicine - Hình ảnh kỹ thuật số và truyền thông trong y tế) đã trở thành thách thức đáng kể cho các bệnh viện. Các hình ảnh y khoa như X-quang, CT scan, MRI, siêu âm,... không chỉ chứa thông tin chẩn đoán quan trọng mà còn đòi hỏi lưu trữ và truy xuất nhanh chóng, chính xác để hỗ trợ quá trình điều trị. Tuy nhiên, việc quản lý thủ công và thực hành lưu trữ không hiệu quả có thể dẫn đến mất dữ liệu, chậm trễ trong chẩn đoán, và cuối cùng ảnh hưởng tiêu cực đến chất lượng dịch vụ chăm sóc sức khỏe.

AIDIMS - Hệ thống quản lý ảnh chụp DICOM tích hợp trí tuệ nhân tạo cho bệnh viện - được thiết kế để giải quyết những thách thức này. Hệ thống tự động hóa việc nhập, lưu trữ và tổ chức hình ảnh DICOM dựa trên các tiêu chí như bệnh nhân, phương thức chụp, ngày tháng và bác sĩ phụ trách. Ngoài ra, AIDIMS tích hợp các thuật toán AI tiên tiến để phân tích và phân loại bệnh từ hình ảnh, từ đó cung cấp hỗ trợ chẩn đoán cho bác sĩ thông qua kết quả tư vấn nâng cao. Hệ thống này không chỉ tối ưu hóa việc quản lý dữ liệu hình ảnh mà còn góp phần nâng cao độ chính xác trong chẩn đoán, đảm bảo thông tin kịp thời và chính xác cho đội ngũ y tế.

## Giải pháp đề xuất
Các tính năng chính của AIDIMS bao gồm:

### Quản lý và lưu trữ hình ảnh DICOM:
- Nhập và lưu trữ hình ảnh DICOM từ các thiết bị chụp (CT, MRI, X-quang, v.v.).
- Tổ chức hình ảnh theo bệnh nhân, phương thức chụp, ngày tháng và bác sĩ phụ trách.
- Hỗ trợ hiển thị và xem hình ảnh DICOM với các công cụ điều chỉnh cơ bản (phóng to, di chuyển, điều chỉnh độ tương phản, v.v.).

### Phân tích AI:
- Tích hợp các mô hình AI để phân loại bệnh từ hình ảnh DICOM.

### Quản lý hồ sơ bệnh nhân:
- Liên kết hình ảnh DICOM với hồ sơ bệnh nhân.
- Theo dõi lịch sử chụp và kết quả chẩn đoán cho từng bệnh nhân.
- Quản lý thông tin cá nhân và y tế của bệnh nhân.

### Hỗ trợ chẩn đoán và tham khảo:
- Cho phép bác sĩ chú thích và thêm ghi chú vào hình ảnh.
- Hỗ trợ so sánh hình ảnh mới và cũ của cùng một bệnh nhân.

### Thông báo và lời nhắc:
- Gửi thông báo về kết quả phân tích AI.
- Nhắc nhở bác sĩ về các trường hợp cần xem xét ưu tiên.

## Yêu cầu chức năng

### Nhân viên tiếp nhận:
- Tạo và cập nhật hồ sơ bệnh nhân.
- Ghi lại triệu chứng của bệnh nhân.
- Chuyển hồ sơ bệnh nhân đến bác sĩ phù hợp dựa trên triệu chứng và chuyên môn.

### Bác sĩ:
- Xem hồ sơ bệnh nhân và đưa ra yêu cầu chụp.
- Xem và phân tích hình ảnh DICOM của bệnh nhân.
- Nhận kết quả phân tích AI và đưa ra chẩn đoán cuối cùng.
- Thêm ghi chú và chú thích vào hình ảnh.
- So sánh hình ảnh qua thời gian của cùng một bệnh nhân.
- Tạo báo cáo chẩn đoán.

### Kỹ thuật viên chụp ảnh:
- Nhập hình ảnh DICOM từ thiết bị chụp vào hệ thống.
- Kiểm tra chất lượng hình ảnh và thực hiện chụp lại nếu cần thiết.
- Gán hình ảnh cho bệnh nhân và thêm thông tin liên quan.

### Quản trị viên hệ thống:
- Quản lý tài khoản người dùng trong hệ thống.
- Giám sát hoạt động hệ thống.
- Cấu hình thông số hệ thống và mô hình AI.

### Hệ thống xử lý:
- Tự động phân tích hình ảnh bằng mô hình AI khi có hình ảnh mới được tải lên.
- Gửi thông báo về kết quả phân tích hoặc trong trường hợp khẩn cấp.

## Yêu cầu phi chức năng:
- **Tính khả dụng**: Giao diện thân thiện, dễ sử dụng cho tất cả các loại người dùng.
- **Bảo mật**: Đảm bảo tính bảo mật của dữ liệu bệnh nhân và thông tin y tế theo quy định hiện hành.
- **Hiệu suất**: Hệ thống phải xử lý nhanh chóng các tệp DICOM lớn và đáp ứng yêu cầu phân tích trong thời gian hợp lý.
- **Khả năng mở rộng**: Có khả năng mở rộng để đáp ứng khối lượng dữ liệu hình ảnh và người dùng ngày càng tăng.

## (*) 3.2. Nội dung đề xuất chính (bao gồm kết quả và sản phẩm)

### Lý thuyết và thực hành (tài liệu):
- Sinh viên nên áp dụng các phương pháp phát triển phần mềm và UML 2.0 để mô hình hóa hệ thống.
- Tài liệu nên bao gồm Yêu cầu người dùng, Đặc tả yêu cầu phần mềm, Thiết kế kiến trúc, Thiết kế chi tiết, Triển khai hệ thống, Tài liệu kiểm thử, Hướng dẫn cài đặt, mã nguồn và các gói phần mềm có thể triển khai.

### Công nghệ phía máy chủ:
- Máy chủ: .NET Core...
- Thiết kế cơ sở dữ liệu: SQL Server...

### Công nghệ phía khách hàng:
- Web Client: HTML5, CSS3, JavaScript, ReactJS...
- Module AI: PyTorch hoặc TensorFlow

### Sản phẩm:
- Mô hình AI: Cho backend.
- Ứng dụng Web: Cho tất cả người dùng.
- API: Để kết nối ứng dụng web với hệ thống backend.

### Nhiệm vụ đề xuất:
- Gói nhiệm vụ 1: Phát triển API backend cho hệ thống.
- Gói nhiệm vụ 2: Phát triển mô hình AI cho backend.
- Gói nhiệm vụ 3: Phát triển ứng dụng web cho tất cả vai trò người dùng.
- Gói nhiệm vụ 4: Xây dựng, triển khai và kiểm thử hệ thống trên máy chủ.
- Gói nhiệm vụ 5: Chuẩn bị tất cả tài liệu cần thiết: Phân tích và thiết kế hệ thống, kế hoạch kiểm thử, hướng dẫn cài đặt, hướng dẫn sử dụng.
