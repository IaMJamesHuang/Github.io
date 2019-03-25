# Hashmap的数据结构
Hashmap的结构比较简单，如下图。

![image.png](img/index_6.png)

# Hash算法
java中的hash算法是
```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
但是这个算法并不是求桶下标的方法，这个方法的作用只是在h小于2^16的情况下，保持对象的hash值不变。

注：**h >>> 16**是指h无符号右移16位，即任何小于2^16的数字右移16位后都是0，而^是异或符号，相同取0，不同取1。即：
```
1001 ^ 1010 = 0011
1010 ^ 0000 = 1010
```
所有数字与0异或结果都等于本身。

而真正决定对象被放到哪个桶的代码是：
```
tab[i = (n - 1) & hash]
```

考虑以下情况，左边是容量为16（与16-1 = 15按位与）的时候的情况，而右边是容量为15的情况。其分别对hash值为8和9的对象做映射，可以看到，左边这两个值被映射到了不同的桶，而右边这两个值被映射到了同一个桶。

![image.png](img/index_5.png)

所以，为了减低hashmap的冲突率，所以hashmap的容量会被设置为2的n次方，就算传入的初始容量不是2的n次方，也会被转化为最接近的2的n次方（大于传入的初始容量）。
```
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        //上面的代码是使n的二进制数的低位全部变为1，比如10，11变为11，100，101，110，111变为111
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```


# Hashmap中的负载因子
当hashmap中的元素越来越多的时候，碰撞的几率越来越高。为了提高查询的效率，需要对hashmap进行扩容（增加桶的个数）。

当hashmap中的元素个数超过数组大小\*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，也就是说，默认情况下，数组大小为16，那么当hashmap中元素个数超过16*0.75=12的时候，就把数组的大小扩展为2\*16=32。即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作。

如果我们已经预知hashmap中元素的个数，那么预设元素的个数能够有效的提高hashmap的性能。比如说，我们有1000个元素new HashMap(1000), 但是理论上来讲new HashMap(1024)更合适，不过上面已经说过，即使是1000，hashmap也自动会将其设置为1024。 但是new HashMap(1024)还不是更合适的，因为0.75*1000 < 1000, 也就是说为了让0.75 * size > 1000, 我们必须这样new HashMap(2048)才最合适，既考虑了&的问题，也避免了resize的问题。 

注意，loadFactor越小，说明hashmap越稀疏，碰撞率越低，查询效率越高，但是同时空间的利用率也越低。反之亦然。