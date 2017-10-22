---
title: 操作系统实验报告 lab1
tags: [操作系统实验]
date: 2017-03-18 10:32
---
操作系统实验报告lab1
<!-- more -->
<link rel="stylesheet" type="text/css" href="http://static.blog.csdn.net/css/csdn_blog_detail.min.css">
<div class="markdown_views"><hr>
<h1 id="练习1">练习1</h1>
<h3 id="11-操作系统镜像文件-ucoreimg-是如何一步一步生成的需要比较详细地解释-makefile-中每一条相关命令和命令参数的含义以及说明命令导致的结果">1.1 操作系统镜像文件 ucore.img 是如何一步一步生成的?(需要比较详细地解释 Makefile 中每一条相关命令和命令参数的含义,以及说明命令导致的结果)</h3>
<p>利用make V= 查看执行了那些命令</p>
<h3 id="生成ucoreimg的代码如下">生成ucore.img的代码如下</h3>
<pre class="prettyprint"><code class="language-makefile hljs ">$(UCOREIMG): $(kernel) $(bootblock)
    $(V)dd if=/dev/zero of=$@ count=10000
    $(V)dd if=$(bootblock) of=$@ conv=notrunc
    $(V)dd if=$(kernel) of=$@ seek=1 conv=notrunc
    $(call create_target,ucore.img)</code></pre>
<p>输出如下图</p>
<p><img alt="![enter description here][1]" src="http://img.blog.csdn.net/20170314024312606?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>指令： <br>
  dd：用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。 <br>
   if=文件名：输入文件名，缺省为标准输入。即指定源文件。&lt; if=input file &gt; <br>
  of=文件名：输出文件名，缺省为标准输出。即指定目的文件。&lt; of=output file &gt; <br>
   count=blocks：仅拷贝blocks个块，块大小等于ibs指定的字节数。 <br>
   conv=conversion：用指定的参数转换文件。 <br>
  conv=notrunc:不截短输出文件</br></br></br></br></br></br></p>
</blockquote>
<p>由上描述可以看出，首先先创建一个大小为10000字节的块，然后再将bootblock，kernel拷贝过去。然而生成ucore.img需要先生成kernel和bootblock</p>
<h4 id="1生成bootblock的相关代码如下">1.生成bootblock的相关代码如下</h4>
<pre class="prettyprint"><code class="language-makefile hljs ">$(bootblock): $(call toobj,$(bootfiles)) | $(call totarget,sign) 
    @echo "========================$(call toobj,$(bootfiles))"
    @echo + ld $@
    $(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)
    @$(OBJDUMP) -S $(call objfile,bootblock) &gt; $(call asmfile,bootblock)
    @$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
    @$(call totarget,sign) $(call outfile,bootblock) $(bootblock)</code></pre>
<p>由上代码可得，到要生成bootblock，首先需要生成bootasm.o、bootmain.o、sign <br>
下图是在编译时生成的中间文件</br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170316101225845?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<ol>
<li>生成bootasm.o、bootmain.o、sign的相关代码为：</li>
</ol>
<p><img alt="![enter description here][3]" src="http://img.blog.csdn.net/20170316101349378?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<blockquote>
<p>其中相关参数的含义为： <br>
  ggdb 生成可供gdb使用的调试信息 <br>
  -m32生成适用于32位环境的代码 <br>
  -gstabs 生成stabs格式的调试信息 <br>
  -nostdinc 不使用标准库 <br>
  -fno-stack-protector 不生成用于检测缓冲区溢出的代码 <br>
  -0s 位减小代码长度进行优化</br></br></br></br></br></br></p>
</blockquote>
<p>拷贝二进制代码bootblock.o到bootblock.out <br>
objcopy -S -O binary obj/bootblock.o obj/bootblock.out <br>
其中关键的参数为 <br>
-S  移除所有符号和重定位信息 <br>
-O   指定输出格式 <br>
使用sign工具处理bootblock.out，生成bootblock <br>
 bin/sign obj/bootblock.out bin/bootblock</br></br></br></br></br></br></p>
<h4 id="title"> </h4>
<pre class="prettyprint"><code class="language-makefile hljs "><span class="hljs-constant">kernel</span> = <span class="hljs-variable">$(call totarget,kernel)</span>

$(kernel): tools/kernel.ld

$(kernel): $(KOBJS)
    @echo + ld $@
    $(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)
    @$(OBJDUMP) -S $@ &gt; $(call asmfile,kernel)
    @$(OBJDUMP) -t $@ | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' &gt; $(call symfile,kernel)

$(call create_target,kernel)</code></pre>
<p>查看命令，生成kernel需要以下文件：</p>
<pre class="prettyprint"><code class="language-makefile hljs ">ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o</code></pre>
<h3 id="12-一个被系统认为是符合规范的硬盘主引导扇区的特征是什么">1.2 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？</h3>
<p>查看sign.c源代码</p>
<pre class="prettyprint"><code class="language-makefile hljs ">buf[510] = 0x55;
buf[511] = 0xAA;
FILE *ofp = fopen(argv[2], "wb+");
<span class="hljs-constant">size</span> = fwrite(buf, 1, 512, ofp);
if (size != 512) {
    fprintf(stderr, "write '%s' error, size is %d.\n", argv[2], size);
    return -1;
}</code></pre>
<p>从上述代码可以看出，要求硬盘主引导扇区的大小是512字节，还需要第510个字节是0x55,第511个字节为0xAA,也就是说扇区的最后两个字节内容是0x55AA</p>
<hr>
<h1 id="练习2">练习2</h1>
<blockquote>
<p>题目要求： <br>
  从 CPU加电后执行的第一条指令开始，单步跟踪 BIOS的执行。 <br>
  在初始化位置 0x7c00 设置实地址断点,测试断点正常。 <br>
  从 0x7c00 开始跟踪代码运行,将单步跟踪反汇编得到的代码与 bootasm.S和 bootblock.asm进行比较。 <br>
  自己找一个 bootloader或内核中的代码位置，设置断点并进行测试</br></br></br></br></p>
</blockquote>
<h3 id="21从-cpu加电后执行的第一条指令开始单步跟踪-bios的执行">2.1从 CPU加电后执行的第一条指令开始，单步跟踪 BIOS的执行。</h3>
<p>1 修改 lab1/tools/gdbinit,内容为:</p>
<pre class="prettyprint"><code class=" hljs dos"><span class="hljs-keyword">set</span> architecture i8086
target <span class="hljs-comment">remote :1234</span></code></pre>
<p>2.在 lab1目录下，执行make debug <br>
执行命令如下图</br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318102314860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>3.设置单步调试si <br>
4.在gdb界面下，可通过如下命令来看BIOS的代码</br></p>
<pre class="prettyprint"><code class=" hljs php"> x /<span class="hljs-number">2</span>i <span class="hljs-variable">$pc</span>  <span class="hljs-comment">//显示当前eip处的汇编指令</span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318102504260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>### 2.2 在初始化位置0x7c00设置实地址断点,测试断点正常 <br>
  在tools/gdbinit结尾加上</br></p>
<pre class="prettyprint"><code class=" hljs lasso">    <span class="hljs-built_in">set</span> architecture i8086  <span class="hljs-comment">//设置当前调试的CPU是8086</span>
    b <span class="hljs-subst">*</span><span class="hljs-number">0x7c00</span>  <span class="hljs-comment">//在0x7c00处设置断点。此地址是bootloader入口点地址，可看boot/bootasm.S的start地址处</span>
    c          <span class="hljs-comment">//continue简称，表示继续执行</span>
    x /<span class="hljs-number">2</span>i <span class="hljs-variable">$pc</span>  <span class="hljs-comment">//显示当前eip处的汇编指令</span>
    <span class="hljs-built_in">set</span> architecture i386  <span class="hljs-comment">//设置当前调试的CPU是80386</span></code></pre>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318102618909?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>所以断点正常</p>
<h3 id="23-从0x7c00开始跟踪代码运行将单步跟踪反汇编得到的代码与bootasms和-bootblockasm进行比较">2.3 从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较</h3>
<p>在tools/gdbinit结尾加上</p>
<pre class="prettyprint"><code class=" hljs perl">    b <span class="hljs-variable">*0x7c00</span>
    c
    <span class="hljs-keyword">x</span> /<span class="hljs-number">10</span>i <span class="hljs-variable">$pc</span></code></pre>
<p>在0x7c00处break，然后使用si和 x/i $pc 指令一行一行的跟踪，将得到的反汇编代码为：</p>
<pre class="prettyprint"><code class="language-x86asm hljs mel"><span class="hljs-number">0x00007c01</span> <span class="hljs-keyword">in</span> ?? ()
(gdb) x/i <span class="hljs-variable">$pc</span>
=&gt; <span class="hljs-number">0x7c01</span>:      cld    
(gdb) si
<span class="hljs-number">0x00007c02</span> <span class="hljs-keyword">in</span> ?? ()
(gdb) x/i <span class="hljs-variable">$pc</span>
=&gt; <span class="hljs-number">0x7c02</span>:      xor    <span class="hljs-variable">%eax</span>,<span class="hljs-variable">%eax</span>
(gdb) si
<span class="hljs-number">0x00007c04</span> <span class="hljs-keyword">in</span> ?? ()
(gdb) x/i <span class="hljs-variable">$pc</span>
=&gt; <span class="hljs-number">0x7c04</span>:      mov    <span class="hljs-variable">%eax</span>,<span class="hljs-variable">%ds</span>
(gdb) </code></pre>
<p>bootblock.S 中的代码为：</p>
<pre class="prettyprint"><code class="language-avrasm hljs "><span class="hljs-preprocessor">.code</span>16                                             <span class="hljs-preprocessor"># Assemble for 16-bit mode</span>
    <span class="hljs-keyword">cli</span>                                             <span class="hljs-preprocessor"># Disable interrupts</span>
    cld                                             <span class="hljs-preprocessor"># String operations increment</span>

    <span class="hljs-preprocessor"># Set up the important data segment registers (DS, ES, SS).</span>
    xorw %ax, %ax                                   <span class="hljs-preprocessor"># Segment number zero</span>
    <span class="hljs-keyword">movw</span> %ax, %ds                                   <span class="hljs-preprocessor"># -&gt; Data Segment</span>
    <span class="hljs-keyword">movw</span> %ax, %es                                   <span class="hljs-preprocessor"># -&gt; Extra Segment</span>
    <span class="hljs-keyword">movw</span> %ax, %ss                                   <span class="hljs-preprocessor"># -&gt; Stack Segment</span>

    <span class="hljs-preprocessor"># Enable A20:</span>
    <span class="hljs-preprocessor">#  For backwards compatibility with the earliest PCs, physical</span>
    <span class="hljs-preprocessor">#  address line 20 is tied low, so that addresses higher than</span>
    <span class="hljs-preprocessor">#  1MB wrap around to zero by default. This code undoes this.</span></code></pre>
<p>bootblock.asm</p>
<pre class="prettyprint"><code class="language-avrasm hljs "><span class="hljs-label">start:</span>
<span class="hljs-preprocessor">.code</span>16                                             <span class="hljs-preprocessor"># Assemble for 16-bit mode</span>
    <span class="hljs-keyword">cli</span>                                             <span class="hljs-preprocessor"># Disable interrupts</span>
    <span class="hljs-number">7</span>c00:   fa                      <span class="hljs-keyword">cli</span>    
    cld                                             <span class="hljs-preprocessor"># String operations increment</span>
    <span class="hljs-number">7</span>c01:   fc                      cld    

    <span class="hljs-preprocessor"># Set up the important data segment registers (DS, ES, SS).</span>
    xorw %ax, %ax                                   <span class="hljs-preprocessor"># Segment number zero</span>
    <span class="hljs-number">7</span>c02:   <span class="hljs-number">31</span> c0                   xor    %eax,%eax
    <span class="hljs-keyword">movw</span> %ax, %ds                                   <span class="hljs-preprocessor"># -&gt; Data Segment</span>
    <span class="hljs-number">7</span>c04:   <span class="hljs-number">8</span>e d8                   <span class="hljs-keyword">mov</span>    %eax,%ds
    <span class="hljs-keyword">movw</span> %ax, %es                                   <span class="hljs-preprocessor"># -&gt; Extra Segment</span>
    <span class="hljs-number">7</span>c06:   <span class="hljs-number">8</span>e c0                   <span class="hljs-keyword">mov</span>    %eax,%es
    <span class="hljs-keyword">movw</span> %ax, %ss                                   <span class="hljs-preprocessor"># -&gt; Stack Segment</span>
    <span class="hljs-number">7</span>c08:   <span class="hljs-number">8</span>e d0                   <span class="hljs-keyword">mov</span>    %eax,%ss
</code></pre>
<p>观察发现他们相同</p>
<hr>
<h1 id="练习3">练习3</h1>
<blockquote>
<p>题目： <br>
  分析bootloader 进入保护模式的过程。 <br>
  BIOS 将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行 bootloader。请分析bootloader是如何完成从实模式进入保护模式的</br></br></p>
</blockquote>
<p>从bootasm.s查看代码(在这里分析bootblock.asm也可以，二者源码相同)，并分析过程</p>
<h3 id="宏定义">宏定义</h3>
<pre class="prettyprint"><code class="language-vbnet hljs ">.<span class="hljs-keyword">set</span> PROT_MODE_CSEG,        <span class="hljs-number">0x8</span>                    <span class="hljs-preprocessor">#内核代码段选择子  </span>
.<span class="hljs-keyword">set</span> PROT_MODE_DSEG,        <span class="hljs-number">0x10</span>                  <span class="hljs-preprocessor">#内核数据段选择子  </span>
.<span class="hljs-keyword">set</span> CR0_PE_ON,             <span class="hljs-number">0x1</span>                              <span class="hljs-preprocessor">#保护模式使能标志  </span></code></pre>
<h3 id="1关闭中断将各个段寄存器重置">1.关闭中断，将各个段寄存器重置</h3>
<p>修改控制方向标志寄存器DF=0，使得内存地址从低到高增加 <br>
它先将各个寄存器置0</br></p>
<pre class="prettyprint"><code class="language-x86asm hljs perl"><span class="hljs-comment">#CPU刚启动为16位模式 </span>
    cli               <span class="hljs-comment"># 关中断</span>
    cld               <span class="hljs-comment"># 清方向标志  </span>
    xorw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%ax</span>     <span class="hljs-comment"># 置零</span>
    movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%ds</span>     <span class="hljs-comment"># -&gt; 数据段寄存器</span>
    movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%es</span>     <span class="hljs-comment"># -&gt; 附加段寄存器</span>
    movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%ss</span>     <span class="hljs-comment"># -&gt; 堆栈段寄存器</span></code></pre>
<h3 id="2-开启a20">2 .开启A20</h3>
<p>开启A20地址线之后，用来表示内存地址的位数变多了。开启前20位，开启后是32位。如果不开启A20地址线内存寻址最大只能找到1M，对于1M以上的地址访问会变成对address mod 1M地址的访问。通过将键盘控制器上的A20线置于高电位，全部32条地址线可用，可以访问4G的内存空间。 <br>
打开A20地址线  为了兼容早期的PC机，第20根地址线在实模式下不能使用所以超过1MB的地址，默认就会返回到地址0，重新从0循环计数，下面的代码打开A20地址线  。</br></p>
<pre class="prettyprint"><code class="language-x86asm hljs perl">seta2<span class="hljs-number">0</span>.<span class="hljs-number">1</span>:
    inb <span class="hljs-variable">$0</span>x64, <span class="hljs-variable">%al</span>    <span class="hljs-comment"># 从0x64端口读入一个字节的数据到al中  </span>
    testb <span class="hljs-variable">$0</span>x2, <span class="hljs-variable">%al</span>   <span class="hljs-comment"># test指令可以当作and指令，只不过它不会影响操作数 </span>
    jnz seta2<span class="hljs-number">0</span>.<span class="hljs-number">1</span><span class="hljs-comment">#如果上面的测试中发现al的第2位为0，就不执行该指令  </span>
否则就循环检查  

    movb <span class="hljs-variable">$0</span>xd1, <span class="hljs-variable">%al</span>    <span class="hljs-comment"># 将0xd1写入到al中 </span>
    outb    <span class="hljs-variable">%al</span>,<span class="hljs-variable">$0</span>x64  <span class="hljs-comment">#将al中的数据写入到端口0x64中  </span>

seta2<span class="hljs-number">0</span>.<span class="hljs-number">2</span>:
    inb <span class="hljs-variable">$0</span>x64, <span class="hljs-variable">%al</span>    
    testb <span class="hljs-variable">$0</span>x2, <span class="hljs-variable">%al</span>
    jnz seta2<span class="hljs-number">0</span>.<span class="hljs-number">2</span>

    movb <span class="hljs-variable">$0</span>xdf, <span class="hljs-variable">%al</span>   <span class="hljs-comment"># 通过0x60写入数据11011111 即将A20置1</span>
    outb <span class="hljs-variable">%al</span>, <span class="hljs-variable">$0</span>x6<span class="hljs-number">0</span></code></pre>
<p>初始化GDT表：一个简单的GDT表和其描述符已经静态储存在引导区中，载入即可</p>
<pre class="prettyprint"><code class=" hljs bash">        lgdt gdtdesc <span class="hljs-comment">#将全局描述符表描述符加载到全局描述符表寄存器  </span></code></pre>
<p>进入保护模式：通过将cr0寄存器PE位置1便开启了保护模式</p>
<pre class="prettyprint"><code class="language-x86asm hljs perl">cr<span class="hljs-number">0</span>中的第<span class="hljs-number">0</span>位为<span class="hljs-number">1</span>表示处于保护模式  
cr<span class="hljs-number">0</span>中的第<span class="hljs-number">0</span>位为<span class="hljs-number">0</span>，表示处于实模式  
把控制寄存器cr<span class="hljs-number">0</span>加载到eax中  


movl <span class="hljs-variable">%cr0</span>, <span class="hljs-variable">%eax</span>
orl <span class="hljs-variable">$CR0_PE_ON</span>, <span class="hljs-variable">%eax</span>
movl <span class="hljs-variable">%eax</span>, <span class="hljs-variable">%cr0</span></code></pre>
<p>通过长跳转更新cs的基地址</p>
<pre class="prettyprint"><code class=" hljs bash">ljmp <span class="hljs-variable">$PROT_MODE_CSEG</span>, <span class="hljs-variable">$protcseg</span>
.code32
protcseg:</code></pre>
<p>设置段寄存器，并建立堆栈</p>
<pre class="prettyprint"><code class="language-x86asm hljs perl">movw <span class="hljs-variable">$PROT_MODE_DSEG</span>, <span class="hljs-variable">%ax</span>
movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%ds</span>
movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%es</span>
movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%fs</span>
movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%gs</span>
movw <span class="hljs-variable">%ax</span>, <span class="hljs-variable">%ss</span>
movl <span class="hljs-variable">$0</span><span class="hljs-keyword">x</span><span class="hljs-number">0</span>, <span class="hljs-variable">%ebp</span>
movl <span class="hljs-variable">$start</span>, <span class="hljs-variable">%esp</span></code></pre>
<p>转到保护模式完成，进入boot主方法</p>
<pre class="prettyprint"><code class=" hljs sql"><span class="hljs-operator"><span class="hljs-keyword">call</span> bootmain</span></code></pre>
<hr>
<h1 id="练习4">练习4</h1>
<blockquote>
<p>题目： <br>
  分析bootloader加载ELF格式的OS的过程 <br>
  1. bootloader如何读取硬盘扇区的？ <br>
  2. bootloader是如何加载 ELF格式的 OS？</br></br></br></p>
</blockquote>
<p>bootmain 代码</p>
<pre class="prettyprint"><code class="language-c++ hljs lasso">bootmain(<span class="hljs-literal">void</span>) {
    readseg((uintptr_t)ELFHDR, SECTSIZE <span class="hljs-subst">*</span> <span class="hljs-number">8</span>, <span class="hljs-number">0</span>);
    <span class="hljs-keyword">if</span> (ELFHDR<span class="hljs-subst">-&gt;</span>e_magic <span class="hljs-subst">!=</span> ELF_MAGIC) {
        goto bad;
    }
    struct proghdr <span class="hljs-subst">*</span>ph, <span class="hljs-subst">*</span>eph;
    ph <span class="hljs-subst">=</span> (struct proghdr <span class="hljs-subst">*</span>)((uintptr_t)ELFHDR <span class="hljs-subst">+</span> ELFHDR<span class="hljs-subst">-&gt;</span>e_phoff);
    eph <span class="hljs-subst">=</span> ph <span class="hljs-subst">+</span> ELFHDR<span class="hljs-subst">-&gt;</span>e_phnum;
    for (; ph <span class="hljs-subst">&lt;</span> eph; ph <span class="hljs-subst">++</span>) {
        readseg(ph<span class="hljs-subst">-&gt;</span>p_va <span class="hljs-subst">&amp;</span> <span class="hljs-number">0xFFFFFF</span>, ph<span class="hljs-subst">-&gt;</span>p_memsz, ph<span class="hljs-subst">-&gt;</span>p_offset);
    }
    ((<span class="hljs-literal">void</span> (<span class="hljs-subst">*</span>)(<span class="hljs-literal">void</span>))(ELFHDR<span class="hljs-subst">-&gt;</span>e_entry <span class="hljs-subst">&amp;</span> <span class="hljs-number">0xFFFFFF</span>))();
bad:
    outw(<span class="hljs-number">0x8A00</span>, <span class="hljs-number">0x8A00</span>);
    outw(<span class="hljs-number">0x8A00</span>, <span class="hljs-number">0x8E00</span>);
    <span class="hljs-keyword">while</span> (<span class="hljs-number">1</span>);
}</code></pre>
<p><code>readsect</code>从设备的第secno扇区读取数据到dst位置</p>
<pre class="prettyprint"><code class=" hljs cs">    <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span>
    readsect(<span class="hljs-keyword">void</span> *dst, uint32_t secno) {
        waitdisk();

        outb(<span class="hljs-number">0x1F2</span>, <span class="hljs-number">1</span>);                         <span class="hljs-comment">// 设置读取扇区的数目为1</span>
        outb(<span class="hljs-number">0x1F3</span>, secno &amp; <span class="hljs-number">0xFF</span>);
        outb(<span class="hljs-number">0x1F4</span>, (secno &gt;&gt; <span class="hljs-number">8</span>) &amp; <span class="hljs-number">0xFF</span>);
        outb(<span class="hljs-number">0x1F5</span>, (secno &gt;&gt; <span class="hljs-number">16</span>) &amp; <span class="hljs-number">0xFF</span>);
        outb(<span class="hljs-number">0x1F6</span>, ((secno &gt;&gt; <span class="hljs-number">24</span>) &amp; <span class="hljs-number">0xF</span>) | <span class="hljs-number">0xE0</span>);
            <span class="hljs-comment">// 上面四条指令联合制定了扇区号</span>
            <span class="hljs-comment">// 在这4个字节线联合构成的32位参数中</span>
            <span class="hljs-comment">//   29-31位强制设为1</span>
            <span class="hljs-comment">//   28位(=0)表示访问"Disk 0"</span>
            <span class="hljs-comment">//   0-27位是28位的偏移量</span>
        outb(<span class="hljs-number">0x1F7</span>, <span class="hljs-number">0x20</span>);                      <span class="hljs-comment">// 0x20命令，读取扇区</span>

        waitdisk();

        insl(<span class="hljs-number">0x1F0</span>, dst, SECTSIZE / <span class="hljs-number">4</span>);         <span class="hljs-comment">// 读取到dst位置，</span>
                                                <span class="hljs-comment">// 幻数4因为这里以DW为单位</span>
    }</code></pre>
<blockquote>
<p>IO地址   功能 <br>
  0x1f0   读数据，当0x1f7不为忙状态时，可以读。 <br>
  0x1f2   要读写的扇区数，每次读写前，你需要表明你要读写几个扇区。最小是1个扇区 <br>
  0x1f3   如果是LBA模式，就是LBA参数的0-7位 <br>
  0x1f4   如果是LBA模式，就是LBA参数的8-15位 <br>
  0x1f5   如果是LBA模式，就是LBA参数的16-23位 <br>
  0x1f6   第0~3位：如果是LBA模式就是24-27位 第4位：为0主盘；为1从盘 <br>
  0x1f7   状态和命令寄存器。操作时先给命令，再读取，如果不是忙状态就从0x1f0端口读数据</br></br></br></br></br></br></br></p>
</blockquote>
<p>加载ELF文件</p>
<pre class="prettyprint"><code class="language-stylus hljs lasso">bootmain(<span class="hljs-literal">void</span>) {
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span><span class="hljs-attribute">...</span><span class="hljs-built_in">.</span>

    <span class="hljs-keyword">if</span> (ELFHDR<span class="hljs-subst">-&gt;</span>e_magic <span class="hljs-subst">!=</span> ELF_MAGIC) {<span class="hljs-comment">//这里有个判断</span>
        goto bad;                 
    }
    struct proghdr <span class="hljs-subst">*</span>ph, <span class="hljs-subst">*</span>eph;

    <span class="hljs-comment">//ELF头部有描述ELF文件应加载到内存什么位置的描述表，这里读取出来将之存入ph</span>
    ph <span class="hljs-subst">=</span> (struct proghdr <span class="hljs-subst">*</span>)((uintptr_t)ELFHDR <span class="hljs-subst">+</span> ELFHDR<span class="hljs-subst">-&gt;</span>e_phoff);
    eph <span class="hljs-subst">=</span> ph <span class="hljs-subst">+</span> ELFHDR<span class="hljs-subst">-&gt;</span>e_phnum;

    <span class="hljs-comment">//按照程序头表的描述，将ELF文件中的数据载入内存</span>
    for (; ph <span class="hljs-subst">&lt;</span> eph; ph <span class="hljs-subst">++</span>) {
        readseg(ph<span class="hljs-subst">-&gt;</span>p_va <span class="hljs-subst">&amp;</span> <span class="hljs-number">0xFFFFFF</span>, ph<span class="hljs-subst">-&gt;</span>p_memsz, ph<span class="hljs-subst">-&gt;</span>p_offset);


    ((<span class="hljs-literal">void</span> (<span class="hljs-subst">*</span>)(<span class="hljs-literal">void</span>))(ELFHDR<span class="hljs-subst">-&gt;</span>e_entry <span class="hljs-subst">&amp;</span> <span class="hljs-number">0xFFFFFF</span>))();<span class="hljs-comment">//根据ELF头表中的入口信息，找到内核的入口并开始运行</span>
bad:
    <span class="hljs-attribute">...</span><span class="hljs-attribute">...</span><span class="hljs-attribute">...</span><span class="hljs-built_in">.</span>
}</code></pre>
<hr>
<h1 id="练习5">练习5</h1>
<blockquote>
<p>题目： <br>
  实现函数调用堆栈跟踪函数</br></p>
</blockquote>
<h3 id="什么是函数栈">什么是函数栈？</h3>
<p>当函数被调用时，首先会把函数的参数依次入栈（这里指的是堆栈传参，当然也可以用寄存器传参）调用函数的栈底压栈到自己函数的栈中（push bp），然后将原来函数栈顶sp作为当前函数的栈底（mov bp,sp）。函数运行完成时，会将压入栈中的bp重新出栈到bp中（pop bp）。同时将计算的结果保存在寄存器中，返回原界面。</p>
<p>那么我们可以粗浅的建立一个栈模型</p>
<pre class="prettyprint"><code class="language-x86asm hljs css"><span class="hljs-tag">ss</span>:<span class="hljs-attr_selector">[ebp-8]</span>   ;变量2
<span class="hljs-tag">ss</span>:<span class="hljs-attr_selector">[ebp-4]</span>   ;变量1
<span class="hljs-tag">ss</span>:<span class="hljs-attr_selector">[ebp]</span>       ;栈针
<span class="hljs-tag">ss</span>:<span class="hljs-attr_selector">[ebp+4]</span>  ;返回地址
<span class="hljs-tag">ss</span>:<span class="hljs-attr_selector">[ebp+8]</span>   ;第一个参数</code></pre>
<h3 id="函数实现">函数实现</h3>
<p>read_ebp()和read_eip()函数来获取当前ebp寄存器和eip 寄存器的信息。</p>
<p>查看print_stackframe函数注释</p>
<pre class="prettyprint"><code class="language-stylus hljs applescript"> /* LAB1 YOUR CODE : STEP <span class="hljs-number">1</span> */
     /* (<span class="hljs-number">1</span>) call read_ebp() <span class="hljs-keyword">to</span> <span class="hljs-keyword">get</span> <span class="hljs-keyword">the</span> value <span class="hljs-keyword">of</span> ebp. <span class="hljs-keyword">the</span> type <span class="hljs-keyword">is</span> (uint32_t);
      * (<span class="hljs-number">2</span>) call read_eip() <span class="hljs-keyword">to</span> <span class="hljs-keyword">get</span> <span class="hljs-keyword">the</span> value <span class="hljs-keyword">of</span> eip. <span class="hljs-keyword">the</span> type <span class="hljs-keyword">is</span> (uint32_t);
      * (<span class="hljs-number">3</span>) <span class="hljs-keyword">from</span> <span class="hljs-number">0</span> .. STACKFRAME_DEPTH
      *    (<span class="hljs-number">3.1</span>) printf value <span class="hljs-keyword">of</span> ebp, eip
      *    (<span class="hljs-number">3.2</span>) (uint32_t)calling arguments [<span class="hljs-number">0.</span><span class="hljs-number">.4</span>] = <span class="hljs-keyword">the</span> <span class="hljs-property">contents</span> <span class="hljs-keyword">in</span> address (unit32_t)ebp +<span class="hljs-number">2</span> [<span class="hljs-number">0.</span><span class="hljs-number">.4</span>]
      *    (<span class="hljs-number">3.3</span>) cprintf(<span class="hljs-string">"\n"</span>);
      *    (<span class="hljs-number">3.4</span>) call print_debuginfo(eip-<span class="hljs-number">1</span>) <span class="hljs-keyword">to</span> print <span class="hljs-keyword">the</span> C calling function <span class="hljs-property">name</span> <span class="hljs-keyword">and</span> line <span class="hljs-type">number</span>, etc.
      *    (<span class="hljs-number">3.5</span>) popup a calling stackframe
      *           NOTICE: <span class="hljs-keyword">the</span> calling funciton's <span class="hljs-constant">return</span> addr eip  = ss:[ebp+<span class="hljs-number">4</span>]
      *                   <span class="hljs-keyword">the</span> calling funciton's ebp = ss:[ebp]
      */</code></pre>
<pre class="prettyprint"><code class="language-stylus hljs cs"> <span class="hljs-keyword">for</span>(i = <span class="hljs-number">0</span>; ebp!=<span class="hljs-number">0</span> &amp;&amp; i &lt; STACKFRAME_DEPTH; i++) {<span class="hljs-comment">//STACKFRAME_DEPTH = 20 一直向上循环找到所有的调用函数为止，一开始没有判断栈针为空的条件</span>
        cprintf(<span class="hljs-string">"ebp:0x%08x eip:0x%08x "</span>,ebp, eip);
        uint32_t *args = (uint32_t *)ebp + <span class="hljs-number">2</span>; <span class="hljs-comment">//传参</span>
        <span class="hljs-keyword">for</span>(j = <span class="hljs-number">0</span>; j &lt; <span class="hljs-number">4</span>; j++)
            cprintf(<span class="hljs-string">"0x%08x "</span>,args[j]); 
        cprintf(<span class="hljs-string">"\n"</span>);
        print_debuginfo(eip-<span class="hljs-number">1</span>);
        <span class="hljs-comment">//模拟函数执行完毕</span>
        eip = *((uint32_t *)ebp+<span class="hljs-number">1</span>);<span class="hljs-comment">//调用函数的返回地址</span>
        ebp = *((uint32_t *)ebp);<span class="hljs-comment">//上一个函数的栈针 </span>
<span class="hljs-comment">//循环直到没有调用函数停止</span>
    }</code></pre>
<p>执行结果如下： <br>
未加ebp!=0 </br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318102722660?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>我们发现ebp的值在0x7bf8之后就为零了，这说明上面没有了调用函数，直接加个判断 <br>
ebp!=0 就可以输出预期的结果。</br></p>
<p>加了ebp!=0 </p>
<hr>
<h1 id="练习6">练习6</h1>
<blockquote>
<p>题目： <br>
  1.中断向量表中一个表项占多少字节？其中哪几位代表中断处理代码的入口？ <br>
  2.请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中，依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。注意除了系统调用中断(T_SYSCALL)以外，其它中断均使用中断门描述符，权限为内核态权限；而系统调用中断使用异常,权限为陷阱门描述符。每个中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可。 <br>
  3.请编程完善trap.c中的中断处理函数trap在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用 print_ticks子程序，向屏幕上打印一行文字100 ticks。</br></br></br></p>
</blockquote>
<h3 id="1中断向量表中一个表项占多少字节其中哪几位代表中断处理代码的入口">1.中断向量表中一个表项占多少字节？其中哪几位代表中断处理代码的入口？</h3>
<p>中断向量表一个表项占用8字节，其中2-3字节是段选择子，0-1字节和6-7字节拼成位移， <br>
两者联合便是中断处理程序的入口地址。 <br>
中断描述符表中一个表项占8个字节，其中每个位的作用如图：</br></br></p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318102822515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p>
<p>其中0<sub>15和48</sub>63分别为offset的低16位和高16位，16~31位是段选择子，通过段选择子得到段基址，再加上段内偏移量就可以得到中断处理代码的入口。</p>
<p><a href="http://undefined"></a><a href="http://undefined"></a><a href="http://undefined"></a><a href="http://undefined"></a></p>
<h3 id="2请编程完善kerntraptrapc中对中断向量表进行初始化的函数idtinit">2.请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init</h3>
<pre class="prettyprint"><code class="language-c++ hljs scss">   extern uintptr_t __vectors<span class="hljs-attr_selector">[]</span>;<span class="hljs-comment">//声明__vertors[] You can use  "extern uintptr_t __vectors[];" to define this extern variable which will be used later.</span>
    int <span class="hljs-tag">i</span>;
    <span class="hljs-function">for(i=<span class="hljs-number">0</span>;i&lt;<span class="hljs-number">256</span>;i++)</span> {
        <span class="hljs-function">SETGATE(idt[i],<span class="hljs-number">0</span>,GD_KTEXT,__vectors[i],DPL_KERNEL)</span>;<span class="hljs-comment">//对整个idt数组进行初始化</span>
    }
    <span class="hljs-function">SETGATE(idt[T_SWITCH_TOK],<span class="hljs-number">0</span>,GD_KTEXT,__vectors[T_SWITCH_TOK],DPL_USER)</span>;<span class="hljs-comment">//在这里先把所有的中断都初始化为内核级的中断</span>
    <span class="hljs-function">lidt(&amp;idt_pd)</span>;<span class="hljs-comment">//使用lidt指令加载中断描述符表 just google it! and check the libs/x86.h to know more.利用google找到了相关函数</span>
}
<span class="hljs-comment">/*

    传入的第一个参数gate是中断的描述符表
    传入的第二个参数istrap用来判断是中断还是trap
    传入的第三个参数sel的作用是进行段的选择
    传入的第四个参数off表示偏移
    传入的第五个参数dpl表示这个中断的优先级

*/</span></code></pre>
<h3 id="3程完善trapc中的中断处理函数trap在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分使操作系统每遇到100次时钟中断后调用-printticks子程序向屏幕上打印一行文字100-ticks">3.程完善trap.c中的中断处理函数trap在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用 print_ticks子程序，向屏幕上打印一行文字100 ticks</h3>
<p>实验代码填写</p>
<pre class="prettyprint"><code class="language-c++ hljs vbnet">    <span class="hljs-keyword">case</span> IRQ_OFFSET + IRQ_TIMER:
        /* LAB1 YOUR CODE : <span class="hljs-keyword">STEP</span> <span class="hljs-number">3</span> */
        /* handle the timer interrupt */
        /* (<span class="hljs-number">1</span>) After a timer interrupt, you should record this <span class="hljs-keyword">event</span> <span class="hljs-keyword">using</span> a <span class="hljs-keyword">global</span> variable (increase it), such <span class="hljs-keyword">as</span> ticks <span class="hljs-keyword">in</span> kern/driver/clock.c
         * (<span class="hljs-number">2</span>) Every TICK_NUM cycle, you can print some info <span class="hljs-keyword">using</span> a funciton, such <span class="hljs-keyword">as</span> print_ticks().
         * (<span class="hljs-number">3</span>) Too Simple? Yes, I think so!
         */</code></pre>
<p>代码如下：</p>
<pre class="prettyprint"><code class="language-stylus hljs mel">ticks++;
    <span class="hljs-keyword">if</span>(ticks<span class="hljs-variable">%TICK_NUM</span> == <span class="hljs-number">0</span>)<span class="hljs-comment">//每次时钟中断之后ticks就会加一 当加到TICK_NUM次数时 打印并重新开始</span>
    print_ticks();<span class="hljs-comment">//前面有定义 打印字符串</span></code></pre>
<p>实验截图如下：</p>
<p><img alt="这里写图片描述" src="http://img.blog.csdn.net/20170318103052252?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE0ODExODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" title=""/></p></hr></hr></hr></hr></hr></hr></div>