# 概要
local的主要问题是可以进行路径穿越去读到服务器所存储的其他文件，即非法查看其他文件  

# 具体步骤
## 1.
<img width="2559" height="1599" alt="image" src="https://github.com/user-attachments/assets/7bda5ac1-f9ff-4f38-a8df-1f0ff08229c3" />
这是正常按照选项选一个，然后提交显示的页面，似乎没办法包括其他文件，但这只是前端的限制而已，我们用bp抓包就可以去直接绕过这层限制  

## 2.
<img width="2559" height="1599" alt="image" src="https://github.com/user-attachments/assets/58e5a87f-c496-47bf-b5cb-7cf62a623223" />
这是用bp抓到的数据包，然后可以看到很明显的 filename= ，我们就可以去尝试去修改这个参数去尝试访问其他的文件，先试试本路径下的其他文件，比如file6.php    

## 3. 
<img width="2559" height="1599" alt="image" src="https://github.com/user-attachments/assets/893e579d-7fdd-4e22-8e56-0eb9dcd1d52f" />
可以看到是没问题的，同路径下能访问，那就证明我们是能够控制这个参数的  
然后再尝试能不能进行路径穿越去包含其他的文件从而获得其他文件的内容     
这里我先贴一张容器里的实际目录截图，方便后面理解，也就不用路径穿越一个一个试了，直接看着试，主要是为了证明能够路径穿越  
<img width="2184" height="1156" alt="image" src="https://github.com/user-attachments/assets/db5130a5-76f3-40d6-8691-ea59394fe8db" />

## 4.
<img width="2559" height="1599" alt="image" src="https://github.com/user-attachments/assets/c82baeb6-22f5-4cce-b9f7-a49d1b0f74cc" />
可以看到是可以实现路径穿越的，我就只试了这一个文件，对于其他文件应该也是可以的


## 个人的其他思考问题
这个漏洞，local的，主要是在服务器本身的范围内进行读取文件，也就是在它的文件系统里，可以去读取一些敏感文件，也可以去结合文件上传漏洞去实现更大的危害
不过因为我还没有学那么多，实操也有限目前，所以我从ai那问了很多其他的危害，先大概知道，日后再去慢慢的理解和实现

## Local文件包含（LFI）各危害详细讲解 + 攻击原理

### 1. 敏感信息泄露（最容易发生的危害）
原理：LFI允许攻击者读取服务器上任意文件的内容。

攻击者怎么做：

正常URL可能是：http://example.com/index.php?page=about.php
攻击者修改为：http://example.com/index.php?page=../../../../etc/passwd
能读到的典型文件：

/etc/passwd → 所有系统用户列表
../../config.php 或 ../../.env → 数据库账号密码
../../.git/config → Git仓库信息
/var/www/html/config/database.php → 源码中的敏感配置
危害程度：高（信息泄露）

### 2. 远程代码执行（RCE）—— 最危险的危害
这是LFI最严重的利用方式，能直接在服务器上执行命令。

#### 方式一：包含日志文件（最经典）
攻击步骤：

攻击者先访问网站，把恶意代码写入Web服务器的访问日志：
curl http://example.com/"<?php system($_GET['cmd']);?>"
日志文件里就保存了这段恶意代码。
再通过LFI包含这个日志文件：
http://example.com/index.php?page=../../../../var/log/apache2/access.log&cmd=id
服务器就会执行 id 命令，攻击者就拿到了服务器权限。

#### 方式二：包含Session文件
很多网站会把用户session存成文件（如 /tmp/sess_xxxx），攻击者可以：

先在session里注入PHP代码
再用LFI包含这个session文件，从而执行代码
方式三：结合文件上传
攻击者上传一张“图片”，实际内容是PHP一句话木马。
图片被保存到 uploads/shell.jpg
再通过LFI包含它：?page=../../uploads/shell.jpg
### 3. 使用PHP伪协议（高级利用）
PHP提供了几种特殊协议，让LFI威力大增：

php://filter：读取源码而不执行

?page=php://filter/read=convert.base64-encode/resource=config.php
可以直接看到PHP文件的base64编码内容，解码后就是源码。

php://input：POST提交代码然后包含执行

data://：直接执行base64编码的代码

### 4. 权限提升与服务器完全控制
当攻击者通过LFI：

读到数据库密码 → 登录数据库导出所有用户数据
读到SSH私钥 → 直接SSH登录服务器
执行命令 → 安装挖矿程序、植入后门、反弹shell
最终可能导致整台服务器被攻陷。

总结：危害等级排序
### Local File Inclusion (LFI) 危害等级表

| 危害类型               | 攻击难度 | 危害程度 | 可直接控制服务器 | 常见利用方法                     | 典型后果                     |
|------------------------|----------|----------|------------------|----------------------------------|------------------------------|
| 敏感信息泄露           | 低       | 高       | 否               | 读取 `/etc/passwd`、`.env`、配置文件 | 数据库凭证、源码、配置泄露   |
| 日志文件代码执行 (RCE) | 中       | **极高** | 是               | 包含 Apache/Nginx 访问日志       | 执行任意系统命令             |
| 结合文件上传 RCE       | 中       | **极高** | 是               | 上传恶意文件后包含执行           | 服务器被完全接管             |
| Session 文件 RCE       | 高       | **极高** | 是               | 包含 PHP Session 文件            | 代码执行 + 后门维持          |
| 使用伪协议读取源码     | 低       | 中       | 否               | `php://filter`、`data://` 等协议 | 应用源码泄露，便于进一步攻击 |

**说明**：  
- **极高** 表示可直接导致服务器失控  
- LFI 是非常危险的漏洞，建议所有动态文件包含都使用白名单严格限制
