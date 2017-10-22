---
title: PHPmailer 邮件发送原理及实现
tags: [PHP]
date: 2017-03-09 15:08
---
简单了解一下PHP发送邮件的过程，利用PHPmailer
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="一-邮件发送原理">一 邮件发送原理</h1>
<h2 id="1组成部分">（1）组成部分</h2>
<pre class="prettyprint"><code class="language-mermaid! hljs brainfuck">    <span class="hljs-comment">graph</span> <span class="hljs-comment">TD;</span>
    <span class="hljs-comment">邮件</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt;<span class="hljs-comment">邮件服务器;</span>
    <span class="hljs-comment">邮件服务器</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt;<span class="hljs-comment">供在网上存储邮件的空间;</span>
    <span class="hljs-comment">邮件</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt;<span class="hljs-comment">用户代理;</span>
    <span class="hljs-comment">用户代理</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt;<span class="hljs-comment">邮件服务器上读取或者发送邮件到邮件服务器上的一个软件</span>
    <span class="hljs-comment">邮件</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt;<span class="hljs-comment">邮件传送协议;</span>
    <span class="hljs-comment">邮件传送协议</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt;<span class="hljs-comment">邮件在传送过程中必须遵守的约定</span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170309150719595?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<pre><code>1. 发信人在用户代理上编辑邮件，并写清楚收件人的邮箱地址；
2. 用户代理根据发信人编辑的信息，生成一封符合邮件格式的邮件；
3. 用户代理把邮件发送到发信人的的邮件服务器上，邮件服务器上面有一个缓冲队列，发送到邮件服务器上面的邮件都会加入到缓冲队列中，等待邮件服务器上的SMTP客户端进行发送；
4. 发信人的邮件服务器使用SMTP协议把这封邮件发送到收件人的邮件服务器上（它会自动根据收件人的邮箱来分析出收件人的邮箱服务器）；
5. 收件人的邮件服务器收到邮件后，把这封邮件放到收件人在这个服务器上的信箱中；
6. 收件人使用用户代理来收取邮件。首先用户代理使用POP3协议来连接收件人所在的邮件服务器，身份验证成功后，用户代理就可以把邮件服务器上面的收件人邮箱里面的邮件读取出来，并展示给收件人。
</code></pre>
<h2 id="2协议简介">（2）协议简介</h2>
<p><strong>协议简介：SMTP</strong></p>
<blockquote>
<p>SMTP(Simple Mail Transfer Protocol)即简单邮件传输协议，是一种提供可靠且有效电子邮件传输的协议。SMTP是建立在FTP文件传输服务上的一种邮件服务，主要用于传输系统之间的邮件信息并提供与来信有关的通知。（来自百度百科）</p>
</blockquote>
<p><strong>协议简介：POP3</strong></p>
<blockquote>
<p>POP3(Post Office Protocol 3)即邮局协议的第3个版本，它是规定个人计算机如何连接到互联网上的邮件服务器进行收发邮件的协议。它是因特网电子邮件的第一个离线协议标准，POP3协议允许用户从服务器上把邮件存储到本地主机（即自己的计算机）上，同时根据客户端的操作删除或保存在邮件服务器上的邮件，而POP3服务器则是遵循POP3协议的接收邮件服务器，用来接收电子邮件的。（来自百度百科） <br>
  POP 协议支持“离线”邮件处理。其具体过程是：邮件发送到服务器上，电子邮件客户端调用邮件客户机程序以连接服务器，并下载所有未阅读的电子邮件。这种离线访问模式是一种存储转发服务，将邮件从邮件服务器端送到个人终端机器上，一般是 PC机或 MAC。一旦邮件发送到 PC 机或 MAC上，邮件服务器上的邮件将会被删除。但目前的POP3邮件服务器大都可以“只下载邮件，服务器端并不删除”，也就是改进的POP3协议。（来自百度百科）</br></p>
</blockquote>
<h2 id="3常用的邮件服务器地址">（3）常用的邮件服务器地址</h2>
<p><strong>126邮箱</strong></p>
<blockquote>
<p>POP3服务器:pop.126.com  <br>
  SMTP服务器:smtp.126.com</br></p>
</blockquote>
<p><strong>163邮箱</strong></p>
<blockquote>
<p>POP3服务器:pop.163.com  <br>
  SMTP服务器:smtp.163.com</br></p>
</blockquote>
<p><strong>yahoo邮箱</strong></p>
<p>注意：yahoo在foxmail 4.1以上的版本设置如下：</p>
<blockquote>
<p>POP3服务器：pop.mail.yahoo.com.cn  <br>
  SMTP服务器：smtp.mail.yahoo.com.cn</br></p>
</blockquote>
<p><strong>Sohu邮箱</strong></p>
<blockquote>
<p>POP3服务器：pop3.sohu.com  <br>
  SMTP服务器：smtp.sohu.com</br></p>
</blockquote>
<p><strong>QQ邮箱</strong></p>
<blockquote>
<p>POP3服务器：pop.qq.com  <br>
  SMTP服务器：smtp.qq.com  <br>
  SMTP服务器需要身份验证</br></br></p>
</blockquote>
<p>从上面大家可以看出，一般的POP3邮件服务器地址为pop然后加上自己的域名，SMTP邮件服务器地址为smtp加上自己的域名。常用的邮件服务器地址都可以在网上找到。各大型邮箱smtp服务器及端口收集 。</p>
<h1 id="二-邮件发送代码phpmailer">二 邮件发送代码（phpmailer）</h1>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span> 
<span class="hljs-comment">// 必要导入</span>
<span class="hljs-keyword">require</span>(<span class="hljs-string">"phpmailer/class.phpmailer.php"</span>);
<span class="hljs-keyword">require</span>(<span class="hljs-string">"phpmailer/class.smtp.php"</span>);
date_default_timezone_set(<span class="hljs-string">'Asia/Shanghai'</span>);<span class="hljs-comment">//设定时区东八区</span>
<span class="hljs-variable">$mail</span> = <span class="hljs-keyword">new</span> PHPMailer(); <span class="hljs-comment">//建立邮件发送类</span>
<span class="hljs-variable">$address</span> = <span class="hljs-string">"xxxx@qq.com"</span>;<span class="hljs-comment">//306800278收件人地址（必须真实）</span>
<span class="hljs-variable">$mail</span>-&gt;IsSMTP(); <span class="hljs-comment">// 使用SMTP方式发送</span>
<span class="hljs-variable">$mail</span>-&gt;CharSet =<span class="hljs-string">"UTF-8"</span>;<span class="hljs-comment">//设置编码，否则发送中文乱码</span>
<span class="hljs-variable">$mail</span>-&gt;Host = <span class="hljs-string">"smtp.qq.com"</span>; <span class="hljs-comment">// 您的企业邮局域名                           </span>
<span class="hljs-variable">$mail</span>-&gt;SMTPAuth = <span class="hljs-keyword">true</span>; <span class="hljs-comment">// 启用SMTP验证功能</span>
<span class="hljs-variable">$mail</span>-&gt;Username = <span class="hljs-string">"yyyy@qq.com"</span>; <span class="hljs-comment">// 发件人邮箱（必须真实）</span>
<span class="hljs-variable">$mail</span>-&gt;Password = <span class="hljs-string">"*****"</span>; <span class="hljs-comment">// 发件人密码（必须真实）</span>
<span class="hljs-variable">$mail</span>-&gt;From = <span class="hljs-string">"yyyyy@qq.com"</span>; <span class="hljs-comment">//邮件发送者email地址（必须真实）</span>
<span class="hljs-variable">$mail</span>-&gt;FromName = <span class="hljs-string">"yz"</span>;<span class="hljs-comment">// 发件人姓名</span>
<span class="hljs-variable">$mail</span>-&gt;AddAddress(<span class="hljs-variable">$address</span>, <span class="hljs-string">"222@qq.com"</span>);<span class="hljs-comment">//收件人收件人地址，可以替换成任何想要接收邮件的email信箱,格式是AddAddress("收件人email","收件人姓名")</span>
<span class="hljs-comment">//$mail-&gt;AddReplyTo("", "");</span>
<span class="hljs-comment">//$mail-&gt;AddAttachment("/var/tmp/file.tar.gz"); // 添加附件</span>
<span class="hljs-comment">//$mail-&gt;IsHTML(true); // set email format to HTML //是否使用HTML格式</span>
<span class="hljs-variable">$mail</span>-&gt;Subject = <span class="hljs-string">"test"</span>; <span class="hljs-comment">//邮件标题</span>
<span class="hljs-variable">$mail</span>-&gt;Body = <span class="hljs-string">"hello"</span>; <span class="hljs-comment">//邮件内容</span>
<span class="hljs-variable">$mail</span>-&gt;AltBody = <span class="hljs-string">"This is the body in plain text for non-HTML mail clients"</span>; <span class="hljs-comment">//附加信息，可以省略</span>

<span class="hljs-keyword">if</span>(!<span class="hljs-variable">$mail</span>-&gt;Send()) {
<span class="hljs-keyword">echo</span> <span class="hljs-string">'Mailer Error: '</span> . <span class="hljs-variable">$mail</span>-&gt;ErrorInfo;
} <span class="hljs-keyword">else</span> {
<span class="hljs-keyword">echo</span> <span class="hljs-string">"Message sent!恭喜，邮件发送成功！"</span>;
}



<span class="hljs-preprocessor">?&gt;</span></code></pre></div>