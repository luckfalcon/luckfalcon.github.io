# More effective C++ 总结


<!--more-->
# 基础议题
- 指针、引用、类型转换、数组、构造
## 条款 1：指针与引用的区别
- 一个引用必须指向某个对象，任何情况下都不可以使用指向空值的引用。设计不允许变量为空时，可以把变量声明为引用。
- 不存在指向空值的引用这个事实意味着使用引用的代码效率比使用指针的要高。因为在使用引用之前不需要测试它的合法性。
- 指针可以被重新赋值以指向另一个不同的对象，引用则总是指向在初始化时被指定的对象，以后不能改变。
- 使用指针的情况：
>1. 考虑到存在不指向任何对象的可能 ( 在这种情况下，你能够设置指针为空 )。
>2. 需要能够在不同的时刻指向不同的对象 ( 在这种情况下，你能改变指的指向 )。
- 使用引用的情况：
>1. 总是指向一个对象并且一旦指向一个对象后就不会改变指向，那么你应该使用引用。
>2. 当你重载某个操作符时，你应该使用引用。如何操作符[ ]返回对象引用
- ***总结***
>当你知道你必须指向一个对象并且不想改变其指向时, 或者在重载操作符并为防止不必要的语义误解时，你不应该使用指针。而在除此之外的其他情况下，则应使用指针。
## 条款 2：尽量使用 C++ 风格的类型转换
- C++ 四种类型转换操作符：static_cast，const_cast，dynamic_cast，和 reinterpret_cast。
- 使用格式：操作符<要转换的类型>(表达式)-->表达式括号必需。
- reinterpret_cast 使用这个操作符的类型转换，其转换结果几乎都是执行期定义 （ implementation-defined ）。因此 ，使用reinterpret_casts 的代码很难移植。
- ***总结***
> 新的类型转换符缺乏美感才能使它弥补了在含义精确性和可辨认性上的缺点，使用新类型转换符的程序更容易被解析，它们允许编译器检测出原来不能发现的错误。
- ## 条款 3：不要对数组使用多态 
- 通过一个基类指针来删除一个含有派生类对象的数组，结果将是不确定的。
- 在基类内操作基类对象数组时的大小是确定的，而在使用基类指针操作派生类对象数组时则会因派生类对象大小发生改变，从而无法确定对象数组的大小。
```c++
class Base{...}
class Derived : public Base{...}
void Base::printarry(&ostream o, Base arr[], int n)
{
  for(int i = 0; i< n; ++i)
  o<< arr[i];
}
Base b[10];
printarry(cout,b,10);// n 的值是正确的
Derived d[10];
printarry(cout,d,10);//因派生类对象大小发生改变，从而导致 n 的计算错误
```
- 多态和指针算法不能混合在一起来用，所以数组与多态也不能用在一起，因为数组调用的本质是调用指针。
- 是如果你不从一个具体类（concrete classes），派生出另一个具体类，那么你就不太可能犯这种使用多态性数组的错误。
- ## 条款 4：避免无用的缺省构造函数 
- **如果一个类没有缺省构造函数，则无法直接建立该类的对象数组，也不可以使用非堆数组 ( new 数组 )。**
```c++
class Obj
{
  public:
  Obj(int n):num(n){}//没用默认构造函数
  private:
  int num;
}
Obj arr[10];//没有默认构造函数，无法直接建立数组
Obj arr[10] = {Obj(1),Obj(2)...};//可行，不过数组大时不现实
Obj *ptr = new Obj[10]; //没有默认构造函数，无法建立堆数组
```
- 可以指针方式避免调用构造函数。这种方法有两个缺点，第一你必须删除数组里每个指针所指向的对象。如果你忘了，就会发生内存泄漏。第二增加了内存分配量，你需要空间来容纳指针。
```c++
typedef Obj* objarr;
objarr arr[10];//可行，没用调用构造函数
objarr *ptr = new objarr[10];//可行
```
- 为数组分配 raw memory ( 原始内存 )，可以避免浪费内存。使用 placement new 方法在内存中构造对象。
```c++
//operator new 函数
void *rawmemery = operator new( 10*sizeof(Obj));
Obj *best = static_cast<Obj*>(rawmemery);
//使用 placement new
for (int i = 0; i < 10; ++i) 
  new(&best[i])Obj(num);
```
- 使用 placement new 的缺点：当你不想让它继续存在使用时，必须手动调用数组对象的析构函数，然后调用操作符 delete[ ] 来释放 raw memory。
```c++
for (int i = 9; i >= 0; --i)
  best[i].~Obj();               //析构对象
operator delete[](rawmemery);   //回收内存
```
- **对于类里没有定义缺省构造函数所造成的第二个问题是它们无法在许多基于模板（template-based ）的容器类里使用。**
>- 在多数情况下，通过仔细设计模板可以杜绝对缺省构造函数的需求。
- **设计虚基类时所面临的要提供缺省构造函数还是不提供缺省构造函数**
>- 几乎所有的派生类在实例化时都必须给虚基类构造函数提供参数。
>- 这就要求所有由没有缺省构造函数的虚基类继承下来的派生类(无论有多远)都必须知道并理解提供给虚基类构造函数的参数的含义。
- **提供无意义的缺省构造函数也会影响类的工作效率。**
>- 如果成员函数必须测试所有的部分是否都被正确地初始化，那么这些函数的调用者就得为此付出更多的时间。
>- 使用这种（没有缺省构造函数的）类的确有一些限制，但是当你使用它时，它也给你提供了一种保证：你能相信这个类被正确地建立和高效地实现。
- # 运算符
- 重载的运算符何时并且如何被调用，它们如何运作，它们应该如何彼此联系，以及如何获得这些方面的控制权。
- ## 条款 5：谨慎定义类型转换函数
- C++ 编译器能够在两种数据类型之间进行隐式转换（implicit conversions）。
- C++ 编译器会为内置数据类型之间进行自动的隐式转换。继承于 C 语言。
-  自定义类型时，可以控制选择是否提供函数让编译器进行隐式类型转换。
-  有**两种函数**允许编译器进行这些的转换：
>1. 单参数构造函数（ single-argument constructors）。  
>2. 隐式类型转换运算符。即：operator 目标转换type()
>- 注：单参数构造函数是指***只用一个参数即可以调用的构造函数***。该函数可以是只定义了一个参数，也可以是虽定义了多个参数但第一个参数以后的所有参数都有缺省值。
- 可以克服隐式类型转换运算符的缺点 -- 它们的存在将导致错误的发生： 
>- 通过不声明运算符（operator）的方法。 
- 防止编译器不加鉴别地调用单参数构造函数进行类型转换方法：
>1. 构造函数名字前加 explicit 关键字。
>2. 编译器不支持关键字 explicit 时，则不声明单一参数构造函数。
>3. 将单一单数用 proxy classes ( 代理类 ) 代替，可以阻止发生调用单一参数构造函数。 ( **编译器只能进行最多一次隐式类型转换。**)
- ## 条款 6：自增(increment)、自减(decrement)操作符前缀形式与后缀形式的区别 
- 前缀形式返回一个引用，后缀形式返回一个 const 类型。
- C++ 规定后缀形式有一个 int 类型参数，当函数被调用时，编译器传递一个 0 做为 int 参数的值给该函数。
```c++
class Obj
{
  public:
  Obj operator ++ ()//前缀形式,返回对象引用
  {
    *this += 1;
    return *this;
  }
  const Obj opertor ++ ( int )//后缀形式，返回一个const 值,
  {//加 const 是为了与内置类型相一致，内置类型不允许执行连续两次++, 如 i++++
    //返回 const 对象的意义实例
    Obj old = *this;
    *this += 1;
    return old;
  }
}
```
- ## 条款 7：不要重载 “&&”, “||”, 或 “,” 
- C++ 使用**布尔表达式短路求值法**( short-circuit evaluation )。这表示一旦确定了布尔表达式的真假值，即使还有部分表达式没有被测试，布尔表达式也停止运算。
- C++ 语言规范没有定义函数参数的计算顺序。

|不能重载的运算符|
----|
|" . "、 " . * "、 " : : "、 " ? : "、 new、 delete、 sizeof、 typeid、 static_cast、 dynamic_cast、 const_cast、 reinterpret_cast|

- 在遇到 " && ", " || " 和 " , " 时，找到一个好理由是困难的，因为无论你怎么努力，也不能让它们的行为特性与所期望的一样。
- ## 条款 8：理解各种不同含义的 new 和 delete 
- placement new ( 需要 `include<new>`)
>- 在已经被分配但是尚未处理的 ( raw ) 内存中构造一个对象。此时可使用一种特殊的 operator new，被称之为 placement new。[见条款 4 placement new 使用](#条款-4避免无用的缺省构造函数)
- new 操作符（new operator）与 operator new
的关系：
>1. 在堆上建立一个对象，用 new 操作符，既分配内存又为对象调用构造函数。
>2. 仅分配内存，则该调用 operator new 函数；它不会调用构造函数。
>3. 定制自己的在堆对象被建立时的内存分配过程，写你自己的 operator new 函数，然后使用 new 操作符，new 操作符会调用你定制的 operator new。
>4. 在一块已经获得指针的内存里建立一个对象，应该用 placement new。
- Deletion and Memory Deallocation 
>- 为了避免内存泄漏，每个动态内存分配必须与一个等同相反的 deallocation 对应。
- Arrays 
- new 和 delete 操作符是内置的，其行为不受你的控制，凡是它们调用的内存分配和释放函数则可以控制。
- # 异常
- ## 条款 9：使用析构函数防止资源泄漏 
- 