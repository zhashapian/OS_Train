## 练习0：填写已有实验



## 练习1：给未被映射的地址映射上物理页

#### 1.函数实现分析

 三个传入参数分别是：应用程序虚拟存储总管mm、错误码error_code和具体出错的虚拟地址addr。

​ 函数功能是对可能出现页访问错误的情况的处理和报错。

​ 对虚拟地址进行判断，如果虚拟地址的范围超过了限制，或者是虚拟地址无法被查找到，即可以说该地址是不合法的，进行了一次非法访问，那么可以直接报错。

​ 对目标访问页的权限判断，比如对一个只读页进行写操作，或者读了一个不可读的页，那么此时可以直接报错。错误码的低2位分别是：P标志（位0）最低位：表示当前的错误是由于不存在页面（0）引起，还是由于违反访问权限（1）引起。W / R标志（位1）：表示当前错误是由于读操作（0）引起还是还是写操作（1）引起。

​ 如果能够顺利通过上述的合法性判断，那么此次虚拟内存访问就被认为是合法的，此时，页访问异常的原因，是由于该合法虚拟页，没有对应物理页的映射导致，因此下一步要建立起这个映射。通过当前应用程序mm所指向的一级页表，以及虚拟地址，去查询有没有对应的二级页表，get_pte函数会进行搜索，如果没有，那么get_pte函数会新建一个二级页表产生对于，如果创建失败则会返回一个NULL，然后报错。

​ 如果是上述新创建的二级页表，那么*ptep就会是0，代表页表为空，此时调用pgdir_alloc_page，对它进行初始化建立映射关系，如果不为0则表示已有对应的映射，需要进行映射的替换。


#### 2.代码

```
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

```



` if ((ret = swap_in(mm, addr, &page)) != 0) {
                cprintf("swap_in in do_pgfault failed\n");
                goto failed;
            }    
            page_insert(mm->pgdir, page, addr, perm);
        //建立虚拟地址和物理地址之间的对应关系，perm设置物理页权限，为了保证和它对应的虚拟页权限一致
            swap_map_swappable(mm, addr, page, 1);//将此页面设置为可交换的 ,也添加到算法所维护的次序队列
        page->pra_vaddr = addr;        //设置页对应的虚拟地址
        }
        else {
            cprintf("no swap_init_ok but ptep is %x, failed\n",*ptep);
            goto failed;
        }
   }`




#### 3.结果：

![](C:\Users\a\AppData\Roaming\marktext\images\2022-07-29-22-43-05-image.png)

## 练习二：补充完成基于FIFO的页面替换算法（需要编程）

#### 1.完善do_pgfault函数 ​

swap_init_ok是一个标记位，代表交换初始化成功，可以开始替换的过程了。首先声明了一个页，之后将结构mm、虚拟地址和这个空页，调用了swap_in函数。该函数首先为传入的空页page分配初始化，之后获取了mm一级页表对应的二级页表，通过swapfs_read尝试将硬盘中的内容换入到新的page中，最后，建立起该页的虚拟地址和物理地址之间的对应关系，然后设置为可交换，该页的虚拟地址设置为传入的地址。至此，do_pgfault结束，建立起了新的映射关系，下次访问不会有异常。

#### 2.FIFO替换算法：

​ 先进先出(First In First Out, FIFO)页替换算法：该算法总是淘汰在内存中驻留时间最久的页予以淘汰。只需把一个应用程序在执行过程中已调入内存的页按先后次序链接成一个队列，队列头指向内存中驻留时间最久的页，队列尾指向最近被调入内存的页。这样需要淘汰页时，从队列头很容易查找到需要淘汰的页。 

#### 3._fifo_map_swappable函数

```
static int _fifomap_swappable(struct mm_struct *mm, uintptr_t addr, struct Page *page, int swap_in) {        
        list_entry_t *head=(list_entry_t*) mm->sm_priv;   
        list_entry_t *entry=&(page->pra_page_link);          
        assert(entry != NULL && head != NULL);   
        list_add(head, entry); //将最近用到的页面添加到次序队尾  
        return 0;   
    } 
```

#### 4._fifo_swap_out_victim函数

```
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

#### 5.结果

![](C:\Users\a\AppData\Roaming\marktext\images\2022-07-29-22-40-57-image.png)




















