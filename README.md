# Giọng Trầm

Phòng chỉnh giọng chạy thẳng trong trình duyệt: kéo giọng bất kỳ về chất **nữ trẻ 18–20, tông trầm ấm**.
Trang tự đo cao độ giọng gốc, tính mức dịch cần thiết để bám target, rồi dịch cao độ mà **giữ nguyên
formant** — nên giọng trầm xuống chứ không bị “nam hoá”.

Một file `index.html` duy nhất, không thư viện, không server backend, không gửi tiếng nói đi đâu cả —
mọi thứ xử lý ngay trên máy.

## Dùng thế nào

Trang **bắt buộc phải mở qua http/https**, không mở thẳng bằng cách bấm đúp vào file: trình duyệt chặn
AudioWorklet trên `file://`. Hai cách:

- Bật GitHub Pages cho repo rồi vào link đó.
- Hoặc chạy tại chỗ: `npx http-server .` rồi mở `http://127.0.0.1:8080`.

Có hai chế độ:

- **Thu từ mic** — bật mic, nói vài câu cho đồng hồ đo được cao độ gốc, bấm *Thu giọng*, thu xong ra
  file WAV nghe thử và tải về ngay. Muốn nghe lại trong lúc thu thì cắm tai nghe rồi bật *Nghe lại*,
  không là hú.
- **Đổi file có sẵn** — thả vào một file mp3/wav/m4a/mp4/webm, trang đo cao độ trung vị của cả file,
  xử lý offline rồi xuất WAV.

## Các nút chính

| Nút | Làm gì |
|---|---|
| Target cao độ | Cao độ muốn giọng bám vào. Giọng nữ trầm thường rơi 175–200 Hz (vùng hồng trên đồng hồ). |
| Tự bám target | Tự đo giọng gốc rồi tính mức dịch. Tắt đi để tự kéo theo nửa cung. |
| Formant | Độ “to nhỏ” của khoang giọng. 1,00 giữ nguyên chất người, kéo lên nghe trẻ và nhỏ người hơn. Nam sang nữ thường cần 1,15–1,30. |
| Formant tự động | Suy formant ra từ mức dịch: dịch lên nhiều thì nới khoang giọng cho ra chất nữ, dịch xuống thì chỉ nâng nhẹ để khỏi trôi về giọng nam. |
| Tinh chỉnh âm sắc | EQ (ấm dày / ù đục / rõ chữ / tiếng xì / hơi thở), vang, nén, âm lượng ra. |

Bảy preset dựng sẵn từ *Trầm ấm 18–20* tới *Nam sang nữ trầm*; chọn cái gần nhất rồi tinh chỉnh sau.

## Bên trong

Lõi là một `AudioWorklet` phase vocoder tự viết (FFT 2048, overlap 4× khi chạy mic và 8× khi xuất file):

- **Dịch cao độ giữ formant.** Mỗi frame tách phổ thành *envelope* (đường bao khoang giọng, lấy bằng
  cepstral liftering) và *excitation* (phần hoà âm). Chỉ excitation bị dịch theo cao độ; envelope được
  đắp lại ở vị trí `f/formant`, nên cao độ và formant chỉnh được độc lập nhau.
- **Identity phase locking.** Chỉ các đỉnh phổ mới tích luỹ pha; các bin quanh một đỉnh giữ nguyên độ
  lệch pha so với đỉnh đó, nhờ vậy từng hoà âm vẫn là một sóng liền mạch thay vì bị nhoè.
- **Bắt transient.** Thấy năng lượng phổ vọt lên là biết có phụ âm bật tới, pha được reset về pha phân
  tích để tiếng “t”, “p”, “ch” không bị bết.
- **Đo cao độ** bằng autocorrelation chuẩn hoá trên tín hiệu hạ mẫu về 8 kHz, có chặn lỗi nhầm quãng
  tám, lấy trung vị 9 khung gần nhất cho khỏi giật.

Đo trên tín hiệu tổng hợp: sai số cao độ dưới 0,5 %, hoà âm cơ bản cao hơn thành phần lạ gần nhất
29–39 dB, khuếch đại tổng thể 0,999× (unity), và nhanh hơn thời gian thực khoảng 10× ở chế độ mic.
