---
title: PHP 函数漏洞总结
tags: [PHP]
date: 2017-03-09 20:59
---
总结了常见的PHP函数漏洞，希望对大家有用
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><hr>
<h2 id="1md5-compare漏洞">1.MD5 compare漏洞</h2>
<p>PHP在处理哈希字符串时，会利用”!=”或”==”来对哈希值进行比较，它把每一个以”0E”开头的哈希值都解释为0，所以如果两个不同的密码经过哈希以后，其哈希值都是以”0E”开头的，那么PHP将会认为他们相同，都是0。 <br>
<strong>常见的payload有</strong></br></p>
<pre class="prettyprint"><code class=" hljs scss">0x01 <span class="hljs-function">md5(str)</span>
    QNKCDZO
    240610708
    s878926199a
    s155964671a
    s214587387a
    s214587387a
     <span class="hljs-function">sha1(str)</span>
    <span class="hljs-function">sha1(<span class="hljs-string">'aaroZmOk'</span>)</span>  
    <span class="hljs-function">sha1(<span class="hljs-string">'aaK1STfY'</span>)</span>
    <span class="hljs-function">sha1(<span class="hljs-string">'aaO8zKZF'</span>)</span>
    <span class="hljs-function">sha1(<span class="hljs-string">'aa3OFF9m'</span>)</span>

0x02 <span class="hljs-function">md5(<span class="hljs-function">md5(str)</span>.<span class="hljs-string">"SALT"</span>)</span>
    2</code></pre>
<p>同时MD5不能处理数组，若有以下判断则可用数组绕过</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-keyword">if</span>(@md5(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'a'</span>]) == @md5(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'b'</span>]))
{
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"yes"</span>;
}
<span class="hljs-comment">//http://127.0.0.1/1.php?a[]=1&amp;b[]=2</span></code></pre>
<h2 id="2ereg函数漏洞00截断">2.ereg函数漏洞：00截断</h2>
<pre class="prettyprint"><code class="language-php hljs ">ereg (<span class="hljs-string">"^[a-zA-Z0-9]+$"</span>, <span class="hljs-variable">$_GET</span>[<span class="hljs-string">'password'</span>]) === <span class="hljs-keyword">FALSE</span></code></pre>
<p>字符串对比解析 <br>
在这里如果 $_GET[‘password’]为数组，则返回值为NULL <br>
如果为123 || asd || 12as || 123%00&amp;&amp;&amp;**，则返回值为true <br>
其余为false</br></br></br></p>
<h2 id="3变量本身的key">3.变量本身的key</h2>
<p>说到变量的提交很多人只是看到了GET/POST/COOKIE等提交的变量的值，但是忘记了有的程序把变量本身的key也当变量提取给函数处理。 </p>
<pre class="prettyprint"><code class="language-php hljs ">    <span class="hljs-preprocessor">&lt;?php</span>

    <span class="hljs-comment">//key.php?aaaa'aaa=1&amp;bb'b=2 </span>

    <span class="hljs-comment">//print_R($_GET); </span>

     <span class="hljs-keyword">foreach</span> (<span class="hljs-variable">$_GET</span> <span class="hljs-keyword">AS</span> <span class="hljs-variable">$key</span> =&gt; <span class="hljs-variable">$value</span>)

    {

            <span class="hljs-keyword">print</span> <span class="hljs-variable">$key</span>.<span class="hljs-string">"\n"</span>;

    }

    <span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="4变量覆盖">4.变量覆盖</h2>
<p>extract()这个函数在指定参数为EXTR_OVERWRITE或者没有指定函数可以导致变量覆盖</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>  
    <span class="hljs-variable">$auth</span> = <span class="hljs-string">'0'</span>;  
    <span class="hljs-comment">// 这里可以覆盖$auth的变量值</span>
    extract(<span class="hljs-variable">$_GET</span>); 
    <span class="hljs-keyword">if</span>(<span class="hljs-variable">$auth</span> == <span class="hljs-number">1</span>){  
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"private!"</span>;  
    } <span class="hljs-keyword">else</span>{  
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"public!"</span>;  
    }  
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>  
<span class="hljs-variable">$a</span>=<span class="hljs-string">'hi'</span>;
<span class="hljs-keyword">foreach</span>(<span class="hljs-variable">$_GET</span> <span class="hljs-keyword">as</span> <span class="hljs-variable">$key</span> =&gt; <span class="hljs-variable">$value</span>) {
        <span class="hljs-keyword">echo</span> <span class="hljs-variable">$key</span>;
        <span class="hljs-variable">$$key</span> = <span class="hljs-variable">$value</span>;
}
<span class="hljs-keyword">print</span> <span class="hljs-variable">$a</span>;
<span class="hljs-preprocessor">?&gt;</span>
</code></pre>
<h2 id="5strcmp">5.strcmp</h2>
<blockquote>
<p>如果 str1 小于 str2 返回 &lt; 0； 如果 str1 大于 str2 返回 &gt; 0；如果两者相等，返回 0。 <br>
      5.2 中是将两个参数先转换成string类型。 <br>
      5.3.3以后，当比较数组和字符串的时候，返回是0。 <br>
      5.5 中如果参数不是string类型，直接return了</br></br></br></p>
</blockquote>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
    <span class="hljs-variable">$password</span>=<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'password'</span>];
    <span class="hljs-keyword">if</span> (strcmp(<span class="hljs-string">'xd'</span>,<span class="hljs-variable">$password</span>)) {
     <span class="hljs-keyword">echo</span> <span class="hljs-string">'NO!'</span>;
    } <span class="hljs-keyword">else</span>{
        <span class="hljs-keyword">echo</span> <span class="hljs-string">'YES!'</span>;
    }
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="6sha1-和-md5-函数">6.sha1 和 md5 函数</h2>
<p>md5 和 sha1 无法处理数组，返回 NULL</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-keyword">if</span> (@sha1([]) ==  <span class="hljs-keyword">false</span>)
    <span class="hljs-keyword">echo</span> <span class="hljs-number">1</span>;
<span class="hljs-keyword">if</span> (@md5([]) ==  <span class="hljs-keyword">false</span>)
    <span class="hljs-keyword">echo</span> <span class="hljs-number">2</span>;
<span class="hljs-keyword">echo</span> var_dump(@sha1([]));</code></pre>
<h2 id="7isnumeric">7.is_numeric</h2>
<p>PHP提供了is_numeric函数，用来变量判断是否为数字。但是函数的范围比较广泛，不仅仅是十进制的数字。</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">echo</span> is_numeric(<span class="hljs-number">233333</span>);       <span class="hljs-comment"># 1</span>
<span class="hljs-keyword">echo</span> is_numeric(<span class="hljs-string">'233333'</span>);    <span class="hljs-comment"># 1</span>
<span class="hljs-keyword">echo</span> is_numeric(<span class="hljs-number">0x233333</span>);    <span class="hljs-comment"># 1</span>
<span class="hljs-keyword">echo</span> is_numeric(<span class="hljs-string">'0x233333'</span>);   <span class="hljs-comment"># 1</span>
<span class="hljs-keyword">echo</span> is_numeric(<span class="hljs-string">'233333abc'</span>);  <span class="hljs-comment"># 0</span>
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="8pregmatch">8.preg_match</h2>
<p>如果在进行正则表达式匹配的时候，没有限制字符串的开始和结束(^ 和 $)，则可以存在绕过的问题</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-variable">$ip</span> = <span class="hljs-string">'1.1.1.1 abcd'</span>; <span class="hljs-comment">// 可以绕过</span>
<span class="hljs-keyword">if</span>(!preg_match(<span class="hljs-string">"/(\d+)\.(\d+)\.(\d+)\.(\d+)/"</span>,<span class="hljs-variable">$ip</span>)) {
  <span class="hljs-keyword">die</span>(<span class="hljs-string">'error'</span>);
} <span class="hljs-keyword">else</span> {
   <span class="hljs-keyword">echo</span>(<span class="hljs-string">'key...'</span>);
}
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="9parsestr">9.parse_str</h2>
<p>与 parse_str() 类似的函数还有 mb_parse_str()，parse_str 将字符串解析成多个变量，如果参数str是URL传递入的查询字符串（query string），则将它解析为变量并设置到当前作用域。</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-comment">//var.php?var=new  </span>
<span class="hljs-variable">$var</span>=<span class="hljs-string">'init'</span>;  
parse_str(<span class="hljs-variable">$_SERVER</span>[<span class="hljs-string">'QUERY_STRING'</span>]);  
<span class="hljs-keyword">print</span> <span class="hljs-variable">$var</span>;</code></pre>
<h2 id="10字符串比较">10.字符串比较</h2>
<p>== 是弱类型的比较，以下比较都为 true</p>
<pre class="prettyprint"><code class="language-php hljs ">
<span class="hljs-preprocessor">&lt;?php</span>  
<span class="hljs-keyword">echo</span> <span class="hljs-number">0</span> == <span class="hljs-string">'a'</span> ;<span class="hljs-comment">// a 转换为数字为 0    重点注意</span>

<span class="hljs-comment">// 0x 开头会被当成16进制54975581388的16进制为 0xccccccccc</span>
<span class="hljs-comment">// 十六进制与整数，被转换为同一进制比较</span>
<span class="hljs-string">'0xccccccccc'</span> == <span class="hljs-string">'54975581388'</span> ;
<span class="hljs-comment">// 字符串在与数字比较前会自动转换为数字，如果不能转换为数字会变成0</span>
<span class="hljs-number">1</span> == <span class="hljs-string">'1'</span>;
<span class="hljs-number">1</span> == <span class="hljs-string">'01'</span>;
<span class="hljs-number">10</span> == <span class="hljs-string">'1e1'</span>;
<span class="hljs-string">'100'</span> == <span class="hljs-string">'1e2'</span> ;    

<span class="hljs-comment">// 十六进制数与带空格十六进制数，被转换为十六进制整数</span>
<span class="hljs-string">'0xABCdef'</span>  == <span class="hljs-string">'     0xABCdef'</span>;
<span class="hljs-keyword">echo</span> <span class="hljs-string">'0010e2'</span> == <span class="hljs-string">'1e3'</span>;
<span class="hljs-comment">// 0e 开头会被当成数字，又是等于 0*10^xxx=0</span>
<span class="hljs-comment">// 如果 md5 是以 0e 开头，在做比较的时候，可以用这种方法绕过</span>
<span class="hljs-string">'0e509367213418206700842008763514'</span> == <span class="hljs-string">'0e481036490867661113260034900752'</span>;
<span class="hljs-string">'0e481036490867661113260034900752'</span> == <span class="hljs-string">'0'</span> ;

var_dump(md5(<span class="hljs-string">'240610708'</span>) == md5(<span class="hljs-string">'QNKCDZO'</span>));
var_dump(md5(<span class="hljs-string">'aabg7XSs'</span>) == md5(<span class="hljs-string">'aabC9RqS'</span>));
var_dump(sha1(<span class="hljs-string">'aaroZmOk'</span>) == sha1(<span class="hljs-string">'aaK1STfY'</span>));
var_dump(sha1(<span class="hljs-string">'aaO8zKZF'</span>) == sha1(<span class="hljs-string">'aa3OFF9m'</span>));
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="11unset">11.unset</h2>
<p>unset(<span class="MathJax_Preview"></span><span aria-readonly="true" class="MathJax" id="MathJax-Element-1-Frame" role="textbox" style=""><nobr><span class="math" id="MathJax-Span-1" style="width: 16.817em; display: inline-block;"><span style="display: inline-block; position: relative; width: 13.44em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.867em, 1000em, 3.237em, -0.44em); top: -2.827em; left: 0em;"><span class="mrow" id="MathJax-Span-2"><span class="mi" id="MathJax-Span-3" style="font-family: MathJax_Math; font-style: italic;">b</span><span class="mi" id="MathJax-Span-4" style="font-family: MathJax_Math; font-style: italic;">a</span><span class="mi" id="MathJax-Span-5" style="font-family: MathJax_Math; font-style: italic;">r</span><span class="mo" id="MathJax-Span-6" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-7" style="font-family: MathJax_Main;">;</span><span class="texatom" id="MathJax-Span-8" style="padding-left: 0.167em;"><span class="mrow" id="MathJax-Span-9"><span class="mo" id="MathJax-Span-10"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>用</span></span></span></span><span class="texatom" id="MathJax-Span-11"><span class="mrow" id="MathJax-Span-12"><span class="mo" id="MathJax-Span-13"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>来</span></span></span></span><span class="texatom" id="MathJax-Span-14"><span class="mrow" id="MathJax-Span-15"><span class="mo" id="MathJax-Span-16"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>销</span></span></span></span><span class="texatom" id="MathJax-Span-17"><span class="mrow" id="MathJax-Span-18"><span class="mo" id="MathJax-Span-19"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>毁</span></span></span></span><span class="texatom" id="MathJax-Span-20"><span class="mrow" id="MathJax-Span-21"><span class="mo" id="MathJax-Span-22"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>指</span></span></span></span><span class="texatom" id="MathJax-Span-23"><span class="mrow" id="MathJax-Span-24"><span class="mo" id="MathJax-Span-25"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>定</span></span></span></span><span class="texatom" id="MathJax-Span-26"><span class="mrow" id="MathJax-Span-27"><span class="mo" id="MathJax-Span-28"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>的</span></span></span></span><span class="texatom" id="MathJax-Span-29"><span class="mrow" id="MathJax-Span-30"><span class="mo" id="MathJax-Span-31"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>变</span></span></span></span><span class="texatom" id="MathJax-Span-32"><span class="mrow" id="MathJax-Span-33"><span class="mo" id="MathJax-Span-34"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>量</span></span></span></span><span class="texatom" id="MathJax-Span-35"><span class="mrow" id="MathJax-Span-36"><span class="mo" id="MathJax-Span-37"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>，</span></span></span></span><span class="texatom" id="MathJax-Span-38"><span class="mrow" id="MathJax-Span-39"><span class="mo" id="MathJax-Span-40"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>如</span></span></span></span><span class="texatom" id="MathJax-Span-41"><span class="mrow" id="MathJax-Span-42"><span class="mo" id="MathJax-Span-43"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>果</span></span></span></span><span class="texatom" id="MathJax-Span-44"><span class="mrow" id="MathJax-Span-45"><span class="mo" id="MathJax-Span-46"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>变</span></span></span></span><span class="texatom" id="MathJax-Span-47"><span class="mrow" id="MathJax-Span-48"><span class="mo" id="MathJax-Span-49"><span style='font-family: STIXGeneral,"Arial Unicode MS",serif; font-size: 80%; font-style: normal; font-weight: normal;'>量</span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.827em;"></span></span></span><span style="border-left: 0em solid; display: inline-block; overflow: hidden; width: 0px; height: 1.446em; vertical-align: -0.379em;"></span></span></nobr></span><script id="MathJax-Element-1" type="math/tex">bar); 用来销毁指定的变量，如果变量 </script>bar 包含在请求参数中，可能出现销毁一些变量而实现程序逻辑绕过。</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>  
<span class="hljs-comment">// http://127.0.0.1/index.php?_CONFIG=123</span>
<span class="hljs-variable">$_CONFIG</span>[<span class="hljs-string">'extraSecure'</span>] = <span class="hljs-keyword">true</span>;

<span class="hljs-keyword">foreach</span>(<span class="hljs-keyword">array</span>(<span class="hljs-string">'_GET'</span>,<span class="hljs-string">'_POST'</span>) <span class="hljs-keyword">as</span> <span class="hljs-variable">$method</span>) {
    <span class="hljs-keyword">foreach</span>(<span class="hljs-variable">$$method</span> <span class="hljs-keyword">as</span> <span class="hljs-variable">$key</span>=&gt;<span class="hljs-variable">$value</span>) {
      <span class="hljs-comment">// $key == _CONFIG</span>
      <span class="hljs-comment">// $$key == $_CONFIG</span>
      <span class="hljs-comment">// 这个函数会把 $_CONFIG 变量销毁</span>
      <span class="hljs-keyword">unset</span>(<span class="hljs-variable">$$key</span>);
    }
}

<span class="hljs-keyword">if</span> (<span class="hljs-variable">$_CONFIG</span>[<span class="hljs-string">'extraSecure'</span>] == <span class="hljs-keyword">false</span>) {
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'flag {****}'</span>;
}
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="12intval">12.intval()</h2>
<p>int转string：</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-variable">$var</span> = <span class="hljs-number">5</span>;  
方式<span class="hljs-number">1</span>：<span class="hljs-variable">$item</span> = (string)<span class="hljs-variable">$var</span>;  
方式<span class="hljs-number">2</span>：<span class="hljs-variable">$item</span> = strval(<span class="hljs-variable">$var</span>); </code></pre>
<p>string转int：intval()函数。</p>
<pre class="prettyprint"><code class="language-php hljs ">var_dump(intval(<span class="hljs-string">'2'</span>)) <span class="hljs-comment">//2  </span>
var_dump(intval(<span class="hljs-string">'3abcd'</span>)) <span class="hljs-comment">//3  </span>
var_dump(intval(<span class="hljs-string">'abcd'</span>)) <span class="hljs-comment">//0 </span></code></pre>
<p>说明intval()转换的时候，会将从字符串的开始进行转换知道遇到一个非数字的字符。即使出现无法转换的字符串，intval()不会报错而是返回0。 <br>
利用代码：</br></p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>  
<span class="hljs-variable">$a</span> = <span class="hljs-string">'10000 union select * from yz'</span>;
<span class="hljs-keyword">if</span>(intval(<span class="hljs-variable">$a</span>)&gt;<span class="hljs-number">1000</span>)   
    <span class="hljs-keyword">echo</span> <span class="hljs-variable">$a</span> ;
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="13switch">13.switch()</h2>
<p>如果switch是数字类型的case的判断时，switch会将其中的参数转换为int类型。如下：</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-variable">$i</span> =<span class="hljs-string">"2abc"</span>;  
<span class="hljs-keyword">switch</span> (<span class="hljs-variable">$i</span>) {  
<span class="hljs-keyword">case</span> <span class="hljs-number">0</span>:  
<span class="hljs-keyword">case</span> <span class="hljs-number">1</span>:  
<span class="hljs-keyword">case</span> <span class="hljs-number">2</span>:  
<span class="hljs-keyword">echo</span> <span class="hljs-string">"i is less than 3 but not negative"</span>;  
<span class="hljs-keyword">break</span>;  
<span class="hljs-keyword">case</span> <span class="hljs-number">3</span>:  
<span class="hljs-keyword">echo</span> <span class="hljs-string">"i is 3"</span>;  
} 
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<p>这个时候程序输出的是i is less than 3 but not negative，是由于switch()函数将$i进行了类型转换，转换结果为2。</p>
<h2 id="14inarray">14.in_array()</h2>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-variable">$array</span>=[<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-string">'3'</span>];  
var_dump(in_array(<span class="hljs-string">'abc'</span>, <span class="hljs-variable">$array</span>)); <span class="hljs-comment">//true  </span>
var_dump(in_array(<span class="hljs-string">'1bc'</span>, <span class="hljs-variable">$array</span>)); <span class="hljs-comment">//true </span></code></pre>
<p>可以看到上面的情况返回的都是true,因为’abc’会转换为0，’1bc’转换为1。 <br>
在所有php认为是int的地方输入string，都会被强制转换</br></p>
<h2 id="15serialize-和-unserialize漏洞">15.serialize 和 unserialize漏洞</h2>
<p><strong>1.魔术方法</strong></p>
<p>这里我们先简单介绍一下php中的魔术方法（这里如果对于类、对象、方法不熟的先去学学吧），即Magic方法，php类可能会包含一些特殊的函数叫magic函数，magic函数命名是以符号__开头的，比如 __construct， __destruct，__toString，__sleep，__wakeup等等。这些函数都会在某些特殊时候被自动调用。 <br>
例如__construct()方法会在一个对象被创建时自动调用，对应的__destruct则会在一个对象被销毁时调用等等。 <br>
这里有两个比较特别的Magic方法，__sleep 方法会在一个对象被序列化的时候调用。 __wakeup方法会在一个对象被反序列化的时候调用。</br></br></p>
<p>在这里介绍一个序列化漏洞，首先不要相信用户输入的一切 <br>
看下面代码</br></p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">test</span>
{</span>
    <span class="hljs-keyword">public</span> <span class="hljs-variable">$username</span> = <span class="hljs-string">''</span>;
    <span class="hljs-keyword">public</span> <span class="hljs-variable">$password</span> = <span class="hljs-string">''</span>;
    <span class="hljs-keyword">public</span> <span class="hljs-variable">$file</span> = <span class="hljs-string">''</span>;
    <span class="hljs-keyword">public</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">out</span><span class="hljs-params">()</span>{</span>
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"username: "</span>.<span class="hljs-variable">$this</span>-&gt;username.<span class="hljs-string">"&lt;br&gt;"</span>.<span class="hljs-string">"password: "</span>.<span class="hljs-variable">$this</span>-&gt;password ;
    }
     <span class="hljs-keyword">public</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">__toString</span><span class="hljs-params">()</span> {</span>
        <span class="hljs-keyword">return</span> file_get_contents(<span class="hljs-variable">$this</span>-&gt;file);
    }
}
<span class="hljs-variable">$a</span> = <span class="hljs-keyword">new</span> test();
<span class="hljs-variable">$a</span>-&gt;file = <span class="hljs-string">'C:\Users\YZ\Desktop\plan.txt'</span>;
<span class="hljs-keyword">echo</span> serialize(<span class="hljs-variable">$a</span>);
<span class="hljs-preprocessor">?&gt;</span>
<span class="hljs-comment">//tostring方法会在输出实例的时候执行，如果实例路径是隐秘文件就可以读取了</span>
</code></pre>
<p>下面就可以读取了C:\Users\YZ\Desktop\plan.txt文件了 <br>
echo unserialize触发了__tostring函数</br></p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">test</span>
{</span>
    <span class="hljs-keyword">public</span> <span class="hljs-variable">$username</span> = <span class="hljs-string">''</span>;
    <span class="hljs-keyword">public</span> <span class="hljs-variable">$password</span> = <span class="hljs-string">''</span>;
    <span class="hljs-keyword">public</span> <span class="hljs-variable">$file</span> = <span class="hljs-string">''</span>;
    <span class="hljs-keyword">public</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">out</span><span class="hljs-params">()</span>{</span>
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"username: "</span>.<span class="hljs-variable">$this</span>-&gt;username.<span class="hljs-string">"&lt;br&gt;"</span>.<span class="hljs-string">"password: "</span>.<span class="hljs-variable">$this</span>-&gt;password ;
    }
     <span class="hljs-keyword">public</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">__toString</span><span class="hljs-params">()</span> {</span>
        <span class="hljs-keyword">return</span> file_get_contents(<span class="hljs-variable">$this</span>-&gt;file);
    }
}
<span class="hljs-variable">$a</span> = <span class="hljs-string">'O:4:"test":3:{s:8:"username";s:0:"";s:8:"password";s:0:"";s:4:"file";s:28:"C:\Users\YZ\Desktop\plan.txt";}'</span>;
<span class="hljs-keyword">echo</span> unserialize(<span class="hljs-variable">$a</span>);
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<h2 id="16session-反序列化漏洞">16.session 反序列化漏洞</h2>
<p>主要原因是 <br>
ini_set(‘session.serialize_handler’, ‘php_serialize’); <br>
ini_set(‘session.serialize_handler’, ‘php’); <br>
两者处理session的方式不同</br></br></br></p>
<p>利用下面代码可以生成session值</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
ini_set(<span class="hljs-string">'session.serialize_handler'</span>, <span class="hljs-string">'php_serialize'</span>);<span class="hljs-comment">//a:1:{s:6:"spoock";s:3:"111";}</span>
<span class="hljs-comment">//ini_set('session.serialize_handler', 'php');//a|s:3:"111"</span>
session_start();
<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">"spoock"</span>]=<span class="hljs-variable">$_GET</span>[<span class="hljs-string">"a"</span>];
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<p>我们来看看生成的session值</p>
<pre class="prettyprint"><code class="language-php hljs ">
spoock|s:<span class="hljs-number">3</span>:<span class="hljs-string">"111"</span>;    <span class="hljs-comment">//session键值|内容序列化</span>
a:<span class="hljs-number">1</span>:{s:<span class="hljs-number">6</span>:<span class="hljs-string">"spoock"</span>;s:<span class="hljs-number">3</span>:<span class="hljs-string">"111"</span>;}a:<span class="hljs-number">1</span>:{s:N:session键值;内容序列化}
在ini_set(<span class="hljs-string">'session.serialize_handler'</span>, <span class="hljs-string">'php'</span>);中把|之前认为是键值后面的视为序列化
那么就可以利用这一漏洞执行一些恶意代码</code></pre>
<p>看下面的例子 <br>
1.php</br></p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
ini_set(<span class="hljs-string">'session.serialize_handler'</span>, <span class="hljs-string">'php_serialize'</span>);
session_start();
<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">"spoock"</span>]=<span class="hljs-variable">$_GET</span>[<span class="hljs-string">"a"</span>];
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<p>2.php</p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span>
   ini_set(<span class="hljs-string">'session.serialize_handler'</span>, <span class="hljs-string">'php'</span>);
session_start();
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">lemon</span> {</span>
    <span class="hljs-keyword">var</span> <span class="hljs-variable">$hi</span>;
    <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">__construct</span><span class="hljs-params">()</span>{</span>
        <span class="hljs-variable">$this</span>-&gt;hi = <span class="hljs-string">'phpinfo();'</span>;
    }

    <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">__destruct</span><span class="hljs-params">()</span> {</span>
         <span class="hljs-keyword">eval</span>(<span class="hljs-variable">$this</span>-&gt;hi);<span class="hljs-comment">//这里很危险，可以执行用户输入的参数</span>
    }
}
<span class="hljs-preprocessor">?&gt;</span>
</code></pre>
<p>在1.PHP里面输入a参数序列化的值|O:5:”lemon”:1:{s:2:”hi”;s:10:”phpinfo();”;}  <br>
则被序列化为 <br>
a:1:{s:6:”spoock”;s:44:”|O:5:”lemon”:1:{s:2:”hi”;s:10:”phpinfo();”;} <br>
在2.PHP里面打开 <br>
就可以执行phpinfo（）了 <br>
<a href="http://web.jarvisoj.com:32784/">有道web题</a> <br>
<a href="http://blog.csdn.net/qq_31481187/article/details/53189113">这里是题解</a></br></br></br></br></br></br></p></hr></div>