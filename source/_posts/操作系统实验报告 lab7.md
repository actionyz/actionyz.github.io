---
title: 操作系统实验报告 lab7
tags: [操作系统实验]
date: 2017-06-12 09:35
---
练习0填写已有实验练习1 理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题0x0 哲学家问题0x1 信号量介绍0x2 P操作V操作实现0x3 代码分析0x4 信号量性质练习2 完成内核级条件变量和基于内核级条件变量的哲学家就餐问题0x1 管程机制0x2 数据结构0x3 组成函数实验截图练习0:填写已有实验使用meld的软件进行对比即可  这里把需要填充的文件罗列如下
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><p><div class="toc"><div class="toc">
<ul>
<li><a href="#练习0填写已有实验" target="_blank">练习0填写已有实验</a></li>
<li><a href="#练习1-理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题" target="_blank">练习1 理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题</a><ul>
<li><a href="#0x0-哲学家问题" target="_blank">0x0 哲学家问题</a></li>
<li><a href="#0x1-信号量介绍" target="_blank">0x1 信号量介绍</a></li>
<li><a href="#0x2-p操作v操作实现" target="_blank">0x2 P操作V操作实现</a></li>
<li><a href="#0x3-代码分析" target="_blank">0x3 代码分析</a></li>
<li><a href="#0x4-信号量性质" target="_blank">0x4 信号量性质</a></li>
</ul>
</li>
<li><a href="#练习2-完成内核级条件变量和基于内核级条件变量的哲学家就餐问题" target="_blank">练习2 完成内核级条件变量和基于内核级条件变量的哲学家就餐问题</a><ul>
<li><a href="#0x1-管程机制" target="_blank">0x1 管程机制</a></li>
<li><a href="#0x2-数据结构" target="_blank">0x2 数据结构</a></li>
<li><a href="#0x3-组成函数" target="_blank">0x3 组成函数</a></li>
</ul>
</li>
<li><a href="#实验截图" target="_blank">实验截图</a></li>
<li><a href="#实验感悟" target="_blank">实验感悟</a></li>
</ul>
</div>
</div>
</p>
<h1 id="练习0填写已有实验">练习0:填写已有实验</h1>
<p>使用meld的软件进行对比即可 <br>
<img alt="这里写图片描述" src="http://img.blog.csdn.net/20170701002606387?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
这里把需要填充的文件罗列如下：</br></img></br></p>
<pre class="prettyprint"><code class=" hljs avrasm">proc<span class="hljs-preprocessor">.c</span>
default_pmm<span class="hljs-preprocessor">.c</span>
pmm<span class="hljs-preprocessor">.c</span>
swap_fifo<span class="hljs-preprocessor">.c</span>
vmm<span class="hljs-preprocessor">.c</span>
trap<span class="hljs-preprocessor">.c</span>
sche<span class="hljs-preprocessor">.c</span></code></pre>
<h1 id="练习1-理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题">练习1 理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题</h1>
<h2 id="0x0-哲学家问题">0x0 哲学家问题</h2>
<pre class="prettyprint"><code class=" hljs ">哲学家就餐问题，即有五个哲学家,他们的生活方式是交替地进行思考和进餐。哲学家们公用一张圆桌,
周围放有五把椅子,每人坐一把。在圆桌上有五个碗和五根筷子,当一个哲学家思考时,他不与其他人交
谈,饥饿时便试图取用其左、右最靠近他的筷子,但他可能一根都拿不到。只有在他拿到两根筷子时,方
能进餐,进餐完后,放下筷子又继续思考。</code></pre>
<h2 id="0x1-信号量介绍">0x1 信号量介绍</h2>
<pre class="prettyprint"><code class=" hljs axapta">struct semaphore {
<span class="hljs-keyword">int</span> <span class="hljs-keyword">count</span>;
queueType queue;
};

<span class="hljs-keyword">void</span> P(semaphore S){
  S.<span class="hljs-keyword">count</span>--；
  <span class="hljs-keyword">if</span> (S.<span class="hljs-keyword">count</span>&lt;<span class="hljs-number">0</span>) {
  把进程置为睡眠态；
  将进程的PCB插入到S.queue的队尾；
  调度，让出CPU；
  }
}

<span class="hljs-keyword">void</span> V(semaphore S){
  S.<span class="hljs-keyword">count</span>++；
  <span class="hljs-keyword">if</span> (S.<span class="hljs-keyword">count</span>≤<span class="hljs-number">0</span>) {
  唤醒在S.queue上等待的第一个进程；
  }
}</code></pre>
<blockquote>
<p>基于上诉信号量实现可以认为，当多个（&gt;1）进程可以进行互斥或同步合作时，一个进程会由于无法满足信号量设置的某条件而在某一位置停止，直到它接收到一个特定的信号（表明条件满足了）。为了发信号，需要使用一个称作信号量的特殊变量。为通过信号量s传送信号，信号量的V操作采用进程可执行原语semSignal(s)；为通过信号量s接收信号，信号量的P操作采用进程可执行原语semWait(s)；如果相应的信号仍然没有发送，则进程被阻塞或睡眠，直到发送完为止。</p>
</blockquote>
<h2 id="0x2-p操作v操作实现">0x2 P操作&amp;V操作实现</h2>
<p><strong>P操作</strong></p>
<blockquote>
<p>具体实现信号量的P操作，首先关掉中断，然后判断当前信号量的value是否大于0。如果是&gt;0，则表明可以获得信号量，故让value减一，并打开中断返回即可；如果不是&gt;0，则表明无法获得信号量，故需要将当前的进程加入到等待队列中，并打开中断，然后运行调度器选择另外一个进程执行。如果被V操作唤醒，则把自身关联的wait从等待队列中删除（此过程需要先关中断，完成后开中断）。具体实现如下所示：</p>
</blockquote>
<pre class="prettyprint"><code class=" hljs lasso">static __noinline uint32_t __down(semaphore_t <span class="hljs-subst">*</span>sem, uint32_t wait_state) {
    bool intr_flag;
    local_intr_save(intr_flag);  <span class="hljs-comment">//关掉中断</span>
    <span class="hljs-keyword">if</span> (sem<span class="hljs-subst">-&gt;</span>value <span class="hljs-subst">&gt;</span> <span class="hljs-number">0</span>) {<span class="hljs-comment">//当前信号量value大于0</span>
        sem<span class="hljs-subst">-&gt;</span>value <span class="hljs-subst">--</span>;<span class="hljs-comment">//直接让value减一</span>
        local_intr_restore(intr_flag);<span class="hljs-comment">//开中断返回</span>
        <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
    }
    <span class="hljs-comment">//当前信号量value小于等于0，表明无法获得信号量</span>
    wait_t __wait, <span class="hljs-subst">*</span>wait <span class="hljs-subst">=</span> <span class="hljs-subst">&amp;</span>__wait;
    wait_current_set(<span class="hljs-subst">&amp;</span>(sem<span class="hljs-subst">-&gt;</span>wait_queue), wait, wait_state);<span class="hljs-comment">//将当前的进程加入到等待队列中</span>
    local_intr_restore(intr_flag);<span class="hljs-comment">//打开中断</span>

    schedule();<span class="hljs-comment">//运行调度器选择其他进程执行</span>

    local_intr_save(intr_flag);<span class="hljs-comment">//关中断</span>
    wait_current_del(<span class="hljs-subst">&amp;</span>(sem<span class="hljs-subst">-&gt;</span>wait_queue), wait);<span class="hljs-comment">//被V操作唤醒，从等待队列移除</span>
    local_intr_restore(intr_flag);<span class="hljs-comment">//开中断</span>

    <span class="hljs-keyword">if</span> (wait<span class="hljs-subst">-&gt;</span>wakeup_flags <span class="hljs-subst">!=</span> wait_state) {
        <span class="hljs-keyword">return</span> wait<span class="hljs-subst">-&gt;</span>wakeup_flags;
    }
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}</code></pre>
<p><strong>V操作</strong></p>
<blockquote>
<p>具体实现信号量的V操作，首先关中断，如果信号量对应的wait queue中没有进程在等待，直接把信号量的value加一，然后开中断返回；如果有进程在等待且进程等待的原因是semophore设置的，则调用wakeup_wait函数将waitqueue中等待的第一个wait删除，且把此wait关联的进程唤醒，最后开中断返回。具体实现如下所示：</p>
</blockquote>
<pre class="prettyprint"><code class=" hljs lasso">static __noinline <span class="hljs-literal">void</span> __up(semaphore_t <span class="hljs-subst">*</span>sem, uint32_t wait_state) {
    bool intr_flag;
    local_intr_save(intr_flag);<span class="hljs-comment">//关闭中断</span>

    {
        wait_t <span class="hljs-subst">*</span>wait;
        <span class="hljs-keyword">if</span> ((wait <span class="hljs-subst">=</span> wait_queue_first(<span class="hljs-subst">&amp;</span>(sem<span class="hljs-subst">-&gt;</span>wait_queue))) <span class="hljs-subst">==</span> <span class="hljs-built_in">NULL</span>) {<span class="hljs-comment">//没有进程等待</span>
            sem<span class="hljs-subst">-&gt;</span>value <span class="hljs-subst">++</span>;<span class="hljs-comment">//信号量的value加一</span>
        }
        <span class="hljs-keyword">else</span> {<span class="hljs-comment">//有进程在等待</span>
            assert(wait<span class="hljs-subst">-&gt;</span>proc<span class="hljs-subst">-&gt;</span>wait_state <span class="hljs-subst">==</span> wait_state);
            wakeup_wait(<span class="hljs-subst">&amp;</span>(sem<span class="hljs-subst">-&gt;</span>wait_queue), wait, wait_state, <span class="hljs-number">1</span>);<span class="hljs-comment">//将`wait_queue`中等待的第一个wait删除，并将该进程唤醒</span>
        }
    }
    local_intr_restore(intr_flag);<span class="hljs-comment">//开启中断返回</span>
}</code></pre>
<h2 id="0x3-代码分析">0x3 代码分析</h2>
<p><code>check_sec</code> <br>
<strong>第一部分是实现基于信号量的哲学家问题,第二部分是实现基于管程的哲学家问题</strong></br></p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">void</span> check_sync(<span class="hljs-keyword">void</span>){

    <span class="hljs-keyword">int</span> i;

    <span class="hljs-comment">//check semaphore</span>
    sem_init(&amp;mutex, <span class="hljs-number">1</span>);
    <span class="hljs-keyword">for</span>(i=<span class="hljs-number">0</span>;i&lt;N;i++){d
        sem_init(&amp;s[i], <span class="hljs-number">0</span>);
        <span class="hljs-keyword">int</span> pid = kernel_thread(philosopher_using_semaphore, (<span class="hljs-keyword">void</span> *)i, <span class="hljs-number">0</span>);
        <span class="hljs-keyword">if</span> (pid &lt;= <span class="hljs-number">0</span>) {
            panic(<span class="hljs-string">"create No.%d philosopher_using_semaphore failed.\n"</span>);
        }
        philosopher_proc_sema[i] = find_proc(pid);
        set_proc_name(philosopher_proc_sema[i], <span class="hljs-string">"philosopher_sema_proc"</span>);
    }

    <span class="hljs-comment">//check condition variable</span>
    monitor_init(&amp;mt, N);
    <span class="hljs-keyword">for</span>(i=<span class="hljs-number">0</span>;i&lt;N;i++){
        state_condvar[i]=THINKING;
        <span class="hljs-keyword">int</span> pid = kernel_thread(philosopher_using_condvar, (<span class="hljs-keyword">void</span> *)i, <span class="hljs-number">0</span>);
        <span class="hljs-keyword">if</span> (pid &lt;= <span class="hljs-number">0</span>) {
            panic(<span class="hljs-string">"create No.%d philosopher_using_condvar failed.\n"</span>);
        }
        philosopher_proc_condvar[i] = find_proc(pid);
        set_proc_name(philosopher_proc_condvar[i], <span class="hljs-string">"philosopher_condvar_proc"</span>);
    }
}</code></pre>
<p>第一部分就是本实验内容</p>
<p>首先实现初始化了一个互斥信号量，然后创建了对应5个哲学家行为的5个信号量，并创建5个内核线程代表5个哲学家，每个内核线程完成了基于信号量的哲学家吃饭睡觉思考行为实现。现在我们继续跟进<code>philosopher_using_semaphore</code>函数观察它的具体实现。</p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">int</span> philosopher_using_semaphore(<span class="hljs-keyword">void</span> * arg) <span class="hljs-comment">/* i：哲学家号码，从0到N-1 */</span>
{
    <span class="hljs-keyword">int</span> i, iter=<span class="hljs-number">0</span>;
    i=(<span class="hljs-keyword">int</span>)arg;
    cprintf(<span class="hljs-string">"I am No.%d philosopher_sema\n"</span>,i);
    <span class="hljs-keyword">while</span>(iter++&lt;TIMES)<span class="hljs-comment">/* 无限循环 */</span>
    {
        cprintf(<span class="hljs-string">"Iter %d, No.%d philosopher_sema is thinking\n"</span>,iter,i); <span class="hljs-comment">// 哲学家正在思考</span>
        do_sleep(SLEEP_TIME);
        phi_take_forks_sema(i); <span class="hljs-comment">// 需要两只叉子，或者阻塞</span>
        cprintf(<span class="hljs-string">"Iter %d, No.%d philosopher_sema is eating\n"</span>,iter,i); <span class="hljs-comment">// 进餐</span>
        do_sleep(SLEEP_TIME);
        phi_put_forks_sema(i); <span class="hljs-comment">// 把两把叉子同时放回桌子</span>
    }
    cprintf(<span class="hljs-string">"No.%d philosopher_sema quit\n"</span>,i);
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}</code></pre>
<p><code>phi_take_forks_sema和phi_put_forks_sema</code></p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">void</span> phi_take_forks_sema(<span class="hljs-keyword">int</span> i) <span class="hljs-comment">/* i：哲学家号码从0到N-1 */</span>
{
        down(&amp;mutex); <span class="hljs-comment">/* 进入临界区 */</span>
        state_sema[i]=HUNGRY; <span class="hljs-comment">/* 记录下哲学家i饥饿的事实 */</span>
        phi_test_sema(i); <span class="hljs-comment">/* 试图得到两只叉子 */</span>
        up(&amp;mutex); <span class="hljs-comment">/* 离开临界区 */</span>
        down(&amp;s[i]); <span class="hljs-comment">/* 如果得不到叉子就阻塞 */</span>
}

<span class="hljs-keyword">void</span> phi_put_forks_sema(<span class="hljs-keyword">int</span> i) <span class="hljs-comment">/* i：哲学家号码从0到N-1 */</span>
{
        down(&amp;mutex); <span class="hljs-comment">/* 进入临界区 */</span>
        state_sema[i]=THINKING; <span class="hljs-comment">/* 哲学家进餐结束 */</span>
        phi_test_sema(LEFT); <span class="hljs-comment">/* 看一下左邻居现在是否能进餐 */</span>
        phi_test_sema(RIGHT); <span class="hljs-comment">/* 看一下右邻居现在是否能进餐 */</span>
        up(&amp;mutex); <span class="hljs-comment">/* 离开临界区 */</span>
}</code></pre>
<h2 id="0x4-信号量性质">0x4 信号量性质</h2>
<p>我们可以看出信号量的计数器value具有有如下性质： <br>
value&gt;0，表示共享资源的空闲数 <br>
vlaue&lt;0，表示该信号量的等待队列里的进程数 <br>
value=0，表示等待队列为空</br></br></br></p>
<h1 id="练习2-完成内核级条件变量和基于内核级条件变量的哲学家就餐问题">练习2 完成内核级条件变量和基于内核级条件变量的哲学家就餐问题</h1>
<h2 id="0x1-管程机制">0x1 管程机制</h2>
<p>即要求首先掌握管程机制,然后基于信号量实现完成条件变量实现,然后用管程机制实现哲学家就餐问题的解决方案。</p>
<p>管程，即定义了一个数据结构和能为并发进程所执行(在该数据结构上)的一组操作,这组操作能同步进程和改变管程中的数据。 <br>
管程相当于一个隔离区，它把共享变量和对它进行操作的若干个过程围了起来，所有进程要访问临界资源时，都必须经过管程才能进入，而管程每次只允许一个进程进入管程,从而需要确保进程之间互斥。 <br>
管程主要由这四个部分组成</br></br></p>
<pre><code>1、管程内部的共享变量;
2、管程内部的条件变量;
3、管程内部并发执行的进程;
4、对局部于管程内部的共享数据设置初始值的语句。
</code></pre>
<p>所谓条件变量，即将等待队列和睡眠条件包装在一起，就形成了一种新的同步机制，称为条件变量。个条件变量CV可理解为一个进程的等待队列,队列中的进程正等待某个条件C变为真。</p>
<p>每个条件变量关联着一个断言Pc。当一个进程等待一个条件变量,该进程不算作占用了该管程,因而其它进程可以进入该管程执行,改变管程的状态,通知条件变量CV其关联的断言Pc在当前状态下为真。</p>
<p>因而条件变量两种操作如下： <br>
- wait_cv: 被一个进程调用,以等待断言Pc被满足后该进程可恢复执行. 进程挂在该条件变量上等待时,不被认为是占用了管程。 <br>
- 被一个进程调用,以指出断言Pc现在为真,从而可以唤醒等待断言Pc被满足的进程继续执行。</br></br></p>
<h2 id="0x2-数据结构">0x2 数据结构</h2>
<p>大概了解了原理之后，接下来我们开始分析具体的代码。 <br>
ucore中的管程机制是基于信号量和条件变量来实现的。管程的数据结构monitor_t如下：</br></p>
<pre class="prettyprint"><code class=" hljs d"><span class="hljs-keyword">typedef</span> <span class="hljs-keyword">struct</span> monitor{
    semaphore_t mutex;      <span class="hljs-comment">// 二值信号量，只允许一个进程进入管程，初始化为1</span>
    semaphore_t next;       <span class="hljs-comment">//配合cv，用于进程同步操作的信号量</span>
    <span class="hljs-keyword">int</span> next_count;         <span class="hljs-comment">// 睡眠的进程数量</span>
    condvar_t *cv;          <span class="hljs-comment">// 条件变量cv</span>
} monitor_t;</code></pre>
<hr>
<p>管程中的条件变量cv通过执行wait_cv，会使得等待某个条件C为真的进程能够离开管程并睡眠，且让其他进程进入管程继续执行；而进入管程的某进程设置条件C为真并执行signal_cv时，能够让等待某个条件C为真的睡眠进程被唤醒，从而继续进入管程中执行。发出signal_cv的进程A会唤醒睡眠进程B，进程B执行会导致进程A睡眠，直到进程B离开管程，进程A才能继续执行，这个同步过程是通过信号量next完成的；而next_count表示了由于发出singal_cv而睡眠的进程个数。</p>
<p>条件变量condvar_t的数据结构如下：</p>
<pre class="prettyprint"><code class=" hljs d"><span class="hljs-keyword">typedef</span> <span class="hljs-keyword">struct</span> condvar{
    semaphore_t sem; <span class="hljs-comment">//用于发出wait_cv操作的等待某个条件C为真的进程睡眠</span>
    <span class="hljs-keyword">int</span> count;       <span class="hljs-comment">// 在这个条件变量上的睡眠进程的个数</span>
    monitor_t * owner; <span class="hljs-comment">// 此条件变量的宿主管程</span>
} condvar_t;</code></pre>
<h2 id="0x3-组成函数">0x3 组成函数</h2>
<hr>
<p><strong>cond_signa函数</strong> <br>
分析完数据结构之后，我们开始分析管程的实现。 <br>
ucore设计实现了条件变量wait_cv操作和signal_cv操作对应的具体函数，即cond_wait函数和cond_signal函数，此外还有cond_init初始化函数。 <br>
先看看cond_signal函数，实现如下：</br></br></br></p>
<pre class="prettyprint"><code class=" hljs lasso"><span class="hljs-literal">void</span>
cond_signal (condvar_t <span class="hljs-subst">*</span>cvp) {
   cprintf(<span class="hljs-string">"cond_signal begin: cvp %x, cvp-&gt;count %d, cvp-&gt;owner-&gt;next_count %d\n"</span>, cvp, cvp<span class="hljs-subst">-&gt;</span>count, cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next_count);
     <span class="hljs-keyword">if</span>(cvp<span class="hljs-subst">-&gt;</span>count<span class="hljs-subst">&gt;</span><span class="hljs-number">0</span>) {  <span class="hljs-comment">//当前存在睡眠的进程</span>
        cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next_count <span class="hljs-subst">++</span>;<span class="hljs-comment">//睡眠的进程总数加一</span>
        up(<span class="hljs-subst">&amp;</span>(cvp<span class="hljs-subst">-&gt;</span>sem));<span class="hljs-comment">//唤醒等待在cv.sem上睡眠的进程</span>
        down(<span class="hljs-subst">&amp;</span>(cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next));<span class="hljs-comment">//把自己睡眠</span>
        cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next_count <span class="hljs-subst">--</span>;<span class="hljs-comment">//睡醒后等待此条件的睡眠进程个数减一</span>
      }
   cprintf(<span class="hljs-string">"cond_signal end: cvp %x, cvp-&gt;count %d, cvp-&gt;owner-&gt;next_count %d\n"</span>, cvp, cvp<span class="hljs-subst">-&gt;</span>count, cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next_count);
}</code></pre>
<hr>
<p><strong>cond_wait函数</strong> <br>
首先进程B判断cv.count，如果不大于0，则表示当前没有睡眠的进程，因此就没有被唤醒的对象了，直接函数返回即可； <br>
如果大于0，这表示当前有睡眠的进程A，因此需要唤醒等待在cv.sem上睡眠的进程A。由于只允许一个进程在管程中执行，所以一旦进程B唤醒了别人（进程A），那么自己就需要睡眠。故让monitor.next_count加一，且让自己（进程B）睡在信号量monitor.next上。如果睡醒了，这让monitor.next_count减一。</br></br></p>
<p>同样，再来看看cond_wait函数，实现如下：</p>
<pre class="prettyprint"><code class=" hljs lasso"><span class="hljs-literal">void</span>
cond_wait (condvar_t <span class="hljs-subst">*</span>cvp) {
    <span class="hljs-comment">//LAB7 EXERCISE1: YOUR CODE</span>
    cprintf(<span class="hljs-string">"cond_wait begin:  cvp %x, cvp-&gt;count %d, cvp-&gt;owner-&gt;next_count %d\n"</span>, cvp, cvp<span class="hljs-subst">-&gt;</span>count, cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next_count);
      cvp<span class="hljs-subst">-&gt;</span>count<span class="hljs-subst">++</span>;<span class="hljs-comment">//需要睡眠的进程个数加一</span>
      <span class="hljs-keyword">if</span>(cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next_count <span class="hljs-subst">&gt;</span> <span class="hljs-number">0</span>)
         up(<span class="hljs-subst">&amp;</span>(cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next));<span class="hljs-comment">//唤醒进程链表中的下一个进程</span>
      <span class="hljs-keyword">else</span>
         up(<span class="hljs-subst">&amp;</span>(cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>mutex));<span class="hljs-comment">//否则唤醒睡在monitor.mutex上的进程</span>
      down(<span class="hljs-subst">&amp;</span>(cvp<span class="hljs-subst">-&gt;</span>sem));<span class="hljs-comment">//将自己睡眠</span>
      cvp<span class="hljs-subst">-&gt;</span>count <span class="hljs-subst">--</span>;<span class="hljs-comment">//睡醒后等待此条件的睡眠进程个数减一</span>
    cprintf(<span class="hljs-string">"cond_wait end:  cvp %x, cvp-&gt;count %d, cvp-&gt;owner-&gt;next_count %d\n"</span>, cvp, cvp<span class="hljs-subst">-&gt;</span>count, cvp<span class="hljs-subst">-&gt;</span>owner<span class="hljs-subst">-&gt;</span>next_count);
}</code></pre>
<hr>
<p>可以看出如果进程A执行了cond_wait函数，表示此进程等待某个条件C不为真，需要睡眠。因此表示等待此条件的睡眠进程个数cv.count要加一。接下来会出现两种情况。 <br>
情况一：如果monitor.next_count如果大于0，表示有大于等于1个进程执行cond_signal函数且睡着了，就睡在了monitor.next信号量上。假定这些进程形成S进程链表。因此需要唤醒S进程链表中的一个进程B。然后进程A睡在cv.sem上，如果睡醒了，则让cv.count减一，表示等待此条件的睡眠进程个数少了一个，可继续执行。 <br>
情况二：如果monitor.next_count如果小于等于0，表示目前没有进程执行cond_signal函数且睡着了，那需要唤醒的是由于互斥条件限制而无法进入管程的进程，所以要唤醒睡在monitor.mutex上的进程。然后进程A睡在cv.sem上，如果睡醒了，则让cv.count减一，表示等待此条件的睡眠进程个数少了一个，可继续执行了！</br></br></p>
<p>这样我们就可以在此基础上继续完成哲学家就餐问题的解决了，主要是就是如下的两个函数：</p>
<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">void</span> phi_take_forks_condvar(<span class="hljs-keyword">int</span> i) {
     down(&amp;(mtp-&gt;mutex));  <span class="hljs-comment">//通过P操作进入临界区</span>
      state_condvar[i]=HUNGRY; <span class="hljs-comment">//记录下哲学家i是否饥饿，即处于等待状态拿叉子</span>
      phi_test_condvar(i);
      <span class="hljs-keyword">while</span> (state_condvar[i] != EATING) {
          cprintf(<span class="hljs-string">"phi_take_forks_condvar: %d didn't get fork and will wait\n"</span>,i);
          cond_wait(&amp;mtp-&gt;cv[i]);<span class="hljs-comment">//如果得不到叉子就睡眠</span>
      }
      <span class="hljs-comment">//如果存在睡眠的进程则那么将之唤醒</span>
      <span class="hljs-keyword">if</span>(mtp-&gt;next_count&gt;<span class="hljs-number">0</span>)
         up(&amp;(mtp-&gt;next));
      <span class="hljs-keyword">else</span>
         up(&amp;(mtp-&gt;mutex));
}

<span class="hljs-keyword">void</span> phi_put_forks_condvar(<span class="hljs-keyword">int</span> i) {
     down(&amp;(mtp-&gt;mutex));<span class="hljs-comment">//通过P操作进入临界区</span>

      state_condvar[i]=THINKING;<span class="hljs-comment">//记录进餐结束的状态</span>
      phi_test_condvar(LEFT);<span class="hljs-comment">//看一下左边哲学家现在是否能进餐</span>
      phi_test_condvar(RIGHT);<span class="hljs-comment">//看一下右边哲学家现在是否能进餐</span>
      <span class="hljs-comment">//如果有哲学家睡眠就予以唤醒</span>
     <span class="hljs-keyword">if</span>(mtp-&gt;next_count&gt;<span class="hljs-number">0</span>)
        up(&amp;(mtp-&gt;next));
     <span class="hljs-keyword">else</span>
        up(&amp;(mtp-&gt;mutex));
}</code></pre>
<h1 id="实验截图">实验截图</h1>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630214154220?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170630214249392?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""> <br>
实验成功</br></img></p>
<h1 id="实验感悟">实验感悟</h1>
<p>本次实验使我对互斥以及同步有了更深的认识， 实验七提供了多种同步互斥手段，包括中断控制、等待队列、信号量、管程机制（包含条件变量设计）等，并基于信号量实现了哲学家问题的执行过程。而练习是要求用管程机制实现哲学家问题的执行过程。使得学到的知识得到了实践。下一步还是要提高自己的动手能力。  </p></hr></hr></hr></hr></div>