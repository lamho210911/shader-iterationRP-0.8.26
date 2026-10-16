# Super resolution trong IterationRP v0.8.26

## 1. Super resolution là gì?

Super resolution (upscaling) là cách render khung hình ở độ phân giải nội bộ thấp hơn rồi tái dựng/phóng ảnh lên độ phân giải đầu ra. Nó có thể giảm chi phí render so với render toàn bộ ở native resolution, nhưng chất lượng và FPS phụ thuộc vào thuật toán, tỉ lệ render, GPU/CPU và cách shaderpack truyền dữ liệu đầu vào.

**Phân biệt quan trọng:** FSR 2 là thuật toán temporal (dùng dữ liệu thời gian qua nhiều frame); FSR 1 EASU là thuật toán spatial (dựa trên ảnh của frame hiện tại). Cấu hình này không phải bằng chứng rằng toàn bộ super resolution trong dự án đều là AI upscaling.

## 2. Những đường xử lý có trong archive

### A. FSR 2.3 — tùy chọn `FSR2_SCALE`

`shaders/lang/en_us.lang` và `shaders/lang/zh_cn.lang` ghi tên tùy chọn là **FSR 2.3 SuperResolution**. Các mức được hiển thị trong file ngôn ngữ:

| Giá trị `FSR2_SCALE` | Nhãn menu trong archive | Render scale theo mỗi chiều được đặt trong `shaders.properties` | Hệ số phóng lên gần đúng theo mỗi chiều |
|---:|---|---:|---:|
| `-1` | Off / 关 | Không dùng nhánh FSR2 theo tùy chọn này | — |
| `1` | Quality 1.5x / 质量 1.5x | `0.6667` | `1.5x` |
| `2` | Balanced 1.7x / 平衡 1.7x | `0.5882` | `1.7x` |
| `3` | Performance 2.0x / 性能 2.0x | `0.5` | `2.0x` |
| `4` | Ultra performance 3.0x / 超级性能 3.0x | `0.3333` | `3.0x` |

Các con số là tỉ lệ kích thước theo **từng chiều**, không phải tỉ lệ diện tích. Ví dụ render ở `0.5 × 0.5` chiều rộng/cao tương ứng khoảng 25% số pixel đầu vào so với render native, trước khi tính thêm chi phí upscaling và các hiệu ứng khác.

Mã cũng có nhánh `FSR2_SCALE == 0` trong `shaders.properties`, nhưng hai file ngôn ngữ được xem xét không có nhãn menu tương ứng cho giá trị `0`; vì vậy tài liệu này không gán tên chế độ cho giá trị đó.

#### Các thành phần liên quan

- `FSR2_SCALE` điều khiển kích thước một số buffer, kích thước render nội bộ, jitter index và nhánh chương trình dùng cho FSR2.
- `composite60.csh` include `Reconstruct_Dilate_CS.glsl`.
- `composite61.csh` include `DepthClip_CS.glsl`.
- `composite62.csh` include `Lock_CS.glsl`.
- `composite63.fsh` include `Accumulate_FS.glsl`.
- Các thuật toán FSR2 khác nằm trong `shaders/Lib/FidelityFX/FSR2/`.
- `FSR2_RCAS_SHARPNESS` là tùy chọn điều chỉnh độ sắc nét gắn với nhánh FSR2.

FSR2 cần dữ liệu temporal. Trong archive, cấu hình có tính đến jitter và các buffer liên quan tới chuyển động/độ sâu. Sai lệch motion vectors, vật thể trong suốt hoặc dữ liệu buffer không tương ứng có thể gây ghosting, nhòe hoặc artefact; changelog của chính gói cũng ghi nhận các bản sửa lỗi ghosting với DH/Voxy khi bật super resolution.

### B. FSR 1 — EASU và RCAS

Trong thư mục `shaders/Lib/Programs/Composite/` có:

- `FsrEASU_FS.glsl`: triển khai bước **EASU (Edge Adaptive Spatial Upsampling)**.
- `PreFsrCAS_FS.glsl`: đoạn shader có tên liên quan tới tiền xử lý cho nhánh RCAS trong pipeline này.

Các file pass tương ứng:

- `composite83.fsh` include `FsrEASU_FS.glsl`.
- `composite82.fsh` include `PreFsrCAS_FS.glsl`.

Trong cấu hình render resolution tùy chỉnh, `CUSTOM_SCALE_MODE == 1` bật `composite82` và `composite83`; `CUSTOM_SCALE_MODE.1` được file ngôn ngữ ghi là **FSR 1**. Đây là đường khác với tùy chọn `FSR2_SCALE`.

Về công nghệ nền: EASU phóng ảnh theo không gian; RCAS là bước tăng độ sắc nét tương phản cục bộ và không tự thực hiện scale. Mô tả thuật toán chính thức của AMD ghi nhận RCAS được dùng sau EASU. Tuy vậy, hãy dựa vào cấu hình cụ thể của IterationRP để xác định thứ tự pass thực tế.

`FSR_SHARPNESS`, `FSR_DENOISE`, `CAS_SHARPNESS` và `CAS_DENOISE` cũng xuất hiện trong menu ngôn ngữ; chúng không nên bị nhầm thành cùng một tùy chọn với `FSR2_RCAS_SHARPNESS`.

### C. Tích hợp super-resolution chung — `superresolution.json`

File `shaders/superresolution.json` trong archive có cấu trúc sau (mô tả tóm tắt, không sao chép toàn bộ JSON):

| Thuộc tính | Giá trị khai báo |
|---|---|
| `schema_version` | `1` |
| Profile | `*` (profile tổng quát) |
| Jitter | Bật |
| Upscale | Bật |
| `internal_format` | `rgba16f` |
| Trigger | `BEFORE` pass `composite67` |
| `motion_jittered` | `false` |
| `hdr` | `true` |
| Pre-exposure | Hằng số `1.0` |
| Auto exposure | Bật |
| Color input | `autotex6`, vùng `[0, 0, -1, -1]` |
| Depth input | `colortex16`, vùng `[0, 0, -1, -1]` |
| Motion vectors input | `colortex10`, vùng `[0, 0, -1, -1]` |
| Exposure input | `colortex17`, vùng `[0, 0, 1, 1]` |
| Upscaled color output | `colortex7`, vùng `[0, 0, -2, -2]` |

Đây là cấu hình khai báo dữ liệu vào/ra cho một hệ thống super-resolution tương thích; nó **không tự chứng minh backend DLSS/FSR cụ thể nào đã được cài hoặc đang hoạt động**. Backend thực tế tùy vào mod tích hợp, phiên bản mod, GPU và cấu hình.

Trong `shaders.properties`, nhánh `SR_ENABLE == 1` dùng `SR_RENDER_SCALE_FACTOR` để tính độ phân giải nội bộ và chọn đường tích hợp chung. Khi nhánh này bật, một số pass FSR2 hoặc FSR1 được bật/tắt theo điều kiện để tránh chạy hai đường upscale cùng lúc.

## 3. Cách chọn chế độ

- **Ưu tiên hình ảnh:** bắt đầu từ Quality 1.5x. Nếu FPS chưa đủ, thử Balanced.
- **Ưu tiên FPS:** thử Performance 2.0x; Ultra Performance 3.0x giảm độ phân giải nội bộ mạnh hơn nhưng thường dễ mất chi tiết/ổn định hình ảnh hơn.
- **Cần tự cân bằng:** so sánh ở cùng cảnh, cùng độ phân giải đầu ra và cùng thiết lập khác. Hãy quan sát mép lá/cỏ, đường chéo, văn bản, vật thể chuyển động và vùng nước/độ trong suốt.
- **Không tăng sharpen quá mức:** sharpening có thể làm viền sáng, nhiễu hoặc răng cưa nổi bật hơn. Thay đổi `FSR2_RCAS_SHARPNESS` từng bước nhỏ và kiểm tra lại khi camera di chuyển.

Không có mức nào bảo đảm một FPS cụ thể. Nếu FPS không tăng khi giảm render scale, có thể game đang bị giới hạn bởi CPU, một pass shader khác, băng thông bộ nhớ hoặc giới hạn đồng bộ khung hình.

## 4. Tránh xung đột cấu hình

1. Không bật đồng thời nhiều đường upscale không tương thích. Tùy chọn `INFO_RES`/`INFO_RES_0` trong ngôn ngữ có cảnh báo không bật độ phân giải tùy chỉnh cùng lúc với super resolution.
2. Chỉ bật **một** đường theo mục tiêu: `FSR2_SCALE`, FSR 1 trong custom render resolution, hoặc mod super-resolution ngoài thông qua `SR_ENABLE`.
3. Nếu dùng custom resolution, kiểm tra cả chiều rộng, chiều cao, aspect ratio và scaling mode; tỷ lệ khác nhau có thể làm ảnh bị méo hoặc letterbox.
4. Sau khi thay đổi chế độ, áp dụng lại shader settings và kiểm tra log khi có artefact.
5. Nếu xuất hiện ghosting, thử tắt tạm các mod/tuỳ chọn ảnh hưởng đến render như DH/Voxy để xác định nguồn; sau đó kiểm tra lại tổ hợp phiên bản. Đây là cách cô lập lỗi, không phải bảo đảm lỗi nằm ở mod cụ thể.

## 5. Checklist kiểm thử

- [ ] Ghi phiên bản Minecraft, loader, Iris, shaderpack và mod liên quan.
- [ ] Ghi GPU, driver, độ phân giải đầu ra và thiết lập đồ họa.
- [ ] Ghi `FSR2_SCALE`, `FSR2_RCAS_SHARPNESS` và những tùy chọn custom resolution liên quan.
- [ ] Đo FPS và frame time ở cùng một cảnh, camera và thời lượng.
- [ ] Kiểm tra ảnh tĩnh lẫn camera chuyển động.
- [ ] Kiểm tra các vùng trong suốt, hạt/particles, nước, bóng và DH/Voxy nếu có dùng.
- [ ] Thay đổi một thiết lập mỗi lần để dễ xác định nguyên nhân.

## Tài liệu tham khảo

- [AMD GPUOpen — FidelityFX FSR 1 source and overview](https://github.com/GPUOpen-Effects/FidelityFX-FSR)
- [AMD GPUOpen — FidelityFX FSR 2 source and overview](https://github.com/GPUOpen-Effects/FidelityFX-FSR2)
- [AMD GPUOpen — FSR 2.3 technical manual](https://gpuopen.com/manuals/fidelityfx_sdk/techniques/super-resolution-temporal/)
