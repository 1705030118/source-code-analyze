- 结构体
```s
typedef struct spinlock {
	union {
		struct raw_spinlock rlock;
                //中间的DEBUG删除了
	      };
} spinlock_t;
```
- 初始化
```s
#define spin_lock_init(_lock)				
do {							
	spinlock_check(_lock);				
	raw_spin_lock_init(&(_lock)->rlock);		
} while (0)
```
- 上锁函数
```s
//          一般的上锁
static inline void spin_lock(spinlock_t *lock)
{
	raw_spin_lock(&lock->rlock);
}
//           通常用在进程中，用来禁止抢断和禁止软中断
static inline void spin_lock_bh(spinlock_t *lock)
{
	raw_spin_lock_bh(&lock->rlock);
}
//          判断有没有上锁
static inline int spin_trylock(spinlock_t *lock)
{
	return raw_spin_trylock(&lock->rlock);
}
//           既禁止本地中断，又禁止内核抢占
static inline void spin_lock_irq(spinlock_t *lock)
{
	raw_spin_lock_irq(&lock->rlock);
}
```
- 解锁函数
```s
static inline void spin_unlock(spinlock_t *lock)
{
	raw_spin_unlock(&lock->rlock);
}
 
static inline void spin_unlock_bh(spinlock_t *lock)
{
	raw_spin_unlock_bh(&lock->rlock);
}
 
static inline void spin_unlock_irq(spinlock_t *lock)
{
	raw_spin_unlock_irq(&lock->rlock);
}
 
static inline void spin_unlock_irqrestore(spinlock_t *lock, unsigned long flags)
{
	raw_spin_unlock_irqrestore(&lock->rlock, flags);
}
```
- 上锁并保存之前的中断屏蔽状态
  
```s
#define spin_lock_irqsave(lock, flags)				\
do {								\
	raw_spin_lock_irqsave(spinlock_check(lock), flags);	\
} while (0)
static inline void spin_unlock_irqrestore(spinlock_t *lock, unsigned long flags)
{
	raw_spin_unlock_irqrestore(&lock->rlock, flags);
}
```
- 使用
```s
spinlock t lock;
unsigned int flags;
spin_lock_init (&lock);
spin_lock_irqsave(&lock, flags)	;  //上锁
/***********/
 
//临界资源
 
/*************/
spin_unlock_irqrestore(&lock, flags);  //解锁
```
- 注意
>自旋锁是一种忙等待

Linux中， 自旋锁当条件不满足时，会一直不断地循环条件是否被满足。如果满足，就解锁，继续运行下面的代码。这种忙等待机制是否对系统的性能有所影响呢?答案是肯定的。内核这样设计自旋锁确定对系统的性能，有所影响，所以在实际编程中，程序员应该注意自旋锁不应该长时间地持有，它 "是一种适合短时间锁定的轻量级的加锁机制