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

## Prank video (`assets.prankvideo`)

```
v1/prankvideo/
  christmas/                  # category "christmas"
    configs/<id>.json         # dữ liệu render: chroma key + track 4 góc khung ảnh theo từng frame
  halloween/                  # category "halloween"
    configs/
```

Repo **chỉ** chứa file render (`configs/`). Video mẫu (có vùng nền xanh) và poster đã host sẵn ở
`https://fake-call.varmeta.date/themes/<id>.mp4|.jpg`, nên không tải về repo.

Mỗi item trong `assets.prankvideo.items` có `thumbnail`, `video`, `config`. Field đường dẫn nào bắt đầu bằng
`http://`/`https://` là URL tuyệt đối, app dùng nguyên; còn lại là đường dẫn tương đối theo `basePath` trên jsDelivr.
Hiện tại `thumbnail`/`video` là URL tuyệt đối, `config` là đường dẫn tương đối.
File `config` **không** nhúng vào manifest (mỗi file ~5–15KB, cả bộ ~230KB) — app chỉ tải khi user mở video đó.

Cấu trúc `configs/<id>.json`:

- `size`: `[width, height]` pixel của video gốc — hệ toạ độ của `track`.
- `fps`, `duration` (giây).
- `slot.aspect`: tỉ lệ rộng/cao của khung ảnh — app crop ảnh user về đúng tỉ lệ này trước khi dán.
- `chroma`: tham số chroma key kiểu OBS — `key` (RGB 0..1 của màu nền xanh), `similarity`, `smoothness`, `spill`, `edge`.
- `track`: mỗi phần tử `[t, x1, y1, x2, y2, x3, y3, x4, y4]` — thời điểm `t` (giây) và 4 góc khung ảnh theo thứ tự
  trên-trái, trên-phải, dưới-phải, dưới-trái. Ngoài khoảng thời gian có trong `track` thì khung ảnh không xuất hiện.

## Letter (`assets.letter`)

```
v1/letter/
  walls/sendletter_new_<n>.webp   # ảnh nền lá thư
  walls/halloween_<n>.webp        # ảnh nền lá thư Halloween (export 3x từ Figma: 1029x1367, webp)
```

`assets.letter.categories` giống fakecall (`id`, `name`, `order`). App hiển thị đúng các category có trong manifest, không tự gom thêm category "tất cả".

Mỗi item trong `assets.letter.items`:

- `categoryIds`: các category chứa mẫu này.
- `wall`: ảnh nền, đường dẫn tương đối theo `basePath`.
- `type`: `free` / `reward` / `iap`.
- `signPosition`: `left` / `right` — phía đặt chữ ký.
- `left`, `right`, `top`: khoảng cách vùng viết chữ tới mép trái/phải/trên, tính theo lá thư có chiều rộng tham chiếu **343** —
  khi render trên ảnh thật thì nhân với `chiều rộng ảnh / 343`.
- `order`: thứ tự hiển thị (giữ đúng thứ tự file nguồn, không theo số trong id).

## Versioning

Mỗi lần publish nội dung mới, tăng `version` trong `manifest.json` rồi tag Git tương ứng (`v1.0.0`, `v1.1.0`, ...).
App build URL CDN theo dạng:

```
https://cdn.jsdelivr.net/gh/anhht-varmeta/Android-FakeCall-Data@<version>/<basePath>/<relativePath>
```
