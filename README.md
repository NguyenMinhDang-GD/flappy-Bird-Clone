# Flappy Bird Remake (Unity 2D) - Technical Summary

Tài liệu tổng hợp các kiến thức kỹ thuật cơ bản và những bài học kinh nghiệm thu được trong quá trình phát triển tựa game **Flappy Bird 2D (Unity 6)**.

---

## 🛠️ 1. Các Kỹ thuật Lập trình Cơ bản Đã Sử Dụng

### 1.1. Vật lý 2D & Xử lý Điều khiển Nhân vật (Bird)
* **Vòng đời MonoBehaviour:** Tách biệt rõ ràng việc đọc phím bấm của người chơi trong `Update()` và thực thi vật lý trong `FixedUpdate()`.
* **Cơ chế Nhảy (Jump):** Can thiệp trực tiếp vào vận tốc `linearVelocity` của `Rigidbody2D` thay vì dùng lực đẩy `AddForce`. Cách này triệt tiêu hoàn toàn đà rơi cũ, giúp cú nhảy dứt khoát và đồng nhất ở mọi lần bấm.
* **Chống Spam Phím (Jump Cooldown):** Kết hợp sự kiện nhấn phím một lần (`wasPressedThisFrame`) với mốc thời gian hồi chiêu (`Time.time + cooldown`) để ngăn nhân vật bị bay vọt lên trời khi bấm phím liên tục.
* **Xoay Góc Nhìn (Juice / Game Feel):** Tách riêng Component xoay góc gắn vào đối tượng con `visual`. Khi vừa bấm nhảy, đầu chim bẻ thẳng lên $+30^\circ$ ngay lập tức; khi thả tay rơi tự do, đầu chim từ từ chúi xuống $-90^\circ$ theo thời gian.

### 1.2. Hệ thống Va chạm & Tính điểm (Collision vs Trigger)
* **`OnCollisionEnter2D` (Va chạm cứng):** Dùng cho Cột ống nước và Mặt đất (bật va chạm vật lý cứng để phát hiện khi chim đâm vào vật cản và kích hoạt trạng thái Game Over).
* **`OnTriggerEnter2D` (Cảm biến xuyên qua):** Đặt một Collider vô hình bật `Is Trigger` ở khoảng trống giữa hai ống nước. Khi chim bay xuyên qua mà không chạm ống, hệ thống ghi nhận và cộng điểm.

### 1.3. Tối ưu Bộ nhớ với Object Pooling (Hệ thống Ống nước)
* **Tái chế Đối tượng (Object Pooling):** Không dùng `Instantiate` và `Destroy` liên tục (để tránh sinh rác bộ nhớ gây giật lag do Garbage Collection).
* **Cơ chế Hoạt động:** Khởi tạo sẵn một số lượng cố định (khoảng 5 cặp ống nước). Khi ống trôi ra khỏi mép trái màn hình, chỉ cần tắt đi (`SetActive(false)`) để đưa về hồ bơi chờ tái sử dụng.
* **Bộ sinh ống tự động (Spawner):** Sử dụng `Coroutine` (`IEnumerator` kết hợp `WaitForSeconds`) để đếm nhịp và lấy các ống rảnh rỗi ra, đặt độ cao Y ngẫu nhiên rồi kích hoạt lại.

### 1.4. Hệ thống Giao diện & Lưu trữ Dữ liệu (UI & Data)
* **Tách biệt Thứ tự Vẽ 2D (Sorting Layer):** Phân tầng hiển thị rõ ràng (`Background` $\rightarrow$ `Pipes` $\rightarrow$ `Ground` $\rightarrow$ `Player` $\rightarrow$ `UI`) để các hình ảnh không bị đè lộn xộn lên nhau.
* **Giao diện Menu Thua cuộc (Game Over UI):** Thiết kế bảng popup hiển thị đầy đủ Điểm hiện tại (`Score`), Điểm ván trước (`Previous Score`), và Kỷ lục (`Best Score`) kèm nút chơi lại màn chơi (`SceneManager.LoadScene`).
* **Lưu trữ Bền vững (`PlayerPrefs`):** Sử dụng hệ thống lưu trữ nhẹ dạng Key-Value của Unity để lưu điểm kỷ lục và điểm ván trước ngay trên ổ cứng, giúp game vẫn nhớ điểm số kể cả khi tắt đi bật lại.

---

## 💡 2. Những Gì Đã Học Được Thông Qua Dự Án Này

1. **Tư duy Kiến trúc Component-Based:**
   * Hiểu bản chất của Unity là **Lắp ghép (Composition)** thay vì **Kế thừa (Inheritance)**. Một đối tượng được tạo thành bằng cách ghép nối các Component nhỏ độc lập (`Rigidbody2D`, `Collider2D`, `SpriteRenderer`, Script...).
   * Biết cách tách biệt giữa **Logic/Vật lý** (đối tượng cha) và **Hình ảnh/Hiển thị** (đối tượng con `visual`) để tránh lỗi méo góc va chạm khi xoay nhân vật.

2. **Làm chủ Vòng đời & Bản chất Hệ thống Unity:**
   * Phân biệt rõ trách nhiệm của các hàm sự kiện: `Awake` (khởi tạo), `Update` (input, logic từng khung hình), `FixedUpdate` (vật lý cố định thời gian), và `OnEnable` (reset trạng thái khi tái sử dụng).
   * Hiểu rõ sự khác biệt giữa hai loại va chạm: Va chạm cứng (`Collision`) và Cảm biến đi xuyên qua (`Trigger`).

3. **Tư duy Tối ưu Hiệu năng từ Sớm:**
   * Hiểu tác hại của việc tạo/xóa liên tục (`Instantiate`/`Destroy`) đối với bộ nhớ RAM và CPU.
   * Biết cách áp dụng mẫu thiết kế **Object Pooling** để game chạy mượt mà ở tốc độ 60/120 FPS mà dung lượng bộ nhớ luôn ổn định.

4. **Kỹ năng Tổ chức Dự án Chuẩn Chỉnh:**
   * Biết cách phân chia cây thư mục khoa học (`Prefabs`, `Scripts`, `Sprites`, `Audio`) và phân nhóm Hierarchy gọn gàng (`--PLAYER--`, `--SYSTEMS--`, `--UI--`, `--ENVIROMENT--`).
   * Hiểu đúng bản chất của **Prefab**: Chỉ đóng gói những đối tượng cần nhân bản hàng loạt (ống nước, chữ số điểm) hoặc cần lưu trữ mẫu (nhân vật); tránh biến các thư mục ảo hoặc các Manager duy nhất thành Prefab.

Link test game https://kamera-gd.itch.io/flappy-bird-clone
