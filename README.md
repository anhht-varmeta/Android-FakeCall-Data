# Android-FakeCall-Data

CDN content cho app FakeCall, phân phối qua [jsDelivr](https://www.jsdelivr.com/?docs=gh) trỏ thẳng vào repo GitHub này.
App đọc URL qua Firebase Remote Config key `cdn_data` (JSON `manifest.json`), không hardcode version trong app.

## Cấu trúc

```
manifest.json               # catalog: version, schemaVersion, 1 danh sách category + 1 danh sách item chung
v1/
  fakecall/
    santa_video_call/        # category "santa_video_call" - video call kèm voice (ringtone) riêng
      thumbs/
      videos/
      voices/
    santa/                    # category "santa_calls" - item nằm phẳng ngay trong category
      thumbs/
      videos/
    celebrities/               # category có sub-folder theo từng nhân vật
      boy_friend/
        thumbs/
        videos/
      camila_cabello/
        thumbs/
        videos/
      ...
    superheros/
      batman/
        thumbs/
        videos/
      ironman/
        thumbs/
        videos/
      ...
    funny/                     # category vừa có item phẳng (funny_1) vừa có sub-folder (boy/, girl/)
      thumbs/
      videos/
      boy/
        thumbs/
        videos/
      girl/
        thumbs/
        videos/
    ghost/ kpophunter/ pedri/ skibidi/ speed/ strangerthings/ scarys/ elsa/ ...
```

Mọi category đều nằm phẳng trực tiếp dưới `v1/fakecall/` (không còn tách riêng theo nguồn crawl `santa_call`/`prank_call`).
Bên trong mỗi category (hoặc mỗi sub-folder nhân vật, nếu category đó chia nhỏ theo nhân vật), file luôn được
tách theo loại: `thumbs/`, `videos/`, và `voices/` (chỉ xuất hiện ở nhóm nào thực sự có file voice riêng).

`manifest.json` gộp cả 2 nguồn thành **một** danh sách `assets.fakecall.items`, mỗi item thuộc 1+ category
trong `assets.fakecall.categories`. Mỗi category có sẵn field `name` — tên hiển thị thật (không phải key),
ví dụ `{"id": "superheros_calls", "name": "Super Heroes", "order": 30}`. App hiển thị thẳng field này,
**không** map qua string resource trong app — nên khi thêm category mới chỉ cần thêm vào đây (id + name +
order) rồi tăng version, không cần update app.

Mỗi item có 2 field media tùy chọn:

- `video`: có giá trị → app cho phép "video call". `null` → không có video.
- `voice`: có giá trị → app có thể phát riêng ringtone/voice trước khi vào call. Hiện tại chỉ nhóm
  category `santa_video_call` có field này; các item còn lại `voice: null` vì audio đã nhúng sẵn trong `video`.

Item không có `video` (chỉ có `voice`) thì app chỉ hiện option audio call — hiện tại chưa có item nào thuộc
dạng này, nhưng schema đã sẵn sàng cho trường hợp đó khi crawl thêm data sau này.

Các mục sau (ví dụ `v1/prankvideo/...`) sẽ được thêm khi có data crawl mới cho tính năng khác.

## Versioning

Mỗi lần publish nội dung mới, tăng `version` trong `manifest.json` rồi tag Git tương ứng (`v1.0.0`, `v1.1.0`, ...).
App build URL CDN theo dạng:

```
https://cdn.jsdelivr.net/gh/anhht-varmeta/Android-FakeCall-Data@<version>/<basePath>/<relativePath>
```
