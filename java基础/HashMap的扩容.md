```

void resize(int newCapacity) {                   //传入新的容量
        HashMapEntry[] oldTable = table;         //引用扩容前的Entry数组
        int oldCapacity = oldTable.length;      
        if (oldCapacity == MAXIMUM_CAPACITY) {   //扩容前的数组大小如果已经达到最大(2^30)了  
            threshold = Integer.MAX_VALUE;       //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
            return;
        }

        HashMapEntry[] newTable = new HashMapEntry[newCapacity]; //初始化一个新的Entry数组
        transfer(newTable);                       //！！将数据转移到新的Entry数组里 
        table = newTable;                         //HashMap的table属性引用新的Entry数组  
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1); //修改阈值
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