---
title: 手写PE文件分析
tags: [windows]
categories: windows
date: 2018-01-11 16:00
grammar_cjkRuby: true
---

大作业需求。。。。动手 理解一下PE格式
<!-- more -->

# 0x01 程序加载流程
手写的PE文件，没有编译器的编译，没有链接器的链接，所以需要手动的去实现这些功能。在一开始我们先分析一下程序的加载流程，一般是经历解析/装载/链接 三个步骤，最后是执行。
## 0x1 解析
解析PE文件格式，解析PE的信息，将各个节区识别出来，并按照节区头部去装载
## 0x2 装载
将文件按照PE提供的RVA的值去装载，最小对其单位为Section Alignment。
## 0x3 链接
利用PE文件中的输入表，将动态链接函数的地址链接在IAT表里面，实现文件与文件之间的连接。
# 0x02 文件格式分析
我们再次分析一下一个PE文件中应该有什么。
DOS头/PE头/区块表/区块/输入输出表  （按照地址从低到高的顺序）

![][1]

## 0x1 DOS头
每个PE文件是以一个DOS程序的开始的，此部分被称为IMAGE_DOS_HEADER，大小为64字节

``` cpp
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
      WORD   e_magic;                     // Magic number
      WORD   e_cblp;                      // Bytes on last page of file
      WORD   e_cp;                        // Pages in file
      WORD   e_crlc;                      // Relocations
      WORD   e_cparhdr;                   // Size of header in paragraphs
      WORD   e_minalloc;                  // Minimum extra paragraphs needed
      WORD   e_maxalloc;                  // Maximum extra paragraphs needed
      WORD   e_ss;                        // Initial (relative) SS value
      WORD   e_sp;                        // Initial SP value
      WORD   e_csum;                      // Checksum
      WORD   e_ip;                        // Initial IP value
      WORD   e_cs;                        // Initial (relative) CS value
      WORD   e_lfarlc;                    // File address of relocation table
      WORD   e_ovno;                      // Overlay number
      WORD   e_res[4];                    // Reserved words
      WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
      WORD   e_oeminfo;                   // OEM information; e_oemid specific
      WORD   e_res2[10];                  // Reserved words
      DWPRD  e_lfanew;                    // File address of new exe header
    } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```
简单解析一下里面最重要的两个字段e_lfanew和e_magic，分别是PE文件的偏移和DOS头文件的标识符，我们正是利用e_lfanew标识头文件。


----------


那么我们首先设计如下：

``` cpp
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
      WORD   0x5A4D;                     // Magic number
      WORD   0x0000;                      // Bytes on last page of file
      WORD   0x4550;                        // Pages in file
      WORD   e_crlc;                      // Relocations
      WORD   e_cparhdr;                   // Size of header in paragraphs
      WORD   e_minalloc;                  // Minimum extra paragraphs needed
      WORD   e_maxalloc;                  // Maximum extra paragraphs needed
      WORD   e_ss;                        // Initial (relative) SS value
      WORD   e_sp;                        // Initial SP value
      WORD   e_csum;                      // Checksum
      WORD   e_ip;                        // Initial IP value
      WORD   e_cs;                        // Initial (relative) CS value
      WORD   e_lfarlc;                    // File address of relocation table
      WORD   e_ovno;                      // Overlay number
      WORD   e_res[4];                    // Reserved words
      WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
      WORD   e_oeminfo;                   // OEM information; e_oemid specific
      WORD   e_res2[10];                  // Reserved words
      DWPRD  0x00000004;                    // File address of new exe header
    } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```
剩下的字节属于PE文件头，DOS头部和PE头部重合在了一起，有效的节约了程序空间。
## 0x2 PE头
包括三部分Signature/IMAGE_FILE_HEADER/IMAGE_OPTIONAL_HEADER

``` cpp
    typedef struct _IMAGE_NT_HEADERS {  
        DWORD Signature;  
        IMAGE_FILE_HEADER FileHeader;  
        IMAGE_OPTIONAL_HEADER32 OptionalHeader;  
    } IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;  
```


由DOS头部的最后四个字节可以看出，PE文件头在文件偏移的4字节处。

所以e_crlc的数据为IMAGE_FILE_HEADER

``` cpp
typedef struct _IMAGE_FILE_HEADER {  
    WORD    Machine;  
    WORD    NumberOfSections;  
    DWORD   TimeDateStamp;  
    DWORD   PointerToSymbolTable;  
    DWORD   NumberOfSymbols;  
    WORD    SizeOfOptionalHeader;  
    WORD    Characteristics;  
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER; 
```
第一个为Machine，其值可以是以下表格中的内容



![][2]

再往下是IMAGE_OPTIONAL_HEADER，在这个结构中有用的数据比较多。

``` cpp
typedef struct _IMAGE_OPTIONAL_HEADER
{
	// 
	// Standard fields.   
	// 
	WORD    Magic;				// 标志字, ROM 映像（0107h）,普通可执行文件（010Bh） 
	BYTE    MajorLinkerVersion;		// 链接程序的主版本号 
	BYTE    MinorLinkerVersion;		// 链接程序的次版本号 
	DWORD   SizeOfCode;			// 所有含代码的节的总大小 
	DWORD   SizeOfInitializedData;      	// 所有含已初始化数据的节的总大小 
	DWORD   SizeOfUninitializedData;    	// 所有含未初始化数据的节的大小 
	DWORD   AddressOfEntryPoint;		// 程序执行入口RVA 
	DWORD   BaseOfCode;			// 代码的区块的起始RVA 
	DWORD   BaseOfData;			// 数据的区块的起始RVA 
	// 
	// NT additional fields.    以下是属于NT结构增加的领域。 
	// 
	DWORD   ImageBase;			// 程序的首选装载地址 
	DWORD   SectionAlignment;		// 内存中的区块的对齐大小 
	DWORD   FileAlignment;			// 文件中的区块的对齐大小 
	WORD    MajorOperatingSystemVersion;	// 要求操作系统最低版本号的主版本号 
	WORD    MinorOperatingSystemVersion;	// 要求操作系统最低版本号的副版本号 
	WORD    MajorImageVersion;		// 可运行于操作系统的主版本号 
	WORD    MinorImageVersion;		// 可运行于操作系统的次版本号 
	WORD    MajorSubsystemVersion;		// 要求最低子系统版本的主版本号 
	WORD    MinorSubsystemVersion;		// 要求最低子系统版本的次版本号 
	DWORD   Win32VersionValue;		// 莫须有字段，不被病毒利用的话一般为0 
	DWORD   SizeOfImage;			// 映像装入内存后的总尺寸
	DWORD   SizeOfHeaders;			// 所有头+ 区块表的尺寸大小 
	DWORD   CheckSum;			// 映像的校检和 
	WORD    Subsystem;			// 可执行文件期望的子系统 
	WORD    DllCharacteristics;		// DllMain()函数何时被调用，默认为0 
	DWORD   SizeOfStackReserve;		// 初始化时的栈大小 
	DWORD   SizeOfStackCommit;		// 初始化时实际提交的栈大小 
	DWORD   SizeOfHeapReserve;		// 初始化时保留的堆大小 
	DWORD   SizeOfHeapCommit;		// 初始化时实际提交的堆大小 
	DWORD   LoaderFlags;			// 与调试有关，默认为0  
	DWORD   NumberOfRvaAndSizes;		// 下边数据目录的项数，这个字段自Windows NT 发布以来        // 一直是16 
	IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
	// 数据目录表 
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

magic 填充0x010b ,
address of entrypoint 填充0x0000008c
Imagebase 填充0x00500000 
文件对其这里必须是4不知道为啥
major os version 填充0x0004
major system version 填充0x0004
Size of image 填充 大于file大小的数
Subsystem 填充2 或3 
NumberOfRvaAndSizes 填充 2

# 0x03 代码部分
我们利用messagebox进行弹窗，看一下调用参数

``` cpp
int WINAPI MessageBox(
  _In_opt_ HWND    hWnd,
  _In_opt_ LPCTSTR lpText,
  _In_opt_ LPCTSTR lpCaption,
  _In_     UINT    uType
);
```
压参数不用多讲
主要是怎样正常执行程序，这里使用jmp进行跳转。
利用edx = 程序入口地址
将edx+0x15压入栈中

![][3]

# 0x04 导入表
因为程序要调用user32.dll中的函数，所以需要编写导入表，根据上一个PE攻略可以很快速地编写处导入表
 结构如下

``` cpp
// 20 Bytes
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;            // 0 for terminating null import descriptor(用0来终止导入描述符)
        DWORD   OriginalFirstThunk;         // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
      									    // RVA, 指向IMAGE_THUNK_DATA结构数组
    };
    DWORD   TimeDateStamp;                  // 0 if not bound,（时间戳, 0表示未绑定, 1表示绑定）
                                            // -1 if bound, and real date\time stamp
                                            //     in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                                            // O.W. date/time stamp of DLL bound to (Old BIND)
    DWORD   ForwarderChain;                 // -1 if no forwarders（链表的前一个结构）
    DWORD   Name;						    // RVA, 指向链接库名称的指针，名称以'\0'结尾
    DWORD   FirstThunk;     // RVA to IAT (if bound this IAT has actual addresses)   
                            // RVA, 指向IMAGE_THUNK_DATA结构数组
} IMAGE_IMPORT_DESCRIPTOR;
typedef IMAGE_IMPORT_DESCRIPTOR UNALIGNED *PIMAGE_IMPORT_DESCRIPTOR;
```
OriginalFirstThunk INT表  填写 0x000000D8 
Name 动态库名称 填写 0x000000F0
FirstThunk IAT表地址 填写 0x00000100



![][4]
 
# 0x5 全部数据

``` 
4D5A0000504500004C01000000000000000000000000000000000F010B0100000000000000000000000000008C000000000000000000000000005000040000000400000004000000000000000400000000000000000100008C00000000000000020000000000000000000000000000000000000000000000020000000000000000000000B0000000180000006A006A0068E20050006A0083C21552FF2500015000C30000000000000000000000000000D80000000000000000000000F0000000000100000000000000000000000000000000000000000000E000000000000000BE014D657373616765426F78410011117573657233322E646C6C0000000000001111111100000000
```
![][5]

在win7 x86系统下能够运行成功


  [1]: ./images/3.png
  [2]: ./images/4.png
  [3]: ./images/5.png
  [4]: ./images/6.png
  [5]: ./images/7.png



