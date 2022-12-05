## 第二章 简单动态字符串

Redis没有直接使用C语言传统的字符串表示(以空字符结尾的字符数组,以下简称C字符串),而是自己构建了一种名为简单动态字符串(simple dynamic string,SDS)的抽象类型,并将SDS用作Redis的默认字符串表示.

在Redis里面,C字符串只会作为字符串字面量(string literal)用在一些无须对字符串值进行修改的地方,比如打印日志:

```c
redisLog(REDIS_WARNING,"Redis is now ready to exit,bye bye...")
```

### 2.1 SDS的定义

```c
struct  sdshdr{  
  //  记录buf数组中已使用字节的数量
  //  等于SDS所保存字符串的长度  
  int  len;  
 
  //  记录buf数组中未使用字节的数量  
  int  free;  
  
  //  字节数组,用于保存字符串  
  char buf[];
};
```

![ScreenShot20221202at095555.png](../assets/database/redis/redis-sds.png)

SDS遵循C字符串以空字符结尾的惯例,保存空字符的1字节空间不计算在SDS的len属性里面,并且为空字符分配额外的1字节空间,以及添加空字符到字符串末尾等操作,都是由SDS函数自动完成的,所以这个空字符对于SDS的使用者来说是完全透明的.

遵循空字符结尾这一惯例的好处是,SDS可以直接重用一部分C字符串函数库里面的函数.

```c
printf("%s", s->buf)
```

### 2.2 SDS于C字符的区别

#### 2.2.1 常数复杂度获取字符串长度

因为C字符串并不记录⾃⾝的⻓度信息,所以为了获取⼀个C字符串的⻓度,程序必须遍历整个字符串,对遇到的每个字符进⾏计数,直到遇到代表字符串结尾的空字符为⽌,这个操作的复杂度为O(N),而SDS记录了字符串长度,复杂度为O(1).

#### 2.2.2 杜绝缓冲区溢出

因为C字符串不记录自身的长度,所以`strcat`假定用户在执行这个函数时,已经为dest分配了足够多的内存,可以容纳src字符串中的所有内容,而一旦这个假定不成立时,就会产生缓冲区溢出.

与C字符串不同,SDS的空间分配策略完全杜绝了发生缓冲区溢出的可能性: 当SDS API需要对SDS进行修改时,API会先检查SDS的空间是否满足修改所需的要求,如果不满足的话,API会自动将SDS的空间扩展至执行修改所需的大小,然后才执行实际的修改操作,所以使用SDS既不需要手动修改SDS的空间大小,也不会出现前面所说的缓冲区溢出问题.

#### 2.2.3 减少修改字符串时带来的内存重分配次数

在SDS中,buf数组的长度不一定就是字符数量加一,数组里面可以包含未使用的字节,而这些字节的数量就由SDS的free属性记录.

- 空间预分配: 修改后小于1M,则 `len == free`, 大于等于1M, 则 `free = 1M`
- 惰性空间释放: trim 之后, 不会立即释放空间

#### 2.2.4 二进制安全

SDS使用len属性的值,而不是空字符来判断字符串是否结束

#### 2.2.5 兼容部分C字符串函数

SDS保存的数据,末尾依旧有空字符(`\0`)

#### 2.2.6 总结


| c 字符串                             | SDS                                  |
| ------------------------------------ | ------------------------------------ |
| 获取字符串长度的复杂度为O(n)         | 获取字符串长度的复杂度为O(1)         |
| API是不安全的,可能会造成缓冲区       | API是安全的,不会造成缓冲区溢出       |
| 修改字符串n次，必须进行n次内存重分配 | 修改字符串n次，最多进行n次内存重分配 |
| 只能保存文本                         | 可以保存文本或二进制数据             |
| 可以使用所有<string.h>库中的函数     | 可以使用一部分<string.h>库中的函数   |

### 2.3 SDP API

### 2.4 重点回顾

#### 2.5 参考资料

## 第3章 链表

### 3.1 链表和链表节点的实现

```c
typedef  struct  listNode    {
    // 前置节点
    struct  listNode  *prev;
  
    // 后置节点
    struct  listNode  *next;
  
    // 节点的值
    void  *value;
} listNode;

typedef  struct  list  {
    //  表头节点
    listNode * head;
  
    //  表尾节点
    listNode * tail;
  
    //  链表所包含的节点数量
    unsigned long len;
  
    // 节点值复制函数
    void *(*dup)(void *ptr);
  
    //  节点值释放函数
    void (*free)(void *ptr);
  
    //  节点值对比函数
    int (*match)(void *ptr,void *key);
} list;

```

- 双端: 链表节点带有prev和next指针,获取某个节点的前置节点和后置节点的复杂度都是O(1).
- 无环: 表头节点的prev指针和表尾节点的next指针都指向NULL,对链表的访问以NULL为终点.
- 带表头指针和表尾指针: 通过list结构的head指针和tail指针,程序获取链表的表头节点和表尾节点的复杂度为O(1).
- 带链表长度计数器: 程序使用list结构的len属性来对list持有的链表·双端:链表节点带有prev和next指针,获取某个节点的前置节点和后置节点的复杂度都是O(1).
- 多态:链表节点使用void*指针来保存节点值,并且可以通过list结构的dup、free、match三个属性为节点值设置类型特定函数,所以链表可以用于保存各种不同类型的值.

### 3.2　链表和链表节点的API

### 3.3　重点回顾

## 第4章 字典

字典,又称为符号表`(symbol table)`、关联数组`(associative array)`或映射`(map)`,是一种用于保存键值对`(key-value pair)`的抽象数据结构.

### 4.1　字典的实现

Redis的字典使用哈希表作为底层实现,一个哈希表里面可以有多个哈希表节点,而每个哈希表节点就保存了字典中的一个键值对.

#### 4.1.1 哈希表

```c
// dict.h/dictht
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
  
    // 哈希表大小
    unsigned long size;
  
    //哈希表大小掩码,用于计算索引值
    //总是等于size-1
    unsigned long sizemask;
  
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;
```

#### 4.1.2　哈希表节点

```c
typedef struct dictEntry {
    // 键
    void *key;
  
    // 值
    union{
        void *val;
        uint64_tu64;
        int64_ts64;
    } v;
  
    // 指向下个哈希表节点,形成链表
    // 将多个哈希值相同的键值对连接在一次,以此来解决键冲突(collision)的问题.
    struct dictEntry *next;
} dictEntry;
```

#### 4.1.3　字典

```c
typedef struct dict {
    // 类型特定函数
    dictType *type;
  
    // 私有数据
    void *privdata;
  
    // 哈希表
    dictht ht[2];
  
    // rehash 索引
    // 当rehash不在进行时,值为-1
    in rehashidx; /* rehashing not in progress if rehashidx == -1 */
} dict;

typedef struct dictType {
    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);
  
    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);
  
    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);
  
    // 对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
  
    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);
  
    // 销毁值的函数
    void (void (*valDestructor)(void *privdata, void *obj);
} dictType;
```

![dict](../assets/database/redis/dict.png)

### 4.2　哈希算法

```c
#使用字典设置的哈希函数,计算键key的哈希值
hash = dict->type->hashFunction(key);

#使用哈希表的sizemask属性和哈希值,计算出索引值
#根据情况不同,ht[x]可以是ht[0]或者ht[1]

index = hash & dict->ht[x].sizemask;
```

当字典被用作数据库的底层实现,或者哈希键的底层实现时,Redis使用MurmurHash2算法来计算键的哈希值.

### 4.3 解决键冲突

当有两个或以上数量的键被分配到了哈希表数组的同一个索引上面时,我们称这些键发生了冲突`(collision)`.

Redis的哈希表使用链地址法`(separate chaining)`来解决键冲突,每个哈希表节点都有一个next指针, 多个哈希表节点可以用next指针构成一个单向链表,被分配到同一个索引上的多个节点可以用这个单向链表连接起来,这就解决了键冲突的问题.

因为`dictEntry`节点组成的链表没有指向链表表尾的指针,所以为了速度考虑,程序总是将新节点添加到链表的表头位置(复杂度为O(1)),排在其他已有节点的前面.

### 4.4 rehash

随着操作的不断执行,哈希表保存的键值对会逐渐地增多或者减少,为了让哈希表的负载因子`(load factor)`维持在一个合理的范围之内,当哈希表保存的键值对数量太多或者太少时,程序需要对哈希表的大小进行相应的扩展或者收缩.

1) 为字典的ht[1]哈希表分配空间,这个哈希表的空间大小取决于要执行的操作,以及ht[0]当前包含的键值对数量(也即是ht[0].used属性的值):

* 如果执行的是扩展操作,那么ht[1]的大小为第一个大于等于ht[0].used*2的2 n(2的n次方幂)；
* 如果执行的是收缩操作,那么ht[1]的大小为第一个大于等于ht[0].used的2 n.

2) 将保存在ht[0]中的所有键值对rehash到ht[1]上面:rehash指的是重新计算键的哈希值和索引值,然后将键值对放置到ht[1]哈希表的指定位置上.
3) 当ht[0]包含的所有键值对都迁移到了ht[1]之后(ht[0]变为空表),释放ht[0],将ht[1]设置为ht[0],并在ht[1]新创建一个空白哈希表,为下一次rehash做准备.

当以下条件中的任意一个被满足时,程序会自动开始对哈希表执行扩展操作:

1) 服务器目前没有在执行`BGSAVE`命令或者`BGREWRITEAOF`命令,并且哈希表的负载因子大于等于`1`.
2) 服务器目前正在执行`BGSAVE`命令或者`BGREWRITEAOF`命令,并且哈希表的负载因子大于等于`5`.

哈希表执行收缩操作:

1) 负载因子小于`0.1`.

```c
# 负载因子= 哈希表已保存节点数量/哈希表大小
load_factor = ht[0].used / ht[0].size
```

根据`BGSAVE`命令或`BGREWRITEAOF`命令是否正在执行,服务器执行扩展操作所需的负载因子并不相同.
这是因为在执行`BGSAVE`命令或`BGREWRITEAOF`命令的过程中,Redis需要创建当前服务器进程的子进程,而大多数操作系统都采用写时复制`(copy-on-write)`技术来优化子进程的使用效率,
所以在子进程存在期间,服务器会提高执行扩展操作所需的负载因子,从而尽可能地避免在子进程存在期间.

### 4.5　渐进式rehash

为了避免`rehash`对服务器性能造成影响,服务器不是一次性将`ht[0]`里面的所有键值对全部`rehash`到`ht[1]`,而是分多次、渐进式地将`ht[0]`里面的键值对慢慢地`rehash`到`ht[1]`.
以下是哈希表渐进式`rehash`的详细步骤:
1)为`ht[1]`分配空间,让字典同时持有`ht[0]`和`ht[1]`两个哈希表.
2)在字典中维持一个索引计数器变量`rehashidx`,并将它的值设置为0,表示rehash工作正式开始.
3)在`rehash`进行期间,每次对字典执行添加、删除、查找或者更新操作时,程序除了执行指定的操作以外,还会顺带将`ht[0]`哈希表在`rehashidx`索引上的所有键值对`rehash`到`ht[1]`,当`rehash`工作完成之后,程序将`rehashidx`属性的值增一.
4)随着字典操作的不断执行,最终在某个时间点上,`ht[0]`的所有键值对都会被`rehash`至`ht[1]`,这时程序将`rehashidx`属性的值设

### 4.6　字典API

### 4.7　重点回顾