## Chapter 1. Accustoming Yourself to C++

### Item 1: View C++ as a federation of languages

经验1：将C++语言看做不同语言的集合

摘要：将C++ 视为不同子语言（sublanguage）的组合进行学习。有利于理解在不同语言环境下的特殊语法。

主要有四种子语言 C、面向对象的C++、模板c++和STL。



### Item 2: Prefer const, enum, and inline to #define

经验2：尽量的使用 const、enum 和inline 而不是#define。

摘要：使用const enum 和inline，使程序尽量的避免进行预编译工作（preprocessor）

#### 1，使用const 代替 define

define 是在预处理阶段，而const 是在编译阶段进行。define 只是简单的进行原位的替换，没有类型检查。const 本质是一个只读的变量，不可以更改，编译器会检查其类型是否匹配。

使用define 的缺点：

* define 定义符号不会进入符号列表
* define 盲目替换，造成多份代码副本
* define 无法使用范围限制。对全局起效

使用const 的优点：

* const 声明变量进入符号表，和常规变量一致。

~~~C++
const std::string authorName("Scott Meyers"); 
~~~

* 使用const 可以控制作用范围，比如在类中声明。

~~~C++
class GamePlayer { 
private: 
 static const int NumTurns = 5; // constant declaration 
 int scores[NumTurns]; // use of constant 
}; 
~~~

加上 static 的const 表明在类的所有对象中，仅有一份，且不可更改。

有编译器要求，对所有声明的变量，必须提供定义，以上代码可以改为如下形式：

~~~C++
class CostEstimate { 
private: 
 static const double FudgeFactor; // declaration of static class 
 ... // constant; goes in header file 
}; 
const double // definition of static class 
 CostEstimate::FudgeFactor = 1.35;// constant; goes in impl. file 
~~~

#### 2，使用enum 代替define

以上定义类中静态常量的方式，是比较经典常用的。但是在编译类时，如果需要确切的知道一常量的值，需要使用enmu技术了。

~~~C++
class GamePlayer { 
private: 
 enum { NumTurns = 5 }; // "the enum hack" — makes 
 // NumTurns a symbolic name for 5 
 int scores[NumTurns]; // fine 
}; 
~~~

在编译私有成员数组score 时，编译器需要确切知道数组的大小，所以使用enum 定义常量。而不建议使用define。

另外，enum 变量，不能对其取地址，所以当不想暴露变量地址时，可以使用enum。

#### 3，使用内联函数代替宏定义函数

Not good, 不建议使用。

~~~C++
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
~~~

建议使用，内敛函数方式：

~~~C++
template<typename T> // because we don't 
inline void callWithMax(const T&a,const T&b) { // know what T is, we 
 // pass by reference-to-const — see Item 20 
 f(a > b ? a : b); //
} 
~~~

终结：使用const enum 和inline，使程序尽量的避免进行预编译工作（preprocessor）。

### Item 3: Use const whenever possible

经验3：任何时候，尽可能的使用const。

摘要：使用const 限制变量成为一种编程习惯。

#### 1，根据const 相对星号的位置判断语义

**以星号 “*” 为界**，如果const 在星号的左侧，表示其所指向的内容不可改变，如果const 在星号的右侧，表示其指向不能改变。

~~~c++
char greeting[] = "Hello";
char *p = greeting; 				// non-const pointer and non-const data
const char *p = greeting; 			// non-const pointer but const data
char * const p = greeting; 			// const pointer but non-const data
const char * const p = greeting; 	// const pointer but const data
~~~

注意区别，以下语句含义是一样的，仅是写法不同。

~~~c++
void f1(const Widget *pw); 	// f1 takes a pointer to a constant Widget object
void f2(Widget const *pw); 	// so does f2
~~~

#### 2， STL 中迭代器等价于 T* 

STL 中的迭代器，为一个类型的指针，使用const 可以分为两种情况。

* const std::vector\<int>::iterator

STL 中的迭代器是以指针为基础实现建模的，当声明一个 iterator const 时，相当于声明了一个指针常量 即：指针的指向不可改变，指向的内容可以改变。

~~~C++
std::vector<int> vec;
vec.begin();
const std::vector<int>::iterator iter = 	// iter acts like a T* const
*iter = 10; 								// OK, changes what iter points to
++iter; 									// error! iter is const
~~~

* std::vector\<int>::const_iterator

当需要其指向的内容不可改变时，需要声明一个  const_iterator  常量迭代器。具体解释如下：

~~~~c++
std::vector<int>::const_iterator cIter = 	// cIter acts like a const T*
vec.begin();
*cIter = 10; 								// error! *cIter is const
++cIter; 									// fine, changes cIter
~~~~

#### 3， 常量函数语义

~~~C++
const Rational operator*(const Rational& lhs, const Rational& rhs);
~~~

可以有效的防止无意间的连续赋值，此时编译会报错！

~~~~C++
if (a * b = c)... // oops, meant to do a comparison!
~~~~

在类中将成员函数声明为const 有两点好处：

* 指明成员函数能否改变对象，使得接口简单易懂。具有语义限制。
* 使其可以操作常量对象，并以常量参数引用的方式传递参数，提高程序效率。

#### 4，不同常量性的函数重载

c++中一个重要的特性是，可以对两个函数仅仅是常量性（constness）不同的函数进行重载。

以下的代码展示了类中括号 [ ] 取值操作的重载。

~~~C++
class TextBlock {
public:
    const char& operator[](std::size_t position) const
    { return text[position]; } 					// operator[] for const objects
    char& operator[](std::size_t position) 		
    { return text[position]; } 					// operator[] for non-const objects
private:
	std::string text;
};
~~~

对取值操作的使用：

~~~C++
TextBlock tb("Hello");				// TextBlock 's operator[] s can be used like this:
std::cout << tb[0]; 				// calls non-const								
const TextBlock ctb("World");		// TextBlock::operator[]
std::cout << ctb[0]; 				// calls const TextBlock::operator[]
~~~

以上代码说明了：

* 对于声明为const 的常量（ctb）只能调用 带有const 的 operator[]
* 为了防止对返回值的修改，需要对返回值加上const，就防止以下情况发生

~~~C++
ctb[0] = 'x'; // error! — writing a const TextBlock 
~~~

因为调用了 带有const 的 operator[] 取值操作，返回了 const 的变量，所以对其修改时，产生错误。

**对常量函数 const 的解释：**

~~~C++
const char& operator[](std::size_t position) const
    { return text[position]; } 					// operator[] for const objects
~~~

* 第一个const 是对函数返回值得限制，反之调用函数对其修改
* 第二个const 是对传入参数的限制，不允许函数对其进行修改
* 第三个const 是对函数的限制，即不允许函数修改任何的变量。

#### 5，mutable 例外

如果确实需要对在，const函数中对变量进行修改，使用mutable 修饰变量，如下：

~~~C++
class CTextBlock { 
public: 
 std::size_t length() const; 
private: 
 char *pText; 
 mutable std::size_t textLength; // these data members may 
 mutable bool lengthIsValid; // always be modified, even in 
}; // const member functions 

std::size_t CTextBlock::length() const { 
 if (!lengthIsValid) { 
     textLength = std::strlen(pText); // now fine 
     lengthIsValid = true; // also fine 
 } 
 return textLength; 
} 
~~~

在const 常量函数 CTextBlock::length() 中，其不允许进行复制操作（=），而对于两个变量（textLength和lengthIsValid）在逻辑上式允许改变的，此时需要，将两个变量生命为 mutable 即可。

#### 6，避免 const 和non-const 函数的代码重复

有时const 函数和non-const 函数存在大量代码重复 ，此时仅需要使用，非const 函数调用 const 函数即可，不足之处是需要进行两次const属性的转换。

~~~C++
class TextBlock { 
public: 
 const char& operator[](std::size_t position) const  { 
 return text[position]; 
 } 
// now just calls const op[] 
 char& operator[](std::size_t position) {
 	return const_cast<char&>( 	// cast away const on op[]'s return type;
 		static_cast<const TextBlock&> (*this)[position] );  
 	 	// add const to *this's type call const version of op[] 
 } 
}; 
~~~

* 首先通过 static_cast<const TextBlock&> 将类TextBlock 转换为const 属性，使其调用 const 函数
* 再通过const_cast<char&> 将返回值去除 const 属性

通过以上两部，即可实使用 const 函数实现 non-const 函数操作。

总结：

* 通过const 限制，帮助编译器发现使用错误。const 可以使用在任何的范围内。
* 编译器执行物理的常量性（bitwise constness）。但是程序员需要保持逻辑的常量性（logical constness）
* 使用非常量函数调用常量函数防止代码重复



### Item 4: Make sure that objects are initialized before they're used

**经验4**：确保对象在使用前被初始化

**概要**：对于内建的数据类型，必须手动的进行初始化。如果是自定义的类，推荐使用初始化列表，对成员变量进行初始化。

#### 1，内建变量必须手动初始化

~~~C++
int x = 0;		// one way
class Point {
    int x, y;	// another way, not sure 
};
~~~

C++ 不能保证在任何时候，内建的数据都能正确的初始化，所以最保险的方式是在使用前手动进行。

#### 2，高效的初始化列表

定义类 ABEntry。代码如下：

~~~C++
class PhoneNumber {... };
class ABEntry { // ABEntry = "Address Book Entry"
public:
    ABEntry(const std::string& name, const std::string& address,
    const std::list<PhoneNumber>& phones);
private:
    std::string theName;				
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int num TimesConsulted;
};
~~~

**在构造函数中对成员变量进行的操作是赋值，而不是初始化，要注意区分**。比如对成员变量theName，首先会调用string 类型的默认构造函数构造 theName变量，然后在 ABEntry（顶层默认构造函数）的构造函数中对其进行赋值。这样造成了不必要的操作浪费。如果使用初始化列表的形式，直接的调用赋值构造函数，“两部并做一步”效率更高。

* 极力不推荐方式：

~~~~C++
ABEntry::ABEntry(const std::string& name, const std::string& address,
const std::list<PhoneNumber>& phones)
{
    theName = name; 		// these are all assignments,
    theAddress = address; 	// not initializations
    thePhones = phones
    numTimesConsulted = 0;
}
~~~~

* **推荐方式：**

~~~C++
ABEntry::ABEntry(const std::string& name, const std::string& address,
const std::list<PhoneNumber>& phones)
    : theName(name),
    theAddress(address), 	// these are now all initializations
    thePhones(phones),
    numTimesConsulted(0)
{} 							// the ctor body is now empty
~~~

另外当类中含有引用 reference 和 const 成员变量的时候，必须使用初始化列表对其进行初始化。

#### 3，初始化顺序造成的错误

* 对于类：**谨记初始化列表变量的顺序和定义顺序要一致。**

C++中初始化的顺序是：

1）基类先于子类。使用子类构造函数参数，先对基类进行初始化（即，子类初始化列表调用基类构造函数）。

2）在单个类中的初始化顺序与其声明的顺序是一致。

* 对于在不同文件中的非局部的静态变量的初始化，存在未定义而进行使用的风险。

**场景：**一个变量的初始化需要用到另外的一个变量时，初始化的顺序不对，会造成未定义错误。

**解决的方式是：**使用局部静态变量代替非局部静态变量，并返回该局部静态变量的引用。

**原因：**C++ 保证第一次遇见局部静态变量时，就进行初始化。（类似单例模式）

在一个文件中定义相关类和全局变量。

~~~C++
// file1
class FileSystem { 					// from your library
public:
    ...
    std::size_t numDisks() const; 	// one of many member functions
    ...
};
extern FileSystem tfs; // object for clients to use; "tfs" = "the file system"
~~~

在另外一个文件中定义相关类和全局变量。

~~~C++
// file2
class Directory { // created by library client
public:
    Directory( params);
    ...
};
Directory::Directory( params)
{
    ...
    std::size_t disks = tfs.numDisks(); // use the tfs object
    ...
}
~~~

假设现在开始使用Directory 进行创建。

~~~C++
Directory tempDir( params)
~~~

问题分析：除非tfs对象 比 tempDir 对象初始化的早，否则 在tempDir 对象就使用了未初始化的对象。这种在不同文件中的初始化，顺序C++是无法保证的。

* **解决方式：**

使用局部静态变量代替全局静态变量，并返回其引用。

~~~C++
class FileSystem {... }; 		// as before
FileSystem& tfs() 				// this replaces the tfs object; it could be
{ 								// static in the FileSystem class
    static FileSystem fs;		// define and initialize a local static object
    return fs; 					// return a reference to it
}
class Directory {... }; 		// as before
Directory::Directory(params) 	// as before, except references to tfs are
{ 								// now to tfs()
    ...
    std::size_t disks = tfs().numDisks();
    ...
}
Directory& tempDir() 			// this replaces the tempDir object; it
{ 								// could be static in the Directory class
    static Directory td; 		// define/initialize local static object
    return td; 					// return reference to it
}
~~~

以上能够完美解决的前提是，**C++保证局部静态变量在第一次遇到时候，即进行初始化**。所以在函数tfs和tempDir中，返回创建的非全局变量的引用。从而保证了在此使用的时候，已经进行了初始化操作。



## Chapter 2. Constructors, Destructors, and Assignment Operators

### Item 5: Know what functions C++ silently writes and calls

经验5：知道c++默默的生成和调用了何种函数

概要：当定义空类的时候，编译器会默认的生成，构造函数，拷贝构造函数，拷贝赋值函数和析构函数。

#### 1，定义空类的 “背后” 操作

~~~c++
class Empty{};
~~~

当定义空类时，等同于如下的操作：

~~~c++
class Empty {
public:
    Empty() {... } 					// default constructor
    Empty(const Empty& rhs) {... }  // copy constructor
    ~Empty() {... } 				// destructor — see below
    								// for whether it's virtual
    Empty& operator=(const Empty& rhs) {...} // copy assignment operator
};
~~~

在合适的时机，编译器会创建以上函数，进行调用：

~~~C++
{
    Empty e1; 		// default constructor and destructor
    Empty e2(e1); 	// copy constructor
    e2 = e1; 		// copy assignment operator
}
~~~

#### 2，默认拷贝“失效” 的发生

编译器创建的默认函数，都是基于常规的操作，有些赋值不符合语法的，肯定是要报错的。

**正常的情况**

~~~C++
template<typename T>
class NamedObject {
public:
    NamedObject(const char *name, const T& value);
    NamedObject(const std::string& name, const T& value);
    ...
private:
    std::string nameValue;
    T objectValue;
};
~~~

**使用方式：**

~~~C++
NamedObject<int> no1("Smallest Prime Number", 2);
NamedObject<int> no2(no1); // calls copy constructor
~~~

**运行分析：**

先创建NamedObject 类的一个对象no1，然后调用默认的拷贝构造函数生成no2。编译器生成的默认拷贝构造会分别的利用no1中的nameValue 和objectValue 对自己的两个成员变量进行初始化。no2.nameValue 会以no1.nameValue 的值为参数调用 string 的拷贝构造函数，因为 T 为int 类型，进行内建数据类型的赋值。

**失效情况：**

~~~C++
template<class T>
class NamedObject {
public:
    // this ctor no longer takes a const name, because nameValue
    // is now a reference-to-non-const string. The char* constructor
    // is gone, because we must have a string to refer to.
    NamedObject(std::string& name, const T& value);
    ... // as above, assume no operator= is declared
private:
    std::string& nameValue; // this is now a reference
    const T objectValue; 	// this is now const
};
~~~

此时类的两个成员变量分别为 **引用类型和const 类型**（他们的初始化必须通过，初始化列表），当编译器再次调用默认生成的拷贝构造时候，发现 nameValue 和objectValue 无法进行初始化（即尝试修改引用和const常量）。即拷贝构造失败。

**总结：**

如果成员变量自身无法完成赋值操作，那默认的拷贝构造肯定会失败，此时需要自行的定义拷贝构造函数。



### Item 6: Explicitly disallow the use of compiler-generated functions you do not want

经验7：显示的不允许编译器生成不需要的函数

概要：当不想进行拷贝构造或是拷贝赋值时，1）显示的声明对应的函数为私有成员函数，2）或是私有继承不可拷贝的基类。

#### 1，将拷贝函数声明为私有

注意：将拷贝复制函数和赋值操作函数声明为私有，**且不需要传入参数，不需要进行相应定义**

~~~C++
class HomeForSale {
public:
	...
private:
    ...
    HomeForSale(const HomeForSale&); 	// declarations only
    HomeForSale& operator=(const HomeForSale&);
};
~~~

此时再次的调用一下 code line 即报错：

~~~C++
HomeForSale h1;
HomeForSale h2;
HomeForSale h3(h1); // attempt to copy h1, not allowed
h1 = h2; 			// attempt to copy h2, not allowed
~~~

当进行拷贝或是赋值是，编译器会报错，如果通过友元函数，不经意的调用，链接也不会成功。可以通过继承的方式，将链接错误，前置到编译阶段，如下所示。

#### 2，定义不可拷贝的基类

~~~C++
class Uncopyable {
protected: 				// allow construction
    Uncopyable() {} 	// and destruction of
    ~Uncopyable() {} 	// derived objects...
private:
    Uncopyable(const Uncopyable&); //...but prevent copying
    Uncopyable& operator=(const Uncopyable&);
};

class HomeForSale: private Uncopyable { // class no longer
	... 								// declares copy ctor or
}; 										// copy assign. operator
~~~

当进行类HomeForSale 对象的拷贝时，其拷贝函数会先调用基类的拷贝函数，但基类的拷贝函数为私有属性，所以编译器会报错。从而达到HomeForSale 的类对象不可拷贝的目的。

注意：1）此时的基类Uncopyable的析构函数并不需要为虚函数，因派生类为私有继承。2）基类中函数的保护属性和私有继承。



### Item 7: Declare destructors virtual in polymorphic base classes

经验7：在多态的基类中将析构函数声明为虚函数

概要：用作多态的基类，其析构函数必须为虚函数。防止析构资源时的“半销毁”状态。不用作多态基类的析构函数不要为虚函数。

**0，添加虚析构的使用范围**

1）只有用作多态的基类，其析构函数必须为虚函数！即允许通过基类的接口，操作子类。比如下面定义的多态基类 TimeKeeper， 通过工厂函数获得了基类的指针，并且通过该指针操作子类对象 AtomicClock 和 WaterClock 等。

2）其他不用做多态的基类，不需要声明虚析构。即“半销毁”状态，只有在多态析构的情况下发生。

#### 1，多态析构时，“半销毁”状态问题。

定义多态行为，如下：

~~~C++
class TimeKeeper{	// 多态基类
public:
    TimeKeeper();
    ~TimeKeeper();	// 注意析构函数状态
    //...
};
// 以下定义派生类
class AtomicClock: public TimeKeeper {... };
class WaterClock: public TimeKeeper {... };
class WristWatch: public TimeKeeper {... };
TimeKeeper* getTimeKeeper(); // 工厂函数
~~~

调用方式：

~~~C++
int main(){
    TimeKeeper *ptk = getTimeKeeper()
    //...
    delete ptk;
    return 0;
}
~~~

通过基类的指针 getTimeKeeper 调用子类的函数。

**问题分析：**

定义多态指针的类型是Timer，而实际类型是子类的派生类类型（AtomicTimer）。因为基类的对象Timer 的析构函数为非虚函数，所以在delete timer1和delete timer2时，仅仅调用了基类Timer的析构函数，而并没有调用子类的析构函数。使得对象将处于一个诡异的 “半销毁” 状态，即其基类部分已经被销毁，而派生类部分的资源并没有被释放。

#### 2，“半销毁”的解决方式

将多态基类的析构函数，设置为虚函数。

~~~C++
class TimeKeeper {
public:
    TimeKeeper();
    virtual ~TimeKeeper();
    ...
};

int main(int argc, char* args[]){
    TimeKeeper *ptk = getTimeKeeper();
    ...
    delete ptk;
}
~~~

在释放资源的时候，首先是调用派生类的析构，再次调用基类的析构，最终资源得到完全的释放。

**注意：** 随意设置类中函数为虚函数，会增加类的大小。

当类不含有虚函数，说明其不准备当做基类来使用。如果为一个不是用来做基类的类声明了虚析构函数，那绝对是一个不好的实践。**因为类中有虚函数，其产生的对象中就会包含虚函数指针，对应的编译器也会创建虚函数表，这无疑增加了对象内存和运行的开销。**

**总结:** 仅当类中含有虚函数时，才会将定义虚的析构函数。

有时候，想声明一个抽象类，但是又没有具体的抽象函数，此时可以将析构函数声明为纯虚构函数，即防止了“半销毁”状态，也满足了抽象类的的要求。

~~~C++
class AWOV { // AWOV = "Abstract w/o Virtuals"
public:
    virtual ~AWOV() = 0; // declare pure virtual destructor
};
~~~

以上需要加入析构函数的定义，否则编译报错。

~~~
AWOV::~AWOV() {} // definition of pure virtual dtor 
~~~

#### 3，工厂函数

在C++中，可以使用工厂函数来封装对象的创建过程，以提供更灵活和可扩展的对象创建方式。工厂函数通常是一个静态成员函数，它返回一个指向基类的指针或引用。通过调用工厂函数，可以创建不同派生类的对象，而无需直接调用派生类的构造函数。

~~~C++
#include <iostream>

class Base {
public:
    virtual void print() = 0;
    virtual ~Base();
};

class Derived1 : public Base {
public:
    void print() override {
       std::cout << "Derived1" << std::endl;
    }
};

class Derived2 : public Base {
public:
    void print() override {
         std::cout << "Derived2" << std::endl;
    }
};
~~~

定义工厂函数：

~~~c++
class Factory {
public:
    static Base* createObject(int type) {
        if (type == 1) {
            return new Derived1();
        } else if (type == 2) {
                       return new Derived2();
        } else {
            return nullptr;
        }
    }
};
~~~

工厂函数的使用：

~~~C++
int main() {
    Base* obj1 = Factory::createObject(1);
      if (obj1) {
        obj1->print();  // 输出 "Derived1"
        delete obj1;
    }
    Base* obj2 = Factory::createObject(2);
    if (obj2) {
        obj2->print();  // 输出 "Derived2"
        delete obj2;
    }
    return 0;
}
~~~



### Item 8: Prevent exceptions from leaving destructors

**经验8**：防止异常逃离析构函数。

**概要**：在类调用析构函数时候，如果不能防止异常的发生，不应该使异常逃离析构函数。解决方式是终止程序或是析构函数使用try 和cash 机制吞下异常。或是定义普通函数处理异常。

#### 1，在析构函数中抛出异常所引发的问题

1）析构函数异常语句之后的命令无法执行，如果必要的释放资源的动作无法执行，会造成资源泄漏问题。

2）在处理异常过程中，再次的发生异常，造成程序的崩溃。

因为在栈上定义的类，发生异常的时候会进行栈展开（stack-unwinding），即因发生异常而逐步退出复合语句和函数定义的过程。栈展开的过程中就会调用已经在栈构造好的对象的析构函数来释放资源，此时若其他析构函数本身也抛出异常，则前一个异常尚未处理，又有新的异常，会造成程序崩溃。

* 使用场景

建立一个数据库连接类DBConnection 并使用资源管理类DBConn 进行资源的管理，在释放类资源的时候进行数据库关闭。

~~~C++
class DBConnection { 
public:
　　 static DBConnection create(); 
　　 void close(); 
};

//这个class用来管理DBConnection对象
class DBConn {
public:
　　DBConn(const DBConnection& db){
       this->db=db;
   }
　 ~DBConn() { //确保数据库连接总是会被关闭
　　    db.close();
　　}
private:
　　 DBConnection db;
};
~~~

#### 2，方案一，让析构函数发生异常时即终止程序

~~~C++
DBConn::~DBconn() {
    try {
	    db.close(); 
    }
    catch(...) {
        std::abort();
    }
}
~~~

在发生异常的时候即终止程序运行。

#### 3，方案二，吐下异常打印记录log

~~~C++
DBConn::~DBConn {
    try{ db.close();}
    catch(...) {
        //制作运转记录，记下对close的调用失败！
    }
}
~~~

#### 4，方案三，让客户端进行调用，并与其进行交互

在资源管理类中添加一个供客户端调用的“关闭”函数，将资源关闭的任务交给客户端，这样提供了一种与异常交互的接口。如果某个操作可能在失败的时候抛出异常，而又存在某种需要必须处理该异常，那么这个异常必须来自析构函数以外的某个函数。因为析构函数吐出异常就是危险，总会带来“过早结束程序”或“发生不明确行为”的风险。

~~~C++
class DBConn {
public:
    void close() {//供客户使用的新函数
        db.close();
        closed = true;
    }
    ~DBConn() {
        if(!closed) {
            try  {      //关闭连接(如果客户不调用DBConn::close) 
                db.close();
            }
            catch(...) {//如果关闭动作失败，记录下来并结束程序或吞下异常
                制作运转记录，记下对close的调用失败；
           }
        }
    }
private:
    DBConnection db;
    bool closed;
};
~~~

以上使用close 关闭数据库连接，如果存在异常则普通函数close 进行抛出，析构函数进行兜底。

**总结：**

1）将关闭资源操作封装为单独的接口函数供用户主动的调用，并设置成功关闭标志。

2）在类的析构函数中，对资源标志进行检查。

3）如果资源没有正常关闭，再次进行关闭资源，如果有异常发生，则采用前两种方案应对



### Item 9: Never call virtual functions during construction or destruction

经验：不要在构造和析构函数中调用虚函数

摘要：

#### 1，构造函数调用虚函数的问题

问题产生的场景。

在买入和卖出股票时，需要进行日志的记录，以备审查。分别建立股票基类和买入卖出子类。

~~~C++
class Transaction { // base class for all transactions 
public:
 Transaction(); 
 // make type-dependent log entry 
 virtual void logTransaction() const = 0;  
}; 
Transaction::Transaction() { // implementation of base class ctor
 logTransaction(); // as final action, log this 
} // transaction
~~~

买入和卖出子类：

~~~C++
class BuyTransaction: public Transaction { // derived class 
public: 
 virtual void logTransaction() const; 
 // how to log transactions of this type 
}; 
 
class SellTransaction: public Transaction { // derived class 
public: 
 virtual void logTransaction() const; 
 // how to log transactions of this type 
}; 
~~~

使用方式：

~~~C++
BuyTransaction b;
~~~

问题分析：

1）当建立子类BuyTransaction时，需要先调用基类的构造函数 Transaction::Transaction()。

2） 在基类构造函数中，调用了虚函数logTransaction()，此时仅会调用基类的函数，并不会到子类中去。

3）属于子类（BuyTransaction-specific parts of the object）还没有构造完毕，相关变量还未初始化，所以编译器认为子类还处于不存在的状态。

4）即在构造函数中不会发生多态行为，“虚函数”并不虚！

#### 2，子类传递必要信息构造父类

解决的办法是，不在构造和析构函数中调用虚函数。将虚函数改为常规函数，并在子类中传递必要信息给基类构造函数。注意，子类中传递信息的帮助函数为静态类型。

~~~C++
class Transaction { 
public: 
 explicit Transaction(const std::string& logInfo); 
 void logTransaction(const std::string& logInfo) const;  
 // now a non-virtual func 
}; 
Transaction::Transaction(const std::string& logInfo) { 
 logTransaction(logInfo); // now a non-virtual call
}  
 
class BuyTransaction: public Transaction { 
public: 
 BuyTransaction( parameters) 
 : Transaction(createLogString( parameters)) // pass log info 
 {... } // to base class 
 ... // constructor 
private: 
 static std::string createLogString( parameters); 
}; 
~~~

帮助函数为静态类型，防止了变量在传递时，为初始化的编译错误。

**总结：**

不要在构造和析构函数中使用虚函数。当基类需要子类信息进行构造时，通过子类的静态帮助函数传递必要信息。



### Item 10: Have assignment operators return a reference to *this

经验10：让赋值运算符返回*this

摘要：为了支持连续的赋值运算，要让赋值运算符返回*this

~~~C++
class Widget { 
public: 
Widget& operator=(const Widget& rhs)// return type is a reference to 
{ // the current class 
 ... 
 return *this; // return the left-hand object 
 } 
  ... 
};
~~~

对于 += 操作，也有同样的约定：

~~~C++
class Widget { 
public: 
 Widget& operator+=(const Widget& rhs // the convention applies to 
 { // +=, -=, *=, etc. 
 return *this; 
 } 
 
 Widget& operator=(int rhs) // it applies even if the 
 { // operator's parameter type 
 ... // is unconventional 
 return *this; 
 } 
}; 
~~~



### Item 11: Handle assignment to self in operator=

经验11：在赋值运算中处理赋值给自己的情况

摘要：通过加入“自己判断”或是构造临时对象进行赋值，解决拷贝赋值函数在给自身赋值时出现的错误。

#### 1，赋值构造给自身赋值时的问题

~~~C++
class Bitmap { ... };
class Widget {
private:
    Bitmap* pb;     //指针，指向一个从heap分配而得的对象
};

Widget& Widget::operator=(const Widget& rhs){    //不安全的operator= 的实现
    delete pb;     					// 删除使用当前的bitmap
    pb = new Bitmap(*rhs.pb);       // 使用 rhs's bitmap的拷贝
    return *this;
}
~~~

* 当拷贝赋值的形参引用rhs 和 *this 不是同一个对象时，没有任何的问题。

* 当两者为同一对象时，**删除了正在使用的 pb，使new 不成功**，进而pb 成为空。

#### 2，方案一，加入自身判断

* 加入自身判断

~~~C++
Widget& Widget::operator=(const Widget& rhs) {
  if (this == &rhs) return *this;   // identity test: if a self-assignment, do nothing
  delete pb;
  pb = new Bitmap(*rhs.pb);
  return *this;
}
~~~

注意：

* 不必要的自身检测带来执行效率的降低
* 根据自身赋值情况的占比，衡量是否需要。

#### 3，方案二，构造临时的对象

~~~C++
Widget& Widget::operator=(const Widget& rhs){
  Bitmap *pOrig = pb;               // remember original pb
  pb = new Bitmap(*rhs.pb);         // make pb point to a copy of *pb
  delete pOrig;                     // delete the original pb
  return *this;
}
~~~

经典的方案，先复制一份，等new构造新的对象后在删除。如果new 发生异常，pb 指针保持不变，新建的临时对象也会在退出函数式，自动销毁。

#### 4，方案三，copy & swap

实现方式一：手动构建临时对象后，再进行交换。

~~~C++
class Widget {
  void swap(Widget& rhs);   // exchange *this's and rhs's data;
  ...                       // see Item 29 for details
};

Widget& Widget::operator=(const Widget& rhs){
  Widget temp(rhs);             // make a copy of rhs's data
  swap(temp);                   // swap *this's data with the copy's
  return *this;
}
~~~

实现方式二：按值传递，编译器完成副本构造后，再进行交换。

~~~C++
Widget& Widget::operator=(Widget rhs) { // rhs is a copy of the object  
    									// passed in — note pass by val
    swap(rhs); // swap *this’s data with the copy’s
    return *this;
}
~~~

方式二基于事实：

1）一个类的拷贝赋值运算符可以被声明为按值传递。

2） 按值传递会对值进行拷贝

3）按值传递，相比于函数内构造临时对象，编译器会更加的高效执行。



### Item 12: Copy all parts of an object

经验12：拷贝对象的所有部分

摘要：1）对于新增加的自定义数据类型，要同时的更新拷贝构造和拷贝赋值函数等相关函数，防止“部分拷贝”。2）在继承关系中，派生类忘记调用基类的拷贝函数，使得基类中的数据成员初始化不正确。解决方式是，在派生类的拷贝函数中，调用基类的拷贝函数。

注意：拷贝函数是指，拷贝构造函数和拷贝赋值操作函数的总称。

#### 1，新增数据后更新拷贝函数及相关

* 自定义拷贝函数，禁止使用编译器默认拷贝函数

~~~C++
void logCall(const std::string& funcName); // make a log entry
class Customer {
public:
    Customer(const Customer& rhs);
    Customer& operator=(const Customer& rhs);
    private:
    std::string name;
};

Customer::Customer(const Customer& rhs):name(rhs.name) { // copy rhs’s data 
    logCall("Customer copy constructor");
};

Customer& Customer::operator=(const Customer& rhs) {
    logCall("Customer copy assignment operator");
    name = rhs.name; 	// copy rhs’s data
    return *this;		// see Item 10
}
~~~

当加入自定义的类型 lastTransaction 的时候，没有更新拷贝函数就发生了部分拷贝。此时编译器不会有任何的警告信息，很难发现这类的错误。

~~~C++
class Date { ... }; // for dates in time
class Customer {
public:
    ... // as before
private:
    std::string name;
    Date lastTransaction;
};
~~~

产生问题的原因是：

1）自定义来了拷贝函数，编译器禁止产生默认版本。

2）增加新数据后，没有及时的更新拷贝函数等。

解决方式是： 在类中新增成员变量时，注意同时的更新拷贝函数和涉及的构造函数等。

#### 2，继承关系中的“部分拷贝”

* 继承关系中，缺失对基类拷贝函数的调用

~~~C++
class PriorityCustomer: public Customer { // a derived class
public:
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer& operator=(const PriorityCustomer& rhs);
private:
    int priority;
};

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
    : priority(rhs.priority) {
    logCall("PriorityCustomer copy constructor");
}

PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs){
    logCall("PriorityCustomer copy assignment operator");
    priority = rhs.priority;
    return *this;
}
~~~

以上代码看似进行了完全的拷贝，但仅仅是对于子类来说。对于基类却只字未提。

当发生子类拷贝时的问题：

* 对于拷贝构造函数，编译器会使用没有参数的构造函数，对基类中的变量进行初始化。如果不存在无参的构造函数，编译器会报错
* 对于拷贝赋值函数，编译器会使用默认基类变量，跟传入的变量的基类无任何的关系。



#### 3，解决：调用基类拷贝函数

* 在拷贝构造函数的初始化列表中，显示调用基类的拷贝构造
* 在拷贝赋值操作函数内，显示的调用基类的拷贝赋值操作函数

~~~C++
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
: Customer(rhs), priority(rhs.priority){  // invoke base class copy ctor
    logCall("PriorityCustomer copy constructor");
}

PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs){
    logCall("PriorityCustomer copy assignment operator");
    Customer::operator=(rhs); 	// assign base class parts
    priority = rhs.priority;
    return *this;
}
~~~

**防止拷贝函数代码重复**

拷贝构造和赋值构造，有时候的代码相似度很高，怎样提高代码利用率。

解决方式为：

* 将共同的代码放入私有函数中（一般命名为init()），供两个拷贝函数调用。

* 不能：拷贝函数之间相互调用。 



## Chapter 3. Resource Management

### Item 13: Use objects to manage resources.

经验13：使用对象管理资源

摘要：为了防止在堆上的资源没有释放，而造成资源泄露。一定要使用对象管理资源，利用对象销毁时候自动调用析构函数，进行资源的释放。可以利用C++中的 RAII 技术。推荐的资源管理类为share_ptr.

#### 1，资源泄露问题

~~~C++
class Investment { /* ... */ };

Investment *createInvestment();	// 工厂函数，返回创建对象的指针

void function( /* ... */ ) {     // 调用函数 
    Investment *invest = createInvestment();
    // ...						// 此处可能发生异常和调用return 和break 等直接返回
    delete invest;
}
~~~

函数 function 在执行 delete 释放资源前，可能发生异常情况，导致delete 不能执行，进而导致资源泄露。

**1）return 直接返回 2）break 和continue 等跳出函数 3）发生异常，程序终止。**

为防止以上情况发生时，导致资源泄露，应该将资源放到类中进行管理。

#### 2，RAII 技术

RAII （Resource Acquisition Is Initialization）技术：即在对象创建时获取对应的资源，在对象生命周期内控制对资源的访问，使之始终保持有效，最后在对象析构时，释放所获取的资源。**RAII 正是利用类来管理资源，将资源与类对象的生命周期绑定**。

~~~C++
void function(){
    std::shared_ptr<Investment> invest(createInvestment());
    //...
}
~~~

以上声明有两个重要操作：

* 在资源获取后，立马将资源移交到资源管理类中进行管理。
* 资源管理类自动调用析构函数，销毁对象。

#### 3，引用计数智能指针 RCSP

资源管理类在销毁对象时。不能存在多个指针指向同一资源，否则造成资源泄露。

处理方式有：

* 仅有一个智能指针指向资源。

典型代表为 auto_ptr, 当发生拷贝时，前一指针将置为Null。

~~~C++
std::auto_ptr<Investment> // pInv1 points to the 
pInv1(createInvestment()); // object returned from createInvestment 
std::auto_ptr<Investment> pInv2(pInv1); // pInv2 now points to the 
 // object; pInv1 is now null 
pInv1 = pInv2; // now pInv1 points to the object, and pInv2 is null 
~~~

* 对引用进行计数，当计数为0时，销毁对象

典型代表为 share_ptr， 当发生拷贝时，计数加一

~~~C++
void f() {
 std::tr1::shared_ptr<Investment> // pInv1 points to the 
 pInv1(createInvestment()); // object returned from createInvestment
 std::tr1::shared_ptr<Investment> // both pInv1 and pInv2 now
 pInv2(pInv1); // point to the object 
 pInv1 = pInv2; // ditto — nothing has changed 
} // pInv1 and pInv2 are destroyed, and the 
 // object they point to is automatically deleted
~~~

**特别注意：**

尽量不要使用智能指针管理数组，使用STL模板， vector 等容器。

~~~~C++
std::auto_ptr<std::string> // bad idea! the wrong 
aps(new std::string[10]); // delete form will be used 
std::tr1::shared_ptr<int> spi(new int[1024]); // same problem 
~~~~

根本原因是，智能指针默认使用 delete 进行资源删除，而创建对象使用了 new []， 两者之间不匹配。

总结：

1）**资源管理方面**

智能指针(std::shared_ptr)是RAII最具代表性的实现，使用了智能指针，可以实现自动的内存管理，不用担心忘记delete造成内存泄漏。

2）**状态管理方面**

线程同步中使用std::unique_lock或std::lock_guard对互斥量std::mutex进行状态管理，通过这种方式，再也不用担心互斥量之间的代码出现异常而造成线程死锁。



### Item 14: Think carefully about copying behavior in resource-managing classes

经验14：在资源管理类中，仔细的考虑拷贝行为

摘要：当拷贝资源管理对象时，需要考虑四种的情况：禁止拷贝，对资源引用计数，拷贝管理的资源，转移对资源的拥有权。针对不同的情形，需要仔细的判别。

#### 1，禁止拷贝

定义一个对锁资源的管理类，在构造函数中进行加锁，在资源销毁调用析构函数时，进行锁的释放。代码如下：

~~~C++
class Lock {
public:
    explicit Lock(Mutex *pm): mutexPtr(pm){
        lock(mutexPtr); 
    }										 	// acquire resource
    ~Lock() { 
        unlock(mutexPtr); 
    } 											// release resource
    private:
    Mutex *mutexPtr;
};
~~~

对于锁，一般对其拷贝是没有意义的，需要禁止。

~~~C++
Lock ml1(&m); 	// lock m
Lock ml2(ml1); 	// copy ml1 to ml2—what should happen here?
~~~

禁止的方式，可以采用继承禁止拷贝的基类：

~~~C++
class Lock: private Uncopyable { 	// prohibit copying — see
    public: 						// Item 6 as before
    ...
};
~~~

也可将其拷贝函数设置为私有属性。

#### 2，对资源的引用计数

典型的例子为 std::share_ptr， 即当对资源引用的数量变为0时，对资源进行销毁。

以下为对锁 lock 引用的计数，当对其引用为0时，期望的是解锁 lock 而不是对锁进行删除，所以利用 share_ptr 对引用进行计数，当计数为0时，析构函数调用传入函数，执行特定的析构行为解锁资源。

~~~C++
class Lock {
public:
    explicit Lock(Mutex *pm) 	// init shared_ptr with the Mutex
    : mutexPtr(pm, unlock){		// to point to and the unlock func as the deleter
    lock(mutexPtr.get());		// see Item 15 for info on "get"
    }
private:
    std::tr1::shared_ptr<Mutex> mutexPtr; 	// use shared_ptr
};							  				// instead of raw pointer
~~~

以上需要注意的是，可以对std::share_ptr 析构函数进行修改，达到释放锁，而不是销毁的操作。

#### 3，对资源进行“深拷贝”

此时，不仅对资源管理类进行了拷贝，同时也对其管理的资源进行了拷贝，即“深拷贝”，使用资源管理类的目的仅是确保每个资源的正确释放。

典型的代表是 标准库中的 string 函数。	

#### 4，资源所有权的转移

此时，需要有且仅有一个资源管理对象指向管理资源。当进行资源管理类的拷贝时，对资源的拥有权，就从被拷贝对象转移到了拷贝对象上。典型的例子就是 auto_ptr.

总结：

* 拷贝的行为，决定了资源管理类中拷贝函数的实现。
* 一般的资源管理类，发生拷贝时进行引用计数，但其它行为也支持。



### Item 15: Provide access to raw resources in resource-managing classes.

经验15：在资源管理类中提供访问裸指针的接口

摘要：在资源管理类RAII中需要提供对裸指针的访问接口，可以是显示和隐式的两种方案。显示的安全，隐式的方便，根据具体的情况进行实现。

#### 1，使用裸指针的情形

~~~C++
std::tr1::shared_ptr<Investment> pInv(createInvestment());
int daysHeld(const Investment *pi);
int days = daysHeld(pInv); // error!
~~~

以上函数daysHeld 传入的形参类型与定义的形参类型是不匹配的。定义需要时 Investment 的指针类型，而传入的却是对该资源的管理类pInv， 编译不会通过。需要在RAII 类pInv 提供对裸指针（Investment） 的访问接口。

对于标准的RAII 函数（std::shared_ptr and auto_ptr）都实现了get() 函数，用于返回管理的裸指针。

~~~C++
int days = daysHeld(pInv.get()); // right!
~~~

通过get 函数的方式称之为 显示的方式返回裸指针。通过对 解引用(->)和星号(*) 的重载实现了隐式的访问裸指针。

~~~C++
class Investment { // root class for a hierarchy of investment types
public: 			
    bool isTaxFree() const;
    ...
};

Investment* createInvestment(); 		// factory function
std::tr1::shared_ptr<Investment> 		// have tr1::shared_ptr
pi1(createInvestment());				// manage a resource
bool taxable1 = !(pi1->isTaxFree()); 	// access resource via operator->
...
std::auto_ptr<Investment> pi2(createInvestment());	// have auto_ptr manage a resource 
bool taxable2 = !((*pi2).isTaxFree()); 				// access resource  via operator*
~~~

如果自己实现对资源的管理类RAII， 需要对 解引用(->)和星号(*) 进行重载，才可使用。

#### 2，RAII类获得裸指针的显示和隐式实现

~~~C++
FontHandle getFont(); 						// from C API—params omitted for simplicity
void releaseFont(FontHandle fh); 			// from the same C API
class Font { 								// RAII class
public:
    explicit Font(FontHandle fh) 			// acquire resource;
    : f(fh) {}  							// use pass-by-value, because the C API does
    FontHandle get() const { return f; }	// explicit conversion function
    ~Font() { releaseFont(f); } 			// release resource
private:
    FontHandle f; 							// the raw font resource
};
~~~

使用：

~~~C++
void changeFontSize(FontHandle f, int newSize); // from the C API
Font f(getFont());
int newFontSize;
...
changeFontSize(f.get(), newFontSize); // explicitly convert Font to FontHandle
~~~

如果client 持续多次的调用，可以使用更加方便的隐式格式：

~~~c++
class Font {
public:
    ...
    operator FontHandle() const { return f;}// implicit conversion function
    ...
};
~~~

使用方式为：

~~~C++
Font f(getFont());
int newFontSize;
...
changeFontSize(f, newFontSize); // implicitly convert Font to FontHandle
~~~

这样会带来一个弊端，会出现无意间将裸指针赋值给其他的变量。

~~~C++
Font f1(getFont());
...
FontHandle f2 = f1; // oops! meant to copy a Font object, but instead implicitly
                    // converted f1 into its underlying FontHandle, then copied that
~~~

以上代码的 f1 是获取资源的裸指针，这样f2就获得了访问裸指针的机会，如果f1 销毁后，资源不可用，那f2 就成为了野指针。 

总结：

* std空间中的智能指针，都提供了显示获取裸指针的get 方法
* 使用 （*或者->）隐式的使用资源管理类中的资源更加方便。自定义的资源管理类需要对 * 和-> 进行重载。



### Item 16: Use the same form in corresponding uses of new and delete.

经验16：使用相匹配的new 和delete

摘要：成对的使用 new 和 delete，且对应的形式要匹配，即 new - delete 和 new [] - delete []。

#### 1，new [] 与delete 的不匹配错误

在堆上使用 new [] 创建数组，但删除时，仅仅使用delete 而没有中括号 []。

~~~C++
std::string *stringArray = new std::string[100];
delete stringArray;
~~~

以上造成的结果是，仅数组的第一个string 对象被删除，而其他99 个都没有释放掉。

编译器在销毁对象的时候，会自动的调用delete，但关键的问题是，**需要调用多少个的 delete，这需要告诉编译器**。对于编译器而言，单纯的一个delete 就是删除一个对象，仅仅一个delete函数被调用。如果加上了中括号，delete [] 就相当于告诉编译器，有多个对象需要删除，会自动的调用多个的delete 释放对象。

常见的错误形式有：

* new 和delete []， 触发未知的错误

  即创建一个对象，却调用多个 delete 释放资源，会导致未知的内存释放。

* new []  和 delete， 触发未知的错误

  已经解释。即多个资源为释放，资源泄露。

#### 2，delete 删除规则

~~~C++
std::string *stringPtr1 = new std::string;
std::string *stringPtr2 = new std::string[100];
delete stringPtr1; // delete an object
delete [] stringPtr2; // delete an array of objects
~~~

谨记：当使用new 时，使用delete 释放资源，使用new []，使用delete [] 释放资源。

* typedef 一种隐秘的形式

~~~C++
typedef std::string AddressLines[4];	// a person's address has 4 lines, each of which is a string
std::string *pal = new AddressLines;	// note that "new AddressLines" returns a string*, just like
                                        // "new string[4]" would
delete pal; 							// undefined!
delete [] pal; 							// fine
~~~

以上code表明，需要根据最初的定义方式，对new 和delete 进行配对。

总结：

new 和delete， new[] 和delete[] 配套使用。



### Item 17: Store newed objects in smart pointers in standalone statements.

经验17：使用单独的语句存储新建的对象到智能指针中

摘要：为了防止编译器对传入参数不同的调用顺序而产生错误，推荐在单独的语句中获取资源并将其放入到资源管理类中。

#### 1，因编译器调用传入参数的不同顺序而产生的错误

有如下函数定义：

~~~C++
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
~~~

使用方式：

~~~C++
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
~~~

编译器调用函数processWidget 前，有三件事情(动作)需要提前完成：

* Call priority.
* Execute "new Widget".
* Call the tr1::shared_ptr constructor

对于C++ 的编译器完成以上三个动作的顺序有很大的自由度。以上动作的顺序保证了编译器顺利的编译。但如果首先得调用  “tr1::shared_ptr constructor”，再调用“new Widget”，此时 对象Widget 还没有构建完成，shared_ptr 的构造函数就会出现编译错误。以为调用产生异常，而造成内存泄露的情况，

* Execute "new Widget ".

* Call priority.

* Call the tr1::shared_ptr constructor

如果按照以上的顺序进行，当“Call priority” 出现异常，就会造成内存的泄露，仅把“Widget ” 对象 new出来，但是没有办法进行delete。

#### 2，单独的语句对资源进行存储

使用一个单独的语句创建资源，并把它放入到资源管理类中。

~~~C++
std::tr1::shared_ptr<Widget> pw(new Widget); 	// store newed object in a smart pointer in a
												// standalone statement
processWidget(pw, priority()); 					// this call won't leak
~~~

这样，生成对象“new Widget” ，调用 shared_ptr 构造函数和调用 priority() 是在不同的语句中进行，编译器不会将其进行混合插入，所以即使“call priority ” 发生异常，也不会造成内存的泄露。

总结：

使用可智能指针推荐以下方式：

~~~C++
std::tr1::shared_ptr<Widget> pw(new Widget);
~~~

更加高效的方式，使用std::make_share<>

~~~C++
auto pw = std::make_share<Widget>("hello"); // 括号中提供构造函数的参数  
~~~



## Chapter 4. Designs and Declarations

### Item 18: Make interfaces easy to use correctly and hard to use incorrectly

经验18：使接口容易使用并且难以用错

摘要：在设计接口的时候要考虑到所有用户可能使用错误的情形，并加以规避。理想情况下，如果使用一个接口没有做到客户希望做到的，代码应该不能通过编译；如果代码通过了编译，那么它就能做到客户想要的。

#### 1，使用新类型防止用户错误

定义类和类成员函数：

~~~C++
class Date {
public:
    Date(int month, int day, int year);
    ...
};
~~~

用户使用以下方式调用，产生异常行为时，而不知所措；

~~~C++
Date d(30, 3, 1995); // Oops! Should be "3, 30", not "30, 3"
Date d(2, 20, 1995); // Oops! Should be "3, 30", not "2, 20"
~~~

以上的代码至少有两处的明显错误：

* 用户可能将年、月、日的顺序搞错
* 年、月、日的正确范围没有进行限定

**改进措施：**

* 为可能出错的参数，定义新的类型；使其相互能区别。

~~~C++
struct Day {  
    explicit Day(int d):val(d) {} 
    int val;  
};
struct Month{
    explicit Month(int m):val(m) {};
    int val;
};
struct Year {
    explicit Year(int y):val(y){};
    int val;
};

class Date {
public:
Date(const Month& m, const Day& d, const Year& y);
    ...
};
~~~

用户在使用的过程中，如果输入的类型不匹配，编译器报错

~~~~C++
Date d(30, 3, 1995); 					// error! wrong types
Date d(Day(30), Month(3), Year(1995)); 	// error! wrong types
Date d(Month(3), Day(30), Year(1995)); 	// okay, types are correct
~~~~

* 预定义正确的参数范围

~~~C++
class Month {
public:
    static Month Jan() { return Month(1); }// functions returning all valid
    static Month Feb() { return Month(2); }// Month values; see below for
    ... // why these are functions, not
    static Month Dec() { return Month(12); } // objects
    ... // other member functions
private:
    explicit Month(int m); // prevent creation of new  Month values
    ... // month-specific data
};

Date d(Month::Mar(), Day(30), Year(1995));
~~~

以上的解决措施，将运行时的传参错误，提前到编译期的名称检查（即检查类型是否匹配等），将错误检查提前到编译期。解决参数顺序和范围的误用。

使用静态函数为了方式初始化先后问题，导致的防止非局部静态变量初始化错误去。

#### 2，为类型提供一致的接口

对于内建数据类型，对于乘法结果的赋值操作是不允许的，如

~~~C++
if (a * b = c) ... // oops, meant to do a comparison!
~~~

可以使用const 对操作符 * 的返回类型进行限定，

~~~C++
const inverst opertor*()
~~~

核心的观点是：

如果没有充足的理由，自定义的类型要和内建数据类型保持一致，因为用户已经习惯了内建类型的操作，如果不一致很容易使用错误。

#### 3，使用资源管理类管理资源

使工厂函数返回智能指针类型，可以很好的避免创建资源，自动释放资源的责任，同时避免在创建资源时候发生异常，导致的资源泄露。

~~~C++
std::tr1::shared_ptr<Investment> createInvestment();
~~~

为了防止用户忘记使用资源管理类 shared_ptr，工厂函数可以返回shared_ptr 类型，这样迫使用户将此变量单独的保存。

* 自定义delete 函数

~~~C++
std::tr1::shared_ptr<Investment> 		// create a null shared_ptr with
    pInv(static_cast<Investment*>(0), 	// getRidOfInvestment as its
    getRidOfInvestment); 				// deleter; see Item 27 for info on static_cast
~~~

总结：

尽最大努力保持接口易用且符合用户习惯，采用方法防止用户调用时产生错误。常用方法包括：

* 自定义类型和内建类型的接口名称和行为应尽量保持一致。
* 为易混淆类型创建不同的包装类型，且限制对该类型的操作行为，限制对象取值，自动管理堆上资源。
* 推荐使用 std::shared_ptr。



### Item 19: Treat class design as type design

经验19：像设计类型一样设计类

摘要：在设计类的时候，就像设计一个类型。

在设计新类型的时，必须将以下问题考虑清楚：

1）新类型的对象怎样的构建和销毁

这涉及到构造函数和析构函数，以及内存的分配和析构。注意成对的使用 new 和delete ，new [] 和delete[].

2）对象的初始化和赋值怎样的区分

涉及构造和赋值运算符的不同。

3）新类型对象的按值传递意味着什么？

新类型对象的赋值构造函数定义了按值传递的具体实现。

4）对新类型参数的合法值，有怎样的限制？

一般，仅有符合类型是有效的。

5）新的类型是否能进行多继承

如果新类型作为派生类，其设计必然受到基类的限制。如果作为基类，被其他派生类继承，注意其析构函数必须为虚函数。

6）对于新类型，怎样的类型转换时允许的。

考虑清楚是否允许隐式的类型转换，否则加入explict 关键字。

7）怎样的操作符和函数对于新类型是合理的

成员函数和非成员函数的区别

8）怎样的函数是不允许的

将其设置为私有的成员函数

9）谁能访问私有成员变量

决定哪些成员是公有的哪些是私有的，并且考虑友元函数是否合适。

10）新类型提供了哪些的保证

比如异常安全的，高新能的，执行效率高的，这些都会对新类型的设计产生影响。

11）新类型有多泛化

如果需要定义一个类型簇，这时需要考虑模板。

12）新类型是否是你真正需要的

如果仅仅是向已经存在的类中加入介个函数，可以使用更加简单的非成员函数或是模板进行实现。



### Item 20: Prefer pass-by-reference-to-const to pass-by-value

经验20：尽量的使用传递常量引用而不是按值传递

摘要：按引用传递要比按值传递高效。而为了防止对源数据的无意修改，最好加上const修饰。

#### 1，默认传值的代价

默认情况，C++将对象按传值的方式传入函数或传出函数，即函数以传入参数的拷贝进行初始化，调用函数获得传出对象的拷贝。往往这种机制是非常耗时的。

~~~~C++
class Person {
public:
    Person(); // parameters omitted for simplicity
    virtual ~Person(); // see Item 7 for why this is virtual
    ...
private:
    std::string name;
    std::string address;
};

class Student: public Person {
public:
    Student(); // parameters again omitted
    ~Student();
    ...
private:
    std::string schoolName;
    std::string schoolAddress;
};
~~~~

考虑使用以下的方式调用：

~~~C++
bool validateStudent(Student s); 			// function taking a Student by value
Student plato; 								// Plato studied under Socrates
bool platoIsOK = validateStudent(plato); 	// call the function
~~~

首先是派生类对象 plato 的创建和销毁，然后是其内部两个string成员变量的创建和销毁，其次是基类对象和其两个string成员变量的创建和销毁。以上函数调用过程，分别发生了6次的对象创建和销毁动作，有些是没有必要的动作。

**解决的方式是，按引用进行传递**

~~~C++
bool validateStudent(const Student& s);
~~~

如果并不希望改变原始的值，需要加上const 限制：

~~~C++
bool validateStudent(const Student& s) const;
~~~

#### 2，切片问题 slicing problem

当派生类对象以基类值进行传递时，就发生了切片问题，因为此时基类的拷贝构造函数被调用，而不是派生类的拷贝构造。结果派生类的特性被切割，仅仅得到一个基类的对象。

~~~C++
class Window {
public:
    std::string name() const; // return name of window
    virtual void display() const; // draw window and contents
};

class WindowWithScrollBars: public Window {
public:
    virtual void display() const;
};
~~~

使用方式：

~~~C++
void printNameAndDisplay(Window w){ // incorrect! parameter may be sliced!
    std::cout << w.name();
    w.display();
}
WindowWithScrollBars wwsb;
printNameAndDisplay(wwsb);
~~~

不管传给printNameAndDisplay 函数的类型是怎样的，其初始化的类型都是基类 Window 类型，所调用的函数都是基类成员函数。相应的派生类特性已经被切割。

解决方式：

~~~C++
void printNameAndDisplay(const Window& w){ // fine, parameter won't be sliced 
    std::cout << w.name();
    w.display();
}
~~~

按引用传递时，会根据传入对象的类型构造对象。即引用和指针类型会根据传入对象的类型进行构造，而传值会根基定义的类型进行构造。

总结:

1）按引用传递，减少了不必要的对象拷贝。推荐加上const限制。

2）引用传递，防止了子类以基类类型传递时的切片问题。

3）对于内建数据类型，STL 迭代器和函数对象类型，需要按值传递。



### Item 21: Don't try to return a reference when you must return an object

经验：当必须要返回一个局部对象时，千万不要返回它的引用

摘要：函数执行完时，其内部的局部变量会销毁，如果返回局部变量的引用，就相当于传递了不存在的对象。

#### 1，返回局部对象引用的错误

~~~C++
class Rational {
public:
    Rational(int numerator = 0, 	// see Item 24 for why this
    int denominator = 1); 			// ctor isn't declared explicit
    ...
private:
    int n, d; 						// numerator and denominator
    friend const Rational operator*(const Rational& lhs, const Rational& rhs); 
    // see Item 3 for why the return type is const
};
~~~

新建对象，有两种的方式，在栈上或是在堆上。在栈上创建对象，即创建临时的局部变量。

* 在栈上的临时变量

~~~C++
const Rational& operator*(const Rational& lhs, const Rational& rhs) { 
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
    return result; // warning! bad code!
}
~~~

以上代码最严重的错误是，返回了局部变量result 的引用。当函数执行完毕后，局部变量result就销毁了，返回的引用也失效了。

* 在堆上建立资源，无法销毁

~~~C++
const Rational& operator*(const Rational& lhs, const Rational& rhs) { 
    Rational *result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return *result; // warning! more bad code!
}
~~~

以上代码的错误是，返回了在堆上的对象后，没有相应的操作去delete，造成了内存的泄露。

#### 2，按值返回新创建对象

函数按值返回创建对象，编译器默认建立副本。

~~~C++
inline const Rational operator*(const Rational& lhs, const Rational& rhs) {
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
~~~

以上代码虽然的没有避免，临时对象的创建和销毁，但是避免了错误使用引用返回对象的行为。在远期看，构建和销毁的代价是微乎其微的。

总结：

在函数返回引用和返回值之间，要根据具体的上下情形，正确的判断函数的行为。



### Item 22: Declare data members private

经验：将数据成员私有化

摘要：数据私有化，有利于数据的封装。

#### 1，私有化后，精准的控制访问权限

如果将数据设置为公有的，任何client 都对其有可读可写的权限。但设置为私有，并且使用公有函数进行访问时，就可以控制不同的访问权限，如下：

~~~C++
class AccessLevels {
public:
    int getReadOnly() const { return readOnly; }
    void setReadWrite(int value) { readWrite = value; }
    int getReadWrite() const { return readWrite; }
    void setWriteOnly(int value) { writeOnly = value; }
private:
    int noAccess; 	// no access to this int
    int readOnly; 	// read-only access to this int
    int readWrite;	// read-write access to this int
    int writeOnly; 	// write-only access to this int
};
~~~

#### 2，对数据的封装

以下的类用于收集每辆汽车的速度，关于函数averageSoFar 有两种实现方式。

~~~C++
class SpeedDataCollection {
public:
    void addValue(int speed); 		// add a new data value
    double averageSoFar() const; 	// return average speed
};
~~~

第一种是：维护一个平均速度的私有变量，averageSoFar 函数调用时候，直接返回该变量。

第二种是：每次的计算平均速度。两种方式各有优缺点，需要根据运行机器是内存敏感还是速度敏感进行选择。

总结：

通过共有函数，实现对内部数据成员的访问。

* 可以内部进行对函数的修改，而不影响client的使用
* 将数据隐藏在函数接口，可以提供灵活的实现方式。
* 可以保证数据是可维护的，因为client 不可以改变数据。

如果将数据设置plublic， 当该数据发生改变时，所有用到该公有数据的派生类，或是客户端都需要改变。



### Item 23: Prefer non-member non-friend functions to member functions

经验：选择非成员、非友元函数而不是成员函数

摘要：类的成员的私有化，是为了更好的封装，而成员函数和友元函数在一定程度上是破坏了类的封装特性。

#### 1，成员函数和非成员函数

在类中需要依次的执行三个成员函数，需要实现一个便捷函数，那此函数是以成员函数实现，还是以非成员函数实现？

~~~C++
class WebBrowser {
public:
    void clearCache();
    void clearHistory();
    void removeCookies();
};
~~~

成员函数实现：

~~~C++
class WebBrowser {
public:
    void clearEverything(); // calls clearCache, clearHistory, and removeCookies
};
~~~

非成员函数实现：

~~~C++
void clearBrowser(WebBrowser& wb){
    wb.clearCache();
    wb.clearHistory();
    wb.removeCookies();
}
~~~

在类的封装角度，非成员函数是比较好的实现：

* 非成员函数提供了更好的封装特性
* 非成员函数提供了更好的对类相关函数的集成特性，进而更少的编译依赖，和灵活的扩展性。

#### 2， 同名称空间中非成员函数

~~~C++
namespace WebBrowserStuff {
    class WebBrowser {... };
    void clearBrowser(WebBrowser& wb);
}
~~~

也可以将不同类型的便捷函数，放到同一个名称空间不同的头文件中。

~~~~C++
// header "webbrowser.h" — header for class WebBrowser itself
// as well as "core" WebBrowser-related functionality
namespace WebBrowserStuff {
    class WebBrowser {... };
    ... // "core" related functionality, e.g.
    	// non-member functions almost all clients need
}

// header "webbrowserbookmarks.h"
namespace WebBrowserStuff {
    ... // bookmark-related convenience
} // functions

// header "webbrowsercookies.h"
namespace WebBrowserStuff {
		... // cookie-related convenience
} // functions
~~~~

一个类可以有多个的便捷函数，如果在使用过程中将不需要的函数也进行了编译加载，肯定是不合适的。将一类的便捷函数放到同一个名称空间不同的头文件中，按需进行加载是行之有效的方式。

总结：

1）非成员非友元函数有利于类的封装。

2）同名称空间，不同头文件对需要的便捷函数按需加载。



### Item 24: Declare non-member functions when type conversions should apply to all parameters

经验：当类型转化需要应用到所有的参数时，声明非成员函数

摘要：非成员函数可以对所有的参数进行类型转化。

#### 1，成员函数operator * 的问题

定义有理数类如下：

~~~C++
class Rational {
public:
    Rational(int numerator = 0, // ctor is deliberately not explicit;
    int denominator = 1);// allows implicit int-to-Rational
    // conversions
    int numerator() const; // accessors for numerator and
    int denominator() const; // denominator — see Item 22
    private:
};
~~~

定义类成员函数，实现乘法法则

~~~C++
class Rational {
public:
    const Rational operator*(const Rational& rhs) const;
};
~~~

**问题分析：**

~~~C++
Rational oneEighth(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf * oneEighth; // fine
result = result * oneEighth; // fine
result = oneHalf * 2; // fine
result = 2 * oneHalf; // error!
~~~

可见对于 2 * oneHalf 不能编译通过的原因是：

~~~C++
result = oneHalf.operator*(2); // fine
result = 2.operator*(oneHalf); // error!
~~~

* 对于2 是一个int 类型，不是Rational 类，所以不能调用类的成员函数 operator*
* 在类外也没有找到合适的  operator* 函数，所以编译失败。

#### 2，解决：允许所有参数进行类型转换

* 非成员函数 

解决问题的关键是，需要让传入参数都在函数参数列表中，进而可以进行隐式类型转换。

方法：定义类的非成员函数

~~~C++
class Rational {
	... // contains no operator*
};
const Rational operator*(const Rational& lhs, // now a non-function
						 const Rational& rhs) {
    return Rational(lhs.numerator() * rhs.numerator(),
                    lhs.denominator() * rhs.denominator());
}
~~~

再次使用：

~~~C++
Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2; // fine
result = 2 * oneFourth; // hooray, it works!
~~~

在调用非成员函数时，有两个参数传入 （int 型 2 和 Rational 型 oneFourth）。 当编译器遇到 int 型时，可以通过调用 Rational  的构造函数，将其转换为 Rational  类型，进而成功的编译。

**注意：**

非成员函数 operator*，使用类的公有接口完成了对私有成员的访问，所以不需要声明为 friend。尽量的不要使用friend 声明函数，破坏类的封装特性。

* 允许隐式转换 

~~~C++
Rational(int numerator = 0, // ctor is deliberately not explicit;
int denominator = 1);// allows implicit int-to-Rational
~~~

对于类的成员函数实现的 operator*，当result = oneFourth * 2 可以编译通过，是由于隐式类型转换是允许的，即 int 2 被隐式的转换成临时Rational  类型，才能正确的调用 operator*(const Rational& rhs)

同样，对于非成员函数，result = 2*oneFourth ， 虽然2 已经在非成员函数的列表中，int 类型 2 也需要隐式类型转换成Rational 类，才能成功的编译。没有explict 是混合乘法编译通过的关键。

总结：

* 如果对所有参数进行类型转换的时，需要非成员函数，将所有参数放入到函数列表中。
* 允许类进行隐式类型转换



### Item 25: Consider support for a non-throwing swap

经验：考虑支持不抛异常样的swap 函数

摘要：当常规swap函数不满足需要时，对其进行完全实例化改写。

#### 1，常规的swap 函数

~~~~C++
namespace std {
template<typename T> 	// typical implementation of std::swap;
void swap(T& a, T& b) { // swaps a's and b's values
    T temp(a);
    a = b;
    b = temp;
	}
}
~~~~

* 代码在进行交换的时候，产生了三次的拷贝。但是对于一些的类，拷贝是不必要的，多余的拷贝造成了资源的浪费，比如句柄类。
* 对于自定义的类型，拷贝的行为取决于拷贝复制和赋值构造函数。

句柄类的定义：

~~~C++
class WidgetImpl { // class for Widget data;
public: 
	... // details are unimportant
private:
    int a, b, c; 			// possibly lots of data —
    std::vector<double> v; 	// expensive to copy!
};

class Widget { // class using the pimpl idiom
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs) { // to copy a Widget, copy its
        // WidgetImpl object. For details on implementing
        *pImpl = *(rhs.pImpl); // operator= in general,
        ... // see Items 10, 11, and 12.
    }
	...
private:
	WidgetImpl *pImpl; // ptr to object with this  Widget's data
};
~~~

关键点在于 **Widget 的赋值操作  operator= 进行了深度拷贝。当使用swap 进行交换两个句柄类时，也会进行深拷贝**，但是期望是仅仅进行私有成员变量 pImpl 的交换。

#### 2，高效的swap

以上的问题在于，标准的swap 并不知道对于 Widget 只需要交换 pImpl 指针即可。接下来问题是如何进行显式告知问题了。通过定义一个完全具体化的模板函数完成。完全具体化得函数的调用先于一般模板函数。

注意：一般不允许改变std空间中的swap 函数，但是可以使用自定义类型对其进行完全的实例化。避免更改同时进行了使用。

~~~C++
namespace std {
template<> // this is a specialized version
void swap<Widget>(Widget& a, Widget& b) { // of std::swap for when T is
										// Widget; this won't compile
	swap(a.pImpl, b.pImpl); // to swap Widgets, just swap their pImpl pointers
	} 
}
~~~

以上代码不会编译通过，应为函数直接使用了 Widget 类的私有成员变量。如何避免那？通过定义非成员成员函数，调用成员函数的方式，而成员函数实际的完成swap。如下：

~~~C++
class Widget { // same as above, except for the
public: // addition of the swap mem func
    void swap(Widget& other) {
        using std::swap; // the need for this declaration
        // is explained later in this Item
        swap(pImpl, other.pImpl); // to swap Widgets, swap their
    } // pImpl pointers
};

namespace std {
template<> // revised specialization of
void swap<Widget>(Widget& a, Widget& b){  // std::swap
    a.swap(b); // to swap Widgets, call their swap member function
	} 
}
~~~

当使用 std空间中的swap 函数时，会优先的调用自定义的 swap 版本，这样在调用类成员函数的 swap 成功实现了句柄类的浅拷贝。

当句柄类都是模板类时，又是另种情况，不做讨论。

总结：

为了避免默认的swap 函数进行不必要的拷贝，需要使用外部函数调用类成员函数完成浅拷贝的操作。



## Chapter 5. Implementations

### Item 26: Postpone variable definitions as long as possible.

经验：尽可能的推迟变量的定义

摘要：直到真正使用变量时才定义变量，且使用确切的值进行“合适”的初始化，从而避免不必要对象的创建和销毁。

#### 1，使用已知的初始值创建对象

一个不恰当定义变量的例子：

~~~C++
std::string encryptPassword(const std::string& password) {
    using namespace std;
    string encrypted;
    if (password.length() < MinimumPasswordLength) {
        throw logic_error("Password is too short");
    }
    ... // do whatever is necessary to place an
    	// encrypted version of password in encrypted
    return encrypted;
}
~~~

以上的例子中，当抛出异常的时候（throw logic_error），变量encrypted 已经创建，且当程序离开encryptPassword 空间时候，又会销毁变量，所以encrypted 变量的定义过早了。

**改进版本：**

~~~C++
std::string encryptPassword(const std::string& password){
    using namespace std;
    if (password.length() < MinimumPasswordLength) {
    	throw logic_error("Password is too short");
    }
    string encrypted;
    ... // do whatever is necessary to place an
    	// encrypted version of password in encrypted
    return encrypted;
}
~~~

以上代码，虽然抛出异常，也不会引起变量encrypted 的创建和销毁，但是还不够紧凑。因为，变量encrypted 并没有避免使用默认的构造函数，依然使用赋值的方式进行初始化! 赋值初始化比默认的拷贝构造函数是低效的，应尽量的使用拷贝构造函数进行变量的创建。

**推荐使用版本如下：**

~~~C++
void encrypt(std::string& s); // encrypts s in place
std::string encryptPassword(const std::string& password){
    ... // check length
    std::string encrypted(password); // define and initialize
    // via copy constructor
    encrypt(encrypted);
    return encrypted;
}
~~~

总结：不仅推迟变量的定义到正确使用时候，而且要使用确切的值对要创建的变量进行初始化！

#### 2，循环中的初始化问题

对于循环该如何进行初始化那？

~~~C++
// Approach A: define outside loop 
Widget w;
for (int i = 0; i < n; ++i){
	w = some value dependent on i; 
	...
}

// Approach B: define inside loop
for (int i = 0; i < n; ++i) {
    Widget w(some value dependent on i);
    ...
 }
~~~

* Approach A : 1 constructor + 1 destructor + n assignments.
*  Approach B : n constructors + n destructors.

除非明确的知道，赋值语句比构造和销毁语句的消耗低！否则推荐使用方法B。

总结：

1）推迟定义变量，直到已知其初始值。使用初始值进行一次的拷贝构造，避免创建后再赋值的行为。

2）对于循环，推荐在循环内进行创建和析构。



### Item 27: Minimize casting.

经验：最小化类型转换

摘要：

#### 1，四种新式类型转换

* 去除常量性转换 

~~~C++
const_cast<T>(expression)
~~~

C++ 中唯一能去除常量性的映射

* 动态类型转换

~~~C++
dynamic_cast<T>(expression)
~~~

主要用于继承关系中。"safe downcasting"

* 重映射

~~~C++
reinterpret_cast<T>(expression)
~~~

底层的映射，比较耗时。比如讲指针转换为int 类型

* 静态映射

~~~C++
static_cast<T>(expression)
~~~

进行强制的类型转换，比如non-const 类型，转换为const类型，由void* 转换为 类型指针。基类指针转换为派生类指针等。

使用例子：调用显示的构造函数将对象传递给函数，即将整形的 int 转换为自定义类Widget

~~~~C++
class Widget {
public:
explicit Widget(int size);
...
};
void doSomeWork(const Widget& w);
doSomeWork(Widget(15)); 
// create Widget from int with function-style cast
doSomeWork(static_cast<Widget>(15)); 
// create Widget from int with C++-style cast
~~~~

总结：

1）在性能敏感的程序中，尽量的不要使用类型的转换，因为转换总是要花费一定的时间。

2）如果要使用类型转换，使用显示的类型转换，方便查找也明确语义。



### Item 28: Avoid returning "handles" to object internals.

经验：避免返回内部对象的句柄

摘要：返回内部变量的指针是危险的，尽量不要这样做。

#### 1，内部数据被返回的引用修改

**返回内部变量指针的例子**

~~~C++
// class for representing points
class Point {
public:
    Point(int x, int y);
    ...
    void setX(int newVal);
    void setY(int newVal);
    ...
};

struct RectData { 	// Point data for a Rectangle
    Point ulhc; 	// ulhc = " upper left-hand corner"
    Point lrhc; 	// lrhc = " lower right-hand corner"
};

class Rectangle {
...
private:
    std::tr1::shared_ptr<RectData> pData;// see Item 13 for info on tr1::shared_ptr
}; 
~~~

**trick:** 为了保证对象 Rectangle 对象尽可能的小，将需要的数据保存在外部的变量中。另外：对于自定义的数据类型（Point），传递指针比传值更加的高效，所以定义了如下的函数：

~~~C++
class Rectangle {
public:
    ...
    Point& upperLeft() const { return pData->ulhc; }
    Point& lowerRight() const { return pData->lrhc; }
    ...
};
~~~

以上代码会造成Point 对象修改！有两点需要注意。

* 两个公有函数upperLeft 和 lowerRight 被设置为const，本意是调用的client 只能读取，不能修改。
* 但是其返回了私有成员变量的引用，造成了client 可以随意的修改Point 的数据

~~~C++
Point coord1(0, 0);
Point coord2(100, 100);
const Rectangle rec(coord1, coord2); // rec is a const rectangle from
// (0, 0) to (100, 100)
rec.upperLeft().setX(50); // now rec goes from
// (50, 0) to (100, 100)!
~~~

#### 2，为返回引用加上const 限制

解决方式为：给返回的引用机上const 限制。

~~~C++
class Rectangle {
public:
    ...
    const Point& upperLeft() const { return pData->ulhc; }
    const Point& lowerRight() const { return pData->lrhc; }
    ...
};
~~~

尽管有const 的加持，返回内部变量的指针，会造成“野指针”的问题。比如以下的例子

~~~C++
class GUIObject {... };
// value; see Item 3 for why return type is const
const Rectangle boundingBox(const GUIObject& obj); 	

GUIObject *pgo; // make pgo point to some GUIObject
...
// get a ptr to the upperleft point of its
const Point *pUpperLeft = &(boundingBox(*pgo).upperLeft()); 
// bounding box
~~~

当调用boundingBox 函数时，会产生临时的变量tmp，且把临时变量的upperLeft 变量的地址，赋值给了pUpperLeft 变量，当离开该语句时，tmp 变量被销毁，此时的pUpperLeft 就成为了“野指针”。

总结：

**尽量的避免返回内部变量的句柄！**



### Item29: Strive for exception-safe code.

经验：努力的书写异常安全的代码

摘要：使用拷贝复制策略，防止异常发生。

#### 1，异常分析

对于一段代码而言，只有两种结果：线程安全或不安全。比如以下的背景更换函数。

~~~C++
class PrettyMenu {
public: // change background image
    void changeBackground(std::istream& imgSrc);
private:
    Mutex mutex; 		// mutex for this object
    Image *bgImage; 	// current background image
    int imageChanges; 	// of times image has been changed
};
~~~

对于更换背景图片有如下的实现：

~~~C++
void PrettyMenu::changeBackground(std::istream& imgSrc){
    lock(&mutex); 		// acquire mutex (as in Item 14)
    delete bgImage; 	// get rid of old background
    ++imageChanges; 	// update image change count
    bgImage = new Image(imgSrc); // install new background
    unlock(&mutex); 	// release mutex
}
~~~

如果 new 资源发生异常，会出现以下的问题，

* 原始资源 bgImage 被删除，内容被更改。

* imageChanges 计数失真。 

* 资源锁没有被打开。 

一个异常安全的程序，我们期望在异常发生时保证以下两点：

* 无资源泄露，正常的释放资源。

* 不会使数据结构崩溃。

对于资源泄露，可以使用资源管理器RAII 方式管理资源。对于防止数据崩溃，可以使用以下方式

#### 2，异常时，防止数据崩溃

方式一：

1）使用智能指针管理图片资源

2）通过智能指针的reset函数实现new 资源成功后，再替换资源。

3）操作成功后，计数加一

~~~C++
class PrettyMenu {
	std::tr1::shared_ptr<Image> bgImage;
};

void PrettyMenu::changeBackground(std::istream& imgSrc){
    Lock ml(&mutex);
    bgImage.reset(new Image(imgSrc));	// replace bgImage's internal pointer with
    									// the result of the "new Image" expression
    ++imageChanges;
}
~~~

以上代码注意，只有new 成功后才会调用智能指针的reset函数。此种方式只是实现了基础的异常保证。

方式二：复制然后交换。copy and swap。

优点是，允许进行all-or-nothing 的操作。要不成功，要不不做任何更改。

~~~C++
struct PMImpl { // PMImpl = "PrettyMenu Impl.";
    std::tr1::shared_ptr<Image> bgImage; //  see below for why it's a struct
    int imageChanges; 
};

class PrettyMenu {
private:
    Mutex mutex;
    std::tr1::shared_ptr<PMImpl> pImpl;
};
~~~

具体的实现函数：

~~~C++
void PrettyMenu::changeBackground(std::istream& imgSrc){
    using std::swap; // see Item 25
    Lock ml(&mutex); // acquire the mutex
    std::tr1::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl)); // copy obj. data
    pNew->bgImage.reset(new Image(imgSrc)); // modify the copy
    ++pNew->imageChanges;
    swap(pImpl, pNew); // swap the new data into place 
}	// release the mutex
~~~

使用结构体的方式实现 PMImpl 而不是类，因为其已经被PrettyMenu 私有化，如果是类会有嵌套的问题。

以上注意实现智能指针拷贝原始数据的方式

~~~C++
std::tr1::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl));
~~~

#### 3，异常安全分类

* 基本的异常安全保证

异常发生时，没有数据崩溃。即变量值没有改变，或是变成了默认的数值。

* 强异常安全保证

异常发生时，要成功就完全的成功，符合预期的结果。要不就完全的没有变化，和没有调用之前的状态一致。

* 不抛出异常。

~~~C++
int doSomething() throw(); // note empty exception spec.
~~~

不是函数不抛出异常，而是异常发生时，调用特定的异常处理函数。

总结：

1）基本的异常安全的函数保证没有资源泄露，不允许变量崩溃。

2）使用拷贝和交换的策略，实现强异常安全保证。



### Item 30: Understand the ins and outs of inlining.

经验：理解内联和非内联函数

摘要：将短小，使用频率高的函数作为内联。

#### 1，内联函数两种表示方式

使用内联函数，可以触发编译器对代码的整体优化“context-specific optimizations”，其本质是，使用函数体代替函数调用行。

内联函数时对编译器的请求，而不是命令。内联函数有显式和隐式两种方式：

**隐式方式为：**

~~~C++
class Person {
public:
    int age() const { return theAge; }	// an implicit inline request: age is
    ... 								// defined in a class definition
private:
    int theAge;
};
~~~

**显式方式：**

~~~C++
template<typename T> // an explicit inline
inline const T& std::max(const T& a, const T& b)// request: std::max is
{ return a < b ? b : a; } // preceded by "inline"
~~~

注意：

* 内联函数必须在头文件中定义。

因为编译器需要知道，具体的函数代码在哪里。

* 模板的定义也通常是在头文件中。

因为编译器需要知道具体的模板是怎样的，才能进行具体的实例化。

#### **2，编译器拒绝内联的情况：**

* 编译器拒绝将过于复杂的函数进行内联：
* 函数指针不会调用内联函数

~~~C++
inline void f() {...}//assume compilers are willing to inline calls to f
void (*pf)() = f; // pf points to f
...
f();// this call will be inlined, because it's a "normal" call
pf(); // this call probably won't be, because it's through
// a function pointer
~~~

* 类的构造和析构函数不适宜内联

因为，构造和析构函数，调用了许多的关联函数。

#### **3，内联对于库设计的影响。**

如果将函数 f() 设计成内联函数，当f() 改变时，所有使用函数 f 的客户端都必须的重新编译。

如果将函数 f() 设计成非内联函数，当f() 改变时，所有使用函数 f 的客户端只需要进行重新连接即可。

#### **4，内联函数对debug 的影响。**

内联函数无法的进行debug，所以在开始时候尽量的不要使用内联。集中找到影响程序的片段（80-20%）后，注意使用inline。

总结：

尽量的将短小，使用频率高的函数作为内联。可以进行debug 的同时，最小化代码膨胀和最大化代码速度。



### Item31: Minimize compilation dependencies between files.

经验：最小化文件之间编译依赖

摘要：将定义和声明分开，并包含声明定义变量。使用句柄类和接口类减少文件依赖。

#### 1，级联编译依赖

（cascading compilation dependencies）

~~~C++
class Person {
public:
Person(const std::string& name, const Date& birthday,const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::string theName; 	// implementation detail
    Date theBirthDate; 		// implementation detail
    Address theAddress; 	// implementation detail
};
~~~

如果正常的编译Person 需要知道相应类型的定义，需要包含特定的头文件

~~~C++
#include <string>
#include "date.h"
#include "address.h"
~~~

其主要的目的是，需要确切的知道在Person中相应 string ，date 和 address 类的大小。

最小化编译依赖的本质是：头文件编译时是自满足的，如果不能，那依赖声明而不是定义。

* 避免使用对象，当引用和指针可以使用时。
* 依赖于类的声明而不是类的定义。
* 将声明和定义分离成两个文件

#### 2，句柄类

句柄类主要的作用是，减少文件之间的依赖。

~~~~C++
#include "Person.h" 
// we're implementing the Person class,
// so we must #include its class definition
#include "PersonImpl.h" 
// we must also #include PersonImpl's class
// definition, otherwise we couldn't call
// its member functions; note that
// PersonImpl has exactly the same
// member functions as Person — their
// interfaces are identical
Person::Person(const std::string& name, const Date& birthday,
				const Address& addr) : pImpl(new PersonImpl(name, birthday, addr)){}
std::string Person::name() const{
return pImpl->name();
}
~~~~

#### 3，接口类

使用虚拟类实现

~~~C++
class Person {
public:
virtual ~Person();
virtual std::string name() const = 0;
virtual std::string birthDate() const = 0;
virtual std::string address() const = 0;
...
};
~~~

具体的使用方式是：

~~~~C++
class Person {
public:
    ...
    static std::tr1::shared_ptr<Person>		// return a tr1::shared_ptr to a new
    create(const std::string& name, 		// Person initialized with the
    const Date& birthday, 					// given params; see Item 18 for
    const Address& addr);					// why a tr1::shared_ptr is returned
    ...
};
~~~~

客户端使用

~~~C++
std::string name;
Date dateOfBirth;
Address address;
...
// create an object supporting the Person interface
std::tr1::shared_ptr<Person> pp(Person::create(name, dateOfBirth, address));
...
std::cout << pp->name() // use the object via the
<< " was born on " // Person interface
<< pp->birthDate()
<< " and now lives at "
<< pp->address();
... // the object is automatically deleted when pp goes out of scope

~~~

总结：

1）将声明和实现分开，并成对改变。

2）使用句柄类和接口类，减少依赖。即仅当声明类的接口改变时，才会重新编译。



## Chapter 6. Inheritance and Object-Oriented Design

### Item 32: Make sure public inheritance models "is-a."

经验：确保公有继承模型为 “是一个” （is-a） 关系。

摘要：公有继承是“is-a” 的关系，即任何派生类都是基类类型，所有适用于基类的函数也适用于派生类，反之则不成立。派生类时基类的延伸扩展。

#### 1，公有继承“is-a” 关系

有如下的继承关系；所有的学生都是人类，但并不是所有的人类都是学生。

~~~C++
class Person {...};
class Student: public Person {...};
void eat(const Person& p); 		// anyone can eat
void study(const Student& s); 	// only students study
Person p; 						// p is a Person
Student s; 						// s is a Student
eat(p); 						// fine, p is a Person
eat(s); 						// fine, s is a Student,and a Student is-a Person
study(s); 						// fine
study(p); 						// error! p isn't a Student
~~~

#### 2，会飞的企鹅

描述：所有的鸟都会飞，企鹅也是鸟，但企鹅不会飞。以下程序实现

~~~C++
class Bird {
	... // no fly function is declared
};
class Penguin: public Bird {
	... // no fly function is declared
};
~~~

使用：

~~~C++
Penguin p;
p.fly(); // error!
~~~

以上代码运行时错误，因为企鹅不能飞，没有定义 fly() 函数。如果在Brid 类中实现了fly 函数，编译和运行期间都不报错，且 p.fly() 调用成功，这不符合逻辑。

#### 3，正方形和长方形的错误继承

几何知识告诉我们，正方形是一个长方形的特例（is-a）关系，所以正方形应该公有继承自长方形。

~~~C++
class Rectangle {
public:
    virtual void setHeight(int newHeight);
    virtual void setWidth(int newWidth);
    virtual int height() const; 		// return current values
    virtual int width() const;
...
};
void makeBigger(Rectangle& r){ 			// function to increase r's area
    int oldHeight = r.height();
    r.setWidth(r.width() + 10); 		// add 10 to r's width
    assert(r.height() == oldHeight); 	// assert that r's
} 
~~~

以上定义了长方形类，并实现了一个变大函数，即宽增加10，高度不变。

~~~C++
class Square: public Rectangle {...};
Square s;
assert(s.width() == s.height());	// this must be true for all squares by inheritance
makeBigger(s); 						// s is-a Rectangle, so we can increase its area
assert(s.width() == s.height()); 	// this must still be true for all squares
~~~

以上代码，正方形公有继承自长方形，并应用了长方形的变大函数，这种行为显然不适用于正方形，从而产生了错误。显然，在长方形类中的makeBigger 函数，不应该被正方形继承。

#### 结论：

公有继承“is-a”，即所有的派生类类型都是基类类型，这要求所有基类方法必须适用于派生类。



### Item 33: Avoid hiding inherited names

经验：避免隐藏继承的名称

摘要：在子类中重写了与基类同名不同参数的函数，需使用using使基类同名函数可见。

#### 1，名称空间中变量隐藏

~~~C++
int x; 				// global variable
void someFunc(){
    double x; 		// local variable
    std::cin >> x; 	// read a new value for local x
}
~~~

在函数作用于的局部变量x 会把全局变量的x 隐藏掉。

#### 2，继承关系中的隐藏

当自派生类中引用基类中的变量或是成员函数的时候，编译器能够找到，这是因为，派生类的作用域是在基类作用域里面的。

~~~C++
class Base {
private:
    int x;
    public:
    virtual void mf1() = 0;
    virtual void mf2();
    void mf3();
    ...
};

class Derived: public Base {
public:
    virtual void mf1();
    void mf4();
    ...
};
~~~

正常的使用方式：

~~~C++
void Derived::mf4(){
    ...
    mf2();
    ...
}
~~~

在派生类中定义mf4 的函数，其中使用到了基类的成员函数 mf2。name lookup 的步骤是现在局部作用域中寻找，其次基类作用域，最后全局作用域，停止。

假设现在进行函数的重载

~~~~C++
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
    ...
};

class Derived: public Base {
public:
    virtual void mf1();
    void mf3();			// not allowed? 不应该重写基类中的非虚函数
    void mf4();
    ...
};
~~~~

以上代码会将基类中的mf1 和mf3 隐藏掉。即派生类中不会继承mf1 和mf3。

~~~C++
Derived d;
int x;
...
d.mf1(); 	// fine, calls Derived::mf1
d.mf1(x); 	// error! Derived::mf1 hides Base::mf1
d.mf2(); 	// fine, calls Base::mf2
d.mf3(); 	// fine, calls Derived::mf3
d.mf3(x); 	// error! Derived::mf3 hides Base::mf3
~~~

#### 3，using使基类中函数可见

正确的做法是：在派生类中声明基类函数，使其全部可见

~~~c++
class Base {
private:
	int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
};
class Derived: public Base {
public:
    using Base::mf1; // make all things in Base named mf1 and mf3
    using Base::mf3; // visible (and public) in Derived's scope
    virtual void mf1();
    void mf3();
    void mf4();
};
~~~

再次使用时，即可以正常调用基类对象

~~~C++
Derived d;
int x;
d.mf1(); // still fine, still calls Derived::mf1
d.mf1(x); // now okay, calls Base::mf1
d.mf2(); // still fine, still calls Base::mf2
d.mf3(); // fine, calls Derived::mf3
d.mf3(x); // now okay, calls Base::mf3
~~~

以上说明了，当从一个带有重载函数的基类中进行继承时，如果想重写其中的函数，必须的再次在派生类中声明，才可以正常的使用。否则会将基类中重载的函数隐藏掉。

总结：

1）子类中同名不同参的函数，会隐藏基类同名函数。

2）using 声明，使基类中同名不同参函数在子类中可见。



### Item 34: Differentiate between inheritance of interface and inheritance of implementation

经验：继承接口和实现之间的不同

摘要：在继承关系中有三类继承 1）纯虚函数，只继承接口不继承实现 2）虚函数，即继承接口又继承实现 3）常规函数，只继承实现。

#### 1，继承接口和实现的不同

基类中有三类函数，如下：

~~~C++
class Shape {
public:
    virtual void draw() const = 0;
    virtual void error(const std::string& msg);
    int objectID() const;
};
~~~

Shape 包含三个成员函数，纯虚函数 deam，普通的虚函数error和非虚函数objectID。公有继承方式：

~~~C++
class Rectangle: public Shape {... };
class Ellipse: public Shape {... };
~~~

* 纯虚函数 

声明一个纯虚函数的目的是：让派生类仅仅继承基类接口。

也就是说，所有继承shape 的派生类，都应该有draw() 函数，但是具体怎样的实现，每个派生了可以自行定义。纯虚函数必须在派生类中重新的声明，且该纯虚函数在抽象类中没有定义。

* 虚函数

派生类继承了虚函数的接口，基类中的虚函数提供了默认的实现，但派生类可以重写。即虚函数的公有继承，即继承了接口又继承了实现。直白讲，派生类必须提供虚函数error 的功能，如果派生类没有提供实现，会回退调用基类的error默认函数。

#### 2，虚函数继承问题

有时虚函数即提供接口又提供默认实现，具有潜在的安全问题。

~~~C++
class Airport {... }; // represents airports
class Airplane {
public:
    virtual void fly(const Airport& destination);
    ...
};

void Airplane::fly(const Airport& destination){
	default code for flying an airplane to the given destination
}
class ModelA: public Airplane {... };
class ModelB: public Airplane {... };
~~~

以上的Airplane 类提供了虚函数 fly 的声明和定义。ModelA 和ModelB 分别的继承Airplane。两种类型的飞机，其飞行的行为是一致的，所以可以继承并使用默认飞行行为。这样做也有利于减少重复的代码。

假如有新的飞机类型 modelC，且其飞行行为不同于前两类。以下的使用方式，会产生错误的行为：

~~~C++
class ModelC: public Airplane {
	... 						// no fly function is declared
};
Airport PDX(...); 				// PDX is the airport near my home
Airplane *pa = new ModelC;
...
pa->fly(PDX); 					// calls Airplane::fly!
~~~

ModelC 并没有定义自身的飞行函数，因其继承自Airplane类，所以调用了默认的飞行行为 Airplane::fly()，这是错误的。

这里的错误问题是Airplane::fly 的默认飞行行为，并不适用于modelC。

* **一种解决方式是，切断虚函数接口和默认实现之间的关联，如果需要基类中的默认函数行为，进行显式的调用**

~~~c++
class Airplane {
public:
    virtual void fly(const Airport& destination) = 0;
    ...
protected:	// 注意默认行为类型为保护属性
    void defaultFly(const Airport& destination);
};

void Airplane::defaultFly(const Airport& destination){
	default code for flying an airplane to the given destination
}
~~~

现在Airplane::fly  被声明为一个纯虚函数，为飞行行为提供了接口。且默认的飞行行为仍在基类Airplane 中，但为保护类型。这样，modelA 和modelB 需要显示的通过内联函数调用默认飞行函数。重点是默认的飞行函数 Airplane::defaultFly() 为非虚函数，这样派生类不能对其进行重写，改变其行为。保证了所有使用默认fly 函数的基类，其行为是相同的。以下是其实现方式：

~~~C++
class ModelA: public Airplane {
public:
    virtual void fly(const Airport& destination)
    { defaultFly(destination); }	// 显式的调用默认飞行函数
    ...
};

class ModelB: public Airplane {
public:
    virtual void fly(const Airport& destination)
    { defaultFly(destination); }	// 显式的调用默认飞行函数
    ...
};
~~~

对于modelC 其绝不会在没有显示调用的情况下，使用默认的飞行函数行为Airplane::defaultFly。且基类的纯虚函数，要求其必须的重写fly 飞行函数，否则编译报错。

~~~c++
class ModelC: public Airplane {
public:
    virtual void fly(const Airport& destination);
    ...
};

void ModelC::fly(const Airport& destination){
	// code for flying a ModelC airplane to the given destination
}
~~~

这种的方式，没有将接口和实现进行分离。

* **第二种解决方式，为纯虚函数定义默认行为**

~~~C++
class Airplane {
public:
    virtual void fly(const Airport& destination) = 0;
    ...
};

void Airplane::fly(const Airport& destination){ 
    // an implementation of  a pure virtual function
	// default code for flying an airplane to the given destination
}

class ModelA: public Airplane {
public:
    virtual void fly(const Airport& destination)
    { Airplane::fly(destination); }	// 调用纯虚函数的定义函数
    ...
};

class ModelB: public Airplane {
public:
    virtual void fly(const Airport& destination)
    { Airplane::fly(destination); }	// 调用纯虚函数的定义函数
    ...
};

class ModelC: public Airplane {
    public:
    virtual void fly(const Airport& destination);
    ...
};

void ModelC::fly(const Airport& destination){
	// code for flying a ModelC airplane to the given destination
}
~~~

纯虚函数fly() 被分为了两个部分，fly() 的声明指明了派生类必须使用的接口，fly() 的定义指明了其默认行为，派生类显示的调用才可以使用，派生类也可以自定义fly 行为。

* 非虚函数

非虚函数其期望在所有的派生类对象中，行为是一致的。其并不支持做任何的改变。所以声明一个非虚函数的目的是，让派生类即继承接口又继承其实现。

#### 3，总结

在公有继承关系中：

* 纯虚函数，目的是让派生类仅仅继承接口
* 虚函数，目的是让派生了即继承接口又继承实现。
* 非虚函数，目的是让派生类，即继承接口强制使用默认实现。



### Item 35: Consider alternatives to virtual functions

经验：考虑虚函数的代替方式

摘要：使用策略代替虚函数行为

情景：设计一款游戏，游戏中的每个角色的健康值，需要根据不同的角色进行计算。

#### 1，普通虚函数版

~~~C++
class GameCharacter {
public:
    virtual int healthValue() const;	// return character's health rating;
    ...									// derived classes may redefine this
};
~~~

#### 2，虚函数包装器 NVI

即通过非虚函数接口调用虚函数。

将healthValue 定义为公有函数作为借口，并定义私有doHealthValue 函数，真正计算健康度。让公有的healthValue 调用私用的doHealthValue 函数。

~~~C++
class GameCharacter {
public:
    int healthValue() const{ 			// derived classes do not redefine
        ... // do "before" stuff — see below
        int retVal = doHealthValue();	// do the real work
        ... // do "after" stuff — see below
        return retVal;
        }
private:
    virtual int doHealthValue() const { // derived classes may redefine this
        ... // default algorithm for calculating character's health
    }
};
~~~

这种让clients 通过调用公有函数接口，间接的调用虚函数的方式称之为 non-virtual interface (NVI) idiom。将公有的非虚函数称之为，虚函数包装器。在设计模式中称之为模板模式。

NVI 的优势在于，在调用doHealthValue 函数之前可以进行准备工作，而且在之后还可以进行必要的清理退出工作。并且基类中的非虚函数不可以在派生类中重写，从而保证了所有client 调用者的行为一直。如果准备和清理工作，通过client 直接的调用虚函数，实现复杂不方便。

这种方式的特点是：共有相同的部分放在基类公有接口中实现（即虚函数前后的上下文），不同的部分通过虚函数放在子类中重写完成。

#### 3，通过函数指针实现

健康度的计算独立于角色类型，即不在是角色定义的一部分。可以要求角色的构造函数需要传递一个指向健康度计算的函数指针。如下代码：

~~~C++
class GameCharacter; // forward declaration

// function for the default health calculation algorithm
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
    : healthFunc(hcf) {}
    
    int healthValue() const { return healthFunc(*this); }
    
private:
    HealthCalcFunc healthFunc;
};
~~~

相比于在GameCharacter 类中通过虚函数方式，以上方法是比较灵活的。

* 相同类型不同实例可以有不同的健康度计算函数

~~~~C++
class EvilBadGuy: public GameCharacter {
public:
    explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc)
    : GameCharacter(hcf) {... }
};

int loseHealthQuickly(const GameCharacter&);	// health calculation
int loseHealthSlowly(const GameCharacter&);		// funcs with different behavior
EvilBadGuy ebg1(loseHealthQuickly); // same-type charac-ters
EvilBadGuy ebg2(loseHealthSlowly); //  with different health-related behavior
~~~~

#### 4，通过tr1::function 函数

可调用对象，有以下几类：

- 一个函数指针
- 一个具有operator()成员函数的类对象(传说中的仿函数)，lambda表达式

- 一个可被转换为函数指针的类对象
- 一个类成员(函数)指针

- bind表达式或其它函数对象

std::function为可调用对象的封装器，可以把std::function看做一个函数对象，用于表示函数这个抽象概念。std::function的实例可以存储、复制和调用任何可调用对象，存储的可调用对象称为std::function的目标，若std::function不含目标，则称它为空，调用空的std::function的目标会抛出std::bad_function_call异常。

以下为应用函数包装器：

~~~C++
tr1::function :
class GameCharacter; // as before
int defaultHealthCalc(const GameCharacter& gc); // as before
class GameCharacter {
public:
    // HealthCalcFunc is any callable entity that can be called with
    // anything compatible with a GameCharacter and that returns anything
    // compatible with an int; see below for details
    typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) {}
    int healthValue() const { return healthFunc(*this); }
    ...
private:
	HealthCalcFunc healthFunc;
};
~~~

使用typedef 和函数包装器，定义了一个函数类型

~~~C++
typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
~~~

即其输入为 const GameCharacter 的引用，输出为int 类型。现在任何符合这一特征的函数对象，都可以使用。

~~~C++
short calcHealth(const GameCharacter&); // health calculation function; note non-int return type
struct HealthCalculator { 				// class for health
    int operator()(const GameCharacter&) const {... } // calculation function objects
};
class GameLevel {
public:
    float health(const GameCharacter&) const; 	// health calculation
    ... 										// mem function; note non-int return type
}; 
class EvilBadGuy: public GameCharacter { ... };	// as before
class EyeCandyCharacter: public GameCharacter { // another character type; 
	... // assume same constructor as EvilBadGuy
}; 
EvilBadGuy ebg1(calcHealth); 				// character using a health calculation function
EyeCandyCharacter ecc1(HealthCalculator()); // character using a health calculation function object
GameLevel currentLevel;
...
EvilBadGuy ebg2( // character using a health calculation member function
std::tr1::bind(&GameLevel::health, currentLevel, _1)); // see below for details
~~~

以上代码说明了，任何可以通过隐士或是显示的转换为符 函数包装器类型的函数对象，都可以别用来进行健康度的计算。



### Item 36: Never redefine an inherited non-virtual function

经验：绝不重定义继承关系中的非虚函数

摘要：基类中的非虚函数时静态绑定的，不允许动态的改变。

现有如下的继承关系

~~~C++
class B {
public:
    void mf();
    ...
};
class D: public B {... };
~~~

使用方式：

~~~C++
D x; 			// x is an object of type D
B *pB = &x; 	// get pointer to x
pB->mf(); 		// call mf through pointer
D *pD = &x; 	// get pointer to x
pD->mf(); 		// call mf through pointer
~~~

在基类中定义的非虚函数，不管使用基类对象还是派生类对象，都可以访问到基类中的非虚函数。这是期望的。如果在派生类中重定义了基类中的非虚函数，就会产生不可预知的行为。

~~~~C++
class D: public B {
public:
    void mf(); 	// hides B::mf
    ...
};
pB->mf(); 		// calls B::mf
pD->mf(); 		// calls D::mf
~~~~

如上所示，派生类中的 mf() 函数将会隐藏基类中的mf() 函数，当派生类的对象再次的调用mf函数时候，就不能再次访问基类中的mf函数了。

原因分析：

继承关系中的非虚函数，是静态绑定的，而虚函数是动态绑定的。静态绑定，即在编译期间就确定了相应变量的类型，其行为根据声明时的类型决定。而动态绑定是运行时决定的，即变量的行为根据其所指向对象的类型决定。

基类中的非虚函数，其本意是在所有的实例化对象中保持不变性。而在派生类中重新的定义，这会造成矛盾，既然一致那为什么又重定义。所以为维护语义的一致性，不要重定义基类中的非虚函数。



### Item 37: Never redefine a function's inherited default parameter value

经验：绝不重新定义继承函数的默认参数值

摘要：虚函数是动态绑定的，而其默认的参数是静态绑定。

#### 1，重定义默认参数问题

首先明确讨论的范围：在基类中仅有两种函数可以继承，虚函数和非虚函数。非虚函数不可以进行重定义，所有只有虚函数可行。现在讨论的范围是，继承基类中带有默认参数的虚函数。

一个对象的静态类型，就是在代码中声明的类型。

~~~C++
class Shape {	// a class for geometric shapes
public:
    enum ShapeColor { Red, Green, Blue };
    // all shapes must offer a function to draw themselves
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};
class Rectangle: public Shape {
public:
    // notice the different default parameter value — bad!
    virtual void draw(ShapeColor color = Green) const;
    ...
};
class Circle: public Shape {
    public:
    virtual void draw(ShapeColor color) const;
    ...
};
~~~

有如下的使用方式：

~~~C++
Shape *ps; 					// static type = Shape*
Shape *pc = new Circle; 	// static type = Shape*
Shape *pr = new Rectangle; 	// static type = Shape*
~~~

以上三个指针的静态类型都是 基类指针类型。而 pc和pr 的动态类型，是其指向的类型，分别是 Circle* 和  Rectangle* 类型。使用方式如下：

~~~C++
pc->draw(Shape::Red); // calls Circle::draw(Shape::Red)
pr->draw(Shape::Red); // calls Rectangle::draw(Shape::Red)
~~~

如果使用虚函数的默认参数。会发生一种错误即，函数是动态绑定的使用派生类函数，但虚函数的默认参数确实静态绑定，来自于基类。

~~~~C++
pr->draw(); // calls Rectangle::draw(Shape::Red)!
~~~~

pr 的动态类型是 Rectangle*， 即调用了Rectangle 类中的draw 函数，是正确的。而 draw 缺省参数，期望调用 ShapeColor color = Green ，但是pr 的静态类型是 Shape\*,  其默认的参数是 Rad。 **所以就造成了调用了派生类的draw 函数，但是使用了 基类中的默认参数 Red。**

#### 2，使用虚函数包装器 NVI 解决

~~~C++
class Shape {
public:
    enum ShapeColor { Red, Green, Blue };
    void draw(ShapeColor color = Red) const { // now non-virtual
        doDraw(color); 		// calls a virtual
}
private:
	virtual void doDraw(ShapeColor color) const = 0; // the actual work is
}; // done in this func

class Rectangle: public Shape {
public:
	...
private:
	virtual void doDraw(ShapeColor color) const; // note lack of a
	... // default param val.
};
~~~

派生类中不能重新的定义非虚函数，当调用函数时候，仅有基类中的一种类型，即避免了派生了再次重定义默认函数。

总结：

基类中虚函数的默认参数，是静态绑定的，不要进行重新定义。





### Item 38: Model "has-a" or "is-implemented-in-terms-of" through composition

经验：通过组合实现“has a” 和 "is-implemented-in-terms-of" 关系

摘要：使用组合的方式，实现功能的复用。

#### 1，组合关系实现

组合有两种的说法：“有一个” 和“在什么角度的实现”。其对应两个领域，应用领域和实现领域。应用领域一般是指现实的生活，即一个人有名字，家庭住址和电话号码等。在实现领域，比如，缓存，锁搜索树等，其关系比较复杂，较难理解。

~~~~C++
class Address {... }; // where someone lives
class PhoneNumber {... };
class Person {
public:
	...
private:
    std::string name; // composed object
    Address address; // ditto
    PhoneNumber voiceNumber; // ditto
    PhoneNumber faxNumber; // ditto
};
~~~~

以上代码表达了一个人有名字，地址和电话号码等。即"has-a " 的关系而不是 “is-a” 的关系。

使用组合关系，实现使用列表list 实现集合set的功能。目标是借助list 已有的功能实现set 的特性，好处是代码的重复利用。即在list 的角度实现set。

~~~C++
template<class T> 		// the right way to use list for Set
class Set {
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;
private:
	std::list<T> rep; 	// representation for Set data
};
~~~

可以使用list 已经有的函数，来实现集合set 的功能，避免了重新的开发函数。

~~~C++
template<typename T>
bool Set<T>::member(const T& item) const {
	return std::find(rep.begin(), rep.end(), item) != rep.end();
}
template<typename T>
void Set<T>::insert(const T& item) {
	if (!member(item)) rep.push_back(item);
}
template<typename T>
void Set<T>::remove(const T& item) {
	typename std::list<T>::iterator it = // see Item 42 for info on
	std::find(rep.begin(), rep.end(), item); // "typename" here
	if (it != rep.end()) rep.erase(it);
}
template<typename T>
std::size_t Set<T>::size() const {
	return rep.size();
}
~~~

以上代码，表明了使用组合的关系利用list 实现set。

总结：组合的关系，通过发生在利用已有的类实现新类的功能，避免了二次开发。



### Item 39: Use private inheritance judiciously

经验：明智而又谨慎的使用私有继承

摘要：不到万不得已，不要使用私用继承，使用包含代替。

#### 私有继承含义

* 私有继承中不会将派生类转换为基类。
* 私有继承中基类的成员函数，全部变为派生类的私有成员函数。

~~~C++
class Person {... };
class Student: private Person {... };// inheritance is now private
void eat(const Person& p); // anyone can eat
void study(const Student& s); // only students study
Person p; // p is a Person
Student s; // s is a Student
eat(p); // fine, p is a Person
eat(s); // error! a Student isn't a Person
~~~

当调用eat函数并将派生类 Student 对象s 当做参数时候，不能够调用基类的eat 函数。说明在私有继承中派生类无法转换为基类。

私有继承意味着在某个方面的实现“is-implemented-in-terms-of”，如果D私有继承自B，说明D期望利用B中的某些特定。只是关于实现，而与接口无关。所以私有继承只继承实现，而忽略接口。

总结：

**推荐建议：**赋能用包含实现，绝不使用私有继承。



### Item 40: Use multiple inheritance judiciously

经验：明智而又谨慎的使用多重继承

摘要：尽量的不要使用多重继承，使用时最好不要超过三层。



## Chapter 7. Templates and Generic Programming

### Item 41: Understand implicit interfaces and compile-time polymorphism

经验：理解隐式接口和编译时多态

摘要：模板隐式接口以“有效表达式”为中心，且在编译时通过实例化不同的对象发生多态

#### 1，STL 编译时多态

面向对象的编程是显示的接口和运行时多态。即：

* 可以在.h 文件中找到关于接口的详细定义
* 在运行时，根据不同的对象类型，调用不同的函数

而在模板编程和元编程中，隐式接口和编译时多态是主要的特征。

~~~C++
template<typename T>
void doProcessing(T& w) {
    if (w.size() > 10 && w != someNastyWidget) {
    T temp(w);
    temp.normalize();
    temp.swap(w);
	}
}
~~~

通过以上的代码可以肯定的是：

* w 的接口是通过模板中的操作所明示的。比如w 必须支持 size() normalize() 和swap 函数。这种的方式称为--隐式接口
* 根据传入w 类型的不同，实例化不同的对象，而调用不同的函数操作，比如 “>” "!=" 。实例化操作是在编译时进行的，这种的方式称为--编译时多态。

#### 2，显式接口和隐式接口的不同

显式接口依赖于函数签名。隐式接口依赖于有效的表达式。

* 对于显式接口

~~~C++
class Widget {
public:
    Widget();
    virtual ~Widget();
    virtual std::size_t size() const;
    virtual void normalize();
    void swap(Widget& other);
};
~~~

包括，构造和析构函数，以及相应的成员函数等。

* 对于隐式接口

~~~C++
template<typename T>
void doProcessing(T& w){
	if (w.size() > 10 && w != someNastyWidget) {
	...
	}
~~~

对于隐式接口T，需要有如下的限制：即需要支持 size函数切返回整形，需要支持 ！= 操作。

其实，更宽泛的是将表达式看做一个整体即，只要w.size()无需返回整形，只要返回数支持调用“>”操作，返回bool 类型即可。同理对于 实例化后的 w 只要支持调用“！=” 操作，返回bool 类型。

总结：

* 类和模板都支持接口和多态
* 对于类，接口是显式的，且以函数签名为中心，通过虚函数在运行阶段进行多态。
* 对于模板，接口是隐式的，以有效表达式为中心，通过在编译阶段，实例化不同的对象和重载函数进行多态。



### Item 42: Understand the two meanings of typename

经验：理解typename 的两种含义

摘要：typename 声明类型参数和声明依赖类型

#### 1，typename和class

当声明模板类型参数时，使用class 和typename 效果相同。

~~~C++
template<class T> class Widget;
template<typename T> class Widget;
~~~

 必须使用typename 的时刻

#### 2，依赖名称 必须使用typename声明

两种概念

* 依赖名称 （dependent names）

~~~C++
template<typename C> 							// print 2nd element in
void print2nd(const C& container) {  			// container; this is not valid C++!
    if (container.size() >= 2) {
    C::const_iterator iter(container.begin());	// get iterator to 1st element
    ++iter; 									// move iter to 2nd element
    int value = *iter; 							// copy that element to an int
    std::cout << value; 						// print the int
	}
}
~~~

代码中的 C::const_iterator 为依赖 typename C 的迭代指针，将其称为 依赖名称。或者是嵌套依赖类型名（nested dependent type name）

* 非依赖名称 （non-dependent names）

对于int 类型，不依赖于任何的变量，称之为非依赖名称

问题：对于C::const_iterator 在有些情况下为类型，而有些情况下不是，比如恰巧C有个静态的数据类型 const_iterator，且 x 为全局变量，这样代码是乘法公式，而不是进行类型的定义。如下所示：

~~~C++
template<typename C>
void print2nd(const C& container){
C::const_iterator * x;
...
}
~~~

对于这种情况，C++中默认 依赖名称不是类型，除非进行特殊的指定。

~~~C++
template<typename C> 	// this is valid C++
void print2nd(const C& container){
    if (container.size() >= 2) {
    	typename C::const_iterator iter(container.begin());
    	...
    }
}
~~~

所以：当遇到依赖类型时候，必须马上加上 typename的类型声明。

~~~C++
template<typename C> 				// typename allowed (as is "class")
void f(const C& container, 			// typename not allowed
    typename C::iterator iter); 	// typename required
~~~

更加实际的例子为：

~~~C++
template<typename IterT>
void workWithIterator(IterT iter){
    typename std::iterator_traits<IterT>::value_type temp(*iter);
    ...
}
~~~

也可以使用 typedef 进行简化

~~~C++
template<typename IterT>
void workWithIterator(IterT iter) {
    typedef typename std::iterator_traits<IterT>::value_type value_type;
    value_type temp(*iter);
	...
}
~~~

总结：

* 当声明模板参数时，“class” 和“typename”时一样的效果
* 使用typename 指明依赖名称的类型，但是在基类列表或是在基类的初始化列表中



### Item 43: Know how to access names in templatized base classes

经验：在模板类中掌握如何的使用名称

摘要：在子类模板中，推荐使用this->，引用基类中的名称。

#### 1，如何使用基类中变量问题

有几家公司，通过模板，根据不同的实例化对象，调用不同的函数。

~~~C++
class CompanyA {
public:
    ...
    void sendCleartext(const std::string& msg);
    void sendEncrypted(const std::string& msg);
    ...
};
class CompanyB {
public:
    ...
    void sendCleartext(const std::string& msg);
    void sendEncrypted(const std::string& msg);
    ...
};
... // classes for other companies
    
class MsgInfo {... }; // class for holding information
// used to create a message

template<typename Company>
class MsgSender {
public:
    ... // ctors, dtor, etc.
    void sendClear(const MsgInfo& info) {
        std::string msg;
        create msg from info;
        Company c;
        c.sendCleartext(msg);
    }
    void sendSecret(const MsgInfo& info)// similar to sendClear, except
    {... } // calls c.sendEncrypted
};
~~~

如果使用模板继承，实例化一个日志类。

~~~C++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    ... // ctors, dtor, etc.
    void sendClearMsg(const MsgInfo& info) {
        write "before sending" info to the log;
        sendClear(info); 		// call base class function;
        						// this code will not compile!
        write "after sending" info to the log;
	}
	...
};
~~~

类 LoggingMsgSender 公有继承 MsgSender\<Company\>类 ,并通过函数sendClearMsg 调用基类中的 sendClear 函数，代码是编译不通过的。因为找不到函数 sendClear 。

**问题：**

在于当LoggingMsgSender  继承模板类 MsgSender\<Company\>时，此时并不知道确切的company 是哪个？因为只有在实例化的时候才能确定。也就是说不知道调用哪个company 中的sendClear 函数。解决方式就是要明确指定！

#### 2，解决的方式三种

* 加上this ->

this 是基类的起始地址，推荐使用。

~~~C++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    void sendClearMsg(const MsgInfo& info){
        write "before sending" info to the log;
        this->sendClear(info); 	// okay, assumes that
            					// sendClear will be inherited
        write "after sending" info to the log;
    }
    ...
};
~~~

* 使用using 声明，被调用的函数

~~~C++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    using MsgSender<Company>::sendClear; 	// tell compilers to assume
    ... 									// that sendClear is in the base class
    void sendClearMsg(const MsgInfo& info){
        ...
        sendClear(info); // okay, assumes that
        ... // sendClear will be inherited
    }
    ...
};
~~~

* 限定基类调用

~~~C++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    using MsgSender<Company>::sendClear; 	// tell compilers to assume
    ... 									// that sendClear is in the base class
    void sendClearMsg(const MsgInfo& info){
        ...
        MsgSender<Company>::sendClear(info); // okay, assumes that
        ... // sendClear will be inherited
    }
    ...
};
~~~

此种方式会阻碍，虚函数绑定行为（virtual binding behavior）。

总结：

在子类模板中，如果需要引用基类中的名称，需要使用 1，this-> 2，using 声明 3，显式的基类限定。



### Item 44: Factor parameter-independent code out of templates

经验：将参数无关的代码分解到模板外

摘要：不同的模板参数实例化的对象，代表不同的对象。

#### 1，模板类对象的重复

模板主要的作用是避免重复性的代码。但是在不经意间，看似简化的代码，实际上却形成了多个副本。比如，带有非类型模板参数的类。

~~~C++
template<typename T, std::size_t n>  // template for n x n matrices of
	 // objects of type T; see below for info
class SquareMatrix { // on the size_t parameter
    public:
    ...
    void invert(); // invert the matrix in place
};
~~~

以上的代码，对于不同的 n ，实例化了不同的类。

~~~C++
SquareMatrix<double, 5> sm1;
...
sm1.invert(); // call SquareMatrix<double, 5>::invert
SquareMatrix<double, 10> sm2;
...
sm2.invert(); // call SquareMatrix<double, 10>::invert
~~~

那如何进行避免？直观的感觉是将有关类型参数代码部分进行分离，如

~~~C++
template<typename T> 		// size-independent base class for
class SquareMatrixBase { 	// square matrices
protected:
    void invert(std::size_t matrixSize);//invert matrix of the given size
};
template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
private:
    using SquareMatrixBase<T>::invert;// avoid hiding base version of invert; see Item 33
public:
    void invert() { this->invert(n); }//make inline call to base class version of invert; 
}; 
~~~

以上需要注意的是，基类中 invert 函数为 保护属性，子类调用invert的消耗应该为0，且子类为私有继承，这表明了基类的主要目的是借助子类的实现，而不是表示“is-a” 的隶属关系。

#### 2，解决方式

还有问题是，基类中的invert 函数如何的知道其操作的数据在哪？推荐的做法如下：

定义基类：

~~~C++
template<typename T>
class SquareMatrixBase {
protected:
SquareMatrixBase(std::size_t n, T *pMem)//store matrix size and a
: size(n), pData(pMem) {} // ptr to matrix values
    void setDataPtr(T *ptr) { pData = ptr; } // reassign pData
    ...
private:
    std::size_t size; // size of matrix
    T *pData; // pointer to matrix values
};
~~~

定义子类：

~~~C++
template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
public:
SquareMatrix() // set base class data ptr to null,
: SquareMatrixBase<T>(n, 0), pData(new T[n*n]) { // allocate memory for matrix values, 
     this->setDataPtr(pData.get());} // save a ptr to thememory, and give a copy of it
    ... // to the base class
private:
	boost::scoped_array<T> pData; // see Item 13 for info on
}; // boost::scoped_array
~~~

总结：

任何依赖于非类型模板参数的模板，都会引起代码膨胀。至于解决方式需要根据具体的情况进行构思。



### Item 45: Use member function templates to accept "all compatible types."

经验：使用模板的成员函数，接受所有兼容的类型

摘要：使用成员函数模板接受所有兼容的类型，并且使用自定义的函数，禁止编译器产生默认的函数。

#### 1，模板中的隐式转换

对于内建指针有类似的隐士转换：

~~~C++
class Top {... };
class Middle: public Top {... };
class Bottom: public Middle {... };
Top *pt1 = new Middle; // convert Middle* Top*
Top *pt2 = new Bottom; // convert Bottom* Top*
const Top *pct2 = pt1; // convert Top* const Top*
~~~

子类的指针可以转换成基类的指针。但是对于模板这种隐式的类型转换时不存在的。

~~~C++
template<typename T>
class SmartPtr {
public: // smart pointers are typically
	explicit SmartPtr(T *realPtr);//initialized by built-in pointers
...
};
SmartPtr<Top> pt1 = SmartPtr<Middle>(new Middle); // convert SmartPtr<Middle> SmartPtr<Top>
SmartPtr<Top> pt2 = SmartPtr<Bottom>(new Bottom); // convert SmartPtr<Bottom> SmartPtr<Top>
SmartPtr<const Top> pct2 = pt1; // convert SmartPtr<Top> SmartPtr<const Top>
~~~

在模板中编译器将 SmartPtr\<Middle\> 和 SmartPtr\<Top\> 看做完全不同的类，不存在任何的继承关系。如果需要转换，就应该显示的进行代码书写：

~~~C++
template<typename T>
class SmartPtr {
public:
    template<typename U> // member template
    SmartPtr(const SmartPtr<U>& other); // for a "generalized
    ... // copy constructor"
};
~~~

以上代码 SmartPtr\<T> 类的构造函数，将其他类型 SmartPtr\<U> 转换为SmartPtr\<T>。

问题是，对类型U和T没有进行限制，会导致，基类转换为子类的矛盾，即（top->bottem）。

#### 2，类型限制

解决的方式是利用编译器对类型之间转换是否成立，进行限制。

~~~~C++
template<typename T>
class SmartPtr {
public:
template<typename U>
SmartPtr(const SmartPtr<U>& other) 			// initialize this held ptr
: heldPtr(other.get()) {... } 				// with other's held ptr
    T* get() const { return heldPtr; }
private: 									// built-in pointer held
    T *heldPtr; 							// by the SmartPtr
};
~~~~

通过私用成员变量定义时候的转化是否成立（heldPtr -> T），从而限制了构造函数的正确行为。即只有当 *heldPtr 到 *T能正确的转换时，编译器才能通过。

下面是智能指针的实例：

~~~C++
template<class T> class shared_ptr {
public:
    shared_ptr(shared_ptr const& r); // copy constructor
    
    template<class Y> // generalized
    shared_ptr(shared_ptr<Y> const& r); // copy constructor
    
    shared_ptr& operator=(shared_ptr const& r); // copy assignment
    
    template<class Y> // generalized
    shared_ptr& operator=(shared_ptr<Y> const& r);//copy assignment
...
};
~~~

总结：

1）拷贝构造函数并没有 explict 的限制，说明它允许进行隐式的类型转换 。

2）重写的模板拷贝函数，禁止编译器产生默认的拷贝函数。



### Item 46: Define non-member functions inside templates when type conversions are desired

经验：当需要类型转换时，在模板类中定义非成员函数

摘要：

#### **1，类型参数T，无法进行隐式推导**

在普通的类中，如果需要对传入参数进行类型的转换，需要定义非成员函数。参看24节。

以下将operator* 非成员函数模板化。

~~~C++
template<typename T>
class Rational {
public:
    Rational(const T& numerator = 0, // see Item 20 for why params
    const T& denominator = 1);// are now passed by reference
    const T numerator() const; // see Item 28 for why return
    const T denominator() const; // values are still passed by value,
    ... // Item 3 for why they're const
};
~~~

非成员函数定义：

~~~C++
template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs)
{... }
~~~

有如下的使用代码

~~~C++
Rational<int> oneHalf(1, 2); 	// this example is from Item 24,
								// except Rational is now a template
Rational<int> result = oneHalf * 2; // error! won't compile
~~~

以上的代码不会编译通过，根本的原因是在实例化时， oneHalf 为 Rational\<int> 类型，可以正常的推导出T的类型为int，但对于给定的 int 类型2 无法的进行T类型推导，即不知道T具体的类型。

#### **2，使用friend 声明，推导T类型**

解决的方式是，通过friend 声明函数，让编译器进行类型的推导。

~~~C++
template<typename T>
class Rational {
public:
	...
    friend // declare operator*
    const Rational operator*(const Rational& lhs, // function (see
    const Rational& rhs); // below for details)
};

template<typename T> // define operator*
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs)
{... }
~~~

friend 关键字，将模板函数进行了声明。这样编译器就可以利用隐式的类型推导，成功的进行编译。

#### **3，解决无法连接问题**

正确的方式是将声明和定义进行融合：

~~~C++
template<typename T>
class Rational {
public:
...
    friend const Rational operator*(const Rational& lhs, const Rational& rhs){
        return Rational(lhs.numerator() * rhs.numerator(),// same impl as in Item 24
        lhs.denominator() * rhs.denominator()); //
        } 
};
~~~

思考问题：

* 为什么模板参数不能进行隐式类型推导
* friend 能进行的原理是什么



Item 47: Use traits classes for information about types

Item 48: Be aware of template metaprogramming

以上 STL 部分，可以参考 《effective STL》 一书，有更多的细节需要掌握和学习！

新坑 effective STL！  



## Chapter 8. Customizing new and delete

这一章节主要是设计了内存的管理，一般编程掌握简单的 new 和delete 的使用即能满足！

自定义内存管理是比较高级的内容了，建议在学习理论知识的基础上，有实际的业务需求的情况下，可以探索下！

## Chapter 9. Miscellany

杂项，不杂，概念性居多，暂不赘述！
