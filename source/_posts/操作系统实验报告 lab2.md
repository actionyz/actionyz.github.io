---
title: 操作系统实验报告 lab2
tags: [操作系统实验]
date: 2017-04-03 09:38
---
操作系统lab2
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h1 id="练习0填写已有实验">练习0：填写已有实验</h1>
<blockquote>
<p>本实验依赖实验1。请把要做的实验1的代码填入本实验中代码有lab1的注释部分。</p>
</blockquote>
<p>直接利用ubuntu的开源工具meld进行比较 <br>
下面是比较的图片</br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170403093622601?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170403093731805?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>由meld比较可知，两个文件需要补全，直接复制即可。</p>
<h1 id="练习1实现first-fit连续物理内存分配算法">练习1：实现first-fit连续物理内存分配算法</h1>
<p>首先了解一下什么是first-fir内存分配算法，该算法是最先适应算法。</p>
<blockquote>
<p>该算法从空闲分区链首开始查找，直至找到一个能满足其大小要求的空闲分区为止。然后再按照作业的大小，从该分区中划出一块内存分配给请求者，余下的空闲分区仍留在空闲分区链中。 <br>
  特点： 该算法倾向于使用内存中低地址部分的空闲区，在高地址部分的空闲区很少被利用，从而保留了高地址部分的大空闲区。显然为以后到达的大作业分配大的内存空间创造了条件。 <br>
  缺点：低地址部分不断被划分，留下许多难以利用、很小的空闲区，而每次查找又都从低地址部分开始，会增加查找的开销。</br></br></p>
</blockquote>
<p>物理页结构属性如下</p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">struct</span> Page {
    <span class="hljs-keyword">int</span> <span class="hljs-keyword">ref</span>;                        <span class="hljs-comment">// page frame's reference counter</span>
    uint32_t flags;                 <span class="hljs-comment">// array of flags that describe the status of the page frame</span>
    unsigned <span class="hljs-keyword">int</span> property;          <span class="hljs-comment">// the num of free block, used in first fit pm manager</span>
    list_entry_t page_link;         <span class="hljs-comment">// free list link</span>
};</code></pre>
<p>对结构体中的变量进行解释：</p>
<ol>
<li>ref <br>
表示这样页被页表的引用记数，应该就是映射此物理页的虚拟页个数。一旦某页表中有一个页表项设置了虚拟页到这个Page管理的物理页的映射关系，就会把Page的ref加一。反之，若是解除，那就减一。</br></li>
<li>flags <br>
表示此物理页的状态标记，有两个标志位，第一个表示是否被保留，如果被保留了则设为1（比如内核代码占用的空间）。第二个表示此页是否是free的。如果设置为1，表示这页是free的，可以被分配；如果设置为0，表示这页已经被分配出去了，不能被再二次分配。</br></li>
<li>property <br>
用来记录某连续内存空闲块的大小，这里需要注意的是用到此成员变量的这个Page一定是连续内存块的开始地址（第一页的地址）。</br></li>
<li>page_link <br>
是便于把多个连续内存空闲块链接在一起的双向链表指针，连续内存空闲块利用这个页的成员变量page_link来链接比它地址小和大的其他连续内存空闲块</br></li>
</ol>
<p>双向链表结构如下</p>
<pre class="prettyprint"><code class=" hljs objectivec"><span class="hljs-keyword">typedef</span> <span class="hljs-keyword">struct</span> {
    list_entry_t free_list;         <span class="hljs-comment">// the list header</span>
    <span class="hljs-keyword">unsigned</span> <span class="hljs-keyword">int</span> nr_free;           <span class="hljs-comment">// # of free pages in this free list</span>
} free_area_t;</code></pre>
<p>物理内存页管理器顺着双向链表进行搜索空闲内存区域，直到找到一个足够大的空闲区域，这是一种速度很快的算法，因为它尽可能少地搜索链表。如果空闲区域的大小和申请分配的大小正好一样，则把这个空闲区域分配出去，成功返回;否则将该空闲区分为两部分，一部分区域与申请分配的大小相等，把它分配出去，剩下的一部分区域形成新的空闲区。其释放内存的设计思路很简单，只需把这块区域重新放回双向链表中即可。 </p>
<hr>
<p>以上是内存管理用到的两大数据结构。接下来利用这些结构完成实验。</p>
<h2 id="实验要求">实验要求</h2>
<pre class="prettyprint"><code class=" hljs mizar">
/* In the first fit algorithm, the allocator keeps a list <span class="hljs-keyword">of</span> free blocks (known <span class="hljs-keyword">as</span> the free list) <span class="hljs-keyword">and</span>,
   on receiving a request <span class="hljs-keyword">for</span> memory, scans along the list <span class="hljs-keyword">for</span> the first block <span class="hljs-keyword">that</span> <span class="hljs-keyword">is</span> large enough to
   satisfy the request. If the chosen block <span class="hljs-keyword">is</span> significantly larger than <span class="hljs-keyword">that</span> requested, <span class="hljs-keyword">then</span> it <span class="hljs-keyword">is</span> 
   usually split, <span class="hljs-keyword">and</span> the remainder added to the list <span class="hljs-keyword">as</span> another free block.
   Please see Page 196~198, Section 8.2 <span class="hljs-keyword">of</span> Yan Wei Min's chinese book "Data Structure -- C programming language"
*/
// LAB2 EXERCISE 1: YOUR CODE
// you should rewrite functions: default_init,default_init_memmap,default_alloc_pages, default_free_pages.
/*
 * Details <span class="hljs-keyword">of</span> FFMA
 * (1) Prepare: In order to implement the First-Fit Mem Alloc (FFMA), we should manage the free mem block use some list.
 *              The <span class="hljs-keyword">struct</span> free_area_t <span class="hljs-keyword">is</span> used <span class="hljs-keyword">for</span> the management <span class="hljs-keyword">of</span> free mem blocks. At first you should
 *              <span class="hljs-keyword">be</span> familiar to the <span class="hljs-keyword">struct</span> list <span class="hljs-keyword">in</span> list.h. <span class="hljs-keyword">struct</span> list <span class="hljs-keyword">is</span> a simple doubly linked list implementation.
 *              You should know howto USE: list_init, list_add(list_add_after), list_add_before, list_del, list_next, list_prev
 *              Another tricky method <span class="hljs-keyword">is</span> to transform a general list <span class="hljs-keyword">struct</span> to a special <span class="hljs-keyword">struct</span> (<span class="hljs-keyword">such</span> <span class="hljs-keyword">as</span> <span class="hljs-keyword">struct</span> page):
 *              you can find some MACRO: le2page (<span class="hljs-keyword">in</span> memlayout.h), (<span class="hljs-keyword">in</span> future labs: le2vma (<span class="hljs-keyword">in</span> vmm.h), le2proc (<span class="hljs-keyword">in</span> proc.h),etc.)
 * (2) default_init: you can reuse the  demo default_init fun to init the free_list <span class="hljs-keyword">and</span> <span class="hljs-keyword">set</span> nr_free to 0.
 *              free_list <span class="hljs-keyword">is</span> used to record the free mem blocks. nr_free <span class="hljs-keyword">is</span> the total number <span class="hljs-keyword">for</span> free mem blocks.
 * (3) default_init_memmap:  CALL GRAPH: kern_init --&gt; pmm_init--&gt;page_init--&gt;init_memmap--&gt; pmm_manager-&gt;init_memmap
 *              This fun <span class="hljs-keyword">is</span> used to init a free block (with parameter: addr_base, page_number).
 *              First you should init each page (<span class="hljs-keyword">in</span> memlayout.h) <span class="hljs-keyword">in</span> this free block, include:
 *                  p-&gt;flags should <span class="hljs-keyword">be</span> <span class="hljs-keyword">set</span> bit PG_property (<span class="hljs-keyword">means</span> this page <span class="hljs-keyword">is</span> valid. In pmm_init fun (<span class="hljs-keyword">in</span> pmm.c),
 *                  the bit PG_reserved <span class="hljs-keyword">is</span> setted <span class="hljs-keyword">in</span> p-&gt;flags)
 *                  if this page  <span class="hljs-keyword">is</span> free <span class="hljs-keyword">and</span> <span class="hljs-keyword">is</span> <span class="hljs-keyword">not</span> the first page <span class="hljs-keyword">of</span> free block, p-&gt;property should <span class="hljs-keyword">be</span> <span class="hljs-keyword">set</span> to 0.
 *                  if this page  <span class="hljs-keyword">is</span> free <span class="hljs-keyword">and</span> <span class="hljs-keyword">is</span> the first page <span class="hljs-keyword">of</span> free block, p-&gt;property should <span class="hljs-keyword">be</span> <span class="hljs-keyword">set</span> to total num <span class="hljs-keyword">of</span> block.
 *                  p-&gt;ref should <span class="hljs-keyword">be</span> 0, because <span class="hljs-keyword">now</span> p <span class="hljs-keyword">is</span> free <span class="hljs-keyword">and</span> no reference.
 *                  We can use p-&gt;page_link to link this page to free_list, (<span class="hljs-keyword">such</span> <span class="hljs-keyword">as</span>: list_add_before(&amp;free_list, &amp;(p-&gt;page_link)); )
 *              Finally, we should sum the number <span class="hljs-keyword">of</span> free mem block: nr_free+=n
 * (4) default_alloc_pages: search find a first free block (block size &gt;=n) <span class="hljs-keyword">in</span> free list <span class="hljs-keyword">and</span> reszie the free block, return the addr
 *              <span class="hljs-keyword">of</span> malloced block.
 *              (4.1) So you should search freelist like this:
 *                       list_entry_t le = &amp;free_list;
 *                       while((le=list_next(le)) != &amp;free_list) {
 *                       ....
 *                 (4.1.1) In while loop, get the <span class="hljs-keyword">struct</span> page <span class="hljs-keyword">and</span> check the p-&gt;property (record the num <span class="hljs-keyword">of</span> free block) &gt;=n?
 *                       <span class="hljs-keyword">struct</span> Page *p = le2page(le, page_link);
 *                       if(p-&gt;property &gt;= n){ ...
 *                 (4.1.2) If we find this p, <span class="hljs-keyword">then</span> it' <span class="hljs-keyword">means</span> we find a free block(block size &gt;=n), <span class="hljs-keyword">and</span> the first n pages can <span class="hljs-keyword">be</span> malloced.
 *                     Some flag bits <span class="hljs-keyword">of</span> this page should <span class="hljs-keyword">be</span> setted: PG_reserved =1, PG_property =0
 *                     unlink the pages <span class="hljs-keyword">from</span> free_list
 *                     (4.1.2.1) If (p-&gt;property &gt;n), we should re-caluclate number <span class="hljs-keyword">of</span> the the rest <span class="hljs-keyword">of</span> this free block,
 *                           (<span class="hljs-keyword">such</span> <span class="hljs-keyword">as</span>: le2page(le,page_link))-&gt;property = p-&gt;property - n;)
 *                 (4.1.3)  re-caluclate nr_free (number <span class="hljs-keyword">of</span> the the rest <span class="hljs-keyword">of</span> all free block)
 *                 (4.1.4)  return p
 *               (4.2) If we can <span class="hljs-keyword">not</span> find a free block (block size &gt;=n), <span class="hljs-keyword">then</span> return NULL
 * (5) default_free_pages: relink the pages into  free list, maybe merge small free blocks into big free blocks.
 *               (5.1) according the base addr <span class="hljs-keyword">of</span> withdrawed blocks, search free list, find the correct position
 *                     (<span class="hljs-keyword">from</span> low to high addr), <span class="hljs-keyword">and</span> insert the pages. (may use list_next, le2page, list_add_before)
 *               (5.2) reset the fields <span class="hljs-keyword">of</span> pages, <span class="hljs-keyword">such</span> <span class="hljs-keyword">as</span> p-&gt;ref, p-&gt;flags (PageProperty)
 *               (5.3) try to merge low addr <span class="hljs-keyword">or</span> high addr blocks. Notice: should change some pages's p-&gt;property correctly.
 */</code></pre>
<p>根据实验指导书，我们第一个实验需要完成的主要是default_pmm.c中的default_init，default_init_memmap，default_alloc_pages， default_free_pages几个函数的修改。</p>
<h2 id="defaultinit">default_init</h2>
<pre class="prettyprint"><code class=" hljs applescript">/*default_init: you can reuse <span class="hljs-keyword">the</span>  demo default_init fun <span class="hljs-keyword">to</span> init <span class="hljs-keyword">the</span> free_list <span class="hljs-keyword">and</span> <span class="hljs-keyword">set</span> nr_free <span class="hljs-keyword">to</span> <span class="hljs-number">0.</span>
free_list <span class="hljs-keyword">is</span> used <span class="hljs-keyword">to</span> <span class="hljs-type">record</span> <span class="hljs-keyword">the</span> free mem blocks. nr_free <span class="hljs-keyword">is</span> <span class="hljs-keyword">the</span> total <span class="hljs-type">number</span> <span class="hljs-keyword">for</span> free mem blocks.*/
static void default_init(void) { 
    list_init(&amp;free_list);
    nr_free = <span class="hljs-number">0</span>;
}</code></pre>
<p>代码已经写好了无需改动</p>
<h2 id="defaultinitmemmap">default_init_memmap</h2>
<pre class="prettyprint"><code class=" hljs oxygene"> * (<span class="hljs-number">3</span>) default_init_memmap:  CALL GRAPH: kern_init --&gt; pmm_init--&gt;page_init--&gt;init_memmap--&gt; pmm_manager-&gt;init_memmap
 *              This fun <span class="hljs-keyword">is</span> used <span class="hljs-keyword">to</span> init a free <span class="hljs-keyword">block</span> (<span class="hljs-keyword">with</span> parameter: addr_base, page_number).
 *              First you should init <span class="hljs-keyword">each</span> page (<span class="hljs-keyword">in</span> memlayout.h) <span class="hljs-keyword">in</span> this free <span class="hljs-keyword">block</span>, include:
 *                  p-&gt;<span class="hljs-keyword">flags</span> should be <span class="hljs-keyword">set</span> bit PG_property (means this page <span class="hljs-keyword">is</span> valid. <span class="hljs-keyword">In</span> pmm_init fun (<span class="hljs-keyword">in</span> pmm.c),
 *                  the bit PG_reserved <span class="hljs-keyword">is</span> setted <span class="hljs-keyword">in</span> p-&gt;<span class="hljs-keyword">flags</span>)
 *                  <span class="hljs-keyword">if</span> this page  <span class="hljs-keyword">is</span> free <span class="hljs-keyword">and</span> <span class="hljs-keyword">is</span> <span class="hljs-keyword">not</span> the first page <span class="hljs-keyword">of</span> free <span class="hljs-keyword">block</span>, p-&gt;<span class="hljs-keyword">property</span> should be <span class="hljs-keyword">set</span> <span class="hljs-keyword">to</span> <span class="hljs-number">0</span>.
 *                  <span class="hljs-keyword">if</span> this page  <span class="hljs-keyword">is</span> free <span class="hljs-keyword">and</span> <span class="hljs-keyword">is</span> the first page <span class="hljs-keyword">of</span> free <span class="hljs-keyword">block</span>, p-&gt;<span class="hljs-keyword">property</span> should be <span class="hljs-keyword">set</span> <span class="hljs-keyword">to</span> total num <span class="hljs-keyword">of</span> <span class="hljs-keyword">block</span>.
 *                  p-&gt;ref should be <span class="hljs-number">0</span>, because now p <span class="hljs-keyword">is</span> free <span class="hljs-keyword">and</span> no <span class="hljs-keyword">reference</span>.
 *                  We can use p-&gt;page_link <span class="hljs-keyword">to</span> link this page <span class="hljs-keyword">to</span> free_list, (such <span class="hljs-keyword">as</span>: list_add_before(&amp;free_list, &amp;(p-&gt;page_link)); )
 *              <span class="hljs-keyword">Finally</span>, we should sum the number <span class="hljs-keyword">of</span> free mem <span class="hljs-keyword">block</span>: nr_free+=n</code></pre>
<p>基本的要求已经知道了，下面修改代码 <br>
源代码</br></p>
<pre class="prettyprint"><code class=" hljs fsharp">default_init_memmap(<span class="hljs-keyword">struct</span> Page *<span class="hljs-keyword">base</span>, size_t n) {
    <span class="hljs-keyword">assert</span>(n &gt; <span class="hljs-number">0</span>);
    <span class="hljs-keyword">struct</span> Page *p = <span class="hljs-keyword">base</span>;
    <span class="hljs-keyword">for</span> (; p != <span class="hljs-keyword">base</span> + n; p ++) {
        <span class="hljs-keyword">assert</span>(PageReserved(p));
        p-&gt;flags = p-&gt;property = <span class="hljs-number">0</span>;
        set_page_ref(p, <span class="hljs-number">0</span>);<span class="hljs-comment">//清空引用</span>
    }
    <span class="hljs-keyword">base</span>-&gt;property = n;<span class="hljs-comment">//说明连续有n个空闲块，属于空闲链表</span>
    SetPageProperty(<span class="hljs-keyword">base</span>);
    nr_free += n;<span class="hljs-comment">//说明连续有n个空闲块，属于空闲链表</span>
    list_add(&amp;free_list, &amp;(<span class="hljs-keyword">base</span>-&gt;page_link));<span class="hljs-comment">//</span>
}</code></pre>
<p>修改为</p>
<pre class="prettyprint"><code class="language-php hljs ">default_init_memmap(struct Page *base, size_t n) {
    assert(n &gt; <span class="hljs-number">0</span>);
    struct Page *p = base;
    <span class="hljs-keyword">for</span> (; p != base + n; p ++) {<span class="hljs-comment">//循环判断是否为保留页</span>
        assert(PageReserved(p));
        p-&gt;flags = p-&gt;property = <span class="hljs-number">0</span>;
        SetPageProperty(p);
        set_page_ref(p, <span class="hljs-number">0</span>);<span class="hljs-comment">//清空引用</span>
        list_add_before(&amp;free_list, &amp;(base-&gt;page_link));<span class="hljs-comment">//插入空闲链表</span>
    }
    base-&gt;property = n;<span class="hljs-comment">//说明连续有n个空闲块，属于物理页管理链表</span>
<span class="hljs-comment">//    SetPageProperty(base);</span>
    nr_free += n;<span class="hljs-comment">//说明连续有n个空闲块，属于空闲链表</span>

}</code></pre>
<h2 id="defaultallocpages">default_alloc_pages</h2>
<p>这个函数的作用是释放已经使用完的页，把他们合并到free_list中。 具体步骤如下： <br>
①在free_list中查找合适的位置以供插入  <br>
②改变被释放页的标志位，以及头部的计数器  <br>
③尝试在free_list中向高地址或低地址合并</br></br></br></p>
<pre class="prettyprint"><code class="language-c++ hljs objectivec">    <span class="hljs-keyword">static</span> <span class="hljs-keyword">struct</span> Page *  
    default_alloc_pages(size_t n) {  
        assert(n &gt; <span class="hljs-number">0</span>);  
        <span class="hljs-keyword">if</span> (n &gt; nr_free) {  
            <span class="hljs-keyword">return</span> <span class="hljs-literal">NULL</span>;  
        }  
        list_entry_t  *len;  
    list_entry_t  *le = &amp;free_list;  
    <span class="hljs-comment">//在空闲链表中寻找合适大小的页块  </span>
        <span class="hljs-keyword">while</span> ((le = list_next(le)) != &amp;free_list) {  
            <span class="hljs-keyword">struct</span> Page *p = le2page(le, page_link);  
    <span class="hljs-comment">//找到了合适大小的页块  </span>
            <span class="hljs-keyword">if</span> (p-&gt;property &gt;= n) {  
    <span class="hljs-keyword">int</span> i;  
    <span class="hljs-keyword">for</span>(i=<span class="hljs-number">0</span>;i&lt;n;i++){  
    len = list_next(le);  
    <span class="hljs-comment">//让pp指向分配的那一页  </span>
    <span class="hljs-comment">//le2page宏可以根据链表元素获得对应的Page指针p  </span>
    <span class="hljs-keyword">struct</span> Page *pp = le2page(temp_le, page_link);  
    <span class="hljs-comment">//设置每一页的标志位  </span>
    SetPageReserved(pp);  
    ClearPageProperty(pp);  
    <span class="hljs-comment">//清除free_list中的链接  </span>
    list_del(le);  
    le = len;  
    }  
    <span class="hljs-keyword">if</span>(p-&gt;property&gt;n){  
    <span class="hljs-comment">//分割的页需要重新设置空闲大小  </span>
    (le2page(le,page_link))-&gt;property = p-&gt;property - n;  
    }  
    <span class="hljs-comment">//第一页重置标志位  </span>
    ClearPageProperty(p);  
    SetPageReserved(p);  
    nr_free -= n;  
    <span class="hljs-keyword">return</span> p;  
    }  
    }  
    <span class="hljs-comment">//否则分配失败  </span>
        <span class="hljs-keyword">return</span> <span class="hljs-literal">NULL</span>;  
    }  </code></pre>
<h2 id="defaultfreepages">default_free_pages</h2>
<p>此函数是用于为进程分配空闲页。 其分配的步骤如下：  <br>
① 寻找足够大的空闲块 ，如果找到了，重新设置标志位 <br>
②从空闲链表中删除此页  <br>
③判断空闲块大小是否合适 ，如果不合适，分割页块 ，如果合适则不进行操作  <br>
④ 计算剩余空闲页个数  <br>
⑤ 返回分配的页块地址 </br></br></br></br></br></p>
<pre class="prettyprint"><code class="language-C++ hljs cs">
    <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span>  
    default_free_pages(<span class="hljs-keyword">struct</span> Page *<span class="hljs-keyword">base</span>, size_t n) {  
    assert(n &gt; <span class="hljs-number">0</span>);  
    assert(PageReserved(<span class="hljs-keyword">base</span>));  
    <span class="hljs-keyword">struct</span> Page *p = <span class="hljs-keyword">base</span>;  
    <span class="hljs-comment">//查找该插入的位置le  </span>
    list_entry_t *le = &amp;free_list;  
    <span class="hljs-keyword">while</span>((le=list_next(le)) != &amp;free_list){  
    p = le2page(le, page_link);  
    <span class="hljs-keyword">if</span>(p&gt;<span class="hljs-keyword">base</span>) <span class="hljs-keyword">break</span>;  
    }  
    <span class="hljs-comment">//向le之前插入n个页（空闲），并设置标志位  </span>
        <span class="hljs-keyword">for</span> (p = <span class="hljs-keyword">base</span>;p&lt;<span class="hljs-keyword">base</span>+n;p++) {  
            list_add_before(le, &amp;(p-&gt;page_link));  
            p-&gt;flags = <span class="hljs-number">0</span>;  
            set_page_ref(p, <span class="hljs-number">0</span>);  
    ClearPageProperty(p);  
    SetPageProperty(p);  
    }  
    <span class="hljs-comment">//将页块信息记录在头部  </span>
        <span class="hljs-keyword">base</span>-&gt;property = n;  
    <span class="hljs-comment">//是否需要合并  </span>
    <span class="hljs-comment">//向高地址合并  </span>
    p = le2page(le, page_link);        
        <span class="hljs-keyword">if</span> (<span class="hljs-keyword">base</span> + n == p) {  
                <span class="hljs-keyword">base</span>-&gt;property += p-&gt;property;  
                p-&gt;property = <span class="hljs-number">0</span> ;  
    }  
    <span class="hljs-comment">//向低地址合并  </span>
    le = list_prev(&amp;(<span class="hljs-keyword">base</span>-&gt;page_link));  
    p = le2page(le, page_link);  
    <span class="hljs-comment">//若低地址已分配则不需要合并  </span>
    <span class="hljs-keyword">if</span>(le!=&amp;free_list &amp;&amp; p==<span class="hljs-keyword">base</span>-<span class="hljs-number">1</span>){  
    <span class="hljs-keyword">while</span>(le!=&amp;free_list){  
    <span class="hljs-keyword">if</span>(p-&gt;property){  
    p-&gt;property +=<span class="hljs-keyword">base</span>-&gt;property;  
    <span class="hljs-keyword">base</span>-&gt;property = <span class="hljs-number">0</span>;  
    <span class="hljs-keyword">break</span>;  
    }  
    le = list_prev(le);  
    p = le2page(le,page_link);  
    }  
    }  
        nr_free += n；  
    }  </code></pre>
<h1 id="练习2实现寻找虚拟地址对应的页表项">练习2：实现寻找虚拟地址对应的页表项</h1>
<blockquote>
<p>这里我们需要实现的是get_pte函数，函数找到一个虚地址对应的二级页表项的内核虚地址，如果此二级页表项不存在，则分配一个包含此项的二级页表。有可能根本就没有对应的二级页表的情况，所以二级页表不必要一开始就分配，而是等到需要的时候再添加对应的二级页表。如果在查找二级页表项时，发现对应的二级页表不存在，则需要根据create参数的值来处理是否创建新的二级页表。如果create参数为0，则get_pte返回NULL；如果create参数不为0，则get_pte需要申请一个新的物理页（通过alloc_page来实现，可在mm/pmm.h中找到它的定义），再在一级页表中添加页目录项指向表示二级页表的新物理页。 <br>
  当建立从一级页表到二级页表的映射时，需要注意设置控制位。这里应该设置同时设置 上PTE_U、PTE_W和PTE_P（定义可在mm/mmu.h）。如果原来就有二级页表，或者新建立了页表，则只需返回对应项的地址即可。</br></p>
</blockquote>
<pre class="prettyprint"><code class=" hljs sql">    pte_t *  
    get_pte(pde_t *pgdir, uintptr_t la, bool <span class="hljs-operator"><span class="hljs-keyword">create</span>) {  
        /*  
         * MACROs <span class="hljs-keyword">or</span> Functions: 
         * PDX(la) = the index <span class="hljs-keyword">of</span> page directory entry <span class="hljs-keyword">of</span> VIRTUAL ADDRESS la. 
         * KADDR(pa) : takes a physical address <span class="hljs-keyword">and</span> returns the <span class="hljs-keyword">corresponding</span> kernel virtual address. 
         * set_page_ref(page,<span class="hljs-number">1</span>) : means the page be referenced <span class="hljs-keyword">by</span> one <span class="hljs-keyword">time</span> 
         * page2pa(page): <span class="hljs-keyword">get</span> the physical address <span class="hljs-keyword">of</span> memory which this (struct Page *) page  manages 
         * struct Page * alloc_page() : allocation a page 
         *memset(void *s, <span class="hljs-keyword">char</span> c, size_t n) : sets the <span class="hljs-keyword">first</span> n bytes <span class="hljs-keyword">of</span> the memory area pointed <span class="hljs-keyword">by</span> s 
         *                                       <span class="hljs-keyword">to</span> the specified <span class="hljs-keyword">value</span> c. 
         * DEFINEs: 
         * PTE_P           <span class="hljs-number">0x001</span>     // page <span class="hljs-keyword">table</span>/directory entry flags <span class="hljs-keyword">bit</span> : Present 
         * PTE_W           <span class="hljs-number">0x002</span>    // page <span class="hljs-keyword">table</span>/directory entry flags <span class="hljs-keyword">bit</span> : Writeable 
         * PTE_U           <span class="hljs-number">0x004</span>    // page <span class="hljs-keyword">table</span>/directory entry flags <span class="hljs-keyword">bit</span> : <span class="hljs-keyword">User</span> can access 
         */  
    //尝试获取页表，注：typedef uintptr_t pte_t;</span>  
        pde_t *pdep = &amp;pgdir[PDX(la)];//若获取不成功则执行下面的语句  
    if (!(*pdep &amp; PTE_P)) {//申请一页  
    struct Page *page;  
    if(!creat || (page = all_page())==NULL){  
    return NULL;  
    } //引用次数需要加1  
    set_page_ref(page, 1);//获取页的线性地址                     
    uintptr_t pa = page2pa(page);   
    memset(KADDR(pa), 0, PGSIZE);//设置权限  
    *pdep  = pa | PTE_U | PTE_W | PTE_P;                   
    }//返回页表地址  
    return &amp;((pte_t *)KADDR(PDE_ADDR(*pdep)))[PTX(la)];            
    }  </code></pre>
<pre><code>pde_t 全称为page directory entry，也就是一级页表的表项（注意：pgdir实际不是表项，而是一级页表本身，pgdir给出页表起始地址。）
pte_t 全称为page table entry，表示二级页表的表项。
uintptr_t 表示为线性地址，由于段式管理只做直接映射，所以它也是逻辑地址。
PTE_U: 位3，表示用户态的软件可以读取对应地址的物理内存页内容
PTE_W: 位2，表示物理内存页内容可写
PTE_P: 位1，表示物理内存页存在
</code></pre>
<h1 id="练习3-释放某虚拟地址所在的页并取消对应的二级页表项的映射">练习3  释放某虚拟地址所在的页并取消对应的二级页表项的映射</h1>
<p>判断此页被引用的次数，如果仅仅被引用一次，则这个页也可以被释放。否则，只能释放页表入口。 <br>
修改函数page_remove_pte</br></p>
<pre class="prettyprint"><code class=" hljs objectivec"><span class="hljs-keyword">static</span> <span class="hljs-keyword">inline</span> <span class="hljs-keyword">void</span>
page_remove_pte(pde_t *pgdir, uintptr_t la, pte_t *ptep) {
<span class="hljs-keyword">if</span> (*ptep &amp; PTE_P) {                    <span class="hljs-comment">//判断页表中该表项是否存在</span>
    <span class="hljs-keyword">struct</span> Page *page = pte2page(*ptep);
    <span class="hljs-keyword">if</span> (page_ref_dec(page) == <span class="hljs-number">0</span>) {      <span class="hljs-comment">//判断是否只被引用了一次</span>
        free_page(page);                <span class="hljs-comment">//如果只被引用了一次，那么可以释放掉此页</span>
    }
    *ptep = <span class="hljs-number">0</span>;                          <span class="hljs-comment">//如果被多次引用，则不能释放此页，只用释放二级页表的表项</span>
    tlb_invalidate(pgdir, la);          <span class="hljs-comment">//更新页表</span>
    }
}</code></pre>
<p>运行结果如下 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170403100438486?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></br></p></hr></div>