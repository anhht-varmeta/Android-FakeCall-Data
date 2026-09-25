# Android-FakeCall-Data

CDN content cho app FakeCall, phân phối qua [jsDelivr](https://www.jsdelivr.com/?docs=gh) trỏ thẳng vào repo GitHub này.
App đọc URL qua Firebase Remote Config key `cdn_data` (JSON `manifest.json`), không hardcode version trong app.

## Cấu trúc

```
manifest.json               # catalog: version, schemaVersion, 1 danh sách category + 1 danh sách item chung
v1/
  fakecall/
    santa_call/              # media gốc "Santa Call" - video + voice (ringtone) riêng
      videos/
      voices/
      thumbs/
    prank_call/               # media gốc "Prank Call" - nhiều category (celebrities, superheros, ...)
      <category>/...
```

`manifest.json` gộp cả 2 nguồn thành **một** danh sách `assets.fakecall.items`, mỗi item thuộc 1+ category
trong `assets.fakecall.categories` và có 2 field media tùy chọn:

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
