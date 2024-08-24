### 深度探索C++对象模型（总结篇）

### 

### 

### C++自己的对象模式

C++对象采用一个连续的内存进行储存，一个内存大致会被分为两块，一块是数据本身的储存，一块是virtual table的储存

对于数据本身的存储

- static member和static function与nonstatic function一样，都单独的储存在属于他们的内存中。
- nonstatic member data一般会和虚函数表放在一起，可能会在虚函数表的前面或者后面。

对于virtual table的存储

- virtual table里面储存大量的指针，这些指针包括

  - virtual 析构函数的指针

  - virtual member

  - 每一个class 所关联的type-info指针

  - 虚拟继承的关系

- 使用virtual table带来的好处与坏处

  - 好处：真正使得virtual function得以实现，一并的也有virtual class（保证多次继承只有一个实例）
  - 坏处：增加了访问成本，导致额外的时间与空间负担

- 一个object里面可以含有多个virtual table，这个情况就是在多重继承的情况下。



### Data member

布局

- 对于nonstatic data member，他们会按照声明的顺序，依次进行创建和排列
- 对于no data member 的class，编译器回往里面塞上一个char，确保他占用一个字节的空间，在运行的时候，编译器也会为他合成出来对应的constructor和destructor
- 对于static 类型的数据，他只有一个实例，并且存在data segment中，如果去他的地址，得到的是对于数据类型的指针，而与其所在的对象毫无关联
- 对于function member，只有他们在需要的时候会被编译器合成出来，并且放在内存中，一般是与对应的class在连续的内存中

存取（通过指针和通过对象进行存取的区别）

- 对于static data member，无论是通过指针还是通过对象都是一样的，因为他是单独放在data segment中。
- 对于nonstatic data member
  - 一般来说通过对象存取的本质是通过this指针完成
  - 在内存上的时候，想要对一个nonstatic data member进行存取操作的时候，编译器需要将class object的起始地址加上data member的偏移位置
  - 从时间上看，从对象的角度来说存取一个nonstatic data member，与C存取一个struct member是没有区别的
  - 但是在与virtual class，virtual function来说，使用指针和使用class，存取速度就会有重大的差别，指针一般就会更加慢一些，class则不然，原因之一就是其使用了virtual table，其次就是因为编译时期我们不知道他到底是那种类型，需要在执行期间确定，也就是我们说的动态绑定

继承

- 一般来讲，非virtual的继承相对于比virtual的继承存取时间上不会增加额外的负担
- 对于单一或多次继承，without virtual
  - 对于单一继承或者是多次继承，无论是继承几次，其访问时间上是不会有很多差异的，基本与C struct相同
  - 对于单一继承或者是多次继承，在空间上，随着继承次数的增多，空间的变化往往要比class内部所含的数据所占的总内存要大，原因是他有需要用来进行内存对齐，填补的bytes
- 对于带有virtual function的继承
  - 空间上，由于导入了一个和class有关的virtual table，用来存放他申明每一个virtual function，这样会导致空间的增加，
  - 时间上，也是因为table 间接转换导致的效率低下也是可见的
  - 他也带来的加上constructor和destcuctor，可以帮助他们设定vptr和销毁vptr
- 对于多重继承
  - 对于多重继承，其时间上的差异与空间上的差异很多，因为它是有多个class继承到单一class的一个非自然继承
  - 在内存上，对于一个多重派生对象，一般来讲将起始地址给定一个基本的base class，其后面的class都要加上或者减去一个对应的数字，用于修改地址
  - 在时间上，多重继承的对象，使用指针和对象进行访问的时间与单一继承带有virtual访问的时间相似
- 对于虚拟继承
  - 与多重继承不同，虚拟继承要支持一种share类型的存储方式（也就是所谓菱形储存）
  - 布局的策略一般是先安排好 derived class 的不变部分，再安排其共享部分

效率

- 对于封装：对于封装如果把优化开关打开，封装就不会带来执行期间的效率成本
- 对于聚合：单一的聚合操作也是一样，在优化开关打开的情况下，聚合也不会带来执行期间的效率成本
- 对于继承：非virtual 的继承效率要远远好处virtual。

指针

- 取得一个nonstatic data member（例如：&A：：x）的地址，将会得到他在class 中的offset，
- 取一个绑定于真正class object身上的data member（例如&A.x )的地址，将会得到真正的地址
- 如果进行优化，那么在运行的时候，非virtual 继承使用指针或者是对象的存取效率是相同的
- virtual 继承随着虚拟函数增多而变大，额外的间接性会降低优化能力



### Function

function的底层的调用方式

- 对于nonstatic member function：

  - 效率上，他与nonmember function，static member具有相同的效率，原因是nonstatic member function会被转换为nonmember function。（this指针）
  - 名称上，class内部的value和function会拥有独一无二的姓名，一般是class名字+变量名

- 对于virtual member function

  - 对于一个指针去调用virtual member function，将会转化成指针指向对于的virtual table解引用再连接对应的成员指针

    ```c++
    ptr->a()==(*ptr->vptr[1])(ptr);
    //其中，vptr也有可能有多个，并且可能会被重新转化成一个新的，独一无二的姓名
    ```

  - 在多重继承之下，所有的指针问题都需要在执行期间进行操作

- 静态成员函数

  - static memeber function的特性是没有this指针，因此他不能直接存取nonstatic members，不能被声明为const ,volatile,virtual，不需要经过class object才能调用（但是可以通过class object调用，这是允许的）
  - 由于它缺乏this指针，因此差不多等同于nonmember function

- inline函数

  - 毫无疑问，inline函数能一定幅度提升函数性能，与之而来的付出代价是程序体积的增大
  - 一般来说，编译器通过计算assignments，function call，virtual function calls的操作次数的综合计算inline函数的复杂性

Pointer-to-member-Function

- 对于一个nonmember function，static member function取地址的话，所取即所得
- 对于member function取地址的话并不是真实的地址，而是对应的offset（上面有写）
- 对于一个virtual member function取地址的话，得到的是virtual table的索引值。



### Constructor ,Destructor,copy

#### default constructor

合成

- default constructor:对于default constructor，只有其在需要的时候才会出现，并不是所有的时候都会合成出来一个default constructor，甚至是class本身都没有指定的constructor的情况下
- 如果class 内部含有value且没有对应的default constructor，那么编译器会帮你合成一个，合成的时机是当真正要用到class 的时候
- 如果class 中声明，继承有virtual class/function，那么也编译器也会合成出default constructor，以便正确初始化class object 的vptr以及virtual table
- 编译器合成出来的default constructor不会显式设定class每一个data member的默认值，编译器合成出来的只是编译器需要的，与是否初始化毫无关系

调用

- 如果class A内含一个或者一个以上的member class objects，那么class A的每一个constructor都调用对应的default constructor，如果不想让A调用default constructor，那么记得在初始化列表中加入对应的初始化
- 如果设计者提供多个constructor，但是其中都没有default constructor的情况下，那么编译器会扩展每一个constructors，将用以所必要的default constructor加进去
- 不要把纯虚函数用在声明constructor上

什么时候使用初始化队列

- 当你初始化一个reference member时候
- 当你初始化一个const member时候
- 当你调用一个base class constructor，而他有参数的时候
- 当你调用一个member class的constructor，而它拥有一组参数时
- 注意：list 中的项目顺序是由class 中member 声明顺序决定的，不是由初始化队列中的排列顺序决定的

#### copy constructor

合成

- 所谓的default copy constructor，也就是bitwise copy，这就是默认的拷贝构造
- 在class 不展现bitwise copy的情况下，默认的copy constructor才会被编译器产生出来

调用

- 三种可能的调用动作：X xx=x，作为函数初值，return xx

  

什么时候不展现bitwise copy

- 当class 内含有一个member object但是后者class 声明有一个copy constructor时候。
- 当class 继承自一个base class 而后者存在一个copy constructor时
- 当class 声明了一个或多个virtual function时
- 当class 派生自一个继承串链，其中有一个或者多个virtual base class

#### destructror

- destructor：如果class 没有定义destructor，那么只有在class 内含有的member object 拥有destructor的情况下，编译器才会自动合成出一个，否则destructor被视为不需要，也就不需要合成
- destructor的调用顺序
  - destructor的函数本体首先被执行
  - 如果class 拥有member class object 而后者拥有destructor，那么他们会以其声明顺序相反顺序调用。
  - 如果object内含一个vptr，现在被重新设定，指向适当的base class的virtual table
  - 如果有任何直接上一层的nonvirtual base classes拥有destructor，他们会以声明的顺序相反的顺序进行调用
  - 如果有任何virtual base classes拥有destructor，且目前这个class 是最尾端的class 那么他们会以其原来的构造顺序相反的顺序被调用
