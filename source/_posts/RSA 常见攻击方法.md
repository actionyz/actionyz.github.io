---
title: RSA 常见攻击方法
tags: [密码学应用]
date: 2017-04-22 15:55
---
0x01 RSA简介  那么，有无可能在已知n和e的情况下，推导出d？   首先要知道        　　（1）ed≡1 (mod φ(n))。只有知道e和φ(n)，才能算出d。       　　（2）φ(n)=(p-1)(q-1)。只有知道p和q，才能算出φ(n)。       　　（3）n=pq。只有将n因数分解，才能算出p和q。   结论：如果n可以被因数分解，d就可以算出，也就意
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="0x01-rsa简介">0x01 RSA简介</h1>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170621001317480?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>那么，有无可能在已知n和e的情况下，推导出d？ <br>
  首先要知道 <img alt="这里写图片描述" src="https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/s=116/sign=39fb85b64010b912bbc1f2fff5fcfcb5/83025aafa40f4bfb0f075c59044f78f0f63618ec.jpg" title=""> <br>
      　　（1）ed≡1 (mod φ(n))。只有知道e和φ(n)，才能算出d。 <br>
      　　（2）φ(n)=(p-1)(q-1)。只有知道p和q，才能算出φ(n)。 <br>
      　　（3）n=pq。只有将n因数分解，才能算出p和q。 <br>
  结论：如果n可以被因数分解，d就可以算出，也就意味着私钥被破解。</br></br></br></br></img></br></p>
</blockquote>
<h1 id="0x02-常见的rsa攻击方法">0x02 常见的RSA攻击方法</h1>
<h2 id="0x1-共模攻击">0x1 共模攻击</h2>
<blockquote>
<p>共模攻击，也称同模攻击，英文原名是 Common Modulus Attack 。 <br>
  同模攻击利用的大前提就是，RSA体系在生成密钥的过程中使用了相同的模数n。 <br>
  假设COMPANY用所有公钥加密了同一条信息M，也就是 <br>
  c1 = m^e1%n <br>
  c2 = m^e2%n <br>
  此时员工A拥有密钥d1他可以通过 <br>
  m = c1^d1%n <br>
  解密得到消息m <br>
  同时员工B拥有密钥d2 <br>
  他可以通过 <br>
  m = c2^d2%n <br>
  解密得到消息m如果，此时有一个攻击者，同时监听了A和B接收到的密文C1,C2，因为模数不变，以及所有公钥都是公开的，那么利用同模攻击，他就可以在不知道d1，d2的情况下解密得到消息m。 </br></br></br></br></br></br></br></br></br></br></br></p>
</blockquote>
<p>贴出破解脚本：</p>
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

    n= x
    e1= x
    e2= x
    c1= x
    c2= x
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
    m = (pow(c1,s1,n)*pow(c2,s2,n))%n
    <span class="hljs-keyword">print</span> m
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">'__main__'</span>:
    main()</code></pre>
<h1 id="0x03-rsa题目">0x03 RSA题目</h1>
<h2 id="0x1-veryeasyrsa">0x1 veryeasyRSA</h2>
<blockquote>
<p>已知RSA公钥生成参数： <br>
  p = 3487583947589437589237958723892346254777  <br>
  q = 8767867843568934765983476584376578389 <br>
  e = 65537 <br>
  求d =  <br>
  请提交PCTF{d} <br>
  直接写py脚本</br></br></br></br></br></br></p>
</blockquote>
<pre class="prettyprint"><code class=" hljs makefile">from libnum import invmod
<span class="hljs-constant">p</span> = 3487583947589437589237958723892346254777 
<span class="hljs-constant">q</span> = 8767867843568934765983476584376578389
<span class="hljs-constant">e</span> = 65537
<span class="hljs-constant">fn</span> = (p-1)*(q-1)
<span class="hljs-constant">d</span> = invmod(e,fn)
print d</code></pre>
<h2 id="0x2-easy-rsa">0x2 Easy RSA</h2>
<blockquote>
<p>还记得veryeasy RSA吗？是不是不难？那继续来看看这题吧，这题也不难。 <br>
  已知一段RSA加密的信息为：0xdc2eeeb2782c且已知加密所用的公钥： <br>
  (N=322831561921859 e = 23) <br>
  请解密出明文，提交时请将数字转化为ascii码提交 <br>
  比如你解出的明文是0x6162，那么请提交字符串ab</br></br></br></br></p>
</blockquote>
<p>首先利用在线分解工具分解大整数 <br>
N = 13574881 * 23781539 <br>
利用脚本解密 <br>
注意要写一个快速计算的额脚本</br></br></br></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-comment"># coding = utf-8</span>
<span class="hljs-keyword">import</span> libnum
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">fastExpMod</span><span class="hljs-params">(b, e, m)</span>:</span>
    <span class="hljs-string">"""
    e = e0*(2^0) + e1*(2^1) + e2*(2^2) + ... + en * (2^n)

    b^e = b^(e0*(2^0) + e1*(2^1) + e2*(2^2) + ... + en * (2^n))
        = b^(e0*(2^0)) * b^(e1*(2^1)) * b^(e2*(2^2)) * ... * b^(en*(2^n))

    b^e mod m = ((b^(e0*(2^0)) mod m) * (b^(e1*(2^1)) mod m) * (b^(e2*(2^2)) mod m) * ... * (b^(en*(2^n)) mod m) mod m
    """</span>
    result = <span class="hljs-number">1</span>
    <span class="hljs-keyword">while</span> e != <span class="hljs-number">0</span>:
        <span class="hljs-keyword">if</span> (e&amp;<span class="hljs-number">1</span>) == <span class="hljs-number">1</span>:
            <span class="hljs-comment"># ei = 1, then mul</span>
            result = (result * b) % m
        e &gt;&gt;= <span class="hljs-number">1</span>
        <span class="hljs-comment"># b, b^2, b^4, b^8, ... , b^(2^n)</span>
        b = (b*b) % m
    <span class="hljs-keyword">return</span> result


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">decryption</span><span class="hljs-params">(C, d, n)</span>:</span>
    <span class="hljs-comment">#RSA M = C^d mod n</span>
    <span class="hljs-keyword">return</span> fastExpMod(C, d, n)

p = <span class="hljs-number">13574881</span>
q = <span class="hljs-number">23781539</span>
n = p * q
fn = (p - <span class="hljs-number">1</span>) * (q - <span class="hljs-number">1</span>)
e = <span class="hljs-number">23</span>
d = libnum.invmod(e,fn)
<span class="hljs-keyword">print</span> d
C = int(<span class="hljs-string">'0xdc2eeeb2782c'</span>, <span class="hljs-number">16</span>)
M = decryption(C, d, n)
flag = str(hex(M))[<span class="hljs-number">2</span>:-<span class="hljs-number">1</span>]
<span class="hljs-keyword">print</span> flag.decode(<span class="hljs-string">'hex'</span>)</code></pre>
<h2 id="0x3-medium-rsa">0x3 Medium RSA</h2>
<p>首先利用openssl 生成n e</p>
<pre class="prettyprint"><code class=" hljs ruby">root<span class="hljs-variable">@kali</span><span class="hljs-symbol">:/media/sf_vboxshare/mediumRSA</span><span class="hljs-comment"># openssl rsa -pubin -text -modulus -in pubkey.pem</span>
<span class="hljs-constant">Public</span>-<span class="hljs-constant">Key</span><span class="hljs-symbol">:</span> (<span class="hljs-number">256</span> bit)
<span class="hljs-constant">Modulus</span><span class="hljs-symbol">:</span>
    <span class="hljs-number">00</span><span class="hljs-symbol">:c2</span><span class="hljs-symbol">:</span><span class="hljs-number">63</span><span class="hljs-symbol">:</span><span class="hljs-number">6</span><span class="hljs-symbol">a:</span><span class="hljs-symbol">e5:</span><span class="hljs-symbol">c3:</span><span class="hljs-symbol">d8:</span><span class="hljs-symbol">e4:</span><span class="hljs-number">3</span><span class="hljs-symbol">f:</span><span class="hljs-symbol">fb:</span><span class="hljs-number">97</span><span class="hljs-symbol">:ab</span><span class="hljs-symbol">:</span>09<span class="hljs-symbol">:</span><span class="hljs-number">02</span><span class="hljs-symbol">:</span><span class="hljs-number">8</span><span class="hljs-symbol">f:</span>
    <span class="hljs-number">1</span><span class="hljs-symbol">a:</span><span class="hljs-symbol">ac:</span><span class="hljs-number">6</span><span class="hljs-symbol">c:</span>0<span class="hljs-symbol">b:</span><span class="hljs-symbol">f6:</span><span class="hljs-symbol">cd:</span><span class="hljs-number">3</span><span class="hljs-symbol">d:</span><span class="hljs-number">70</span><span class="hljs-symbol">:eb</span><span class="hljs-symbol">:ca</span><span class="hljs-symbol">:</span><span class="hljs-number">28</span><span class="hljs-symbol">:</span><span class="hljs-number">1</span><span class="hljs-symbol">b:</span><span class="hljs-symbol">ff:</span><span class="hljs-symbol">e9:</span><span class="hljs-number">7</span><span class="hljs-symbol">f:</span>
    <span class="hljs-symbol">be:</span><span class="hljs-number">30</span><span class="hljs-symbol">:dd</span>
<span class="hljs-constant">Exponent</span><span class="hljs-symbol">:</span> <span class="hljs-number">65537</span> (<span class="hljs-number">0x10001</span>)
<span class="hljs-constant">Modulus</span>=<span class="hljs-constant">C2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD</span>
writing <span class="hljs-constant">RSA</span> key
-----<span class="hljs-constant">BEGIN</span> <span class="hljs-constant">PUBLIC</span> <span class="hljs-constant">KEY</span>-----
<span class="hljs-constant">MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMJjauXD2OQ</span>/+<span class="hljs-number">5</span>erCQKPGqxsC/bNPXDr
yigb/+l/vjDdAgMBAAE=</code></pre>
<p>其中Exponent即为e值，Modulus即为N值，用yafu分解。 <br>
N = 87924348264132406875276140514499937145050893665602592992418171647042491658461 <br>
factor(87924348264132406875276140514499937145050893665602592992418171647042491658461)</br></br></p>
<p>得到</p>
<p>p = 275127860351348928173285174381581152299 <br>
q = 319576316814478949870590164193048041239 <br>
d = 10866948760844599168252082612378495977388271279679231539839049698621994994673</br></br></p>
<p>利用python生成秘钥</p>
<pre class="prettyprint"><code class=" hljs java"># coding=utf-<span class="hljs-number">8</span>
<span class="hljs-keyword">import</span> math
<span class="hljs-keyword">import</span> sys
from Crypto.PublicKey <span class="hljs-keyword">import</span> RSA

keypair = RSA.generate(<span class="hljs-number">1024</span>)

keypair.p = <span class="hljs-number">275127860351348928173285174381581152299</span>
keypair.q = <span class="hljs-number">319576316814478949870590164193048041239</span>
keypair.e = <span class="hljs-number">65537</span>

keypair.n = keypair.p * keypair.q
Qn = <span class="hljs-keyword">long</span>((keypair.p-<span class="hljs-number">1</span>) * (keypair.q-<span class="hljs-number">1</span>))

i = <span class="hljs-number">1</span>
<span class="hljs-keyword">while</span> (True):
    x = (Qn * i ) + <span class="hljs-number">1</span>
    <span class="hljs-keyword">if</span> (x % keypair.e == <span class="hljs-number">0</span>):
        keypair.d = x / keypair.e
        <span class="hljs-keyword">break</span>
    i += <span class="hljs-number">1</span>

<span class="hljs-keyword">private</span> = open(<span class="hljs-string">'private.pem'</span>,<span class="hljs-string">'w'</span>)
<span class="hljs-keyword">private</span>.<span class="hljs-title">write</span>(keypair.<span class="hljs-title">exportKey</span>())
<span class="hljs-keyword">private</span>.<span class="hljs-title">close</span>()</code></pre>
<p>然后直接用私钥解密。</p>
<pre class="prettyprint"><code class=" hljs ruby">root<span class="hljs-variable">@kali</span><span class="hljs-symbol">:/media/sf_vboxshare/mediumRSA</span><span class="hljs-comment"># openssl rsautl -decrypt -in flag.enc -inkey private.pem -out flag.dec</span>
root<span class="hljs-variable">@kali</span><span class="hljs-symbol">:/media/sf_vboxshare/mediumRSA</span><span class="hljs-comment"># cat flag.dec</span>
<span class="hljs-constant">PCTF</span>{<span class="hljs-number">256</span>b_i5_m3dium}
</code></pre></div>