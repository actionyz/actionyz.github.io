---
title: 利用OpenSSL 实现明文加解密
tags: [密码学应用]
date: 2016-12-26 09:59
---
最近在写OpenSSL的作业，发现网上的代码很乱，整理了一下
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="rsa">RSA</h1>
<h2 id="1密钥生成">1.密钥生成</h2>
<pre class="prettyprint"><code class=" hljs objectivec"><span class="hljs-keyword">void</span> generateKey() {
    <span class="hljs-comment">/* 生成公钥 */</span>
    RSA* rsa = RSA_generate_key(<span class="hljs-number">1024</span>, RSA_F4, <span class="hljs-literal">NULL</span>, <span class="hljs-literal">NULL</span>);
    BIO *bp = BIO_new(BIO_s_file());
    BIO_write_filename(bp, <span class="hljs-string">"public.pem"</span>);
    PEM_write_bio_RSAPublicKey(bp, rsa);
    BIO_free_all(bp);
    <span class="hljs-comment">/* 生成私钥 */</span>
    bp = BIO_new_file(<span class="hljs-string">"private.pem"</span>, <span class="hljs-string">"w+"</span>);
    PEM_write_bio_RSAPrivateKey(bp, rsa, <span class="hljs-literal">NULL</span>, <span class="hljs-literal">NULL</span>, <span class="hljs-number">4</span>, <span class="hljs-literal">NULL</span>, <span class="hljs-literal">NULL</span>);
    BIO_free_all(bp);
    RSA_free(rsa);
}</code></pre>
<h2 id="2公钥加密">2.公钥加密</h2>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-built_in">string</span> bio_read_publicKey(<span class="hljs-built_in">string</span> data) {
    OpenSSL_add_all_algorithms();
    BIO* bp = BIO_new(BIO_s_file());
    BIO_read_filename(bp, <span class="hljs-string">"public.pem"</span>);
    RSA* rsaK = PEM_read_bio_RSAPublicKey(bp, NULL, NULL, NULL);
    <span class="hljs-keyword">if</span> (NULL == rsaK) {
        perror(<span class="hljs-string">"read key file fail!"</span>);
    }
    <span class="hljs-keyword">else</span> {
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"read success!"</span>);
        <span class="hljs-keyword">int</span> nLen = RSA_size(rsaK);
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"len:%d\n"</span>, nLen);
    }
    <span class="hljs-keyword">int</span> nLen = RSA_size(rsaK);
    <span class="hljs-keyword">char</span> *pEncode = <span class="hljs-keyword">new</span> <span class="hljs-keyword">char</span>[nLen + <span class="hljs-number">1</span>];
    <span class="hljs-keyword">int</span> ret = RSA_public_encrypt(data.length(), (<span class="hljs-keyword">const</span> <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)data.c_str(),
        (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)pEncode, rsaK, RSA_PKCS1_PADDING);
    <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span> strRet;
    <span class="hljs-keyword">if</span> (ret &gt;= <span class="hljs-number">0</span>) {
        strRet = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>(pEncode, ret);  
    }
    <span class="hljs-keyword">delete</span>[] pEncode;
    CRYPTO_cleanup_all_ex_data();
    BIO_free_all(bp);
    RSA_free(rsaK);
    <span class="hljs-keyword">return</span> strRet;
}</code></pre>
<h2 id="3私钥解密">3.私钥解密</h2>
<pre class="prettyprint"><code class=" hljs haskell"><span class="hljs-title">string</span> bio_read_privateKey(string <span class="hljs-typedef"><span class="hljs-keyword">data</span>) <span class="hljs-container">{
    <span class="hljs-type">OpenSSL_add_all_algorithms</span>();
    <span class="hljs-type">BIO</span>* <span class="hljs-title">bp</span> = <span class="hljs-type">BIO_new</span>(<span class="hljs-type">BIO_s_file</span>());
    <span class="hljs-type">BIO_read_filename</span>(<span class="hljs-title">bp</span>, "<span class="hljs-title">private</span>.<span class="hljs-title">pem</span>");
    <span class="hljs-type">RSA</span>* <span class="hljs-title">rsaK</span> = <span class="hljs-type">PEM_read_bio_RSAPrivateKey</span>(<span class="hljs-title">bp</span>, <span class="hljs-type">NULL</span>, <span class="hljs-type">NULL</span>, <span class="hljs-type">NULL</span>);
    <span class="hljs-title">if</span> (<span class="hljs-type">NULL</span> == <span class="hljs-title">rsaK</span>) {
        <span class="hljs-title">perror</span>("<span class="hljs-title">read</span> <span class="hljs-title">key</span> <span class="hljs-title">file</span> <span class="hljs-title">fail</span>!");
    }</span></span>
    <span class="hljs-keyword">else</span> {
        printf(<span class="hljs-string">"read success!\n"</span>);
    }
    int nLen = <span class="hljs-type">RSA_size</span>(rsaK); 
    char *pEncode = new char[nLen + <span class="hljs-number">1</span>];
    int ret = <span class="hljs-type">RSA_private_decrypt</span>(<span class="hljs-typedef"><span class="hljs-keyword">data</span>.length<span class="hljs-container">()</span>, <span class="hljs-container">(<span class="hljs-title">const</span> <span class="hljs-title">unsigned</span> <span class="hljs-title">char</span>*)</span><span class="hljs-keyword">data</span>.c_str<span class="hljs-container">()</span>, <span class="hljs-container">(<span class="hljs-title">unsigned</span> <span class="hljs-title">char</span> *)</span>pEncode, rsaK, <span class="hljs-type">RSA_PKCS1_PADDING</span>);</span>
    string strRet;
    <span class="hljs-keyword">if</span> (ret &gt;= <span class="hljs-number">0</span>) {
        strRet = string(pEncode, ret);  
    }
    delete[] pEncode;
    <span class="hljs-type">CRYPTO_cleanup_all_ex_data</span>();
    <span class="hljs-type">BIO_free_all</span>(bp);
    <span class="hljs-type">RSA_free</span>(rsaK);
    return strRet;
}</code></pre>
<h2 id="4密文转为十进制实现">4.密文转为十进制实现</h2>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-keyword">int</span> len = <span class="hljs-built_in">strlen</span>((<span class="hljs-keyword">const</span> <span class="hljs-keyword">char</span>*)pEncode);
BIGNUM *a = BN_new();
BN_bin2bn((<span class="hljs-keyword">const</span> <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)pEncode, len,  a);
b = BN_bn2dec((<span class="hljs-keyword">const</span> BIGNUM *)a);
这里的b就是十进制字符串</code></pre>
<h1 id="5完整代码">#5.完整代码</h1>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-preprocessor">#include "stdafx.h"</span>
<span class="hljs-preprocessor">#include &lt;stdio.h&gt;  </span>
<span class="hljs-preprocessor">#include &lt;stdlib.h&gt;  </span>
<span class="hljs-preprocessor">#include &lt;windows.h&gt;</span>
<span class="hljs-preprocessor">#include &lt;openssl/rsa.h&gt;  </span>
<span class="hljs-preprocessor">#include&lt;openssl/pem.h&gt;  </span>
<span class="hljs-preprocessor">#include&lt;openssl/err.h&gt;  </span>
<span class="hljs-preprocessor">#include &lt;openssl/bio.h&gt;  </span>
<span class="hljs-preprocessor">#include &lt;fstream&gt;  </span>
<span class="hljs-preprocessor">#include &lt;iostream&gt;  </span>
<span class="hljs-preprocessor">#include &lt;string&gt;  </span>
<span class="hljs-keyword">using</span> <span class="hljs-keyword">namespace</span> <span class="hljs-built_in">std</span>;
<span class="hljs-preprocessor">#pragma comment(lib, "libeay32.lib")  </span>
<span class="hljs-preprocessor">#pragma comment(lib, "ssleay32.lib")  </span>
<span class="hljs-keyword">char</span> *b;
<span class="hljs-keyword">void</span> generateKey() {
    <span class="hljs-comment">/* 生成公钥 */</span>
    RSA* rsa = RSA_generate_key(<span class="hljs-number">1024</span>, RSA_F4, NULL, NULL);
    BIO *bp = BIO_new(BIO_s_file());
    BIO_write_filename(bp, <span class="hljs-string">"public.pem"</span>);
    PEM_write_bio_RSAPublicKey(bp, rsa);
    BIO_free_all(bp);
    <span class="hljs-comment">/* 生成私钥 */</span>
    bp = BIO_new_file(<span class="hljs-string">"private.pem"</span>, <span class="hljs-string">"w+"</span>);
    PEM_write_bio_RSAPrivateKey(bp, rsa, NULL, NULL, <span class="hljs-number">4</span>, NULL, NULL);
    BIO_free_all(bp);
    RSA_free(rsa);
}
<span class="hljs-built_in">string</span> bio_read_privateKey(<span class="hljs-built_in">string</span> data) {
    OpenSSL_add_all_algorithms();
    BIO* bp = BIO_new(BIO_s_file());
    BIO_read_filename(bp, <span class="hljs-string">"prikey.pem"</span>);
    RSA* rsaK = PEM_read_bio_RSAPrivateKey(bp, NULL, NULL, NULL);
    <span class="hljs-keyword">if</span> (NULL == rsaK) {
        perror(<span class="hljs-string">"read key file fail!"</span>);
    }
    <span class="hljs-keyword">else</span> {
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"read success!\n"</span>);
    }
    <span class="hljs-keyword">int</span> nLen = RSA_size(rsaK); 
    <span class="hljs-keyword">char</span> *pEncode = <span class="hljs-keyword">new</span> <span class="hljs-keyword">char</span>[nLen + <span class="hljs-number">1</span>];
    <span class="hljs-keyword">int</span> ret = RSA_private_decrypt(data.length(), (<span class="hljs-keyword">const</span> <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)data.c_str(), (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)pEncode, rsaK, RSA_PKCS1_PADDING);
    <span class="hljs-built_in">string</span> strRet;
    <span class="hljs-keyword">if</span> (ret &gt;= <span class="hljs-number">0</span>) {
        strRet = <span class="hljs-built_in">string</span>(pEncode, ret);  
    }
    <span class="hljs-keyword">delete</span>[] pEncode;
    CRYPTO_cleanup_all_ex_data();
    BIO_free_all(bp);
    RSA_free(rsaK);
    <span class="hljs-keyword">return</span> strRet;
}



<span class="hljs-built_in">string</span> bio_read_publicKey(<span class="hljs-built_in">string</span> data) {
    OpenSSL_add_all_algorithms();
    BIO* bp = BIO_new(BIO_s_file());
    BIO_read_filename(bp, <span class="hljs-string">"pubkey.pem"</span>);
    RSA* rsaK = PEM_read_bio_RSAPublicKey(bp, NULL, NULL, NULL);
    <span class="hljs-keyword">if</span> (NULL == rsaK) {
        perror(<span class="hljs-string">"read key file fail!"</span>);
    }
    <span class="hljs-keyword">else</span> {
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"read success!"</span>);
        <span class="hljs-keyword">int</span> nLen = RSA_size(rsaK);
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"len:%d\n"</span>, nLen);
    }
    <span class="hljs-keyword">int</span> nLen = RSA_size(rsaK);
    <span class="hljs-keyword">char</span> *pEncode = <span class="hljs-keyword">new</span> <span class="hljs-keyword">char</span>[nLen + <span class="hljs-number">1</span>];
    <span class="hljs-keyword">int</span> ret = RSA_public_encrypt(data.length(), (<span class="hljs-keyword">const</span> <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)data.c_str(),
        (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)pEncode, rsaK, RSA_PKCS1_PADDING);
    <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span> strRet;
    <span class="hljs-keyword">if</span> (ret &gt;= <span class="hljs-number">0</span>) {
        strRet = <span class="hljs-built_in">std</span>::<span class="hljs-built_in">string</span>(pEncode, ret);  
    }
    <span class="hljs-keyword">int</span> len = <span class="hljs-built_in">strlen</span>((<span class="hljs-keyword">const</span> <span class="hljs-keyword">char</span>*)pEncode);
    BIGNUM *a = BN_new();
    BN_bin2bn((<span class="hljs-keyword">const</span> <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *)pEncode, len,  a);

    b = BN_bn2dec((<span class="hljs-keyword">const</span> BIGNUM *)a);
    <span class="hljs-keyword">delete</span>[] pEncode;
    CRYPTO_cleanup_all_ex_data();
    BIO_free_all(bp);
    RSA_free(rsaK);
    <span class="hljs-keyword">return</span> strRet;
}
<span class="hljs-keyword">int</span> main() {
<span class="hljs-comment">//  generateKey();</span>

    <span class="hljs-keyword">char</span> *str = <span class="hljs-string">"我爱密码"</span>;
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"原文：%s\n"</span>, str);
    <span class="hljs-built_in">string</span> m = bio_read_publicKey(str);
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"密文：\n------------%s--------------\n\n"</span>, m.c_str());
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"十进制：\n-----------%s------------\n\n"</span>, b);
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *p =(<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> *) m.c_str();
    <span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>;
    <span class="hljs-keyword">while</span> (*(p+j) != NULL) { <span class="hljs-built_in">printf</span>(<span class="hljs-string">"%x"</span>, *(p + j)); j++; }
    <span class="hljs-built_in">cout</span> &lt;&lt; endl;
    <span class="hljs-built_in">string</span> miwen = m;
    <span class="hljs-built_in">string</span> c = bio_read_privateKey(miwen);
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"解密后：\n------------%s--------------\n\n"</span>, c.c_str());
    system(<span class="hljs-string">"pause"</span>);
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}</code></pre>
<h1 id="aes">AES</h1>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-preprocessor">#include "stdafx.h"</span>
<span class="hljs-preprocessor">#include &lt;stdio.h&gt;</span>
<span class="hljs-preprocessor">#include &lt;string.h&gt;</span>
<span class="hljs-preprocessor">#include &lt;stdlib.h&gt;</span>
<span class="hljs-preprocessor">#include &lt;openssl/aes.h&gt;</span>
<span class="hljs-preprocessor">#include &lt;iostream&gt;</span>
<span class="hljs-preprocessor">#define AES_BITS 128</span>
<span class="hljs-preprocessor">#define MSG_LEN 100000</span>
<span class="hljs-preprocessor">#pragma comment(lib, "libeay32.lib")</span>
<span class="hljs-preprocessor">#pragma comment(lib, "ssleay32.lib")  </span>


<span class="hljs-preprocessor">#pragma comment(lib, "ws2_32.lib")</span>
<span class="hljs-preprocessor">#define IP "127.0.0.1"</span>
<span class="hljs-preprocessor">#define PORT 8888</span>
<span class="hljs-keyword">using</span> <span class="hljs-keyword">namespace</span> <span class="hljs-built_in">std</span>;
<span class="hljs-keyword">int</span> main(<span class="hljs-keyword">int</span> argc, <span class="hljs-keyword">char</span> *argv[])
{
    <span class="hljs-keyword">int</span> len;
    <span class="hljs-keyword">char</span> key[AES_BLOCK_SIZE];
    <span class="hljs-built_in">memcpy</span>(key, <span class="hljs-string">"123412341234123"</span>, <span class="hljs-number">16</span>);
    <span class="hljs-built_in">cout</span> &lt;&lt; key[<span class="hljs-number">0</span>];
    system(<span class="hljs-string">"pause"</span>);
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>  iv[AES_BLOCK_SIZE];
    FILE *fp,*fp1;
    fp = fopen(<span class="hljs-string">"1.txt"</span>, <span class="hljs-string">"rb"</span>);
    fp1 = fopen(<span class="hljs-string">"2.txt"</span>,<span class="hljs-string">"ab"</span>);

    fseek(fp, <span class="hljs-number">0L</span>, SEEK_END);
    len = ftell(fp);
    fseek(fp, <span class="hljs-number">0L</span>, SEEK_SET);

    <span class="hljs-keyword">char</span> sourceStringTemp[MSG_LEN];
    <span class="hljs-keyword">char</span> dstStringTemp[MSG_LEN];
    <span class="hljs-built_in">memset</span>((<span class="hljs-keyword">char</span>*)sourceStringTemp, <span class="hljs-number">0</span>, MSG_LEN);
    <span class="hljs-built_in">memset</span>((<span class="hljs-keyword">char</span>*)dstStringTemp, <span class="hljs-number">0</span>, MSG_LEN);
    fread(sourceStringTemp, <span class="hljs-number">1</span>, len, fp);


    <span class="hljs-keyword">int</span> i;



    AES_KEY aes;
    <span class="hljs-keyword">if</span> (AES_set_encrypt_key((<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)key, <span class="hljs-number">128</span>, &amp;aes) &lt; <span class="hljs-number">0</span>)
    {
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
    }
    AES_cbc_encrypt((<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)sourceStringTemp, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)dstStringTemp, len, &amp;aes, iv, AES_ENCRYPT);
    <span class="hljs-comment">/***************下面为解密*************/</span>
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"enc %d:"</span>, <span class="hljs-built_in">strlen</span>((<span class="hljs-keyword">char</span>*)dstStringTemp));
    <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; dstStringTemp[i]; i += <span class="hljs-number">1</span>) {
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"%x"</span>, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>)dstStringTemp[i]);
    }
    <span class="hljs-built_in">memset</span>((<span class="hljs-keyword">char</span>*)sourceStringTemp, <span class="hljs-number">0</span>, MSG_LEN);

    <span class="hljs-keyword">if</span> (AES_set_decrypt_key((<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)key, <span class="hljs-number">128</span>, &amp;aes) &lt; <span class="hljs-number">0</span>)
    {
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
    }
    AES_cbc_encrypt((<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)dstStringTemp, (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>*)sourceStringTemp, len, &amp;aes, iv, AES_DECRYPT);
    fwrite(sourceStringTemp, <span class="hljs-number">1</span>, len, fp1);
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"\n"</span>);
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"dec %d:"</span>, <span class="hljs-built_in">strlen</span>((<span class="hljs-keyword">char</span>*)sourceStringTemp));
    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"%s\n"</span>, sourceStringTemp);

    <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; sourceStringTemp[i]; i += <span class="hljs-number">1</span>) {
        <span class="hljs-built_in">printf</span>(<span class="hljs-string">"%x"</span>, ( <span class="hljs-keyword">char</span>)sourceStringTemp[i]);
    }

    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"\n"</span>);
    system(<span class="hljs-string">"pause"</span>);
    fclose(fp);
    fclose(fp1);
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}



</code></pre></div>