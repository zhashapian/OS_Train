#### lab0

##### 实验目的：

- 了解操作系统开发实验环境
- 熟悉命令行方式的编译、调试工程
- 掌握基于硬件模拟器的调试技术
- 熟悉C语言编程和指针的概念
- 了解X86汇编语言

##### 实验操作：

配置虚拟机环境（避免中文路径）









#### lab1

##### 实验目的：

###### 练习1：

1.操作系统镜像文件 ucore.img 是如何一步一步生成的?(需要比较详细地解释 Makefile 中每一条相关命令和命令参数的含义,以及说明命令导致的结果)

输入make "V="后，得到以下显示

```
 cc kern/init/init.c           //编译init.c
      gcc -c kern/init/init.c -o obj/kern/init/init.o

+ cc kern/libs/readline.c       //编译readline.c
      gcc -c kern/libs/readline.c -o 
      obj/kern/libs/readline.o

+ cc kern/libs/stdio.c          //编译stdlio.c
      gcc -c kern/libs/stdio.c -o obj/kern/libs/stdio.o

+ cc kern/debug/kdebug.c        //编译kdebug.c
      gcc -c kern/debug/kdebug.c -o obj/kern/debug/kdebug.o

+ cc kern/debug/kmonitor.c      //编译komnitor.c
      gcc  -c kern/debug/kmonitor.c -o         
      obj/kern/debug/kmonitor.o

+ cc kern/debug/panic.c         //编译panic.c
      gcc  -c kern/debug/panic.c -o obj/kern/debug/panic.o

+ cc kern/driver/clock.c        //编译clock.c
      gcc  -c kern/driver/clock.c -o obj/kern/driver/clock.o

+ cc kern/driver/console.c      //编译console.c
      gcc -c kern/driver/console.c -o 
      obj/kern/driver/console.o

+ cc kern/driver/intr.c         //编译intr.c
      gcc -c kern/driver/intr.c -o obj/kern/driver/intr.o

+ cc kern/driver/picirq.c       //编译prcirq.c
      gcc -c kern/driver/picirq.c -o 
      obj/kern/driver/picirq.o

+ cc kern/trap/trap.c           //编译trap.c
      gcc -c kern/trap/trap.c -o obj/kern/trap/trap.o

+ cc kern/trap/trapentry.S      //编译trapentry.S
      gcc -c kern/trap/trapentry.S -o 
      obj/kern/trap/trapentry.o

+ cc kern/trap/vectors.S        //编译vectors.S
      gcc -c kern/trap/vectors.S -o obj/kern/trap/vectors.o

+ cc kern/mm/pmm.c              //编译pmm.c
      gcc -c kern/mm/pmm.c -o obj/kern/mm/pmm.o

+ cc libs/printfmt.c            //编译printfmt.c
      gcc -c libs/printfmt.c -o obj/libs/printfmt.o

+ cc libs/string.c              //编译string.c
      gcc -c libs/string.c -o obj/libs/string.o

+ ld bin/kernel                 //链接成kernel
      ld -o bin/kernel  
      obj/kern/init/init.o      obj/kern/libs/readline.o 
      obj/kern/libs/stdio.o     obj/kern/debug/kdebug.o 
      obj/kern/debug/kmonitor.o obj/kern/debug/panic.o 
      obj/kern/driver/clock.o   obj/kern/driver/console.o 
      obj/kern/driver/intr.o    obj/kern/driver/picirq.o
      obj/kern/trap/trap.o      obj/kern/trap/trapentry.o 
      obj/kern/trap/vectors.o   obj/kern/mm/pmm.o  
      obj/libs/printfmt.o       obj/libs/string.o

+ cc boot/bootasm.S             //编译bootasm.c
     gcc  -c boot/bootasm.S -o obj/boot/bootasm.o

+ cc boot/bootmain.c            //编译bootmain.c
     gcc -c boot/bootmain.c -o obj/boot/bootmain.o

+ cc tools/sign.c               //编译sign.c
    gcc -c tools/sign.c -o obj/sign/tools/sign.o
    gcc -O2 obj/sign/tools/sign.o -o bin/sign

+ ld bin/bootblock              //根据sign规范生成bootblock
    ld -m  elf_i386 -nostdlib -N -e start -Ttext 0x7C00 
    obj/boot/bootasm.o  obj/boot/bootmain.o
    -o obj/bootblock.o

     //创建大小为10000个块的ucore.img，初始化为0，每个块为512字节
dd if=/dev/zero of=bin/ucore.img count=10000
    //把bootblock中的内容写到第一个块
dd if=bin/bootblock of=bin/ucore.img conv=notrunc
    //从第二个块开始写kernel中的内容
dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
```

由上分析可知，ucore.img的生成过程大致按以下步骤执行

1. 编译所有生成bin/kernel所需的文件

2. 链接生成bin/kernel 

3. 编译bootasm.S ;bootmain.c; sign.c 

4. 根据sign规范生成obj/bootblock.o 

5.  生成ucore.img





2.一个被系统认为是符合规范的硬盘主引导扇区的特征是什么?

(截取sign.c中的代码说明)

```
buf[510]=0x55; //定义buff数组
buf[511]=0XAA; // 把buf数组的最后两位置为 0x55, 0xAA
FILE * ofp=fopen(argv[2],"wb+");
 size=fwrite(buf,1,512, ofp);
  if(size!=512){   //大小为512字节
fprintf(stderr,"write '%s' error, size is %d.\n", argv[2], size);
 return-1;
}


```

硬盘主引导扇区的特征是:

1. 大小为512字节 

2. 多余的空间填0 

   3 .第510个（倒数第二个）字节是0x55

   4 .第511个（倒数第一个）字节是0xAA



###### 练习2：使用qemu执行并调试lab1中的文件

修改lab1/tools/gdbinit ,内容为：

```
set architecture i8086
target remote :1234
```

然后在 lab1执行：

```
si
```

来单步跟踪

在gdb的调试界面，执行如下命令，来查看BIOS代码：

```
 x /2i $pc  //显示当前eip处的汇编指令
```

###### 练习2.2： 在初始化位置0x7c00 设置实地址断点,测试断点正常

修改 gdbinit文件：

```
set architecture i8086
target remote :1234
b *0x7c00
c
x/2i $pc
```

###### 练习2.3：从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。

```
debug: $(UCOREIMG)
        $(V)$(TERMINAL) -e "$(QEMU) -S -s -d in_asm -D  $(BINDIR)/q.log -parallel stdio -hda $< -serial null"
        $(V)sleep 2
        $(V)$(TERMINAL) -e "gdb -q -tui -x tools/gdbinit
```

###### 



###### 练习2.4：自己找一个bootloader或内核中的代码位置，设置断点并进行测试。

修改gdbinit文件，在0x7c4a处设置断点 (调用bootmain函数处)

```
set architecture i8086
target remote :1234
break *0x7c4a
```

输入make debug,得到以下图示，表明断点设置正常。

```
Breakpoint 1， 0x00007c4a in ?? ()
(gdb)  x/2i $pc
=> 0x7c4a:    call    0x7ccf
   0x7c4b:    call    %al,(%bx,%si)
```



###### 练习3：

1.关中断和清除数据段寄存器

```
.globl start
start:
.code16                                             
    cli              //关中断                          
    cld              //清除方向标志                           
    xorw %ax, %ax    //ax清0                           
    movw %ax, %ds    //ds清0                                
    movw %ax, %es    //es清0                               
    movw %ax, %ss    //ss清0   

```

###### 练习3.1： 为何开启A20，以及如何开启A20？

初始时A20为0，访问超过1MB的地址时，就会从0循环计数，将A20地址线置为1之后，才可以访问4G内存。A20地址位由8042控制，8042有2个有两个I/O端口：0x60和0x64；

打开流程：

1.等待8042 Input buffer为空；
2.发送Write 8042 Output Port （P2）命令到8042 Input buffer；
3.等待8042 Input buffer为空；
4.将8042 Output Port（P2）得到字节的第2位置1，然后写入8042 Input buffer；


###### 练习3.2：初始化gdt表

1.载入GDT表

```
lgdt gdtdesc       //载入GDT表
```

2.进入保护模式

cro的第0位为1表示处于保护模式

```
movl %cr0, %eax       //加载cro到eax
orl $CR0_PE_ON, %eax  //将eax的第0位置为1
movl %eax, %cr0       //将cr0的第0位置为1
```

3.通过长跳转更新cs的基地址:

```
ljmp $PROT_MODE_CSEG, $protcseg
.code32                          
protcseg:
```

4.置段寄存器，并建立堆栈

```
 movw $PROT_MODE_DSEG, %ax //                      
 movw %ax, %ds                                  
 movw %ax, %es                                   
 movw %ax, %fs                                   
 movw %ax, %gs                                   
 movw %ax, %ss                                   
 movl $0x0, %ebp  //设置帧指针
 movl $start, %esp  //设置栈指针

```



5.转到保护模式完成，进入boot主方法

```
call bootmain //调用bootmain函数
```

###### 练习4： 分析bootloader加载ELF格式的OS的过程。

###### 练习4.1：bootloader如何读取硬盘扇区的？

bootloader让CPU进入保护模式后，下一步的工作就是从硬盘上加载并运行OS。考虑到实现的简单性，bootloader的访问硬盘都是LBA模式的PIO（Program IO）方式，即所有的IO操作是通过CPU访问硬盘的IO地址寄存器完成。

每个扇区的大小为512字节

扇区读取流程：

1.等待磁盘准备好；  
2.发出读取扇区的命令；  
3.等待磁盘准备好；  
4.把磁盘扇区数据读到指定内存。



###### 练习5：实现函数调用堆栈跟踪函数



print_stackframe函数实现：

```
void print_stackframe(void) {
    uint32_t ebp = read_ebp(), eip = read_eip();  //获取ebp和eip的值
    int i, j;
    //#define STACKFRAME_DEPTH 20
    for (i = 0; ebp != 0 && i < STACKFRAME_DEPTH; i ++) {
        cprintf("ebp:0x%08x eip:0x%08x args:", ebp, eip); 
        uint32_t *args = (uint32_t *)ebp + 2; //参数的首地址
        for (j = 0; j < 4; j ++) {
            cprintf("0x%08x ", args[j]); //打印4个参数
        }
        cprintf("\n");
        print_debuginfo(eip - 1);  //打印函数信息
        eip = ((uint32_t *)ebp)[1]; //更新eip
        ebp = ((uint32_t *)ebp)[0]; //更新ebp
    }
}

```

###### 练习6：完善中断初始化和处理

完善idt_init函数：

```
void idt_init(void) {
    extern uintptr_t __vectors[];  //保存在vectors.S中的256个中断处理例程的入口地址数组
    int i;
   //使用SETGATE宏，对中断描述符表中的每一个表项进行设置
    for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i ++) { //IDT表项的个数
    //在中断门描述符表中通过建立中断门描述符，其中存储了中断处理例程的代码段GD_KTEXT和偏移量__vectors[i]，特权级为DPL_KERNEL。这样通过查询idt[i]就可定位到中断服务例程的起始地址。
     SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
    }
    SETGATE(idt[T_SWITCH_TOK], 0, GD_KTEXT,     
    __vectors[T_SWITCH_TOK], DPL_USER);
     //建立好中断门描述符表后，通过指令lidt把中断门描述符表的起始地址装入IDTR寄存器中，从而完成中段描述符表的初始化工作。
    lidt(&idt_pd);
}

```

完善rap_dispatch函数：

```
static void
trap_dispatch(struct trapframe *tf) {
    char c;
    switch (tf->tf_trapno) {
    case IRQ_OFFSET + IRQ_TIMER:
        ticks ++;
        if (ticks % TICK_NUM == 0) {
            print_ticks();
        }
        break;
    
    ......
    }
......
}


```





#### lab2：物理内存管理

##### 实验目的：

理解基于段页式内存地址的转换机制

理解页表的建立和使用方法

理解物理内存的管理方法





###### 操作过程：

###### 练习0：

需要更改的文件为kdebug.c和trap.c，具体更改的代码如下：

kdebug.c：

```kdebug.c：
uint32_tt_ebp=read_ebp();
uint32_tt_eip=read_eip();
inti,j;
for(i=0;i<STACKFRAME_DEPTH&&t_ebp!=0;i++){
cprintf("ebp=%08x,eip=%08x,args:",t_ebp,t_eip);
uint32_t*args=(uint32_t*)t_ebp+2;
for(j=0;j<4;j++){
cprintf("0x%08x",args[j]);
}
cprintf("\n");
print_debuginfo(t_eip-1);
t_eip=((uint32_t*)t_ebp)[1];
t_ebp=((uint32_t*)t_ebp)[0];
}
```





trap.c

```trap.c：
void trap(structtrapframe*tf){
ticks++;//每一次时钟信号会使变量ticks加1
if(ticks==TICK_NUM){
//TICK_NUM已经被预定义成了100，每到100便调用print_ticks()函数打印
ticks=0;print_ticks();
}ticks++;//每一次时钟信号会使变量ticks加1
if(ticks==TICK_NUM){//TICK_NUM已经被预定义成了100，每到100便调用print_ticks()函数打印
ticks=0;
print_ticks();
}}
void idt_init(void){extern uintptr_t__vectors[]; 
inti;//初始化idt 
for(i=0;i<256;i++){
SETGATE(idt[i],0,GD_KTEXT,__vectors[i],DPL_KERNEL);
}
SETGATE(idt[T_SWITCH_TOK],0,GD_KTEXT,__vectors[T_SWITCH_TOK],DPL_USER);
SETGATE(idt[T_SWITCH_TOU],0,GD_KTEXT,__vectors[T_SWITCH_TOU],DPL_KERNEL);
lidt(&idt_pd);
}
```



##### 练习1：实现first-fit连续物理内存分配算法（需要编程）

###### 1.first-fit算法：

只要寻找到符合条件的块，就会将它调用。

具体思路：

物理内存页管理器顺着双向链表进行搜索空闲内存区域，直到找到一个足够大的空闲区域，因为它尽可能少地搜索链表。如果空闲区域的大小和申请分配的大小正好一样，则把这个空闲区域分配出去，成功返回；否则将该空闲区分为两部分，一部分区域与申请分配的大小相等，把它分配出去，剩下的一部分区域形成新的空闲区。其释放内存的设计思路很简单，只需把这块区域重新放回双向链表中即可。


###### 2.函数理解与实现：

```
struct Page {
    int ref;                        // 页表计数器
    uint32_t flags;                 // 页表状态
    unsigned int property;          // 连续空闲页的数量
    list_entry_t page_link;         
/*便于把多个连续内存空闲块链接在一起的双向链表指针，
连续内存空闲块利用这个页的成员变量page_link来链接比它地址小和大的其他连续内存空闲块，
释放的时候只要将这个空间通过指针放回到双向链表中。*/
};

```



```
typedef struct {
    list_entry_t free_list;         
// list_entry结构的双向链表指针（指向首地址）
    unsigned int nr_free;          
 // 空闲链表数目
} free_area_t;

```





###### default——init：

这部分是一个初始化的部分，就是将双向链表初始化，同时将空闲页总数nr_free初始化为0。

```
free_area_t free_area;

#define free_list (free_area.free_list)
#define nr_free (free_area.nr_free)

static void
default_init(void) {
    list_init(&free_list);
    nr_free = 0;
}

```

###### default_init_memmap：

```
static void
default_init_memmap(struct Page *base, size_t n) {
    assert(n > 0);   //判断n是否大于0
    struct Page *p = base;
    for (; p != base + n; p ++) { //初始化n块物理页
        assert(PageReserved(p)); //检查此页是否为保留页
        p->flags = p->property= 0; //标志位清0
        SetPageProperty(p);       //设置标志位为1
        set_page_ref(p, 0);  //清除引用此页的虚拟页的个数
       //加入空闲链表
        list_add_before(&free_list, &(p->page_link)); 
    }
    nr_free += n;  //计算空闲页总数
    base->property = n; //修改base的连续空页值为n
}

```

###### default_alloc_pages：

```
static void
default_free_pages(struct Page *base, size_t n) {
    assert(n > 0);  
    assert(PageReserved(base));    //检查需要释放的页块是否已经被分配
    list_entry_t *le = &free_list; 
    struct Page * p;
    while((le=list_next(le)) != &free_list) {    //寻找合适的位置
      p = le2page(le, page_link); //获取链表对应的Page
      if(p>base){    
        break;
      }
    }
    for(p=base;p<base+n;p++){              
      list_add_before(le, &(p->page_link)); //将每一空闲块对应的链表插入空闲链表中
    }
    base->flags = 0;         //修改标志位
    set_page_ref(base, 0);    
    ClearPageProperty(base);
    SetPageProperty(base);
    base->property = n;      //设置连续大小为n
    //如果是高位，则向高地址合并
    p = le2page(le,page_link) ;
    if( base+n == p ){
      base->property += p->property;
      p->property = 0;
    }
     //如果是低位且在范围内，则向低地址合并
    le = list_prev(&(base->page_link));
    p = le2page(le, page_link);
    if(le!=&free_list && p==base-1){ //满足条件，未分配则合并
      while(le!=&free_list){
        if(p->property){ //连续
          p->property += base->property;
          base->property = 0;
          break;
        }
        le = list_prev(le);
        p = le2page(le,page_link);
      }
    }

    nr_free += n;
    return ;
}

```

###### 该first-fit算法是否有改进空间：

defalut_pmm.c中包含两个check函数，用来检测是否正确实现了first fit算法。而defalut_pmm.c中的算法要改进的地方在于：first-fit算法要求将空闲内存块按照地址从小到大的方式链接起来，而defalut_pmm.c中的算法没有实现这一点。

##### **练习2：实现寻找虚拟地址对应的页表项（需要编程）**

一些常量分析：

```
PDX(la)： 返回虚拟地址la的页目录索引

KADDR(pa): 返回物理地址pa相关的内核虚拟地址

set_page_ref(page,1): 设置此页被引用一次

page2pa(page): 得到page管理的那一页的物理地址

struct Page * alloc_page() : 分配一页出来

memset(void * s, char c, size_t n) : 设置s指向地址的前面n个字节为字节‘c’

PTE_P 0x001 表示物理内存页存在

PTE_W 0x002 表示物理内存页内容可写

PTE_U 0x004 表示可以读取对应地址的物理内存页内容

```





###### get_pte函数：

```
pte_t *
get_pte(pde_t *pgdir, uintptr_t la, bool create) {
        pde_t *pdep = &pgdir[PDX(la)];  //尝试获得页表
        if (!(*pdep & PTE_P)) { //如果获取不成功
            struct Page *page;
            //假如不需要分配或是分配失败
            if (!create || (page = alloc_page()) == NULL) { 
                return NULL;
        }
        set_page_ref(page, 1); //引用次数加一
        uintptr_t pa = page2pa(page);  //得到该页物理地址
        memset(KADDR(pa), 0, PGSIZE); //物理地址转虚拟地址，并初始化
        *pdep = pa | PTE_U | PTE_W | PTE_P; //设置控制位
    }
    return &((pte_t *)KADDR(PDE_ADDR(*pdep)))[PTX(la)]; 
    //KADDR(PDE_ADDR(*pdep)):这部分是由页目录项地址得到关联的页表物理地址， 再转成虚拟地址
    //PTX(la)：返回虚拟地址la的页表项索引
    //最后返回的是虚拟地址la对应的页表项入口地址
}

```



###### 

###### 请描述页目录项（Page Directory Entry）和页表项（Page Table Entry）中每个组成部分的含义以及对ucore而言的潜在用处。



###### 如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

当启动分页机制以后，如果一条指令或数据的虚拟地址所对应的物理页不在内存中或者访问的类型有误（比如写一个只读页或用户态程序访问内核态的数据等），就会发生页错误异常。
而产生页面异常的原因主要有:
1.目标页面不存在（页表项全为0，即该线性地址与物理地址尚未建立映射或者已经撤销）；
2.相应的物理页面不在内存中（页表项非空，但Present标志位=0，比如将页表交换到磁盘）；
3.访问权限不符合（比如企图写只读页面）


当出现上面情况之一,那么就会产生页面page fault(#PF)异常。产生异常的虚拟地址存储在CR2中，并且将是page fault的错误类型保存在error code中。引发异常后将外存的数据换到内存中，进行上下文切换，退出中断，返回到中断前的状态。


#### 练习3：**释放某虚地址所在的页并取消对应二级页表项的映射（需要编程）**

当释放一个包含某虚地址的物理内存页时，需要让对应此物理内存页的管理数据结构Page做相关的清除处理，使得此物理内存页成为空闲；另外还需把表示虚地址与物理地址对应关系的二级页表项清除。请仔细查看和理解page_remove_pte函数中的注释。为此，需要补全在 kern/mm/pmm.c中的page_remove_pte函数。







###### 1.数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？

Page的每一项记录一个物理页的信息，而每个页目录项记录一个页表的信息，每个页表项则记录一个物理页的信息。假设系统中共有N个物理页，那么Page共有N项，第i项对应第i个物理页的信息。而页目录项和页表项的第31~12位构成的20位数分别对应一个物理页编号，因此也就和Page的对应元素一一对应。页目录项和页表项的前20位就可以表明它是哪个Page。
(1)将虚拟地址向下对齐到页大小，换算成物理地址(-KERNBASE), 再将其右移GSHIFT(12)位获得在pages数组中的索引PPN，&pages[PPN]就是所求的Page结构地址。
(2)PTE按位与0xFFF获得其指向页的物理地址，再右移PGSHIFT(12)位获得在pages数组中的索引PPN，&pages[PPN]就PTE指向的地址对应的Page结构。


###### 2.如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？

1.由背景知识知，gcc编译出的虚拟起始地址从0xC0100000开始，ucore被bootloader放置在从物理地址0x100000处开始的物理内存中，首先得将虚拟起始地址设置为0x100000。

2.由背景知识得，最后虚拟地址和物理地址的映射关系满足：  
physical address + KERNBASE = virtual address  可尝试将KERNBASE改为0





###### page_remove_pte的补全

```
static inline void
page_remove_pte(pde_t *pgdir, uintptr_t la, pte_t *ptep) {
if (*ptep & PTE_P) {  //页表项存在
        struct Page *page = pte2page(*ptep); //找到页表项
        if (page_ref_dec(page) == 0) {  //只被当前进程引用
            free_page(page); //释放页
        }
        *ptep = 0; //该页目录项清零
        tlb_invalidate(pgdir, la); 
        //修改的页表是进程正在使用的那些页表，使之无效
    }
}

```

###### 

###### 数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？

所有的物理页都有一个描述它的Page结构，所有的页表都是通过alloc_page()分配的，每个页表项都存放在一个Page结构描述的物理页中；如果 PTE 指向某物理页，同时也有一个Page结构描述这个物理页。所以有两种对应关系：



1.可以通过 PTE 的地址计算其所在的页表的Page结构：

将虚拟地址向下对齐到页大小，换算成物理地址(减 KERNBASE), 再将其右移固定位获得在pages数组中的索引就是所求的Page结构地址。



2.可以通过 PTE 指向的物理地址计算出该物理页对应的Page结构：

PTE 按位与 0xFFF获得其指向页的物理地址，再右移固定位获得在pages数组中的索引，得到 PTE 指向的地址对应的Page结构。



#### lab3

##### 练习0：将之前的代码复制至lab3相应文件中

##### 练习1： 给未被映射的地址映射上物理页

###### 1.VMA分析：

```
    struct vma_struct {  
        // the set of vma using the same PDT  
        struct mm_struct *vm_mm;  
        uintptr_t vm_start;      // start addr of vma  
        uintptr_t vm_end;      // end addr of vma  
        uint32_t vm_flags;     // flags of vma  
        //linear list link which sorted by start addr of vma  
        list_entry_t list_link;  
    }; 
/*
vm_start和vm_end描述的是一
    个合理的地址空间范围（即严格确保 vm_start < vm_end的关系）；
list_link是一个双向链表，按照从
    小到大的顺序把一系列用vma_struct表示的
    虚拟内存空间链接起来，并且还要求这些链起来的vma_str
    uct应该是不相交的，即vma之间的地址空间无交集；
vm_flags表示了这个虚拟内存空间的属性，目前的属性包括
*/

```



```
/*vm_mm是一个指针，如果说vma描述和管理的是一系列虚拟内存块结构，
那么mm则用来描述，究竟是哪个应用程序使用了这些vma，它们是通过vma中
的成员mm联系在一起的。mm也有五个成员：*/
struct mm_struct {  
        // linear list link which sorted by start addr of vma  
        list_entry_t mmap_list;  
        // current accessed vma, used for speed purpose  
        struct vma_struct *mmap_cache;  
        pde_t *pgdir; // the PDT of these vma  
        int map_count; // the count of these vma  
        void *sm_priv; // the private data for swap manager  
    };  
/* mmap_list是双向链表头，链接了所有属于同一页目录表的虚拟内存空间
mmap_cache是指向当前正在使用的虚拟内存空间
pgdir所指向的就是 mm_struct数据结构所维护的页表
map_count记录mmap_list里面链接的vma_struct的个数
sm_priv指向用来链接记录页访问情况的链表头*/

```



###### 2.函数实现：

三个传入参数分别是：应用程序虚拟存储总管mm、错误码error_code和具体出错的虚拟地址addr。

​ 函数功能是对可能出现页访问错误的情况的处理和报错。

​ 对虚拟地址进行判断，如果虚拟地址的范围超过了限制，或者是虚拟地址无法被查找到，即可以说该地址是不合法的，进行了一次非法访问，那么可以直接报错。

​ 对目标访问页的权限判断，比如对一个只读页进行写操作，或者读了一个不可读的页，那么此时可以直接报错。错误码的低2位分别是：P标志（位0）最低位：表示当前的错误是由于不存在页面（0）引起，还是由于违反访问权限（1）引起。W / R标志（位1）：表示当前错误是由于读操作（0）引起还是还是写操作（1）引起。

​ 如果能够顺利通过上述的合法性判断，那么此次虚拟内存访问就被认为是合法的，此时，页访问异常的原因，是由于该合法虚拟页，没有对应物理页的映射导致，因此下一步要建立起这个映射。通过当前应用程序mm所指向的一级页表，以及虚拟地址，去查询有没有对应的二级页表，get_pte函数会进行搜索，如果没有，那么get_pte函数会新建一个二级页表产生对于，如果创建失败则会返回一个NULL，然后报错。

​ 如果是上述新创建的二级页表，那么*ptep就会是0，代表页表为空，此时调用pgdir_alloc_page，对它进行初始化建立映射关系，如果不为0则表示已有对应的映射，需要进行映射的替换。



###### do_pgfault函数：

```
int do_pgfault(struct mm_struct *mm, uint32_t error_code, uintptr_t addr) {
    int ret = -E_INVAL; 
 //#############对虚拟地址进行判断##################
    //尝试寻找包括addr的vma
    struct vma_struct *vma = find_vma(mm, addr);
    pgfault_num++;
    //判断addr是否在vma的范围中
    if (vma == NULL || vma->vm_start > addr) {
        cprintf("not valid addr %x, and  can not find it in vma\n", addr);
        goto failed;
    }
  //############对目标访问页的权限判断###############
    switch (error_code & 3) {
    default:
            /* error code flag : default is 3 ( W/R=1, P=1): write, 
present */
    case 2: /* error code flag : (W/R=1, P=0): write, not present */
        if (!(vma->vm_flags & VM_WRITE)) {
            cprintf("do_pgfault failed: error code flag = write AND 
not present, but the addr's vma cannot write\n");
            goto failed;
        }
        break;
    case 1: /* error code flag : (W/R=0, P=1): read, present */
        cprintf("do_pgfault failed: error code flag = 
read AND present\n");
        goto failed;
	//如果无法读取直接报错
    case 0: /* error code flag : (W/R=0,read, not present */
 
        if (!(vma->vm_flags & (VM_READ | VM_EXEC))) {
            cprintf("do_pgfault failed: error code flag = read AND not present, but the addr's vma cannot read or exec\n");
            goto failed;
        }
    }
    uint32_t perm = PTE_U;
                //prem:给物理页赋予权限的中间变量
    if (vma->vm_flags & VM_WRITE) {
        perm |= PTE_W;
    }
    addr = ROUNDDOWN(addr, PGSIZE);
    ret = -E_NO_MEM;
    pte_t *ptep=NULL;
/*##################修改部分#####################*/
    //查找页目录，如果不存在则失败
    if ((ptep = get_pte(mm->pgdir, addr, 1)) == NULL) {
        cprintf("get_pte in do_pgfault failed\n");
        goto failed;
    }
    if (*ptep == 0) { // 如果是新创建的二级页表。
        //初始化建立虚拟地址与物理页之间的映射关系
        if (pgdir_alloc_page(mm->pgdir, addr, perm) == NULL) {
            //初始化失败报错并退出。
            cprintf("pgdir_alloc_page in do_pgfault failed\n");
            goto failed;
        }
    }
/*#################练习二实现##################*/    
   //页表项非空，尝试换入页面
    else { 
   
        if(swap_init_ok) {
            struct Page *page=NULL;
	    //根据mm结构和addr地址，尝试将硬盘中的内容换入至page中
            if ((ret = swap_in(mm, addr, &page)) != 0) {
                cprintf("swap_in in do_pgfault failed\n");
                goto failed;
            }    
            page_insert(mm->pgdir, page, addr, perm);
	    //建立虚拟地址和物理地址之间的对应关系，perm设置物理页权限，
         //   为了保证和它对应的虚拟页权限一致
            swap_map_swappable(mm, addr, page, 1);
        //将此页面设置为可交换的 ,也添加到算法所维护的次序队列
	    page->pra_vaddr = addr;		//设置页对应的虚拟地址
        }
        else {
            cprintf("no swap_init_ok but ptep is %x, failed\n",*ptep);
            goto failed;
        }
   }
/*#################修改部分#######################*/
   ret = 0;
failed:
    return ret;
}

```

###### 3.请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处

     分页机制的实现确保了虚拟地址和物理地址之间的对应关系。

一方面，通过查找虚拟地址是否存在于一二级页表中可知发现该地址是否是合法的；同时可以通过修改映射关系实现页替换操作。

另一方面，在实现页替换时涉及到换入换出：换入时需要将某个虚拟地址对应的磁盘的一页内容读入到内存中，换出时需要将某个虚拟页的内容写到磁盘中的某个位置。而页表项可以记录该虚拟页在磁盘中的位置，为换入换出提供磁盘位置信息，页目录项则是用来索引对应的页表。

同时，我们可知PDE和PTE均保留了一些位给操作系统使用，具体可以应用在页替换算法时,present位为0时CPU不使用PTE上内容，这时候这些位便会闲置，可以将闲置位用于保存别的信息，例如页替换算法被换出的物理页在交换分区的位置等，并且需要注意到dirty位，操作系统根据脏位可以判断是否对页数据进行write through。


###### 4.如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

首先调用中断机制，引起中断  
保存现场：将发生错误的线性地址保存在寄存器中;

并且把访问异常码error code保存到中断栈；  
根据中断描述符表查询到对应页访问异常的ISR，跳转到对应的ISR处执行，实 现缺页服务例程



##### 练习2：补充完成基于FIFO的页面替换算法（需要编程）

###### 1.继续完善do_pgfault函数（代码内容见上文）

swap_init_ok是一个标记位，代表交换初始化成功，可以开始替换的过程了。首先声明了一个页，之后将结构mm、虚拟地址和这个空页，调用了swap_in函数。该函数首先为传入的空页page分配初始化，之后获取了mm一级页表对应的二级页表，通过swapfs_read尝试将硬盘中的内容换入到新的page中，最后，建立起该页的虚拟地址和物理地址之间的对应关系，然后设置为可交换，该页的虚拟地址设置为传入的地址。至此，do_pgfault结束，建立起了新的映射关系，下次访问不会有异常。


###### 2.fifo算法

该算法总是淘汰在内存中驻留时间最久的页予以淘汰。只需把一个应用程序在执行过程中已调入内存的页按先后次序链接成一个队列，队列头指向内存中驻留时间最久的页，队列尾指向最近被调入内存的页。这样需要淘汰页时，从队列头很容易查找到需要淘汰的页。





###### 3.函数实现

###### _fifo_map_swappable：

```
static int _fifo_map_swappable(struct mm_struct *mm, 
uintptr_t addr, struct Page *page, int swap_in) {        
        list_entry_t *head=(list_entry_t*) mm->sm_priv;   
        list_entry_t *entry=&(page->pra_page_link);          
        assert(entry != NULL && head != NULL);   
        list_add(head, entry); //将最近用到的页面添加到次序队尾  
        return 0;   
    } 

```

###### _fifo_swap_out_victim函数:

```使用链表操作，删除掉最早进入的那个页，并按照注释将这个页面给到传入参数ptr_page。
static int  _fifo_swap_out_victim(struct mm_struct *mm, struct Page ** ptr_page, int in_tick) {       
    list_entry_t *head=(list_entry_t*) mm->sm_priv;          
    assert(head != NULL);      
    assert(in_tick==0);
    list_entry_t *le = head->prev;   // 取出链表头，即最早进入的物理页面
    assert(head!=le);  // 确保链表非空
    struct Page *p = le2page(le, pra_page_link);
    //找到对应的物理页面的Page结构
    list_del(le);      //将进来最早的页面从队列中删除      
    assert(p !=NULL);       
    *ptr_page = p; //将这一页的地址存储在ptr_page中
    return 0; 
}

```

###### 4.需要被换出的页的特征是什么？

最早被换入，且最近没有再被访问的页

###### 5.在ucore中如何判断具有这样特征的页？

通过判断是否访问过，对未访问过的物理页FIFO即可

###### 6.何时进行换入和换出操作？

当需要调用的页不在页表中时，并且在页表已满的情况下，会引发PageFault，此时需要进行换入和换出操作。具体是当保存在磁盘中的内存需要被访问时，需要进行换入操作；当位于物理页框中的内存被页面替换算法选择时，需要进行换出操作。








