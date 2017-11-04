### Set
 * Set无序、不可重复。取出元素的方法只有迭代器。不可以存放重复元素，元素存取是无序的
 * HashSet 无序，不可重复
   使用原因:假如我们现在想要在一大堆数据中查找X数据。LinkedList的数据结构就不说了，查找效率低的可怕。ArrayList哪，如果我们不知道X的位置序号，还是一样要全部遍历一次直到查到结果，效率一样可怕。HashSet天生就是为了提高查找效率的
   public boolean contains(Object o) {
            return containsKey(o);
        }
   不能重复存储equal相同的数据，原因就是equals相同，数据的散列码也就相同（hashCode必须和equals兼容）。大量相同的数据将存放在同一个散列单元所指向的链表中，造成严重的散列冲突，对查找效率是灾难性的
 * TreeSet 有序、可进行排序

###List 
 * Verctor 线程安全
 * ArrayList 线程不安全、查询速度快 --->why
 
 * LinkedList 线程不安全、增删速度快

### Map
 * Hashtable
   线程安全，速度快
 * HashMap
   线程不安全，速度慢
 * LinkedHashMap
   保证HashMap有序
 * TreeMap
   

### SortedSet
### SortedMap
### HashSet
### TreeSet
### ArrayList
### LinkedList
### Vector
### Collections
### Arrays
### AbstractCollection