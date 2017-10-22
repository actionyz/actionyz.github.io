---
title: Sublime Text3  配置安装及插件选择
tags: [软件及应用配置]
date: 2017-03-29 18:42
---
简述这段时间一直在配置软件用了很多时间，想写篇文档规整一下sublime的配置选择
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="0x01-简述">0x01 简述</h1>
<p>这段时间一直在配置软件用了很多时间，想写篇文档规整一下sublime的配置选择</p>
<h1 id="0x02-license">0x02 License</h1>
<pre class="prettyprint"><code class=" hljs cpp">—– BEGIN LICENSE —–
Michael Barnes
Single User License
EA7E-<span class="hljs-number">821385</span>
<span class="hljs-number">8</span>A353C41 <span class="hljs-number">872</span>A0D5C DF9B2950 AFF6F667
C458EA6D <span class="hljs-number">8</span>EA3C286 <span class="hljs-number">98</span>D1D650 <span class="hljs-number">131</span>A97AB
AA919AEC EF20E143 B361B1E7 <span class="hljs-number">4</span>C8B7F04
B085E65E <span class="hljs-number">2F</span>5F5360 <span class="hljs-number">8489</span>D422 FB8FC1AA
<span class="hljs-number">93F</span>6323C FD7F7544 <span class="hljs-number">3F</span>39C318 D95E6480
FCCC7561 <span class="hljs-number">8</span>A4A1741 <span class="hljs-number">68F</span>A4223 ADCEDE07
<span class="hljs-number">200</span>C25BE DBBC4855 C4CFB774 C5EC138C
<span class="hljs-number">0F</span>EC1CEF D9DCECEC D3A5DAD1 <span class="hljs-number">01316</span>C36
—— END LICENSE ——</code></pre>
<pre class="prettyprint"><code class=" hljs sql">—– <span class="hljs-operator"><span class="hljs-keyword">BEGIN</span> LICENSE —–
Free Communities Consultoria em Informática Ltda
Single <span class="hljs-keyword">User</span> License
EA7E-<span class="hljs-number">801302</span>
C154C122 <span class="hljs-number">4</span>EFA4415 F1AAEBCC <span class="hljs-number">315</span>F3A7D
<span class="hljs-number">2580735</span>A <span class="hljs-number">7955</span>AA57 <span class="hljs-number">850</span>ABD88 <span class="hljs-number">72</span>A1DDD8
<span class="hljs-number">8</span>D2CE060 CF980C29 <span class="hljs-number">890</span>D74F2 <span class="hljs-number">53131895</span>
<span class="hljs-number">281E324</span>E <span class="hljs-number">98</span>EA1FEF <span class="hljs-number">7</span>FF69A12 <span class="hljs-number">17</span>CA7784
<span class="hljs-number">490862</span>AF <span class="hljs-number">833E133</span>D FD22141D D8C89B94
<span class="hljs-number">4</span>C10A4D2 <span class="hljs-number">24693</span>D70 AE37C18F <span class="hljs-number">72</span>EF0BE5
<span class="hljs-number">1</span>ED60704 <span class="hljs-number">651</span>BC71F <span class="hljs-number">16</span>CA1B77 <span class="hljs-number">496</span>A0B19
<span class="hljs-number">463</span>EDFF9 <span class="hljs-number">6</span>BEB1861 CA5BAD96 <span class="hljs-number">89</span>D0118E
—— <span class="hljs-keyword">END</span> LICENSE ——
</span></code></pre>
<h1 id="0x03-初始化配置">0x03 初始化配置</h1>
<p>按住ctrl+`,调出面板输入</p>
<pre class="prettyprint"><code class=" hljs perl">
import urllib2,os; pf=<span class="hljs-string">'Package Control.sublime-package'</span>; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) <span class="hljs-keyword">if</span> <span class="hljs-keyword">not</span> os.path.<span class="hljs-keyword">exists</span>(ipp) <span class="hljs-keyword">else</span> None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); <span class="hljs-keyword">open</span>( os.path.<span class="hljs-keyword">join</span>( ipp, pf), <span class="hljs-string">'wb'</span> ).<span class="hljs-keyword">write</span>( urllib2.urlopen( <span class="hljs-string">'http://sublime.wbond.net/'</span> +pf.replace( <span class="hljs-string">' '</span>,<span class="hljs-string">'%20'</span> )).<span class="hljs-keyword">read</span>()); <span class="hljs-keyword">print</span>( <span class="hljs-string">'Please restart Sublime Text to finish installation'</span>)
</code></pre>
<h1 id="0x04-插件选择">0x04 插件选择</h1>
<p>按住Shift+ctrl+p，输入install package。下面是一些插件的名称及安装方法，需要安装过程的会详细描述，</p>
<h2 id="monokai-gray">Monokai Gray</h2>
<p>比较漂亮的主题</p>
<hr>
<h2 id="autofilename">AutoFileName</h2>
<p>自动补全路径，挺好用的</p>
<hr>
<h2 id="sublimerepl">SublimeREPL</h2>
<p>按F5可以跑python 程序 <br>
按键绑定 user填写</br></p>
<pre class="prettyprint"><code class=" hljs cs">{
            <span class="hljs-string">"keys"</span>: [<span class="hljs-string">"f5"</span>],<span class="hljs-comment">//可以自己改变</span>
            <span class="hljs-string">"caption"</span>: <span class="hljs-string">"SublimeREPL: Python - RUN current file"</span>,
            <span class="hljs-string">"command"</span>: <span class="hljs-string">"run_existing_window_command"</span>, 
            <span class="hljs-string">"args"</span>:
            {
                <span class="hljs-string">"id"</span>: <span class="hljs-string">"repl_python_run"</span>,
                <span class="hljs-string">"file"</span>: <span class="hljs-string">"config/Python/Main.sublime-menu"</span>
            }
    },</code></pre>
<hr>
<h2 id="side-bar-sidebar-separate">Side bar &amp;&amp; Sidebar Separate</h2>
<p>侧栏增强工具与背景颜色相同</p>
<hr>
<h2 id="emmet">Emmet</h2>
<blockquote>
<p>初始化文档 <br>
  HTML文档需要包含一些固定的标签，比如、、等，现在你只需要1秒钟就可以输入这些标签。比如输入“!”或“html:5”，然后按Tab键或ctrl+e： <br>
  html:5 或!：用于HTML5文档类型 <br>
  html:xt：用于XHTML过渡文档类型 <br>
  html:4s：用于HTML4严格文档类型 </br></br></br></br></p>
<p>轻松添加类、id、文本和属性 <br>
  1、连续输入元素名称和ID，Emmet会自动为你补全，比如输入p#foo： <br>
  2、连续输入类和id，比如p.bar#foo，会自动生成： <br>
  3、下面来看看如何定义HTML元素的内容和属性。你可以通过输入h1{foo}和a[href=#]，就可以自动生成如下代码：</br></br></br></p>
<p>声明一个带类的标签，只需输入div.item，就会生成</p><div class="item"></div>在过去版本中，可以省略掉div，即输入.item即可生成<div class="item"></div>现在如果只输入.item，则Emmet会根据父标签进行判定。比如在<ul> 中输入.item，就会生成 <br>
<li class="item"></li>下面是所有的隐式标签名称： <br>
  li：用于ul和ol中 <br>
  tr：用于table、tbody、thead和tfoot中 <br>
  td：用于tr中 <br>
  option：用于select和optgroup中</br></br></br></br></br></ul><p></p>
</blockquote>
<hr>
<h2 id="sublimecodeintel">SublimeCodeIntel</h2>
<p>安装各种语言的补全工具。</p>
<h3 id="javascript">javascript</h3>
<p>找到”JavaScript”代码段，将 <br>
    “codeintel_selected_catalogs”: [“jQuery”]  </br></p>
<p>改为：</p>
<p>[html] view plain copy <br>
在CODE上查看代码片派生到我的代码片</br></p>
<pre><code>"codeintel_selected_catalogs": ["JavaScript"]  
</code></pre>
<h3 id="python">python</h3>
<p>修复在 ST3 下 SublimeCodeIntel 对 Python 无法自动补全 import 语句里的模块名的问题</p>
<pre class="prettyprint"><code class=" hljs xml"><span class="hljs-pi">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="hljs-doctype">&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">plist</span> <span class="hljs-attribute">version</span>=<span class="hljs-value">"1.0"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">dict</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">key</span>&gt;</span>scope<span class="hljs-tag">&lt;/<span class="hljs-title">key</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">string</span>&gt;</span>source.python<span class="hljs-tag">&lt;/<span class="hljs-title">string</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">key</span>&gt;</span>settings<span class="hljs-tag">&lt;/<span class="hljs-title">key</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">dict</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-title">key</span>&gt;</span>cancelCompletion<span class="hljs-tag">&lt;/<span class="hljs-title">key</span>&gt;</span>
        <span class="hljs-comment">&lt;!-- !!! WARNING !!! --&gt;</span>
        <span class="hljs-comment">&lt;!-- This a modified version or the Python Package from Sublime Text 2 --&gt;</span>
        <span class="hljs-comment">&lt;!--
            WAS:
                &lt;string&gt;^(.*\b(and|or)$)|(\s*(pass|return|and|or|(class|def|import)\s*[a-zA-Z_0-9]+)$)&lt;/string&gt;
         --&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-title">string</span>&gt;</span>^(.*\b(and|or)$)|(\s*(pass|return|and|or|(class|def)\s*[a-zA-Z_0-9]+)$)<span class="hljs-tag">&lt;/<span class="hljs-title">string</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-title">dict</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">dict</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">plist</span>&gt;</span></code></pre>
<p>将上述代码 放到 …/Sublime Text 3/Packages/Python 中。</p>
<p>可能需要删除文件夹 …/Sublime Text 3/Cache 和文件 …/Sublime Text 3/Local/Session.sublime_session，并重启 Sublime Text 后才能生效。 </p>
<p>亲测有效，只适用于windows</p>
<hr>
<h2 id="alignment">Alignment</h2>
<p>等号对齐 <br>
按Ctrl+Alt+A，可以是凌乱的代码以等号为准左右对其，适合有代码洁癖的朋友。</br></p>
<hr>
<h2 id="converttoutf-8">ConvertToUTF-8</h2>
<p>sublime text本身是不支持中文编码的,所以需要通过安装插件来解决</p>
<hr>
<h2 id="goto-document">goto document</h2>
<p>这个插件能帮助我们快速查看手册。 比如我们在写php代码时， 突然忘记了某个函数怎么用了，将鼠标放在这个函数上，然后按F1，它能快速打开PHP手册中说明这个函数用法的地方</p>
<hr>
<h2 id="python-pep8-autoformat">Python PEP8 Autoformat</h2>
<p>python 代码对其非常有用</p>
<hr>
<h2 id="anaconda">Anaconda</h2>
<p>python自动补全，还带实例 <br>
一是直接关闭Anaconda的这项提示，Sublime &gt; Preferences &gt; Package Settings &gt; Anaconda &gt; Settings User 中添加如下代码：</br></p>
<p><code>{"anaconda_linting": false}</code></p>
<h1 id="0x05-按键配置">0x05 按键配置</h1>
<pre class="prettyprint"><code class=" hljs cs">[
    {
            <span class="hljs-string">"keys"</span>: [<span class="hljs-string">"f5"</span>],<span class="hljs-comment">//可以自己改变</span>
            <span class="hljs-string">"caption"</span>: <span class="hljs-string">"SublimeREPL: Python - RUN current file"</span>,
            <span class="hljs-string">"command"</span>: <span class="hljs-string">"run_existing_window_command"</span>, 
            <span class="hljs-string">"args"</span>:
            {
                <span class="hljs-string">"id"</span>: <span class="hljs-string">"repl_python_run"</span>,
                <span class="hljs-string">"file"</span>: <span class="hljs-string">"config/Python/Main.sublime-menu"</span>
            }
    },
   {
    <span class="hljs-string">"keys"</span>: [<span class="hljs-string">"f1"</span>],
    <span class="hljs-string">"command"</span>: <span class="hljs-string">"side_bar_files_open_with"</span>,
    <span class="hljs-string">"args"</span>: {
        <span class="hljs-string">"paths"</span>: [],
        <span class="hljs-string">"application"</span>: <span class="hljs-string">"D:\\Firefox\\firefox.exe"</span>,
        <span class="hljs-string">"extensions"</span>: <span class="hljs-string">".*"</span>
    }
    },
    { <span class="hljs-string">"keys"</span>: [<span class="hljs-string">"shift+ctrl+a"</span>], <span class="hljs-string">"command"</span>: <span class="hljs-string">"alignment"</span> },

]</code></pre></hr></hr></hr></hr></hr></hr></hr></hr></hr></hr></div>