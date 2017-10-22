---
title: Docker 使用总结
tags: [软件及应用配置]
date: 2017-08-20 13:12
---
docker在平时的使用中还是比较常见的，这里简单介绍一下docker以及其使用方法，希望对大家有用
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>docker在平时的使用中还是比较常见的，这里简单介绍一下docker以及其使用方法，希望对大家有用。</p>
<h1 id="0x01-初识docker">0x01 初识docker</h1>
<p>Docker 从 0.9 版本开始使用 libcontainer 替代 lxc，libcontainer 和 Linux 系统的交互图如下： <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170820095326651?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p>
<p>并且docker安装容器的操作系统位数和物理机是统一的，也就是说64位操作系统只能使用64位docker容器，docker分为image和容器，一个镜像可以生成多个容器，并且容器的环境是相同的。</p>
<h1 id="0x02-基本指令">0x02 基本指令</h1>
<p>参数： <br>
–name  指定容器的名字 <br>
–rm      容器运行完毕会自动删除 <br>
-i -t       创建一个提供交互式shell的容器。 <br>
-d         在后台运行容器，并且打印出容器的ID。 <br>
后面加的是容器刚生成是运行的程序默认是/bin/bash</br></br></br></br></br></p>
<h2 id="0x1-容器生成">0x1 容器生成</h2>
<p><code>docker run --name  weblogic -i -t centos</code> <br>
上述命令会生成一个名字为weblogic 的交互式容器，默认执行的是/bin/bash，所以会进入交互界面。如果本地没有centos的镜像会从远程下载到本地。</br></p>
<h2 id="0x2-生成容器时的其他属性">0x2 生成容器时的其他属性</h2>
<p>docker run 命令用于生成容器 ，生成容器的属性之后不能再改变</p>
<h3 id="端口映射">端口映射</h3>
<p>一般拥有web服务的容器都进行端口映射，将docker的80端口映射到主机的8000端口 <br>
<code>docker run -d -p 8000:80  foo/live /bin/bash</code></br></p>
<h3 id="环境变量">环境变量</h3>
<p>在配置mysql  docker的时候使用-e参数设置环境变量 <br>
<code>docker run --name first-mysql -p 3306:3306 -e MYSQL\_ROOT\_PASSWORD=123456 -d mysql</code></br></p>
<h2 id="0x3-打开关闭附加删除">0x3 打开/关闭/附加/删除</h2>
<p><code>docker  start  id  使上述生成的容器启动</code> <br>
<code>docker  stop id  使上述生成的容器关闭</code> <br>
<code>docker exec -it id  /bin/bash  附加正在运行的容器</code> <br>
<code>docker attach continer id/name</code> <br>
当重新启动容器时，会沿用创建容器（docker run）命令时指定的参数来运行，可能需要按回车才进入。 <br>
这时就已经相当于在容器内部了的shell操作了。如果操作过程中，退出了shell。容器也会随之停止。</br></br></br></br></br></p>
<p>exec与attach的区别是当exit容器的时候exec不停止容器，而attach会将容器停止</p>
<hr>
<p>删除时必须保证容器时关闭的 <br>
<code>docker  rm name/id 删除指定容器</code> <br>
删除所有镜像  <br>
<code>docker rmi $(docker images -q)1</code> <br>
根据格式删除所有镜像 <br>
<code>docker rm $(docker ps -qf status=exited)</code></br></br></br></br></br></p>
<h2 id="0x4-压缩导出上传">0x4 压缩/导出/上传</h2>
<p>我们用镜像生成了容器，然后我们可以将容器生成镜像，然后将镜像打成压缩包方便导入导出，同时也可以把镜像上传至dockerhub，供别人下载使用</p>
<h3 id="压缩容器">压缩(容器)</h3>
<p>我们使用容器 furious_bell，现在要将这个容器保存为一个文件 myunbuntu-export-1204.tar <br>
<code>docker export furious_bell &gt; /home/myubuntu-export-1204.tar</code></br></p>
<h3 id="导入">导入</h3>
<p><code>docker import  /home/myubuntu-export-1204.tar  alias</code></p>
<h3 id="压缩镜像">压缩（镜像）</h3>
<p>这里有个基础镜像：ubuntu:12.04，现在要将这个镜像保存为一个文件myubuntu-save-1204.tar <br>
<code>docker save 9610cfc68e8d &gt; /home/myubuntu-save-1204.tar</code> <br>
有点慢，稍微等待一下，没有任何warn信息就表示保存OK。9610cfc68e8d 是镜像ID</br></br></p>
<h3 id="导入-1">导入</h3>
<p><code>docker load /home/myubuntu-export-1204.tar  alias</code></p>
<h3 id="上传至远程仓库">上传至远程仓库</h3>
<p>首先拥有一个dockerhub账号 <br>
上传之前要登录 <br>
<code>docker login</code> <br>
然后输入用户名密码</br></br></br></p>
<p><code>docker push image name</code></p>
<p>关于docker file的编写，有时间再补上</p>
<h1 id="0x3-dockerfile-编写">0x3 Dockerfile 编写</h1>
<blockquote>
<p>Dockfile是一种被Docker程序解释的脚本，Dockerfile由一条一条的指令组成，每条指令对应Linux下面的一条命令。Docker程序将这些Dockerfile指令翻译真正的Linux命令。Dockerfile有自己书写格式和支持的命令，Docker程序解决这些命令间的依赖关系，类似于Makefile。</p>
</blockquote>
<p>首先给个实例 <br>
运行指令<code>docker build -t name .</code></br></p>
<pre class="prettyprint"><code class=" hljs vbnet"><span class="hljs-keyword">FROM</span> ubuntu:<span class="hljs-number">16.04</span> 

MAINTAINER <span class="hljs-number">4</span>t10n &lt;act01n@<span class="hljs-number">163.</span>com&gt;
ENV DEBIAN_FRONTEND noninteractive 
<span class="hljs-preprocessor">#这里添加更新源 </span>
RUN sed -i <span class="hljs-comment">'s/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list</span>

RUN apt-<span class="hljs-keyword">get</span> update -y &amp;&amp; \ 
    apt-<span class="hljs-keyword">get</span> install -y apache2 \
    vim \
    tar \
    php7<span class="hljs-number">.0</span>-fpm \
    php7<span class="hljs-number">.0</span>-mcrypt \ 
    php7<span class="hljs-number">.0</span>-mysql  \ 
    mysql-client \
    mysql-server   \
    &amp;&amp; /etc/init.d/mysql start \
    &amp;&amp; mysqladmin -uroot password root  \
    &amp;&amp; rm -rf /var/<span class="hljs-keyword">lib</span>/apt/lists/* 

WORKDIR /tmp  
<span class="hljs-preprocessor">#COPY  ./start.sh  /tmp/</span>
<span class="hljs-preprocessor">#COPY  ./init.sql  /tmp/ </span>
<span class="hljs-preprocessor">#RUN  chmod a+x start.sh </span>

<span class="hljs-preprocessor">#设置数据库 </span>
RUN <span class="hljs-keyword">set</span> -x \
    &amp;&amp; service mysql start \ 
    &amp;&amp; mysql  -e <span class="hljs-string">"CREATE DATABASE  blog  DEFAULT CHARACTER SET utf8;"</span>  -uroot  -proot \ 
    &amp;&amp;  mysql -e <span class="hljs-string">"grant select,insert on blog.* to 'admin'@'localhost' identified by 'password' "</span>  -uroot -proot  

<span class="hljs-preprocessor"># copy 源码</span>
<span class="hljs-preprocessor">#COPY  ./default /etc/nginx/sites-available/default</span>
<span class="hljs-preprocessor">#COPY ./src /usr/share/nginx/html/</span>
COPY ./<span class="hljs-number">1.</span>php /var/www/html/

<span class="hljs-preprocessor"># 设置可写权限 </span>
RUN chown -R  www-data:www-data /var/www/html/
EXPOSE <span class="hljs-number">80</span> <span class="hljs-number">3306</span> 

CMD [<span class="hljs-string">"/tmp/start.sh"</span>]</code></pre>
<h2 id="0x1-from">0x1 FROM</h2>
<p>FROM指定一个基础镜像， 一般情况下一个可用的 Dockerfile一定是 FROM 为第一个指令。</p>
<h2 id="0x2-maintainer">0x2 MAINTAINER</h2>
<p>这里是用于指定镜像制作者的信息</p>
<h2 id="0x3-run">0x3 RUN</h2>
<p>RUN命令将在当前image中执行任意合法命令并提交执行结果。命令执行提交后，就会自动执行Dockerfile中的下一个指令。 <br>
层级 RUN 指令和生成提交是符合Docker核心理念的做法。它允许像版本控制那样，在任意一个点，对image 镜像进行定制化构建。</br></p>
<h2 id="0x4-env">0x4 ENV</h2>
<p>ENV指令可以用于为docker容器设置环境变量 <br>
ENV设置的环境变量，可以使用 docker inspect命令来查看。同时还可以使用docker run –env =来修改环境变量。</br></p>
<h2 id="0x5-workdir">0x5 WORKDIR</h2>
<p>WORKDIR 用来切换工作目录的。Docker 默认的工作目录是/，只有 RUN 能执行 cd 命令切换目录，而且还只作用在当下下的 RUN，也就是说每一个 RUN 都是独立进行的。如果想让其他指令在指定的目录下执行，就得靠 WORKDIR。WORKDIR 动作的目录改变是持久的，不用每个指令前都使用一次 WORKDIR。</p>
<h2 id="0x6-copy">0x6 COPY</h2>
<p>COPY 将文件从路径<code>&lt;src&gt;</code>复制添加到容器内部路径 <code>&lt;dest&gt;</code></p>
<p><code>&lt;src&gt;</code>必须是想对于源文件夹的一个文件或目录，也可以是一个远程的url，<code>&lt;dest&gt;</code> <br>
是目标容器中的绝对路径。 <br>
所有的新文件和文件夹都会创建UID 和 GID 。事实上如果<code>&lt;src&gt;</code> 是一个远程文件URL，那么目标文件的权限将会是600。</br></br></p>
<h2 id="0x7-add">0x7 ADD</h2>
<p>ADD 将文件从路径  复制添加到容器内部路径  <br>
同COPY</br></p>
<h2 id="0x8-expose">0x8 EXPOSE</h2>
<p>EXPOSE 指令指定在docker允许时指定的端口进行转发。</p>
<h2 id="0x9-cmd">0x9 CMD</h2>
<p>Dockerfile.中只能有一个CMD指令。 如果你指定了多个，那么最后个CMD指令是生效的。 <br>
CMD指令的主要作用是提供默认的执行容器。这些默认值可以包括可执行文件，也可以省略可执行文件。 <br>
当你使用shell或exec格式时， CMD <br>
会自动执行这个命令。</br></br></br></p></hr></div>