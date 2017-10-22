---
title: 操作系统实验报告 lab8
tags: [操作系统实验]
date: 2017-06-12 01:05
---
练习0 填写已有实验将已完成的lab7和lab8进行对比  需要修改的文件如下：proc.cdefault_pmm.cpmm.cswap_fifo.cvmm.ctrap.csche.cmonitor.check_sync.c练习1 完成读文件操作的实现  首先了解打开文件的处理流程，然后参考本实验后续的文件读写操作的过程分析，编写在sfs_inode.c中sfs_io_nolo
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="练习0-填写已有实验">练习0 填写已有实验</h1>
<p>将已完成的lab7和lab8进行对比 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170707102000629?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
需要修改的文件如下：</br></img></br></p>
<pre class="prettyprint"><code class=" hljs avrasm">proc<span class="hljs-preprocessor">.c</span>
default_pmm<span class="hljs-preprocessor">.c</span>
pmm<span class="hljs-preprocessor">.c</span>
swap_fifo<span class="hljs-preprocessor">.c</span>
vmm<span class="hljs-preprocessor">.c</span>
trap<span class="hljs-preprocessor">.c</span>
sche<span class="hljs-preprocessor">.c</span>
monitor.
check_sync<span class="hljs-preprocessor">.c</span></code></pre>
<h1 id="练习1-完成读文件操作的实现">练习1 完成读文件操作的实现</h1>
<blockquote>
<p>首先了解打开文件的处理流程，然后参考本实验后续的文件读写操作的过程分析，编写在sfs_inode.c中sfs_io_nolock读文件中数据的实现代码。</p>
</blockquote>
<h2 id="0x1-ucore文件系统总体介绍">0x1 ucore文件系统总体介绍</h2>
<p>根据实验指导书，我们可以了解到，ucore的文件系统架构主要由四部分组成：</p>
<ul>
<li><strong>通用文件系统访问接口层:</strong>该层提供了一个从用户空间到文件系统的标准访问接口。这一层访问接口让应用程序能够通过一个简单的接口获得ucore内核的文件系统服务。</li>
<li><strong>文件系统抽象层:</strong>向上提供一个一致的接口给内核其他部分（文件系统相关的系统调用实现模块和其他内核功能模块）访问。向下提供一个抽象函数指针列表和数据结构来屏蔽不同文件系统的实现细节。</li>
<li><strong>Simple FS文件系统层:</strong>一个基于索引方式的简单文件系统实例。向上通过各种具体函数实现以对应文件系统抽象层提出的抽象函数。向下访问外设接口</li>
<li><strong>外设接口层:</strong>向上提供device访问接口屏蔽不同硬件细节。向下实现访问各种具体设备驱动的接口,比如disk设备接口/串口设备接口/键盘设备接口等。</li>
</ul>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170612013345716?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="0x2-ucore文件相关关键数据结构及其关系">0x2 ucore文件相关关键数据结构及其关系</h2>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170612014054434?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="0x3-重要数据结构">0x3 重要数据结构</h2>
<p>首先是file数据结构：</p>
<pre class="prettyprint"><code class=" hljs rust"><span class="hljs-keyword">struct</span> file {
<span class="hljs-keyword">enum</span> {
FD_NONE, FD_INIT, FD_OPENED, FD_CLOSED,
} status;       <span class="hljs-comment">//访问文件的执行状态</span>
<span class="hljs-keyword">bool</span> readable; <span class="hljs-comment">//文件是否可读</span>
<span class="hljs-keyword">bool</span> writable; <span class="hljs-comment">//文件是否可写</span>
<span class="hljs-keyword">int</span> fd;        <span class="hljs-comment">//文件在filemap中的索引值</span>
off_t pos;    <span class="hljs-comment">//访问文件的当前位置</span>
<span class="hljs-keyword">struct</span> inode *node;<span class="hljs-comment">//该文件对应的内存inode指针</span>
atomic_t open_count;<span class="hljs-comment">//打开此文件的次数</span>
};</code></pre>
<p>接下来inode数据结构，它是位于内存的索引节点，把不同文件系统的特定索引节点信息(甚至不能算是一个索引节点)统一封装起来，避免了进程直接访问具体文件系统</p>
<pre class="prettyprint"><code class=" hljs rust"><span class="hljs-keyword">struct</span> inode {
union { <span class="hljs-comment">//包含不同文件系统特定inode信息的union域</span>
<span class="hljs-keyword">struct</span> device <span class="hljs-number">__</span>device_info;  <span class="hljs-comment">//设备文件系统内存inode信息</span>
<span class="hljs-keyword">struct</span> sfs_inode <span class="hljs-number">__</span>sfs_inode_info; <span class="hljs-comment">//SFS文件系统内存inode信息</span>
} in_info;
<span class="hljs-keyword">enum</span> {
inode_type_device_info = <span class="hljs-number">0x1234</span>,
inode_type_sfs_inode_info,
} in_type;  <span class="hljs-comment">//此inode所属文件系统类型</span>
atomic_t ref_count;   <span class="hljs-comment">//此inode的引用计数</span>
atomic_t open_count;  <span class="hljs-comment">//打开此inode对应文件的个数</span>
<span class="hljs-keyword">struct</span> fs *in_fs;     <span class="hljs-comment">//抽象的文件系统,包含访问文件系统的函数指针</span>
<span class="hljs-keyword">const</span> <span class="hljs-keyword">struct</span> inode_ops *in_ops;   <span class="hljs-comment">//抽象的inode操作,包含访问inode的函数指针</span>
};</code></pre>
<p>内存中的索引节点</p>
<pre class="prettyprint"><code class=" hljs applescript">struct sfs_inode {
    struct sfs_disk_inode *din;                     /* <span class="hljs-function_start"><span class="hljs-keyword">on</span></span>-disk inode */
    uint32_t ino;                                   /* inode <span class="hljs-type">number</span> */
    uint32_t flags;                                 /* inode flags */
    bool dirty;                                     /* <span class="hljs-constant">true</span> <span class="hljs-keyword">if</span> inode modified */
    int reclaim_count;                              /* kill inode <span class="hljs-keyword">if</span> <span class="hljs-keyword">it</span> hits zero */
    semaphore_t sem;                                /* semaphore <span class="hljs-keyword">for</span> din */
    list_entry_t inode_link;                        /* entry <span class="hljs-keyword">for</span> linked-<span class="hljs-type">list</span> <span class="hljs-keyword">in</span> sfs_fs */
    list_entry_t hash_link;                         /* entry <span class="hljs-keyword">for</span> hash linked-<span class="hljs-type">list</span> <span class="hljs-keyword">in</span> sfs_fs */
};</code></pre>
<p>SFS中的磁盘索引节点代表了一个实际位于磁盘上的文件。首先我们看看在硬盘上的索引节点的内容：</p>
<pre class="prettyprint"><code class=" hljs fsharp"><span class="hljs-keyword">struct</span> sfs_disk_inode {
    uint32_t size;                              如果inode表示常规文件，则size是文件大小
    uint16_t <span class="hljs-class"><span class="hljs-keyword">type</span>;                                  <span class="hljs-title">inode</span>的文件类型</span>
    uint16_t nlinks;                               此inode的硬链接数
    uint32_t blocks;                              此inode的数据块数的个数
    uint32_t direct[SFS_NDIRECT];                此inode的直接数据块索引值（有SFS_NDIRECT个）
    uint32_t indirect;                            此inode的一级间接数据块索引值
};</code></pre>
<h2 id="0x4-打开文件原理">0x4 打开文件原理</h2>
<blockquote>
<p>首先假定用户进程需要打开的文件已经存在在硬盘上。以user/sfs_filetest1.c为例，首先用户进程会调用在main函数中的如下语句： <br>
  int fd1 = safe_open(“/test/testfile”, O_RDWR | O_TRUNC);</br></p>
</blockquote>
<p>①通用文件访问接口层的处理流程</p>
<blockquote>
<p>首先进入通用文件访问接口层的处理流程，即进一步调用如下用户态函数： open-&gt;sys_open-&gt;syscall，从而引起系统调用进入到内核态。到了内核态后，通过中断处理例程，会调用到sys_open内核函数，并进一步调用sysfile_open内核函数。到了这里，需要把位于用户空间的字符串”/test/testfile”拷贝到内核空间中的字符串path中，并进入到文件系统抽象层的处理流程完成进一步的打开文件操作中。</p>
</blockquote>
<p>②文件系统抽象层的处理流程</p>
<blockquote>
<p>Ⅰ、分配一个空闲的file数据结构变量file在文件系统抽象层的处理中，首先调用的是file_open函数，它要给这个即将打开的文件分配一个file数据结构的变量，这个变量其实是当前进程的打开文件数组current-&gt;fs_struct-&gt;filemap[]中的一个空闲元素（即还没用于一个打开的文件），而这个元素的索引值就是最终要返回到用户进程并赋值给变量fd1。到了这一步还仅仅是给当前用户进程分配了一个file数据结构的变量，还没有找到对应的文件索引节点。 <br>
  为此需要进一步调用vfs_open函数来找到path指出的文件所对应的基于inode数据结构的VFS索引节点node。vfs_open函数需要完成两件事情：通过vfs_lookup找到path对应文件的inode；调用vop_open函数打开文件。</br></p>
<p>Ⅱ、找到文件设备的根目录“/”的索引节点需要注意，这里的vfs_lookup函数是一个针对目录的操作函数，它会调用vop_lookup函数来找到SFS文件系统中的“/test”目录下的“testfile”文件。为此，vfs_lookup函数首先调用get_device函数，并进一步调用vfs_get_bootfs函数（其实调用了）来找到根目录“/”对应的inode。这个inode就是位于vfs.c中的inode变量bootfs_node。这个变量在init_main函数（位于kern/process/proc.c）执行时获得了赋值。</p>
<p>Ⅲ、找到根目录“/”下的“test”子目录对应的索引节点，在找到根目录对应的inode后，通过调用vop_lookup函数来查找“/”和“test”这两层目录下的文件“testfile”所对应的索引节点，如果找到就返回此索引节点。</p>
<p>Ⅳ、把file和node建立联系。完成第3步后，将返回到file_open函数中，通过执行语句“file-&gt;node=node;”，就把当前进程的current-&gt;fs_struct-&gt;filemap[fd]（即file所指变量）的成员变量node指针指向了代表“/test/testfile”文件的索引节点node。这时返回fd。经过重重回退，通过系统调用返回，用户态的syscall-&gt;sys_open-&gt;open-&gt;safe_open等用户函数的层层函数返回，最终把把fd赋值给fd1。自此完成了打开文件操作。但这里我们还没有分析第2和第3步是如何进一步调用SFS文件系统提供的函数找位于SFS文件系统上的“/test/testfile”所对应的sfs磁盘inode的过程。下面需要进一步对此进行分析。</p>
</blockquote>
<p>③SFS文件系统层的处理流程</p>
<blockquote>
<p>这里需要分析文件系统抽象层中没有彻底分析的vop_lookup函数到底做了啥。下面我们来看看。在sfs_inode.c中的sfs_node_dirops变量定义了“.vop_lookup = sfs_lookup”，所以我们重点分析sfs_lookup的实现。</p>
<p>sfs_lookup有三个参数：node，path，node_store。其中node是根目录“/”所对应的inode节点；path是文件“testfile”的绝对路径“/test/testfile”，而node_store是经过查找获得的“testfile”所对应的inode节点。</p>
<p>Sfs_lookup函数以“/”为分割符，从左至右逐一分解path获得各个子目录和最终文件对应的inode节点。在本例中是分解出“test”子目录，并调用sfs_lookup_once函数获得“test”子目录对应的inode节点subnode，然后循环进一步调用sfs_lookup_once查找以“test”子目录下的文件“testfile1”所对应的inode节点。当无法分解path后，就意味着找到了testfile1对应的inode节点，就可顺利返回了。</p>
<p>sfs_lookup_once将调用sfs_dirent_search_nolock函数来查找与路径名匹配的目录项，如果找到目录项，则根据目录项中记录的inode所处的数据块索引值找到路径名对应的SFS磁盘inode，并读入SFS磁盘inode对的内容，创建SFS内存inode。</p>
</blockquote>
<h2 id="0x5-代码填写">0x5 代码填写</h2>
<p>调用了SFS文件系统层的vfs_lookup函数去寻找node，这里在sfs_inode.c中我们能够知道.vop_lookup = sfs_lookup <br>
<code>sfs_lookup</code></br></p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> sfs_lookup(<span class="hljs-keyword">struct</span> inode *node, <span class="hljs-keyword">char</span> *path, <span class="hljs-keyword">struct</span> inode **node_store) {
<span class="hljs-keyword">struct</span> sfs_fs *sfs = fsop_info(vop_fs(node), sfs);
assert(*path != <span class="hljs-string">'\0'</span> &amp;&amp; *path != <span class="hljs-string">'/'</span>);    <span class="hljs-comment">//以“/”为分割符，从左至右逐一分解path获得各个子目录和最终文件对应的inode节点。</span>
vop_ref_inc(node);
<span class="hljs-keyword">struct</span> sfs_inode *<span class="hljs-built_in">sin</span> = vop_info(node, sfs_inode);
<span class="hljs-keyword">if</span> (<span class="hljs-built_in">sin</span>-&gt;din-&gt;type != SFS_TYPE_DIR) {
    vop_ref_dec(node);
    <span class="hljs-keyword">return</span> -E_NOTDIR;
}
<span class="hljs-keyword">struct</span> inode *subnode;
<span class="hljs-keyword">int</span> ret = sfs_lookup_once(sfs, <span class="hljs-built_in">sin</span>, path, &amp;subnode, NULL);  <span class="hljs-comment">//循环进一步调用sfs_lookup_once查找以“test”子目录下的文件“testfile1”所对应的inode节点。</span>

vop_ref_dec(node);
<span class="hljs-keyword">if</span> (ret != <span class="hljs-number">0</span>) {  
    <span class="hljs-keyword">return</span> ret;
}
*node_store = subnode;  <span class="hljs-comment">//当无法分解path后，就意味着找到了需要对应的inode节点，就可顺利返回了。</span>
<span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}</code></pre>
<p>sfs_lookup_once函数，它调用sfs_dirent_search_nolock函数来查找与路径名匹配的目录项，如果找到目录项，则根据目录项中记录的inode所处的数据块索引值找到路径名对应的SFS磁盘inode，并读入SFS磁盘inode对的内容，创建SFS内存inode。 </p>
<pre class="prettyprint"><code class=" hljs cpp"><span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> sfs_lookup_once(<span class="hljs-keyword">struct</span> sfs_fs *sfs, <span class="hljs-keyword">struct</span> sfs_inode *<span class="hljs-built_in">sin</span>, <span class="hljs-keyword">const</span> <span class="hljs-keyword">char</span> *name, <span class="hljs-keyword">struct</span> inode **node_store, <span class="hljs-keyword">int</span> *slot) {
<span class="hljs-keyword">int</span> ret;
uint32_t ino;
lock_sin(<span class="hljs-built_in">sin</span>);
{   <span class="hljs-comment">// find the NO. of disk block and logical index of file entry</span>
    ret = sfs_dirent_search_nolock(sfs, <span class="hljs-built_in">sin</span>, name, &amp;ino, slot, NULL);
}
unlock_sin(<span class="hljs-built_in">sin</span>);
<span class="hljs-keyword">if</span> (ret == <span class="hljs-number">0</span>) {
    <span class="hljs-comment">// load the content of inode with the the NO. of disk block</span>
    ret = sfs_load_inode(sfs, node_store, ino);
}
<span class="hljs-keyword">return</span> ret;
}</code></pre>
<p>接下来我们需要完成sfs_io_nolock函数中读文件的过程，代码如下，这里只将我们所需要填写的部分罗列出来了：</p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span>
sfs_io_nolock(<span class="hljs-keyword">struct</span> sfs_fs *sfs, <span class="hljs-keyword">struct</span> sfs_inode *sin, <span class="hljs-keyword">void</span> *buf, off_t offset, size_t *alenp, <span class="hljs-keyword">bool</span> write) {
......
......  

<span class="hljs-keyword">if</span> ((blkoff = offset % SFS_BLKSIZE) != <span class="hljs-number">0</span>) {                   <span class="hljs-comment">//读取第一部分的数据</span>
    size = (nblks != <span class="hljs-number">0</span>) ? (SFS_BLKSIZE - blkoff) : (endpos - offset); <span class="hljs-comment">//计算第一个数据块的大小</span>
    <span class="hljs-keyword">if</span> ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &amp;ino)) != <span class="hljs-number">0</span>) {   <span class="hljs-comment">//找到内存文件索引对应的block的编号ino</span>
        <span class="hljs-keyword">goto</span> <span class="hljs-keyword">out</span>;
    }

    <span class="hljs-keyword">if</span> ((ret = sfs_buf_op(sfs, buf, size, ino, blkoff)) != <span class="hljs-number">0</span>) {   
        <span class="hljs-keyword">goto</span> <span class="hljs-keyword">out</span>;
    }
    <span class="hljs-comment">//完成实际的读写操作</span>
    alen += size;
    <span class="hljs-keyword">if</span> (nblks == <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">goto</span> <span class="hljs-keyword">out</span>;
    }
    buf += size, blkno ++, nblks --;
}

<span class="hljs-comment">//读取中间部分的数据，将其分为size大学的块，然后一次读一块直至读完</span>
size = SFS_BLKSIZE;
<span class="hljs-keyword">while</span> (nblks != <span class="hljs-number">0</span>) {
    <span class="hljs-keyword">if</span> ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &amp;ino)) != <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">goto</span> <span class="hljs-keyword">out</span>;
    }
    <span class="hljs-keyword">if</span> ((ret = sfs_block_op(sfs, buf, ino, <span class="hljs-number">1</span>)) != <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">goto</span> <span class="hljs-keyword">out</span>;
    }
    alen += size, buf += size, blkno ++, nblks --;
}
<span class="hljs-comment">//读取第三部分的数据</span>
<span class="hljs-keyword">if</span> ((size = endpos % SFS_BLKSIZE) != <span class="hljs-number">0</span>) {
    <span class="hljs-keyword">if</span> ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &amp;ino)) != <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">goto</span> <span class="hljs-keyword">out</span>;
    }
    <span class="hljs-keyword">if</span> ((ret = sfs_buf_op(sfs, buf, size, ino, <span class="hljs-number">0</span>)) != <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">goto</span> <span class="hljs-keyword">out</span>;
    }
    alen += size;
}</code></pre>
<h1 id="练习2-完成基于文件系统的执行程序机制的实现">练习2 完成基于文件系统的执行程序机制的实现</h1>
<blockquote>
<p>改写proc.c中的load_icode函数和其他相关函数，实现基于文件系统的执行程序机制。执行：make qemu。如果能看看到sh用户程序的执行界面，则基本成功了。如果在sh用户界面上可以执行”ls”,”hello”等其他放置在sfs文件系统中的其他执行程序，则可以认为本实验基本成功。</p>
</blockquote>
<p><code>在proc.c中，根据注释我们需要先初始化fs中的进程控制结构，即在alloc_proc函数中我们需要做一下修改，加上一句proc-&gt;filesp = NULL;从而完成初始化。</code></p>
<p>然后就是要实现load_icode函数，具体的实现及注释如下所示：</p>
<pre class="prettyprint"><code class=" hljs fsharp"><span class="hljs-keyword">static</span> int
load_icode(int fd, int argc, char **kargv) {

    /* (<span class="hljs-number">1</span>) create a <span class="hljs-keyword">new</span> mm <span class="hljs-keyword">for</span> current process
     * (<span class="hljs-number">2</span>) create a <span class="hljs-keyword">new</span> PDT, <span class="hljs-keyword">and</span> mm-&gt;pgdir= kernel virtual addr <span class="hljs-keyword">of</span> PDT
     * (<span class="hljs-number">3</span>) copy TEXT/DATA/BSS parts <span class="hljs-keyword">in</span> binary <span class="hljs-keyword">to</span> memory space <span class="hljs-keyword">of</span> process
     *    (<span class="hljs-number">3.1</span>) read raw data content <span class="hljs-keyword">in</span> file <span class="hljs-keyword">and</span> resolve elfhdr
     *    (<span class="hljs-number">3.2</span>) read raw data content <span class="hljs-keyword">in</span> file <span class="hljs-keyword">and</span> resolve proghdr based on info <span class="hljs-keyword">in</span> elfhdr
     *    (<span class="hljs-number">3.3</span>) call mm_map <span class="hljs-keyword">to</span> build vma related <span class="hljs-keyword">to</span> TEXT/DATA
     *    (<span class="hljs-number">3.4</span>) callpgdir_alloc_page <span class="hljs-keyword">to</span> allocate page <span class="hljs-keyword">for</span> TEXT/DATA, read contents <span class="hljs-keyword">in</span> file
     *          <span class="hljs-keyword">and</span> copy them into the <span class="hljs-keyword">new</span> allocated pages
     *    (<span class="hljs-number">3.5</span>) callpgdir_alloc_page <span class="hljs-keyword">to</span> allocate pages <span class="hljs-keyword">for</span> BSS, memset zero <span class="hljs-keyword">in</span> these pages
     * (<span class="hljs-number">4</span>) call mm_map <span class="hljs-keyword">to</span> setup user stack, <span class="hljs-keyword">and</span> put parameters into user stack
     * (<span class="hljs-number">5</span>) setup current process's mm, cr3, reset pgidr (using lcr3 MARCO)
     * (<span class="hljs-number">6</span>) setup uargc <span class="hljs-keyword">and</span> uargv <span class="hljs-keyword">in</span> user stacks
     * (<span class="hljs-number">7</span>) setup trapframe <span class="hljs-keyword">for</span> user environment
     * (<span class="hljs-number">8</span>) <span class="hljs-keyword">if</span> up steps failed, you should cleanup the env.
     */
    <span class="hljs-keyword">assert</span>(argc &gt;= <span class="hljs-number">0</span> &amp;&amp; argc &lt;= EXEC_MAX_ARG_NUM);
    <span class="hljs-comment">//(1)建立内存管理器</span>
    <span class="hljs-keyword">if</span> (current-&gt;mm != NULL) {    <span class="hljs-comment">//要求当前内存管理器为空</span>
        panic(<span class="hljs-string">"load_icode: current-&gt;mm must be empty.\n"</span>);
    }

    int ret = -E_NO_MEM;    <span class="hljs-comment">// E_NO_MEM代表因为存储设备产生的请求错误</span>
    <span class="hljs-keyword">struct</span> mm_struct *mm;  <span class="hljs-comment">//建立内存管理器</span>
    <span class="hljs-keyword">if</span> ((mm = mm_create()) == NULL) {
        goto bad_mm;
    }

    <span class="hljs-comment">//(2)建立页目录</span>
    <span class="hljs-keyword">if</span> (setup_pgdir(mm) != <span class="hljs-number">0</span>) {
        goto bad_pgdir_cleanup_mm;
    }
    <span class="hljs-keyword">struct</span> Page *page;<span class="hljs-comment">//建立页表</span>

    <span class="hljs-comment">//(3)从文件加载程序到内存</span>
    <span class="hljs-keyword">struct</span> elfhdr __elf, *elf = &amp;__elf;
    <span class="hljs-keyword">if</span> ((ret = load_icode_read(fd, elf, sizeof(<span class="hljs-keyword">struct</span> elfhdr), <span class="hljs-number">0</span>)) != <span class="hljs-number">0</span>) {<span class="hljs-comment">//读取elf文件头</span>
        goto bad_elf_cleanup_pgdir;           
    }

    <span class="hljs-keyword">if</span> (elf-&gt;e_magic != ELF_MAGIC) {
        ret = -E_INVAL_ELF;
        goto bad_elf_cleanup_pgdir;
    }

    <span class="hljs-keyword">struct</span> proghdr __ph, *ph = &amp;__ph;
    uint32_t vm_flags, perm, phnum;
    <span class="hljs-keyword">for</span> (phnum = <span class="hljs-number">0</span>; phnum &lt; elf-&gt;e_phnum; phnum ++) {  <span class="hljs-comment">//e_phnum代表程序段入口地址数目，即多少各段</span>
        off_t phoff = elf-&gt;e_phoff + sizeof(<span class="hljs-keyword">struct</span> proghdr) * phnum;  <span class="hljs-comment">//循环读取程序的每个段的头部   </span>
        <span class="hljs-keyword">if</span> ((ret = load_icode_read(fd, ph, sizeof(<span class="hljs-keyword">struct</span> proghdr), phoff)) != <span class="hljs-number">0</span>) {
            goto bad_cleanup_mmap;
        }
        <span class="hljs-keyword">if</span> (ph-&gt;p_type != ELF_PT_LOAD) {
            continue ;
        }
        <span class="hljs-keyword">if</span> (ph-&gt;p_filesz &gt; ph-&gt;p_memsz) {
            ret = -E_INVAL_ELF;
            goto bad_cleanup_mmap;
        }
        <span class="hljs-keyword">if</span> (ph-&gt;p_filesz == <span class="hljs-number">0</span>) {
            continue ;
        }
        vm_flags = <span class="hljs-number">0</span>, perm = PTE_U;<span class="hljs-comment">//建立虚拟地址与物理地址之间的映射</span>
        <span class="hljs-keyword">if</span> (ph-&gt;p_flags &amp; ELF_PF_X) vm_flags |= VM_EXEC;
        <span class="hljs-keyword">if</span> (ph-&gt;p_flags &amp; ELF_PF_W) vm_flags |= VM_WRITE;
        <span class="hljs-keyword">if</span> (ph-&gt;p_flags &amp; ELF_PF_R) vm_flags |= VM_READ;
        <span class="hljs-keyword">if</span> (vm_flags &amp; VM_WRITE) perm |= PTE_W;
        <span class="hljs-keyword">if</span> ((ret = mm_map(mm, ph-&gt;p_va, ph-&gt;p_memsz, vm_flags, NULL)) != <span class="hljs-number">0</span>) {
            goto bad_cleanup_mmap;
        }
        off_t offset = ph-&gt;p_offset;
        size_t off, size;
        uintptr_t start = ph-&gt;p_va, <span class="hljs-keyword">end</span>, la = ROUNDDOWN(start, PGSIZE);


        ret = -E_NO_MEM;

        <span class="hljs-comment">//复制数据段和代码段</span>
        <span class="hljs-keyword">end</span> = ph-&gt;p_va + ph-&gt;p_filesz;      <span class="hljs-comment">//计算数据段和代码段终止地址</span>
        <span class="hljs-keyword">while</span> (start &lt; <span class="hljs-keyword">end</span>) {               
            <span class="hljs-keyword">if</span> ((page = pgdir_alloc_page(mm-&gt;pgdir, la, perm)) == NULL) {
                ret = -E_NO_MEM;
                goto bad_cleanup_mmap;
            }
            off = start - la, size = PGSIZE - off, la += PGSIZE;
            <span class="hljs-keyword">if</span> (<span class="hljs-keyword">end</span> &lt; la) {
                size -= la - <span class="hljs-keyword">end</span>;
            }
            <span class="hljs-comment">//每次读取size大小的块，直至全部读完</span>
            <span class="hljs-keyword">if</span> ((ret = load_icode_read(fd, page2kva(page) + off, size, offset)) != <span class="hljs-number">0</span>) {       <span class="hljs-comment">//load_icode_read通过sysfile_read函数实现文件读取</span>
                goto bad_cleanup_mmap;
            }
            start += size, offset += size;
        }
        <span class="hljs-comment">//建立BSS段</span>
        <span class="hljs-keyword">end</span> = ph-&gt;p_va + ph-&gt;p_memsz;   <span class="hljs-comment">//同样计算终止地址</span>

        <span class="hljs-keyword">if</span> (start &lt; la) {     
            <span class="hljs-keyword">if</span> (start == <span class="hljs-keyword">end</span>) {   
                continue ;
            }
            off = start + PGSIZE - la, size = PGSIZE - off;
            <span class="hljs-keyword">if</span> (<span class="hljs-keyword">end</span> &lt; la) {
                size -= la - <span class="hljs-keyword">end</span>;
            }
            memset(page2kva(page) + off, <span class="hljs-number">0</span>, size);
            start += size;
            <span class="hljs-keyword">assert</span>((<span class="hljs-keyword">end</span> &lt; la &amp;&amp; start == <span class="hljs-keyword">end</span>) || (<span class="hljs-keyword">end</span> &gt;= la &amp;&amp; start == la));
        }

        <span class="hljs-keyword">while</span> (start &lt; <span class="hljs-keyword">end</span>) {
            <span class="hljs-keyword">if</span> ((page = pgdir_alloc_page(mm-&gt;pgdir, la, perm)) == NULL) {
                ret = -E_NO_MEM;
                goto bad_cleanup_mmap;
            }
            off = start - la, size = PGSIZE - off, la += PGSIZE;
            <span class="hljs-keyword">if</span> (<span class="hljs-keyword">end</span> &lt; la) {
                size -= la - <span class="hljs-keyword">end</span>;
            }
            <span class="hljs-comment">//每次操作size大小的块</span>
            memset(page2kva(page) + off, <span class="hljs-number">0</span>, size);
            start += size;
        }
    }
    sysfile_close(fd);<span class="hljs-comment">//关闭文件，加载程序结束</span>

    <span class="hljs-comment">//(4)建立相应的虚拟内存映射表</span>
    vm_flags = VM_READ | VM_WRITE | VM_STACK;
    <span class="hljs-keyword">if</span> ((ret = mm_map(mm, USTACKTOP - USTACKSIZE, USTACKSIZE, vm_flags, NULL)) != <span class="hljs-number">0</span>) {
        goto bad_cleanup_mmap;
    }
    <span class="hljs-keyword">assert</span>(pgdir_alloc_page(mm-&gt;pgdir, USTACKTOP-PGSIZE , PTE_USER) != NULL);
    <span class="hljs-keyword">assert</span>(pgdir_alloc_page(mm-&gt;pgdir, USTACKTOP-<span class="hljs-number">2</span>*PGSIZE , PTE_USER) != NULL);
    <span class="hljs-keyword">assert</span>(pgdir_alloc_page(mm-&gt;pgdir, USTACKTOP-<span class="hljs-number">3</span>*PGSIZE , PTE_USER) != NULL);
    <span class="hljs-keyword">assert</span>(pgdir_alloc_page(mm-&gt;pgdir, USTACKTOP-<span class="hljs-number">4</span>*PGSIZE , PTE_USER) != NULL);
    <span class="hljs-comment">//(5)设置用户栈</span>
    mm_count_inc(mm);
    current-&gt;mm = mm;
    current-&gt;cr3 = PADDR(mm-&gt;pgdir);
    lcr3(PADDR(mm-&gt;pgdir));

    <span class="hljs-comment">//(6)处理用户栈中传入的参数，其中argc对应参数个数，uargv[]对应参数的具体内容的地址</span>
    uint32_t argv_size=<span class="hljs-number">0</span>, i;
    <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; i &lt; argc; i ++) {
        argv_size += strnlen(kargv[i],EXEC_MAX_ARG_LEN + <span class="hljs-number">1</span>)+<span class="hljs-number">1</span>;
    }

    uintptr_t stacktop = USTACKTOP - (argv_size/sizeof(long)+<span class="hljs-number">1</span>)*sizeof(long);
    char** uargv=(char **)(stacktop  - argc * sizeof(char *));

    argv_size = <span class="hljs-number">0</span>;
    <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; i &lt; argc; i ++) {         <span class="hljs-comment">//将所有参数取出来放置uargv</span>
        uargv[i] = strcpy((char *)(stacktop + argv_size ), kargv[i]);
        argv_size +=  strnlen(kargv[i],EXEC_MAX_ARG_LEN + <span class="hljs-number">1</span>)+<span class="hljs-number">1</span>;
    }

    stacktop = (uintptr_t)uargv - sizeof(int);   <span class="hljs-comment">//计算当前用户栈顶</span>
    *(int *)stacktop = argc;              
    <span class="hljs-comment">//(7)设置进程的中断帧   </span>
    <span class="hljs-keyword">struct</span> trapframe *tf = current-&gt;tf;     
    memset(tf, <span class="hljs-number">0</span>, sizeof(<span class="hljs-keyword">struct</span> trapframe));<span class="hljs-comment">//初始化tf，设置中断帧</span>
    tf-&gt;tf_cs = USER_CS;      
    tf-&gt;tf_ds = tf-&gt;tf_es = tf-&gt;tf_ss = USER_DS;
    tf-&gt;tf_esp = stacktop;
    tf-&gt;tf_eip = elf-&gt;e_entry;
    tf-&gt;tf_eflags = FL_IF;
    ret = <span class="hljs-number">0</span>;
    <span class="hljs-comment">//(8)错误处理部分</span>
out:
    <span class="hljs-keyword">return</span> ret;           <span class="hljs-comment">//返回</span>
bad_cleanup_mmap:
    exit_mmap(mm);
bad_elf_cleanup_pgdir:
    put_pgdir(mm);
bad_pgdir_cleanup_mm:
    mm_destroy(mm);
bad_mm:
    goto out;
}</code></pre>
<p>load_icode主要是将文件加载到内存中执行，根据注释的提示分为了一共七个步骤：</p>
<pre><code>1、建立内存管理器
2、建立页目录
3、将文件逐个段加载到内存中，这里要注意设置虚拟地址与物理地址之间的映射
4、建立相应的虚拟内存映射表
5、建立并初始化用户堆栈
6、处理用户栈中传入的参数
7、最后很关键的一步是设置用户进程的中断帧
</code></pre>
<h1 id="实验结果">实验结果</h1>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170707102143669?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h2 id="实验心得">实验心得</h2>
<p>本次实验让我重新认识了文件系统的管理，其中第二个小是一个大综合，结合了前面的物理内存，虚拟内存，进程，文件操作系统，用户栈···，做过之后感觉对以前知识的掌握更加的牢固。但同时我对于文件用户的权限管理不是太了解，接下来会进一步学习。</p></div>