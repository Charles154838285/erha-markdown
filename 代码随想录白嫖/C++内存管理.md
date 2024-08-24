# C++ 内存管理



## 内存分配

---



### 内存分配的每一层面

C++内存分配的深度一般是有五个层面，它们分别是
- C++ application 常见：vector等容器
- C++ allocator 常见：vector自带的allocator分配器
- C++ primitives 常见：new delete
- CRT 常见：malloc free
- OS API 常见：heapalloc等等

	一般情况下，为了保证可移植性质，一般最底层就到了malloc free，如果调用OS API就基本上丧失了可移植性质
	每一个层面的对应的基本上都会依次向下调用，然后进行内存的分配

---



### 简单调用几个内存的分配

```
//对于malloc
void *p=malloc(512)//分配512字节
//对于allocator
int *p=allocator<int>().allocate(3)//分配三个int
//对于new，他的底层调用的依然是malloc
int *p=new int();
```
---



### new与delete的底层实现

对于new
- 首先：调用malloc，分配内存，
- 然后将指针转型
- 然后由编译器调用构造函数，进行构造

对于delete

- 首先，调用析构函数，析构所有的元素
- 然后调用free，释放内存

对于new【】

- 除了对于new的基本操作之外，还有一个就是他还要使用一个cookie记录分配的次数（调用了几次malloc）
- 如果操作对象是class，那么一定是调用了三次默认构造函数，原因是new一个对象数组的时候它是没有机会给你设置初始值的

对于delete【】 

- 在析构函数之前确定cookie的信息，之后重复那两部
- 对于加不加【】，对于非class with pointer对象没有影响

---



### placement new

用法： char *p=new char（sizeof（class））

​			class *x=new（p）class（1，2）

placement new干的事情

- placement new允许我们将object翻入已经分配的allocated memory之中
- 没有所谓placement delete，因为这个placement new就没有分配内存

对于placement new

- 首先同样调用operator new，但是与new调用的operator  new不同，他只是传入一个参数。operator new传入两个参数并且将（）中的参数也一并传入

---



### 重载operator new/delelete/new[]/delete[]

首先我们必须认识到，重载operator实际上就是重载new里面第一步调用operator new的动作，也就是自己来分配内存，重载operator delete也是如此。至于后面两步的转型和呼叫构造函数还是通过new和编译器完成

****



在重载operator new/new【】时候，我们只用传入一个参数，就是分配内存的大小，在重载operator delete/delete【】时候，也是传入一个参数，就是对应的指针

```
inline void *operator new(size_t a);
inline void operator delete(void *ptr);
```

一般来讲我们都吃重载class::operator xxx，当然全局的：：operator也可以重载，但是影响很大

****

在重载 placement new 的时候，其第一个参数必须是size-t，这个size-t表示对象的大小，其他的参数理论上说可以放其他的，但是最好还是按照标准库放一个指针

```
foo *pf=new(size_t a,typename ...arg);
```

当我们在重载过placement new，一般情况下，我们不需要写出placement delete，即使写出，在程序运行正常的情况下也不会调用，但是当程序会抛出异常的时候。程序就有可能调用你写出来的所谓placement delete(),它可以帮助销毁未成功销毁的对象

---



### new handler

我们在分配内存的时候。分配内存可能会有失败的情况，在失败的时候往往会抛出异常。对于内存，我们在抛出失败之前。可以先调用一个自己制作的handler，这个handler可能能帮助正确的分配内存或者终止进程

```
typedef void(*new handler)();//定义一个handler
new_handler set_new_handler(new_handler p)throw();//设定一个handler
```

new handler的两种选择

- 让更多memory 可以被使用
- 抛出异常（abort（）或exit（））

```
//例子
void handlerx(){//类似这样的就可以
	cerr<<"no";
	abort();
}
void main(){
	set_new_handler(handler);
	..
}
```

---



### =delete,=default与operator new/delete的关系

operator new/delete是可以被设定为=delete，当=delete时候。其不允许被分配

---



## C++ allocator

### GNU allocator总述（pool_allocator)

allocator，作为一个STL的分配器，其底层是由new/delete进行实现的

**在GNU的编译器里面**

- 对于比较大的或者一般的allocator，我们一般调用：：operator new/delete allocator和deallocate进行
- 对于较小块内存的分配，在GNU2.9里面有一个比较特殊的设计。

**GNU pool allocator分配器简述**

核心思想

- 首先分配一个数组，里面含有16个指针，这16个指针在未来会再一次指向对应的内存空间，每一个指针都会负责比前一个指针多8bytes的数据
- 每一个指针再一次分配的时候。一般会分配40个和对象+一个上一个分配空间/4的一样大的空间，每一个空间里面包括一个嵌入式指针，指向下一块空间，其中20个作为现在的用池，用来进行对象的安放。剩下的就作为后面的战备池每一次分配出去一个，指针就往下移动一个
- 当需要分配新的对象的时候
  - 如果战备池子有充足的空间，就从战备池子取一定的空间，具体看战备池能取出多少就取出多少，最多不超过20个
  - 如果战备池子没有多余的空间，就重新分配一块内存，同时如果战备池的空间不足以分配当前一个对象，就将这个空间交还给对应的链表
  - 如果在无法分配内存的情况下，就会从现有的，比他大一级的池子（右边第一个池子）里面分配内存，只是裁剪出来一块挂到对应的链表中。当右边没有的时候就会分配失败

优点与缺点

- 优点
  - 减少了cookie，使得软件拥有了更大的内存分配空间
- 缺点
  - 无法释放已经分配的内存，存在着不少的内存浪费的情况



### loki allocator

loki allocator是分配器里面一个比较特殊的设计

**设计框架**

```
底层：
   底层作为alloc直接管理的一个基层，他是中层的一个嵌套类
class chunk{
	unsigned char* pData;  --指针，指向分配内存的头部
	unsigned char firstAvailableBlock;--记录下一个可以供给分配的内存
	unsigned char blocks;--记录总的可分配格子的大小
};

中层：
   中层作为alloc管理基层的工具，以一个小型的vector为基础。担任分配和释放的操作
class FixedAlloctor{
	vector<chuck> chucks; --管理底层的vector
	chuck* allocChucks(); --分配chuck
	chuck* deallocChucks(); --销毁chuck
};

高层：
	高层继承中层，作为主要的对外接口，去分配和释放内存，他的客户是STL
class SmallObjAllocator{
	vector<FixedAllocator> pool; --管理中层的vector
	FixedAllocator* pLastAlloc;  --指向最后一个可以分配的alloc
	FixedAllocator* pLastDealloc;--指向最后一个销毁的alloc
}
```

**使用方法**

1.首先，创建一个smallobjallocator，然后制定分配的内存和大小

- 一般情况下，一次性默认要4096字节空间，超过256字节就不使用loki了

2.每一次进行分配内存的时候指定内存大小，然后进行分配

3.在释放的时候需要其制定的内存大小和指针就可以进行释放了

**工作逻辑**

chuck

1.chuck首先按照上层的要求进行初始化。调用init。初始化大小，初始化内存空间及其对应的数字。

2.然后索要对应的内存，进行分配，如果分配的内存大小和之前相同，就从之前的取

3.在alloc时候，

- chuck首先找到下一个可以分配的内存，将其分配出去，
- 同时将内存写好中的数字记录下来，这个数字就是下一个可以分配的内存空间

4.在dealloc的时候，

- 首先利用一个从中间向两边找的准则，找到对应的位置（中间层，剩下的是底层）

- chuck然后找到将进来的指针进行强制类型转换。
- 然后将原先的下一个内存的数字给他，
- 然后通过指针去找他和之前有几个格子的距离，为firstAvailblocks赋予新的值

其他两层按照vector的方式去进行操作和管理



### new_allocator与malloc_allocator

new_allocator与malloc-allocator唯一的区别就是

- new allocator可以重载operator：：new,可以实现自由的构建malloc不行，他直接调用malloc去使用
- 重载operator::new的一个功能就是可以在一定程度上接管alloc所作的工作，去灵活的管理



### array_allocator

array_allocator是一个数组类型的分配器，他可以分配固定的内存

在main程序运行之前，底层函数就已经通过固定的程序让array_allocator这个其依赖的基本数据结构array可用了



### debug_allocator

这个是一个allocator的适配器，类似于stack，他可以帮助程序员调试allocator的内部操作，没什么用



### bitmap_allocator

bitmap_allocator结构分为两层

记录层：这层主要用来记录那些blocks的内存被分配掉了,private/public关系省略了

```
class bitmap{
	unsigned int useCount；--记录使用的blocks数目
	unsigned int superBlockSize; --记录整个记录层和管理层一共占用的空间总数
	char bitmapGuide[bitmapSize] = ‘F’; --用16个字符记录使用的数目,bitmapSize是一个可以变的数目，后续根据是否需要扩容决定
}；
```

管理层：这层主要用来管理对应的分配出来的blocks，做实质的alloc/dealloc操作

```
template <typename T>
class block{
	T *p; --有一个指针
	block(int number = 4096){
		p=new T(number/sizeof(T()))--为T分配一块内存，这块内存按照传入的字节大小去除一个T对象本身的大小（注意这段代码是伪代码，本身是错误的）
	}
}；

tempate <typename T>
class superBlock{
	block<T> blocks[64] --这块默认是64个blocks，64个blocks组成一个superBlocks
}
//在有以上两个之后，然后
superBlock<T> blockss;  --创建内存
mini_vector<T*> vector(2,nullptr);  --创建一个minivector，这个是用来储存blockss的头尾指针的
vector[0]=&(blockss[0]);
vector[1]=&(blockss[63]);
```

**工作原理**

对于少量内存分配

- 选择一格，把他们分配出去
- 在记录层记录blocks数目的记录加一（已经使用一格）
- 更改bitmap
  - 注意，bitmap的读条方式是，bitmap地图从左往右，对应内存池的从右边向左边的格子，利用二进制表明其是否被占用

对于大量内存分配

- 如果在一个bitmap不够的情况下就变成原来的两倍，也就是bitmap大小为32，有128个blocks
- 后面每一次分配都会加量一次（原先x2），每一次全回收都会减量一次（原先/2）

对于回收

- 和malloc一样，他会有一个“垃圾寄存处”，当不需要的时候，会重新分配一个minivector来寄存这些已经全回收的内存，如果需要的话也是优先看手上有没有
- 如果有，用，当有64个组以上的minivector的时候，进行回收



## malloc

**总述**

- malloc属于C 的部分。
- malloc主要通过它的内存管理器：SBH进行管理

**malloc获得内存的 过程**

- 首先，malloc获得内存是从CRT里面去获取内存的
- CRT通过调用操作系统的底层函数，去分配内存
- 然后把或得到的内存切块，作为一个一个header进行管理，一共16块

**malloc管理内存的结构**

- malloc管理这块内存，对于每一个header，都通过两个指针进行管理
- 其中一个指针指向通过virtual alloc分配的内存，共8*32个page
- 另外一个指向一个bitmap，通过这个bitmap进行管理。
- 每一个bitmap里面的bit又对应着32个group
- 每一个group下面对应一个bitmap和64组list。
- 最后一个list连接着八个page，用来管理这些page
- page之间通过指针相互连接。

**malloc内存的分配**

- 如果之前没有分配过内存，且之前没有已经分配好的malloc块，则通过规划出来一个上面管理的内存，然后从page1开始分配
- 分配完成之后将剩余的内存挂在group的链表上，使用嵌入式指针，并且标记分配加一
- 当再次分配内存的时候，如果链表上还有就是用链表上的再一次进行分配，如果不够就是用新的内存（page）
- 分配之后的内存有两个标记他们是被分配的。然后再分配的内存里面才会放入cookie。和debug head

**malloc内存的释放**

- 一般来讲，首先对于需要释放的内存。随机化/清空所在的内存块的数据
- 然后放入两个嵌入式指针
- 将他们挂在链表上
- 将分配的次数减一

**malloc内存的合并**

- 对于一块已经不需要的数据，他会根据两个之前header查看。如果向上对应的也是一个最后一个bit为0的，那么就和上面进行合并，如果不是就不合并
- 如果上面的不行。那么就通过加上之前记录的向下的内存块，看看下面是否有bit为0的，如果是就合并。
- 如果所有page都空（计数器为0）那么就可以将这块内存交还给操作系统，反之则不

对于为空的malloc，也不会立刻交换，而是会暂时保存待用

