# Rate Limit Management

Ceph Object Gateway cho phép ta thiết lập rate limits cho users và buckets.

Rate limit bao gồm số lượng tối đa hoạt động read và write mỗi phút và số byte mỗi phút có thể write hoặc read cho user hoặc bucket.

Request dạng GET hoặc HEAD được coi là "read request", nếu khống thì sẽ được tính là "write request".

Mỗi Object Gateway theo dõi các metrics của user và bucket riêng biệt, các chỉ số này sẽ không được chia sẻ với các Object gateway khác. Điều đó có nghĩa là các cấu hình limit phải phải được phân chia có số lượng Object Gateway đang hoạt động. 
- Ví dụ: Nếu user A cần cấu hình limit 10 ops/min và có 2 Object Gateway đang hoạt động => phải cấu hình limit cho user A là 5 (10 ops / 2 RGWs). 

Nếu request không được cân bằng giữa các RGW thì các cấu hình rate limit có thể được sử dụng không đúng.
- Ví dụ: set limit là 5 mà có 2 RGWs, nhưng request chỉ được phân tải vào 1 RGW thì giới hạn sẽ là 5, vì giới hạn này được thực thi trên mỗi RGW. Nếu đạt đến giới hạn cho bucket (không phải cho user) hoặc ngược lại thì request cũng sẽ bị hủy bỏ.

