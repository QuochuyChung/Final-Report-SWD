# LegacyVault – Ghi chú nghiệp vụ (SWD391)

> Hệ thống Quản lý Di sản số & Bàn giao Tài sản Kỹ thuật số
> Tài liệu này ghi lại những gì cả nhóm đã thống nhất để các sơ đồ trong report khớp nhau về tính năng và luồng. Nguồn gốc là spec nghiệp vụ gốc (5 actor, 23 yêu cầu).

---

## 1. Tổng quan nghiệp vụ

LegacyVault cho phép một người (Vault Owner) lưu trữ có mã hóa các tài sản số (tài khoản ngân hàng, ví crypto, mạng xã hội, tài liệu quan trọng) cùng tài liệu pháp lý (di chúc, giấy tờ sở hữu). Hệ thống dùng cơ chế **Dead Man's Switch (DMS)**: owner phải định kỳ xác nhận "còn sống". Nếu owner không phản hồi trong thời gian chờ, hệ thống kích hoạt bàn giao. Vault chỉ được mở khóa sau khi **Legal Verifier** xác minh giấy chứng tử và ký điện tử. Sau đó từng **Beneficiary** xác thực danh tính, nhận tài sản đã giải mã và xác nhận đã nhận.

### 1.1 Actor

| # | Actor | Vai trò |
|---|---|---|
| 1 | Vault Owner | Chủ kho, tạo và quản lý tài sản, cấu hình DMS |
| 2 | Digital Executor / Trustee | Người thi hành, kích hoạt xác minh và theo dõi bàn giao |
| 3 | Beneficiary | Người thụ hưởng, nhận tài sản |
| 4 | Legal Verifier / Notary | Xác minh pháp lý, ký mở khóa |
| 5 | System Administrator | Quản trị người dùng, chính sách, tích hợp |
| – | System Scheduler (actor thời gian) | Gửi check-in, kích hoạt bàn giao khi hết thời gian chờ |
| – | eKYC / OTP Service (ngoài) | Xác thực danh tính |
| – | Digital Signature / e-Notary Service (ngoài) | Kiểm tra và cấp chữ ký số |
| – | Email / SMS / Push Provider (ngoài) | Gửi thông báo |

---

## 2. Các quyết định đã chốt

| Vấn đề | Quyết định | Lý do |
|---|---|---|
| Số mức timeout của DMS | **1 mức**: Active → Waiting → Triggered. Bỏ việc tách Warning và Waiting | Spec chỉ có chu kỳ check-in + "thời gian chờ" |
| Owner cancel bằng OTP | **Bỏ** | Không có trong spec. Owner phản hồi trong lúc Waiting thì về Active là đủ |
| Conceptual diagram | **Giữ nguyên** | 23 mũi tên khớp 1:1 với 23 yêu cầu, dùng làm bản đồ truy vết |
| Chữ ký số / e-notary | **Hệ thống ngoài riêng** | Spec: Admin quản lý tích hợp bên thứ ba (chữ ký số, cổng công chứng điện tử) |
| Executor xem gì trước khi mở khóa | Chỉ **metadata** (tên, loại, người nhận) và hướng dẫn | Chỉ beneficiary nhận bản giải mã |
| Đóng hồ sơ | **Theo từng tài sản**. Case hoàn tất khi mọi tài sản đã đóng | Spec: "đóng hồ sơ tài sản tương ứng" |
| Gán beneficiary | 1 tài sản = 1 beneficiary (gán trực tiếp hoặc qua nhóm tài sản) | Spec: "từng tài sản hoặc nhóm tài sản" |
| Executor | 1 executor active mỗi vault | Spec dùng số ít |
| Kênh trao đổi Executor ↔ Verifier | Là một thành phần của hệ thống. Yêu cầu bổ sung hồ sơ và nộp lại đi qua kênh này | Spec: "qua kênh tích hợp trong hệ thống" |
| Login/Register, Create Vault, Resubmit | Không thêm use case riêng | Đã bao hàm trong các chức năng khác |
| Appoint Executor | Gộp thành "Appoint / Update / Revoke Executor" | Spec chỉ ghi cập nhật/thu hồi, chỉ định là ngầm định |

---

## 3. Các chức năng

### 3.1 Chức năng người dùng (23 theo spec + 1 ngầm định)

| FE | Chức năng | Actor |
|---|---|---|
| **FE-01** Tài sản & tài liệu | Tạo và mã hóa hồ sơ tài sản số | Owner |
| | Tải lên và mã hóa E2E tài liệu pháp lý | Owner |
| **FE-02** Chỉ định người | Gán beneficiary cho tài sản / nhóm tài sản | Owner |
| | Chỉ định / cập nhật / thu hồi Executor | Owner |
| **FE-03** Dead Man's Switch | Cấu hình DMS (chu kỳ check-in + thời gian chờ) | Owner |
| | Phản hồi check-in "còn sống" *(ngầm định)* | Owner |
| **FE-04** Nhật ký & giám sát | Xem nhật ký hoạt động và lịch sử truy cập | Owner |
| | Giám sát audit log và cảnh báo truy cập bất thường | Admin |
| **FE-05** Kích hoạt & theo dõi | Nhận thông báo kích hoạt bàn giao | Executor |
| | Xem danh sách tài sản được ủy quyền và hướng dẫn bàn giao | Executor |
| | Theo dõi tiến độ bàn giao từng tài sản | Executor |
| **FE-06** Xác minh & mở khóa | Gửi yêu cầu xác minh (giấy chứng tử) | Executor |
| | Trao đổi tài liệu xác minh qua kênh tích hợp | Executor, Verifier |
| | Duyệt / từ chối yêu cầu xác minh | Verifier |
| | Xác thực chữ ký số và tính hợp lệ của tài liệu | Verifier |
| | Ký điện tử cấp phép mở khóa vault | Verifier |
| | Xem lịch sử hồ sơ đã xác minh | Verifier |
| **FE-07** Nhận tài sản | Nhận thông báo có tài sản được bàn giao | Beneficiary |
| | Xác thực danh tính OTP/eKYC (include của hai chức năng kế bên) | Beneficiary |
| | Xem danh sách và chi tiết tài sản thừa kế | Beneficiary |
| | Tải tài liệu / khóa truy cập đã giải mã | Beneficiary |
| | Xác nhận đã nhận, đóng hồ sơ tài sản tương ứng | Beneficiary |
| **FE-08** Quản trị | Quản lý người dùng, vai trò, phân quyền (RBAC) | Admin |
| | Cấu hình chính sách mã hóa, sao lưu, quy tắc DMS mặc định | Admin |
| | Quản lý tích hợp bên thứ ba (eKYC, chữ ký số, e-notary) | Admin |

### 3.2 Chức năng tự động (không có giao diện, dùng cho mục II.4 của template)

- Lên lịch và gửi check-in định kỳ; kích hoạt bàn giao khi hết thời gian chờ (Scheduler).
- Mã hóa E2E khi lưu, giải mã khi mở khóa.
- Mở khóa vault khi request được duyệt; tạo Handover Case và Assignment cho từng tài sản.
- Gửi thông báo Email / SMS / Push.
- Ghi audit log cho mọi thay đổi trạng thái.
- Sao lưu dữ liệu định kỳ.

---

## 4. Quy tắc nghiệp vụ

| ID | Quy tắc |
|---|---|
| BR-01 | Mỗi vault có đúng 1 DMS. Admin đặt quy tắc mặc định, owner cấu hình lại cho vault của mình |
| BR-02 | Owner phản hồi check-in, kể cả trong lúc Waiting, thì DMS về Active và lên lịch chu kỳ mới |
| BR-03 | Hết thời gian chờ mà owner không phản hồi thì DMS chuyển Triggered |
| BR-04 | Mỗi vault có tối đa 1 executor active. Owner có thể cập nhật/thu hồi khi còn hoạt động |
| BR-05 | Mỗi tài sản có đúng 1 beneficiary (gán trực tiếp hoặc qua nhóm) |
| BR-06 | Trước khi mở khóa, executor chỉ thấy metadata tài sản và hướng dẫn bàn giao, không thấy nội dung |
| BR-07 | Chỉ yêu cầu xác minh ở trạng thái Approved (có chữ ký điện tử của verifier) mới mở khóa được vault |
| BR-08 | Từ chối yêu cầu bắt buộc có lý do. Executor bổ sung hồ sơ qua kênh trao đổi và gửi lại |
| BR-09 | Beneficiary chỉ thấy và tải tài sản của mình, sau khi xác thực OTP/eKYC |
| BR-10 | Xác nhận đã nhận thì đóng hồ sơ của tài sản đó. Case Completed khi mọi tài sản đã đóng |
| BR-11 | Mọi thay đổi trạng thái đều ghi audit log |

**Giả định ngoài spec** (ghi vào mục Assumptions của use case description, không vẽ vào sơ đồ):
- Giới hạn số lần thử lại OTP/eKYC (ví dụ 3 lần).
- Cách xử lý khi beneficiary không bao giờ xác nhận (ví dụ executor/admin đóng case sau N ngày kèm ghi chú).

---

## 5. Vòng đời trạng thái (State diagram)

```
[Start] → Active → Waiting → Triggered → Pending Verification → Unlocked → Completed → [End]
              ↑        │                       │  ↑
              └────────┘ (owner phản hồi)      ↓  │
                                             Rejected ──(gửi lại)──┘
```

State diagram mô tả vòng đời của vault/case. Ánh xạ sang entity trong Class diagram:

| State | Thuộc entity | Ghi chú |
|---|---|---|
| Active, Waiting, Triggered | `DeadManSwitch.status` | Active → Waiting khi đến hạn mà chưa check-in |
| Pending Verification, Rejected | `LegalVerificationRequest.status` | Rejected → Pending khi executor gửi lại |
| Unlocked (đang bàn giao) | `Vault.status` | Sau khi request Approved |
| Completed | `HandoverProgress.status` | Khi mọi tài sản đã đóng |

---

## 6. Luồng thao tác của người dùng (theo màn hình)

### 6.1 Owner thiết lập kho (làm một lần)
1. **Đăng nhập:** email + mật khẩu, rồi nhập OTP.
2. **Dashboard:** số tài sản, số tài liệu, người thụ hưởng, executor, trạng thái DMS **Active**, ngày check-in tiếp theo.
3. **Tab Tài sản → Thêm tài sản:** chọn loại, điền thông tin truy cập, Lưu. Hệ thống mã hóa trước khi lưu.
4. **Tab Tài liệu pháp lý:** kéo thả file di chúc / giấy tờ sở hữu, hệ thống mã hóa E2E.
5. **Tab Người thụ hưởng:** chọn tài sản hoặc nhóm tài sản, gán một người nhận.
6. **Tab Người thi hành:** chỉ định executor kèm hướng dẫn bàn giao. Có nút Cập nhật / Thu hồi.
7. **Tab Dead Man's Switch:** chọn chu kỳ check-in (ví dụ 30 ngày) và thời gian chờ (ví dụ 14 ngày), lưu.
8. **Tab Nhật ký:** ai đã truy cập gì và khi nào.

### 6.2 Check-in định kỳ
1. Đến hạn, owner nhận email/SMS/push: "Hãy xác nhận bạn vẫn ổn".
2. Bấm link, đăng nhập, màn hình có nút lớn **"Tôi vẫn ổn"**.
3. Bấm xong: "Đã ghi nhận, lần kiểm tra tiếp theo ngày …". Trạng thái về **Active**.
4. Nếu không bấm: trạng thái **Waiting** (đếm ngược thời gian chờ, nhắc thêm). Owner bấm lúc này vẫn về Active. Hết thời gian chờ thì **Triggered**.

### 6.3 Executor yêu cầu mở khóa
1. Nhận thông báo "Quy trình bàn giao của [tên owner] đã được kích hoạt", đăng nhập.
2. **Màn hình case:** trạng thái **Triggered**. Hiện danh sách tài sản (tên, loại, người nhận; nội dung vẫn khóa) và hướng dẫn của owner.
3. **Gửi yêu cầu xác minh:** upload giấy chứng tử và hồ sơ, bấm Gửi. Trạng thái **Pending Verification**.
4. **Khung trao đổi** với verifier: bổ sung hồ sơ khi được yêu cầu.
5. Nếu bị từ chối: thấy lý do, trạng thái **Rejected**. Gửi lại thì về Pending.

### 6.4 Verifier xử lý
1. Đăng nhập, vào **Hộp thư yêu cầu**.
2. **Chi tiết yêu cầu:** xem giấy chứng tử, hồ sơ. Hệ thống kiểm tra chữ ký số và hiện kết quả hợp lệ hay không.
3. Hai nút:
   - **Duyệt + ký điện tử:** ký xong thì hệ thống mở khóa vault.
   - **Từ chối:** bắt buộc nhập lý do. Có thể mở khung trao đổi để yêu cầu bổ sung.
4. **Tab Lịch sử:** các hồ sơ đã xác minh và trạng thái.

### 6.5 Beneficiary nhận tài sản
1. Nhận thông báo "Bạn có tài sản được bàn giao" kèm link.
2. **Xác thực danh tính:** OTP hoặc eKYC. Sai thì báo lỗi, cho thử lại.
3. **Danh sách tài sản thừa kế:** chỉ hiện tài sản của người này.
4. **Chi tiết tài sản:** thông tin đã giải mã, nút Tải xuống tài liệu / khóa truy cập.
5. Nút **"Tôi đã nhận"**: hồ sơ tài sản chuyển **Đã đóng**.
6. Bên executor, bảng tiến độ cập nhật (ví dụ 2/5). Đủ 5/5 thì case **Completed**.

### 6.6 Admin
- **Người dùng và vai trò:** thêm, khóa, đổi vai trò, phân quyền.
- **Giám sát:** nhật ký bảo mật và cảnh báo truy cập bất thường.
- **Cấu hình:** mã hóa, sao lưu, mặc định DMS.
- **Tích hợp:** bật/tắt và cấu hình eKYC, chữ ký số, e-notary.

> Mỗi luồng trên là Normal Flow của use case description (mục II.5.2.2 của template). Lỗi như OTP sai, bị từ chối thì thành Alternative Flows / Exceptions.

---

## 7. Thống nhất tên gọi

| Dùng | Thay cho |
|---|---|
| `LegalVerificationRequest` | "Verification Record" (Sequence), "Verification Request" (UC) |
| Handover Case | "handover profile", "handover tasks", `HandoverProgress` |
| `BeneficiaryAssignment` | giữ nguyên, một bản ghi cho mỗi tài sản |
| Email / SMS / Push Provider | "Notification Service" ở Context (Notification Service là service nội bộ trong Component) |
| Digital Signature / e-Notary Service | dùng thống nhất ở Context, UC, Sequence, Component |

---

## 8. Tình trạng sơ đồ và việc cần làm

### 8.1 Hiện có
Context, Activity, State, Conceptual, Component, Class (đã merge vào main). Use Case và Sequence nằm trên branch `feat/sequence_and_usecase_diagram` của bạn Quang, chưa merge. Jira chưa cập nhật.

### 8.2 Chưa có
ERD (SWD391-27), Package (31), Database Diagram (32), Integrated Communication (29). Template còn cần Communication Diagram (III.1.2) và System High-Level Design (IV.2).

### 8.3 Cần sửa theo các quyết định ở trên

| Sơ đồ | Việc cần làm |
|---|---|
| Context | Thêm Digital Signature / e-Notary Service; đổi Notification Service thành Email / SMS / Push Provider |
| Use Case | Gộp thành "Appoint / Update / Revoke Executor"; kiểm tra khớp 24 chức năng ở mục 3.1 |
| Activity | Xóa mũi tên thừa từ "Submit Death Certificate" sang "Reject"; chuyển Reject sang lane Verifier/System; thêm action owner phản hồi check-in; thêm loop theo từng tài sản trước khi close case; thêm bước executor theo dõi tiến độ |
| State | Gộp Warning + Waiting thành Waiting; bỏ "Owner cancels (OTP)"; đổi "Unlocked" thứ hai thành "Completed"; ghi rõ ánh xạ entity (mục 5) |
| Class | Thêm quan hệ `LegalVerificationRequest` → `Vault`; thêm `rejectionReason`, `digitalSignature`; thêm `AssetGroup`; thêm Role/Permission và Policy; liệt kê giá trị các enum cho khớp State |
| Component | Thêm Verifier Portal; route gateway tới Legal Verification Service và Executor & Beneficiary Service; thêm thành phần kênh trao đổi tài liệu; thêm data store cho DMS/Handover/Request; Orchestrator gửi event sang Audit; thêm Digital Signature / e-Notary vào External Services |
| Sequence | Thêm lifeline Audit Log; thêm flow Check-in → Trigger và flow Beneficiary nhận tài sản; nhánh reject nối với kênh trao đổi và gửi lại |
| Conceptual | Giữ nguyên |
| Report (docx) | Còn nguyên nội dung mẫu: Cafeteria Ordering System, SWD392, Da Nang 2025, thông tin team. Cần thay bằng nội dung LegacyVault |

### 8.4 Gợi ý chia Major Features (mục II.2)
FE-01 Tài sản & tài liệu · FE-02 Chỉ định người · FE-03 Dead Man's Switch · FE-04 Nhật ký & giám sát · FE-05 Kích hoạt & theo dõi · FE-06 Xác minh & mở khóa · FE-07 Nhận tài sản · FE-08 Quản trị.

---

## 9. Câu hỏi còn mở

- Có chấp nhận các giả định ngoài spec ở mục 4 (retry OTP, timeout khi beneficiary không xác nhận) không?
- Ai merge branch Use Case/Sequence và cập nhật Jira (Component đã xong nhưng vẫn In Progress)?
- Template ghi SWD392, Jira ghi SWD391: dùng mã nào trong report?
