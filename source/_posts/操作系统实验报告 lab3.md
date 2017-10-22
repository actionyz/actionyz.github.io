---
title: 操作系统实验报告 lab3
tags: [操作系统实验]
date: 2017-04-24 09:26
---
操作系统实验报告 lab3
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><h2 id="操作系统实验报告-lab3"><center><strong>操作系统实验报告 lab3</strong> </center></h2>
<p><strong>目录：</strong></p>
<p><div class="toc"><div class="toc">
<ul>
<li><ul>
<li><a href="#操作系统实验报告-lab3">操作系统实验报告 lab3   </a></li>
</ul>
</li>
<li><a href="#练习0-填写已有实验">练习0 填写已有实验</a></li>
<li><a href="#练习1-给未被映射的地址映射上物理页">练习1 给未被映射的地址映射上物理页</a><ul>
<li><a href="#1-问题分析">问题分析</a></li>
<li><a href="#2-结构体分析">结构体分析</a><ul>
<li><a href="#vmastruct">vma_struct</a></li>
<li><a href="#mmstruct">mm_struct</a></li>
</ul>
</li>
<li><a href="#3-管理结构关系">管理结构关系</a></li>
<li><a href="#实现代码">实现代码</a></li>
</ul>
</li>
<li><a href="#练习2-补充完成基于fifo的页面替换算法">练习2 补充完成基于FIFO的页面替换算法</a><ul>
<li><a href="#问题分析">问题分析</a></li>
<li><a href="#函数分析">函数分析</a></li>
<li><a href="#实现代码-1">实现代码</a></li>
</ul>
</li>
<li><a href="#实验截图">实验截图</a></li>
<li><a href="#总结">总结</a></li>
</ul>
</div>
</div>
</p>
<blockquote>
<p>本次实验主要完成ucore内核对虚拟内存的管理工作。首先完成初始化虚拟内存管理机制，即需要设置好哪些页需要放在物理内存中，哪些页不需要放在物理内存中，而是可被换出到硬盘上，并涉及完善建立页表映射、页错误异常处理操作等函数实现。最后测试编写的代码有没有达到预期的效果。整体的实验不难，但需要掌握一些数据结构之间的关系，这样对理解虚拟内存管理有很大的帮助。</p>
</blockquote>
<h1 id="练习0-填写已有实验">练习0 填写已有实验</h1>
<blockquote>
<p>将实验二代码补全至实验三</p>
</blockquote>
<p>在这里了仍然采用meld工具直接进行比较，截图如下</p>
<p></p><center> <br>
<img alt="*这里有图 1*" src="http://img.blog.csdn.net/20170424222601557?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
图1 文件夹对比</br></img></br></center><p></p>
<p>总共需要修改的文件有三个</p>
<pre class="prettyprint"><code class=" hljs avrasm">default_pmm<span class="hljs-preprocessor">.c</span>
pmm<span class="hljs-preprocessor">.c</span>
trap<span class="hljs-preprocessor">.c</span></code></pre>
<p>将代码直接从<code>lab2</code>复制至<code>lab3</code>，修改完之后进入下面的实验</p>
<h1 id="练习1-给未被映射的地址映射上物理页">练习1 给未被映射的地址映射上物理页</h1>
<blockquote>
<p>完成 do_pgfault函数,给未被映射的地址映射上物理页。设置访问权限的时候要参考所在页面的VMA权限，同时需要注意映射物理页时需要操作内存控制结构指定的页表，而不是内核页表</p>
</blockquote>
<h2 id="1-问题分析">1 问题分析：</h2>
<p>当启动分页机制以后，如果一条指令或数据的虚拟地址所对应的物理页不在内存中，或者访问权限不够，那么就会产生页错误异常。其具体原因有以下三点：</p>
<ol>
<li>页表项全为0——虚拟地址与物理地址未建立映射关系或已被撤销。</li>
<li>物理页面不在内存中——需要进行换页机制。</li>
<li>访问权限不够——应当报错。</li>
</ol>
<p>当出现上面情况之一,那么就会产生页面page fault(#PF)异常。产生异常的线性地址存储在 <br>
CR2中,并且将是page fault的产生类型保存在 error code 中</br></p>
<h2 id="2-结构体分析">2 结构体分析</h2>
<h3 id="vmastruct">vma_struct</h3>
<p>每个进程只有一个<code>mm_struct</code>结构，在每个进程的<code>task_struct</code>结构体中，有一个指向该进程的结构。可以说，<code>mm_struct</code>结构是对整个用户空间的描述。 <br>
</br></p><center></center><p></p>
<pre class="prettyprint"><code class=" hljs sql"> struct vma_struct {  
        // the <span class="hljs-operator"><span class="hljs-keyword">set</span> <span class="hljs-keyword">of</span> vma <span class="hljs-keyword">using</span> the same PDT  
        struct mm_struct *vm_mm;</span>  
        uintptr_t vm_start;      // <span class="hljs-operator"><span class="hljs-keyword">start</span> addr <span class="hljs-keyword">of</span> vma  
        uintptr_t vm_end;</span>      // <span class="hljs-operator"><span class="hljs-keyword">end</span> addr <span class="hljs-keyword">of</span> vma  
        uint32_t vm_flags;</span>     // flags of vma  
        //linear list link which sorted by <span class="hljs-operator"><span class="hljs-keyword">start</span> addr <span class="hljs-keyword">of</span> vma 
        list_entry_t list_link;</span>  
    };  </code></pre>
<blockquote>
<p>1.vm_start和vm_end描述的是一个合理的地址空间范围（即严格确保 vm_start &lt; vm_end的关系）； <br>
  2.list_link是一个双向链表，按照从小到大的顺序把一系列用vma_struct表示的虚拟内存空间链接起来，并且还要求这些链起来的vma_struct应该是不相交的，即vma之间的地址空间无交集； <br>
  3.vm_flags表示了这个虚拟内存空间的属性，目前的属性包括 <br>
   #define VM_READ 0x00000001   //只读 <br>
    #define VM_WRITE 0x00000002  //可读写 <br>
    #define VM_EXEC 0x00000004   //可执行 <br>
  4.vm_mm是一个指针，指向一个比vma_struct更高的抽象层次的数据结构mm_struct</br></br></br></br></br></br></p>
</blockquote>
<h3 id="mmstruct">mm_struct</h3>
<p>较高层次的结构<code>vm_area_struct</code>描述了虚拟地址空间的一个区间(简称虚拟区)。 <br>
</br></p><center></center><p></p>
<pre class="prettyprint"><code class=" hljs cs">   <span class="hljs-keyword">struct</span> mm_struct {  
        <span class="hljs-comment">// linear list link which sorted by start addr of vma  </span>
        list_entry_t mmap_list;  
        <span class="hljs-comment">// current accessed vma, used for speed purpose  </span>
        <span class="hljs-keyword">struct</span> vma_struct *mmap_cache;  
        pde_t *pgdir; <span class="hljs-comment">// the PDT of these vma  </span>
        <span class="hljs-keyword">int</span> map_count; <span class="hljs-comment">// the count of these vma  </span>
        <span class="hljs-keyword">void</span> *sm_priv; <span class="hljs-comment">// the private data for swap manager  </span>
    }; </code></pre>
<blockquote>
<p>1.mmap_list是双向链表头，链接了所有属于同一页目录表的虚拟内存空间 <br>
  2.mmap_cache是指向当前正在使用的虚拟内存空间 <br>
  3.pgdir所指向的就是 mm_struct数据结构所维护的页表 <br>
  4.map_count记录mmap_list里面链接的vma_struct的个数 <br>
  5.sm_priv指向用来链接记录页访问情况的链表头</br></br></br></br></p>
</blockquote>
<h2 id="3-管理结构关系">3 管理结构关系</h2>
<p>在进程的<code>task_struct</code>结构中包含一个指向 <code>mm_struct</code>结构的指针，<code>mm_strcut</code>用来描述一个进程的虚拟地址空间。 <br>
进程的 <code>mm_struct</code> 则包含装入的可执行映像信息以及进程的页目录指针<code>pgd</code>。该结构还包含有指向 <code>vm_area_struct</code> 结构的几个指针. <br>
每个<code>vm_area_struct</code> 代表进程的一个虚拟地址区间。<code>vm_area_struct</code> 结构含有指向<code>vm_operations_struct</code> 结构的一个指针，<code>vm_operations_struct</code> 描述了在这个区间的操作。<code>vm_operations</code> 结构中包含的是函数指针；其中，<code>open</code>、<code>close</code> 分别用于虚拟区间的打开、关闭，而<code>nopage</code> 用于当虚存页面不在物理内存而引起的“缺页异常”时所应该调用的函数，当 Linux 处理这一缺页异常时（请页机制），就可以为新的虚拟内存区分配实际的物理内存。 <br>
</br></br></br></p><center> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424085008770?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
图2 管理结构图 <br>
<center> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424090053714?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
图3 关系图</br></img></br></center></br></br></img></br></center><p></p>
<h2 id="实现代码">实现代码</h2>
<pre><code>1.目标页面不存在（页表项全为0，即该线性地址与物理地址尚未建立映射或者已经撤销）;
2.相应的物理页面不在内存中（页表项非空，但Present标志位=0，比如在swap分区或磁盘文件上）
3.访问权限不符合（此时页表项P标志=1，比如企图写只读页面）.
</code></pre>
<p></p><center></center><p></p>
<pre class="prettyprint"><code class="language-c++ hljs objectivec">
<span class="hljs-keyword">int</span> do_pgfault(<span class="hljs-keyword">struct</span> mm_struct *mm,uint32_t error_code,uintptr addr)
{
  pte_t *ptep=<span class="hljs-literal">NULL</span>;<span class="hljs-comment">//页目录</span>

 <span class="hljs-keyword">if</span> ((ptep = get_pte(mm-&gt;pgdir, addr, <span class="hljs-number">1</span>)) == <span class="hljs-literal">NULL</span>) { <span class="hljs-comment">//查找页目录 如果不存在则失败</span>
        cprintf(<span class="hljs-string">"get_pte in do_pgfault failed\n"</span>);
        <span class="hljs-keyword">goto</span> failed;
    }

    <span class="hljs-keyword">if</span> (*ptep == <span class="hljs-number">0</span>) {   <span class="hljs-comment">//如果物理地址不存在，那么分配一个物理页 并且与虚拟内存建立对应关系</span>
        <span class="hljs-keyword">if</span> (pgdir_alloc_page(mm-&gt;pgdir, addr, perm) == <span class="hljs-literal">NULL</span>) {
            cprintf(<span class="hljs-string">"pgdir_alloc_page in do_pgfault failed\n"</span>);
            <span class="hljs-keyword">goto</span> failed;
        }
    }
    <span class="hljs-keyword">else</span> {   <span class="hljs-comment">//页表项非空，可以尝试换入页面 </span>
        <span class="hljs-keyword">if</span>(swap_init_ok) {
            <span class="hljs-keyword">struct</span> Page *page=<span class="hljs-literal">NULL</span>;<span class="hljs-comment">//新建内存页指针</span>
            <span class="hljs-keyword">if</span> ((ret = swap_in(mm, addr, &amp;page)) != <span class="hljs-number">0</span>) {<span class="hljs-comment">//根据mm结构和addr地址，尝试将硬盘中的内容换入至page中</span>
                cprintf(<span class="hljs-string">"swap_in in do_pgfault failed\n"</span>);
                <span class="hljs-keyword">goto</span> failed;
            }    
            page_insert(mm-&gt;pgdir, page, addr, perm);<span class="hljs-comment">//建立虚拟地址和物理地址之间的对应关系 </span>
            swap_map_swappable(mm, addr, page, <span class="hljs-number">1</span>);<span class="hljs-comment">//将此页面设置为可交换的  </span>
            page-&gt;pra_vaddr = addr;
        }
        <span class="hljs-keyword">else</span> {
            cprintf(<span class="hljs-string">"no swap_init_ok but ptep is %x, failed\n"</span>,*ptep);
            <span class="hljs-keyword">goto</span> failed;
        }
   }
   ret = <span class="hljs-number">0</span>
   failed:
   <span class="hljs-keyword">return</span> ret;
}</code></pre>
<h1 id="练习2-补充完成基于fifo的页面替换算法">练习2 补充完成基于FIFO()的页面替换算法</h1>
<blockquote>
<p>完成vmm.c中的do_pgfault函数，并且在实现FIFO算法的swap_fifo.c中完成map_swappable和swap_out_vistim函数。通过对swap的测试。</p>
</blockquote>
<h2 id="问题分析">问题分析</h2>
<p>根据练习1，当页错误异常发生时，有可能是因为页面保存在swap区或者磁盘文件上造成的，练习2需要利用页面替换算法解决这个问题。</p>
<h2 id="函数分析">函数分析</h2>
<p><code>map_swappable()</code> 主要解决的问题是将最近被用到的页面添加到算法所维护的次序队列。 <br>
链表是由<code>list_entry_t</code>指针串起来的 <br>
<code>mm_struct</code>结构体的<code>sm_priv</code>元素是交换管理的私有数据 是一串链表 满足<code>FIFO</code>先进先出</br></br></p>
<p><code>_fifo_swap_out_victim()</code>函数是用来查询哪个页面需要被换出，它的主要作用是用来查询哪个页面需要被换出。</p>
<h2 id="实现代码-1">实现代码</h2>
<p><strong>_fifo_map_swappable()</strong></p>
<pre class="prettyprint"><code class="language-c++ hljs objectivec"><span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> _fifo_map_swappable(<span class="hljs-keyword">struct</span> mm_struct *mm, uintptr_t addr, <span class="hljs-keyword">struct</span> Page *page, <span class="hljs-keyword">int</span> swap_in) {        
        list_entry_t *head=(list_entry_t*) mm-&gt;sm_priv;   
        list_entry_t *entry=&amp;(page-&gt;pra_page_link);          
        assert(entry != <span class="hljs-literal">NULL</span> &amp;&amp; head != <span class="hljs-literal">NULL</span>);   
        list_add(head, entry); <span class="hljs-comment">//将最近用到的页面添加到次序队尾  </span>
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;   
    } </code></pre>
<p><strong>_fifo_swap_out_victim()</strong></p>
<pre class="prettyprint"><code class=" hljs rust"><span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span>  
<span class="hljs-number">_f</span>ifo_swap_out_victim(<span class="hljs-keyword">struct</span> mm_struct *mm, <span class="hljs-keyword">struct</span> Page ** ptr_page, <span class="hljs-keyword">int</span> in_tick) {       
    list_entry_t *head=(list_entry_t*) mm-&gt;sm_priv;          
    <span class="hljs-keyword">assert</span>(head != NULL);      
    <span class="hljs-keyword">assert</span>(in_tick==<span class="hljs-number">0</span>);
    list_entry_t *le = head-&gt;prev;  <span class="hljs-comment">//用le指示需要被换出的页，最先进入队列的元素head的另一端             </span>
    <span class="hljs-keyword">assert</span>(head!=le);  
    <span class="hljs-keyword">struct</span> Page *p = le2page(le, pra_page_link);<span class="hljs-comment">//le2page宏可以根据链表元素获得对应的Page指针p 这里是物理页属性      </span>
    list_del(le);      <span class="hljs-comment">//将进来最早的页面从队列中删除      </span>
    <span class="hljs-keyword">assert</span>(p !=NULL);       
    *ptr_page = p; <span class="hljs-comment">//将这一页的地址存储在ptr_page中</span>
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>; 
}</code></pre>
<h1 id="实验截图">实验截图</h1>
<p></p><center> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424222730808?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
图4 实验效果图</br></img></br></center><p></p>
<pre class="prettyprint"><code class=" hljs erlang-repl"><span class="hljs-function_or_atom">check_vma_struct</span>() <span class="hljs-function_or_atom">succeeded</span><span class="hljs-exclamation_mark">!</span>
<span class="hljs-function_or_atom">page</span> <span class="hljs-function_or_atom">fault</span> <span class="hljs-function_or_atom">at</span> <span class="hljs-number">0</span>x00000100: <span class="hljs-variable">K</span>/<span class="hljs-variable">W</span> [<span class="hljs-function_or_atom">no</span> <span class="hljs-function_or_atom">page</span> <span class="hljs-function_or_atom">found</span>].
<span class="hljs-function_or_atom">check_pgfault</span>() <span class="hljs-function_or_atom">succeeded</span><span class="hljs-exclamation_mark">!</span>
<span class="hljs-function_or_atom">check_vmm</span>() <span class="hljs-function_or_atom">succeeded</span>.</code></pre>
<p></p><center> <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170424222810399?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
图5 实验效果图</br></img></br></center><p></p>
<p><code>check_swap() succeeded!</code> <br>
实验运行成功！</br></p>
<h1 id="总结">总结</h1>
<p>1.整个实验下来感觉自己收获很多，通过上网查找资料，自己深刻学习了虚拟内存管理的机制，通过实验更是加深了印象。 <br>
2.实验中遇到了很多的问题，比如进程虚拟内存的管理涉及到的一些结构体的关系，由于种类比较多很容易混乱，通过列图梳理了他们之间的关系。 <br>
3.操作系统这门课程比较基础，涉及到的内容比较多。有时学起来会比较吃力，但这门课程非常重要，是以后专业课的基石，希望在以后自己留给更多的时间在做实验上，从而更加深入的理解、掌握操作系统</br></br></p></div>