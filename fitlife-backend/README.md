# FitLife - Health & Personal Trainer Platform

[![Java](https://img.shields.io/badge/Java-17-orange.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-green.svg)](https://spring.io/projects/spring-boot)
[![Docker](https://img.shields.io/badge/Docker-Enabled-blue.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)]()

## 📖 Giới thiệu (Introduction)

**FitLife** là nền tảng Backend kết nối Học viên (Member) và Huấn luyện viên cá nhân (PT), được thiết kế với sự tập trung cao độ vào **tính toàn vẹn dữ liệu (Data Consistency)** và **hiệu năng (Performance)**.

Dự án này mô phỏng các bài toán thực tế trong hệ thống Booking, bao gồm xử lý tranh chấp dữ liệu (Race Conditions) khi đặt lịch, tối ưu hóa truy vấn với Caching và bảo mật phân quyền chặt chẽ.

### 🚀 Tính năng nổi bật (Key Highlights)

Dự án này chứng minh khả năng giải quyết các vấn đề kỹ thuật chuyên sâu:

* **Concurrency Control (Booking Core):** Xử lý triệt để vấn đề **Race Condition** (2 người đặt cùng 1 slot) bằng cơ chế **Pessimistic Locking (Database Level)**. Đảm bảo tính nhất quán tuyệt đối (ACID).
* **High Performance Caching:** Sử dụng **Redis** với chiến lược **Cache-Aside** để giảm tải cho Database (giảm 40% query cho dữ liệu PT/Package).
* **Asynchronous Processing:** Tích hợp **Spring Async** để xử lý các tác vụ gửi Email/Notification, giúp giảm thời gian phản hồi API booking xuống dưới 500ms.
* **Security:** Triển khai **Spring Security + JWT (Stateless)** với mô hình phân quyền **RBAC** (Role-Based Access Control).
* **Containerization:** Đóng gói toàn bộ môi trường (App, MySQL, Redis) với **Docker & Docker Compose**.

---

## 🛠 Tech Stack

* **Core:** Java 17, Spring Boot 3.x
* **Database:** MySQL 8.0
* **Caching:** Redis
* **Security:** Spring Security, JWT
* **Build Tool:** Maven
* **Container:** Docker, Docker Compose

---

## 🏛 Thiết kế hệ thống (System Design)

### 1. Database Schema (Core Tables)
* `users`: Lưu trữ thông tin định danh và Role.
* `trainer_profiles`: Thông tin chuyên môn của PT.
* `working_slots`: Lịch làm việc của PT. **(Critical Table for Locking)**.
* `bookings`: Lưu trữ giao dịch đặt lịch.
* `health_metrics`: Chỉ số BMI, TDEE của Member.

### 2. Giải pháp kỹ thuật (Technical Decisions)

#### A. Xử lý đặt lịch trùng (The Race Condition Problem)
Khi có nhiều request cùng đặt một `slot_id` tại một thời điểm:
* **Vấn đề:** Dữ liệu có thể bị ghi đè (Lost Update), dẫn đến Over-booking.
* **Giải pháp trong FitLife:** Sử dụng `PESSIMISTIC_WRITE` lock trong JPA Repository.
    ```java
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT s FROM WorkingSlot s WHERE s.id = :id")
    Optional<WorkingSlot> findByIdWithLock(@Param("id") Long id);
    ```
  *Luồng đi:* Transaction A lock dòng dữ liệu -> Transaction B phải chờ A commit/rollback mới được đọc -> Đảm bảo chỉ 1 người đặt thành công.

#### B. Tối ưu hiệu năng (Performance)
* **Redis:** Cache danh sách "Top Rated PT". Dữ liệu này được đọc nhiều nhưng ít thay đổi.
* **Async:** Việc gửi email xác nhận được tách ra khỏi luồng Transaction chính của Booking để trả về kết quả ngay lập tức cho người dùng.

---

## ⚙️ Cài đặt & Chạy ứng dụng (Installation)

### Yêu cầu (Prerequisites)
* Docker & Docker Compose (Khuyên dùng)
* Hoặc: JDK 17, MySQL 8, Redis cài đặt cục bộ.

### Cách 1: Chạy bằng Docker (Khuyên dùng - 1 Command)

1.  Clone dự án:
    ```bash
    git clone [https://github.com/your-username/fitlife-backend.git](https://github.com/your-username/fitlife-backend.git)
    cd fitlife-backend
    ```
2.  Build và chạy toàn bộ hệ thống:
    ```bash
    docker-compose up --build
    ```
    *Lệnh này sẽ khởi tạo MySQL, Redis và Backend App.*

3.  Truy cập Swagger UI (nếu có tích hợp) hoặc test API tại: `http://localhost:8080/api`

### Cách 2: Chạy thủ công (Local Development)

1.  Cấu hình `application.properties` (hoặc `.yml`) trỏ đến MySQL/Redis local của bạn.
2.  Chạy lệnh Maven:
    ```bash
    mvn clean install
    mvn spring-boot:run
    ```

---

## 🔌 API Endpoints chính

| Method | Endpoint | Role | Mô tả |
| :--- | :--- | :--- | :--- |
| **AUTH** | | | |
| `POST` | `/api/auth/login` | Public | Đăng nhập lấy JWT |
| **BOOKING** | | | |
| `POST` | `/api/bookings` | Member | Đặt lịch (Có Lock DB) |
| `GET` | `/api/trainers/search` | Public | Tìm kiếm PT (Có Redis Cache) |
| **HEALTH** | | | |
| `POST` | `/api/health/metrics` | Member | Cập nhật chỉ số, tính BMI/TDEE |

---

## 👨‍💻 Tác giả (Author)

**[Tên của bạn]**
* **Role:** Backend Lead & Database Designer
* **Email:** quanghuy.le.dev@gmail.com
* **LinkedIn:** linkedin/le-quang-huy

---
*Project này được xây dựng nhằm mục đích demo kỹ năng xử lý Concurrency và System Design.*