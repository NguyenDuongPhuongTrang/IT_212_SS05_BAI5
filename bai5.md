# BÀI 5: THIẾT KẾ QUY TRÌNH & PROMPT CHO BỘ GIỚI HẠN TẦN SUẤT (RATE LIMITER)

## Mô tả bài toán

Hệ thống SafePay cần triển khai cơ chế Rate Limiting để ngăn chặn DDoS và lạm dụng API.

Quy tắc nghiệp vụ:

* Mỗi API Key được phép gửi tối đa 100 requests trong 1 phút.
* Nếu vượt quá giới hạn, hệ thống trả về HTTP 429 (Too Many Requests).

---

# I. Ý đồ thiết kế quy trình 3 bước

Quy trình được thiết kế theo nguyên tắc từ phân tích → thiết kế → triển khai:

### Bước 1 - Khảo sát giải pháp

Yêu cầu AI đóng vai System Architect để đề xuất nhiều thuật toán Rate Limiting và so sánh ưu nhược điểm.

Mục tiêu:

* Hiểu các lựa chọn hiện có.
* Chọn thuật toán phù hợp.

---

### Bước 2 - Mở rộng sang môi trường phân tán

Giả lập hệ thống chạy trên nhiều máy chủ.

Mục tiêu:

* Nhận diện vấn đề khi dùng Local Memory.
* Đề xuất Redis làm kho dữ liệu tập trung.

---

### Bước 3 - Sinh mã nguồn thực tế

Yêu cầu AI đóng vai Senior Developer.

Mục tiêu:

* Sinh code Spring Boot hoàn chỉnh.
* Tích hợp Redis.
* Có cơ chế Fallback khi Redis gặp sự cố.
* Ghi log bằng @Slf4j.
* Đáp ứng tiêu chuẩn production.

---

# II. BƯỚC 1 - TƯ VẤN VÀ LỰA CHỌN THUẬT TOÁN

## Prompt

```text
Bạn là một System Architect chuyên về Java Backend.

Tôi đang xây dựng API Gateway cho ứng dụng tài chính SafePay.

Yêu cầu:

- Mỗi API Key chỉ được gửi tối đa 100 requests trong 1 phút.
- Vượt giới hạn phải trả về HTTP 429.

Hãy:

1. Đề xuất ít nhất 2 thuật toán Rate Limiting:
   - Fixed Window Counter
   - Token Bucket

2. Giải thích nguyên lý hoạt động.

3. So sánh:
   - Hiệu năng
   - Độ chính xác
   - Độ phức tạp
   - Khả năng mở rộng
   - Khả năng triển khai bằng Java

4. Đưa ra bảng so sánh.

5. Kết luận thuật toán phù hợp nhất cho SafePay.
```

---

## Log Chat Bước 1

### User

```text
(Prompt ở trên)
```

### AI

```text
1. Fixed Window Counter

Nguyên lý:
- Chia thời gian thành các cửa sổ cố định.
- Mỗi request tăng bộ đếm.
- Quá 100 request trong cửa sổ hiện tại sẽ bị chặn.

Ưu điểm:
- Dễ triển khai.
- Tốn ít tài nguyên.

Nhược điểm:
- Có thể xuất hiện burst traffic tại ranh giới cửa sổ.

------------------------------------

2. Token Bucket

Nguyên lý:
- Mỗi API Key sở hữu một bucket chứa token.
- Request muốn được xử lý phải tiêu thụ token.
- Token được nạp lại theo thời gian.

Ưu điểm:
- Kiểm soát burst tốt.
- Linh hoạt.

Nhược điểm:
- Triển khai phức tạp hơn.

------------------------------------

BẢNG SO SÁNH

| Tiêu chí | Fixed Window | Token Bucket |
|-----------|-------------|-------------|
| Độ phức tạp | Thấp | Trung bình |
| Hiệu năng | Cao | Cao |
| Độ chính xác | Trung bình | Cao |
| Burst Control | Kém | Tốt |
| Distributed | Tốt | Tốt |
| Java Spring | Dễ | Trung bình |

Kết luận:

Token Bucket phù hợp hơn với SafePay vì khả năng kiểm soát burst traffic tốt hơn và được sử dụng phổ biến trong API Gateway hiện đại.
```

---

# III. BƯỚC 2 - THIẾT KẾ CHO HỆ THỐNG PHÂN TÁN

## Prompt

```text
Giả sử SafePay được triển khai trên cụm 3 máy chủ Spring Boot phía sau Load Balancer.

Nếu bộ đếm Rate Limiting được lưu trong Local Memory của từng máy chủ thì sẽ xảy ra sai lệch.

Hãy:

1. Phân tích nguyên nhân.
2. Cho ví dụ thực tế.
3. Đề xuất Redis làm kho lưu trữ tập trung.
4. Thiết kế luồng xử lý.
5. Trình bày sơ đồ kiến trúc dạng text.
6. Giải thích Redis INCR.
7. Nêu ưu và nhược điểm.
```

---

## Log Chat Bước 2

### User

```text
(Prompt ở trên)
```

### AI

```text
Nguyên nhân:

Mỗi server có bộ đếm riêng.

Ví dụ:

Server A = 40 request
Server B = 35 request
Server C = 45 request

Tổng thực tế = 120 request

Nhưng không server nào vượt 100 request.

=> Rate Limiting bị sai.

----------------------------------

Giải pháp Redis

Redis trở thành kho lưu trữ tập trung.

Kiến trúc:

Client
   |
Load Balancer
   |
----------------------
|      |      |
S1     S2     S3
 \      |     /
      Redis

----------------------------------

Luồng xử lý

1. Nhận request.
2. Lấy API Key.
3. Redis INCR key.
4. Nếu count > 100:
      HTTP 429
5. Ngược lại:
      Cho phép xử lý.

----------------------------------

Redis INCR

Lệnh INCR tăng giá trị atomically.

Dù nhiều request đồng thời vẫn đảm bảo dữ liệu chính xác.

----------------------------------

Ưu điểm

- Chính xác trong môi trường distributed.
- Hiệu năng cao.
- Dễ scale.

Nhược điểm

- Phụ thuộc Redis.
- Redis trở thành điểm lỗi tiềm năng.
```

---

# IV. BƯỚC 3 - SINH MÃ NGUỒN SPRING BOOT

## Prompt

```text
Bạn là Senior Java Developer.

Hãy thiết kế Rate Limiter phân tán cho SafePay bằng:

- Spring Boot 3
- Spring Data Redis
- Lombok
- SLF4J

Yêu cầu:

1. Giới hạn 100 request/phút/API Key.
2. Redis là nơi lưu bộ đếm.
3. Trả HTTP 429 khi vượt giới hạn.
4. Viết:
   - RedisConfig
   - RateLimiterService
   - RateLimitFilter
   - Exception
   - GlobalExceptionHandler
5. Ghi log bằng @Slf4j.
6. Đảm bảo thread-safe.
7. Nếu Redis bị sập:
   - Phân tích rủi ro.
   - Tạo cơ chế fallback bằng ConcurrentHashMap.
8. Tuân thủ Clean Code.
```

---

## Log Chat Bước 3

### User

```text
(Prompt ở trên)
```

### AI

### Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

### RateLimitExceededException

```java
public class RateLimitExceededException extends RuntimeException {

    public RateLimitExceededException(String message) {
        super(message);
    }
}
```

### RateLimiterService

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class RateLimiterService {

    private static final long LIMIT = 100;

    private final RedisTemplate<String, Integer> redisTemplate;

    private final ConcurrentHashMap<String, AtomicInteger> fallbackStorage =
            new ConcurrentHashMap<>();

    public void validate(String apiKey) {

        String key = "rate_limit:" + apiKey;

        try {

            Long count = redisTemplate.opsForValue().increment(key);

            if (count == 1) {
                redisTemplate.expire(
                        key,
                        Duration.ofMinutes(1)
                );
            }

            log.info("API Key={} Count={}", apiKey, count);

            if (count > LIMIT) {

                log.warn(
                        "Rate limit exceeded for {}",
                        apiKey
                );

                throw new RateLimitExceededException(
                        "Too Many Requests"
                );
            }

        } catch (Exception ex) {

            log.error(
                    "Redis unavailable. Switching to fallback mode.",
                    ex
            );

            AtomicInteger counter =
                    fallbackStorage.computeIfAbsent(
                            apiKey,
                            k -> new AtomicInteger(0)
                    );

            int count = counter.incrementAndGet();

            if (count > LIMIT) {

                throw new RateLimitExceededException(
                        "Too Many Requests"
                );
            }
        }
    }
}
```

### RateLimitFilter

```java
@Component
@RequiredArgsConstructor
public class RateLimitFilter extends OncePerRequestFilter {

    private final RateLimiterService rateLimiterService;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String apiKey = request.getHeader("X-API-KEY");

        rateLimiterService.validate(apiKey);

        filterChain.doFilter(request, response);
    }
}
```

### GlobalExceptionHandler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(
            RateLimitExceededException.class
    )
    public ResponseEntity<String> handle() {

        return ResponseEntity
                .status(HttpStatus.TOO_MANY_REQUESTS)
                .body("Too Many Requests");
    }
}
```

### Fallback Strategy

Nếu Redis bị sập:

1. Ghi log ERROR.
2. Chuyển sang ConcurrentHashMap.
3. Tiếp tục giới hạn lưu lượng tạm thời.
4. Bảo vệ hệ thống trước khi Redis phục hồi.

```

---

# V. KẾT LUẬN

Giải pháp cuối cùng:

- Thuật toán: Token Bucket hoặc Fixed Window.
- Redis làm bộ đếm tập trung.
- HTTP 429 khi vượt ngưỡng.
- Spring Data Redis.
- @Slf4j để giám sát.
- Fallback bằng ConcurrentHashMap khi Redis lỗi.

Thiết kế này phù hợp với môi trường production và hệ thống API Gateway phân tán của SafePay.
```
