# 《Web安全深度剖析》的读书笔记
## 1. 深入HTTP请求流程

HTTP请求方法
* GET
* HEAD
HEAD方法除了服务器不能在响应里返回消息主体外，其他都与GET方法相同。此方法经常被用来测试超文本链接的有效性，可访问性和最近的改变。
攻击者编写扫描工具时，就常用此方法，因为只测试超文本链接的有效性，而不用返回消息主题，所以速度一定是最快的。
 * POST
 * PUT
 PUT方法用于请求服务器把请求中的实体存储在请求资源下，如果请求资源已经在服务器中存在，那么将会用此请求中的数据替换原先的数据，作为
 指定资源的最新修改版。如果请求指定的资源不存在，将会创建这个资源，且数据位请求正文。
 * DELETE
 * TRACE
 * CONNECT
 * OPTIONS
 OPTIONS方法是
 用于请求获得由URI标识的资源在请求/响应的通信过程中可以使用的功能选项。
 通过这个方法，客户端可以在采用具体资源请求之前，决定对该资源采取何种必要措施，或者了解服务器的性能。
 
 HTTP消息
 * 请求头
    * Host
    * User-Agent
      User-Agent请求报头域允许客户端将它的操作系统，浏览器和其他属性告诉服务器。
    * Referer
    * Cookie
    * Range
    * x-forward-for
       x-forward-for即XXF头，它代表请求端的IP，可以有多个，中间以逗号隔开。
    * Accept
    * Accept-Charset
* 响应头
   * Server
     服务器所使用的Web服务器名称，如Server:Apache/1.3.6(Unix)，攻击者通过查看此头，可以探测Web服务器名称。所以**建议在服务器端进行修改此头的信息**。

   * Set-Cookie
   * Last-Modified
   * Location
   * Refresh

### 2.2 截取HTTP请求
很多网站为了减少服务器端的压力，在后台方面减少验证，而只在Web前端使用JavaScript进行验证，殊不知这样大大增强了安全隐患。
作为一名Web开发人员，一定要牢记，前端JavaScript验证是为了防止用户输入错误，服务器端验证是为了防止恶意攻击。
* Burp Suite Proxy初体验
* Fiddler

## 3. 信息探测
 
 * Google Hack
    * site:  指定域名
    * intext: 正文中存在关键字的网页
    * intitle: 标题中存在关键字的网页
    * info: 一些基本信息
    * inurl: URL存在关键字的网页
    * filetype: 搜索指定文件类型
  
* Nmap
   Nmap是一个开源的网络连接端扫描软件，用来扫描计算机开放的网络连接端，确定哪些服务运行在哪些连接端，并且推断计算机运行哪个操作系统。
另外，它也用于评估网络系统安全。NMap增加了许多实用的插件，可以用来检测SQL注射，网页爬行，数据库密码检测等，号称”扫描之王“。

* DirBuster
 在渗透测试中，探测Web目录结构和隐藏的敏感文件是必不可少的一部分。通过探测可以了解网站的结构，获取管理员的一些敏感信息，
 比如网站的后台管理界面，文件上传界面，有时甚至可能扫描出网站的源代码。
 国内也有不少优秀的扫描器，如WWWScan,御剑后台扫描珍藏版等。
 
 * 指纹识别
 AppPrint是一款Web容器指纹识别工具，可针对单个IP，域名或IP段进行Tomcat，Web logic，WebSphere，IIS等Web容器识别。
 
## 4.漏洞扫描
 
 漏洞扫描器可以快速帮助我们发现漏洞，例如，SQL注入漏洞（SQL injection）,跨站点脚本攻击（cross-site scripting)，缓冲区溢出（buffer overflow)。
 
 * Burp Suite
 * AWVS
  AWVS(Acunetix Web Vulnerability Scanner)是一个自动化的Web应用程序安全测试工具，他可以扫描任何可通过Web浏览器访问和遵循HTTP/HTTPS规则的Web站点和Web应用程序。
 WVS可以快速扫描XSS，SQL注入攻击，代码执行，目录遍历攻击，文件入侵，脚本源代码泄露，CRLF注入，PHP代码注入，XPath注入，LDAP注入，Cookie操纵，URL重定向，应用程序错误消息等。
 
 * AppScan
  AppScan是IBM公司出品的一个领先的WEb应用安全测试工具，曾以Watchfire AppScan的名称享誉业界。AppScan可自动化Web应用的安全漏洞评估工作，
  能扫描和检测所有常见的Web应用安全漏洞，例如，SQL注入（SQL injection），跨站点脚本攻击（cross-site scripting）,缓冲区溢出(buffer overflow)及最新的Flash/Flex应用和
  Web 2.0应用暴露等方面的安全漏洞扫描。
  
  
## 5. SQL注入
  SQL注入攻击的问题最终归于用户可以控制输入，SQL注入，XSS，文件包含，命令执行都可归于此。这验证了一句老话：**有输入的地方，就可能存在风险。**
  目前流行的注入工具有： SQLMap,Pangolin, BSQL Hacker, Havij 和 The Mole，这些注入工具的功能大同小异。
  根据SQL注入的分类，防御主要分给两种：数据类型判断和特殊字符转义，使用预编译语句，框架技术。
  
## 6. 上传漏洞
中国菜刀和一句话图片木马
  很多程序员仅仅使用JavaScript来拒绝非法文件上传。这样验证对一些普通用户防止上传错误还可以，对专业的技术人员来说，这是非常低级的验证。
一般放在服务器端做验证。服务器端验证分为很多种：白名单与黑名单扩展名过滤，文件类型检测，文件重命名等操作。这样看起来似乎无懈可击，但不要忘记一点，那就是解析漏洞。
如果Web开发人员不考虑解析问题，上传漏洞配合解析漏洞，可以绕过大多数上传验证。
1. 攻击方法
    * 使用FireBug
        FireBug可以删除JavaScript事件。
    * 中间人攻击
        用Burp Suite按照正常流畅通过JavaScript验证，然后在传输中的HTTP层做手脚。
2. 对应方法
    * 白名单与黑名单验证。
    * MIME验证
    * 目录验证
    * 截断上传攻击
    
文本编辑器上传漏洞
常见的文本编辑器有CKEditor,Ewebeditor, UEditor, KindEditor, XHeditor等。
 
## 7. XSS跨站脚本漏洞
 XSS是指攻击者在网页中嵌入客户端脚本，通常是JavaScript编写的恶意代码，当用户使用浏览器浏览被嵌入恶意代码的网页时，恶意代码将会在用户的浏览器上执行。
 XSS主要分为3中类型
 * 反射型XSS
 * 存储型XSS
 * DOM XSS
 
检测XSS
* 手工检测
* 自动检测

随着XSS漏洞的兴起，一些XSS漏洞利用框架也随之出现，其中比较优秀的又BeFF,XSS Proxy, Backframe，像国内的XSSER.ME(XSS Platform)也是比较优秀的XSS漏洞利用框架。
 