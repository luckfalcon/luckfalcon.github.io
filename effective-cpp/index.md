# Effective C++

[luckfalcon的github page](http://luckfalcon.github.io)
# C++编程注意条款
- 拷贝构造函数与拷贝赋值运算符
>- 拷贝构造函数 : 初始化 --> A(B) --> 只发生在对象创建时
>- 拷贝赋值运算符 : 同类型对象值赋值给创建对象 --> A=B
- 值传递与引用传递
>- 值传递 ( pass-by-value ) : 调用构造函数
>- 引用传递 (passed-by-reference) : 不调用构造函数

- C++ 包含四大次级语言

>- c 语言部分编程 、class 类编程、template 模板编程、STL 标准库编程。
## 条款2：尽量以 const，enum，inline 替换 #define
- #define 没有作用域概念，且不做类型检查，仅仅是变量替换 (包括变量的前后缀，表达式代入)。
```c
#define max(a,b) f((a) > (b)? (a) : (b))
int a = 5,b = 0;
max(++a, b);	//a被累加2次
max(++a, b+10);	//a被累加1次
```
- enum hack (枚举hack) 常用来初始化类内数组大小，是模板元编程的基础技术。
```c++
class Test
{	
	static const int num = 1;//此处仅为声明而非定义
	int arr[num];
};
const int num;//此处为定义,放在实现文件中而非头文件中

enum{num = 5}//可代替上 static const int 
```
```c++
class Test
{
	private:
	enum {arr_size = 10};//在编译器不支持编译期间类内static初值时(仅限整数情况)
	static const int arr[arr_size];
};
```
>***小结***
- const 对象替换 define 常量
- inline 函数替换 define 形似函数
## 条款3：尽可能使用 const

- 令函数返回一个常量值，可以降低客户错误和意外发生，而不至于放弃安全性和高效性
```c++
//有理数operator*
class Rational{}
const Rational operator*(const Rational &lhs,const Rational &rhs);
//可以避免客户暴力行为如:
Rational a,b,c;
(a * b) = c;//在a*b的结果上调用赋值运算符operator =
```
- const 成员函数，可确保类的 const 对象可被操作
- 如过函数的返回类型是内置类型，改动函数返回值是非法的
```c++
class text
{
	public:
	const &char operator[](std::size_t position)const//处理const对象，不可修改对象
	{return st[position];}
	&char operator[](std::size_t position)//处理非const对象,可修改对象
	{return st[position];}
	private:
	std::string st;
};
```
- 将类成员声明为 mutable 类型，可以实现在 const 对象内修改对象的值，但并能完全解决 const 与 non-const 的全部问题。
```c++
class Text
{
	public:
	std::size_t length()const;
	private:
	char *ptex;
	mutable std::size_t textlength;//这些变量可以被修改，即使在const对象内部
	mutable bool LengthIsValid;
};
std::size_t Text:: length()const
{
	if(!LengthIsValid)
	{
		textlength = strlen(ptex);
		LengthIsValid = true;
	}
	return textlength;
}
```
- 在 const 和 non-const 成员函数中避免重复使用，策略是利用 non-const 调用 const 的版本来实现。
```c++
char operator[](std::size_t position)
{
	return 
	const_cast<char &>(		//将op[]返回值去除const
		static_cast<const text &>(*this)//为this加上const，调用const op[]
		)//若不给this加const，则会无限递归自己
}
```
>***小结***
- 将某些东西声明为 const 可以帮助编译器检查错误用法。const 可以被加在任何作用域内的对象、函数参数、函数返回类型、成员函数。
- 编译器强制 bitwise constness (位常量性)，但你编程时应使用 "概念上的常量性" ( conceptual constness ) 。
- 当 const 函数与 non-const 函数有等价的实现时，用 non-const 版本调用 const 版本可避免代码重复。
## 条款4：确定对象被使用前已先被初始化
- 未初始化对象可导致未定义行为 ( undefined behavior )，最终导致程序错误。
- 永远在对象使用之前对其初始化，内置类型直接初始化，类类型由其构造函数完成初始化 ( 类成员初始化发生在构造函数体之前，构造函数体内为赋值，初始化类成员应使用类成员初始化列表 )
```c++
class Obj
{
	public:
	Obj(int i_nit,char c_init,const string &s_init)
		:i(i_nit),c(c_init),s(s_init){}//初始化，通常效率更高
	Obj(int i_nit,char c_init,const string &s_init)
	{
		i = i_nit;//赋值，会先调用default构造函数，再进行拷贝赋值=
		c = c_init;
		s = s_init;
	}
	private:
	int i;
	char c;
	string s;
};
```
- c++ 对 " 定义于不同编译单元内的 non-local static 对象 " 的初始化**次序**并无明确定义。通过设计消除这种情况，做法是，将每个 non-local static 对象搬到自己的专属函数内 ( 该对象在此函数内被声明为 static ) 。这些函数返回一个 reference 指向它所含的对象。用户调用这些函数获得 static 对象的引用，而非直接使用 static 对象本身。
>编译单元 ( translation unit ) 是指产出单一目标文件 ( sigle object file ) 的源码。基本上它是单一源码文件加上其所含入的头文件 ( #include file )。
```c++
class Ob1
{
public:
	Ob1 &obj( )//在类外定义为内联函数
	{
		static Ob1 ob;
		return ob;//指向static对象的reference，而不再使用static对象自身
	}
};
```
>***小结***
- 内置类型手动初始化。
- 构造函数使用成员的初始化列表初始化，而非构造函数体内进行赋值，成员初始化次序最好与声明次序相同。
- 为免除"跨编译单元初始化次序"问题，以 local static 对象替换 non-local static 对象。
## 条款5：了解c++默默写并调用哪些函数
- 空类，编译器默认生成一个无实参的 default 构造函数、一个有实参拷贝构造函数、一个拷贝赋值运算符、一个析构函数。
- 用户自定义构造函数后，default 构造函数不指定则编译器不再提供。
- 对于成员类型为引用的类，编译器默认不生成拷贝赋值运算符 ( 原因是 c++不允许改变引用的对象 ) 。
>***小结***
- 编译器可以暗自为 class 创建 default 构造函数、copy 构造函数、copy assighnment 操作符，以及析构函数。
## 条款6：若不想使用编译器自动生成的函数，就该明确拒绝
- 为了阻止类的拷贝赋值行为，你可以通过继承一个基类的拷贝复制操作是 private 的类来实现。
```c++
class Uncopy
{
	public:
	Uncopy();
	~Uncopy();//由于非虚函数，会带来多重继承的问题
	private:
	Uncopy(const Uncopy&);
	Uncopy &operator = (const Uncopy&);
};
class Obj:private Uncopy{};//此时class Obj不再声明copy函数或copy assighnment操作符
```
>***小结***
- 为阻止编译器自动提供的功能，可将相应的成员函数声明为 private 并且不予实现，如 uncopy 基类的做法。
## 条款7：为多态基类声明 virtual 析构函数
```c++
class Base
{
	public:
	Base();
	virtual function();//带有虚成员函数的类才需要虚析构函数
	virtual ~Base();//析构函数声明为虚函数
}
class DerivedA:public Base{};
class DerivedB:protected Base{};
class DerivedC :private Base{};
```
- 每个带有 virtual 函数的 class 都有一个相应的 vtbl ( virtual table ) ，在运行时，由 vptr ( virtual table pointer ) 指出，从而实现带 virtual 函数。将不含 virtual 函数的类的析构函数声明为 virtual 会影响类的可移植性。
- 基类的析构函数非 virtual 的时，不要使用基类指针指向派生类，否则在销毁基类指针时无法调用派生类的析构函数，从而造成内存泄漏。
- pure virtual 析构函数的类为抽象类，无法实例化，只适合做基类。
```c++
class Obj
{
public:
virtual ~Obj() = 0;//纯虚函数,该类为抽象类，无法实例化，只适合做基类
};
Obj::~Obj(){};//纯虚函数的定义,不做这个定义，连接器会出错
```
- 并非所有的基类设计都是为多态用途，即不是所有的基类都需要 virtual 析构函数。
>***小结***
- polymorphic ( 带有多态性质的 ) base classes应该声明一个 virtual 析构函数。如何 class 带有 virtual 函数，那么应该声明 virtual 析构函数。
- classes 的设计目的如果不是作为 base classes 使用，或者不是为了具备多态性 ( polymorphically ) ，就不应该声明 virtual 析构函数。
## 条款8：别让异常逃离析构函数
- c++不希望在类的析构函数中抛出异常 ( 这会导致一些容器由于多个类对象析构异常而导致不明确行为 )
- 转接给客户一次处理异常的机会
```c++
class Obj
{
	public:
	static obj create();//返回一个obj对象
	void closed();//关闭，失败会抛出异常
};
class Manageobj
{
	public:
	void closed()
	{
		closed();//非析构函数抛出异常可供用户选择处理,若在析构函数内，用户无法处理，只能选择终止或者忽略异常
		closed = true;
	}
	~Manageobj()
	{
		if(!closed)
		{
			try
			{
				closed();
			}
			catch()
			{
			//记录close调用失败
			};
		}
	}
	private:
	Obj oj;
	bool closed;
};
```
>***小结***
- 析构函数绝对不要抛出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下他们 ( 阻止不传播 ) 或结束程序。
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么 class 应该提供一个普通的函数 ( 而非在析构函数中 ) 执行该操作。
## 条款9：绝不在构造和析构过程中调用 virtual 函数
>***小结***
- 在构造和析构期间不要调用 virtual 函数，因为这类调用从不下降至 derived class ( 比起当前执行构造函数和析构函数的那层 ) 。
## 条款10：令operator = 返回一个reference to *this
```c++
class Obj
{
	public:
	Obj &operator = (const Obj&rhs)
	{
		//...
		return *this;
	}//为了实现 x = y = z = Value;
};
```
>***小结***
- 令赋值 ( assignment ) 操作符返回一个 reference to *this。
## 条款11：在 operator = 中处理 " 自我赋值 "
- 让 operator = 具备 " 异常安全性 " 同时也会获得 " 自我赋值安全 " 。
```c++
class Op{};
class Obj
{
	public:
	private:
	Op *ptr; //类成员含有指针或引用
};
Obj &Obj::operator = (const Obj &rhs)
{
	Op *p = ptr;		//记住原先的ptr
	ptr = new Op(rhs.ptr);//令ptr指向*ptr的一个副本
	delete p;			//删除原先的ptr
	return *this;
	
}
```
- 复制交换 ( copy and swap ) 技术，可以作为上述方案的替代方案。
```c++
class Op{};
class Obj
{
	public:
	void swap(Obj &rhs);
	private:
	Op *ptr; //类成员含有指针或引用
};
Obj &Obj::operator = (const Obj &rhs)
{
	Op temp(rhs);//将rhs数据制作一份副本
	swap(temp);//将*this数据和上述副本的数据交换
	return *this;
	
}
//值传递(passed by value)--有时候可令编译器生成高效的代码
Obj &Obj::operator = (Obj rhs)
{
	swap(ths);
	return *this;
}
```
>***小结***
- 确保当对象自我赋值时 perator = 有良好的行为。其中的技术包括比较 " 来源对象 " 和 " 目标对象 " 的地址、精心周到的语句顺寻、以及 copy-and-swap。
- 确定任何函数如何操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。
## 条款12：复制对象时勿忘其每一个成分
- derived class 的 base class 成分在 derived class 的构造函数中应调用 base class 的构造函数完成。
- 不应该令 copy assignment 操作符调用 copy 构造函数，反之也同样没有意义。
- 如果 copy assignment 操作符和 copy 构造函数有相同重复的代码，通常在 private 区设置一个 init 函数将相同部分代码包含进去。
>***小结***
- ***copy***函数 ( copy assignment 操作符和 copy 构造函数统称 ) 应该确保复制 " 对象内的所有成员变量 " 及 " 所有 base class 成分 " 。
- 不要尝试以某个 copy 函数实现另一个 copy 函数。应该将共同机能放进第三个函数中，并由两个 copy 函数共同调用。
## 条款13：以对象管理资源
- 获取资源后立即放进管理对象内，管理对象运用析构函数确保资源被释放。以对象管理资源常被称为资源取得时机便是初始化时机 ( Resource Acquisition Is Initialization, ***RAII*** ) 。
```c++
class Obj
{
	public:
	Obj *create();
};
void f()
{		//c++11后已经弃用auto_ptr,用unique_ptr代替
	//这样做的前提是绝对没有一个以上的auto_ptr同时指向该对象
	std::auto_ptr<Obj>p_Obj(create());//auto_ptr为一个类指针对象模板
	//...
}
```
- auto_ptrs 一个特殊性质：若通过 copy assignment 操作符和 copy 构造函数复制它们，它们会变成 null，而复制所得的指针将取得资源的唯一拥有权。
```c++
{
	std::auto_ptr1<Obj>p_Obj(create());//ptr1指向create返回对象
	std::auto_ptr2(ptr1);		//ptr2指向该对象，ptr1被置为null
	ptr1 = ptr2;			//ptr1指向该对象，ptr2被置为null
}
```
- 克服 auto_ptr 的弊端，改用引用计数智能指针 ( reference-counting smart pointer,RCSP )，如 shared_ptr。
- 没有针对 " c++动态分配数组 " 的智能指针，因为 vector 和 string 几乎可以完全代替动态分配数组。
>***小结***
- 为防止资源泄露，请使用 RAII 对象，它们在构造函数中获取资源并在析构函数中释放资源。
- 两个常用的 RAII classes 分别是 tr1::shared_ptr 和 auto_ptr ( c++11 后用 unique_ptr ) 。前者通常是最佳选择，因为其 copy 行为比较直观。若选择 auto_ptr，复制动作会使它 ( 被复制物 ) 指向 null。
## 条款14：在资源管理类中小心 copying 行为
>***小结***
- 复制 RAII 对象必须一并复制它所管理的资源，所以资源的copying行为 ( 通常需要深拷贝 ) 决定 RAII 对象的 copying 行为。
- 普通而常见的 RAII class copying 行为：抑制 copying、(可用 shared_ptr) 施行引用计数方法 ( reference counting ) 。不过其他行为也都可能实现。
## 条款15：在资源管理类中提供对原始资源的访问
>***小结***
- APIs 往往要求访问原始资源 ( raw resources )，所以每个 RAII class 应该提供一个 " 取得其所管理之资源 " 的办法。
- 对原始资源的访问可能经由显示转换或隐式转换。一般而言显示转换比较安全，但隐式转换对客户比较方便。
## 条款16：成对使用 new 和 delete 时要采用相同形式
>***小结***
- new 表达式中使用 []，必须在相应的 delete 表达式中也使用 []。new 表达式中不使用 []，一定不要在相应的 delete 表达式中使用 []。
## 条款17：以独立语句将 newed 对象置入智能指针
```c++
class Obj{};
int func1();
void func2(shared_ptr<obj>,int);
func2(shared_ptr<obj>(new obj),func1());
//此处编译器要进行三个过程，new Obj、调用shared_ptr初始化指针、调用func1,
//但三者没有固定顺序，若func1发生在其它两个过程之间，且发生异常，则会导致new Obj发生泄漏
```
>***小结***
- 以独立语句将 newed 对象存入 ( 置入 ) 智能指针内。如果不这样，一旦异常被抛出，有可能导致难以察觉的资源泄漏。
## 条款18：让接口容易被正确使用，不易被误用
- 尽量让你的 types 的行为与内置 types 一致。
- 返回 shared_ptr 让接口设计者得以阻止大部分资源泄漏错误。
- 智能指针管理非 new 的资源时需要手动传入一个删除器。
>***小结***
- 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达到这个性质。
- "促进争取使用"的办法包括接口的一致性，以及与内置类型的行为兼容。
- "阻止误用"的办法是建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
- shared_ptr 支持定制型删除器 ( custom deleter )。这可防止 DLL ( 动态链接库 ) 问题，可被用来自动解除互斥锁 ( mutexes ) 等等。
>所谓的 " cross-DLL problem "，这个问题发生于 " 对象在动态连接程序库 ( DLL ) 中被 new 创建，却在另一个 DLL 内被 delete 销毁 " 。
## 条款19：设计 class 犹如设计 type
>新 class 设计规范问题：
- 新的 type 应该如何被创建和销毁？
- 对象的初始化和对象的赋值该有什么样的差别？
- 新 type 对象如果被 passed by value ( 以值传递 )，意味着什么？
- 什么是新 type 的 " 合法值 " ？
- 你的新 type 需要配合某个继承图系 ( inheritance graph ) 吗？
- 你的新 type 需要什么样的转换？
- 什么样的操作符和函数对此新 type 而言是合理的？
- 什么样的标准函数应该驳回？
- 谁该取用新 type 的成员？
- 什么是新 type 的"未声明接口"？
- 你的新 type 有多么一般化？
- 你真的需要一个新 type 吗？
>***小结***
- class 的设计就是 type 的设计。在定义一个新type时请考虑以上问题。
## 条款20：宁可以 passed-by-reference-to-const 替换 passed-by-value
- 缺省情况下 C++ 以 by value 方式传递对象至函数。
- 一般而言，pass by value 唯一对象是内置类型和 STL 的迭代器和函数对象。
>***小结***
- 尽量以 passed-by-reference-to-const 替换 passed-by-value。前者通常高效，并可避免切割问题 (slicing problem)。
- 以上规则不适合内置类型，以及STL的迭代器和函数对象。对它们而已，passed-by-value往往比较适当。
## 条款21：必须返回对象时，别妄想返回其 reference
- 一个 " 必须返回新对象 " 的正确写法：
```c++
class Rational
{
	public:
	 Rational(int numerator = 0,int denominator = 1);
	private:
	int n;//分子(numerator)
	int d;//分母(denominator)
	friend const Rational operator*(const Rational &lhs,const Rational &rhs)
	{
		return Rational(lhs.n*rhs.n,lhs.d*rhs.d);
	}
};
```
>***小结***
- 绝不要返回 pointer 或 reference 指向一个 local stack 对象，或返回 reference 指向一个 heap-allocated 对象，或返回 pointer 或 reference 指向一个 local static 对象而有可能同时需要多个这样的对象。
## 条款22：将成员变量声明为 private
>***小结***
- 切记将成员变量声明为 private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供 class 作者以充分的实现弹性。
- protected 并不比 public 更具封装性。
## 条款23：宁以 non-member、non-friend 替换 member 函数
- 将所有便利函数放在多个头文件内但隶属同一个命名空间，意味着客户可以扩展这一组便利函数。
```c++
// class_def.h
namespace name
{
	class Obj 
	{
	public:
	void fun1();
	void fun2();
	void fun3();
	};
	void do_fun1(Obj &oj){oj.fun1();}//核心机能，包含客户都需要的non-member函数
	void do_fun3(Obj &oj){oj.fun3();}//非成员函数可以进一步提高class的封装性
}
//fun1_use.h//客户扩展头文件
namespace name
{
	void fun1();
}
```
>***小结***
- 宁以拿 non-member、non-friend 替换 member 函数。这样可以增加封装性、包裹弹性 ( packaging flexibility ) 和机能扩充性。
## 条款24：若所有参数皆需类型转换，请为此采用 non-member 函数
```c++
class Rational
{
	public:
	 Rational(int numerator = 0,int denominator = 1);
	 int numerator()const;
	 int denominator()const;
	private:
	int n;//分子(numerator)
	int d;//分母(denominator)
};
//非成员友元函数声明方式
const Rational operator*(const Rational &lhs,const Rational &rhs)
	{
		return Rational(lhs.numerator()*rhs.numerator(),lhs.denominator()*rhs.denominator());
	}
```
>***小结***
- 如果你需要为某个函数的所有参数 ( 包括被 this 指针所指的那个隐喻参数 ) 进行类型转换，那么这个函数必须是 non-member 函数。
## 条款25：考虑写出一个不抛出异常的 swap 函数
- 所有的 STL 容器都提供有一个 public swap 成员函数和 std::swap 特化版本 ( 用以调用前者 )。
- c++ 只允许对 class template 偏特化 ( partially specialize )，但不能对 function template 偏特化。
- c++ std 空间可以全特化所有内含 template，但不可以添加新的 template 进去。
```c++
namespace name_obj
{
	class Obj
	{
		public:
		void swap(const Obj &b){using std::swap;};
	}
	void swap(Obj<T> &a,Obj<T> &b){a.swap(b);}//合法
}
namespace std//在std内特化
{
template<>
void swap<Obj>(obj &a,Obj &b){a.swap(b);};//特例化某个类，合法
//Obj成为类模板的时
template<typename T>
void swap<Obj<T>>(Obj<T> &a,Obj<T> &b){a.swap(b);}//函数模板偏特化，错误，不合法
template<typename T>
void swap(Obj<T> &a,Obj<T> &b){a.swap(b);}//也不合法
}
```
>pimpl 是 "pointer to implementation" 的缩写。
- swap 的版本包含，default swap (适合不含指针成员的类或类模板)、member swap、non-member swap、std::swap 特化版。
- 如果缺省 swap 效率不足 ( class 或 template 使用了某种 pimpl 手法 )，可试以下方法：
>1. 提供一个 public swap 成员函数，让它高效地置换你的类型的两个对象值。该函数不应抛出异常。  
>2. 在你的 class 或 template 所在的命名空间内提供一个 non-member swap，并令它调用上述 swap 成员函数。
>3. 如果你正编写一个 class ( 而非 class template )，为你的 class 特化 std::swap。并令它调用你的 swap 成员函数。
- swap 一个最好的应用是帮助 classes( 和 class templates ) 提供强烈的异常 ( exception-safety ) 保障。
>***小结***
- 当 std::swap 对你的类型效率不高时，提供一个成员 swap 函数，并确定这个函数不抛出异常。
- 如果你提供一个 member swap，也该提供一个 non-member swap 用来调用前者。对于 class ( 而非 template )，也请特化 std::swap。
- 调用 swap 时应针对 std::swap 使用 using 声明式，然后调用 swap 并且不带任何 " 命名空间资格修饰 " 。
- 为 " 用户定义类型 " 进行 std templates 全特化是好的，但千万不要尝试在 std 内加入某些对 std 而言全新的东西。

> ~~***实现 (implementations)***~~
## 条款26：尽可能延后变量定义式的出现时间
- 目的是减少不必要的对象构造和析构
```c++
class Obj{};
Obj oj;	//A：1次构造+1次析构+n次赋值				
for(int i;i < n;++i)	
{
	oj = i;
};
//
for(int i;i < n;++i)	
{
	Obj oj;	//B：n次构造+n次析构，通常B比A更高效一点，除非，赋值比构造+析构成本低
	oj = i;
};
```
>***小结***
- 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。
## 条款27：尽量少做转型动作
- c++ 四类型转换形式 :
<table border="1">
<tr>
<th>const_cast<T>(expression)</th>
<th>dynamic_cast<T>(expression)</th>
<th>reinterpret_cast<T>(expression)</th>
<th>static_cast<T>(expression) </th>
</tr>
<tr>
<td>const 到 non-const 转换</td>
<td>运行时，安全向下类型转换 </td>
<td>执行低级转型</td>
<td>除去 const 到 non-const 的转换，其他一般类型转换</td>
</tr>
</table>

>***小结***
- 如果可以，尽量避免转型，特别是在注重效率的代码中避免 dynamic_cast。
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需要将转型放到他们自己的代码内。
- 宁可使用 C++-style ( 新式 ) 转型，不要使用旧式转型。前者容易辨识出来，而且也比较有着分门别类的执掌。
## 条款28：避免返回 handles 指向对象内部成分
>***小结***
- 避免 handles ( 包括 references、指针、迭代器 ) 指向对象内部。遵守这个条款可以增加封装性，帮助 const 成员函数的行为像个 const，并将发生"虚吊号码牌"( dangling handles ) 可能降到最低。
>虚吊号码牌：使用已经销毁的对象的引用、指针或迭代器。
## 条款29：为 " 异常安全 " 而努力是值得的
- 异常安全性两个条件：
>1. 异常抛出时，不泄漏任何资源。
>2. 异常抛出时，不允许数据败坏。
- 异常安全函数提供三种不同级别保证：

<table border="1">
<tr>
  <th style="width: 200px" style="text-align: center">基本保证</th>
  <th style="width: 200px" style="text-align: center">强烈保证 </th>
  <th style="width: 200px" style="text-align: center">不抛保证</th>
</tr>
<tr>
  <td>异常抛出后，程序内所有事物保持有效状态，这些状态是未预知的|异常抛出后，程序状态不改变，回到被调用前状态</td>
  <td>异常抛出后，程序状态不改变，回到被调用前状态</td>
  <td>承诺不抛出异常，因其总能完成原有的功能。作用于内置类型所有操作应提供 nothrow 保证</td>
</tr>
</table>

>***小结***
- 异常安全函数 ( Exception-safty functions ) 即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型。
- "强烈保证" 往往能够以 copy-and-swap 实现出来，但 " 强烈保证 " 并非对所有函数都可实现或具备现实意义。
- 函数提供 " 异常安全保证 " 通常最高只等于其所调用之各个函数的 " 异常安全保证 " 中的最弱者。 
## 条款30：透彻了解 inlining 的里里外外
- inlining 在大多数 c++ 程序中是编译期行为。
- inline 函数通常一定被置于头文件中，通常置于函数本体小的函数前，向编译器申请在调用处替换展开本体，但非强制命令，编译器可忽略。
- 对于函数本体较大的函数 inlining 会引发代码膨胀。
- 编译器通常不会对 " 通过函数指针而进行的调用 " 实施 inlining。 
>***小结***
- 将大多数 inlining 限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级 ( binary upgradability ) 更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
- 不要只因为 function templates 出现在头文件，就将它们声明为 inline。
## 条款31：将文件间的编译依存关系降至最低
- 将接口与实现分离。
- 编译依存性最小化的本质：现实中让头文件尽可能自我满足，万一做不到，则让它与其
他文件内的声明式 ( 而非定义式 ) 相依 -- 该设计策略如下：
>1. 如果使用 object references 或 object pointers 可以完成任务，就不要使用
objects。
>2. 如果能够，尽量以 class 声明式替换 class 定义式。
>3. 为声明式和定义式提供不同的头文件。
- Handle classes 和 Interface classes 解除了接口和实现之间的耦合关系，从而降低文件间的编译依存性 ( compilation dependencies ) 。
>1. 在 Handle classes 身上，成员函数必须通过 implementation pointer 取得对象数据。那会为每一次访问增加一层间接性。
>2. 至于 Interface classes，由于每个函数都是 virtual，所以你必须为每次函数调用付出一个间接跳跃 ( indirect jump ) 成本 ( 见条款 7 ) 。

>***小结***
- 支持 " 编译依存性最小化 " 的一般构想是: 相依于声明式，不要相依于定义式。基于此构想的两个手段是 Handle classes 和 Interface classes。
- 程序库头文件应该以 " 完全且仅有声明式 " ( full and declaration-only forms ) 的形式存在。这种做法不论是否涉及 templates 都适用。
> ***~~六、继承与面向对象~~***
## 条款32：确定你的 public 继承塑模出 is-a 关系
- 以 C++ 进行面向对象编程，最重要的一个规则是：public inheritance ( 公有继承 ) 意味 " is-a " ( 是一种 ) 的关系。
- is-a 并非是唯一存在于 classes 之间的关系。另两个常见的关系是 has-a ( 有一个 ) 和 is-implemented-in-terms-of ( 根据某物实现出 ) 。 
>***小结***
- " public 继承 " 意味 is-a。适用于 base classes 身上的每一件事情一定也适用于 derived classes 身上，因为每一个 derived class 对象也都是一个 base class 对象。
## 条款 33：避免遮掩继承而来的名称
- 内层作用域的名称会遮掩 ( 遮蔽 ) 外围作用域的名称。
- 编译器查找名称规则由内层作用域查找到名称后即停止查找，并不关注类型。
- derived class 作用域被嵌套在 base class 作用域内。
>***小结***
- derived classes 内的名称会遮掩 base classes 内的名称。 在 public 继承下从来没有人希望如此。
- 为了让被遮掩的名称再见天日，可使用 using 声明式或转交函数 ( forwarding functions ) 。
>using 声明式和转交函数：
 ```c++
class base{ public: virtual void fun1(int x);};
class derived1 : public base
{
	public:
	using base::fun1;
	void fun1();
};
class derived1 : private base
{
	public:
	virtual void fun1()//转交函数
	{
		base::fun1();
	}
};
```
## 条款 34：区分接口继承和实现继承
- pure virtual 函数有两个最突出的特性：它们必须被任何 " 继承了它们 " 的具象 class 重新声明，而且它们在抽象 class 中通常没有定义。
>声明一个 pure virtual 函数的目的是为了让 derived classes 只继承函数接口。
- derived classes 继承其函数接口, 但 impure virtual 函数会提供一份实现代码, derived classes 可能覆写 ( override ) 它。
>声明简朴的(非纯) impure virtual 函数的目的，是让 derived classes 继承该函数的接口和缺省实现。
- 声明 non-virtual 函数的目的是为了令 derived classes 继承函数的接口及一份强制性实现。

<table border="1" align="center" valign="left">
<tr>
<th style="width: 200px" style="text-align: center">pure virtual 函数</th>
<th style="width: 200px" style="text-align: center">simple ( impure ) virtual 函数</th>
<th style="width: 200px" style="text-align: center">non-virtual 函数</th>
</tr>
<tr>
<td>只继承接口</td>
<td>继承接口和一份缺省实现</td>
<td>继承接口和一份强制实现</td>
</tr>
</table>

- " 80 - 20 " 法则：一个典型的程序有 80 % 的执行时间花费在 20 % 的代码身上 。
>***小结***
- 接口继承和实现继承不同。在 public 继承之下，derived classes 总是继承 base class 的接口。
- pure virtual 函数只具体指定接口继承。
- 简朴的 ( 非纯 ) impure virtual 函数具体指定接口继承及缺省实现继承。
- non-virtual 函数具体指定接口继承以及强制性实现继承。
## 条款 35：考虑 virtual 函数以外的其他选择
- 通过 Non-Virtual Interface 手法实现 Template Method 模式：
```c++
class Base
{
	public:
	int fun()const//这个no-virtual 函数为 virtual 函数的外覆器 (wrapper)
	{
		int relval=dofun();
		return relval;
	}
	private:
	virtual int dofun()const{...};
};
```
>令客户通过 public non-virtual 成员函数间接调用 private virtual 函数 -- 为 non-virtual interface ( NVI ) 手法。

> derived classes 可重新定义继承而来的 private virtual 函数。
- 通过 Function Pointers 实现 Strategy 模式
- 通过 函数模板 function < T > 完成 Strategy 模式
>trl1: :bind，可改变可调对象入口参数个数和顺序，同时返回一个新的可调对象。
- 古典的 Strategy 模式 (设计模式 ( design patterns ) ) 。
>几种 virtual 函数替代方案如下：
1. 使用 non-virtual interface ( NVI ) 手法，那是 Template Method 设计模式的一种特殊形式。它以 public non-virtual 成员函数包庄较低访问性 ( private 或 protected ) 的 virtual 函数。
2. 将 virtual 函数替换为 " 函数指针成员变量 "，这是 Strategy 设计模式的一种分解表现形式。
3. 以 trl::function 成员变量替换 virtual 函数，因而允许使用任何可调用物 ( callable entity ) 搭配一个兼容于需求的签名式。这也是 Strategy 设计模式的某种形式。
4. 将继承体系内的 virtual 函数替换为另一个继承体系内的 virtual 函数。这是 Strategy 设计模式的传统实现手法。
>***小结***
- virtual 函数的替代方案包括 NVI 手法及 Strategy 设计模式的多种形式。NVI手法自身是一个特殊形式的 Template Method 设计模式。
- 将机能从成员函数移到 class 外部函数，带来的一个缺点是，非成员函数无法访问 class 的 non-public 成员。
- trl::function 对象的行为就像一般函数指针。这样的对象可接纳 " 与给定之目标签名式 ( target signature ) 兼容 " 的所有可调用物 ( callable entities ) 。
## 条款 36：绝不重新定义继承而来的 non-virtual 函数
- derived classes 绝对不该重新定义一个继承而来的non-virtual 函数 ( 此处指的是 base class 析构函数 ) 。
>***小结***
- 绝对不要重新定义继承而来的 non-virtual 函数。
## 条款 37: 绝不重新定义继承而来的缺省 ( 默认 ) 参数值
- 只能继承两种函数: virtual 和 non-virtual 函数。
- virtual 函数系动态绑定 ( dynamically bound ) ，而缺省(默认)参数值却是静态绑定 ( statically bound ) 。
>对象的所谓静态类型 ( static type ) ，就是它在程序中被声明时所采用的类型。

>对象的所谓动态类型 ( dynamic tvpe ) 则是指 " 目前所指对象的类型 "。
```c++
class base
{
	public:
	virtual void set(base b = bs)const = 0;//带有默认参数
}；
class derived:public base
{
	public:
	virtual void set(base b = bs)const;//会出现重复定义
	virtual void set(base b = ds)const;//默认参数不一样
	virtual void set(base b)const;//动态绑定(base指针引用访问时)时才会继承base默认参数，
	//静态绑定(derived对象访问)时不会继承默认参数
};
```
- 条款35列了不少 virtual 函数的替代设计，其中之一是 NVI ( non-virtual interface ) 手法令 base class 内的一个 public non-virtual 函数调用 private virtual 函
数，后者可被 derived classes 重新定义。
>***小结***
- 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而 virtual 函数一一你唯一应该覆写的东西一一却是动态绑定。
## 条款 38: 通过复合塑模出 has-a 或 " 根据某物实现出 "
- 复合 ( composition ) 是类型之间的一种关系，当某种类型的对象内含它种类型的对象，便是这种关系
>复合 ( composition ) 这个术语有许多同义词，包括 layering ( 分层 ) ，constainment ( 内含 ) , aggregation ( 聚合 ) 和 embedding ( 内嵌 ) 。
- 应用域 ( appjicarion domain ) 与实现域 ( implementation domain )
>1. 程序中的对象其实相当于你所塑造的世界中的某些事物 ( 如人、汽车 )，这样的对象属于应用域 ( appjicarion domain ) 部分。
>2. 其他对象则纯粹是实现细节上的人工制品(如缓冲区、互斥锁)，这些对象相当于你的软件的实现域。
>3. 当复合发生于应用域内的对象之间，表现出 has-a 的关系; 当它发生于实现域内则是表现 is-implemented-in-terms-of 的关系。
- set 和 list 的关系非 is-a 关系，而是 is-implemented-in-terms-of 关系。
>***小结***
- 复合 ( composition ) 的意义和 public 继承完全不同。
- 在应用域 ( application domain ) ，复合意味 has-a ( 有一个 )。在实现域 ( implementation domain ) ，复合意味 is-implemented-in-terms-of ( 根据某物实现出 ) 。
## 条款 39: 明智而审慎地使用 Private 继承
- Private 继承意味 is-implemented-in-terms-of ( 根据某物实现出 )。
- 尽可能使用复合，必要时才使用 private 继承。
- private 继承主要用于 " 当一个意欲成为 derived class 者想访问一个意欲成为 base class 者的 protected 成分，或为了重新定义一或多个 virtual 函数 "。
```c++
class empty{};
sizeof(empty) = 1;
class derived:private empty
{
	private:
	int x;
};
sizeof(derived) == sizeof(int);
//编译器 EBO (empty base optimization; 空白基类最优化)，一般发生在单一继承才可行，多继承则不会发生。
```
>- 现实中的 " empty " classes 并不真的是 empty，往往内含 typedefs，enums， static 成员变量，或 non-virtual 函数。
>- 许多技术用途的 empty classes，其中内含有用的成员 ( 通常是 typedefs )，包括 base classes unary_function 和 binary_function，这些是 " 用户自定义之函数对象 " 通常会继承的 classes。

>***小结***
- Private 继承意味 is-implemented-in-terms of ( 根据某物实现出 ) 。它通常比复合 ( composition ) 的级别低。但是当 derived class 需要访问 protected base class 的
成员，或需要重新定义继承而来的 virtual 函数时，这么设计是合理的。
- 和复合 ( composition ) 不同，private 继承可以造成 empty base 最优化。这对致力于 " 对象尺寸最小化 " 的程序库开发者而言，可能很重要。
## 条款 40: 明智而审慎地使用多重继承
- 多重继承 ( multiple inheritance；MI )，单一继承 ( single inheritance; SI ) 。
- C++ 用来解析 ( resolving ) 重载函数调用的规则：在看到是否有个函数可取用之前，C++ 首先确认这个函数对此调用之言是最佳匹配。找出最佳匹配函数后才检验其可取用性。
- 多重继承会导致二义性问题 ( 避免继承得来的成员变量重复 )，通常采用 virtual 继承:
```c++
class derived:virtual public base1, virtual public base2{};
```
>***小结***
- 多重继承比单一继承复杂。它可能导致新的歧义性，以及对 virtual 继承的需要。
- virtual 继承会增加大小、速度、初始化 ( 及赋值 ) 复杂度等等成本。如果 virtual base classes 不带任何数据，将是最具实用价值的情况。
- 多重继承的确有正当用途。其中一个情节涉及 " public 继承某个 Interface class " 和 " private 继承某个协助实现的 class " 的两相组合。
>七、模板与泛型编程
## 条款 41：了解隐式接口和编译期多态
- 面向对象编程世界总是以显式接口 ( explicit interfaces ) 和运行期多态 ( runtime polymorphism ) 解决问题。
- Templates 及泛型编程的世界以隐式接口 ( implicit interfaces ) 和编译期多态 ( compile-time polymorphism ) 为主。 
>***小结***
- classes 和 templates 都支持接口 ( interfaces ) 和多态 ( polymorphism ) 。
- 对 classes 而言接口是显式的 ( explicit ), 以函数签名为中心。多态则是通过 virtual 函数发生于运行期。
- 对 template 参数而言，接口是隐式的 ( implicit ) ，奠基于有效表达式。多态则是通过 template 具现化和函数重载解析 ( function overloading resolution ) 发生于编译期。
## 条款 42：了解 typename 的双重意义
- C++ 有个规则可以解析 ( resoive ) 此一歧义状态：如果解析器在 template 中遭遇一个嵌套从属名称，它便假设这名称不是个类型，除非你告诉它是。所以缺省情况下嵌套从属名称不是类型。
>- template 内出现的名称如果相依于某个 template 参数，称之为从属名称 ( dependent names ) 。
>- 如果从属名称在 class 内呈嵌套状，我们称它为嵌套从属名称 ( nested dependent rame ) 。
- 只要在嵌套从属名称之前放置关键字 typename，即可告知解析器该名称为类型。
```c++
template<typename T>
typename T::const_iterator it();//告知解析器T::const_iterator 为类型
T t;
//T 非嵌套从属名称
```
- typename 不可以出现在 base classes list 内的嵌套从属类型名称之前，也不可在 mermber initialization list ( 成员初值列 ) 中作为 base class 修饰符。
```c++
template<typename T>
class derived:public base<T>::nested // base classes list
{
	public:
	explicit derived(int x):base<T>::nested(x)	//成员列表初始化
	{
		typename  base<T>::nested temp;
		//...
	}
};
```
>***小结***
- 声明 template 参数时，前缀关键字 class 和typename 可互换。
- 请使用关键字 typename 标识嵌套从属类型名称; 但不得在 base class lists ( 基类列 ) 或 member initialization list ( 成员初值列 ) 内以它作为 base class 修饰符。
## 条款 43：学习处理模板化基类内的名称
- 模板化基类 ( templatized base classes )内的函数名称会被 derived classes 掩盖。
- 若基类模板存在特例化版本，则在 derived class template 的时候会发生编译错误，这时可以将 derived class template 涉及 base class template 的成员函数前加 " this-> " 来指涉 base class template 内的函数。 
>***小结***
- 可在 derived class templates 内通过 " this-> " 指涉 base class templates 内的成员名称，或通过一个明白写出的 " base class 资格修饰符 " 完成。
## 条款 44：将与参数无关的代码抽离 templates
- 共性与变性分析 ( commonality and variability analysis ) 

>***小结***
- Templates 生成多个 classes 和多个函数，所以任何 template 代码都不该与某个造成膨胀的 template 参数产生相依关系。
- 因非类型模板参数 ( non-type template parameters ) 而造成的代码膨胀，往往可消除，做法是以函数参数或 class 成员变量替换 template 参数。
- 因类型参数 ( type parameters ) 而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述 ( binary representations ) 的具现类型 ( instantiation types ) 共享实现码。

## 条款 45：运用成员函数模板接受所有兼容类型
- 如果以带有 base-derived 关系的 B, D 两类型分别具现化某个 template，产生出来的两个具现体并不带有 base-derived 关系。
- Templates 和泛型编程 ( Generic Programming )
- 泛化 ( generalized ) copy构造函数
```c++
template<typename T>
class SmartPtr
{
	public:
	//泛化copy构造函数,并保证只根据U生成T对象,保证public继承的隐式转换
	//泛化copy构造函数不等于copy构造函数，如果需要控制copy行为，还需自定义copy构造函数
	SmartPtr(const SmartPtr &sptr);//普通copy构造函数
	template<typename U>		//泛化copy构造函数
	SmartPtr(const SmartPtr<U> &other):heldPtr(other.get())
	{...}
	T *get()const {return heldPtr;}
	private:
	T *heldPtr;
};
```
>***小结***
- 请使用 member function templates ( 成员函数模板 ) 生成 " 可接受所有兼容类型 " 的函数。
- 如果你声明 member templates 用于 " 泛化 copy构造函数 " 或 " 泛化 assigmment操作 "，你还是需要声明正常的 copy 构造函数和 copy assigmmenmt 操作符。
## 条款 46：需要类型转换时请为模板定义非成员函数
- 在 function template 实参推导过程中从不进行隐式类型转换。
- Class templates 并不倚赖 tetmplate 实参推导 ( 后者只施行于 function templates 身上 ) ，所以编译器总是能够在 class template 具现化时得知 T。
```c++
template<typename T>
class Rational//template class
{
	public:
	 Rational(T numerator = 0,T denominator = 1);
	 T numerator()const;
	 T denominator()const;
	 friend const Rational operator*(const Rational &lhs,const Rational &rhs)
	{
		return Rational(lhs.numerator()*rhs.numerator(),lhs.denominator()*rhs.denominator());
	}//定义在外部时，将只可通过编译却无法连接
	private:
	T n;//分子(numerator)
	T d;//分母(denominator)
};
```
>***小结***
- 当我们编写一个 class template，而它所提供之 " 与此 template 相关的 " 函数支持 " 所有参数之隐式类型转换 " 时，请将那些函数定义为 " class template 内部的 friend 函数 " 。
## 条款 47：请使用 traits classes 表现类型信息
- STL 主要由 " 用以表现容器、迭代器和算法 " 的 templates 构成，以及若干工具性 templates。
- STL 共有 5 种迭代器分类

<table border="1">
<tr>
<th style="width: 200px" style="text-align: center" style="text-valign: center">Input 迭代器 </th>
<th style="width: 200px" style="text-align: center" style="text-valign: center">Output 迭代器  </th>
<th style="width: 200px" style="text-align: center" style="text-valign: center">Forward 迭代器</th>
<th style="width: 200px" style="text-align: center" style="text-valign: center">Bidirectional迭代器 </th>
<th style="width: 200px" style="text-align: center" style="text-valign: center">Random access 迭代器  </th>
<tr>
<tr>
<td>Istream_iterators</td>
<td>Ostream_iterators</td>
<td>不支持linked list</td>
<td>STL list, set, multiset, map, multimap 的迭代器</td>
<td>vector,deque, string 的迭代器</td>
</tr>
<tr>
<td>只能向前最多一次读操作</td>
<td>只能向前最多一次写操作</td>
<td>向前多次操作</td>
<td>可以向前移动，还可以向后移动</td>
<td>支持迭代器算术，双向</td>
</tr>
</table>

>is-a关系：
>- input、output
>- forward : public input
>- bidirectional : public forward
>- random : public bidirectional
- advance(Iter, Disance)
- 设计并实现一个 traits class：
>- 确认若干你希望将来可取得的类型相关信息。例如对迭代器而言，我们希望将来可取得其分类 ( category ) 。
>- 为该信息选择一个名称 ( 例如 iterator_category ) 。
>- 提供一个 template 和一组特化版本 ( 例如稍早说的 iterator_traits ) ，内含你希望支持的类型相关信息。
- 如何使用一个 traits class：
>- 建立一组重载函数 ( 身份像劳工 ) 或函数模板 ( 例如 doadvance ) ，彼此间的差异只在于各自的 traits 参数。令每个函数实现码与其接受之 traits 信息相应和。
>- 建立一个控制函数 ( 身份像工头 ) 或函数模板 ( 例如 advance ) ，它调用上述那些 " 劳工函数 " 并传递 traits class 所提供的信息。

>***小结***
- Traits classes 使得 " 类型相关信息 " 在编译期可用。 它们以 templates 和 " templates 特化 " 完成实现。
- 整合重载技术 ( overloading ) 后，traits classes 有可能在编译期对类型执行 if...else 测试。
## 条款 48：认识 template 元编程
- Template metaprogramming ( TMP，模板元编程 ) 是编写 template-based C++ 程序并执行于编译期的过程。
- template metaprogram ( 模板元程序 ) 是以 C++ 写成、执行于 C++ 编译器内的程序。
```c++
//阶层(factorial)模板元编程,递归模板具现化实现循环
template<unsigned n>
struct Factorial
{
	enum{value = n*Factorial<n-1>::value};
}
template<>
struct Factorial<0>
{
	enum{Value = 1};
}
//main.cpp
cout<<Factorial<5>::value<<endl;//计算5的阶层
```
>***小结***
- Template metaprogramming ( TMP，模板元编程 ) 可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率。
- TMP 可被用来生成 " 基于政策选择组合 " ( based on combinations of policy choices ) 的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。
> ~~***第八章 定制 new 和 delete***~~
## 条款 49：了解 new-handler 的行为
>***小结***
- set_new_handler 允许客户指定一个函数，在内存分配无法获得满足时被调用。
- Nothrow new是一个颇为局限的工具，因为它只适用于内存分配; 后继的构造函数调用还是可能抛出异常。
## 条款 50：了解 new 和 qelete 的合理替换时机
- 何时可在 " 全局性的 " 或 " class 专属的 " 基础上合理替换缺省的 new 和 delete:
>- 为了检测运用错误。
>- 为了收集动态分配内存之使用统计信息。
>- 为了增加分配和归还的速度。
>- 为了降低缺省内存管理器带来的空间额外开销。
>- 为了弥补缺省分配器中的非最佳齐位 ( suboptimal atignment ) 。 
>- 为了将相关对象成簇集中。new 和 delete 的 " placement 版本 " ( 见条款52 ) 有可能完成这样的集簇行为。
>- 为了获得非传统的行为。

>***小结***
- 有许多理由需要写个自定的 new 和 delete，包括改善效能、对 heap 运用错误进行调试、收集 heap 使用信息。
## 条款 51：编写new 和 delete 时需固守常规
>***小结***
- OPerator new 应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用 new-handler。它也应该有能力处理 0 bytes 申请。Class 专属版本则还应该处理 " 比正确大小更大的 ( 错误 ) 申请 " 。
- operator delete 应该在收到 null 指针时不做任何事。Class 专属版本则还应该处理 " 比正确大小更大的 ( 错误 ) 申请 " 。
## 条款52：写了 placement new 也要写 placement delete
- 如果 operator new 接受的参数除了一定会有的那个 size_t 之外还有其他，这便是个所谓的 placerment new。
```c++
#include<new>//标准库
static void* operator new(std: :size_t, void* PMemory) throw(); //纳入标准库内的pacement new
//非标准
static void* operator new(std: :size_t, std::ostream &) throw(std::bad_alloc);
static void operator delete(void *, std::ostream &) throw();//placement delete
```
- 缺省情况下 C++ 在 global 作用域内提供以下形式的 operator new：
```c++
#include<new>
static void* operator new(std: :size_t)throw();//正常的new
static void* operator new(std: :size_t, void *) throw();//placement new
static void* operator new(std: :size_t, const std::nothrow_t &) throw();//nothrow new
```
- 如果你在 class 内声明任何 operator news，它会遮掩上述这些标准形式。为使这些函数有着平常的行为，只要令你的 class 专属版本调用 global 版本即可：
```c++
//将标准形式放在一个class中，然后让客户继承及using声明式取得标准形式
class standard
{
	public:
	static void* operator new(std::size_t size)throw(std::bad_alloc)
	{
		return ::operator new(size);//分别调用标准形式
	}
	...
}
class wiget:public standard
{
	public:
	using standard::operator new;
	//定义自己的operator new
}
```
>***小结***
- 当你写一个 placement operator new，请确定也写出了对应的 placerment operator delete。如果没有这样做，你的程序可能会发生隐微而时断时续的内存泄漏。
- 当你声明 placement new 和 placement delete，请确定不要无意识 ( 非故意 )  地遮掩了它们的正常版本。
>第九章杂项讨论
## 条款 53：不要轻忽编译器的警告
- 警告信息天生和编译器相依，不同的编译器有不同的警告标准。
>***小结***
- 严肃对待编译器发出的警告信息。努力在你的编译器的最高 ( 最严苛 ) 警告级别下争取 " 无任何警告 " 的荣誉 。
- 不要过度倚赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦移植到另一个编译器上，你原本倚赖的警告信息有可能消失。
## 条款 54：让自己熟悉包括 TR1 在内的标准程序库
>***小结***
- C++ 标准程序库的主要机能由 STL、iostreams、locales 组成。并包含 C99 标准程序库。
- TR1 添加了智能指针 ( 例如 trl: :shared_ptr ) 、一般化函数指针 ( trl: :function ) 、hash-based 容器、正则表达式 ( regular expressions ) 以及另外 10个组件的支持。
- TR1 自身只是一份规范。为获得 TR1 提供的好处，你需要一份实物。一个好的实物来源是 Boost。
## 条款 55：让自己熟悉 Boost
- Boost 是一个 C++ 开发者集结的社群, 也是一个可自由下载的 C++ 程序库群。[boost网址]( http://boost.org )
- Boost Graph Library (用于编写任意 graph 结构)。
- Boost MPL Library ( 一个元编程程序库，metaprogramming library )。
- Boost 程序库内包含种类：
>- 字符串与文本处理。
>- 容器。
>- 函数对象和高级编程。
>- 泛型编程 ( Generic programming )。
>- 模板元编程 ( Template metaprogramming，TMP ) 。
>- 数学和数值 ( Math and numerics ) 。
>- 正确性与测试 ( Correctness and testing ) 。
>- 数据结构。
>- 语言间的支持 ( Inter-language support ) 。
>- 内存。
>- 等等

>***小结***
- Boost 是一个社群，也是一个网站。致力于免费、源码开放、同僚复审的 C++ 程序库开发。Boost 在 C++ 标准化过程中扮演深具影响力的角色。
- Boost 提供许多 TR1 组件实现品，以及其他许多程序库。

