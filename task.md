- 进程描述符task_struct
```s
struct task_struct {
	/* ... */
	/* -1 unrunnable, 0 runnable, >0 stopped: */
	volatile long			state;		/* 进程状态位 */

	/* ... */

	void				*stack;		/* 指向内核栈 */

	/* ... */

	/* 优先级 */
	int				prio;
	int				static_prio;	/* 普通进程的静态优先级 */
	int				normal_prio;
	unsigned int			rt_priority;

	/* ... */

	/* 调度策略 */
	unsigned int			policy;

	cpumask_t			cpus_mask;	/* 允许进程在哪些CPU执行, cpuset相关 */

	struct sched_info		sched_info;	/* 调度信息 */

	struct mm_struct		*mm;		/* 内存描述符 */

	struct vmacache			vmacache;	/* 虚拟内存管理 */

	/* 进程退出相关 */
	int				exit_state;
	int				exit_code;
	int				exit_signal;
	/* The signal sent when the parent dies: */
	int				pdeath_signal;

	/* ... */

	pid_t				pid;   /* 进程号 */
	pid_t				tgid;  /* 线程组标识符 */

	/* Real parent process: */
	struct task_struct __rcu	*real_parent;		/* 指向真实父进程 */

	/* ... */
	char				comm[TASK_COMM_LEN];	/* 进程名 */

	/* Filesystem information: */
	struct fs_struct		*fs;		/* 文件系统描述符 */

	/* Open file information: */
	struct files_struct		*files;		/* 打开的文件信息 */

	/* 信号处理相关 */
	struct signal_struct		*signal;
	struct sighand_struct		*sighand;
	sigset_t			blocked;
	sigset_t			real_blocked;

	/* ... */
	struct thread_struct		thread;  /* CPU的部分状态(寄存器)保存在thread中 */

	/* ... */
};
```
- 进程状态
```s
/* Used in tsk->state: */
#define TASK_RUNNING			0x0000 // 运行
#define TASK_INTERRUPTIBLE		0x0001 // 可中断
#define TASK_UNINTERRUPTIBLE		0x0002 // 不可中断
#define __TASK_STOPPED			0x0004 // 停止执行
#define __TASK_TRACED			0x0008 // 被其他进程跟踪
/* Used in tsk->exit_state: */
#define EXIT_DEAD			0x0010 // 退出
#define EXIT_ZOMBIE			0x0020
```