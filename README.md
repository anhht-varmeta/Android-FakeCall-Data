# Android-FakeCall-Data

CDN content cho app FakeCall, phân phối qua [jsDelivr](https://www.jsdelivr.com/?docs=gh) trỏ thẳng vào repo GitHub này.
App đọc URL qua Firebase Remote Config key `cdn_data` (JSON `manifest.json`), không hardcode version trong app.

## Cấu trúc

```
manifest.json               # catalog: version, schemaVersion, danh sách item + đường dẫn tương đối
v1/
  fakecall/
    santa_call/              # bộ sưu tập "Santa Call" - video call kèm voice riêng
      videos/
      voices/
      thumbs/
    prank_call/               # bộ sưu tập "Prank Call" - nhiều category (santa, celebrities, superheros, ...)
      <category>/...
```

Các mục sau (ví dụ `v1/prankvideo/...`) sẽ được thêm khi có data crawl mới cho tính năng khác.

## Versioning

Mỗi lần publish nội dung mới, tăng `version` trong `manifest.json` rồi tag Git tương ứng (`v1.0.0`, `v1.1.0`, ...).
App build URL CDN theo dạng:

```
https://cdn.jsdelivr.net/gh/anhht-varmeta/Android-FakeCall-Data@<version>/<basePath>/<relativePath>
```
