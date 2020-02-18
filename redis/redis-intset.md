- 特性
> 整数集合是集合键的底层实现之一、底层实现为数组（有序、唯一）、升级操作（操作灵活性、节约内存）、不支持降级
- intset
```s
typedef struct intset {
    uint32_t encoding;
    uint32_t length;
    int8_t contents[];// 从小到大，唯一
} intset;
```