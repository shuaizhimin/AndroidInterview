## 问题
* HashMap 结构
* HashMap 扩容、扩容大小
* HashMap hashCode Hash冲突碰撞解决

## HashMap 结构

![](http://olax25rea.bkt.clouddn.com/1024555-20161113235348670-746615111.png)

HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度依然为O(1)，因为最新的Entry会插入链表头部，急需要简单改变引用链即可，而对于查找操作来讲，此时就需要遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好。

初始的时候并未为数组创建空间,put时候才真正构建table数组

## HashMap 扩容、扩容大小
HashMap初始大小为16

```
    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }
        createEntry(hash, key, value, bucketIndex);
    }
    
     void resize(int newCapacity) {
        HashMapEntry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
        HashMapEntry[] newTable = new HashMapEntry[newCapacity];
        transfer(newTable);
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }

    void transfer(HashMapEntry[] newTable) {
        int newCapacity = newTable.length;
        for (HashMapEntry<K,V> e : table) {
            while(null != e) {
                HashMapEntry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```
当数组长度大于阈值时,初始化为当前table的两倍,当超过1<<30时,扩容为Max_VALUE

## HashMap hashCode Hash冲突碰撞解决
Hash冲突解决方法:

* 开放地址(线性探测再散列，二次探测再散列，伪随机探测再散列)
* 再哈希法
* 链地址法
* 建立一个公共溢出区

HashMap 里面采用的是链地址法  这也就是HashMap为什么为数组+链表的组成