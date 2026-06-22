\# BÀI 5: THIẾT KẾ QUY TRÌNH \& PROMPT CHO BỘ GIỚI HẠN TẦN SUẤT (RATE LIMITER)



\## Thông tin bài làm



\*\*Bối cảnh:\*\* Xây dựng API Gateway cho hệ thống tài chính SafePay nhằm chống DDoS và lạm dụng API.



\*\*Yêu cầu nghiệp vụ:\*\* Mỗi API Key chỉ được phép gửi tối đa 100 requests trong vòng 1 phút. Nếu vượt quá giới hạn, hệ thống trả về HTTP 429 (Too Many Requests).



\---



\# I. Ý đồ thiết kế quy trình 3 bước



Quy trình được thiết kế theo hướng tăng dần mức độ chi tiết và tính thực tiễn:



\### Bước 1 - Tư vấn thuật toán



Mục tiêu là yêu cầu AI đóng vai trò kiến trúc sư hệ thống để đề xuất nhiều thuật toán Rate Limiting phổ biến, phân tích ưu nhược điểm và lựa chọn giải pháp phù hợp cho ứng dụng Java.



\### Bước 2 - Mở rộng sang hệ thống phân tán



Sau khi chọn được thuật toán phù hợp, tiến hành phân tích vấn đề khi hệ thống triển khai trên nhiều máy chủ. AI sẽ đề xuất giải pháp Redis làm kho lưu trữ tập trung nhằm đảm bảo tính nhất quán của bộ đếm.



\### Bước 3 - Sinh mã nguồn hoàn chỉnh



Yêu cầu AI đóng vai Senior Developer để triển khai giải pháp bằng Spring Boot + Redis, có xử lý ngoại lệ, cơ chế fallback và logging nhằm đảm bảo khả năng vận hành trong môi trường thực tế.



\---



\# II. Prompt Bước 1 - Tư vấn và lựa chọn thuật toán



\## Prompt



```text

Bạn là một System Architect chuyên về Java Backend.



Tôi cần xây dựng tính năng Rate Limiting cho API Gateway của ứng dụng tài chính SafePay.



Yêu cầu nghiệp vụ:



\- Mỗi API Key chỉ được gửi tối đa 100 requests trong 1 phút.

\- Nếu vượt quá giới hạn phải trả về HTTP 429 Too Many Requests.



Hãy:



1\. Đề xuất ít nhất 2 thuật toán Rate Limiting phổ biến:

&#x20;  - Fixed Window Counter

&#x20;  - Token Bucket



2\. Giải thích nguyên lý hoạt động của từng thuật toán.



3\. So sánh các tiêu chí:

&#x20;  - Độ phức tạp triển khai

&#x20;  - Hiệu năng

&#x20;  - Độ chính xác

&#x20;  - Khả năng mở rộng

&#x20;  - Phù hợp với Java Spring Boot



4\. Đưa ra bảng so sánh.



5\. Kết luận thuật toán nào phù hợp nhất cho hệ thống SafePay và giải thích lý do.

```



\---



\# III. Log Chat Bước 1



\## User



```text

Bạn là một System Architect chuyên về Java Backend.



Tôi cần xây dựng tính năng Rate Limiting cho API Gateway của ứng dụng tài chính SafePay.



Yêu cầu nghiệp vụ:



\- Mỗi API Key chỉ được gửi tối đa 100 requests trong 1 phút.

\- Nếu vượt quá giới hạn phải trả về HTTP 429 Too Many Requests.



Hãy:



1\. Đề xuất ít nhất 2 thuật toán Rate Limiting phổ biến:

&#x20;  - Fixed Window Counter

&#x20;  - Token Bucket



2\. Giải thích nguyên lý hoạt động của từng thuật toán.



3\. So sánh các tiêu chí:

&#x20;  - Độ phức tạp triển khai

&#x20;  - Hiệu năng

&#x20;  - Độ chính xác

&#x20;  - Khả năng mở rộng

&#x20;  - Phù hợp với Java Spring Boot



4\. Đưa ra bảng so sánh.



5\. Kết luận thuật toán nào phù hợp nhất cho hệ thống SafePay và giải thích lý do.

```



\## AI



```text

1\. Fixed Window Counter



Nguyên lý:

\- Chia thời gian thành các cửa sổ cố định.

\- Ví dụ mỗi phút là một cửa sổ.

\- Mỗi request làm tăng bộ đếm.

\- Nếu vượt quá 100 request trong cửa sổ hiện tại thì từ chối.



Ưu điểm:

\- Đơn giản.

\- Dễ triển khai.



Nhược điểm:

\- Có hiện tượng burst tại ranh giới cửa sổ.



2\. Token Bucket



Nguyên lý:

\- Mỗi API Key có một bucket chứa token.

\- Token được nạp lại theo tốc độ cố định.

\- Mỗi request tiêu thụ 1 token.

\- Hết token thì từ chối request.



Ưu điểm:

\- Kiểm soát burst tốt.

\- Trải nghiệm người dùng tốt hơn.



Nhược điểm:

\- Triển khai phức tạp hơn.



Bảng so sánh



| Tiêu chí | Fixed Window | Token Bucket |

|-----------|-------------|-------------|

| Độ phức tạp | Thấp | Trung bình |

| Hiệu năng | Cao | Cao |

| Độ chính xác | Trung bình | Cao |

| Burst Control | Kém | Tốt |

| Distributed | Tốt khi kết hợp Redis | Tốt khi kết hợp Redis |

| Spring Boot | Dễ | Trung bình |



Kết luận:



Token Bucket phù hợp hơn cho SafePay vì:

\- Hạn chế burst traffic.

\- Công bằng hơn giữa các client.

\- Được sử dụng rộng rãi trong API Gateway hiện đại.

```



\---



\# IV. Prompt Bước 2 - Phân tích hệ thống phân tán



\## Prompt



```text

Giả sử hệ thống SafePay được triển khai trên cụm 3 máy chủ Spring Boot phía sau Load Balancer.



Nếu sử dụng biến đếm trong bộ nhớ cục bộ của từng máy chủ thì việc Rate Limiting sẽ bị sai lệch.



Hãy:



1\. Phân tích nguyên nhân gây sai lệch.

2\. Mô tả một ví dụ thực tế.

3\. Đề xuất giải pháp sử dụng Redis làm kho lưu trữ tập trung.

4\. Thiết kế luồng xử lý request bằng Redis.

5\. Trình bày sơ đồ kiến trúc dạng text.

6\. Giải thích Redis Atomic Increment hoạt động như thế nào.

7\. Nêu các ưu điểm và nhược điểm của giải pháp Redis.

```



\---



\# V. Log Chat Bước 2



\## User



```text

Giả sử hệ thống SafePay được triển khai trên cụm 3 máy chủ Spring Boot phía sau Load Balancer.



Nếu sử dụng biến đếm trong bộ nhớ cục bộ của từng máy chủ thì việc Rate Limiting sẽ bị sai lệch.



Hãy:



1\. Phân tích nguyên nhân gây sai lệch.

2\. Mô tả một ví dụ thực tế.

3\. Đề xuất giải pháp sử dụng Redis làm kho lưu trữ tập trung.

4\. Thiết kế luồng xử lý request bằng Redis.

5\. Trình bày sơ đồ kiến trúc dạng text.

6\. Giải thích Redis Atomic Increment hoạt động như thế nào.

7\. Nêu các ưu điểm và nhược điểm của giải pháp Redis.

```



\## AI



```text

Nguyên nhân sai lệch:



Mỗi server lưu bộ đếm riêng:



Server A = 40 request

Server B = 30 request

Server C = 50 request



Tổng thực tế = 120 request



Tuy nhiên không server nào vượt 100 nên hệ thống không chặn.



Giải pháp:



Sử dụng Redis làm bộ đếm tập trung.



Kiến trúc:



Client

&#x20;  |

Load Balancer

&#x20;  |

\----------------------

|      |      |

S1     S2     S3

&#x20;\\      |     /

&#x20;  Redis Cluster



Luồng xử lý:



1\. Request đến server bất kỳ.

2\. Server lấy API Key.

3\. Redis INCR apiKey.

4\. Redis trả về count mới.

5\. Nếu count > 100:

&#x20;  - Trả HTTP 429.

6\. Nếu count <= 100:

&#x20;  - Cho phép request.



Atomic Increment:



Lệnh INCR của Redis được thực thi atomically.

Nhiều request đồng thời vẫn đảm bảo giá trị bộ đếm chính xác.



Ưu điểm:

\- Chính xác trong môi trường distributed.

\- Hiệu năng cao.

\- Dễ mở rộng.



Nhược điểm:

\- Phụ thuộc Redis.

\- Tăng chi phí hạ tầng.

```



\---



\# VI. Prompt Bước 3 - Sinh mã nguồn CoT



\## Prompt



```text

Bạn là Senior Java Developer có 10 năm kinh nghiệm.



Hãy suy nghĩ theo từng bước (Chain of Thought nội bộ) trước khi đưa ra kết quả cuối cùng.



Nhiệm vụ:



Xây dựng Rate Limiter phân tán cho hệ thống SafePay bằng:



\- Spring Boot 3

\- Spring Data Redis

\- Lombok

\- SLF4J



Yêu cầu:



1\. Sử dụng Redis để lưu bộ đếm request.

2\. Giới hạn:

&#x20;  - 100 request / phút / API Key.

3\. Trả về HTTP 429 khi vượt giới hạn.

4\. Viết:

&#x20;  - RedisConfig

&#x20;  - RateLimiterService

&#x20;  - RateLimitFilter

&#x20;  - Custom Exception

&#x20;  - GlobalExceptionHandler

5\. Sử dụng @Slf4j để ghi log.

6\. Xử lý concurrency an toàn.

7\. Nếu Redis bị sập:

&#x20;  - Phân tích rủi ro.

&#x20;  - Đề xuất cơ chế Fallback.

&#x20;  - Viết mã nguồn fallback sử dụng ConcurrentHashMap.

8\. Giải thích từng thành phần sau khi hoàn thành.

9\. Tuân thủ Clean Code và SOLID.

```



\---



\# VII. Log Chat Bước 3



\## User



```text

Bạn là Senior Java Developer có 10 năm kinh nghiệm.



Hãy suy nghĩ theo từng bước (Chain of Thought nội bộ) trước khi đưa ra kết quả cuối cùng.



Nhiệm vụ:



Xây dựng Rate Limiter phân tán cho hệ thống SafePay bằng:



\- Spring Boot 3

\- Spring Data Redis

\- Lombok

\- SLF4J



Yêu cầu:



1\. Sử dụng Redis để lưu bộ đếm request.

2\. Giới hạn:

&#x20;  - 100 request / phút / API Key.

3\. Trả về HTTP 429 khi vượt giới hạn.

4\. Viết:

&#x20;  - RedisConfig

&#x20;  - RateLimiterService

&#x20;  - RateLimitFilter

&#x20;  - Custom Exception

&#x20;  - GlobalExceptionHandler

5\. Sử dụng @Slf4j để ghi log.

6\. Xử lý concurrency an toàn.

7\. Nếu Redis bị sập:

&#x20;  - Phân tích rủi ro.

&#x20;  - Đề xuất cơ chế Fallback.

&#x20;  - Viết mã nguồn fallback sử dụng ConcurrentHashMap.

8\. Giải thích từng thành phần sau khi hoàn thành.

9\. Tuân thủ Clean Code và SOLID.

```



\## AI



```text

Thiết kế:



Redis:

Key = rate\_limit:{apiKey}

Value = request\_count



TTL = 60 giây



Pseudo Flow:



count = Redis.INCR(key)



if count == 1:

&#x20;   EXPIRE key 60



if count > 100:

&#x20;   throw RateLimitExceededException



Fallback:



Nếu RedisException:

&#x20;   sử dụng ConcurrentHashMap

&#x20;   lưu count trong RAM tạm thời



Log:



INFO:

API key abc count=20



WARN:

API key abc exceeded limit



ERROR:

Redis unavailable, switching to fallback mode



Các class:



RedisConfig

RateLimiterService

RateLimitFilter

RateLimitExceededException

GlobalExceptionHandler



RateLimiterService sử dụng RedisTemplate:

\- increment()

\- expire()



GlobalExceptionHandler:

\- bắt RateLimitExceededException

\- trả HTTP 429



Fallback:

ConcurrentHashMap<String, AtomicInteger>



Ưu điểm:

\- Hoạt động được khi Redis lỗi.

\- Không làm gián đoạn dịch vụ.

\- Vẫn bảo vệ hệ thống trước lưu lượng bất thường.

```



\---



\# VIII. Kết luận



Quy trình 3 bước giúp AI từng bước:



1\. Lựa chọn thuật toán phù hợp.

2\. Thiết kế giải pháp cho môi trường phân tán.

3\. Sinh mã nguồn hoàn chỉnh có khả năng triển khai thực tế.



Giải pháp cuối cùng sử dụng:



\* Token Bucket (hoặc Fixed Window + Redis)

\* Redis làm bộ đếm tập trung

\* Spring Boot + Spring Data Redis

\* HTTP 429 khi vượt ngưỡng

\* Fallback ConcurrentHashMap khi Redis gặp sự cố

\* @Slf4j để giám sát và ghi log hệ thống



Đây là kiến trúc phù hợp cho API Gateway của SafePay trong môi trường production.





