# Thông tin học viên

- Họ và tên: Nguyen Duc Anh
- MSHV: 2A202604049
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: DEVducanh
- Repo bài nộp (URL): https://github.com/DEVducanh/K5-L23-Day08-2A202604049-NguyenDucAnh-AI-Assisted-Labeling
- Commit chốt bài (hash): HEAD `00fbe1a` trước các thay đổi CP5–CP6; các thay đổi hiện tại chưa commit.
- Máy: Windows, PyTorch CPU-only; CUDA không khả dụng.
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: **400 / 8 / 20** (mẫu số: 428 box máy); added: **19** (mẫu số: 427 đối tượng sau duyệt).
- So với gold: precision / recall của **máy**: **54.2% / 97.0%** (32 TP, 27 FP, 1 FN; 33 vật gold).
- Bắt bẫy: **10/12 = 83.3%**.
- Thời gian: gán tay **5 phút/ảnh**; review có pre-label **1.25 phút/ảnh**.
- Lỗi QC sau rework: **0% theo ảnh (0/5)**, **0% theo đối tượng (0/88)**; cận trên Wilson 95% theo ảnh vẫn 43% do mẫu nhỏ.
- **Quyết định:** **Release** sau khi sửa các lỗi QC ban đầu; mẫu cuối không có lỗi rõ và tỷ lệ bắt bẫy vượt 80%, nhưng mẫu nhỏ chưa chứng minh được ngưỡng lỗi thật ≤2%.
