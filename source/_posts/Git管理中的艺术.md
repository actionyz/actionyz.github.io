---
title: Git管理中的艺术
tags: [软件及应用配置]
date: 2017-05-07 09:04
---
在日常的生活中，或多或少的会接触到Git，这个开源代码系统给我们提供了很大的方便。下面我就Git简介、创建、使用三个方面来给大家讲解一下。自己会的东西也不多，也是一个学习记录的过程。
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p><div class="toc"><div class="toc">
<ul>
<li><a href="#0x00-git简介">0x00 Git简介</a></li>
<li><a href="#0x01-建立并配置git">0x01 建立并配置Git</a><ul>
<li><a href="#0x0-检查ssh">0x0 检查SSH</a><ul>
<li><a href="#第一步">第一步</a></li>
<li><a href="#第二步">第二步</a></li>
<li><a href="#第三步">第三步</a></li>
</ul>
</li>
<li><a href="#0x1-拥有自己的github账号">0x1 拥有自己的Github账号</a></li>
<li><a href="#0x2-建立仓库仓库名字可以不和github上的一样">0x2 建立仓库仓库名字可以不和Github上的一样</a></li>
<li><a href="#0x3-配置本地信息">0x3 配置本地信息</a></li>
</ul>
</li>
<li><a href="#0x02-git使用">0x02 Git使用</a><ul>
<li><a href="#0x0-git命令简介">0x0 Git命令简介</a></li>
<li><a href="#0x1-图解git分支">0x1 图解Git分支</a></li>
<li><a href="#0x2-命令详解">0x2 命令详解</a><ul>
<li><a href="#diff">DIff</a></li>
<li><a href="#commit">Commit</a></li>
<li><a href="#reset">Reset</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>
</p>
<blockquote>
<p>在日常的生活中，或多或少的会接触到Git，这个开源代码系统给我们提供了很大的方便。下面我就Git简介、创建、使用三个方面来给大家讲解一下。自己会的东西也不多，也是一个学习记录的过程。</p>
</blockquote>
<h1 id="0x00-git简介">0x00 Git简介</h1>
<blockquote>
<p>Git 是一个快速、可扩展的分布式版本控制系统，它具有极为丰富的命令集，对内部系统提供了高级操作和完全访问.Git与你熟悉的大部分版本控制系统的差别是很大的。也许你熟悉Subversion、CVS、Perforce、Mercurial 等等，他们使用“增量文件系统” （Delta Storage systems）, 就是说它们存储每次提交(commit)之间的差异。Git正好与之相反，它会把你的每次提交的文件的全部内容（snapshot）都会记录下来。 <br>
    理论上，Git 可以保存任何文档，但是最善于保存文本文档，因为它本来就是为解决软件源代码（也是一种文本文档）版本管理问题而开发的，提供了许多有助于文本分析的工具。对于非文本文档，Git 只是简单地为其进行备份并实施版本管理。</br></p>
</blockquote>
<h1 id="0x01-建立并配置git">0x01 建立并配置Git</h1>
<h2 id="0x0-检查ssh">0x0 检查SSH</h2>
<p>因为GitHub会用到SSH，因此需要在shell里检查是否可以连接到GitHub： <br>
<code>ssh -T git@github.com</code> <br>
如果看到：</br></br></p>
<pre class="prettyprint"><code class=" hljs applescript">Warning: Permanently added ‘github.com,<span class="hljs-number">204.232</span><span class="hljs-number">.175</span><span class="hljs-number">.90</span>’ (RSA) <span class="hljs-keyword">to</span> <span class="hljs-keyword">the</span> <span class="hljs-type">list</span> <span class="hljs-keyword">of</span> known hosts. 
 Permission denied (publickey).</code></pre>
<p>则说明可以连接。</p>
<hr>
<p>否则需要安装SSH </p>
<h3 id="第一步">第一步</h3>
<p>执行<code>ssh-keygen -t rsa -C "你自己的github对应的邮箱地址"</code>语句</p>
<blockquote>
<p>注1：“”是需要的！  <br>
  注2：是在ssh目录下进行的！</br></p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507092540759?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
发现，id_rsa（私钥）和id_rsa.pub（公钥）这两个文件被创建了  <br>
（通过ls查看～/.ssh下面的所有内容查看）</br></br></img></p>
<h3 id="第二步">第二步</h3>
<p>将刚刚创建的ssh keys添加到github中  <br>
（1）利用gedit/cat命令，查看id_rsa.pub的内容  <br>
（2）在GitHub中，依次点击<code>Settings -&gt; SSH Keys -&gt; Add SSH Key</code>，将id_rsa.pub文件中的字符串复制进去，注意字符串中没有换行和空格。</br></br></p>
<h3 id="第三步">第三步</h3>
<p>再次检查SSH连接情况（在～/.ssh目录下）： <br>
输入如下命令： <br>
<code>ssh -T git@github.com</code> <br>
如果看到如下所示，则表示添加成功： <br>
<code>Hi alioth310! You’ve successfully authenticated, but GitHub does not provide shell access.</code></br></br></br></br></p>
<p>至此本地仓库与远程仓库的连接创建完成。</p>
<h2 id="0x1-拥有自己的github账号">0x1 拥有自己的Github账号</h2>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507085624280?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
一个全新的ubunt系统，需要安装Git（系统是不具有该工具的），方法如下：  <br>
在terminel中输入如下命令： <br>
<code>sudo apt-get install git git-core git-gui git-doc git-svn git-cvs gitweb gitk git-email git-daemon-run git-el git-arch11</code></br></br></br></img></p>
<h2 id="0x2-建立仓库仓库名字可以不和github上的一样">0x2 建立仓库（仓库名字可以不和Github上的一样）</h2>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507085724365?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507090941002?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></img></p>
<h2 id="0x3-配置本地信息">0x3 配置本地信息</h2>
<p>下图是Github上建立完仓库之后给的信息 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507091900717?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>下面我们按照他的信息提示一步一步建立</p>
<pre class="prettyprint"><code class=" hljs ruby">yz<span class="hljs-variable">@yz</span>-<span class="hljs-constant">ThinkPad</span>-<span class="hljs-constant">T430s</span><span class="hljs-symbol">:~/Git/test</span><span class="hljs-variable">$ </span>echo <span class="hljs-number">123</span> &gt; readme.md

yz<span class="hljs-variable">@yz</span>-<span class="hljs-constant">ThinkPad</span>-<span class="hljs-constant">T430s</span><span class="hljs-symbol">:~/Git/test</span><span class="hljs-variable">$ </span>git add .

yz<span class="hljs-variable">@yz</span>-<span class="hljs-constant">ThinkPad</span>-<span class="hljs-constant">T430s</span><span class="hljs-symbol">:~/Git/test</span><span class="hljs-variable">$ </span>git commit -m<span class="hljs-string">"first"</span>

yz<span class="hljs-variable">@yz</span>-<span class="hljs-constant">ThinkPad</span>-<span class="hljs-constant">T430s</span><span class="hljs-symbol">:~/Git/test</span><span class="hljs-variable">$ </span>git remote add origin git<span class="hljs-variable">@github</span>.<span class="hljs-symbol">com:</span>actionyz/just_test.git


yz<span class="hljs-variable">@yz</span>-<span class="hljs-constant">ThinkPad</span>-<span class="hljs-constant">T430s</span><span class="hljs-symbol">:~/Git/test</span><span class="hljs-variable">$ </span>git push --set-upstream origin master
</code></pre>
<h1 id="0x02-git使用">0x02 Git使用</h1>
<h2 id="0x0-git命令简介">0x0 Git命令简介</h2>
<p></p><center> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507093617103?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507093824150?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
上图很直观详细的说明了git命令之间的关系 <br>
Git在本地 主要分为工作区、stage、master三个阶段</br></br></img></br></img></br></center><p></p>
<h2 id="0x1-图解git分支">0x1 图解Git分支</h2>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507094332278?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
上面的四条命令在工作目录、暂存目录(也叫做索引)和仓库之间复制文件。</br></img></p>
<ul>
<li>git add files 把当前文件放入暂存区域。 </li>
<li>git commit 给暂存区域生成快照并提交。 </li>
<li>git reset –files 用来撤销最后一次git add files，你也可以用</li>
<li>git reset     撤销所有暂存区域文件。 </li>
<li>git checkout – files 把文件从暂存区域复制到工作目录，用来丢弃本地修改。</li>
</ul>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507094621812?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<ul>
<li>git commit -a 相当于运行 git add 把所有当前目录下的文件加入暂存区域再运行。git commit. </li>
<li>git commit files 进行一次包含最后一次提交加上工作目录中文件快照的提交。并且文件被添加到暂存区域。</li>
<li>git checkout HEAD – files 回滚到复制最后一次提交</li>
</ul>
<hr>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507094742000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>绿色的5位字符表示提交的ID，分别指向父节点。分支用橘色显示，分别指向特定的提交。当前分支由附在其上的HEAD标识。 这张图片里显示最后5次提交，ed489是最新提交。 master分支指向此次提交，另一个maint分支指向祖父提交节点。</p>
</blockquote>
<h2 id="0x2-命令详解">0x2 命令详解</h2>
<h3 id="diff">DIff</h3>
<p>有许多种方法查看两次提交之间的变动。下面是一些示例。 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507094942400?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h3 id="commit">Commit</h3>
<blockquote>
<p>提交时，git用暂存区域的文件创建一个新的提交，并把此时的节点设为父节点。然后把当前分支指向新的提交节点。下图中，当前分支是master。 在运行命令之前，master指向ed489，提交后，master指向新的节点f0cec并以ed489作为父节点。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507095039667?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>即便当前分支是某次提交的祖父节点，git会同样操作。下图中，在master分支的祖父节点maint分支进行一次提交，生成了1800b。 这样，maint分支就不再是master分支的祖父节点。此时，合并 (或者 衍合) 是必须的。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507095328752?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>如果想更改一次提交，使用 git commit –amend。git会使用与当前提交相同的父节点进行一次新提交，旧的提交会被取消。 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507095443377?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<h3 id="reset">Reset</h3>
<blockquote>
<p>reset命令把当前分支指向另一个位置，并且有选择的变动工作目录和索引。也用来在从历史仓库中复制文件到索引，而不动工作目录。 <br>
  如果不给选项，那么当前分支指向到那个提交。如果用–hard选项，那么工作目录也更新，如果用–soft选项，那么都不变。</br></p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507100215273?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>如果没有给出提交点的版本号，那么默认用HEAD。这样，分支指向不变，但是索引会回滚到最后一次提交，如果用–hard选项，工作目录也同样。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507100300731?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>如果给了文件名(或者 -p选项), 那么工作效果和带文件名的checkout差不多，除了索引被更新。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170507100336476?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>紧接着强制推送到远程分支： <br>
  git push -f</br></p>
</blockquote>
<p>指令先写这么多如果以后用到在继续更新</p></hr></hr></div>