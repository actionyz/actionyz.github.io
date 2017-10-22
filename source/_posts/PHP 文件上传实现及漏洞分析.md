---
title: PHP 文件上传实现及漏洞分析
tags: [PHP]
date: 2016-12-21 21:31
---
PHP文件上传功能直接上代码    文件信息    上传文件: <input
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="php文件上传功能">PHP文件上传功能</h1>
<p>直接上代码</p>
<pre class="prettyprint"><code class=" hljs handlebars"><span class="xml"><span class="hljs-doctype">&lt;!DOCTYPE html&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">head</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">title</span>&gt;</span>文件信息<span class="hljs-tag">&lt;/<span class="hljs-title">title</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">meta</span> <span class="hljs-attribute">charset</span>=<span class="hljs-value">"utf-8"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">body</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">form</span> <span class="hljs-attribute">action</span>=<span class="hljs-value">""</span> <span class="hljs-attribute">enctype</span>=<span class="hljs-value">"multipart/form-data"</span> <span class="hljs-attribute">method</span>=<span class="hljs-value">"POST"</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"uploadfile"</span>&gt;</span>
    上传文件: <span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">type</span>=<span class="hljs-value">"file"</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"upfile"</span> /&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">type</span>=<span class="hljs-value">"submit"</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"http://dearch.blog.51cto.com/10423918/上传"</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"submit"</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">form</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">html</span>&gt;</span>
<span class="hljs-comment">&lt;!-- 完全没有过滤，任意文件上传 --&gt;</span>
<span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'submit'</span>])) {
    var_dump(<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>]);
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件名："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'name'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件大小："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'size'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件类型："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'type'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"临时路径："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'tmp_name'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"上传后系统返回值："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'error'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"====================保存分各线========================&lt;br /&gt;"</span>;
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'error'</span>] == <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">if</span> (!is_dir(<span class="hljs-string">"./upload"</span>)) {
            mkdir(<span class="hljs-string">"./upload"</span>);
        }
        <span class="hljs-variable">$dir</span> = <span class="hljs-string">"./upload/"</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'name'</span>];
        move_uploaded_file(<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'tmp_name'</span>],<span class="hljs-variable">$dir</span>);
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件保存路径："</span>.<span class="hljs-variable">$dir</span>.<span class="hljs-string">"&lt;br /&gt;"</span>;
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"上传成功...&lt;br /&gt;"</span>;
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"图片预览：&lt;br /&gt;"</span>;

    }
}
 <span class="hljs-preprocessor">?&gt;</span></span></span></code></pre>
<p>实现了任意文件上传的功能，可以用挂马的方法去读取服务器文件</p>
<h1 id="漏洞防范一">漏洞防范一</h1>
<p>漏洞形成原因没有对文件进行过滤 <br>
于是有对文件名的过滤</br></p>
<pre class="prettyprint"><code class=" hljs handlebars"><span class="xml"><span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'submit'</span>])) {
    var_dump(<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>]);
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件名："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'name'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件大小："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'size'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件类型："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'type'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"临时路径："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'tmp_name'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"上传后系统返回值："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'error'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"====================保存分各线========================&lt;br /&gt;"</span>;
    <span class="hljs-variable">$flag</span> = <span class="hljs-number">0</span>;
    <span class="hljs-keyword">switch</span> (<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'type'</span>]) {
        <span class="hljs-keyword">case</span> <span class="hljs-string">'image/jpeg'</span>:
            <span class="hljs-variable">$flag</span> = <span class="hljs-number">1</span>;
            <span class="hljs-keyword">break</span>;
        <span class="hljs-keyword">default</span>:
            <span class="hljs-keyword">die</span>(<span class="hljs-string">"文件类型错误....."</span>);
            <span class="hljs-keyword">break</span>;
    }
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'error'</span>] == <span class="hljs-number">0</span> &amp;&amp; <span class="hljs-variable">$flag</span> ) {
        <span class="hljs-keyword">if</span> (!is_dir(<span class="hljs-string">"./upload"</span>)) {
            mkdir(<span class="hljs-string">"./upload"</span>);
        }
    <span class="hljs-variable">$dir</span> = <span class="hljs-string">"./upload/"</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'name'</span>];
    move_uploaded_file(<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'tmp_name'</span>],<span class="hljs-variable">$dir</span>);
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件保存路径："</span>.<span class="hljs-variable">$dir</span>.<span class="hljs-string">"&lt;br /&gt;"</span>;
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"上传成功...&lt;br /&gt;"</span>;
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"图片预览：&lt;br /&gt;"</span>;
    }
}
 <span class="hljs-preprocessor">?&gt;</span></span></span></code></pre>
<p>这种可以利用%00截断攻击，也可以直接将名字变成.php上传</p>
<blockquote>
<p>尽管我们知道我们上传的是一个PHP文件，但是如果不进行%00截断，我们上传的文件在服务器上是以&lt; xxx.php.jpg&gt;格式保存也就是说这是一个图片文件，PHP是不会解析这个文件。当我们进行%00截断后，服务器就会将%00后的&lt;.jpg&gt;进行截断，这是我们的的文件将以&lt; xxx.php&gt;  的形式保存在服务器上，我们的一句话木马也就成功的时上传成功了。</p>
</blockquote>
<h1 id="漏洞防范二">漏洞防范二</h1>
<pre class="prettyprint"><code class=" hljs xml"><span class="hljs-doctype">&lt;!DOCTYPE html&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">head</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">title</span>&gt;</span>文件信息<span class="hljs-tag">&lt;/<span class="hljs-title">title</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">meta</span> <span class="hljs-attribute">charset</span>=<span class="hljs-value">"utf-8"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">body</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-title">form</span> <span class="hljs-attribute">action</span>=<span class="hljs-value">""</span> <span class="hljs-attribute">enctype</span>=<span class="hljs-value">"multipart/form-data"</span> <span class="hljs-attribute">method</span>=<span class="hljs-value">"POST"</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"uploadfile"</span>&gt;</span>
    上传文件: <span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">type</span>=<span class="hljs-value">"file"</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"upfile"</span> /&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">input</span> <span class="hljs-attribute">type</span>=<span class="hljs-value">"submit"</span> <span class="hljs-attribute">value</span>=<span class="hljs-value">"http://dearch.blog.51cto.com/10423918/上传"</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"submit"</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">form</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-title">html</span>&gt;</span>
<span class="php"><span class="hljs-preprocessor">&lt;?php</span>
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">isset</span>(<span class="hljs-variable">$_POST</span>[<span class="hljs-string">'submit'</span>])) {
    var_dump(<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>]);
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件名："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'name'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件大小："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'size'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件类型："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'type'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"临时路径："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'tmp_name'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"上传后系统返回值："</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'error'</span>].<span class="hljs-string">"&lt;br /&gt;"</span>;
    <span class="hljs-keyword">echo</span> <span class="hljs-string">"====================保存分各线========================&lt;br /&gt;"</span>;
    <span class="hljs-variable">$flag</span> = <span class="hljs-number">0</span>;
    <span class="hljs-variable">$path_parts</span> = pathinfo(<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'name'</span>]);
    <span class="hljs-keyword">echo</span> <span class="hljs-string">'---&lt;br&gt;'</span>;
    var_dump(<span class="hljs-variable">$path_parts</span>);    <span class="hljs-comment">//返回文件路径信息</span>
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$path_parts</span>[<span class="hljs-string">'extension'</span>] == <span class="hljs-string">'jpg'</span> &amp;&amp; <span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'type'</span>] == <span class="hljs-string">'image/jpeg'</span>) {
        <span class="hljs-variable">$flag</span> = <span class="hljs-number">1</span>;
    }<span class="hljs-keyword">else</span>{
        <span class="hljs-keyword">die</span>(<span class="hljs-string">"文件类型错误...."</span>);
    }
    <span class="hljs-keyword">if</span> (<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'error'</span>] == <span class="hljs-number">0</span> &amp;&amp; <span class="hljs-variable">$flag</span> ) {
        <span class="hljs-keyword">if</span> (!is_dir(<span class="hljs-string">"./upload"</span>)) {
            mkdir(<span class="hljs-string">"./upload"</span>);
        }
        <span class="hljs-variable">$dir</span> = <span class="hljs-string">"./upload/"</span>.<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'name'</span>];
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"文件保存路径："</span>.<span class="hljs-variable">$dir</span>.<span class="hljs-string">"&lt;br /&gt;"</span>;
        move_uploaded_file(<span class="hljs-variable">$_FILES</span>[<span class="hljs-string">'upfile'</span>][<span class="hljs-string">'tmp_name'</span>],<span class="hljs-variable">$dir</span>);
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"上传成功...&lt;br /&gt;"</span>;
        <span class="hljs-keyword">echo</span> <span class="hljs-string">"图片预览：&lt;br /&gt;"</span>;
    }
}
 <span class="hljs-preprocessor">?&gt;</span></span></code></pre>
<p>如果对文件名进行过滤就可以达到防任意上传的效果</p>
<blockquote>
<p>为什么这次不能进行绕过？我们对文件名进行截断后，当数据包到Apache的时候，Apache会对截断处理这时截断的文件 名变为&lt; xxx.php&gt;当PHP判断时会发现文件的后缀为&lt;  php&gt;，然后我们就上传失败了….</p>
</blockquote>
<p>还是多实现，理解的比较清楚~~</p></div>