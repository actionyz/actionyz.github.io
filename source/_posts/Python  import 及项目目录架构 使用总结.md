---
title: Python  import 及项目目录架构 使用总结
tags: [python]
date: 2017-03-31 20:24
---
总结了python的使用方法
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p>假设文件结构是</p>
<pre class="prettyprint"><code class="language-css hljs "><span class="hljs-tag">parent</span>/
        <span class="hljs-tag">yz</span>/&lt;<span class="hljs-tag">br</span>&gt;
                　__<span class="hljs-tag">init__</span><span class="hljs-class">.py</span>
                  <span class="hljs-tag">test2</span><span class="hljs-class">.py</span>
        <span class="hljs-tag">test1</span><span class="hljs-class">.py</span>
        <span class="hljs-tag">test2</span><span class="hljs-class">.py</span></code></pre>
<p>/test2.py</p>
<pre class="prettyprint"><code class="language-python hljs ">i = <span class="hljs-number">2</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">function</span><span class="hljs-params">()</span>:</span>
    <span class="hljs-keyword">print</span> <span class="hljs-number">123</span>
    a = <span class="hljs-string">'hello'</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ClassName</span><span class="hljs-params">(object)</span>:</span>
    <span class="hljs-string">"""docstring for ClassName"""</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span><span class="hljs-params">(self, arg)</span>:</span>
        super(ClassName, self).__init__()
        self.arg = arg
</code></pre>
<p>yz/test2.py</p>
<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-comment"># -*- coding:utf8 -*- </span>
<span class="hljs-string">'''wer'''</span>
a = <span class="hljs-number">37</span>                    
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">foo</span><span class="hljs-params">()</span>:</span>
    <span class="hljs-keyword">print</span> <span class="hljs-string">"I'm foo"</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">bar</span>:</span>               
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">grok</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">print</span> <span class="hljs-string">"I'm bar.grok"</span>
b = bar()               </code></pre>
<h2 id="import-同一文件夹下的文件">import 同一文件夹下的文件</h2>
<p>test1 引用 test2</p>
<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-keyword">import</span> test2
<span class="hljs-keyword">print</span> test2.i
test2.function()
b = test2.bar()</code></pre>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> test2 <span class="hljs-keyword">import</span> i
<span class="hljs-keyword">print</span> i</code></pre>
<p>import 文件名 <br>
仅仅是把文件名引到了python的命名空间 <br>
利用.寻找其中的值</br></br></p>
<h2 id="import-不同文件夹下的文件">import 不同文件夹下的文件</h2>
<p>test1 引用 yz/test2 <br>
注意这时必须在/yz文件中设置<strong>init</strong>.py(才能把文件夹识别为目录)</br></p>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> yz.test2 <span class="hljs-keyword">import</span> a
<span class="hljs-keyword">print</span> a</code></pre>
<pre class="prettyprint"><code class=" hljs python"><span class="hljs-keyword">from</span> yz <span class="hljs-keyword">import</span> test2
<span class="hljs-keyword">print</span> test2.a</code></pre>
<h2 id="initpy的作用">__init__.py的作用</h2>
<p>　　偶尔可以看到有些人写的包下面还会有一个__init__.py，它的作用是在导入包时首先执行的。</p>
<h2 id="if-name-main">if __name__ == “__main__”</h2>
<pre class="prettyprint"><code class=" hljs rsl">　　也有时候会看到 .<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span> 语句，它的作用就是当此文件没有被作为导入的文件使用时执行 <span class="hljs-keyword">if</span> 语句块里的程序。

　　假如 <span class="hljs-built_in">exp</span>.py 中加入了 **<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>** ，然后 **python3 <span class="hljs-built_in">exp</span>.py**，就会执行这个语句块里的内容

　　而 如果** <span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"exp"</span>**，时则是被 其他文件 以** <span class="hljs-string">"import exp"</span>**导入时执行的部分

　　有如果是** <span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"one.exp"</span>**，时则是被 其他文件 以** <span class="hljs-string">"import one.exp"</span>**导入时执行的部分

　　注意 在  <span class="hljs-string">"import exp"</span>时是不会执行 <span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"one.exp"</span>中的内容的！同样： <span class="hljs-string">"import one.exp“是不会执行 if __name__ == "</span><span class="hljs-built_in">exp</span><span class="hljs-string">"中的内容的</span></code></pre>
<h2 id="项目目录架构">项目目录架构</h2>
<pre class="prettyprint"><code class=" hljs 1c">假设你的项目名为foo, 我比较建议的最方便快捷目录结构这样就足够了:

Foo/
<span class="hljs-string">|-- bin/</span>
<span class="hljs-string">|   |-- foo</span>
<span class="hljs-string">|</span>
<span class="hljs-string">|-- foo/</span>
<span class="hljs-string">|   |-- tests/</span>
<span class="hljs-string">|   |   |-- __init__.py</span>
<span class="hljs-string">|   |   |-- test_main.py</span>
<span class="hljs-string">|   |</span>
<span class="hljs-string">|   |-- __init__.py</span>
<span class="hljs-string">|   |-- main.py</span>
<span class="hljs-string">|</span>
<span class="hljs-string">|-- docs/</span>
<span class="hljs-string">|   |-- conf.py</span>
<span class="hljs-string">|   |-- abc.rst</span>
<span class="hljs-string">|</span>
<span class="hljs-string">|-- setup.py</span>
<span class="hljs-string">|-- requirements.txt</span>
<span class="hljs-string">|-- README</span>
</code></pre>
<p>简要解释一下:</p>
<pre class="prettyprint"><code class=" hljs avrasm">
    bin/: 存放项目的一些可执行文件，当然你可以起名script/之类的也行。
    foo/: 存放项目的所有源代码。
        (<span class="hljs-number">1</span>) 源代码中的所有模块、包都应该放在此目录。不要置于顶层目录。
        (<span class="hljs-number">2</span>) 其子目录tests/存放单元测试代码；
        (<span class="hljs-number">3</span>) 程序的入口最好命名为main<span class="hljs-preprocessor">.py</span>。
    docs/: 存放一些文档。
    setup<span class="hljs-preprocessor">.py</span>: 安装、部署、打包的脚本。
    requirements<span class="hljs-preprocessor">.txt</span>: 存放软件依赖的外部Python包列表。
    README: 项目说明文件。</code></pre></div>