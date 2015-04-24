# lab4

##EX1 alloc_proc
```
>
实现过程：
        proc->state = PROC_UNINIT;//state
        proc->pid = -1;//pid
        proc->runs = 0;//running times
        proc->kstack = 0;//process kernal stack
        proc->need_resched = 0;//need to be receduled to release cpu
        proc->parent = NULL;//parent proc
        proc->mm = NULL;//
        memset(&(proc->context), 0, sizeof(struct context));//clean context
        proc->tf = NULL;//clear trapframe
        proc->cr3 = boot_cr3;//init cr3
        proc->flags = 0;//flag=0
        memset(proc->name, 0, PROC_NAME_LEN);//name..
   创建一个内核线程，首先分配一个线程块kmalloc(sizeof(struct proc_struct));
   如果成功则继续设置参数。首先设置状态state为proc_uninit，然后设置pid为-1，设置runs为0，kstack为0，need_resched为0，初始parent父线程为NULL，mm设置为0，之后清空上下文context，设置trapframe为NULL，crt为基础boot_cr3，标志flag位0，最后清空name名称位。
   这里统一使用memset清空结构或者字符串，即把所有位全部置为0。

   因为这里是初始化分配，因此所有标志位上或者地址都是表示为无效的0或者NULL。

思考题：
   这里使用的context是保存的线程上下文。在需要切换线程的时候，需要保存线程中基本的寄存器等基本信息，这些内容就是保存在context结构中。在之后用于恢复进程的环境。
   tf也就是trapframe，用于保存中断线程时的需要保存的线程信息。在处理完毕后用于恢复现场。
      
```
###EX2 do_fork
```
>
实现过程：
   复制线程。
   1. 首先申请分配一个新线程，调用alloc_proc()
   2. 设置parent为当前的线程，setup_kstack() copy_mm()，检查初始化的错误。
   3. 调用local_intr_save
   4. 之后调用get_pid()设置好复制线程的pid，hash_proc()，之后把这个新线程加入到线程队列中list_add()，最后全局线程数量nr_process++
   5. 调用local_intr_restore()之后唤醒线程wakeup+proc()，返回值设置为当前复制线程的pid。

思考题：
   是可以达到线程id唯一的。在get_pid中已经存在可以保证所有调用这个函数的地方都可以获得一个唯一的pid，而在fork的时候时候也是调用的这个函数，因此可以保证pid即使是do_fork也是唯一的。
```
##EX3
```
>
proc_run:
   这是运行线程的实现函数。在一开始需要判断当前线程是否可运行。因此如果当前线程不是可以运行的状态，就需要调用线程切换函数。切换有以下步骤：
       1. 确定当前的线程proc
       2. 切换内核堆栈，把栈顶的指针esp0设置为当前需要切换的proc的栈顶
       3. 切换页表，设置页目录的基地址寄存器cr3为需要切换的proc的cr3
       4. 切换上下文context
   以上完成后就实现了线程的切换。

在本实验的执行过程中， 创建且运行了几个内核线程？
   有2个线程。分别是idleproc和initproc。

语句 local_intr_save( intr_flag) ; . . . . local_intr_restore( intr_flag) ; 在这里有何作用?请说明理由	
   作用：开中断、关中断
   理由：因为进程的切换不能再被中断，因此这里需要清除中断标记位，在切换完成之后再恢复。

```
```
>列出你认为本实验中重要的知识点，以及与对应的OS原理中的知识点
   三状态模型
   线程的切换，中断
   线程复制fork的实际实现调用过程
>列出你认为OS原理中重要的知识点，但在实验中没有对应上
   没有更细节的表现出三状态模型的过程
```
