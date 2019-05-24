# 操作系统学习

[TOC]



### 进程管理

##### Linux进程管理

###### 	进程描述符 task_struct

>  	1. 进程状态
>       	1. state
>           	2. exit_state
>  	2. 相关标识符信息
>           	1. pid_t pid //进程标识符 PID
>               	2. pid_t tgid //线程组零头线程的PID&线程组所属进程的PID
>                   	3. pid_t pgrp //进程所属进程组的领头进程的PID
>                       	4. session //进程所属会话的领头进程的PID
>                           	5. uid //进程的用户ID
>                               	6. gid //进程属于哪个用户组
>                                   	7. euid&egid //有效的uid&gid 安全权限
>                                       	8. suid&sgid //备份的uid&gid
>                                           	9. fsuid&fsgid //文件系统的uid&gid，用于对文件系统操作时的合法检查。（NFS
> 	3. 家族关系
>     	1. struct task_struct *group_leader //指向所属进程组的领头进程的指针 有自己的终端tty，当前进程就是从这个终端上创建
>     	2. struct list_head thread_group //指向该进程所在进程组中所有进程组成的链表
>     	3. struct task_struct *real_parent //指向创建当前进程的父进程
>     	4. struct task_struct *parent //指向当前父进程 gdb下为gdb
>     	5. struct list_head children //指向该进程的子进程链表的头部
>     	6. struct list_head sibling //指向兄弟进程链表的下一个或前一个节点
> 	4. 进程调度相关信息
>     	1. int prio //动态优先级
>     	2. static_prio //静态优先级
>     	3. normal_prio //常规动态优先级
>     	4. unsigned int rt_priority //实时进程优先级
>     	5. struct list_head run_list //记录进程在就绪队列rq中的位置
>     	6. const struct sched_class *sched_class //进程所采用的调度器类。（Linux系统根据调度策略不同设置了五种调度器类：stop_sched_class -> dl_sched_class -> rt_sched_class -> fair_sched_class -> idle_sched_class，链接成一个简单的单向列表，且各调度器类的优先级沿链表顺序递减
>     	7. struct sched_entity se //普通进程调度实体
>     	8. struct sched_rt_entity rt //实时进程调度实体
>     	9. unsigned int policy //本进程采用的调度策略
>     	10. int on_cpu //本进程当前由哪个cpu来运行
>     	11. cpumask_t cpus_allowed //在多处理器环境下，规定本进程能在哪些cpu上运行
>     	12. unsigned int time_slice //进程剩余时间片长度
>     	13. unsigned long nvcsw, nivcsw 表示进程的上下文切换次数，nvcsw主动切换的次数，nivcsw记录被动切换次数（内核被抢占的次数
> 	5. 进程地址空间及文件系统信息
>     	1. struct mm_struct *mm //进程用户地址空间描述符，指向由若干vm_area_struct描述的虚存块。内核线程的mm字段为NULL
>     	2. struct mm_struct *active_mm //指向进程最近常使用的地址空间，普通进程active_mm和mm相同
>     	3. struct fs_struct *fs //表示进程与文件系统的联系，如进程的当前目录和根目录
>     	4. struct files_struct *files //记录进程当前打开的文件，前三项分别预先设置为标准输入、标准输出和出错信息输出文件
> 	6. void *stack //指向进程内核栈始址
> 	7. 进程信号处理相关信息
>     	1. sigset_t saved_sigmask //进程的信号掩码，置位表示屏蔽，复位表示不屏蔽
>     	2. struct signal_struct *signal //指向进程的信号描述符
>     	3. struct sighand_struct *sighand //指向进程的信号处理程序描述符
>     	4. struct sigpending pending //记录进程所有已经触发但是还没有处理的信号
>     	5. sigset_t blocked, real_blocked //被阻塞信号的掩码，后者表示临时掩码
> 	8. 时间及定时器相关信息
>     	1. cputime_t utime, stime //记录进程在用户态(utime)和内核态(stime)的运行时间
>     	2. u64 start_time //进程创建时间
>     	3. unsigned long timer_slack_ns //进程不活跃时间，单位ns，常用于poll()（文件描述符轮询函数）和select()（文件描述符轮询函数）

###### 进程创建

> 1. fork() vfork() clone()
>    1. fork() //创建一个普通进程，调用一次返回两次：在子进程中返回0，在父进程中返回子进程的pid
>    2. vfork() //子进程能贡献父进程的地址空间，且父进程会一直阻塞，直到子进程调用exit()终止运行或调用exec()加载另一个可执行文件，即子进程优先运行
>    3. clone() //接受一个指向某函数的指针和该函数的参数，刚创建的子进程将马上执行该函数。clone()允许子进程有选择性地继承父进程的资源，因此通常用它来创建线程
> 2. do_fork()
>    1. 定义在kernel/fork.c文件中，完成进程创建的所有工作，其间调用copy_process()完成创建进程的大部分工作
>    2. 设置新进程的进程空间创建和描述符域定义的大部分工作
>       1. 调用dup_task_struct()为新进程创建内核栈、thread_info和task_struct结构
>       2. 检查进程资源限制、用户拥有进程数量限制、系统进程数量限制等，若超出限制则出错返回
>       3. 初始化task_struct中部分信息：互斥变量及一些统计信息
>       4. 调用sched_fork()设置新进程初始调度相关信息，如分配运行cpu、建立调度实体并将状态置为TASK_RUNNING状态
>       5. 根据clone_flags（创建进程时的控制标识信息）的值调用一组copy_操作复制父进程的内容，如信号量信息、文件系统信息、信号信息、地址空间信息等，但只是设置新进程的相关指针，并增加资源数据结构的引用计数。
>       6. 调用copy_thread()复制父进程的内核栈给新进程，并将新进程内核栈中的eax设置为0，因此当从子进程返回时，fork()的返回值为0
>       7. 为新进程分配pid，并建立pid结构
>       8. 初始化表示进程间关系的相关字段，并将新进程插入若干链表
>       9. 完成扫尾工作并返回新进程的task_struct结构的指针