## **STL组件**

STL分为六大组件：

- 容器(container)：常用数据结构，大致分为两类，序列容器，如vector，list，deque，关联容器，如set，map。在实现上，是类模板(class template)
- 迭代器(iterator)：一套访问容器的接口，行为类似于指针。它为不同算法提供的相对统一的容器访问方式，使得设计算法时无需关注过多关注数据。（“算法”指广义的算法，操作数据的逻辑代码都可认为是算法）
- 算法(algorithm)：提供一套常用的算法，如sort，search，copy，erase … 在实现上，可以认为是一种函数模板(function template)。
- 配置器(allocator)：为容器提供空间配置和释放，对象构造和析构的服务，也是一个class template。
- 仿函数(functor)：作为函数使用的对象，用于泛化算法中的操作。
- 配接器(adapter)：将一种容器修饰为功能不同的另一种容器，如以容器vector为基础，在其上实现stack，stack的行为也是一种容器。这就是一种配接器。除此之外，还有迭代器配接器和仿函数配接器。

## STL 常用函数总结

https://blog.csdn.net/codedz/article/details/110493577

## vector

https://blog.csdn.net/XIONGXING_xx/article/details/115168163

vector维护的是一个**连续线性空间**，所以vector支持随机存取 。

vector动态增加大小时，并不是在原空间之后持续新空间（因为无法保证原空间之后尚有可供配置的空间），而是以**原大小的两倍**另外配置一块较大的空间，然后**将原内容拷贝过来，然后才开始在原内容之后构造新元素，并释放原空间**。因此， 对vector的任何操作，一旦引起空间重新配置，**指向原vector的所有迭代器就都失效了** 。这是易犯的一个错误，务需小心。



**使用shrink_to_fit函数**

在c++11以后增加了这个函数，它的作用是释放掉未使用的内存，假设我们先调用clear函数把所有元素清掉，这样是不是整块容器都变成未使用了，然后再调用shrink_to_fit函数把未使用的部分内存释放掉，那不就把这个vector内存释放掉了吗。



释放vector内存最简单的方法是`vector.swap(nums)`



**访问元素使用 .at()，用安全检查是否越界。**



#### vector 的reserve 和 resize的区别

1、vector的reserve增加了vector的capacity，但是它的size没有改变！而resize改变了vector的capacity同时也增加了它的size！
2、reserve是容器预留空间，**但在空间内不真正创建元素对象**，所以在没有添加新的对象之前，不能引用容器内的元素。加入新的元素时，要调用push_back()/insert()函数。
3、resize是改变容器的大小，且在创建对象，因此，调用这个函数之后，就可以引用容器内的对象了，因此当加入新的元素时，用operator[]操作符，或者用迭代器来引用元素对象。此时再调用push_back()函数，是加在这个新的空间后面的。
4、两个函数的参数形式也有区别的，reserve函数之后一个参数，即需要预留的容器的空间；resize函数可以有两个参数，第一个参数是容器新的大小，第二个参数是要加入容器中的新元素，如果这个参数被省略，那么就调用元素对象的默认构造函数。
————————————————
版权声明：本文为CSDN博主「JMW1407」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/JMW1407/article/details/108785448



## 内存管理

https://blog.csdn.net/qq_36721032/article/details/107111319

堆和栈相比，由于大量new/delete的使用，容易造成大量的内存碎片；由于没有专门的系统支持，效率很低；由于可能引发用户态和核心态的切换，内存的申请，代价变得更加昂贵。所以栈在程序中是应用最广泛的，就算是函数的调用也利用栈去完成，函数调用过程中的参数，返回地址，EBP和局部变量都采用栈的方式存放。所以，我们推荐大家尽量用栈，而不是用堆。

应用程序一般使用malloc，realloc，new等函数从堆中分配到一块内存，使用完后，程序必须负责相应的调用free或delete释放该内存块，否则，这块内存就不能被再次使用，我们就说这块**内存泄漏**了。



## string类

string继承自basic_string,其实是**对char* 进行了封装**，封装的string包含了char*数组，容量，长度等等属性。

string可以进行动态扩展，**在每次扩展的时候另外申请一块原空间大小两倍的空间（2*n）**，然后将原字符串拷贝过去，并加上新增的内容。

https://blog.csdn.net/wzh1378008099/article/details/105687998

初始化的方式有很多种，上面那个链接的开头就注明了，可以仔细观看。

字符串截取：

```c++
//左闭右开

str.substr(2);
//返回 str[2]~str[n-1], 不对 str 进行操作

str.substr(2,5);
返回 str[2]~str[5-1], 不对 str 进行操作
```

字符串删除：

```c++
str.erase(3);
//删除 str[3]~str[n-1] , 返回 str
//例: str="0123456789"
//运行之后, str="012"

str.erase(3, 2);
//删除 str[3]~str[3+2-1] , 返回 str
```



### string源码实现(用char*)

```c++
#include <cstring>  // For string manipulation functions

class string {
private:
    char* data;       // 指向字符串数据的指针
    size_t length;    // 字符串的长度

public:
    // 默认构造函数
    string() : data(nullptr), length(0) {}
    // 构造函数
    string(const char* str) {
        length = std::strlen(str);  // 计算字符串长度
        data = new char[length + 1];  // 分配内存
        std::strcpy(data, str);  // 复制字符串内容
    }

    // 拷贝构造函数
    string(const string& other) {
        length = other.length;
        data = new char[length + 1];
        std::strcpy(data, other.data);
    }

    // 析构函数
    ~string() {
        delete[] data;  // 释放内存
    }

    // 获取字符串长度
    size_t size() const {
        return length;
    }

    // 获取 C 风格的字符串
    const char* c_str() const {
        return data;
    }

    // 重载赋值运算符
    string& operator=(const string& other) {
        if (this != &other) {
            delete[] data;  // 释放原有内存
            length = other.length;
            data = new char[length + 1];
            std::strcpy(data, other.data);
        }
        return *this;
    }

    // 重载加法运算符
    string operator+(const string& other) const {
        string result;
        result.length = length + other.length;
        result.data = new char[result.length + 1];
        std::strcpy(result.data, data);
        std::strcat(result.data, other.data);
        return result;
    }
};
```


## unique 去重

这个函数只能对"**相同元素在并邻在一块的**"序列进行去重. **不能对相同元素七零八落地分布的一般序列进行去重**, 可以对一般数组进行**排序**后再用unique()实现去重目的即可。

**实现原理总结**：用一个标记符标记最后一个不重复元素，然后比较相邻元素是否重复，并更新标记。



std::unique 是 C++ 标准库中的一个算法，它用于在容器中移除相邻的重复元素，并返回指向不重复元素范围的迭代器。

std::unique 的底层实现原理如下：

1. std::unique 函数接受两个迭代器参数，表示要去除重复元素的范围。通常是一个容器的开始迭代器和结束迭代器。

2. std::unique 从范围的第二个元素开始，将当前元素与前一个元素进行比较。如果两个元素相等，则将当前元素移除。如果两个元素不相等，则将当前元素作为下一个不重复元素，并将其移动到适当的位置。

3. std::unique 使用元素类型的 operator== 运算符（或自定义的比较函数）来比较元素是否相等。

4. std::unique 返回一个迭代器，**指向不重复元素范围的末尾**。这个迭代器用于确定不重复元素的新范围。原范围中的重复元素新的元素覆盖，并且可以使用容器的 erase 函数来删除它们。即：**一个含有n个元素的序列里面有n1个不重复元素, 其余n2个是重复的.n = n1 + n2. unique()按序列顺序从头到尾把这n1个元素依次对序列的前n1个元素地址赋值, 然后剩下n2个地址里面的元素依然是原先序列后面n2个元素.**https://blog.csdn.net/qq_43008718/article/details/104122059

5. std::unique 并不会改变容器的大小，它只是将重复元素放到容器的末尾并返回一个新的终止迭代器，**需要结合 erase 函数来实际删除重复元素。**



## set

底层实现：红黑树，https://zhuanlan.zhihu.com/p/91960960

平衡二叉树的一种。（考研书本上学的是AVL树，四种操作变平衡）



https://blog.csdn.net/qq_21989927/article/details/108063613



为什么不用hash？

首先set，不像map那样是key-value对，它的key与value是相同的。关于set有两种说法，第一个是STL中的set，用的是红黑树；第二个是hash_set，底层用得是hash table。红黑树与hash table最大的不同是，红黑树是有序结构，而hash table不是。但不是说set就不能用hash，如果只是判断set中的元素是否存在，那么hash显然更合适，因为set 的访问操作时间复杂度是log(N)的，而使用hash底层实现的hash_set是近似O(1)的。然而，set应该更加被强调理解为“集合”，而集合所涉及的操作并、交、差等，即STL提供的如交集set_intersection()、并集set_union()、差集set_difference()和对称差集set_symmetric_difference在。()，都需要进行大量的比较工作，那么使用底层是有序结构的红黑树就十分恰当了，这也是其相对hash结构的优势所


**特点：去重**

当你向Set中插入元素时，Set会自动检查该元素是否已经存在于集合中。如果已经存在，则插入操作会被忽略，不会导致重复的元素被插入。



## unordered_set

`unordered_set`是一个无序集合，它基于哈希表实现，提供了快速的插入、删除和查找操作。元素在集合中的存储顺序是根据哈希值而不是插入顺序确定的。元素是唯一的。



## list

双向链表

支持在任意位置高效地插入和删除元素。它没有随机访问功能，因此访问元素的时间复杂度是线性的。然而，它对于频繁的插入和删除操作非常高效。

```C++
list数据元素插入和删除操作
push_back(elem);//在容器尾部加入一个元素
pop_back();//删除容器中最后一个元素
push_front(elem);//在容器开头插入一个元素
pop_front();//从容器开头移除第一个元素
insert(pos,elem);//在pos位置插elem元素的拷贝，返回新数据的位置。
insert(pos,n,elem);//在pos位置插入n个elem数据，无返回值。
insert(pos,beg,end);//在pos位置插入[beg,end)区间的数据，无返回值。
clear();//移除容器的所有数据
erase(beg,end);//删除[beg,end)区间的数据，返回下一个数据的位置。
erase(pos);//删除pos位置的数据，返回下一个数据的位置。
remove(elem);//删除容器中所有与elem值匹配的元素。

————————————————
版权声明：本文为CSDN博主「Allen_Xu17」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/XUCHEN1230/article/details/86535993
```

## map

map的底层实现是**红黑树**（为啥不用哈希表，因为它有序）

**它是有序键值对的集合**，它基于红黑树实现，确保键按升序排列。它提供了高效的插入、删除和查找操作，键是唯一的。

```C++
map 常用操作
	 begin()         返回指向map头部的迭代器
     clear(）        删除所有元素
     count()         返回指定元素出现的次数
     empty()         如果map为空则返回true
     end()           返回指向map末尾的迭代器
     equal_range()   返回特殊条目的迭代器对
     erase()         删除一个元素，参数可以是迭代器，可以是关键字
     find()          查找一个元素，返回迭代器
     get_allocator() 返回map的配置器
     insert()        插入元素，插入pair
     key_comp()      返回比较元素key的函数
     lower_bound()   返回键值>=给定元素的第一个位置
     max_size()      返回可以容纳的最大元素个数
     rbegin()        返回一个指向map尾部的逆向迭代器
     rend()          返回一个指向map头部的逆向迭代器
     size()          返回map中元素的个数
     swap()           交换两个map
     upper_bound()    返回键值>给定元素的第一个位置
     value_comp()     返回比较元素value的函数
```

## unordered_map

哈希表

无序键值对



unordered_map的底层实现是hashtable，采用开链法（也就是用桶）来解决哈希冲突，当桶的大小超过8时，就自动转为红黑树进行组织。

**解决哈希冲突的方式？**

1. 线性探查。该元素的哈希值对应的桶不能存放元素时，循序往后一一查找，直到找到一个空桶为止，在查找时也一样，当哈希值对应位置上的元素与所要寻找的元素不同时，就往后一一查找，直到找到吻合的元素，或者空桶。
2. 二次探查。该元素的哈希值对应的桶不能存放元素时，就往后寻找12,22,32,42…i^2个位置。
3. 双散列函数法。当第一个散列函数发生冲突的时候，使用第二个散列函数进行哈希，作为步长。
4. 开链法。在每一个桶中维护一个链表，由元素哈希值寻找到这个桶，然后将元素插入到对应的链表中，STL的hashtable就是采用这种实现方式。
5. 建立公共溢出区。当发生冲突时，将所有冲突的数据放在公共溢出区。

**如果在使用双散列函数时出现再次冲突**，可以采取以下策略处理：

1. 探测序列（Probing Sequence）：在双散列法中，如果发生冲突，可以使用探测序列来寻找下一个可用的位置。探测序列是指在哈希表中依次检查一系列位置，直到找到一个空槽位或者找遍整个哈希表。常见的探测序列包括线性探测、二次探测等。

1. 调整散列函数：如果发生再次冲突，可以尝试调整散列函数的参数，或者选择另一个散列函数。通过调整散列函数，可以改变元素在哈希表中的分布，从而减少冲突的可能性。

1. 动态调整哈希表大小：如果发生频繁的冲突，可能是因为哈希表的大小不足以容纳所有的元素。在这种情况下，可以考虑动态调整哈希表的大小，以增加槽位的数量，从而减少冲突的概率。通常，当哈希表中的负载因子（即元素数量与槽位数量的比值）超过某个阈值时，就会触发重新哈希（Rehash）操作，创建一个更大的哈希表并重新插入所有元素。



### hashmap源码实现

以下是一个简单的C++代码示例，展示了如何实现一个基于哈希表的HashMap：

```cpp
#include <iostream>
#include <vector>
#include <list>

// 定义键值对数据结构
template<typename K, typename V>
struct KeyValuePair {
    K key;
    V value;

    KeyValuePair(const K& k, const V& v) : key(k), value(v) {}  //存储键值对
};

// 实现HashMap
template<typename K, typename V>
class HashMap {
private:
    std::vector<std::list<KeyValuePair<K, V>>> buckets;   //开链法，用数组存双向链表list，也叫桶吧
    size_t size;
    size_t capacity;

public:
    HashMap(size_t capacity) : size(0), capacity(capacity) {
        buckets.resize(capacity);
    }

    size_t hashFunction(const K& key) {
        // 哈希函数示例：简单取余
        return std::hash<K>{}(key) % capacity;  //实现哈希函数
    }

    void put(const K& key, const V& value) {
        size_t index = hashFunction(key);
        std::list<KeyValuePair<K, V>>& bucket = buckets[index];   //通过hash函数得到索引桶

        // 检查是否已存在相同的键
        for (auto& kv : bucket) {
            if (kv.key == key) {
                kv.value = value;
                return;
            }
        }

        // 创建新的键值对并插入到桶中
        bucket.push_back(KeyValuePair<K, V>(key, value));
        size++;
    }

    V get(const K& key) {
        size_t index = hashFunction(key);
        std::list<KeyValuePair<K, V>>& bucket = buckets[index];

        // 在桶中查找键
        for (auto& kv : bucket) {
            if (kv.key == key) {
                return kv.value;
            }
        }

        // 键不存在
        throw std::out_of_range("Key not found");
    }

    void remove(const K& key) {
        size_t index = hashFunction(key);
        std::list<KeyValuePair<K, V>>& bucket = buckets[index];

        // 在桶中查找并删除键值对，直接用list的erase函数
        for (auto it = bucket.begin(); it != bucket.end(); ++it) {
            if (it->key == key) {
                bucket.erase(it);
                size--;  //注意减一
                
                return;
            }
        }

        // 键不存在
        throw std::out_of_range("Key not found");
    }

    size_t getSize() const {
        return size;
    }
};

int main() {
    HashMap<std::string, int> hashMap(10);
    hashMap.put("apple", 5);
    hashMap.put("banana", 8);
    hashMap.put("orange", 12);

    std::cout << "Size of HashMap: " << hashMap.getSize() << std::endl;

    int value = hashMap.get("banana");
    std::cout << "Value of 'banana': " << value << std::endl;

    hashMap.remove("apple");

    try {
        int nonExistentValue = hashMap.get("apple");
        std::cout << "Value of 'apple': " << nonExistentValue << std::endl;
    }
    catch (const std::out_of_range& e) {
        std::cout << "Key 'apple' not found" << std::endl;
    }

    return 0;
}
```

这个示例实现了一个简单的基于哈希表的HashMap，使用了C++的STL容器`std::vector`和`std::list`来存储桶和键值对。它提供了`put`、`get`和`remove`方法来插入、获取和删除键值对，并且支持哈希函数来计算键的哈希值以确定索引位置。在`main`函数中，演示了如何使用HashMap进行操作，并输出结果。请注意，这只是一个简单的示例，实际的HashMap实现可能需要更多的功能和优化。

## stack

后进先出，可以用数组实现



## queue

先进先出，可以用围成一个环，用数组实现，维护front 和 back两个指针，具体见数据结构

常用操作

```c++
q.empty()             如果队列为空返回true，否则返回false
q.size()                返回队列中元素的个数
q.pop()                 删除队列首元素但不返回其值
q.front()               返回队首元素的值，但不删除该元素
q.push()                在队尾压入新元素
q.back()                返回队列尾元素的值，但不删除该元素
q.swap()     			交换两个队列的元素

```



## priority_queue

优先队列 https://blog.csdn.net/qq_46514118/article/details/120698465

1.区别：它和queue不同的就在于我们可以**自定义其中数据的优先级**, 让优先级高的排在队列前面,优先出队，并且没有front()，back()函数，只用top()访问队首元素
2.相同：优先队列**具有队列的所有特性**，包括基本操作，只是在这基础上添加了内部的一个排序，它本质是一个**堆**实现的，都包含同一个头文件。**默认是大顶堆(优先级高的在前)**

堆又可称之为**完全二叉堆**。这是一个逻辑上基于**完全二叉树**、物理上一般基于线性数据结构（如数组、向量、链表等）的一种数据结构。



```c++
//三个参数
Type 就是数据类型，
Container 就是容器类型（Container必须是用数组实现的容器，比如vector,deque等等，但不能用 list。STL里面默认用的是vector），
Functional 就是比较的方式
    
//降序队列(默认大根堆)
priority_queue <int,vector<int>,less<int> >q;

//升序队列(小根堆)
priority_queue <int,vector<int>,greater<int> > q;

priority_queue< pair<int, int> > a;
```

改变优先级的两种方式：重写<运算符 和 重写仿函数 （重载>运算符会编译报错）



## deque

双端队列 没有队列和栈这样的限制级，它允许**两端进行入队和出队操作**，也就是说元素可以从队头出队和入队，也可以从队尾出队和入队。

