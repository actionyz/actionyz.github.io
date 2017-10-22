---
title: 操作系统实验报告 lab5
tags: [操作系统实验]
date: 2017-05-31 20:59
---
实验目的：   1.了解第一个用户进程创建过程    2.了解系统调用框架的实现机制    3.了解ucore如何实现系统调用sys_fork/sys_exec/sys_exit/sys_wait来进行进程管理练习0 填写已有实验idt_init函数trip_dispatch函数alloc_proc函数do_fork函数练习1加载应用程序并执行load_icode函数执行流程分析
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><blockquote>
<p>实验目的： <br>
  1.了解第一个用户进程创建过程  <br>
  2.了解系统调用框架的实现机制  <br>
  3.了解ucore如何实现系统调用sys_fork/sys_exec/sys_exit/sys_wait来进行进程管理</br></br></br></p>
</blockquote>
<p><div class="toc"><div class="toc">
<ul>
<li><a href="#练习0-填写已有实验">练习0 填写已有实验</a><ul>
<li><a href="#idtinit函数">idt_init函数</a></li>
<li><a href="#tripdispatch函数">trip_dispatch函数</a></li>
<li><a href="#allocproc函数">alloc_proc函数</a></li>
<li><a href="#dofork函数">do_fork函数</a></li>
</ul>
</li>
<li><a href="#练习1加载应用程序并执行">练习1加载应用程序并执行</a><ul>
<li><a href="#loadicode函数执行流程分析">load_icode函数执行流程分析</a></li>
<li><a href="#代码填写">代码填写</a></li>
<li><a href="#堆栈情况">堆栈情况</a></li>
</ul>
</li>
<li><a href="#练习2-父进程复制自己的内存空间给子进程">练习2 父进程复制自己的内存空间给子进程</a></li>
<li><a href="#练习3-阅读分析源代码理解进程执行forkexecwaitexit的实现以及系统调用的实现">练习3 阅读分析源代码理解进程执行forkexecwaitexit的实现以及系统调用的实现</a><ul>
<li><a href="#关于系统调用">关于系统调用</a></li>
</ul>
</li>
<li><a href="#实验心得">实验心得</a></li>
</ul>
</div>
</div>
</p>
<h1 id="练习0-填写已有实验">练习0 填写已有实验</h1>
<p>利用meld软件进行对比，截图如下 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170531195549846?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
找到要修改的地方：</br></img></br></p>
<pre class="prettyprint"><code class=" hljs avrasm">proc<span class="hljs-preprocessor">.c</span>
default_pmm<span class="hljs-preprocessor">.c</span>
pmm<span class="hljs-preprocessor">.c</span>
swap_fifo<span class="hljs-preprocessor">.c</span>
vmm<span class="hljs-preprocessor">.c</span>
trap<span class="hljs-preprocessor">.c</span>
kdebug<span class="hljs-preprocessor">.c</span></code></pre>
<p>题目并没有结束，还要将代码进一步改进</p>
<h2 id="idtinit函数">idt_init函数</h2>
<pre class="prettyprint"><code class=" hljs cs">    <span class="hljs-comment">/* LAB5 YOUR CODE */</span>   
         <span class="hljs-comment">//you should update your lab1 code (just add ONE or TWO lines of code), let user app to use syscall to get the service of ucore  </span>
         <span class="hljs-comment">//so you should setup the syscall interrupt gate in here  </span>
        <span class="hljs-keyword">extern</span> uintptr_t __vectors[];  
        <span class="hljs-keyword">int</span> i;  
        <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-keyword">sizeof</span>(idt) / <span class="hljs-keyword">sizeof</span>(<span class="hljs-keyword">struct</span> gatedesc); i ++) {  
            SETGATE(idt[i], <span class="hljs-number">0</span>, GD_KTEXT, __vectors[i], DPL_KERNEL);  
        }  
    SETGATE(idt[T_SYSCALL], <span class="hljs-number">1</span>, GD_KTEXT, __vectors[T_SYSCALL], DPL_USER);<span class="hljs-comment">//设置相应中断门即可  </span>
    lidt(&amp;idt_pd);  </code></pre>
<p>改进<code>SETGATE(idt[T_SYSCALL], 1, GD_KTEXT, __vectors[T_SYSCALL], DPL_USER);//设置相应中断门即可</code> <br>
<code>在上述代码中，可以看到在执行加载中断描述符表lidt指令前，专门设置了一个特殊的中断描述符idt[T_SYSCALL]，它的特权级设置为DPL_USER，中断向量处理地址在__vectors[T_SYSCALL]处。这样建立好这个中断描述符后，一旦用户进程执行“INT T_SYSCALL”后，由于此中断允许用户态进程产生（注意它的特权级设置为DPL_USER），所以CPU就会从用户态切换到内核态，保存相关寄存器，并跳转到__vectors[T_SYSCALL]处开始执行，形成如下执行路径：</code> <br>
<code>vector128(vectors.S)--\&gt; <br>
\_\_alltraps(trapentry.S)--\&gt;trap(trap.c)--\&gt;trap\_dispatch(trap.c)----\&gt;syscall(syscall.c)-</br></code> <br>
在syscall中，根据系统调用号来完成不同的系统调用服务。</br></br></br></p>
<h2 id="tripdispatch函数">trip_dispatch函数</h2>
<pre class="prettyprint"><code class=" hljs http">

<span class="livecodeserver">    <span class="hljs-comment">/* LAB5 YOUR CODE */</span>  
            <span class="hljs-comment">/* you should upate you lab1 code (just add ONE or TWO lines of code): 
             *    Every TICK_NUM cycle, you should set current process's current-&gt;need_resched = 1 
             */</span>  
            <span class="hljs-built_in">ticks</span> ++;  
            <span class="hljs-keyword">if</span> (<span class="hljs-built_in">ticks</span> % TICK_NUM == <span class="hljs-number">0</span>) {  
                assert(current != <span class="hljs-constant">NULL</span>);  
                current-&gt;need_resched = <span class="hljs-number">1</span><span class="hljs-comment">;//时间片用完设置为需要调度  </span>
            }  </span></code></pre>
<p>代码改进<code>current-&gt;need_resched = 1;//时间片用完设置为需要调度</code></p>
<h2 id="allocproc函数">alloc_proc函数</h2>
<pre class="prettyprint"><code class=" hljs lasso">    <span class="hljs-comment">//LAB5 YOUR CODE : (update LAB4 steps)  </span>
        <span class="hljs-comment">/* 
         * below fields(add in LAB5) in proc_struct need to be initialized   
         *       uint32_t wait_state;                        // waiting state 
         *       struct proc_struct *cptr, *yptr, *optr;     // relations between processes 
         */</span>  
        proc<span class="hljs-subst">-&gt;</span>state <span class="hljs-subst">=</span> PROC_UNINIT;<span class="hljs-comment">//设置进程为未初始化状态</span>
        proc<span class="hljs-subst">-&gt;</span>pid <span class="hljs-subst">=</span> <span class="hljs-subst">-</span><span class="hljs-number">1</span>;          <span class="hljs-comment">//未初始化的进程id=-1</span>
        proc<span class="hljs-subst">-&gt;</span>runs <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;          <span class="hljs-comment">//初始化时间片</span>
        proc<span class="hljs-subst">-&gt;</span>kstack <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;      <span class="hljs-comment">//初始化内存栈的地址</span>
        proc<span class="hljs-subst">-&gt;</span>need_resched <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;   <span class="hljs-comment">//是否需要调度设为不需要</span>
        proc<span class="hljs-subst">-&gt;</span><span class="hljs-keyword">parent</span> <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;      <span class="hljs-comment">//置空父节点</span>
        proc<span class="hljs-subst">-&gt;</span>mm <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;      <span class="hljs-comment">//置空虚拟内存</span>
        memset(<span class="hljs-subst">&amp;</span>(proc<span class="hljs-subst">-&gt;</span>context), <span class="hljs-number">0</span>, sizeof(struct context));<span class="hljs-comment">//初始化上下文</span>
        proc<span class="hljs-subst">-&gt;</span>tf <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;      <span class="hljs-comment">//中断帧指针设置为空</span>
        proc<span class="hljs-subst">-&gt;</span>cr3 <span class="hljs-subst">=</span> boot_cr3;      <span class="hljs-comment">//页目录设为内核页目录表的基址</span>
        proc<span class="hljs-subst">-&gt;</span>flags <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;      <span class="hljs-comment">//初始化标志位</span>
        memset(proc<span class="hljs-subst">-&gt;</span>name, <span class="hljs-number">0</span>, PROC_NAME_LEN);<span class="hljs-comment">//置空进程名</span>
        proc<span class="hljs-subst">-&gt;</span>wait_state <span class="hljs-subst">=</span> <span class="hljs-number">0</span>;  <span class="hljs-comment">//初始化进程等待状态  </span>
        proc<span class="hljs-subst">-&gt;</span>cptr <span class="hljs-subst">=</span> proc<span class="hljs-subst">-&gt;</span>optr <span class="hljs-subst">=</span> proc<span class="hljs-subst">-&gt;</span>yptr <span class="hljs-subst">=</span> <span class="hljs-built_in">NULL</span>;<span class="hljs-comment">//进程相关指针初始化  </span>
</code></pre>
<p>代码改进 <br>
<code>proc-&gt;wait_state = 0; //初始化进程等待状态 <br>
            proc-&gt;cptr = proc-&gt;optr = proc-&gt;yptr = NULL;//进程相关指针初始化</br></code> <br>
实验涉及到用户进程，所以会用涉及调度问题，进程相关指针会被初始化</br></br></p>
<h2 id="dofork函数">do_fork函数</h2>
<pre class="prettyprint"><code class=" hljs haskell">    //<span class="hljs-type">LAB5</span> <span class="hljs-type">YOUR</span> <span class="hljs-type">CODE</span> : (update <span class="hljs-type">LAB4</span> steps)  
       /* <span class="hljs-type">Some</span> <span class="hljs-type">Functions</span> 
        *    set_links:  set the relation links <span class="hljs-keyword">of</span> process.  <span class="hljs-type">ALSO</span> <span class="hljs-type">SEE</span>: remove_links:  lean the relation links <span class="hljs-keyword">of</span> process  
        *    <span class="hljs-comment">------------------- </span>
        *    update step <span class="hljs-number">1</span>: set child <span class="hljs-keyword">proc</span>'s parent to current process, make sure current process's wait_state is <span class="hljs-number">0</span> 
        *    update step <span class="hljs-number">5</span>: insert proc_struct into hash_list &amp;&amp; proc_list, set the relation links <span class="hljs-keyword">of</span> process 
        */  
        <span class="hljs-keyword">if</span> ((<span class="hljs-keyword">proc</span> = alloc_proc()) == <span class="hljs-type">NULL</span>) {  
            goto fork_out;  
        }  
        <span class="hljs-keyword">proc</span>-&gt;parent = current;  
        assert(current-&gt;wait_state == <span class="hljs-number">0</span>); //确保当前进程为等待进程  
        <span class="hljs-keyword">if</span> (setup_kstack(<span class="hljs-keyword">proc</span>) != <span class="hljs-number">0</span>) {  
            goto bad_fork_cleanup_proc;  
        }  
        <span class="hljs-keyword">if</span> (copy_mm(clone_flags, <span class="hljs-keyword">proc</span>) != <span class="hljs-number">0</span>) {  
            goto bad_fork_cleanup_kstack;  
        }  
        copy_thread(<span class="hljs-keyword">proc</span>, stack, tf);  
        bool intr_flag;  
        local_intr_save(intr_flag);  
        {  
            <span class="hljs-keyword">proc</span>-&gt;pid = get_pid();  
            hash_proc(<span class="hljs-keyword">proc</span>);  
            set_links(<span class="hljs-keyword">proc</span>);//设置进程的相关链接  
        }  
        local_intr_restore(intr_flag);  
        wakeup_proc(<span class="hljs-keyword">proc</span>);  
    ret = <span class="hljs-keyword">proc</span>-&gt;pid;  </code></pre>
<p>代码改进 <br>
<code>assert(current-&gt;wait_state == 0); //确保当前进程为等待进程</code> <br>
<code>set_links(proc);//设置进程的相关链接</code></br></br></p>
<h1 id="练习1加载应用程序并执行">练习1：加载应用程序并执行</h1>
<blockquote>
<p>do_execv函数调用load_icode（位于kern/process/proc.c中）来加载并解析一个处于内存中的ELF执行文件格式的应用程序，建立相应的用户内存空间来放置应用程序的代码段、数据段等，且要设置好proc_struct结构中的成员变量trapframe中的内容，确保在执行此进程后，能够从应用程序设定的起始执行地址开始执行。需设置正确的trapframe内容。 </p>
</blockquote>
<h2 id="loadicode函数执行流程分析">load_icode函数执行流程分析</h2>
<p>①调用mm_create函数来申请进程的内存管理数据结构mm所需内存空间，并对mm进行初始化；</p>
<p>②调用setup_pgdir来申请一个页目录表所需的一个页大小的内存空间，并把描述ucore内核虚空间映射的内核页表（boot_pgdir所指）的内容拷贝到此新目录表中，最后让mm-&gt;pgdir指向此页目录表，这就是进程新的页目录表了，且能够正确映射内核虚空间；</p>
<p>③根据应用程序执行码的起始位置来解析此ELF格式的执行程序，并调用mm_map函数根据ELF格式的执行程序说明的各个段（代码段、数据段、BSS段等）的起始位置和大小建立对应的vma结构，并把vma插入到mm结构中，从而表明了用户进程的合法用户态虚拟地址空间；</p>
<p>④调用根据执行程序各个段的大小分配物理内存空间，并根据执行程序各个段的起始位置确定虚拟地址，并在页表中建立好物理地址和虚拟地址的映射关系，然后把执行程序各个段的内容拷贝到相应的内核虚拟地址中，至此应用程序执行码和数据已经根据编译时设定地址放置到虚拟内存中了；</p>
<p>⑤需要给用户进程设置用户栈，为此调用mm_mmap函数建立用户栈的vma结构，明确用户栈的位置在用户虚空间的顶端，大小为256个页，即1MB，并分配一定数量的物理内存且建立好栈的虚地址&lt;–&gt;物理地址映射关系；</p>
<p>⑥至此,进程内的内存管理vma和mm数据结构已经建立完成，于是把mm-&gt;pgdir赋值到cr3寄存器中，即更新了用户进程的虚拟内存空间，此时的initproc已经被程序的代码和数据覆盖，成为了第一个用户进程，但此时这个用户进程的执行现场还没建立好；</p>
<p>⑦先清空进程的中断帧，再重新设置进程的中断帧，使得在执行中断返回指令“iret”后，能够让CPU转到用户态特权级，并回到用户态内存空间，使用用户态的代码段、数据段和堆栈，且能够跳转到用户进程的第一条指令执行，并确保在用户态能够响应中断；</p>
<p>至此，用户进程的用户环境已经搭建完毕。此时initproc将按产生系统调用的函数调用路径原路返回，执行中断返回指令“iret”（位于trapentry.S的最后一句）后，将切换到用户进程hello的第一条语句位置_start处（位于user/libs/initcode.S的第三句）开始执行。</p>
<h2 id="代码填写">代码填写</h2>
<pre class="prettyprint"><code class=" hljs mizar">#<span class="hljs-keyword">define</span> KSTACKPAGE          2                           // # <span class="hljs-keyword">of</span> pages <span class="hljs-keyword">in</span> kernel stack
#<span class="hljs-keyword">define</span> KSTACKSIZE          (KSTACKPAGE * PGSIZE)       // sizeof kernel stack

#<span class="hljs-keyword">define</span> USERTOP             0xB0000000
#<span class="hljs-keyword">define</span> USTACKTOP           USERTOP
#<span class="hljs-keyword">define</span> USTACKPAGE          256                         // # <span class="hljs-keyword">of</span> pages <span class="hljs-keyword">in</span> user stack
#<span class="hljs-keyword">define</span> USTACKSIZE          (USTACKPAGE * PGSIZE)       // sizeof user stack

#<span class="hljs-keyword">define</span> USERBASE            0x00200000
#<span class="hljs-keyword">define</span> UTEXT               0x00800000                  // where user programs generally <span class="hljs-keyword">begin</span>
#<span class="hljs-keyword">define</span> USTAB               USERBASE    

    /* LAB5:EXERCISE1 YOUR CODE 
     * should <span class="hljs-keyword">set</span> tf_cs,tf_ds,tf_es,tf_ss,tf_esp,tf_eip,tf_eflags 
     * NOTICE: If we <span class="hljs-keyword">set</span> trapframe correctly, <span class="hljs-keyword">then</span> the user level process can return to USER MODE <span class="hljs-keyword">from</span> kernel. So 
     *          tf_cs should <span class="hljs-keyword">be</span> USER_CS segment (see memlayout.h) 
     *          tf_ds=tf_es=tf_ss should <span class="hljs-keyword">be</span> USER_DS segment 
     *          tf_esp should <span class="hljs-keyword">be</span> the top addr <span class="hljs-keyword">of</span> user stack (USTACKTOP) 
     *          tf_eip should <span class="hljs-keyword">be</span> the entry point <span class="hljs-keyword">of</span> this binary program (elf-&gt;e_entry) 
     *          tf_eflags should <span class="hljs-keyword">be</span> <span class="hljs-keyword">set</span> to enable computer to produce Interrupt 
     */  
tf-&gt;tf_cs = USER_CS;  
    tf-&gt;tf_ds = tf-&gt;tf_es = tf-&gt;tf_ss = USER_DS;  
    tf-&gt;tf_esp = USTACKTOP;  
    tf-&gt;tf_eip = elf-&gt;e_entry;  
    tf-&gt;tf_eflags = FL_IF;//FL_IF为中断打开状态  
    ret = 0;  
out:  
    return ret;  
bad_cleanup_mmap:  
    exit_mmap(mm);  
bad_elf_cleanup_pgdir:  
    put_pgdir(mm);  
bad_pgdir_cleanup_mm:  
    mm_destroy(mm);  
bad_mm:  
    goto out;  
}  </code></pre>
<p>题目中要求对trapframe，确保在执行此进程后，能够从应用程序设定的起始执行地址开始执行。这里其实一个保存现场的作用，当执行完程序之后能够回到原有的地方。 <br>
具体的使用是 <br>
在执行trap函数前，软件还需进一步保存执行系统调用前的执行现场，即把与用户进程继续执行所需的相关寄存器等当前内容保存到当前进程的中断帧trapframe中（注意，在创建进程是，把进程的trapframe放在给进程的内核栈分配的空间的顶部）。软件做的工作在vector128和__alltraps的起始部分：</br></br></p>
<pre class="prettyprint"><code class=" hljs perl">vectors.S::vector128起始处:
  pushl <span class="hljs-variable">$0</span>
  pushl <span class="hljs-variable">$1</span>28
......
trapentry.S::__alltraps起始处:
pushl <span class="hljs-variable">%ds</span>
  pushl <span class="hljs-variable">%es</span>
  pushal
…… </code></pre>
<p>自此，用于保存用户态的用户进程执行现场的trapframe的内容填写完毕，操作系统可开始完成具体的系统调用服务。在sys_getpid函数中，简单地把当前进程的pid成员变量做为函数返回值就是一个具体的系统调用服务。完成服务后，操作系统按调用关系的路径原路返回到__alltraps中。然后操作系统开始根据当前进程的中断帧内容做恢复执行现场操作。其实就是把trapframe的一部分内容保存到寄存器内容。恢复寄存器内容结束后，调整内核堆栈指针到中断帧的tf_eip处，这是内核栈的结构如下：</p>
<pre class="prettyprint"><code class=" hljs applescript">/* <span class="hljs-keyword">below</span> here defined <span class="hljs-keyword">by</span> x86 hardware */
    uintptr_t tf_eip;
    uint16_t tf_cs;
    uint16_t tf_padding3;
    uint32_t tf_eflags;
/* <span class="hljs-keyword">below</span> here only when crossing rings */
    uintptr_t tf_esp;
    uint16_t tf_ss;
    uint16_t tf_padding4;</code></pre>
<p>这时执行“IRET”指令后，CPU根据内核栈的情况回复到用户态，并把EIP指向tf_eip的值，即“INT T_SYSCALL”后的那条指令。这样整个系统调用就执行完毕了。</p>
<h2 id="堆栈情况">堆栈情况</h2>
<p>将上述代码结合着下面的堆栈图可以很容易的理解 填写内容</p>
<pre class="prettyprint"><code class=" hljs brainfuck"><span class="hljs-comment">/*</span> <span class="hljs-comment">*</span>
 <span class="hljs-comment">*</span> <span class="hljs-comment">Virtual</span> <span class="hljs-comment">memory</span> <span class="hljs-comment">map:</span>                                          <span class="hljs-comment">Permissions</span>
 <span class="hljs-comment">*</span>                                                              <span class="hljs-comment">kernel/user</span>
 <span class="hljs-comment">*</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">4G</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>                                 <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>         <span class="hljs-comment">Empty</span> <span class="hljs-comment">Memory</span> <span class="hljs-comment">(*)</span>        <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>                                 <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0xFB000000</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>   <span class="hljs-comment">Cur</span><span class="hljs-string">.</span> <span class="hljs-comment">Page</span> <span class="hljs-comment">Table</span> <span class="hljs-comment">(Kern</span><span class="hljs-string">,</span> <span class="hljs-comment">RW)</span>    <span class="hljs-comment">|</span> <span class="hljs-comment">RW/</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span> <span class="hljs-comment">PTSIZE</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">VPT</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0xFAC00000</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>        <span class="hljs-comment">Invalid</span> <span class="hljs-comment">Memory</span> <span class="hljs-comment">(*)</span>       <span class="hljs-comment">|</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">/</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">KERNTOP</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0xF8000000</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>                                 <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>    <span class="hljs-comment">Remapped</span> <span class="hljs-comment">Physical</span> <span class="hljs-comment">Memory</span>     <span class="hljs-comment">|</span> <span class="hljs-comment">RW/</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span> <span class="hljs-comment">KMEMSIZE</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>                                 <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">KERNBASE</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0xC0000000</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>        <span class="hljs-comment">Invalid</span> <span class="hljs-comment">Memory</span> <span class="hljs-comment">(*)</span>       <span class="hljs-comment">|</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">/</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">USERTOP</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0xB0000000</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>           <span class="hljs-comment">User</span> <span class="hljs-comment">stack</span>            <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>                                 <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">:</span>                                 <span class="hljs-comment">:</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>         <span class="hljs-comment">~~~~~~~~~~~~~~~~</span>        <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">:</span>                                 <span class="hljs-comment">:</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>                                 <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>       <span class="hljs-comment">User</span> <span class="hljs-comment">Program</span> <span class="hljs-comment">&amp;</span> <span class="hljs-comment">Heap</span>       <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">UTEXT</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0x00800000</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>        <span class="hljs-comment">Invalid</span> <span class="hljs-comment">Memory</span> <span class="hljs-comment">(*)</span>       <span class="hljs-comment">|</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">/</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>  <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span> <span class="hljs-literal">-</span>  <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>    <span class="hljs-comment">User</span> <span class="hljs-comment">STAB</span> <span class="hljs-comment">Data</span> <span class="hljs-comment">(optional)</span>    <span class="hljs-comment">|</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">USERBASE</span><span class="hljs-string">,</span> <span class="hljs-comment">USTAB</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0x00200000</span>
 <span class="hljs-comment">*</span>                            <span class="hljs-comment">|</span>        <span class="hljs-comment">Invalid</span> <span class="hljs-comment">Memory</span> <span class="hljs-comment">(*)</span>       <span class="hljs-comment">|</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">/</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">0</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-literal">+</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-literal">+</span> <span class="hljs-comment">0x00000000</span>
 <span class="hljs-comment">*</span> <span class="hljs-comment">(*)</span> <span class="hljs-comment">Note:</span> <span class="hljs-comment">The</span> <span class="hljs-comment">kernel</span> <span class="hljs-comment">ensures</span> <span class="hljs-comment">that</span> <span class="hljs-comment">"Invalid</span> <span class="hljs-comment">Memory"</span> <span class="hljs-comment">is</span> <span class="hljs-comment">*never*</span> <span class="hljs-comment">mapped</span><span class="hljs-string">.</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">"Empty</span> <span class="hljs-comment">Memory"</span> <span class="hljs-comment">is</span> <span class="hljs-comment">normally</span> <span class="hljs-comment">unmapped</span><span class="hljs-string">,</span> <span class="hljs-comment">but</span> <span class="hljs-comment">user</span> <span class="hljs-comment">programs</span> <span class="hljs-comment">may</span> <span class="hljs-comment">map</span> <span class="hljs-comment">pages</span>
 <span class="hljs-comment">*</span>     <span class="hljs-comment">there</span> <span class="hljs-comment">if</span> <span class="hljs-comment">desired</span><span class="hljs-string">.</span>
 <span class="hljs-comment">*</span>
 <span class="hljs-comment">*</span> <span class="hljs-comment">*/</span>
</code></pre>
<h1 id="练习2-父进程复制自己的内存空间给子进程">练习2: 父进程复制自己的内存空间给子进程</h1>
<blockquote>
<p>创建子进程的函数do_fork在执行中将拷贝当前进程（即父进程）的用户内存地址空间中的合法内容到新进程中（子进程），完成内存资源的复制。具体是通过copy_range函数（位于kern/mm/pmm.c中）实现的，请补充copy_range的实现，确保能够正确执行。</p>
</blockquote>
<p><code>如题，这个工作的完整由do_fork函数完成，具体是调用copy_range 函数，而这里我们的任务就是补全这个函数。 <br>
这个具体的调用过程是由do_fork函数调用copy_mm函数，然后copy_mm函数调用dup_mmap函数，最后由这个dup_mmap函数调用copy_range函数。</br></code></p>
<p>即</p>
<p><code>do_fork()----&gt;copy_mm()----&gt;dup_mmap()----&gt;copy_range()</code></p>
<p>代码填写如下</p>
<pre class="prettyprint"><code class=" hljs lasso">int copy_range(pde_t <span class="hljs-subst">*</span><span class="hljs-keyword">to</span>, pde_t <span class="hljs-subst">*</span>from, uintptr_t start, uintptr_t end, bool share) {
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>    
    <span class="hljs-literal">void</span> <span class="hljs-subst">*</span> kva_src <span class="hljs-subst">=</span> page2kva(page);<span class="hljs-comment">//返回父进程的内核虚拟页地址  </span>
    <span class="hljs-literal">void</span> <span class="hljs-subst">*</span> kva_dst <span class="hljs-subst">=</span> page2kva(npage);<span class="hljs-comment">//返回子进程的内核虚拟页地址  </span>
    memcpy(kva_dst, kva_src, PGSIZE);<span class="hljs-comment">//复制父进程到子进程  </span>
    ret <span class="hljs-subst">=</span> page_insert(<span class="hljs-keyword">to</span>, npage, start, perm);<span class="hljs-comment">//建立子进程页地址起始位置与物理地址的映射关系(prem是权限)  </span>
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span>
}</code></pre>
<h1 id="练习3-阅读分析源代码理解进程执行forkexecwaitexit的实现以及系统调用的实现">练习3 阅读分析源代码,理解进程执行fork/exec/wait/exit的实现,以及系统调用的实现</h1>
<p><strong>fork</strong></p>
<p><code>首先当程序执行fork时，fork使用了系统调用SYS_fork,而系统调用SYS_fork则主要是由do_fork和wakeup_proc来完成的。do_fork()完成的工作在lab4的时候已经做过详细介绍，这里再简单说一下，主要是完成了以下工作：</code></p>
<pre><code>1、分配并初始化进程控制块(alloc_proc 函数);
2、分配并初始化内核栈(setup_stack 函数);
3、根据 clone_flag标志复制或共享进程内存管理结构(copy_mm 函数);
4、设置进程在内核(将来也包括用户态)正常运行和调度所需的中断帧和执行上下文(copy_thread 函数);
5、把设置好的进程控制块放入hash_list 和 proc_list 两个全局进程链表中;
6、自此,进程已经准备好执行了,把进程状态设置为“就绪”态;
7、设置返回码为子进程的 id 号。
</code></pre>
<p><code>而wakeup_proc函数主要是将进程的状态设置为等待，即proc-&gt;wait_state = 0，此处不赘述。</code></p>
<hr>
<p><strong>exec</strong></p>
<p><code>当应用程序执行的时候，会调用SYS_exec系统调用,而当ucore收到此系统调用的时候，则会使用do_execve()函数来实现，因此这里我们主要介绍do_execve()函数的功能，函数主要时完成用户进程的创建工作，同时使用户进程进入执行。 <br>
主要工作如下：</br></code></p>
<pre><code>1、首先为加载新的执行码做好用户态内存空间清空准备。如果mm不为NULL，则设置页表为内核空间页表，且进一步判断mm的引用计数减1后是否为0，如果为0，则表明没有进程再需要此进程所占用的内存空间，为此将根据mm中的记录，释放进程所占用户空间内存和进程页表本身所占空间。最后把当前进程的mm内存管理指针为空。
2、接下来是加载应用程序执行码到当前进程的新创建的用户态虚拟空间中。之后就是调用load_icode从而使之准备好执行。（具体load_icode的功能在练习1已经介绍的很详细了，这里不赘述了）
</code></pre>
<hr>
<p><strong>wait</strong></p>
<p> <code> 当执行wait功能的时候，会调用系统调用SYS_wait，而该系统调用的功能则主要由do_wait函数实现，完成对子进程的最后回收工作，即回收子进程的内核栈和进程控制块所占内存空间。 <br>
  具体的功能实现如下：</br></code></p>
<pre><code>1、 如果 pid!=0，表示只找一个进程 id 号为 pid 的退出状态的子进程，否则找任意一个处于退出状态的子进程;
2、 如果此子进程的执行状态不为PROC_ZOMBIE，表明此子进程还没有退出，则当前进程设置执行状态为PROC_SLEEPING（睡眠），睡眠原因为WT_CHILD(即等待子进程退出)，调用schedule()函数选择新的进程执行，自己睡眠等待，如果被唤醒，则重复跳回步骤 1 处执行;
3、 如果此子进程的执行状态为 PROC_ZOMBIE，表明此子进程处于退出状态，需要当前进程(即子进程的父进程)完成对子进程的最终回收工作，即首先把子进程控制块从两个进程队列proc_list和hash_list中删除，并释放子进程的内核堆栈和进程控制块。自此，子进程才彻底地结束了它的执行过程，它所占用的所有资源均已释放。
</code></pre>
<hr>
<p><strong>exit</strong></p>
<p><code>当执行exit功能的时候，会调用系统调用SYS_exit，而该系统调用的功能主要是由do_exit函数实现。具体过程如下：</code></p>
<pre><code>1、先判断是否是用户进程，如果是，则开始回收此用户进程所占用的用户态虚拟内存空间;（具体的回收过程不作详细说明）
2、设置当前进程的中hi性状态为PROC_ZOMBIE，然后设置当前进程的退出码为error_code。表明此时这个进程已经无法再被调度了，只能等待父进程来完成最后的回收工作（主要是回收该子进程的内核栈、进程控制块）
3、如果当前父进程已经处于等待子进程的状态，即父进程的wait_state被置为WT_CHILD，则此时就可以唤醒父进程，让父进程来帮子进程完成最后的资源回收工作。
4、如果当前进程还有子进程,则需要把这些子进程的父进程指针设置为内核线程init,且各个子进程指针需要插入到init的子进程链表中。如果某个子进程的执行状态是 PROC_ZOMBIE,则需要唤醒 init来完成对此子进程的最后回收工作。
5、执行schedule()调度函数，选择新的进程执行。
</code></pre>
<p><code>所以说该函数的功能简单的说就是，回收当前进程所占的大部分内存资源,并通知父进程完成最后的回收工作。</code></p>
<h2 id="关于系统调用">关于系统调用</h2>
<p>ucore所有的系统调用 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630195351263?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
与用户态的函数库调用执行过程相比，系统调用执行过程的有四点主要的不同：</br></img></br></p>
<pre><code>不是通过“CALL”指令而是通过“INT”指令发起调用；
不是通过“RET”指令，而是通过“IRET”指令完成调用返回；
当到达内核态后，操作系统需要严格检查系统调用传递的参数，确保不破坏整个系统的安全性；
执行系统调用可导致进程等待某事件发生，从而可引起进程切换；
</code></pre>
<p>根据之前的分析，应用程序调用的 exit/fork/wait/getpid 等库函数最终都会调用 syscall 函数,只是调用的参数不同而已（分别是 SYS_exit / SYS_fork / SYS_wait / SYS_getid ）</p>
<blockquote>
<p>当调用系统函数时，一般执行INT T_SYSCALL指令后，CPU 根据操作系统建立的系统调用中断描述符，转入内核态，然后开始了操作系统系统调用的执行过程，在执行之前，会保留系统调用前的执行现场，然后保存当前进程的trapframe中，之后操作系统就可以开始完成具体的系统调用服务，完成服务后，调用IRET，CPU根据内核栈的情况恢复到用户态，并把EIP指向tf_eip的值。这样整个系统调用就执行完毕了。</p>
</blockquote>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630202619210?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>实验成功</p>
<h1 id="实验心得">实验心得</h1>
<p>通过本次实验，我了解到了用户进程的创建过程，同时了解了系统调用框架的实现机制。知道了系统调用<code>sys_fork/sys_exec/sys_exit/sys_wait</code>的实现，一开始对系统进程的切换比较模糊，通过实验实践了解到了它的具体实现过程，可以说收货还是很多的。通过跟踪程序流了解了其中的调用顺序和实现机制，如果在试验中能够多一些图解感觉会比较容易理解一点。</p></hr></hr></hr></div>