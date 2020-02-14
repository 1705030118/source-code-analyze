- 成员变量
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
     // hashMap数组的初始容量 16
     static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
     // 负载因子 0.75f;
     static final float DEFAULT_LOAD_FACTOR = 0.75f;
     // 树形化阈值 8，若桶中链表元素超过8时，会自动尝试转化成红黑树
     static final int TREEIFY_THRESHOLD = 8;
     // 解除树形化阈值 6，若桶中元素小于等于6时，树结构还原成链表形式。
     static final int UNTREEIFY_THRESHOLD = 6;
     // 树形化的另一条件 Map数组的长度阈值 64
     // 树形化阈值的第二条件。当数组的长度小于这个值时，就算树形化阈达标，链表也不会转化为红黑树，而是优先扩容数组resize()。
     static final int MIN_TREEIFY_CAPACITY = 64
     // 这个就是hashMap的内部数组了，而Node则是链表节点对象。
     transient Node<K,V>[] table;
     // 数组扩容阈值，扩容的容量为当前 HashMap 总容量的两倍。
     int threshold;
}
```
- 节点类
```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        java.util.HashMap.Node<K, V> next;

        Node(int hash, K key, V value, java.util.HashMap.Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
 static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        java.util.HashMap.TreeNode<K, V> parent;  // red-black tree links
        java.util.HashMap.TreeNode<K, V> left;
        java.util.HashMap.TreeNode<K, V> right;
        java.util.HashMap.TreeNode<K, V> prev;    // needed to unlink next upon deletion
        boolean red;

        TreeNode(int hash, K key, V val, java.util.HashMap.Node<K, V> next) {
            super(hash, key, val, next);
        }
    }
```
- hash方法
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    // 用hashCode的高16位与低16位进行异或运算
        static final int hash(Object key) {
            int h;
            return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
        }
}

```
- get方法
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    public V get(Object key) {
            Node<K, V> e;
            // 也会获取节点时也调用了hash()方法
            return (e = getNode(hash(key), key)) == null ? null : e.value;
        }
    
    final Node<K, V> getNode(int hash, Object key) {
            // tab：内部数组  first: 索引位首节点 n: 数组长度 k: 索引位首节点的key
            Node<K, V>[] tab;
            Node<K, V> first, e;
            int n;
            K k;
            // 数组不为null 数组长度大于0 索引位首节点不为null
            if ((tab = table) != null && (n = tab.length) > 0 &&
                    (first = tab[(n - 1) & hash]) != null) {
                // 如果索引位首节点的hash==key的hash 或者 key和索引位首节点的k相同
                if (first.hash == hash && // always check first node
                        ((k = first.key) == key || (key != null && key.equals(k))))
                    // 返回索引位首节点(值对象)
                    return first;
                if ((e = first.next) != null) {
                    // 如果是红黑色则到红黑树中查找
                    if (first instanceof TreeNode)
                        return ((TreeNode<K, V>) first).getTreeNode(hash, key);
                    do {
                        // 发送碰撞 key.equals(k)
                        if (e.hash == hash &&
                                ((k = e.key) == key || (key != null && key.equals(k))))
                            return e;
                    } while ((e = e.next) != null);
                }
            }
            return null;
        }
}
```
- put方法
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    public V put(K key, V value) {
            return putVal(hash(key), key, value, false, true);
        }
    
        // onlyIfAbsent：当存入键值对时，如果该key已存在，是否覆盖它的value。false为覆盖，true为不覆盖 参考putIfAbsent()方法。
        // evict：用于子类LinkedHashMap。
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
            HashMap.Node<K, V>[] tab; // tab：内部数组
            HashMap.Node<K, V> p;   // p：hash对应的索引位中的首节点
            int n, i;  // n：内部数组的长度    i：hash对应的索引位
    
            // 首次put时，内部数组为空，扩充数组。
            if ((tab = table) == null || (n = tab.length) == 0)
                n = (tab = resize()).length;
            // 计算数组索引，获取该索引位置的首节点，如果为null，添加一个新的节点
            if ((p = tab[i = (n - 1) & hash]) == null)
                tab[i] = newNode(hash, key, value, null);
            else {
                HashMap.Node<K, V> e;
                K k;
                // 如果首节点的key和要存入的key相同，那么直接覆盖value的值。
                if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                    e = p;
                    // 如果首节点是红黑树的，将键值对插添加到红黑树
                else if (p instanceof HashMap.TreeNode)
                    e = ((HashMap.TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
                    // 此时首节点为链表，如果链表中存在该键值对，直接覆盖value。
                    // 如果不存在，则在末端插入键值对。然后判断链表是否大于等于7，尝试转换成红黑树。
                    // 注意此处使用“尝试”，因为在treeifyBin方法中还会判断当前数组容量是否到达64，
                    // 否则会放弃次此转换，优先扩充数组容量。
                else {
                    // 走到这里，hash碰撞了。检查链表中是否包含key，或将键值对添加到链表末尾
                    for (int binCount = 0; ; ++binCount) {
                        // p.next == null，到达链表末尾，添加新节点，如果长度足够，转换成树结构。
                        if ((e = p.next) == null) {
                            p.next = newNode(hash, key, value, null);
                            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                                treeifyBin(tab, hash);
                            break;
                        }
                        // 检查链表中是否已经包含key
                        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                            break;
                        p = e;
                    }
                }
    
                // 覆盖value的方法。
                if (e != null) { // existing mapping for key
                    V oldValue = e.value;
                    if (!onlyIfAbsent || oldValue == null)
                        e.value = value;
                    afterNodeAccess(e);
                    return oldValue;
                }
            }
            ++modCount; // fail-fast机制
    
            // 如果元素个数大于阈值，扩充数组。
            if (++size > threshold)
                resize();
            afterNodeInsertion(evict);
            return null;
        }
}
```
- resize数组扩容
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    final HashMap.Node<K, V>[] resize() {
            HashMap.Node<K, V>[] oldTab = table;
            int oldCap = (oldTab == null) ? 0 : oldTab.length;
            int oldThr = threshold;
            int newCap, newThr = 0;
            if (oldCap > 0) {
                // 如果数组已经是最大长度，不进行扩充。
                if (oldCap >= MAXIMUM_CAPACITY) {
                    threshold = Integer.MAX_VALUE;
                    return oldTab;
                }
                // 否则数组容量扩充一倍。（2的N次方）
                else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                        oldCap >= DEFAULT_INITIAL_CAPACITY)
                    newThr = oldThr << 1; // double threshold
            }
            // 如果数组还没创建，但是已经指定了threshold（这种情况是带参构造创建的对象），threshold的值为数组长度
            // 在 "构造函数" 那块内容进行过说明。
            else if (oldThr > 0) // initial capacity was placed in threshold
                newCap = oldThr;
                // 这种情况是通过无参构造创建的对象
            else {               // zero initial threshold signifies using defaults
                newCap = DEFAULT_INITIAL_CAPACITY;
                newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
            }
            // 可能是上面newThr = oldThr << 1时，最高位被移除了，变为0。
            if (newThr == 0) {
                float ft = (float) newCap * loadFactor;
                newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                        (int) ft : Integer.MAX_VALUE);
            }
            threshold = newThr;
    
            // 到了这里，新的数组长度已经被计算出来，创建一个新的数组。
            @SuppressWarnings({"rawtypes", "unchecked"})
            HashMap.Node<K, V>[] newTab = (HashMap.Node<K, V>[]) new HashMap.Node[newCap];
            table = newTab;
    
            // 下面代码是将原来数组的元素转移到新数组中。问题在于，数组长度发生变化。 
            // 那么通过hash%数组长度计算的索引也将和原来的不同。
            // jdk 1.7中是通过重新计算每个元素的索引，重新存入新的数组，称为rehash操作。
            // 这也是hashMap无序性的原因之一。而现在jdk 1.8对此做了优化，非常的巧妙。
            if (oldTab != null) {
    
                // 遍历原数组
                for (int j = 0; j < oldCap; ++j) {
                    // 取出首节点
                    HashMap.Node<K, V> e;
                    if ((e = oldTab[j]) != null) {
                        oldTab[j] = null;
                        // 如果链表只有一个节点，那么直接重新计算索引存入新数组。
                        if (e.next == null)
                            newTab[e.hash & (newCap - 1)] = e;
                            // 如果该节点是红黑树，执行split方法，和链表类似的处理。
                        else if (e instanceof HashMap.TreeNode)
                            ((HashMap.TreeNode<K, V>) e).split(this, newTab, j, oldCap);
    
                            // 此时节点是链表
                        else { // preserve order
                            // loHead，loTail为原链表的节点，索引不变。
                            HashMap.Node<K, V> loHead = null, loTail = null;
                            // hiHeadm, hiTail为新链表节点，原索引 + 原数组长度。
                            HashMap.Node<K, V> hiHead = null, hiTail = null;
                            HashMap.Node<K, V> next;
    
                            // 遍历链表
                            do {
                                next = e.next;
                                // 新增bit为0的节点，存入原链表。
                                if ((e.hash & oldCap) == 0) {
                                    if (loTail == null)
                                        loHead = e;
                                    else
                                        loTail.next = e;
                                    loTail = e;
                                }
                                // 新增bit为1的节点，存入新链表。
                                else {
                                    if (hiTail == null)
                                        hiHead = e;
                                    else
                                        hiTail.next = e;
                                    hiTail = e;
                                }
                            } while ((e = next) != null);
                            // 原链表存回原索引位
                            if (loTail != null) {
                                loTail.next = null;
                                newTab[j] = loHead;
                            }
                            // 新链表存到：原索引位 + 原数组长度
                            if (hiTail != null) {
                                hiTail.next = null;
                                newTab[j + oldCap] = hiHead;
                            }
                        }
                    }
                }
            }
            return newTab;
        }
}
```
- 树化（链表转红黑树）
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    final void treeifyBin(Node<K, V>[] tab, int hash) {
            int n, index;
            Node<K, V> e;
            // 如果当前数组容量太小（小于64），放弃转换，扩充数组。
            if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY) {
                resize();
            } else if ((e = tab[index = (n - 1) & hash]) != null) {
                // 将链表转成红黑树...
            }
        }
}    
```