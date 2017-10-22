---
title: 操作系统实验报告 lab4
tags: [操作系统实验]
date: 2017-04-30 22:44
---
操作系统实验报告 lab4
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p><div class="toc">
<ul>
<li><a href="#练习0-填写已有实验">练习0  填写已有实验</a></li>
<li><a href="#练习1-分配并初始化一个进程控制块需要编码">练习1 分配并初始化一个进程控制块需要编码</a><ul>
<li><a href="#关键数据结构-struct-procstruct">关键数据结构 struct proc_struct</a></li>
<li><a href="#代码填写">代码填写</a></li>
<li><a href="#context和tf的作用分析">context和tf的作用分析</a></li>
</ul>
</li>
<li><a href="#练习2-为新创建的内核线程分配资源">练习2 为新创建的内核线程分配资源</a><ul>
<li><a href="#执行步骤">执行步骤</a></li>
<li><a href="#代码填写-1">代码填写</a></li>
</ul>
</li>
<li><a href="#练习3-理解procrun和它调用的函数如何完成进程切换的">练习3 理解proc_run和它调用的函数如何完成进程切换的</a><ul>
<li><a href="#schedule代码分析">schedule代码分析</a></li>
<li><a href="#procrun代码分析">proc_run代码分析</a></li>
<li><a href="#switchto函数分析">switch_to函数分析</a></li>
<li><a href="#运行截图">运行截图</a></li>
</ul>
</li>
<li><a href="#实验总结">实验总结</a></li>
</ul>
</div>
</p>
<h1 id="练习0-填写已有实验">练习0  填写已有实验</h1>
<blockquote>
<p>本实验依赖实验1～实验3。请把已做的实验1～实验3的代码填入本实验中代码中有lab1，lab2，lab3的注释部分。</p>
</blockquote>
<p>运用meld软件进行比较，截图如下： <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170524125559566?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
可以看到，需要补全的文件是</br></img></br></p>
<pre class="prettyprint"><code class=" hljs avrasm">default_pmm<span class="hljs-preprocessor">.c</span>
pmm<span class="hljs-preprocessor">.c</span>
swap_fifo<span class="hljs-preprocessor">.c</span>
vmm<span class="hljs-preprocessor">.c</span>
trap<span class="hljs-preprocessor">.c</span></code></pre>
<hr>
<h1 id="练习1-分配并初始化一个进程控制块需要编码">练习1 分配并初始化一个进程控制块（需要编码）</h1>
<blockquote>
<p>alloc_proc函数（位于kern/process/proc.c中）负责分配并返回一个新的struct proc_struct结构，用于存储新建立的内核线程的管理信息。ucore需要对这个结构进行最基本的初始化，完成这个初始化过程。</p>
</blockquote>
<h2 id="关键数据结构-struct-procstruct">关键数据结构 struct proc_struct</h2>
<pre class="prettyprint"><code class=" hljs http">

<span class="cs">    <span class="hljs-keyword">struct</span> proc_struct {  
        <span class="hljs-keyword">enum</span> proc_state state; <span class="hljs-comment">// Process state  </span>
        <span class="hljs-keyword">int</span> pid; <span class="hljs-comment">// Process ID  </span>
        <span class="hljs-keyword">int</span> runs; <span class="hljs-comment">// the running times of Proces  </span>
        uintptr_t kstack; <span class="hljs-comment">// Process kernel stack  </span>
        <span class="hljs-keyword">volatile</span> <span class="hljs-keyword">bool</span> need_resched; <span class="hljs-comment">// need to be rescheduled to release CPU?  </span>
        <span class="hljs-keyword">struct</span> proc_struct *parent; <span class="hljs-comment">// the parent process  </span>
        <span class="hljs-keyword">struct</span> mm_struct *mm; <span class="hljs-comment">// Process's memory management field  </span>
        <span class="hljs-keyword">struct</span> context context; <span class="hljs-comment">// Switch here to run process  </span>
        <span class="hljs-keyword">struct</span> trapframe *tf; <span class="hljs-comment">// Trap frame for current interrupt  </span>
        uintptr_t cr3; <span class="hljs-comment">// the base addr of Page Directroy Table(PDT)  </span>
        uint32_t flags; <span class="hljs-comment">// Process flag  </span>
        <span class="hljs-keyword">char</span> name[PROC_NAME_LEN + <span class="hljs-number">1</span>]; <span class="hljs-comment">// Process name  </span>
        list_entry_t list_link; <span class="hljs-comment">// Process link list  </span>
        list_entry_t hash_link; <span class="hljs-comment">// Process hash list  </span>
    };  </span></code></pre>
<p>下面对参数进行简单的讲解</p>
<blockquote>
<p>mm：内存管理的信息，包括内存映射列表、页表指针等。 <br>
  state：进程所处的状态。 <br>
  parent：用户进程的父进程（创建它的进程）。 <br>
  kstack：记录了分配给该进程/线程的内核桟的位置。 <br>
  need_resched：是否需要调度 <br>
  context：进程的上下文，用于进程切换 <br>
  tf：中断帧的指针 <br>
  cr3: cr3 保存页表的物理地址</br></br></br></br></br></br></br></p>
</blockquote>
<h2 id="代码填写">代码填写</h2>
<p>根据题目中的提示填写代码</p>
<pre class="prettyprint"><code class=" hljs fsharp">    <span class="hljs-keyword">static</span> <span class="hljs-keyword">struct</span> proc_struct *  
    alloc_proc(<span class="hljs-keyword">void</span>) {  
        <span class="hljs-keyword">struct</span> proc_struct *proc = kmalloc(sizeof(<span class="hljs-keyword">struct</span> proc_struct));  
        <span class="hljs-keyword">if</span> (proc != NULL) {  
        <span class="hljs-comment">//LAB4:EXERCISE1 YOUR CODE  </span>
        /* 
         * below fields <span class="hljs-keyword">in</span> proc_struct need <span class="hljs-keyword">to</span> be initialized 
         *       enum proc_state state;                      <span class="hljs-comment">// Process state </span>
         *       int pid;                                    <span class="hljs-comment">// Process ID </span>
         *       int runs;                                   <span class="hljs-comment">// the running times of Proces </span>
         *       uintptr_t kstack;                           <span class="hljs-comment">// Process kernel stack </span>
         *       volatile bool need_resched; <span class="hljs-comment">// bool value: need to be rescheduled to release CPU? </span>
         *       <span class="hljs-keyword">struct</span> proc_struct *parent;                 <span class="hljs-comment">// the parent process </span>
         *       <span class="hljs-keyword">struct</span> mm_struct *mm;               <span class="hljs-comment">// Process's memory management field </span>
         *       <span class="hljs-keyword">struct</span> context context;                     <span class="hljs-comment">// Switch here to run process </span>
         *       <span class="hljs-keyword">struct</span> trapframe *tf;                       <span class="hljs-comment">// Trap frame for current interrupt </span>
         *       uintptr_t cr3;      <span class="hljs-comment">// CR3 register: the base addr of Page Directroy Table(PDT) </span>
         *       uint32_t flags;                             <span class="hljs-comment">// Process flag </span>
         *       char name[PROC_NAME_LEN + <span class="hljs-number">1</span>];               <span class="hljs-comment">// Process name </span>
         */  
            proc-&gt;state = PROC_UNINIT;  <span class="hljs-comment">//设置进程为“初始”态  </span>
            proc-&gt;pid = -<span class="hljs-number">1</span>; <span class="hljs-comment">//设置进程pid的未初始化值  </span>
            proc-&gt;runs = <span class="hljs-number">0</span>;<span class="hljs-comment">//初始化时间片  </span>
            proc-&gt;kstack = <span class="hljs-number">0</span>;<span class="hljs-comment">//内核栈的地址  </span>
            proc-&gt;need_resched = <span class="hljs-number">0</span>;<span class="hljs-comment">//是否需要调度  </span>
            proc-&gt;parent = NULL;<span class="hljs-comment">//父节点为空  </span>
            proc-&gt;mm = NULL; <span class="hljs-comment">//内存管理初始化  </span>
            memset(&amp;(proc-&gt;context), <span class="hljs-number">0</span>, sizeof(<span class="hljs-keyword">struct</span> context));<span class="hljs-comment">//进程上下文初始化  </span>
            proc-&gt;tf = NULL; <span class="hljs-comment">//中断帧指针置为空，总是能够指向中断前的trapframe  </span>
            proc-&gt;cr3 = boot_cr3;<span class="hljs-comment">//设置内核页目录表的基址  </span>
            proc-&gt;flags = <span class="hljs-number">0</span>; <span class="hljs-comment">//标志位初始化  </span>
            memset(proc-&gt;name, <span class="hljs-number">0</span>, PROC_NAME_LEN); <span class="hljs-comment">//进程名初始化  </span>
        }  
        <span class="hljs-keyword">return</span> proc;  
    }  </code></pre>
<h2 id="context和tf的作用分析">context和*tf的作用分析</h2>
<p>①context：进程的上下文，用于进程切换。起到的作用就是保存了现场。在 ucore中，所有的进程在内核中也是相对独立的，因此context 保存寄存器的目的就在于在内核态中能够进行上下文之间的切换。实际利用context进行上下文切换的函数是在kern/process/switch.S中定义switch_to。</p>
<p>② tf：中断帧的指针，总是指向内核栈的某个位置：当进程从用户空间跳到内核空间时，中断帧记录了进程在被中断前的状态。当内核需要跳回用户空间时，需要调整中断帧以恢复让进程继续执行的各寄存器值。除此之外，ucore内核允许嵌套中断。因此为了保证嵌套中断发生时tf 总是能够指向当前的tf，ucore 在内核栈上维护了 tf 的链。</p>
<hr>
<h1 id="练习2-为新创建的内核线程分配资源">练习2 为新创建的内核线程分配资源</h1>
<blockquote>
<p>创建一个内核线程需要分配和设置好很多资源。kernel_thread函数通过调用do_fork函数完成具体内核线程的创建工作。do_kernel函数会调用alloc_proc函数来分配并初始化一个进程控制块，但alloc_proc只是找到了一小块内存用以记录进程的必要信息，并没有实际分配这些资源。ucore一般通过do_fork实际创建新的内核线程。do_fork的作用是，创建当前内核线程的一个副本，它们的执行上下文、代码、数据都一样，但是存储位置不同。在这个过程中，需要给新内核线程分配资源，并且复制原进程的状态。完成在kern/process/proc.c中的do_fork函数中的处理过程。</p>
</blockquote>
<h2 id="执行步骤">执行步骤</h2>
<p>①调用alloc_proc，首先获得一块用户信息块。 <br>
②为进程分配一个内核栈。 <br>
③复制原进程的内存管理信息到新进程（但内核线程不必做此事） <br>
④复制原进程上下文到新进程 <br>
⑤将新进程添加到进程列表 <br>
⑥唤醒新进程 <br>
⑦返回新进程号（设置子进程号为返回值）</br></br></br></br></br></br></p>
<h2 id="代码填写-1">代码填写</h2>
<p>根据代码提示填写代码如下，同时标注了对于代码的认识理解</p>
<pre class="prettyprint"><code class=" hljs fsharp">    int  
    do_fork(uint32_t clone_flags, uintptr_t stack, <span class="hljs-keyword">struct</span> trapframe *tf) {  
        int ret = -E_NO_FREE_PROC;  
        <span class="hljs-keyword">struct</span> proc_struct *proc;  
        <span class="hljs-keyword">if</span> (nr_process &gt;= MAX_PROCESS) {  
            goto fork_out;  
        }  
        ret = -E_NO_MEM;  
        <span class="hljs-comment">//LAB4:EXERCISE2 YOUR CODE  </span>
        /* 
         * Some Useful MACROs, Functions <span class="hljs-keyword">and</span> DEFINEs, you can <span class="hljs-keyword">use</span> them <span class="hljs-keyword">in</span> below implementation. 
         * MACROs <span class="hljs-keyword">or</span> Functions: 
         *   alloc_proc:   create a proc <span class="hljs-keyword">struct</span> <span class="hljs-keyword">and</span> init fields (lab4:exercise1) 
         *   setup_kstack: alloc pages <span class="hljs-keyword">with</span> size KSTACKPAGE <span class="hljs-keyword">as</span> process kernel stack 
         *   copy_mm:      process <span class="hljs-string">"proc"</span> duplicate OR share process <span class="hljs-string">"current"</span><span class="hljs-attribute">'s</span> mm according clone_flags 
         *                 <span class="hljs-keyword">if</span> clone_flags &amp; CLONE_VM, <span class="hljs-keyword">then</span> <span class="hljs-string">"share"</span> ; <span class="hljs-keyword">else</span> <span class="hljs-string">"duplicate"</span> 
         *   copy_thread:  setup the trapframe on the  process's kernel stack top <span class="hljs-keyword">and</span> 
         *                 setup the kernel entry point <span class="hljs-keyword">and</span> stack <span class="hljs-keyword">of</span> process 
         *   hash_proc:    add proc into proc hash_list 
         *   get_pid:      alloc a unique pid <span class="hljs-keyword">for</span> process 
         *   wakup_proc:   set proc-&gt;state = PROC_RUNNABLE 
         * VARIABLES: 
         *   proc_list:    the process set's list 
         *   nr_process:   the number <span class="hljs-keyword">of</span> process set 
         */  

        <span class="hljs-comment">//    1. call alloc_proc to allocate a proc_struct  </span>
        <span class="hljs-comment">//    2. call setup_kstack to allocate a kernel stack for child process  </span>
        <span class="hljs-comment">//    3. call copy_mm to dup OR share mm according clone_flag  </span>
        <span class="hljs-comment">//    4. call copy_thread to setup tf &amp; context in proc_struct  </span>
        <span class="hljs-comment">//    5. insert proc_struct into hash_list &amp;&amp; proc_list  </span>
        <span class="hljs-comment">//    6. call wakup_proc to make the new child process RUNNABLE  </span>
    <span class="hljs-comment">//    7. set ret vaule using child proc's pid  </span>
    <span class="hljs-comment">//第一步：申请内存块，如果失败，直接返回处理  </span>
        <span class="hljs-keyword">if</span> ((proc = alloc_proc()) == NULL) {  
            goto fork_out;  
        }  
    <span class="hljs-comment">//将子进程的父节点设置为当前进程  </span>
        proc-&gt;parent = current;  
    <span class="hljs-comment">//第二步：为进程分配一个内核栈  </span>
        <span class="hljs-keyword">if</span> (setup_kstack(proc) != <span class="hljs-number">0</span>) {  
            goto bad_fork_cleanup_proc;  
    }  
    <span class="hljs-comment">//第三步：复制父进程的内存信息到子进程  </span>
        <span class="hljs-keyword">if</span> (copy_mm(clone_flags, proc) != <span class="hljs-number">0</span>) {  
            goto bad_fork_cleanup_kstack;  
        }  
        <span class="hljs-comment">//第四步：复制父进程相关寄存器信息（上下文）  </span>
    copy_thread(proc, stack, tf);  
    <span class="hljs-comment">//第五步：将新进程添加到进程列表（此过程需要加保护锁）  </span>
        bool intr_flag;  
        local_intr_save(intr_flag);  
        {  
            proc-&gt;pid = get_pid();  
    <span class="hljs-comment">//建立散列映射方便查找  </span>
            hash_proc(proc);  
    <span class="hljs-comment">//将进程链节点加入进程列表  </span>
            list_add(&amp;proc_list, &amp;(proc-&gt;list_link));  
    <span class="hljs-comment">//进程数+1  </span>
            nr_process ++;  
        }  
        local_intr_restore(intr_flag);  
    <span class="hljs-comment">//第六步：一切准备就绪，唤醒子进程  </span>
        wakeup_proc(proc);  
    <span class="hljs-comment">//第七步：别忘了设置返回的子进程号  </span>
        ret = proc-&gt;pid;  
    fork_out:  
        <span class="hljs-keyword">return</span> ret;  

    bad_fork_cleanup_kstack:  
        put_kstack(proc);  
    bad_fork_cleanup_proc:  
        kfree(proc);  
        goto fork_out;  
    }  </code></pre>
<p>在使用 fork 或 clone 系统调用时产生的进程均会由内核分配一个新的唯一的PID值。 <br>
具体来说，就是在分配PID时，设置一个保护锁，暂时不允许中断，保证了ID的唯一性。上述操作真正完成了资源分配的工作，与第一步中的工作有着明显的区别。do_fork只是创建当前进程的副本，他们执行的上下文，寄存器，代码都是一样的。</br></p>
<hr>
<h1 id="练习3-理解procrun和它调用的函数如何完成进程切换的">练习3 理解proc_run和它调用的函数如何完成进程切换的</h1>
<h2 id="schedule代码分析">schedule代码分析</h2>
<p>在分析 proc_run 函数之前，我们先分析调度函数 schedule() 。 <br>
schedule()代码如下：</br></p>
<pre class="prettyprint"><code class=" hljs perl">void
schedule(void) {
    bool intr_flag;
    list_entry_t <span class="hljs-variable">*le</span>, <span class="hljs-variable">*last</span>;
    struct proc_struct <span class="hljs-variable">*next</span> = NULL;
    local_intr_save(intr_flag);
    {
        current-&gt;need_resched = <span class="hljs-number">0</span>;
        <span class="hljs-keyword">last</span> = (current == idleproc) ? &amp;proc_list : &amp;(current-&gt;list_link);
        le = <span class="hljs-keyword">last</span>;
        <span class="hljs-keyword">do</span> {
            <span class="hljs-keyword">if</span> ((le = list_next(le)) != &amp;proc_list) {
                <span class="hljs-keyword">next</span> = le2proc(le, list_link);
                <span class="hljs-keyword">if</span> (<span class="hljs-keyword">next</span>-&gt;<span class="hljs-keyword">state</span> == PROC_RUNNABLE) {
                    <span class="hljs-keyword">break</span>;
                }
            }
        } <span class="hljs-keyword">while</span> (le != <span class="hljs-keyword">last</span>);
        <span class="hljs-keyword">if</span> (<span class="hljs-keyword">next</span> == NULL || <span class="hljs-keyword">next</span>-&gt;<span class="hljs-keyword">state</span> != PROC_RUNNABLE) {
            <span class="hljs-keyword">next</span> = idleproc;
        }
        <span class="hljs-keyword">next</span>-&gt;runs ++;
        <span class="hljs-keyword">if</span> (<span class="hljs-keyword">next</span> != current) {
            proc_run(<span class="hljs-keyword">next</span>);
        }
    }
    local_intr_restore(intr_flag);
}</code></pre>
<blockquote>
<p>简单介绍一下schedule的执行过程： <br>
  ①设置当前内核线程current-&gt;need_resched为0；  <br>
  ②proc_list队列存储着所有状态的进程/线程，在其中查找下一个处于“就绪”态的线程或进程next；  <br>
  ③找到这样的进程后，就调用proc_run函数，保存当前进程current的执行现场（进程上下文），恢复新进程的执行现场，完成进程切换。 <br>
  至此，新的进程next就开始执行了。由于在proc10中只有两个内核线程，且idleproc要让出CPU给initproc执行，我们可以看到schedule函数通过查找proc_list进程队列，只能找到一个处于“就绪”态的initproc内核线程。</br></br></br></br></p>
</blockquote>
<h2 id="procrun代码分析">proc_run代码分析</h2>
<pre class="prettyprint"><code class=" hljs haskell"><span class="hljs-title">void</span> proc_run(struct proc_struct *<span class="hljs-keyword">proc</span>) {
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">proc</span> != current) {
        bool intr_flag;
        struct proc_struct *prev = current, *next = <span class="hljs-keyword">proc</span>;
        local_intr_save(intr_flag);
        {
            current = <span class="hljs-keyword">proc</span>;
            load_esp0(next-&gt;kstack + <span class="hljs-type">KSTACKSIZE</span>);
            lcr3(next-&gt;cr3);
            switch_to(&amp;(prev-&gt;context), &amp;(next-&gt;context));
        }
        local_intr_restore(intr_flag);
    }
}</code></pre>
<blockquote>
<p>通过proc_run和进一步的switch_to函数完成两个执行现场的切换，具体流程如下： <br>
  ①让current指向next内核线程initproc； <br>
  ②设置任务状态段ts中特权态0下的栈顶指针esp0为next内核线程initproc的内核栈的栈顶，即next-&gt;kstack + KSTACKSIZE ； <br>
  ③设置CR3寄存器的值为next内核线程initproc的页目录表起始地址next-&gt;cr3，这实际上是完成进程间的页表切换； <br>
  由switch_to函数完成具体的两个线程的执行现场切换，即切换各个寄存器，当switch_to函数执行完“ret”指令后，就切换到initproc执行了。</br></br></br></br></p>
</blockquote>
<h2 id="switchto函数分析">switch_to函数分析</h2>
<p>switch_to函数的执行流程：</p>
<pre class="prettyprint"><code class=" hljs perl">.globl switch_to  
switch_to: <span class="hljs-comment"># switch_to(from, to)  </span>
<span class="hljs-comment"># 保存前一个进程的执行现场，前两条汇编指令（如下所示）保存了进程在返回switch_to函数后的指令地址到context.eip中  </span>
movl <span class="hljs-number">4</span>(<span class="hljs-variable">%esp</span>), <span class="hljs-variable">%eax</span> <span class="hljs-comment"># eax points to from  </span>
popl <span class="hljs-number">0</span>(<span class="hljs-variable">%eax</span>) <span class="hljs-comment"># esp--&gt; return address, so save return addr in FROM’s </span>
<span class="hljs-comment"># 保存前一个进程的其他7个寄存器到context中的相应成员变量中。 </span>
movl <span class="hljs-variable">%esp</span>, <span class="hljs-number">4</span>(<span class="hljs-variable">%eax</span>)
movl <span class="hljs-variable">%ebx</span>, <span class="hljs-number">8</span>(<span class="hljs-variable">%eax</span>)
movl <span class="hljs-variable">%ecx</span>, <span class="hljs-number">12</span>(<span class="hljs-variable">%eax</span>)
movl <span class="hljs-variable">%edx</span>, <span class="hljs-number">16</span>(<span class="hljs-variable">%eax</span>)
movl <span class="hljs-variable">%esi</span>, <span class="hljs-number">20</span>(<span class="hljs-variable">%eax</span>)
movl <span class="hljs-variable">%edi</span>, <span class="hljs-number">24</span>(<span class="hljs-variable">%eax</span>)
movl <span class="hljs-variable">%ebp</span>, <span class="hljs-number">28</span>(<span class="hljs-variable">%eax</span>) 
<span class="hljs-comment">#再往后是恢复向一个进程的执行现场，这其实就是上述保存过程的逆执行过程，即从 context 的高地址的域 ebp 开始，逐一把相关域的值赋值给对应的寄存器。</span></code></pre>
<p>设置了initproc-&gt;context.eip = (uintptr_t)forkret，这样，当执行switch_to函数并返回后，initproc将执行其实际上的执行入口地址forkret。</p>
<h2 id="运行截图">运行截图</h2>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170524222250501?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<hr>
<h1 id="实验总结">实验总结</h1>
<p>本次实验主要针对内核线程的管理，所有内核线程直接使用共同的ucore内核内存空间，而用户进程需要维护各自的用户内存空间。以及了解到了进程切换的相关细节操作，更加深一步的了解了操作系统。</p></hr></hr></hr></hr></div>