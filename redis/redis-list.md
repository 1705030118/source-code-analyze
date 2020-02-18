- 特性
> 双端、无环、带表头指针和表尾指针、带链表长度计数器、多态
- list
```s
typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len; // 链表所包含的节点数量
} list;
```
- listNode
```s
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value; // 链表可以保存不同类型的值
} listNode;
```