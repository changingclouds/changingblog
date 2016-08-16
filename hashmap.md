###HashMap
	* 介绍HashMap的原理
	基于哈希表的 Map 接口的实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键,此类不保证映射的顺序。
	
	HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象。”这里关键点在于指出，HashMap是在bucket中储存键对象和值对象，作为Map.Entry。这一点有助于理解获取对象的逻辑。如果你没有意识到这一点，或者错误的认为仅仅只在bucket中存储值的话，你将不会回答如何从HashMap中获取对象的逻辑。这个答案相当的正确，也显示出面试者确实知道hashing以及HashMap的工作原理。
	
	* 底层结构
	HashMap底层就是一个数组结构，数组中的每一项又是一个链表。

	* 当两个对象的hashcode相同会发生什么？
	因为hashcode相同，所以它们的bucket[ˈbʌkɪt]位置相同，‘碰撞’会发生。因为HashMap使用LinkedList存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在LinkedList中。
	
	* 如果两个键的hashcode相同，你如何获取值对象？
	当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。面试官提醒他如果有两个值对象储存在同一个bucket，他给出答案:将会遍历LinkedList直到找到值对象。面试官会问因为你并没有值对象去比较，你是如何确定确定找到值对象的？除非面试者直到HashMap在LinkedList中存储的是键值对，否则他们不可能回答出这一题。

　　其中一些记得这个重要知识点的面试者会说，找到bucket位置之后，会调用keys.equals()方法去找到LinkedList中正确的节点，最终找到要找的值对象。完美的答案！

　　许多情况下，面试者会在这个环节中出错，因为他们混淆了hashCode()和equals()方法。因为在此之前hashCode()屡屡出现，而equals()方法仅仅在获取值对象的时候才出现。一些优秀的开发者会指出使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。

	* 如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？
		默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

	* 那么在这个位置上的元素将以链表的形式存放，新加入的放在链头，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。

	* 为什么String, Interger这样的wrapper类适合作为键？
	
	String, Interger这样的wrapper类作为HashMap的键是再适合不过了，而且String最为常用。因为String是不可变的，也是final的，而且已经重写了equals()和hashCode()方法了。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。不可变性还有其他的优点如线程安全。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的，那么请这么做吧。因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的。如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小些，这样就能提高HashMap的性能。
	
	* Fail-Fast机制：
   我们知道java.util.HashMap不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了map，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。
   这一策略在源码中的实现是通过modCount域，modCount顾名思义就是修改次数，对HashMap内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。

	* 我们可以使用自定义的对象作为键吗？
	 这是前一个问题的延伸。当然你可能使用任何对象作为键，只要它遵守了equals()和hashCode()方法的定义规则，并且当对象插入到Map中之后将不会再改变了。如果这个自定义对象时不可变的，那么它已经满足了作为键的条件，因为当它创建之后就已经不能改变了。
	 
	* 我们可以使用CocurrentHashMap来代替HashTable吗？
	
	这是另外一个很热门的面试题，因为ConcurrentHashMap越来越多人用了。我们知道HashTable是synchronized的，但是ConcurrentHashMap同步性能更好，因为它仅仅根据同步级别对map的一部分进行上锁。ConcurrentHashMap当然可以代替HashTable，但是HashTable提供更强的线程安全性。看看这
	
	* 解决hash冲突的办法
	
	开放定址法（线性探测再散列，二次探测再散列，伪随机探测再散列）
	再哈希法
	链地址法
	建立一个公共溢出区
	Java中hashmap的解决办法就是采用的链地址法
	
	* 再散列rehash过程	当哈希表的容量超过默认容量时，必须调整table的大小。当容量已经达到最大可能值时，那么该方法就将容量调整到Integer.MAX_VALUE返回，这时，需要创建一张新表，将原表的映射到新表中。

	* index相关问题
	`
	final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

	static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }

	`

	1当调用put(key,value)时，首先获取key的hashcode，int hash = key.hashCode(); 
    2再把hash通过一下运算得到一个int h.
	为什么要经过这样的运算呢？这就是HashMap的高明之处。先看个例子，一个十进制数32768(二进制1000 0000 0000 0000)，经过上述公式运算之后的结果是35080(二进制1000 1001 0000 1000)。看出来了吗？或许这样还看不出什么，再举个数字61440(二进制1111 0000 0000 0000)，运算结果是65263(二进制1111 1110 1110 1111)，现在应该很明显了，它的目的是让“1”变的均匀一点，散列的本意就是要尽量均匀分布。那这样有什么意义呢？看第3步。 
	3得到h之后，把h与HashMap的承载量（HashMap的默认承载量length是16，可以自动变长。在构造HashMap的时候也可以指定一个长 度。这个承载量就是上图所描述的数组的长度。）进行逻辑与运算，即 h & (length-1)，这样得到的结果就是一个比length小的正数，我们把这个值叫做index。其实这个index就是索引将要插入的值在数组中的 位置。第2步那个算法的意义就是希望能够得出均匀的index，这是HashTable的改进，HashTable中的算法只是把key的 hashcode与length相除取余，即hash % length，这样有可能会造成index分布不均匀。还有一点需要说明，HashMap的键可以为null，它的值是放在数组的第一个位置。 
	4我们用table[index]表示已经找到的元素需要存储的位置。先判断该位置上有没有元素（这个元素是HashMap内部定义的一个类Entity， 基本结构它包含三个类，key，value和指向下一个Entity的next）,没有的话就创建一个Entity对象，在 table[index]位置上插入，这样插入结束；如果有的话，通过链表的遍历方式去逐个遍历，看看有没有已经存在的key，有的话用新的value替 换老的value；如果没有，则在table[index]插入该Entity，把原来在table[index]位置上的Entity赋值给新的 Entity的next，这样插入结束。 
	总结：key->hashcode->h->index->遍历链表->插入 
