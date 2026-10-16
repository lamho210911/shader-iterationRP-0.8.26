IterationRP Shaders — tài liệu kỹ thuật
Bộ tài liệu GitHub, không phải shaderpack cài đặt được. Gói này được soạn dựa trên archive IterationRP-Shaders-All-Versions-v0.8.26.zip do người dùng cung cấp. Mã shader và tài nguyên gốc không được đóng gói lại vì LICENSE.md đi kèm giới hạn việc host/phân phối khi chưa có sự cho phép của tác giả.

Tài liệu này mô tả cấu trúc shader, các đường xử lý hình ảnh và những cơ chế super resolution có thể xác nhận từ cấu hình của bản v0.8.26. Đây không phải xác nhận về hiệu năng trên mọi GPU hay về khả năng tương thích của mọi phiên bản Minecraft/Iris/mod.
Tổng quan
IterationRP là một Minecraft shader pack với các chương trình vertex/fragment/geometry/compute, nhiều pass hậu kỳ và thư viện GLSL dùng chung. Trong archive được kiểm tra, cấu hình có các thành phần sau:
- G-buffer: các chương trình gbuffers_* cho terrain, entity, nước, thời tiết, bàn tay và những bề mặt khác.
- Shadow: các chương trình shadow.* và clrwl_shadow.*.
- Composite / post-processing: các pass composite* dùng để chạy hiệu ứng theo từng giai đoạn.
- Thư viện dùng chung: shaders/Lib/, gồm Programs, PathTracing, FidelityFX, RTWSM, Uniform và những thư mục hỗ trợ khác.
- Cấu hình shader: shaders/shaders.properties, các tệp .json, .properties và bản dịch trong shaders/lang/.
- Super resolution: FSR 2.3 theo nhiều mức render scale; đường FSR 1/EASU–RCAS trong chế độ render resolution tùy chỉnh; và cấu hình tích hợp chung trong shaders/superresolution.json.
- Tương thích theo cấu hình: có tham chiếu đến Iris, Distant Horizons (DH) và Voxy. Khả năng sử dụng thực tế còn tùy phiên bản mod và môi trường chơi.
Tài liệu
Tài liệu	Nội dung
[Kiến trúc shader](docs/SHADER_ARCHITECTURE.md)	Các loại shader, pass, thư mục thư viện và cách đọc cấu hình
[Super resolution](docs/SUPER_RESOLUTION.md)	FSR 2.3, FSR 1/EASU–RCAS, tích hợp superresolution.json, chọn chế độ và xử lý lỗi
[Bản v0.8.26](docs/VERSION_0.8.26.md)	Tóm tắt changelog có trong archive
[Kiểm kê nguồn](docs/SOURCE_AUDIT.md)	Thành phần và dấu kiểm tra của archive đã được xem xét
[Đưa bộ tài liệu lên GitHub](docs/GITHUB_PUSH.md)	Các lệnh Git cơ bản để đẩy repository này lên GitHub
[Thông báo giấy phép upstream](UPSTREAM_LICENSE_NOTICE.md)	Giới hạn được ghi trong giấy phép của shader gốc


Super resolution — tóm tắt nhanh
Chế độ được hiển thị trong ngôn ngữ của shader	Hệ số render theo mỗi chiều (xấp xỉ)	Ý nghĩa thực tế
FSR 2.3 — Quality	0,6667	Render khoảng 2/3 chiều đầu ra; tỉ lệ phóng lên khoảng 1,5×
FSR 2.3 — Balanced	0,5882	Render khoảng 58,82% mỗi chiều; khoảng 1,7×
FSR 2.3 — Performance	0,5000	Render ở một nửa mỗi chiều; khoảng 2×
FSR 2.3 — Ultra Performance	0,3333	Render khoảng 1/3 mỗi chiều; khoảng 3×
Off	—	Không dùng nhánh FSR 2.3 theo FSR2_SCALE


Các mức trên được lấy từ nhãn FSR2_SCALE và những hệ số kích thước trong shaders.properties; chất lượng hình ảnh và FPS thực tế cần đo trong game. Hạ độ phân giải nội bộ thường giảm tải GPU, nhưng upscaling không bảo đảm FPS tăng nếu giới hạn nằm ở CPU, bộ nhớ hoặc một pass khác.
Trạng thái repository
- ✅ Có tài liệu kỹ thuật và ghi chú phiên bản.
- ✅ Có hướng dẫn GitHub.
- ❌ Không chứa mã shader gốc, thư viện GLSL gốc hoặc gói cài đặt shader.
- ⚠️ Muốn phát hành công khai toàn bộ shader, cần có quyền rõ ràng từ chủ sở hữu theo giấy phép đi kèm archive.
Nguồn tham khảo kỹ thuật
- AMD GPUOpen — FidelityFX Super Resolution 1
- AMD GPUOpen — FidelityFX Super Resolution 2
- Tài liệu AMD FidelityFX FSR 2.3
- Iris Shaders — hướng dẫn shader pack
Các đường dẫn này giải thích công nghệ nền tảng; chúng không thay thế giấy phép của IterationRP và không có nghĩa các upstream shader đã được đóng gói trong repository này.
