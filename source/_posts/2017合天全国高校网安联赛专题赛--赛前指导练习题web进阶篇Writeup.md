---
title: 2017合天全国高校网安联赛专题赛--赛前指导练习题web进阶篇Writeup
tags: [write-up]
date: 2017-08-14 18:52
---
趁着放假，再刷一波题目
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>趁着放假，再刷一波题目</p>
<h2 id="0x01-捉迷藏">0x01 捉迷藏</h2>
<p><a href="http://218.76.35.75:20111/">链接</a> <br>
这个有个假flag，太坑了，查看源码有个链接点进去就是flag</br></p>
<h2 id="0x02-简单问答">0x02 简单问答</h2>
<p><a href="http://218.76.35.75:20112">链接</a> <br>
答案是 2016 lol 22 <br>
记得把q4改成q3，js会有函数干扰 ，用burpsuit就ok</br></br></p>
<h2 id="0x03-后台后台后台">0x03     后台后台后台</h2>
<p><a href="http://218.76.35.75:20113">链接</a> <br>
<code>PHPSESSID=qoercfqn062v19035374ten1d2; User=JohnTan101; Member=Tm9ybWFs</code> <br>
提示用admin登录 <br>
发现 member是base64编码的将Admin用base64编码提交即可</br></br></br></p>
<h2 id="0x04-php是最好的语言">0x04 php是最好的语言</h2>
<p><a href="http://218.76.35.75:20114">链接</a></p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
show_source(<span class="hljs-keyword">__FILE__</span>);
<span class="hljs-variable">$v1</span>=<span class="hljs-number">0</span>;<span class="hljs-variable">$v2</span>=<span class="hljs-number">0</span>;<span class="hljs-variable">$v3</span>=<span class="hljs-number">0</span>;
<span class="hljs-variable">$a</span>=(<span class="hljs-keyword">array</span>)json_decode(@<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'foo'</span>]);
<span class="hljs-keyword">if</span>(is_array(<span class="hljs-variable">$a</span>)){
    is_numeric(@<span class="hljs-variable">$a</span>[<span class="hljs-string">"bar1"</span>])?<span class="hljs-keyword">die</span>(<span class="hljs-string">"nope"</span>):<span class="hljs-keyword">NULL</span>;
    <span class="hljs-keyword">if</span>(@<span class="hljs-variable">$a</span>[<span class="hljs-string">"bar1"</span>]){
        (<span class="hljs-variable">$a</span>[<span class="hljs-string">"bar1"</span>]&gt;<span class="hljs-number">2016</span>)?<span class="hljs-variable">$v1</span>=<span class="hljs-number">1</span>:<span class="hljs-keyword">NULL</span>;
    }
    <span class="hljs-keyword">if</span>(is_array(@<span class="hljs-variable">$a</span>[<span class="hljs-string">"bar2"</span>])){
        <span class="hljs-keyword">if</span>(count(<span class="hljs-variable">$a</span>[<span class="hljs-string">"bar2"</span>])!==<span class="hljs-number">5</span> <span class="hljs-keyword">OR</span> !is_array(<span class="hljs-variable">$a</span>[<span class="hljs-string">"bar2"</span>][<span class="hljs-number">0</span>])) <span class="hljs-keyword">die</span>(<span class="hljs-string">"nope"</span>);
        <span class="hljs-variable">$pos</span> = array_search(<span class="hljs-string">"nudt"</span>, <span class="hljs-variable">$a</span>[<span class="hljs-string">"a2"</span>]);
        <span class="hljs-variable">$pos</span>===<span class="hljs-keyword">false</span>?<span class="hljs-keyword">die</span>(<span class="hljs-string">"nope"</span>):<span class="hljs-keyword">NULL</span>;
        <span class="hljs-keyword">foreach</span>(<span class="hljs-variable">$a</span>[<span class="hljs-string">"bar2"</span>] <span class="hljs-keyword">as</span> <span class="hljs-variable">$key</span>=&gt;<span class="hljs-variable">$val</span>){
            <span class="hljs-variable">$val</span>===<span class="hljs-string">"nudt"</span>?<span class="hljs-keyword">die</span>(<span class="hljs-string">"nope"</span>):<span class="hljs-keyword">NULL</span>;
        }
        <span class="hljs-variable">$v2</span>=<span class="hljs-number">1</span>;
    }
}
<span class="hljs-variable">$c</span>=@<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'cat'</span>];
<span class="hljs-variable">$d</span>=@<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'dog'</span>];
<span class="hljs-keyword">if</span>(@<span class="hljs-variable">$c</span>[<span class="hljs-number">1</span>]){
    <span class="hljs-keyword">if</span>(!strcmp(<span class="hljs-variable">$c</span>[<span class="hljs-number">1</span>],<span class="hljs-variable">$d</span>) &amp;&amp; <span class="hljs-variable">$c</span>[<span class="hljs-number">1</span>]!==<span class="hljs-variable">$d</span>){
        eregi(<span class="hljs-string">"3|1|c"</span>,<span class="hljs-variable">$d</span>.<span class="hljs-variable">$c</span>[<span class="hljs-number">0</span>])?<span class="hljs-keyword">die</span>(<span class="hljs-string">"nope"</span>):<span class="hljs-keyword">NULL</span>;
        strpos((<span class="hljs-variable">$c</span>[<span class="hljs-number">0</span>].<span class="hljs-variable">$d</span>), <span class="hljs-string">"htctf2016"</span>)?<span class="hljs-variable">$v3</span>=<span class="hljs-number">1</span>:<span class="hljs-keyword">NULL</span>;
    }
}
<span class="hljs-keyword">if</span>(<span class="hljs-variable">$v1</span> &amp;&amp; <span class="hljs-variable">$v2</span> &amp;&amp; <span class="hljs-variable">$v3</span>){
    <span class="hljs-keyword">include</span> <span class="hljs-string">"flag.php"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-variable">$flag</span>;
}
<span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p>这个好像是去年的题目，做了好几次，说点不一样的 <br>
1. 记住一点字符串与数字比较时会先转换成数字再比较 <br>
2. 还有一点<code>$pos = array_search("nudt", $a["a2"]);</code>这个在绕过的时候也是利用上面的那一点，所以有0就可以了 <br>
3. eregi的%00截断</br></br></br></p>
<h2 id="0x5-login">0x5 login</h2>
<p><a href="http://218.76.35.75:20115">链接</a> <br>
打开一看典型的文件包含，利用PHP协议读取所有源码 <br>
利用方法<code>http://218.76.35.75:20115/?page=php://filter/read=convert.base64-encode/resource=main</code></br></br></p>
<p>index.php</p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-variable">$pwhash</span>=<span class="hljs-string">"ffd313052dab00927cb61064a392f30ee454e70f"</span>;

<span class="hljs-keyword">if</span> (@<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'log'</span>]) {
    <span class="hljs-keyword">if</span>(file_exists(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'log'</span>].<span class="hljs-string">".log"</span>)){
        <span class="hljs-keyword">include</span>(<span class="hljs-string">"flag.txt"</span>);
}
}
<span class="hljs-keyword">if</span>(@<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'page'</span>] != <span class="hljs-string">'index'</span>){
    <span class="hljs-keyword">include</span>((@<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'page'</span>]?<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'page'</span>].<span class="hljs-string">".php"</span>:<span class="hljs-string">"main.php"</span>));
}

<span class="hljs-preprocessor">?&gt;</span></span>
</code></pre>
<p>login.php</p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-variable">$login</span>=@<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'login'</span>];
<span class="hljs-variable">$password</span>=@<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'password'</span>];
<span class="hljs-keyword">if</span>(@<span class="hljs-variable">$login</span>==<span class="hljs-string">"admin"</span> &amp;&amp; sha1(@<span class="hljs-variable">$password</span>)==<span class="hljs-variable">$pwhash</span>){
    <span class="hljs-keyword">include</span>(<span class="hljs-string">'flag.txt'</span>);
}<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (@<span class="hljs-variable">$login</span>&amp;&amp;@<span class="hljs-variable">$password</span>&amp;&amp;@<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'debug'</span>]) {
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"Login error, login credentials has been saved to ./log/"</span>.htmlentities(<span class="hljs-variable">$login</span>).<span class="hljs-string">".log"</span>;
    <span class="hljs-variable">$logfile</span> = <span class="hljs-string">"./log/"</span>.<span class="hljs-variable">$login</span>.<span class="hljs-string">".log"</span>;
    file_put_contents(<span class="hljs-variable">$logfile</span>, <span class="hljs-variable">$login</span>.<span class="hljs-string">"\n"</span>.<span class="hljs-variable">$password</span>);
} 
<span class="hljs-preprocessor">?&gt;</span></span>
    <span class="hljs-tag">&lt;<span class="hljs-title">center</span>&gt;</span>
        login<span class="hljs-tag">&lt;<span class="hljs-title">br</span>/&gt;</span><span class="hljs-tag">&lt;<span class="hljs-title">br</span>/&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-title">form</span> <span class="hljs-attribute">action</span>=<span class="hljs-value">""</span> <span class="hljs-attribute">method</span>=<span class="hljs-value">"POST"</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"login"</span> <span class="hljs-attribute">placeholder</span>=<span class="hljs-value">"login"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-title">br</span>/&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"password"</span> <span class="hljs-attribute">placeholder</span>=<span class="hljs-value">"password"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-title">br</span>/&gt;</span><span class="hljs-tag">&lt;<span class="hljs-title">br</span>/&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">type</span>=<span class="hljs-value">"submit"</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"Go!"</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-title">form</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-title">center</span>&gt;</span>

</code></pre>
<p>代码逻辑很明确,首先在login.php生成./log/xxx.log <br>
接着访问index.php即可</br></p>
<h2 id="0x06-http-头注入">0x06 http 头注入</h2>
<p><a href="http://218.76.35.75:20121">链接</a> <br>
经过一番测试发现在referer处有报错</br></p>
<p>具体的利用方法是insert注入，可以参考我<a href="http://blog.csdn.net/qq_31481187/article/details/59727015#t23">以前写的文章</a></p>
<p>具体的payload:<code>1123' or extractvalue(1,concat(0x5c,(select flag from flag))) or '','123')#</code></p>
<h2 id="0x07-简单的文件上传">0x07 简单的文件上传</h2>
<p><a href="http://218.76.35.75:20122">链接</a> <br>
不懂是什么意思 上传个php文件就可以</br></p>
<h2 id="0x08-简单的js">0x08 简单的JS</h2>
<p><a href="http://218.76.35.75:20123">链接</a></p>
<p>这个也是摸不到头脑，赋值粘贴执行初来链接 <br>
<code>http://218.76.35.75:20123/fl0a.php</code> <br>
查看cookie 得到flag</br></br></p>
<h2 id="0x09-php-是门松散的语言">0x09 php 是门松散的语言</h2>
<p><a href="http://218.76.35.75:20124">链接</a> <br>
可以把它归结为简单的变量覆盖 <br>
parse_str导致的漏洞 <br>
最后的payload <br>
<code>http://218.76.35.75:20124/?heetian=he%3Dabcd</code></br></br></br></br></p>
<h2 id="0x0a-试试xss">0x0a 试试xss</h2>
<p><a href="http://218.76.35.75:20125/">链接</a> <br>
有回显的 <br>
<code>' onerror="javasript:alert(document.domain)"</code></br></br></p>
<h2 id="0x0b-简单的文件包含">0x0b 简单的文件包含</h2>
<p><a href="http://218.76.35.75:20126">链接</a> <br>
直接包含文件,看源码得到flag</br></p>
<h2 id="0x0c-简单的验证">0x0c 简单的验证</h2>
<p><a href="218.76.35.75:20127">链接</a> <br>
看cookie有个guess字段 <br>
直接爆破admin对应的guess值</br></br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170814163135715?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="0x0d-vote">0x0d vote</h2>
<p><a href="http://218.76.35.75:65080/">链接</a> <br>
这个题目，和我出的一道题目神似<a href="https://github.com/actionyz/Web/tree/master/%E4%BA%8C%E6%AC%A1%E6%B3%A8%E5%85%A5">下载链接</a> <br>
找到备份源码之后用虚拟机vim还原</br></br></p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">include</span> <span class="hljs-string">'db.php'</span>;
session_start();
<span class="hljs-keyword">if</span> (!<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'login'</span>])) {
    <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'login'</span>] = <span class="hljs-string">'guest'</span>.mt_rand(<span class="hljs-number">1e5</span>, <span class="hljs-number">1e6</span>);
}
<span class="hljs-variable">$login</span> = <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'login'</span>];

<span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'submit'</span>])) {
    <span class="hljs-keyword">if</span> (!<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'id'</span>], <span class="hljs-variable">$_POST</span>[<span class="hljs-string">'vote'</span>]) || !is_numeric(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'id'</span>]))
        <span class="hljs-keyword">die</span>(<span class="hljs-string">'please select ...'</span>);
    <span class="hljs-variable">$id</span> = <span class="hljs-variable">$_POST</span>[<span class="hljs-string">'id'</span>];
    <span class="hljs-variable">$vote</span> = (int)<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'vote'</span>];
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$vote</span> &gt; <span class="hljs-number">5</span> || <span class="hljs-variable">$vote</span> &lt; <span class="hljs-number">1</span>)
        <span class="hljs-variable">$vote</span> = <span class="hljs-number">1</span>;
    <span class="hljs-variable">$q</span> = mysql_query(<span class="hljs-string">"INSERT INTO t_vote VALUES ({$id}, {$vote}, '{$login}')"</span>);
    <span class="hljs-variable">$q</span> = mysql_query(<span class="hljs-string">"SELECT id FROM t_vote WHERE user = '{$login}' GROUP BY id"</span>);
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;p&gt;&lt;b&gt;Thank you!&lt;/b&gt; Results:&lt;/p&gt;'</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;table border="1"&gt;'</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;tr&gt;&lt;th&gt;Logo&lt;/th&gt;&lt;th&gt;Total votes&lt;/th&gt;&lt;th&gt;Average&lt;/th&gt;&lt;/tr&gt;'</span>;
    <span class="hljs-keyword">while</span> (<span class="hljs-variable">$r</span> = mysql_fetch_array(<span class="hljs-variable">$q</span>)) {
        <span class="hljs-variable">$arr</span> = mysql_fetch_array(mysql_query(<span class="hljs-string">"SELECT title FROM t_picture WHERE id = "</span>.<span class="hljs-variable">$r</span>[<span class="hljs-string">'id'</span>]));
        <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;tr&gt;&lt;td&gt;'</span>.<span class="hljs-variable">$arr</span>[<span class="hljs-number">0</span>].<span class="hljs-string">'&lt;/td&gt;'</span>;
        <span class="hljs-variable">$arr</span> = mysql_fetch_array(mysql_query(<span class="hljs-string">"SELECT COUNT(value), AVG(value) FROM t_vote WHERE id = "</span>.<span class="hljs-variable">$r</span>[<span class="hljs-string">'id'</span>]));
        <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;td&gt;'</span>.<span class="hljs-variable">$arr</span>[<span class="hljs-number">0</span>].<span class="hljs-string">'&lt;/td&gt;&lt;td&gt;'</span>.round(<span class="hljs-variable">$arr</span>[<span class="hljs-number">1</span>],<span class="hljs-number">2</span>).<span class="hljs-string">'&lt;/td&gt;&lt;/tr&gt;'</span>;
    }
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;/table&gt;'</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;br&gt;&lt;a href="index.php"&gt;goBack&lt;/a&gt;&lt;br&gt;'</span>;
    <span class="hljs-keyword">exit</span>;
}
<span class="hljs-preprocessor">?&gt;</span></span>
<span class="hljs-tag">&lt;<span class="hljs-title">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">head</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">title</span>&gt;</span>Movie vote<span class="hljs-tag">&lt;/<span class="hljs-title">title</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">body</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">p</span>&gt;</span>Welcome, Movie vote<span class="hljs-tag">&lt;/<span class="hljs-title">p</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">form</span> <span class="hljs-attribute">action</span>=<span class="hljs-value">"index.php"</span> <span class="hljs-attribute">method</span>=<span class="hljs-value">"POST"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">table</span> <span class="hljs-attribute">border</span>=<span class="hljs-value">"1"</span> <span class="hljs-attribute">cellspacing</span>=<span class="hljs-value">"5"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">tr</span>&gt;</span>
<span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-variable">$q</span> = mysql_query(<span class="hljs-string">'SELECT * FROM t_picture'</span>);
<span class="hljs-keyword">while</span> (<span class="hljs-variable">$r</span> = mysql_fetch_array(<span class="hljs-variable">$q</span>)) {
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'&lt;td&gt;&lt;img src="./images/'</span>.<span class="hljs-variable">$r</span>[<span class="hljs-string">'image'</span>].<span class="hljs-string">'"&gt;&lt;div align="center"&gt;'</span>.<span class="hljs-variable">$r</span>[<span class="hljs-string">'title'</span>].<span class="hljs-string">'&lt;br&gt;&lt;input type="radio" name="id" value="'</span>.<span class="hljs-variable">$r</span>[<span class="hljs-string">'id'</span>].<span class="hljs-string">'"&gt;&lt;/div&gt;&lt;/td&gt;'</span>;
}
<span class="hljs-preprocessor">?&gt;</span></span>
<span class="hljs-tag">&lt;/<span class="hljs-title">tr</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">table</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">p</span>&gt;</span>Your vote:
<span class="hljs-tag">&lt;<span class="hljs-title">select</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"vote"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">option</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"1"</span>&gt;</span>1<span class="hljs-tag">&lt;/<span class="hljs-title">option</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">option</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"2"</span>&gt;</span>2<span class="hljs-tag">&lt;/<span class="hljs-title">option</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">option</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"3"</span>&gt;</span>3<span class="hljs-tag">&lt;/<span class="hljs-title">option</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">option</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"4"</span>&gt;</span>4<span class="hljs-tag">&lt;/<span class="hljs-title">option</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">option</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"5"</span>&gt;</span>5<span class="hljs-tag">&lt;/<span class="hljs-title">option</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">select</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-title">p</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">type</span>=<span class="hljs-value">"submit"</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"submit"</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"Submit"</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">form</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">html</span>&gt;</span>
</code></pre>
<p>经过观察可控字段只能是id <br>
所以id是注入点 <br>
经过二次注入之后，找到flag <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170814173522394?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></p>
<p>写的不太详细，不懂得可以私我</p>
<h2 id="0x0e-gg">0x0e GG</h2>
<p><a href="http://218.76.35.75:65380/">链接</a> <br>
利用网页js美化 <br>
找到关键点，结束时的处理函数</br></br></p>
<pre class="prettyprint"><code class=" hljs actionscript"> <span class="hljs-keyword">this</span>.mayAdd = <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(a)</span> {</span>
            <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.scores.length &lt; <span class="hljs-keyword">this</span>.maxscores) <span class="hljs-keyword">return</span> <span class="hljs-number">1E6</span> &lt; a &amp;&amp; (a = <span class="hljs-keyword">new</span> p, a.<span class="hljs-keyword">set</span>(<span class="hljs-string">"urlkey"</span>, <span class="hljs-string">"webqwer"</span> [<span class="hljs-number">1</span>] + <span class="hljs-string">"100.js"</span>, <span class="hljs-number">864E5</span>)), !<span class="hljs-number">0</span>;
            <span class="hljs-keyword">for</span> (<span class="hljs-keyword">var</span> b = <span class="hljs-keyword">this</span>.scores.length - <span class="hljs-number">1</span>; <span class="hljs-number">0</span> &lt;= b; --b)
                <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.scores[b].score &lt; a) <span class="hljs-keyword">return</span> <span class="hljs-number">1E6</span> &lt; a &amp;&amp; (a = <span class="hljs-keyword">new</span> p, a.<span class="hljs-keyword">set</span>(<span class="hljs-string">"urlkey"</span>,
                    <span class="hljs-string">"webqwer"</span> [<span class="hljs-number">1</span>] + <span class="hljs-string">"100.js"</span>, <span class="hljs-number">864E5</span>)), !<span class="hljs-number">0</span>;
            <span class="hljs-keyword">return</span> !<span class="hljs-number">1</span>
        };</code></pre>
<p>访问e100.js <br>
得到另一种js编码方式，直接console执行代码及可</br></p>
<h2 id="0x0f-reappear">0x0f Reappear</h2>
<p><a href="http://218.76.35.75:65180/">链接</a> <br>
直接查找相关漏洞<a href="http://www.jb51.net/hack/198171.html">漏洞内容</a></br></p>
<p>路径泄露 <br>
找到 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170814182548358?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
flag文件名已经找到了 <br>
直接搜索就OK<code>/var/www/html/Web/kind/kindeditor/attached/flag_clue.php</code></br></br></img></br></br></p>
<h2 id="0x10-drinkcoffee">0x10 DrinkCoffee</h2>
<p><a href="http://218.76.35.75:65280/">链接</a> <br>
直接改两个字段 <br>
<code>referer</code>和<code>X_FORWARDED_FOR</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170814233633905?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></p>
<h2 id="0x11-最安全的笔记管理系统">0x11 最安全的笔记管理系统</h2>
<p><a href="http://218.76.35.74:20128">链接</a> <br>
这道题目的思路挺不错的</br></p>
<h3 id="0x1-sql注入分析">0x1 SQL注入分析</h3>
<p>一开始拿到题目以为是SQL注入，首先分析一下waf检测 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170818181116273?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现是有转义的，所以又测试了编码绕过，无果。 <br>
下面开始寻找二次注入的地方，也没有发现。 <br>
那么注入是不可能的了</br></br></br></img></br></p>
<h3 id="0x2-代码审计">0x2 代码审计</h3>
<p>因为SQL注入无果所以开始转向其他思路。发现网页本身存在文件包含漏洞，利用PHP filter协议进行读取PHP代码 <br>
得到整个源代码后开始代码审计 登录之后会有session以及cookie的设置 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170818181819404?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
在目录扫描之后发现有admin目录，admin/index.php有用户身份检测 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170818182012136?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
同时<code>$userid!==false&amp;&amp;level!==false</code> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170818182153703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
应为初始值的原因所以<code>$userid!==false&amp;&amp;level!==false</code>是成立的 <br>
下面就看身份检测函数就可以了 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170818181910755?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
如果是想要绕过这里的检测 我们必须要知道SECURITY_KEY的值，也就是要知道rand的值。后来搜了一下找到了相关知识 <br>
<a href="https://www.leavesongs.com/PENETRATION/safeboxs-secret.html">这里写链接内容</a> <br>
于是在本地测试是可以登录admin的但是在本题死活登不上，下面的工作就是二次注入。因为没有登录admin所以也没有继续写。</br></br></br></img></br></br></br></img></br></br></img></br></br></img></br></br></p>
<h2 id="0x12-document">0x12 Document</h2>
<p><a href="http://218.76.35.74:20129">链接</a> <br>
试了半天，还好有同学提示，这题考察的是apache解析漏洞</br></p>
<h2 id="0x13-阳光总在风雨后">0x13 阳光总在风雨后</h2>
<p><a href="http://218.76.35.74:20130">链接</a> <br>
一开始会检测username所以在这里存在有盲注 <br>
绕过姿势<code>uname=admin'/1=(1=(exists(select(1)from(admin))))/'1'='1&amp;passwd=ad</code> <br>
或者<code>1'%(1)%'1</code> 或者 <code>1'^!1^'1</code> <br>
脚本就不贴了，如果不会可以参考我<a href="http://blog.csdn.net/qq_31481187/article/details/59727015#t24">以前的博客</a> <br>
得到密码<code>50f87a3a3ad48e26a5d9058418fb78b5</code> <br>
碰撞得到<code>shuangshuang</code> <br>
后面是一个命令执行的绕过 <br>
可以用${IFS}也可以是其他这里有些<a href="http://blog.csdn.net/qq_27446553/article/details/73927518">小trick</a> <br>
最后只显示最后一行 这里可以用<code>tail -n +3000 | head -n 1000</code>指定显示第几行 <br>
搜索到9ef89ad913e848b64b73e3aa721e44e4目录 <br>
接着找到flag文件<code>ls${IFS}/var/www/html/9ef89ad913e848b64b73e3aa721e44e4/|head${IFS}-n${IFS}1</code></br></br></br></br></br></br></br></br></br></br></br></p>
<h2 id="0x14-default">0x14 default</h2>
<p><a href="http://218.76.35.74:20131/">链接</a></p>
<p>首先是扫描到index2.php <br>
接着就好写了 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170818192641738?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></p></div>