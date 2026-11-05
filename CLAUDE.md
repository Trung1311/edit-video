# edit-video

Người dùng gửi một video (đính kèm trong chat hoặc tên/link file trên Google Drive)
và chỉ muốn nhận lại **một video đã edit**. Không hỏi lại trừ khi thật sự không làm được.
Trả lời bằng tiếng Việt, ngắn gọn.

## Quy trình mặc định (khi người dùng không dặn gì thêm)

1. Lấy video: file đính kèm nằm trong `~/.claude/uploads/`; file Drive thì tải về bằng Google Drive connector.
   Làm việc trong thư mục scratchpad, không commit video vào repo.
2. Đo video: `hyperframes measure <file>` / `ffprobe` (thời lượng, khổ hình, có giọng nói không).
3. Edit bằng HyperFrames (đọc skill `/hyperframes` trước khi dựng):
   - Cắt đoạn im lặng / thừa ở đầu, cuối và giữa.
   - Phụ đề tiếng Việt cháy chữ (burn-in), chữ to dễ đọc trên điện thoại.
     Tạo phụ đề: `NODE_USE_ENV_PROXY=1 hyperframes transcribe <file> -m medium -l vi`
     (model `*.en` chỉ dùng cho tiếng Anh). Rà lại chính tả tiếng Việt trước khi gắn.
   - Cân bằng âm lượng giọng nói; chỉnh màu nhẹ nếu video tối/nhạt.
   - Giữ nguyên khổ hình gốc (dọc vẫn dọc, ngang vẫn ngang).
4. Kiểm tra: `hyperframes check` và `hyperframes snapshot` vài khung hình, xem tận mắt trước khi xuất.
5. Xuất MP4 (H.264) và gửi lại bằng SendUserFile. File quá lớn thì tải lên Google Drive và gửi link.
6. Báo 2–3 dòng: đã làm gì, thời lượng trước/sau.

Nếu người dùng dặn cụ thể (cắt còn 60s, khổ 9:16, thêm nhạc, ...), làm theo lời dặn thay cho mặc định.

## Môi trường

- Chọn môi trường cloud **edit-video** (đã mở mạng cho Hugging Face và PyTorch).
- Lệnh `hyperframes` gọi mạng phải có `NODE_USE_ENV_PROXY=1`.
- Nếu container mới chưa có công cụ: `npm install -g hyperframes@0.9.16`, `pip install kokoro-onnx soundfile`,
  rồi build whisper.cpp theo 3 lệnh đã được cho phép trong `.claude/settings.json` và
  `ln -sf /opt/src/whisper.cpp/build/bin/whisper-cli /usr/local/bin/whisper-cli`.
  Chạy `hyperframes doctor` để kiểm tra.
