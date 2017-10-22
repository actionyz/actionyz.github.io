---
title: RSA&AES实现可靠通信
tags: [密码学应用]
date: 2017-01-19 11:05
---
密码作业要求自己设计一个安全可靠的通信，目前已有的SSL通信是广为流传的大家都在用的通信机制，在这用Socket实现模拟SSL协议通信过程
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="通信过程设计">通信过程设计</h1>
<p>在做实验之前，想直接用封装好的SSL安全套接字接口。为了搞懂其中的加密机制自己用Socket接口模拟了简单的安全套接字的验证过程（这里我没有认证机构，也没有证书，但用一对事先生成的RSA）。先来简单的描述SSL验证机制，接着介绍我设计的安全通信协议。</p>
<h2 id="一ssl安全通信机制">一、SSL安全通信机制</h2>
<h3 id="1通信机制">1.通信机制</h3>
<blockquote>
<p>CA 机构，又称为证书认证中心 (Certificate Authority) 中心，是一个负责发放和管理数字证书的第三方权威机构，它负责管理PKI结构下的所有用户(包括各种应用程序)的证书，把用户的公钥和用户的其他信息捆绑在一起，在网上验证用户的身份。CA机构的数字签名使得攻击者不能伪造和篡改证书。</p>
</blockquote>
<p>1．SSL客户端（也是TCP的客户端）在TCP链接建立之后，发出一个Clienth*llo来发起握手，这个消息里面包含了自己可实现的算法列表和其它一些需要的消息。 <br>
2．SSL的服务器端会回应一个Serverh*llo，这里面确定了这次通信所需要的算法，然后发过去自己的证书（里面包含了身份和自己的公钥以及可靠机构的数字签名）。 <br>
3．Client在收到这个消息后会生成一个秘密消息，用SSL服务器的公钥加密后传过去。 <br>
4．SSL服务器端用自己的私钥解密后，会话密钥协商成功，双方可以用同一份会话密钥来通信了。</br></br></br></p>
<p><strong>证书校验：</strong> <br>
证书的目的就是让客户端相信公钥确实是服务器发过来的，CA用自己的私钥加密证书，Client用CA的公钥解密，如果解密成功，既证明了证书的真实性。</br></p>
<h3 id="2存在的问题">2.存在的问题</h3>
<p>利用SSL通信能够有效的防治第三者盗取修改服务器公钥以及截获客户端发送的密文。但是存在以下不可规避的问题：</p>
<blockquote>
<p>客户端say hello 后，被代理服务器拦截，代理服务器也有CA颁发的证书，他把自己的合法证书发给客户端，客户端用证书里面的公钥解密签名之后，得到了各种信息，一看信息都对上了，没毛病信任。然后代理服务器宰相真正的服务器say hello，服务器下发真正的证书，代理服务器假装自己是客户端，信任了证书，然后校验通过，发用真正公钥加密后的随机数给服务器，服务器当然能解开，也就是代理服务器作为客户端和真正的服务器建立了合法的通信。再看客户端在校验了代理服务器自己的证书后也建立了合法的通信，这个时候，代理服务器作为中间人，因为他有自己的私钥和真正的公钥，可以欺上瞒下，俩边的信息都从它这里过了一次，信息被他偷窥到。攻击成功。</p>
</blockquote>
<p>在这里解决办法就是客户端内置真正的公钥，当代理服务器把它自己的证书传过来的时候，客户端用内置的公钥去解密证书中的签名，因为不是真正的私钥加密的所以解密失败，校验也就失败，连接中断。这也就是客户端信任所有的证书的风险—被中间人攻击。</p>
<h1 id="二设计可靠通信过程">二、设计可靠通信过程</h1>
<p>在这里为了简单实现，自己当做了认证机构，证书格式不是标准格式只有公钥hash值签名以及公钥明文（这里就没有写其他的信息例如持有人、签发机构、国家、地区等），同时颁发证书也是直接物理给的没有设服务器，并将证书公钥直接内置到client（这里防止中间代理服务器攻击上文所述）。</p>
<p><strong>通信协议如下：</strong> <br>
1. 客户端连接服务端首先建立没有安全机制的Socket TCP连接 <br>
2. 服务端向客户端颁发证书（这里是自定义格式 上文所述） <br>
3. 客户端用CA内置公钥（自己是认证机构）验证服务器证书的真实性（通过CA公钥解密 签过名的Server公钥并与文件MD5比对） <br>
4. 前面身份认证完成后，客户端在本地生成随机AES加密密钥以及初始向量（Key&amp;Vi）,使用证书中的公钥加密后传送给服务器 <br>
5. 服务器收到客户端信息后，用自己申请证书的私钥解密，得到客户端随机生成的AES加密密钥，完成了密钥协商的操作 <br>
6. 之后客户端、服务器对等通信，相互发送加密消息</br></br></br></br></br></br></p>
<p><strong>通信图如下：</strong> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170119110259057?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>通过以上方法模拟实现的SSL通信协议，简化了协议本身，但基本实现了协议功能。</p>
<h1 id="三代码实现">三、代码实现</h1>
<h2 id="1server">1.Server</h2>
<p><strong>RSA密钥生成及证书颁发</strong></p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-comment">//generateKey();                                                    //生成密钥</span>
    FILE *Public;
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">int</span> Pub_len;
    <span class="hljs-keyword">char</span> RSA_Buf[RSA_Buf_Len] = { <span class="hljs-number">0</span> };
    Public = fopen(<span class="hljs-string">"public.pem"</span>, <span class="hljs-string">"rb"</span>);
    fseek(Public, <span class="hljs-number">0L</span>, SEEK_END);                                        <span class="hljs-comment">//获取长度</span>
    Pub_len = ftell(Public);
    fseek(Public, <span class="hljs-number">0L</span>, SEEK_SET);
    fread(RSA_Buf, <span class="hljs-number">1</span>, Pub_len, Public);

    send(ClientSocket, (<span class="hljs-keyword">char</span> *)&amp;Pub_len, <span class="hljs-number">4</span>, <span class="hljs-number">0</span>);                         <span class="hljs-comment">//发送公钥长度</span>
    send(ClientSocket, RSA_Buf, Pub_len, <span class="hljs-number">0</span>);                            <span class="hljs-comment">//发送公钥文本</span>

    md5(RSA_Buf);                                                       <span class="hljs-comment">//获取公钥的md5值</span>

    <span class="hljs-built_in">string</span> m;
    m.assign(MD5_string, <span class="hljs-number">32</span>);
    <span class="hljs-built_in">string</span> m1 = DS_privateKey(m);                                       <span class="hljs-comment">//进行md5的数字签名利用第三方提供的私钥 这里可以是证书</span>

    <span class="hljs-keyword">char</span> *plain = (<span class="hljs-keyword">char</span> *)m1.c_str();
    send(ClientSocket, plain, <span class="hljs-number">256</span>, <span class="hljs-number">0</span>);                                  <span class="hljs-comment">//发送签证后的md5码</span>
</code></pre>
<p><strong>接收来自S的RSA加密的AES密钥</strong></p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">int</span> KEY_Len;
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> KEY_RSA[MSG_LEN];
    <span class="hljs-built_in">memset</span>(KEY_RSA, <span class="hljs-number">0</span>, MSG_LEN);

    recvn(ClientSocket, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)&amp;KEY_Len, <span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">int</span>));<span class="hljs-comment">//接收来自另一方RSA公钥加密的AES密钥长度</span>
    recvn(ClientSocket, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)KEY_RSA, KEY_Len);             <span class="hljs-comment">//接收公钥加密的AES密钥</span>

    <span class="hljs-built_in">string</span> miwen;
    miwen.assign((<span class="hljs-keyword">char</span> *)KEY_RSA, <span class="hljs-number">256</span>);
    <span class="hljs-built_in">string</span> c = DS_publicKey(miwen);
    c = bio_read_privateKey(c);
    <span class="hljs-built_in">memcpy</span>(key, c.c_str(), <span class="hljs-number">16</span>);                                         <span class="hljs-comment">//利用私钥解开AES密钥密文</span></code></pre>
<h2 id="2client">2.Client</h2>
<p><strong>RSA公钥接收&amp;身份认证</strong></p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-keyword">char</span> RSA_Buf[RSA_Buf_Len];
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> KEY_RSA[MSG_LEN];
    <span class="hljs-built_in">memset</span>(RSA_Buf, <span class="hljs-number">0</span>, MSG_LEN);
    recvn(CientSocket, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)&amp;Pub_len, <span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">int</span>));
    recvn(CientSocket, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)RSA_Buf, Pub_len);
    fwrite(RSA_Buf, Pub_len, <span class="hljs-number">1</span>, Public);
    fclose(Public);                                                             <span class="hljs-comment">//接收明文传送的公钥</span>

    md5(RSA_Buf);                                                               <span class="hljs-comment">//获取公钥的md5值</span>
    recvn(CientSocket, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)KEY_RSA, <span class="hljs-number">256</span>);                          <span class="hljs-comment">//接收签名过的MD5值</span>
    <span class="hljs-built_in">string</span> DS_public;
    DS_public.assign((<span class="hljs-keyword">char</span> *)KEY_RSA, <span class="hljs-number">256</span>);
    <span class="hljs-built_in">string</span> Real_MD5 = DS_publicKey(DS_public);                                  <span class="hljs-comment">//校验MD5值</span>
    <span class="hljs-built_in">cout</span> &lt;&lt; <span class="hljs-string">"Hash值校验："</span> &lt;&lt; endl;
    <span class="hljs-built_in">cout</span> &lt;&lt; <span class="hljs-string">"Server MD5:"</span> &lt;&lt; Real_MD5&lt;&lt;endl;
    <span class="hljs-built_in">cout</span> &lt;&lt; <span class="hljs-string">"Client MD5:"</span> &lt;&lt; MD5_string&lt;&lt;endl;
    <span class="hljs-keyword">if</span> (Real_MD5 == MD5_string)
        <span class="hljs-built_in">cout</span> &lt;&lt; <span class="hljs-string">"对方身份认证成功！！！请放心通信！！！"</span> &lt;&lt; endl;
    <span class="hljs-keyword">else</span>
        <span class="hljs-built_in">cout</span> &lt;&lt; <span class="hljs-string">"对方身份验证失败！！！请重新连接！！！"</span> &lt;&lt; endl;
    <span class="hljs-built_in">cout</span> &lt;&lt; <span class="hljs-string">"本次通信采用的安全算法:RSA 2048、AES（CBC）"</span> &lt;&lt; endl;</code></pre>
<p><strong>利用公钥向C端发送KEY&amp;VI</strong></p>
<pre class="prettyprint"><code class=" hljs cpp">
    randnum();                                                                  <span class="hljs-comment">//随机生成AES密钥</span>
    <span class="hljs-built_in">string</span> m = bio_read_publicKey(key);
    <span class="hljs-built_in">string</span> m1 = DS_privateKey(m);
    <span class="hljs-built_in">string</span> c = DS_publicKey(m1);

    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *p = (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)m1.c_str();
    KEY_Len = <span class="hljs-number">256</span>;
    Ret = send(CientSocket, (<span class="hljs-keyword">char</span> *)&amp;KEY_Len, <span class="hljs-number">4</span>, <span class="hljs-number">0</span>);
    send(CientSocket, (<span class="hljs-keyword">char</span> *)p, KEY_Len, <span class="hljs-number">0</span>);                                   <span class="hljs-comment">//发送公钥加密的AES密钥</span></code></pre>
<h1 id="四功能演示">四、功能演示</h1>
<p>1.Server <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170119103559255?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
2.Client <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170119103621287?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></br></img></br></p>
<p>3.通信过程 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170119103951980?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h1 id="五实验感悟">五、实验感悟</h1>
<p>通过本次实验我又进一步学习了SSL协议以及各种加密算法，虽然没有用到安全套接字但是自己模拟出了简单的安全通信协议，实践才是检验知识掌握的有效方法。</p>
<blockquote>
<p>参考资料： <br>
<a href="http://www.tuicool.com/articles/BnInUvy">SSL证书的合法性检验</a> <br>
<a href="http://www.cnblogs.com/AloneSword/p/3662962.html">SSL协议与数字证书原理</a> <br>
<a href="http://www.wosign.com/Basic/howsslwork.htm">SSL工作原理</a> <br>
<a href="https://segmentfault.com/a/1190000002568019">PKI、CA与证书格式&amp;使用</a></br></br></br></br></p>
</blockquote></div>