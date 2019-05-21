# 操作系统学习

### 进程管理

##### Linux进程管理

###### 	进程描述符task_struct:

 	1. 进程状态
      	1. state
      	2. exit_state
 	2. 相关标识符信息
      	1. pid_t pid //进程标识符 PID
      	2. pid_t tgid //线程组零头线程的PID&线程组所属进程的PID
      	3. pid_t pgrp //进程所属进程组的领头进程的PID
      	4. session //进程所属会话的领头进程的PID
      	5. uid //进程的用户ID
      	6. gid //进程属于哪个用户组
      	7. euid&egid //有效的uid&gid 安全权限
      	8. suid&sgid //备份的uid&gid
      	9. fsuid&fsgid //文件系统的uid&gid，用于对文件系统操作时的合法检查。（NFS
	3. 家族关系
    	1. struct task_struct *group_leader //指向所属进程组的领头进程的指针 有自己的终端tty，当前进程就是从这个终端上创建
    	2. struct list_head thread_group //指向该进程所在进程组中所有进程组成的链表
    	3. struct task_struct *real_parent //指向创建当前进程的父进程
    	4. struct task_struct *parent //指向当前父进程 gdb下为gdb
    	5. struct list_head children //指向该进程的子进程链表的头部
    	6. struct list_head sibling //指向兄弟进程链表的下一个或前一个节点
	4. 进程调度相关信息
    	1. int prio //动态优先级
    	2. static_prio //静态优先级
    	3. normal_prio //常规动态优先级
    	4. unsigned int rt_priority //实时进程优先级
    	5. struct list_head run_list //记录进程在就绪队列rq中的位置
    	6. const struct sched_class *sched_class //进程所采用的调度器类。（Linux系统根据调度策略不同设置了五种调度器类：stop_sched_class -> dl_sched_class -> rt_sched_class -> fair_sched_class -> idle_sched_class，链接成一个简单的单向列表，且各调度器类的优先级沿链表顺序递减
    	7. struct sched_entity se //普通进程调度实体
    	8. struct sched_rt_entity rt //实时进程调度实体
    	9. unsigned int policy //本进程采用的调度策略
    	10. int on_cpu //本进程当前由哪个cpu来运行
    	11. cpumask_t cpus_allowed //在多处理器环境下，规定本进程能在哪些cpu上运行
    	12. unsigned int time_slice //进程剩余时间片长度
    	13. unsigned long nvcsw, nivcsw 表示进程的上下文切换次数，nvcsw主动切换的次数，nivcsw记录被动切换次数（内核被抢占的次数
	5. 进程地址空间及文件系统信息
    	1. struct mm_struct *mm //进程用户地址空间描述符，指向由若干vm_area_struct描述的虚存块。内核线程的mm字段为NULL
    	2. struct mm_struct *active_mm //指向进程最近常使用的地址空间，普通进程active_mm和mm相同
    	3. struct fs_struct *fs //表示进程与文件系统的联系，如进程的当前目录和根目录
    	4. struct files_struct *files //记录进程当前打开的文件，前三项分别预先设置为标准输入、标准输出和出错信息输出文件
    	5. 