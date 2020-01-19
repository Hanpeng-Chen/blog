---
title: Python从小白到攻城狮(20)——使用SMTP和POP模块实现收发电子邮件
urlname: python-smtp-and-pop-email
tags:
  - python教程
  - 电子邮件
  - SMTP/POP3
categories:
  - Python
date: 2020-01-17 00:00:00
---

Email的历史比Web还要久远，直到现在，Email也是互联网上应用非常广泛的服务。

几乎所有的编程语言都支持发送和接收电子邮件，接下来我们一起用Python来实现电子邮件的收发。

# SMTP发送电子邮件
SMTP是发送邮件的协议，python内置对SMTP的支持，可以发送纯文本邮件、HTML邮件以及带附件的邮件。

Python对SMTP支持有`smtplib`和`email`两个模块，`email`负责构造邮件，`smtplib`负责发送邮件。

## 发送纯文本邮件
首先，我们来构造一个纯文本邮件：

邮件正文：构造`MIMEText`对象时，第一个参数就是邮件正文，第二个参数是MIME的subtype，传入`'plain'`表示纯文本，最终的MIME就是`'text/plain'`，最后一定要用`utf-8`编码保证多语言兼容性。

由于邮件主题、如何显示发件人、收件人等信息并不是通过SMTP协议发给MTA，而是包含在发给MTA的文本中的，所以，我们必须把`From`、`To`和`Subject`添加到`MIMEText`中。

我们编写了一个函数`format_addr()`来格式化一个邮件地址。注意不能简单地传入`name <addr@example.com>`，因为如果包含中文，需要通过`Header`对象进行编码。

`msg['To']`接收的是字符串而不是`list`，如果有多个邮件地址，用`,`分隔即可。

我们用`set_debuglevel(1)`就可以打印出和SMTP服务器交互的所有信息。SMTP协议就是简单的文本命令和响应。`login()`方法用来登录SMTP服务器，`sendmail()`方法就是发邮件，由于可以一次发给多个人，所以传入一个`list`，邮件正文是一个`str`，`as_string()`把`MIMEText`对象变成`str`。

然后，通过SMTP发出去，完整代码如下：

```python
from email.mime.text import MIMEText
from email import encoders
from email.header import Header
from email.utils import parseaddr, formataddr

import smtplib

# 格式化邮件地址，并用Header对象进行编码（防止包含中文）
def format_addr(s):
    name, addr = parseaddr(s)
    return formataddr((Header(name, 'utf-8').encode(), addr))

# 输入Email地址和口令
from_address = '*********@qq.com'
from_account_password = '****************' # 第三方登录邮箱的授权码，QQ邮箱为16位授权码
# 输入收件人地址
to_address = '******@163.com'

message = MIMEText('This email is sended by Python', 'plain', 'utf-8')
# 设置发件人信息
message['From'] = format_addr('Python开发 <%s>' % from_address)
# 设置收件人信息
message['To'] = format_addr('学员 <%s>' % to_address)
# 设置邮件主题
message['Subject'] = Header('学习使用Python发送邮件', 'utf-8').encode()

# 输入SMTP服务器地址
smtp_server = 'smtp.qq.com'
# SMTP协议默认端口为25
server = smtplib.SMTP(smtp_server, 25)
server.set_debuglevel(1)
server.login(from_address, from_account_password)
server.sendmail(from_address, [to_address], message.as_string())
server.quit()
```
发送成功的话，我们在收件人邮箱中收到刚刚发送的邮件，如下图所示：
{% qnimg python_series/20-1.png %}

> 你看到的收件人的名字很可能不是我们传入的`学员`，因为很多邮件服务商在显示邮件时，会把收件人名字自动替换为用户注册的名字，但是其他收件人名字的显示不受影响。

## 发送HTML邮件
如果我们要发送HTML邮件，而不是普通的纯文本邮件，只需在构造`MIMEText`对象时，把HTML字符串传进去，再把第二个参数由`plain`变为 `html`就可以了。

HTML字符串如下：
```python
html = '<html><body><h1>Hello</h1>' +
    '<p>更多文章请移步：<a href="http://www.chenhanpeng.com">代码视界</a></p>' +
    '</body></html>'

message = MIMEText(html, 'html', 'utf-8')
```

发送成功后，收到的HTML邮件如下所示：
{% qnimg python_series/20-2.png %}

## 发送附件
如果Email中要加上附件怎么办？带附件的邮件可以看做包含若干部分的邮件：文本和各个附件本身，所以，可以构造一个`MIMEMultipart`对象代表邮件本身，然后往里面加上一个`MIMEText`作为邮件正文，再继续往里面加上表示附件的`MIMEBase`对象即可：
```python
message = MIMEMultipart()
# 设置发件人信息
message['From'] = format_addr('Python开发 <%s>' % from_address)
# 设置收件人信息
message['To'] = format_addr('学员 <%s>' % to_address)
# 设置邮件主题
message['Subject'] = Header('发送带附件邮件', 'utf-8').encode()

message.attach(MIMEText('这是一封带附件的email', 'plain', 'utf-8'))

f = open('./20-1.png', 'rb')
# 设置附件的MIME和文件名
mime = MIMEBase('image', 'png', filename='20.png')
# 加上必要的头信息
mime.add_header('Content-Disposition', 'attachment', filename='20.png')
mime.add_header('Content-ID', '<0>')
mime.add_header('X-Attachment-Id', '0')
# 把附件的内容读进来:
mime.set_payload(f.read())
# 用Base64编码:
encoders.encode_base64(mime)
# 添加到MIMEMultipart:
message.attach(mime)
f.close()
```

发送成功后，收到的带附件的邮件如下所示：
{% qnimg python_series/20-3.png %}

## 在 HTML 文本中添加图片
如果要把图片嵌入到邮件正文中，我们该怎么做？直接在HTML中链接图片地址？这种处理方案是行不通的，因为大部分邮件服务商都会自动屏蔽带有外链的图片，因为不知道这些链接是否指向恶意网站。

要把图片嵌入到邮件正文中，我们只需按照发送附件的方式，先把邮件作为附件添加进去，然后，在HTML中通过引用`src="cid:image1`就可以把附件作为图片嵌入了。如果有多个图片，给它们依次编号，然后引用不同的`cid:x`即可。

把上面代码加入`MIMEMultipart`的`MIMEText`从`plain`改为`html`，然后在适当的位置引用图片：
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import smtplib
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.header import Header

# 输入Email地址和口令
from_address = '1036445344@qq.com'
from_account_password = '****************' # 第三方登录邮箱的授权码，QQ邮箱为16位授权码
# 输入收件人地址
to_address = 'chenhp1994@163.com'

msgRoot = MIMEMultipart('related')
msgRoot['From'] = Header("Python开发", 'utf-8')
msgRoot['To'] =  Header("测试", 'utf-8')
subject = 'Python SMTP 邮件测试'
msgRoot['Subject'] = Header(subject, 'utf-8')
 
msgAlternative = MIMEMultipart('alternative')
msgRoot.attach(msgAlternative)

mail_msg = """
<p>Python 邮件发送测试...</p>
<p><a href="http://www.chenhanpeng.com">代码视界</a></p>
<p>图片演示：</p>
<p><img src="cid:image1"></p>
"""
msgAlternative.attach(MIMEText(mail_msg, 'html', 'utf-8'))
 
# 指定图片为当前目录
fp = open('./20-1.png', 'rb')
msgImage = MIMEImage(fp.read())
fp.close()
 
# 定义图片 ID，在 HTML 文本中引用
msgImage.add_header('Content-ID', '<image1>')
msgRoot.attach(msgImage)

# 输入SMTP服务器地址
smtp_server = 'smtp.qq.com'
# SMTP协议默认端口为25
server = smtplib.SMTP(smtp_server, 25)
server.login(from_address, from_account_password)
server.sendmail(from_address, [to_address], msgRoot.as_string())
print('邮件发送成功')
server.quit()
```

再次发送，就可以看到图片直接嵌入到邮件正文的效果：
{% qnimg python_series/20-4.png %}

## 同时支持HTML和Plain格式
如果我们发送HTML邮件，收件人通过浏览器或者Outlook之类的软件是可以正常浏览邮件内容的，但是，如果收件人使用的设备太古老，查看不了HTML邮件怎么办？

办法是在发送HTML的同时再附加一个纯文本，如果收件人无法查看HTML格式的邮件，就可以自动降级查看纯文本邮件。

利用`MIMEMultipart`就可以组合一个HTML和Plain，要注意指定subtype是`alternative`：
```python
message = MIMEMultipart('alternative')
# 设置发件人信息
message['From'] = format_addr('Python开发 <%s>' % from_address)
# 设置收件人信息
message['To'] = format_addr('学员 <%s>' % to_address)
# 设置邮件主题
message['Subject'] = Header('同时支持HTML和Plain格式', 'utf-8').encode()

message.attach(MIMEText('hello Python', 'plain', 'utf-8'))
message.attach(MIMEText('<html><body><h1>hello Python</h1></body></html>', 'html', 'utf-8'))
# 正常发送msg对象...
```

## 加密SMTP
使用标准的25端口连接SMTP服务器时，使用的是明文传输，发送邮件的整个过程可能会被窃听。要更安全地发送邮件，可以加密SMTP会话，实际上就是先创建SSL安全连接，然后再使用SMTP协议发送邮件。

某些邮件服务商，例如Gmail，提供的SMTP服务必须要加密传输。我们来看看如何通过Gmail提供的安全SMTP发送邮件。

必须知道，Gmail的SMTP端口是587，因此，修改代码如下：
```python
smtp_server = 'smtp.gmail.com'
smtp_port = 587
server = smtplib.SMTP(smtp_server, smtp_port)
server.starttls()
# 剩下的代码和前面的一模一样:
server.set_debuglevel(1)
```
只需要在创建SMTP对象后，立刻调用starttls()方法，就创建了安全连接。后面的代码和前面的发送邮件代码完全一样。


# POP3收取邮件
上面我们讲了如何使用SMTP发送邮件，那么如何用Python实现邮件的收取呢？

收取邮件就是编写一个MUA作为客户端，从MDA把邮件获取到用户的电脑或者手机上。收取邮件最常用的协议是**POP**协议，目前版本号是3，俗称**POP3**。

Python内置了一个poplib模块，实现了POP3协议，可以直接用来收邮件。

注意到POP3协议收取的不是一个已经可以阅读的邮件本身，而是邮件的原始文本，这和SMTP协议很像，SMTP发送的也是经过编码后的一大段文本。

要把POP3收取的文本变成可以阅读的邮件，还需要用email模块提供的各种类来解析原始文本，变成可阅读的邮件对象。

所以，收取邮件分两步：

- 第一步：用poplib把邮件的原始文本下载到本地；

- 第二部：用email模块把原始邮件解析为Message对象，然后用适当的形式把邮件内容展示为用户。

## 通过POP3下载邮件
POP3协议本身很简单，我们来获取最新的一封邮件内容，示例代码如下：
```python
# 链接POP3服务器
server = poplib.POP3(pop3_server)

# 身份认证
server.user(email)
server.pass_(pw)

# stat()返回邮件数量和占用空间
print('Message: %s.  Size: %s' % server.stat())

# list()返回所有邮件的编号:
resp, mails, octets = server.list()
# 可以查看返回的列表类似[b'1 82923', b'2 2184', ...]
# print(mails)

# 获取最新一封邮件, 注意索引号从1开始:
index = len(mails)
resp, lines, octets = server.retr(index)

# lines存储了邮件的原始文本的每一行,
# 可以获得整个邮件的原始文本:
msg_content = b'\r\n'.join(lines).decode('utf-8')
```

用POP3获取邮件其实很简单，要获取所有邮件，只需要循环使用`retr()`把每一封邮件内容拿到即可。真正麻烦的是把邮件的原始内容解析为可以阅读的邮件对象。

## 解析邮件
我们只需要用一行代码就可以把刚才获取到的邮件内容解析为`Message`对象：
```python
msg = Parser().parsestr(msg_content)
```

但是这个`Message`对象本身可能是一个`MIMEMultipart`对象，即包含嵌套的其他MIMEBase对象，嵌套可能还不止一层。

所以我们要递归地打印出Message对象的层次结构：
```python
# 递归打印出Message对象的层次结构
def print_info(msg, indent=0):
    if indent == 0:
        for header in ['From', 'To', 'Subject']:
            value = msg.get(header, '')
            if value:
                if header == 'Subject':
                    value = decode_str(value)
                else:
                    hdr, addr = parseaddr(value)
                    name = decode_str(hdr)
                    value = u'%s <%s>' % (name, addr)
            print('%s%s: %s' % ('  ' * indent, header, value))
    if (msg.is_multipart()):
        parts = msg.get_payload()
        for n, part in enumerate(parts):
            print('%spart %s' % ('  ' * indent, n))
            print('%s--------------------' % ('  ' * indent))
            print_info(part, indent + 1)
    else:
        content_type = msg.get_content_type()
        if content_type == 'text/plain' or content_type == 'text/html':
            content = msg.get_payload(decode=True)
            charset = guess_charset(msg)
            if charset:
                content = content.decode(charset)
            print('%sText: %s' % ('  ' * indent, content + '...'))
        else:
            print('%sAttachment: %s' % ('  ' * indent, content_type))
```
邮件的Subject或者Email中包含的名字都是经过编码后的str，要正常显示，就必须decode：
```python
def decode_str(s):
    value, charset = decode_header(s)[0]
    if charset:
        value = value.decode(charset)
    return value
```
`decode_header()`返回一个`list`，因为像`Cc`、`Bcc`这样的字段可能包含多个邮件地址，所以解析出来的会有多个元素。上面的代码我们偷了个懒，只取了第一个元素。

文本邮件的内容也是str，还需要检测编码，否则，非UTF-8编码的邮件都无法正常显示：
```python
def guess_charset(msg):
    charset = msg.get_charset()
    if charset is None:
        content_type = msg.get('Content-Type', '').lower()
        pos = content_type.find('charset=')
        if pos >= 0:
            charset = content_type[pos + 8:].strip()
    return charset
```

运行程序，结果如下：
```
Message: 739.  Size: 144832488
From: chenhp <chenhp1994@163.com>
To:  <1036445344@qq.com>
Subject: 测试邮件
part 0
--------------------
  Text: Hello Python！
测试邮件...
part 1
--------------------
  Text: <div style="line-height:1.7;color:#000000;font-size:14px;font-family:Arial"><div>Hello Python！</div><div>测试邮件</div></div><br><br><span title="neteasefooter"><p>&nbsp;</p></span>...
```

**完整的示例代码：** [python-learning](https://github.com/HamptonChen/python-learning/tree/master/20-%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6)

未完待续，持续更新中......