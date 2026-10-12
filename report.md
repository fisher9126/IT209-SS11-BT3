# Bài 3: Tối ưu tốc độ CI bằng Gradle Dependency Caching

h
## Thay đổi

Trong bước `actions/setup-java@v5` của `.github/workflows/ci.yml`, thêm tham số `cache: 'gradle'`:

```yaml
- name: Thiết lập môi trường Java JDK 17
  uses: actions/setup-java@v5
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: 'gradle'
```

Action tự tạo cache key từ hash của các file `*.gradle*`, `gradle-wrapper.properties`... Lần chạy đầu chưa có cache nên tải thư viện từ Internet rồi lưu cache. Các lần sau khôi phục cache nếu các file trên không đổi.

## Cách kiểm tra

1. Push lần 1 (chưa có cache), ghi lại thời gian từng bước.
2. Push lần 2 (commit nhỏ bất kỳ, không đổi file Gradle), ghi lại thời gian.
3. Mở log bước `Thiết lập môi trường Java JDK 17` ở lần 2, tìm dòng:

   ```
   Cache restored successfully
   Cache restored from key: gradle-...
   ```

## So sánh thời gian thực thi

Điền số liệu thật từ tab Actions sau khi chạy xong (không điền số ước chừng):

| Bước | Lần 1 (chưa cache) | Lần 2 (đã cache) | Chênh lệch |
| --- | --- | --- | --- |
| Thiết lập Java (`setup-java`) | ... giây | ... giây | ... |
| `./gradlew test` | ... giây | ... giây | ... |
| Toàn bộ Job | ... | ... | ... |

Ảnh chụp màn hình đính kèm:
- `screenshots/run1.png`: log lần 1 (hiện thời gian từng bước)
- `screenshots/run2.png`: log lần 2 (hiện dòng `Cache restored successfully`)

## Nhận xét

Điền sau khi có số liệu. Ví dụ: lần 2 không phải tải lại thư viện nên bước `./gradlew test` ngắn hơn, tổng thời gian Job giảm.

## Lỗi thường gặp

- Không thấy `Cache restored`: lần push đầu chưa có cache, hoặc đã sửa file `build.gradle`/`gradle-wrapper.properties` nên cache key đổi.
- Cache không tạo được: repo cần có file Gradle (`build.gradle`, `build.gradle.kts` hoặc `settings.gradle`) ở thư mục gốc, nếu không action không tính được key.
- Thiếu quyền chạy `./gradlew`: giữ bước `chmod +x ./gradlew`.
