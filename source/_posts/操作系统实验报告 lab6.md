---
title: 操作系统实验报告 lab6
tags: [操作系统实验]
date: 2017-06-12 02:47
---
练习0 填写已有实验meld的软件进行对比即可 现在将需要修改的文件罗列如下：proc.cdefault_pmm.cpmm.cswap_fifo.cvmm.ctrap.c根据注释的提示，主要是一下两个函数需要额外加以修改。alloc_proc函数 完整代码如下：static struct proc_struct *alloc_proc(void) {    struct proc_
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p><div class="toc"><div class="toc">
<ul>
<li><a href="#练习0-填写已有实验">练习0 填写已有实验</a></li>
<li><a href="#练习1-使用round-robin调度算法">练习1 使用Round Robin调度算法</a><ul>
<li><a href="#0x1-执行过程">0x1 执行过程</a></li>
<li><a href="#0x2-算法实现">0x2 算法实现</a></li>
</ul>
</li>
<li><a href="#练习2-实现stride-scheduling调度算法">练习2 实现Stride Scheduling调度算法</a></li>
<li><a href="#实验结果">实验结果</a></li>
<li><a href="#实验心得">实验心得</a></li>
</ul>
</div>
</div>
</p>
<h1 id="练习0-填写已有实验">练习0 填写已有实验</h1>
<p>meld的软件进行对比即可 <br>
现在将需要修改的文件罗列如下：</br></p>
<pre class="prettyprint"><code class=" hljs avrasm">proc<span class="hljs-preprocessor">.c</span>
default_pmm<span class="hljs-preprocessor">.c</span>
pmm<span class="hljs-preprocessor">.c</span>
swap_fifo<span class="hljs-preprocessor">.c</span>
vmm<span class="hljs-preprocessor">.c</span>
trap<span class="hljs-preprocessor">.c</span></code></pre>
<p>根据注释的提示，主要是一下两个函数需要额外加以修改。</p>
<hr>
<p><strong>alloc_proc函数</strong> <br>
完整代码如下：</br></p>
<pre class="prettyprint"><code class=" hljs lasso">static struct proc_struct <span class="hljs-subst">*</span>
alloc_proc(<span class="hljs-literal">void</span>) {
    struct proc_struct <span class="hljs-subst">*</span>proc <span class="hljs-subst">=</span> kmalloc(sizeof(struct proc_struct));
    <span class="hljs-keyword">if</span> (proc <span class="hljs-subst">!=</span> <span class="hljs-built_in">NULL</span>) {
        proc<span class="hljs-subst">-&gt;</span>state <span class="hljs-subst">=</span> PROC_UNINIT; <span class="hljs-comment">//设置进程初始状态</span>
        proc<span class="hljs-subst">-&gt;</span>pid <span class="hljs-subst">=</span> <span class="hljs-subst">-</span><span class="hljs-number">1</span>;            <span class="hljs-comment">//进程id=-1</span>
        proc<span class="hljs-subst">-&gt;</span>runs <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;            <span class="hljs-comment">//初始化时间片为0</span>
        proc<span class="hljs-subst">-&gt;</span>kstack <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;          <span class="hljs-comment">//初始化内存栈的地址为0</span>
        proc<span class="hljs-subst">-&gt;</span>need_resched <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;    <span class="hljs-comment">//是否需要调度设为不需要</span>
        proc<span class="hljs-subst">-&gt;</span><span class="hljs-keyword">parent</span> <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;       <span class="hljs-comment">//将父节点置空</span>
        proc<span class="hljs-subst">-&gt;</span>mm <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;           <span class="hljs-comment">//置空虚拟内存</span>
        memset(<span class="hljs-subst">&amp;</span>(proc<span class="hljs-subst">-&gt;</span>context), <span class="hljs-number">0</span>, sizeof(struct context));<span class="hljs-comment">//初始化上下文</span>
        proc<span class="hljs-subst">-&gt;</span>tf <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;           <span class="hljs-comment">//将中断帧指针置空</span>
        proc<span class="hljs-subst">-&gt;</span>cr3 <span class="hljs-subst">=</span> boot_cr3;      <span class="hljs-comment">//将页目录设内核页目录表的基址</span>
        proc<span class="hljs-subst">-&gt;</span>flags <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;           <span class="hljs-comment">//初始化标志位</span>
        memset(proc<span class="hljs-subst">-&gt;</span>name, <span class="hljs-number">0</span>, PROC_NAME_LEN);<span class="hljs-comment">//置空进程名</span>
        proc<span class="hljs-subst">-&gt;</span>wait_state <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;      <span class="hljs-comment">//初始化进程等待状态  </span>
        proc<span class="hljs-subst">-&gt;</span>cptr<span class="hljs-subst">=</span>proc<span class="hljs-subst">-&gt;</span>yptr<span class="hljs-subst">=</span>proc<span class="hljs-subst">-&gt;</span>optr <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;<span class="hljs-comment">//初始化进程相关指针  </span>
        proc<span class="hljs-subst">-&gt;</span>rq <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;<span class="hljs-comment">//置空运行队列 </span>
        list_init(<span class="hljs-subst">&amp;</span>(proc<span class="hljs-subst">-&gt;</span>run_link));<span class="hljs-comment">//初始化运行队列的指针</span>
        proc<span class="hljs-subst">-&gt;</span>time_slice <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;      <span class="hljs-comment">//初始化时间片</span>
        proc<span class="hljs-subst">-&gt;</span>lab6_run_pool<span class="hljs-built_in">.</span>left <span class="hljs-subst">=</span> proc<span class="hljs-subst">-&gt;</span>lab6_run_pool<span class="hljs-built_in">.</span>right <span class="hljs-subst">=</span> proc<span class="hljs-subst">-&gt;</span>lab6_run_pool<span class="hljs-built_in">.</span><span class="hljs-keyword">parent</span> <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>; <span class="hljs-comment">//初始化各类指针为空</span>
        proc<span class="hljs-subst">-&gt;</span>lab6_stride <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;     <span class="hljs-comment">//初始化当前运行步数</span>
        proc<span class="hljs-subst">-&gt;</span>lab6_priority <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;   <span class="hljs-comment">//初始化优先级</span>
    }
    <span class="hljs-keyword">return</span> proc;
}</code></pre>
<hr>
<p>相比于lab5，lab6对proc_struct结构体再次做了扩展，这里主要是多出了以下部分</p>
<pre class="prettyprint"><code class=" hljs lasso">
    proc<span class="hljs-subst">-&gt;</span>rq <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;             <span class="hljs-comment">//初始化运行队列为空</span>
    list_init(<span class="hljs-subst">&amp;</span>(proc<span class="hljs-subst">-&gt;</span>run_link));<span class="hljs-comment">//初始化运行队列的指针</span>
    proc<span class="hljs-subst">-&gt;</span>time_slice <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;        <span class="hljs-comment">//初始化时间片</span>
    proc<span class="hljs-subst">-&gt;</span>lab6_run_pool<span class="hljs-built_in">.</span>left <span class="hljs-subst">=</span> proc<span class="hljs-subst">-&gt;</span>lab6_run_pool<span class="hljs-built_in">.</span>right proc<span class="hljs-subst">-&gt;</span>lab6_run_pool<span class="hljs-built_in">.</span><span class="hljs-keyword">parent</span> <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>; <span class="hljs-comment">//初始化各类指针为空，包括父进程等待</span>
    proc<span class="hljs-subst">-&gt;</span>lab6_stride <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;       <span class="hljs-comment">//步数初始化 </span>
    proc<span class="hljs-subst">-&gt;</span>lab6_priority <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;     <span class="hljs-comment">//初始化优先级</span></code></pre>
<hr>
<p><strong>trap_dispatch</strong>函数</p>
<pre class="prettyprint"><code class=" hljs lasso">static <span class="hljs-literal">void</span>
trap_dispatch(struct trapframe <span class="hljs-subst">*</span>tf) {
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>
    ticks <span class="hljs-subst">++</span>;  
    assert(current <span class="hljs-subst">!=</span> <span class="hljs-built_in">NULL</span>);  
    run_timer_list(); <span class="hljs-comment">//更新定时器，并根据参数调用调度算法  </span>
    break;  
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>
}</code></pre>
<h1 id="练习1-使用round-robin调度算法">练习1 使用Round Robin调度算法</h1>
<blockquote>
<p>理解并分析sched_calss中各个函数指针的用法，并接合Round Robin 调度算法描ucore的调度执行过程</p>
</blockquote>
<h2 id="0x1-执行过程">0x1 执行过程</h2>
<blockquote>
<p>让所有runnable态的进程分时轮流使用CPU时间。RR调度器维护当前runnable进程的有序运行队列。当前进程的时间片用完之后，调度器将当前进程放置到运行队列的尾部，再从其头部取出进程进行调度。RR调度算法的就绪队列在组织结构上也是一个双向链表，只是增加了一个成员变量，表明在此就绪进程队列中的最大执行时间片。而且在进程控制块proc_struct中增加了一个成员变量time_slice，用来记录进程当前的可运行时间片段。这是由于RR调度算法需要考虑执行进程的运行时间不能太长。在每个timer到时的时候，操作系统会递减当前执行进程的time_slice，当time_slice为0时，就意味着这个进程运行了一段时间（这个时间片段称为进程的时间片），需要把CPU让给其他进程执行，于是操作系统就需要让此进程重新回到rq的队列尾，且重置此进程的时间片为就绪队列的成员变量最大时间片max_time_slice值，然后再从rq的队列头取出一个新的进程执行。</p>
</blockquote>
<h2 id="0x2-算法实现">0x2 算法实现</h2>
<p><strong>RR_init完成了对进程队列的初始化</strong></p>
<pre class="prettyprint"><code class=" hljs cs">    <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span>  
    RR_init(<span class="hljs-keyword">struct</span> run_queue *rq) {  
        list_init(&amp;(rq-&gt;run_list));  
        rq-&gt;proc_num = <span class="hljs-number">0</span>;  
    }  </code></pre>
<hr>
<p><strong>RR_enqueue的函数实现如下表所示。即把某进程的进程控制块指针放入到rq队列末尾，且如果进程控制块的时间片为0，则需要把它重置为rq成员变量max_time_slice。这表示如果进程在当前的执行时间片已经用完，需要等到下一次有机会运行时，才能再执行一段时间。</strong></p>
<pre class="prettyprint"><code class=" hljs haskell">    static void  
    <span class="hljs-type">RR_enqueue</span>(struct run_queue *rq, struct proc_struct *<span class="hljs-keyword">proc</span>) {  
        assert(list_empty(&amp;(<span class="hljs-keyword">proc</span>-&gt;run_link)));  
        list_add_before(&amp;(rq-&gt;run_list), &amp;(<span class="hljs-keyword">proc</span>-&gt;run_link));  
        <span class="hljs-keyword">if</span> (<span class="hljs-keyword">proc</span>-&gt;time_slice == <span class="hljs-number">0</span> || <span class="hljs-keyword">proc</span>-&gt;time_slice &gt; rq-&gt;max_time_slice) {  
            <span class="hljs-keyword">proc</span>-&gt;time_slice = rq-&gt;max_time_slice;  
        }  
        <span class="hljs-keyword">proc</span>-&gt;rq = rq;  
        rq-&gt;proc_num ++;  
    }  </code></pre>
<hr>
<p><strong>RR_dequeue的函数实现如下表所示。即把就绪进程队列rq的进程控制块指针的队列元素删除，并把表示就绪进程个数的proc_num减一。</strong></p>
<pre class="prettyprint"><code class=" hljs haskell">    static void  
    <span class="hljs-type">RR_dequeue</span>(struct run_queue *rq, struct proc_struct *<span class="hljs-keyword">proc</span>) {  
        assert(!list_empty(&amp;(<span class="hljs-keyword">proc</span>-&gt;run_link)) &amp;&amp; <span class="hljs-keyword">proc</span>-&gt;rq == rq);  
        list_del_init(&amp;(<span class="hljs-keyword">proc</span>-&gt;run_link));  
        rq-&gt;proc_num <span class="hljs-comment">--;  </span>
    }  </code></pre>
<hr>
<p><strong>RR_pick_next的函数实现如下表所示。即选取就绪进程队列rq中的队头队列元素，并把队列元素转换成进程控制块指针。</strong></p>
<pre class="prettyprint"><code class=" hljs objectivec">    <span class="hljs-keyword">static</span> <span class="hljs-keyword">struct</span> proc_struct *  
    RR_pick_next(<span class="hljs-keyword">struct</span> run_queue *rq) {  
        list_entry_t *le = list_next(&amp;(rq-&gt;run_list));  
        <span class="hljs-keyword">if</span> (le != &amp;(rq-&gt;run_list)) {  
            <span class="hljs-keyword">return</span> le2proc(le, run_link);  
        }  
        <span class="hljs-keyword">return</span> <span class="hljs-literal">NULL</span>;  
    }  </code></pre>
<hr>
<p><strong>RR_proc_tick的函数实现如下表所示。即每次timer到时后，trap函数将会间接调用此函数来把当前执行进程的时间片time_slice减一。如果time_slice降到零，则设置此进程成员变量need_resched标识为1，这样在下一次中断来后执行trap函数时，会由于当前进程程成员变量need_resched标识为1而执行schedule函数，从而把当前执行进程放回就绪队列末尾，而从就绪队列头取出在就绪队列上等待时间最久的那个就绪进程执行。</strong></p>
<pre class="prettyprint"><code class=" hljs haskell"><span class="hljs-title">static</span> void <span class="hljs-type">RR_proc_tick</span>(struct run_queue *rq, struct proc_struct *<span class="hljs-keyword">proc</span>) {  
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">proc</span>-&gt;time_slice &gt; <span class="hljs-number">0</span>) {  
        <span class="hljs-keyword">proc</span>-&gt;time_slice <span class="hljs-comment">--;  </span>
    }  
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">proc</span>-&gt;time_slice == <span class="hljs-number">0</span>) {  
        <span class="hljs-keyword">proc</span>-&gt;need_resched = <span class="hljs-number">1</span>;  
    }  
}  </code></pre>
<h1 id="练习2-实现stride-scheduling调度算法">练习2 实现Stride Scheduling调度算法</h1>
<blockquote>
<p>首先需要换掉RR调度器的实现，即用default_sched_stride_c覆盖default_sched.c。然后根据此文件和后续文档对Stride度器的相关描述，完成Stride调度算法的实现。</p>
</blockquote>
<p>首先，根据实验指导书的要求，先用default_sched_stride_c覆盖default_sched.c，即覆盖掉Round Robin调度算法的实现。 <br>
覆盖掉之后需要在该框架上实现Stride Scheduling调度算法。 <br>
关于Stride Scheduling调度算法，经过查阅资料和实验指导书，我们可以简单的把思想归结如下：</br></br></p>
<pre><code>1、为每个runnable的进程设置一个当前状态stride，表示该进程当前的调度权。另外定义其对应的pass值，表示对应进程在调度后，stride 需要进行的累加值。
2、每次需要调度时，从当前 runnable 态的进程中选择 stride最小的进程调度。对于获得调度的进程P，将对应的stride加上其对应的步长pass（只与进程的优先权有关系）。
3、在一段固定的时间之后，回到步骤2，重新调度当前stride最小的进程
</code></pre>
<p>首先是初始化函数stride_init。 <br>
开始初始化运行队列，并初始化当前的运行队，然后设置当前运行队列内进程数目为0。</br></p>
<pre class="prettyprint"><code class=" hljs lasso">static <span class="hljs-literal">void</span>  
stride_init(struct run_queue <span class="hljs-subst">*</span>rq) {  
     <span class="hljs-comment">/* LAB6: YOUR CODE */</span>  
     list_init(<span class="hljs-subst">&amp;</span>(rq<span class="hljs-subst">-&gt;</span>run_list)); 
     rq<span class="hljs-subst">-&gt;</span>lab6_run_pool <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>; 
     rq<span class="hljs-subst">-&gt;</span>proc_num <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;
}  </code></pre>
<hr>
<p>然后是入队函数stride_enqueue，根据之前对该调度算法的分析，这里函数主要是初始化刚进入运行队列的进程 proc 的stride属性，然后比较队头元素与当前进程的步数大小，选择步数最小的运行，即将其插入放入运行队列中去，这里并未放置在队列头部。最后初始化时间片，然后将运行队列进程数目加一。</p>
<pre class="prettyprint"><code class=" hljs haskell"><span class="hljs-title">static</span> void  
<span class="hljs-title">stride_enqueue</span>(struct run_queue *rq, struct proc_struct *<span class="hljs-keyword">proc</span>) {  
     /* <span class="hljs-type">LAB6</span>: <span class="hljs-type">YOUR</span> <span class="hljs-type">CODE</span> */  
<span class="hljs-preprocessor">#if USE_SKEW_HEAP  </span>
     rq-&gt;lab6_run_pool = //在使用优先队列的实现中表示当前优先队列的头元素  
          skew_heap_insert(rq-&gt;lab6_run_pool, &amp;(<span class="hljs-keyword">proc</span>-&gt;lab6_run_pool), proc_stride_comp_f);//比较队头元素与当前进程的步数大小，选择步数最小的运行  
<span class="hljs-preprocessor">#else  </span>
     assert(list_empty(&amp;(<span class="hljs-keyword">proc</span>-&gt;run_link)));  
     list_add_before(&amp;(rq-&gt;run_list), &amp;(<span class="hljs-keyword">proc</span>-&gt;run_link));//将 <span class="hljs-keyword">proc</span>插入放入运行队列中去  
<span class="hljs-preprocessor">#endif  </span>
     <span class="hljs-keyword">if</span> (<span class="hljs-keyword">proc</span>-&gt;time_slice == <span class="hljs-number">0</span> || <span class="hljs-keyword">proc</span>-&gt;time_slice &gt; rq-&gt;max_time_slice) {//初始化时间片  
          <span class="hljs-keyword">proc</span>-&gt;time_slice = rq-&gt;max_time_slice;  
     }  
     <span class="hljs-keyword">proc</span>-&gt;rq = rq;  
     rq-&gt;proc_num ++;  
}  </code></pre>
<hr>
<p>然后是出队函数stride_dequeue，即完成将一个进程从队列中移除的功能，这里使用了优先队列。最后运行队列数目减一。</p>
<pre class="prettyprint"><code class=" hljs haskell"><span class="hljs-title">static</span> void  
<span class="hljs-title">stride_dequeue</span>(struct run_queue *rq, struct proc_struct *<span class="hljs-keyword">proc</span>) {  
     /* <span class="hljs-type">LAB6</span>: <span class="hljs-type">YOUR</span> <span class="hljs-type">CODE</span> */  
<span class="hljs-preprocessor">#if USE_SKEW_HEAP  </span>
     rq-&gt;lab6_run_pool =   
          skew_heap_remove(rq-&gt;lab6_run_pool, &amp;(<span class="hljs-keyword">proc</span>-&gt;lab6_run_pool), proc_stride_comp_f);// 在斜堆中删除相应元素  
<span class="hljs-preprocessor">#else  </span>
     assert(!list_empty(&amp;(<span class="hljs-keyword">proc</span>-&gt;run_link)) &amp;&amp; <span class="hljs-keyword">proc</span>-&gt;rq == rq);  
     list_del_init(&amp;(<span class="hljs-keyword">proc</span>-&gt;run_link));// 从运行队列中删除相应元素  
<span class="hljs-preprocessor">#endif  </span>
     rq-&gt;proc_num <span class="hljs-comment">--;  </span>
}  </code></pre>
<hr>
<p>接下来就是进程的调度函数stride_pick_next，观察代码，它的核心是先扫描整个运行队列，返回其中stride值最小的对应进程，然后更新对应进程的stride值，将步长设置为优先级的倒数，如果为0则设置为最大的步长。</p>
<pre class="prettyprint"><code class=" hljs lasso">static struct proc_struct <span class="hljs-subst">*</span>
stride_pick_next(struct run_queue <span class="hljs-subst">*</span>rq) {
     <span class="hljs-comment">/* LAB6: YOUR CODE */</span>
<span class="hljs-variable">#if</span> USE_SKEW_HEAP
     <span class="hljs-keyword">if</span> (rq<span class="hljs-subst">-&gt;</span>lab6_run_pool <span class="hljs-subst">==</span> <span class="hljs-built_in">NULL</span>) <span class="hljs-keyword">return</span> <span class="hljs-built_in">NULL</span>;
     struct proc_struct <span class="hljs-subst">*</span>p <span class="hljs-subst">=</span> le2proc(rq<span class="hljs-subst">-&gt;</span>lab6_run_pool, lab6_run_pool);
<span class="hljs-variable">#else</span>
     list_entry_t <span class="hljs-subst">*</span>le <span class="hljs-subst">=</span> list_next(<span class="hljs-subst">&amp;</span>(rq<span class="hljs-subst">-&gt;</span>run_list));

     <span class="hljs-keyword">if</span> (le <span class="hljs-subst">==</span> <span class="hljs-subst">&amp;</span>rq<span class="hljs-subst">-&gt;</span>run_list)
          <span class="hljs-keyword">return</span> <span class="hljs-built_in">NULL</span>;

     struct proc_struct <span class="hljs-subst">*</span>p <span class="hljs-subst">=</span> le2proc(le, run_link);
     le <span class="hljs-subst">=</span> list_next(le);
     <span class="hljs-keyword">while</span> (le <span class="hljs-subst">!=</span> <span class="hljs-subst">&amp;</span>rq<span class="hljs-subst">-&gt;</span>run_list)
     {
          struct proc_struct <span class="hljs-subst">*</span>q <span class="hljs-subst">=</span> le2proc(le, run_link);
          <span class="hljs-keyword">if</span> ((int32_t)(p<span class="hljs-subst">-&gt;</span>lab6_stride <span class="hljs-subst">-</span> q<span class="hljs-subst">-&gt;</span>lab6_stride) <span class="hljs-subst">&gt;</span> <span class="hljs-number">0</span>)
               p <span class="hljs-subst">=</span> q;
          le <span class="hljs-subst">=</span> list_next(le);
     }
<span class="hljs-variable">#endif</span>
    <span class="hljs-comment">//更新对应进程的stride值</span>
     <span class="hljs-keyword">if</span> (p<span class="hljs-subst">-&gt;</span>lab6_priority <span class="hljs-subst">==</span> <span class="hljs-number">0</span>)<span class="hljs-comment">//优先级设置  </span>
          p<span class="hljs-subst">-&gt;</span>lab6_stride <span class="hljs-subst">+=</span> BIG_STRIDE;<span class="hljs-comment">//步长为0则设置为最大步长保持相减的有效性  </span>
     <span class="hljs-keyword">else</span> p<span class="hljs-subst">-&gt;</span>lab6_stride <span class="hljs-subst">+=</span> BIG_STRIDE <span class="hljs-subst">/</span> p<span class="hljs-subst">-&gt;</span>lab6_priority;<span class="hljs-comment">//步长设置为优先级的倒数  </span>
     <span class="hljs-keyword">return</span> p;  
}</code></pre>
<hr>
<p>函数stride_proc_tick的主要工作是检测当前进程的时间片是否已经用完。如果时间片已经用完,就会按照正确的流程进行进程的切换工作。这里和之前实现的Round Robin调度算法一样，所采用的思想也是一致的</p>
<p>优先队列比较函数proc_stride_comp_f的实现，主要利用思路是通过相减之后的值，进行判断大小</p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span>  
proc_stride_comp_f(<span class="hljs-keyword">void</span> *a, <span class="hljs-keyword">void</span> *b)  
{  
     <span class="hljs-keyword">struct</span> proc_struct *p = le2proc(a, lab6_run_pool);  
     <span class="hljs-keyword">struct</span> proc_struct *q = le2proc(b, lab6_run_pool);  
     int32_t c = p-&gt;lab6_stride - q-&gt;lab6_stride;<span class="hljs-comment">//步数相减，通过正负比较大小关系  </span>
     <span class="hljs-keyword">if</span> (c &gt; <span class="hljs-number">0</span>) <span class="hljs-keyword">return</span> <span class="hljs-number">1</span>;  
     <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (c == <span class="hljs-number">0</span>) <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;  
     <span class="hljs-keyword">else</span> <span class="hljs-keyword">return</span> -<span class="hljs-number">1</span>;  
}  </code></pre>
<h1 id="实验结果">实验结果</h1>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630212834411?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630213052599?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<h1 id="实验心得">实验心得</h1>
<p>通过这一次实验我对轮转法以及stride法有了更多的了解，stride法其实就是轮转法的一种升级。stride法加入了对进程优先级的调整，步数越小，进程优先级越大。这种改变更加合理。进程的调度极大的提高了CPU的利用率。更加深刻的理解了进程切换的原理，对Stride Schedule算法的原理和算法可控性和确定性有了更深的认识。</p></hr></hr></hr></hr></hr></hr></hr></hr></hr></hr></hr></div>