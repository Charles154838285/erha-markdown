MoreEffectiveCpp_快学笔记



# More effective C++

# Basics 基础议题

## 1 区别Pointer和Reference

条款1：区别Pointer和Reference

1. Pointer：可能会不指向任何对象、可以换绑
2. Refer：一定绑定了一个对象、不可以换绑、opertor[]等操作符通常返回引用

## 2 最好使用C++转型操作符

不要用(int) expre等等形式的，要使用static_cast、dynamic_cast、const_cast、reinterpret_cast（强转）

1. static_cast：类型转换(主要是前两种)
2. const_cast：去除变量底层的常量性
3. dynamic_cast：涉及继承体系的指针和引用的转换
4. reinterpret_cast：常用于函数指针的转换但不保证转换后能转换回来p15

## 3 不用以多态的方法处理数组

1. 传入base数组或传入subbase数组会导致 

   关于指针的计算表达式

    结果出错

   1. sun在工程形态上面用的就很少。

```c++
void print(const Base array[] ) {  // const SubBase array[]
    //     *(array + i);   // ptr + sizeOf(Base) , sizeOf(SubBase) 会不一样
}
```

## 4 非必要不要设定default constructor(ctor)

| **好处**                                               | **坏处**                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| 每个实例化出的对象都是用户要的对象                     | 想用Base的数组。但是因为无法构造**Base base[]**，必须得用`Base* array[]`，同时不用了还得手动析构array对应的对象因为可能会调用默认ctor，有些模板函数如果依赖默认的构造函数就不能用了。p22 |
| **大话孙的实践表明**：大多时候还是要默认的构造函数的。 |                                                              |

# Operators 操作符

## 5 两种可能的“类型转换的”函数保持警觉

1. 单变量(或其他变量有默认参数）构造函数ctor：可能触发不必要的隐式转换，所以尽量用explict修饰
2. **隐式类型转换符最好只有bool类型**，其他类型的用别名函数代替（asDouble）

```c++
struct Rational{
    Rational(int number, int denominator = 1);
    // 推荐：exlicit Rational(int number， int denominator = 1);
    operator double() const;
    // 推荐： opeartor asDouble() const;
};

Rational r(1, 2); // 1/2
// 没有定义 opeartor<<，为了让<<调用成功而默认执行了 double()
cout << r << endl;   // r->double-> 0.5;
```



## 6 increment和decrement的两种类型，且尽量用前置来定义后置

1. 前置返回引用。                                                                     UPInt& operator++(); 
2. 后装返回常量对象，从而避免使得++++运算符合法化。  const UPInt operator--(int); 



## 7 不用重载&&、||、逗号运算符，会让用户误以为重载的运算符还保持短路语义



## 8 不同意义的new和delete操作符

1. new operator通常包含**分配内存+构造对象**的语义，调用operator new() 实现
   1. 特别的一种new operator，**定位new实现**【new (targetNeedConstructPtr) Base(ctorParams)】，调用特殊的operator
2. delete operator操作符也是类似的，有类似的opeator delete来实现
3. 总结：不能改变new op和delete op的语义，但可以改变其构造的细节。



todo疑问：某种不是new出来的ptr，如sharedMemory，不能用delete和析构释放



# Exceptions 异常处理

## 9 利用 destructors 避免泄露资源

用对象来管理资源的构造、析构。避免程序在资源声明和释放之间发生异常时，无法释放资源。

1. 也可用智能指针来管理类中的指针资源

## 10 在constructors 内阻止资源泄露（resource leak)

用函数**try语句块来确保构造函数**不抛出异常、**用智能指针来管理类中的指针资源避免手动释放。**

> TITLE：在constructors内阻止资源泄露（resource leak)

1. **注意没有构造完成的对象，是无法被析构的（就是不会调用析构函数）**

```c++
template<typename T>
Blob<T>::Blob(initializer_list<T> il) try: data(make_shared<vector<T>>(il) { 
    // 函数体
} catch(const std::bad_alloc &e) { handle_out_of_memory(e); }
```



## **11 禁止exceptions流出destructors之外**

> 方法是**在析构函数里面catch所有异常**

1. 这样可以避免terminate()函数在栈暂开过程中被调用，就是避免程序中断
2. 这样可以保证析构函数析构完毕

## 12 抛出一个exceptions和**传递一个参数或调用一个虚函数之间**的差异

函数接收实参或对象调用虚函数 和 将对象抛出成为一个exception时的三种不同：

1. **当对象被当成exception throw时，一定会被复制且以静态类型为本**
   1. throw Obj 和 throw 的区别：前者抛出Obj的副本，后者抛出其原本。
   2. 捕获异常时`catch(Base& ex)`，用形参`non-const reference`可以捕获所有异常（包括`const BaseObj`)。  但是注意，普通函数`func(Base& pram)` 的形参`param`是不能接受`底层const`的`BaseObj`实参的。p65页
2. **函数接收实参时允许的类型转换挺多的，exception只允许两种类型转换：**
   1. `catch(runtime_error*) `和可以捕获继承架构中的类型转换，比如可以捕获overflow_error类型的异常。p67
   2. catch(const void*)可以捕获任何指针类型的exception。p67
3. **catch子句是按照出现顺序匹配异常的，而函数是按照形参实参匹配度匹配函数的**

```C%2B%2B
catch(Base& ex) {
    // ...
    throw ex; // 如果ex是Drived，抛出的是副本（ex会被复制一次），且类型是其静态类型
    throw;    // 如果ex是Drived，抛出的是原本（没有复制），且类型是Drived   p64
}
```

https://www.runoob.com/cplusplus/cpp-exceptions-handling.html

![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2VkZjkyZWZkYTFiZTNkMzk3YWNlZWFlODI4NTNiYjVfN0tJRjUyMHM1YnNFZGowcVZVakxHR1RJSU51V0lLcE9fVG9rZW46Ym94Y25KT3pORGdwUzR5a3p2N1JFenJNZ0lkXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)



## 14 请以`reference`捕捉异常，即`catch(Base& ex)`

1. by reference捕捉异常的好处有3：①throw对象不会被切割、②标准库里exception 通常都throw对象（我们我们用by pointer类型不匹配），而refernce可以绑定对象、③约束了exception obj复制的次数（就throw时复制一次）
2. by pointer缺点：不符合上述②，标准库里都throw对象
3. by value缺点：不符合上述①，throw出的对象会被切割，无法调用虚函数。



## 14明 智运用 `exception specifications`

谨慎使用。某函数在编译期间，检查出与 exception specification  不一致的异常抛出（bad_alloc)就会调用 unexpected函数

> **void func() throw(int);**

1. 不要将`templates`和`exception specifications` 混合使用。比如`templates`里面用到了==、&都可能抛出和`specifications`不同的异常
2. A函数调用B函数，B函数无异常声明，A最好也别有。不然B抛出了非预期的异常，A岂不是就乱套了。

```C%2B%2B
template<class T>
bool operator=(const T& lhs, const T& rhs) throw() {
    return &lhs == &rhs;
}
```

## 15 exception 会带来一些成本

- 使用了throw和try_catch就是会有一点点成本，但是相对程序崩溃来说这点成本不算什么。

```C%2B%2B
Status func() {}

{ success, faild, resourceNotFound， .. }
```





# Efficiency 效率

## 16 记住8-2法则

软件中的80%的时间浪费在了20%的代码上。



## 17 考虑使用`Lazy evaluation`

举了三个例子：

- 避免非必要的对象复制。

  - 比如string s2 = s1，但是后续的操作op1_s2, op2_s2，等等都是不改变s2的，所以只要让s2和s1共享即可，当真正要改变s2时才拷贝s1。

- 区分读和写操作，只有在写的时候才坑需要额外的操作。

  （请跳过，言之无物）

  - 对于一个refer但是此处法无法判定是否能区分，如果能够在operator[]里面区分读和写，就可以判定需不需要做额外的操作，p87页。s[3]作为左边的值是写，s[3]作为右边的值是读。
  - 请见Ex30，里面用CharProxy实现了读和写的区分，不过感觉没什么必要。

- Lazy 读取对象。比如在数据库中一个BigClass有许多mem，只有需要其中一个mem时，才从数据库里读出来，不然Bigclass对象太大了取出来成本太高。

- lazy 表达式计算。



## 18 分期摊还预期的计算成本（over-eager evaluation 超前计算）

**举例子：**

- 对一个list，如果想知道聚合结果。如果用户调用的频繁，我们应当超前计算min,max,avg并缓存以平均成本；调用的不平凡，就不必超前计算。
- 用本地Cache，避免对数据库额外访问。



## 19 了解临时对象的来源

临时对象是：只要你产生一个`non-heap`对象而没有重命名，比如`func(const string& tmp);  func("helloTmpObj")` 这里**就会产生临时对象。通常产生临时对象的时机：**

1. 函数调用时发生的隐式类型转换，注意只有`reference-to-const`形参才能产生临时对象；否则产生的临时变量被使用了。**举例：char[]传递给func(string& tmp)。**
2. 函数返回对象时。
   1. 返回可以用类似复合语句(+=, -=，...)来优化返回时临时对象的个数。
   2. 注意函数返回引用：是不产生临时对象的。

```C%2B%2B
const Number operator+(const Number& lhs, const Number& rhs) {
    return Obj(lhs)+=rhs;
}
```



## 20 协助完成返回值优化 `return value optimization`

步骤如下：

1. 如果可以用类似Ctor arguments的方式返回对象，编译器可帮你消除临时对象+返回值对象的拷贝
2. 如果还可以将函数inline，编译器还消除了函数调用的成本

```C%2B%2B
inline const Rational operator*(const Rational& lhs, const Rational& rhs) {
    return Rational(lhs.numberator() * rhs.number(), ...); 
    // 这就是ctor arguments，直接一个return 没有临时对象的成本，就能够被编译器优化！
    
    // 总共两个成本：
    //  1. 临时对象的成本：Rational(lhs.numberator() * rhs.number(), ...); 
    //  2. ByValue返回值的成本：因为你const Rational& 在这里无法办到
}

Rational c = a * b;  // 调用这个时能够被忽略！！
```



## 21 重载技术避免隐式类型转换

好处：就是重载函数，可以避免隐式类型转换（产生临时对象）

坏处：重载后，函数太多了

```C%2B%2B
const UPInt operator+(const UPInt& lhs, const UPInt& rhs);
const UPInt operator+(int lhs, const UPInt& rhs); 
const UPInt operator+(const UPInt& lhs, int rhs);
```



## 22 操作符函数独身形式(op) 依赖复合形式（op=) 实现 

常见注意点：

1. 定义位置不同：复合形式（+=， -=，...) 是类的内部的声明，重载op常是（类外部的函数+有元声明）
2. 临时对象`Rational(lhs)`比命名对象`Rational result = lhs `效率高，编译器还能优化

```C%2B%2B
const Rational operator+(const Rational& lhs, const Rational& rhs) {
    return Rational(lhs) += rhs; // 这就是ctor argu..
}
```



## 23 考虑使用类似的、效率更高的库

出现性能问题后，比如stdio效率高于iostream，你可以换库



## 24 虚函数、多重继承、虚基类、RTTI的成本

虚函数有4个成本需要注意：

1. 每个拥有虚函数的类，需要一个vtbl（其大小由虚函数个数决定)，用于保存其可调用虚函数的指针。
2. 每个拥有虚函数的对象，需要一个vptr，用于指向vtbl
3. 虚函数应当不写inline关键字。避免这样写，有些编译器能帮忙优化，有些编译器不会。
   1. vtbl放在那里呢？一般而言：`class's vtbl`通常放在其第一个`non-inline, non-pure`虚函数定义的目标文件中，比如`class C1`的vtbl可能放在定义`C1::~C1`的目标文件中。或者编译器会让你选择，vtbl放在那里。
   2. 因为虚函数如果可以inline（相当于define在header中），会导致`vtbl`需要拷贝到每一个使用了此`class's vtbl`的目标文件中。p115
4. 每个class对应的vtbl，需要指向一个type_info对象，type_info对象用于保存其类型信息。



**其他注意：**

- class的vtble常见的位置，是在第一个【非inline，非纯虚函数】(non-inline, non-pure) 的定义文件中。

- 多重继承导致需要虚基类

  ，而虚基类会让隐藏指针vptr变多，占用空间。p119

  - A是BC的虚基类，D继承BC，所以对D来说，它的内存结果如下右图。BC要指向共同的A的pointer to virtual base部分，BCAD都有自己的vptr；

![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=YTlkMmMwMTU4Y2Q2MjhiZmVlNDc4MDI2NjZlNDZhZWFfSXRoVlNVM0NhRW8yRTFSeWVLVkVJTllmYXJ3aGFoSTdfVG9rZW46Ym94Y25NTHRSNzNMNlJYazFVYlZ6Mm4yU0FiXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=NTlkYzJhZDVmOWU5MjYxMjVmZmYzNmRlOGQ5ZDgwYzVfcVJlWWxqQjczWTNrdFBEU3V3OTlVUTFHVjBnb1UwV2ZfVG9rZW46Ym94Y25KaDhMYXZPeTJnY2hXNEVWUE5XOW9lXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)





# Techniques 技术 

## 25 将constructors和non-member funciton虚化

- ctor虚化，基类的

  虚ctor函数

  可以创建

  子类的对象

  ，并且返回值也是指针

  - 本质上是：vitual ctor调用自身的 ctor。p126

- **non-member函数虚化**：non-member根据对象类型去调用实际工作的虚函数即可。p128

```C%2B%2B
class Base {
    // virtual ctor
    virtual Base* clone() { return new Base(*this); }      
    virtual ostream& print(ostream& s) const = 0;
};

class SubBase: public Base {
    virtual Base* clone() overide { return new SubBase(*this); }      
    virtual ostream& print(ostream& s) const = 0;
};

// virtual non-member，虚化非成员函数。
inline ostream& operator<<(ostream& s, const Base& c) {
    return c.print(s);
}

// ------------------------
vector<Base*> vec;      
// vec可插入所有的：Base* base(subBase), base2(base)搜用
```



## 26 限制某个class所能产生的对象数量

- 单例

  的实现方法：私有的构造函数+public static getInstance()+ public dtor

  - 辨析p133

    ：类的静态成员class_static_member_var和 函数的静态局部变量 function_static_var的初始化时机：

    - 类的静态变量class_static_member_var：一定会被构造和析构，在bss区；但是不同的编译单元内的static变量初始化顺序不保证p134。
    - 静态局部编码 function_static_var：函数被调用才被构造；可能会也可能永远不会被初始化。

  - 单例的实现方法：必须是non-inline的，否则拷贝多个static就是多例了呀。p134

- 多例的实现方法：class_static_member统计个数，xxxxp135。

- **关于 private ctor 或者 private dtor 对于构造对象个数的影响：**某个类含有私有ctor，但它也可以构造出任意数量的对象。p137用伪构造函数，同理也可以有伪析构函数。但是private ctor或private dtor阻碍了类的继承p146，继承需要用到构造和析构呀。

```C%2B%2B
class Sigliton {
public:
    ~Sigliton() {};
    // 必须是非inline的实现，这里是为了方便写。
    static GetInstance() {       
        static Sigliton obj;
        return obj;
    };
private:
    Sigliton() {};
};

// 某个类含有私有ctor，但它也可以构造出任意数量的对象。p137
class FSA {
public:
    static FSA* makeFSA() { return new FSA();}
private:
    FSA() = default;
};
```



## 27 要求（或禁止）对象产生于heap中

- 要求对象产生在堆上？有办法的！！！

  - 方案1 p146：private/protected dtor + 伪析构函数( public destory() { delete this; }。
  - 因为私有化构造函数，用户如果不用new的方式显示声明，显示destroy，依赖变量离开作用域自动destroy就会因为无法调用析构函数而报错。

- 判断对象是否产生在堆上怎么办？

  **没有办法！！！但是可以判断ptr是否指向heap的对象有方法。**

  - 因为判定不了obj是否在heap上，所以不必判定p152

  - 但是

    判定指针是否指向heap的对象可以实现。

    - 错误方法：可以在全局空间自定义通过operator new()和 operator delete()实现，如果指针是指向heap对象，你delete它就很安全p153。但是做这个也没什么意义p153，因为你改了全局空间默认的operator new()难免不出岔子p153
    - 有效方法：p155，如果你想判定某个ptr是否指向的是heap上的对象，你可以定义一个abstract min HeapTracked基类，然后让你想判定的类继承它即可。[参考HeapTracked源码](https://segmentfault.com/a/1190000040890556)

- 禁止对象产生在heap上怎么办？

  和上面一样没有办法

  。

  - 做法：new opeartor调用private operator new()，能够禁止一个类的对象产生于堆；
    - **可以禁止单个类A产生于堆p157，也顺便禁止了其派生类产生于堆**
    - **但不能禁止包含A类成员的类B产生于堆。p158**
  - 总结：你不能完全禁止，一个类可以被用在三个地方p157（类实例化，被继承，**被其他类当成成员**），所以你怎么可能都禁止呢？但是如果一个类A做为B类的成员，则new operator又不会递归的调用A的private operator new()，所以还是禁止不了。

无法复制加载中的内容





## 28 智能指针 SmartPointers p159-182

cpp用智能指针来提供dump ptr的使用效果，如果有3个sharded_ptr指向一个obj，当三个sp都离开作用域时obj被delete；如果unique_ptr指向一个obj，当这个up离开作用域时obj被delete。



**旧auto_ptr的反常点和易错点：**

1. p163 auto_ptr被复制或被赋值时，它的对象拥有权就被剥夺了。

   因为auto_ptr的拷贝构造和赋值函数都的实现，和普通对象的拷贝构造和赋值函数的实现不一样。普通对象的copy fun和assgin fun不会改变传入参数的性质，而auto_ptr会先剥夺旧auto_ptr的对象拥有权，在转移到新对象上面。

   1. copy ctor：auto_ptr(auto_ptr& rhs);       // **rhs的对象拥有权被剥夺，新对象从而拥有这个权利**

2. p164 函数参数如果是auto_ptr则不能是by-value的方式，而应该用by-reference-to-const的方式。如果你违反规则，可会出现double free。从1中可以看出，如果旧auto_ptr对象权利被剥夺了，而你以为它还存在，不就GG

```C%2B%2B
void func(auto_ptr<TreeNode> p);
int main() {
    auto_ptr<TreeNode> ptn(new TreeNode());
    func(ptn);       // p发生拷贝构造，且p死亡时对象已经被delete
    // use ptn here     // double free 或者 ptn 为nullptr
}
```



**测试SmartPointers是否是null**

- 书中最后用operator!()实现

  （但我在<memory>库里面没有找到这个函数）

  - 千万别通过实现`dumpPtr* SmartPtr::operator()`实现，不然delete smartPtr岂不是也合法了，p173。
  - **if(sp) 可以判空， if(!sp) 可以判定不空**

- tips：单一自变量constructors可以作为类型转换操作符使用。p172

- tips：Cpp不允许执行

  两次用户定义的转换

  。p172。

  - eg：即如果TupeAccessors类，实现了`TupeAccessors::TupeAccessors(Tuple * a)`，又实现了`Tuple* DBPtr::operator() `隐式类型转换。那么你想让`DBPtr<Tuple>---> Tuple* ---> call func(TupeAccessors ta1)`是不成功的，因为要对DBPtr执行两次隐式类型转换。



**用SmartPtr<Base>的地方能用SmartPtr<SubClass>吗。**

- 一般可以用base *ptr的地方，就可以用subclass* subptr替换，但是呢SmartPtr不存在这样的性质，所以不建议这样使用。

```C%2B%2B
void print(base* ptr);           // 可以传入 subbase ptr的，这是多态。
```

- 那怕利用现有的模板函数隐式生成的实例化的函数，也不是一个好的选择。p179。



**SmartPtr<const Base>和SmartPtr<Base>之间可以互相转换吗。**

- 不建议这样使用，虽然理论上可行。p181



## 29 引用计数 Reference counting

为了两个目的：2.1简化记录对象的拥有者们的工作，新建对象要创建记录，删除要删除记录，转移对象所有权要更新记录；2.2如果多个对象（eg，String），在内存里拥有的是同一个实例(eg, HelloWorld")，那么是不是减少了内存使用。p183



引用技术怎么实现（基本原理）：

- 比如用户类`String`，**其内部用一个共享对象**`**RCStringValue(refCount, realCharArray)**`**来帮助实现**。
- 对于String的copy ctor，**拷贝的时候（创建新对象的时候）**大家还是可以共用一个`RCStringValue`的，所以拷贝时相当于是浅拷贝，而非深拷贝。
- 只有当String的**类似**`**non-const operator[]()**`**操作符调用时（真正的写操作发生了）**，在这个操作符里面可能发生对RCStringValue的改变，所以此时可能还有其他String在使用共享对象RCStringValue，所以此时必须创建新的RCStringValue（即深拷贝）。**（copy-on-write思想，如果用户****不会改变****共享的对象，所以就不用真的拷贝；如果用户要写共享对象了，那么必须有一份自己的共享对象），****p190-192。**

![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=M2ZhNzBkYzM3ZjI4YTcwYjkyMjQ5ZTAxOWUyYmU2MzlfcURXckEyU1FGNmg5QVlUVjNHc2VRUk9QUU9wMkQ5V2xfVG9rZW46Ym94Y25adWdWVERvTUd0VUx6UUlBdUNNN1doXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)

```C%2B%2B
class String {
private:
    struct StringValue {        // 被多个String共享的，共享对象。
        int refCount;
        char* data;
    };
    StringValue *data;
public:
    String();
    // ...
};
```





Tips补充：

- p201-line4，E11补充建议我们为所有内含指针的classes撰写copy constructor(及 assignment-operator)，不然编译器合成的copy ctor的行为不一定符合预期。比如创建StringValue对象时，编译器合成的copy ctor仅仅拷贝data指针，而不会创建新的data数组，从而还是浅拷贝不符合要求。
- **引用计数的****基类RCObject 和 引用计数的智能指针类RCPtr****，p194-p202页**
- **Refer-count用于改造现有的类，p202-p213页**



## 30 Proxy classes(替身类、代理类）

> 这个tips建议看书，细节繁多。



例子1：实现二维数组。

- int* data[dim1][dim2]，dim1和dim2不能是变量，必须编译时确定
- 所以作者实现了Array2D来模拟二维数组

```C%2B%2B
Array2D<int> data;
cout << data(2, 3);       // opeartor(int dix1, int idx2)
```



例子2：String含有内部类CharProxy。

- 对于String s1, s2;     s1[3] = s2[3];
- 用CharProxy类来代理StringValue的operator[]()
- 如果接下来又调用了CharProxy::operator=()操作，那就说明是写操作呀
- 所以可以区分StringValue的non-const opeartor[]()是读还是写操作。p217

```C%2B%2B
class String {
friend class CharProxy;
public:
    class CharProxy {    // 用于代理
    public:
        CharProxy(String& str, int index);
        CharProxy& operator=(const CharProxy& rhs);  // 如果被调用了，就说明operator[]是写操作呀
        operator char() const;
    private:
        String& theStirng;
        int charIndex;
    };

    // 被CharProxy代理，用于区分opeartor[]是读还是写操作，进而更精细化的控制引用计数
    const CharProxy operator[](int index) const;  
    CharProxy operator[](int index);
private:
    RCPtr<StringValue> value;
}
```

![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=M2Y5OTU4OWRhMzcyMzI1Mjg5ODk1MmFjM2VmMTIzMjlfNzdSN0dacUU1VXdmdTNhenc1NzAwSGZCQ1BOVzMxMG9fVG9rZW46Ym94Y25pYjVYQWlMTW8xdndJd2FyRmZFVzZ5XzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)

**代理技术大多数时候是满足需要的，但也有存在的缺点：**

- p223，CharProxy不可能定义完所有的运算符。所以原来 char* p = &stringObj[1]的地方，用char* p = &charProxy就不可以实现。
- p225，原来的类的public函数没有被代理类重载，那么也是不可以调用的。比如Rational.someFunc()是可以调用的，Array<Rational> array.someFunc()是不可以调用的。
- p226。reference-to-non-const的函数形参，是不能接受代理对象的。
  - 使用后，s[0]是CharProxy类型的，有char()类型转换也不能调用。 因为CharProxy->char后，生成的char是临时对象，临时对象是无法被引用的

```C%2B%2B
String s = "+C+"; 
// 不使用CharProxy，可以传入s[0]；
// 使用后，s[0]是CharProxy类型的，有char()类型转换也不能调用。
// 因为CharProxy->char后，生成的char是临时对象，临时对象是无法被引用的
void swap(char& a, char b);     
```

![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=NTMzNzVhYzNkZjQ1MWNmNTc1MzM4MDYyM2MyZGYwMTdfTExSbmlCMDJmN3dQTVY3WnhBcDZEVjF6MTdCVlZlelBfVG9rZW46Ym94Y240d2wwWDBxQVBTNTZHNUVxM1FpQ0tmXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)

- p227，如果没有代理类时，编译器能把int->TVStation，进而在watchTV(TVStation& station)中使用。那么有了代理类，如果编译器用过关于CharProxy的隐式类型转换后，可能proxy<int>->int后，编译器会限制进一步的转换，那么能就不能用转换了。P227



## 31 让函数根据一个及以上的对象类型来决定如何虚化

process(obj1, obj2) 的虚拟化，相当于(obj1, obj2).process()

1. p230-231，最基本的面向过程的函数：①会抛出非GameObject异常，**②在新增Base的子类时需要给每个子类重写****多个参数的虚函数。**
2. p232-233，虚函数+RTTI：还是存在需要改子类代码的问题，问题②
3. p233-245，只用虚函数、仿真虚函数表格等等，存在问题①②等，虽然某些还可以，但最好的实现是下面这种。
4. p245，提供了一种最好的碰撞处理函数的示例。
   1. p249，虽然问题2出现了还是要重新写一定的代码，但是比改每个文件好多了p249
   2. p250，有好好写了写Register，Map的curd，可以更好的来process1, process2了。



# Miscellany 杂项讨论

## 32 在未来时态下发展程序

关于拷贝构造函数copy ctor和赋值函数assignment operator=()的建议：

1. 如果一个类里面有成员指针，一定要显示定义一个copy ctor+assignment或者定义它为private。否则合成的可能会报undefine或者空指针问题。p253



其他建议：

- p254，为了更好的采用封装的性质，请尽量采用匿名namespace，static global func，static obj。

- p255-256，

  有一些

  **老版本的string**

  是没有virtual dtor的

  ，所以这些string不能作为base class，如果用户违反了规则，就会在析构时出问题。

  - 书中提到一些古老版本的类string，此类写了很多注释强调用户不能继承它，因为它没有提供virtual dtor。但是因为它没有从编译上杜绝继承，所以用户还是会违反的，所以最好就是杜绝让用户继承它。



## 33 将非尾端类设计为抽象类

**如果没有AbstractAnimal会存在什么情况：**

1. p259，**基类函数部分赋值问题(只有Animal的成员会被赋值的问题)**。Animal是基类, Chicken *c1= &lizard1, *c2= &lizard2,   *c1 = *c2 会调用Animal::operator=()。
2. p260-261，虚函数的方案。内部用了dynamic_cast可能会出现异形赋值，会throw exception
3. p262，还是会存在throw exception的问题。
4. p263甚至连同类型的赋值都不行了，Animal->Animal
5. **p264->265，所以建议用下面的这种抽象类的方法。**
   1. 可以同形赋值，避免了异形赋值和**部分赋值**。p265-line2

![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=YmM0MmRiNjI1MTQzNzk4NTUxNTliZTA2MmVkNDQxOTRfV2JLWWVHeWw2VmtxRm9lT0NKZnlpZ2U3NldPMVNGd0JfVG9rZW46Ym94Y25wMk43QlV4bDBRMU5KNDlQWTlaYndmXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=NTEyZTRhODgzOGZjMDJlMzdiMWVkNDA5Y2RiMTExMWZfYlhRd0IzTmFLVHZJQ2VyUEFiMjljVDJ4UFNKMDRHU0RfVG9rZW46Ym94Y25mdDRxMWNRU0ZUQTkxM2Q2OTJMSktnXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)

![img](https://qppw4bc6rk.feishu.cn/space/api/box/stream/download/asynccode/?code=NDhmZmQzMDIzNWM0NjNhYzNhOGM2M2U1YzcyMDA3NjlfVjF4MFlVeE9SVERNWWltOFNRa3BDV2NFdjFOWWEwRDFfVG9rZW46Ym94Y25TUUx1MFY3SDN1YnFFZXVxWVkxRW9kXzE2ODg1NjA0MzQ6MTY4ODU2NDAzNF9WNA)

1. p268-270。什么时候应该设计成抽象类+具体类的方式呢？

   如果你发现连续两个类都不是抽象类的时候，也就是一个具体类继承了另一个具体类的时候

   1. **纯虚的析构函数**，必须被实现，即用空实现。
   2. 其他的纯虚函数，是不用实现的。

```C%2B%2B
class AbstractAnimal {
protected:
    AbstractAnimal& operator=(const AbstractAnimal& rhs);

public:
    virtual ~AbstractAnimal() = 0;      // 纯虚的析构函数必须被实现，其他的虚构函数不一定被实现。因为子类的dtor会调用它。p266
};
```



## 34 在同一个程序中开发C和CPP

Tips:

- p270，编译器A编译出的文件，和编译器B编译出的obj，可能不能同时被链接

- 链接器ld往往要求所有函数不能同名，但是cpp overload会重名的命名函数，所以name mangling(命名重整)会重命名函数，比如drawLine----- 命名重整(name mangling)---> xxxy()。

- p273，

  有些代码会在main开始前构造，main结束后释放。

  - 具体来讲有p273：全局对象、文件范围内的或namespace空间（静态+非静态的）内的对象，**static class对象（即class的static成员)**。

- > 全局对象：是外部可以extern的对象

- > 补充一下static的用法总结：

  - > C语言的标准:

    - > ① static globalvar;      ② function() {   static var;  // static局部对象，会在第一次被调用是初始化。}

    - > static function(), 

  - > CPP的标准

    - > class Item { static mem_var;  }

    - > class Item { static funciton(); }

> https://blog.csdn.net/Bruce_0712/article/details/56480613

```C%2B%2B
GlobalVar va1;
static GlobalVar va2;

namespace {
    GlobalVar va3;                // 匿名空间中的，非静态全局对象
    static GlobalVar va4;       // 匿名空间中的，静态全局对象
}

class GlobalVar {
    static int static_class_var;   // 类的静态成员变量
};

int GlobalVar::static_class_var=100;    // 在GlobalVar第一个object出现时，就已经初始化了，参考
main() {
                            // 编译器自动生成的，比 main code还前面的对象的构造
    // real code 
                            // 编译器自动生成的，比 main code还后面的对象的系统，先系统static
}
```



## 35 习惯于C++语言

- 要用STL什么的，
- STL=算法+数据结构+迭代器，什么的也就没必要了。

暂时无法在文档外展示此内容





# 彩蛋（完结！！！）