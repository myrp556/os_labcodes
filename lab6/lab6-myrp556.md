#LAB5 REPORT
学堂账号：myrp556
##[练习0]
```
添加在proc_init中，对lab6需要的内容的初始化：
	初始proc->rq为NULL，表示运行队列为空！
	设置运行队列链表proc->run_link.prev=proc->run_link.next=NULL为空。
	设置时间片proc->time_slice=0，表示初始化为时间片为0	额外添加proc->lab6_run_pool.left=proc->lab6_run_pool.right=proc->lab6_run_pool.parent=NULL为空。
	以及添加proc->lab_stride=lab6_priority=0的初始化内容
添加函数lab6_set_priority()，为lab6准备修改进程priority的函数，并且默认修改为1。
```
##[练习1]
```

lab6 RR算法执行过程
	1. RR_init()：初始化run_queue
    	2. RR_enqueue()：把proc加入到进程队列
    	3. RR_dequeue()：把proce移出进程队列
    	4. RR_pick_next()：从进程队列中找到下一个可以调度的进程，RR算法中就是找到第一个进程
    	5. RR_proc_tick()： 实现触发器功能。每次函数把time_slice-1，如果值为0表示时间片用完需要被调度。

问答题：
1.请理解并分析sched_calss中各个函数指针的用法，并结合Round Robin调度算法描ucore的调度执行过程
        执行过程：
        首先执行关中断，然后判断如果当前进程已经就绪，则插入到就绪进程队列汇总。之后调用函数找到一个可以调度的进程，
        判断如果这个进程是空，不存在，那么就把它设置为idleproc。之后判断如果这个选择的进程不是当前进程，就执行这个进程。
        最后开中断。
        
2.请在实验报告中简要说明如何设计实现”多级反馈队列调度算法“，给出概要设计，鼓励给出详细设计
        由于是多级反馈队列，需要设计根据优先级的多个队列。设计中优先级越低表示应该被多调用，因此设置它的时间片较长一些。
        当进程使用完时间片但是还没有结束进程的时候，把它插入到优先级较低的队列中。
        进程完成创建的时候直接把它加入到最高优先级的队列中。
```

##[练习2]
```
实验过程：
stride_init:
	根据注释内容：
		run_list使用list_init(&(rq->run_list))初始化为一个空List
		lab6_run_pool应该设置为NULL
		proc_num设置为0.
	以上是根据stride初始化的需要，因此与标准并无差别。
stride_enqueue:
	1.根据stride注释描述可以使用skew_heap_insert，因此函数分为两种情况，以USE_SKEW_HEAP作为区分标准。一种直接使用skew_heap_inster函数，另外一种则使用list_add_before插入到队列的最后。
	2.之后需要修改时间片time_slice，这里设置为rq->max_time_slice
	3.设置proc->rq为rq
	4.rq->proc_num++
	
	与标准答案的区别：
	这里个人的思考是，对于插入的新的时间片time_slice的设置，开始自己只考虑到设置所有的time_slice都为max_time_slice，问题可能存在有已经设置好time_slice的情况，并非所有的都是一定未初始化的time_slice==0.另外也许并无太大意义还是修改判断条件为<=0修改为max_time_slice。

stride_dequeue:
	1.与前面一个函数相似的处理，分为skew_heap和list，分别使用skew_heap_remove函数和list_del_init函数
	2.rq->proc_num--
	这里根据注释的内容说明，与标准答案并无差别。

stride_pick_next:
	1.同样的处理，对于skew_heap调用函数le2ptoc，如果之前判断发现rq->lab6_run_pool==NULL则直接返回NULL。如果是list则根据stride依次访问每一个元素，找到当前最小的lab6_stride的元素作为当前需要调度的进程.
	2.之后再把这个进程的stride值加上相应是数量。
	3.最后返回找到的这个需要调度的proc。
	
	与标准答案的区别：
		这里根据我的想法，认为for循环比标准答案的while循环更简洁效率更高，其次判断语句的时候也只需要判断大小与的关系并不需要像标准答案一样运算一次减法，这样运算量也少一点。
	
stride_proc_tick:
	1.判断如果当前的时间片proc->time_slice仍然>0则把它-1，表示时间片减少一个计数。
	2.判断如果当前的时间片proc->time_slice==0，也就是时间片已经用完了，设置proc->need_resched为1，表示需要调度proc。
	
	与标准答案的区别：
		开始进入函数我直接使用把proc->time_slice-1，后来发现这样并不稳妥，因此也像答案一样添加一个判断条件也就是proc->time_slive>0.
```

#其他
```
1. 列出你认为本实验中重要的知识点，以及与对应的OS原理中的知识点
  进程之间的调度规则，ucore调度进程的过程
  调度过程的实现，各个函数的意义
  
2. 列出你认为OS原理中重要的知识点，但在实验中没有对应上
  进程调度仅仅使用了stride，还没有更多其他的介绍
```
