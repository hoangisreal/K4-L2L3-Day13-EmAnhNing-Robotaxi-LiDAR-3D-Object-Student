# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

Số liệu dưới đây lấy từ `ket-qua-nhom-01` đã có trên máy. Không suy ra số hộp hoặc `mean_z` ngoài `summary.csv`. Tên người chạy, MSSV và nhận xét cá nhân không có trong output.

## Nhóm và provenance

- Mã nhóm/phòng: EmAnhNing. Phòng: chưa có trong file kết quả.
- Thành viên: xem `TEAMMATES.md` (họ tên/MSSV, vai trò từng lượt). Bảng thành viên vẫn trống.
- Trạng thái: `executed-by-group`. `smoke.json` có `"status": "passed"`.
- Người thực sự chạy: không ghi trong output. Thời gian trong `smoke.json`: `2026-10-01T07:51:48.292875+00:00` đến `2026-10-01T07:52:25.969320+00:00`. Runtime ghi trong file: `os` linux, `architecture` amd64.
- Image tag và image ID: `day13-pointpillars:lc-20261001-amd64`; `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`. Phiên bản repo ghi trong `smoke.json`: `0831856d921609312d42c7582c366e5a311bb7b1`, `working_tree_dirty`: true.
- PCD được cấp / frame_id: `demo`, `dataset` KITTI. `input_sha256`: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`. Fingerprint riêng của LC: không có trong output.
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`; sha256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Phạm vi: runner `bundle/student-bundle.py` không truyền `--full-scene` và truyền `--score-thresh 0.3`. Ba file JSON không có trường score threshold.
- Giả định kênh thứ tư/intensity: bản PCD demo bỏ reflectance nguồn và để RGB = 0, theo `manifest` của gói. `z_ground` trong cả ba JSON và trong `qc-cases/manifest.json`: `0.07500000000000001` m.

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy **Số hộp** từ `n_boxes`, **mean_z** từ `mean_z` trong `run-A/B/C/summary.csv`; không tự tính lại hoặc đoán. `mean_z` không phải điểm chất lượng. Mở `side-*.png`, đối chiếu `boxes-*.json` để ghi quan sát. Số hộp không phải đáp án cần khớp nhóm khác.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `run-A/boxes-demo-delta-0-voxel-0.16.json`, `run-A/side-demo-delta-0-voxel-0.16.png`, `run-A/summary.csv` | JSON: 1 hộp `vehicles`, tâm x=13.154117584228516, y=-0.4510990381240845, z=0.33010525703430177, score=0.32236409187316895. Ảnh Side tiêu đề `delta=0 voxel=0.16 boxes=1`, một hộp đỏ quanh cụm điểm gần x≈13 m. |
| B | 1.73 | 0.16 | 13 | 1.034 | `run-B/boxes-demo-delta-1.73-voxel-0.16.json`, `run-B/side-demo-delta-1.73-voxel-0.16.png`, `run-B/summary.csv` | JSON: 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian`. Ảnh Side tiêu đề `delta=1.73 voxel=0.16 boxes=13`. Script vẽ `vehicles` màu đỏ, lớp còn lại màu cam. |
| C | 1.73 | 0.32 | 6 | 1.091 | `run-C/boxes-demo-delta-1.73-voxel-0.32.json`, `run-C/side-demo-delta-1.73-voxel-0.32.png`, `run-C/summary.csv` | JSON: 6 hộp, cả 6 đều `pedestrian`. Ảnh Side tiêu đề `delta=1.73 voxel=0.32 boxes=6`, chỉ thấy hộp cam. |

- A/B — chỉ đổi delta: A có 1 hộp; B có 13 hộp. `run-A/summary.csv` có `mean_z` 0.330; `run-B/summary.csv` có `mean_z` 1.034. Ảnh `side-demo-delta-0-voxel-0.16.png` có 1 hộp; `side-demo-delta-1.73-voxel-0.16.png` có 13 hộp trên cùng dải điểm. Đây là chạy lại model trên input khác, không chỉ dịch hộp cũ: số hộp và danh sách class khác nhau. Điều còn chưa chắc là hộp nào bám đúng vật thể, vì output không có nhãn đối chiếu.
- B/C — chỉ đổi pillar: B có 13 hộp; C có 6 hộp. `mean_z` trong CSV là 1.034 và 1.091. B có `vehicles`, `two-wheels` và `pedestrian`; C trong JSON chỉ còn `pedestrian`. Có đủ bằng chứng để kết luận cấu hình nào tốt hơn không? Không. Output không có ground truth; số hộp và `mean_z` không phải điểm chất lượng.
- Giới hạn ROI và góc Side: runner không bật `--full-scene`, nên vật ngoài cửa sổ phía trước không nằm trong ba file JSON này. Ảnh Side là chiếu x-z; code vẽ hộp bằng `length` theo x và `height` theo z, không hiện `y` hay yaw. Hai hộp khác `y` có thể chồng trên ảnh. Đường `z = 0` là `axhline` của đồ thị.
- JSON nào còn chưa đủ cơ sở để import? Cả `run-A`, `run-B`, `run-C` và mọi file trong `qc-cases`. Đây là PCD KITTI `demo`, không phải frame Robotaxi. Cần đúng frame, schema và phép chuyển của job CVAT trước khi import; việc đó không làm từ các file này.

## Ca QC có kiểm soát — không import CVAT

`qc-cases/manifest.json` ghi `training_only: true`, nguồn là `boxes-demo-delta-1.73-voxel-0.16.json`, `box_count` 13, `delta_m` 1.73, `z_ground_m` 0.07500000000000001, `height_offset_m` 1.805. So từng hộp với JSON lượt B: class, x, y, yaw, length, width, height và score không đổi. Chỉ `z` đổi ở các ca dưới.

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0 / 13 | 0 | Không | Bản đối chiếu của prediction B, không phải nhãn đúng | `case-correct.json` trùng `z` với lượt B |
| case-batch-z | 13 / 13 | −1.805 m ở mọi hộp | Không | Dừng batch, báo LC kiểm pipeline | `case-batch-z.json`; ảnh `side-batch-z.png` cho cả 13 hộp nằm dưới dải điểm |
| case-one-box-z | 1 / 13 | −1.805 m ở hộp index 0; 12 hộp còn lại lệch 0 | Không | Kiểm từng hộp, không kết luận cả pipeline sai | `case-one-box-z.json`; ảnh `side-one-box-z.png` có một hộp tụt xuống dưới đường z=0 |

Helper tạo biến đổi có chủ đích từ prediction lượt B. Ba ca này không phải ba lần inference và không phải nhãn đúng.

## Nhận xét cá nhân

Mỗi thành viên tự viết một mục vào đây. Output không chứa vai trò hay lời nhận xét của từng người, nên mục này không được điền thay.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
