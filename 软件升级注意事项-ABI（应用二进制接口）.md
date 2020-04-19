# 软件升级注意事项-ABI（应用二进制借口）

> 在软件开发中，**应用二进制接口**（Application Binary Interface，ABI）是指两程序模块间的接口；通常其中一个程序模块会是库或操作系统所提供的服务，而另一边的模块则是用户所运行的程序。
> 一个ABI定义了机器代码如何访问数据结构与运算程序，此处所定义的界面相当低端并且相依于硬件。而类似概念的API（Application Programming Interface，应用程序接口）则在源代码定义这些，则较为高端，并不直接相依于硬件，通常会是人类可阅读的代码。一个ABI常见的样貌即是调用约定：数据怎么成为计算程序的输入或者从中得到输出；x86的调用约定即是一个ABI的例子。
> 决定要不要采取既定的ABI（不论是否由官方提供），通常由编译器，操作系统或库的开发者来决定；然而，如果撰写一个混合多个编程语言的应用程序，就必须直接处理ABI，采用外部函数调用来达成此目的。
> ——《维基百科》

> A library is binary compatible, if a program linked dynamically to a former version of the library continues running with newer versions of the library without the need to recompile.（如果动态链接到该库的旧版本的程序无需重新编译就可以与该库的新版本一起运行，则该库是二进制兼容的。）

## The Do's and Don'ts

> 以下内容引自《Policies/Binary Compatibility Issues With C++》by KDE Community Wiki.

可以做的：
* 添加新的非虚函数，包括信号、槽和构造函数；
* 添加新的枚举到类中；
* 在已有的枚举中附加新的枚举值；
	* 异常：如果该操作导致编译器为枚举选择了更大的基础类型，则更改将与二进制不兼容。建议增加一个MAX_XXX枚举值，它具有显著的大值（比如255，1<<15等），以创建一个保证可以适合所选的基础类型。
* 如果链接到先前版本动态库的程序调用的事基类的实现，而非派生类的，则可以重新实现在基类层次结构中定义的虚函数（也就是说，在第一个非虚基类中定义的虚函数）。但这依然很危险。
	* 例外：如果重写函数具有**协变返回类型**（a covariant return type）^[参考备注]，则仅当派生类型的指针地址始终与基类类型的指针地址相同时，才是二进制兼容的更改。如果有疑问，请不要使用协变返回类型重写。
* 更改内联函数或使内联函数非内联（如果与库的早期版本链接的程序可以安全地调用旧实现）。这很棘手，可能很危险。做这件事之前要三思。
* 如果私有非虚拟函数没有被任何内联函数调用（而且从未被调用过），删除它们是可行的。
* 如果类中的私有静态成员变量没有被任何内联函数使用，则可以删除它们。
* 添加新的静态数据成员。
* 更改方法的默认参数。只是它需要重新编译才能使用新的默认值。
* 增加新类。
* 导出先前未曾导出过的类。
* 向类添加或移除友元声明。
* rename reserved member types。
* 扩展保留位字段，但前提是这不会导致位字段越过其基础类型的边界（比如char和bool占1个字节，short占2个字节，int占4个字节）。
* 如果一个类继承自QObject，那可以向该类中加入Q_OBJECT宏。
* 添加Q_PROPERTY、Q_ENUMS和Q_FLAGS宏，因为这只修改MOC生成的元对象，不会修改类本身。

不可以做的：
* 对于已经存在的类：
	* 取消导出或移除之前曾导出的类。
	* 以任何方式更改类层次结构（添加、删除或重新排序基类）。
	* 移除final属性。
* 对于模板类：
	* 以任何方式更改模板参数（添加、删除或重新排序）。
* 对于任何类型的现有函数：
	* 不再导出它。
	* 将其移除。
		* 删除现有已声明函数的实现。符号来自于功能的实现，所以这就是有效的函数。（The symbol comes from the implementation of the function, so this is effectively the function.）
	* 使其成为内联函数：包括添加inline关键字，以及不添加inline，而通过将其函数体移到类内。
	* 添加该函数的重载函数（如果该函数已经被重载过，再添加重载也没关系）。
	* 对于静态非私有成员或非静态非成员公共数据
		* 移除或者不再导出它；
		* 改变它的类型；
		* 改变它的CV修饰符（const和volatile）；
	* 对于非静态成员
		* 添加新的数据成员到已有的类中；
		* 改变类中非静态数据数据成员的顺序；
		* 更改成员的类型，签名除外；
		* 从现有类中移除非静态数据成员；
		
如果要从现有函数中添加、扩展或修改参数列表，应该采用添加新函数的方式而不是改变参数列表。在这种情况下，你可能需要添加一个简短的说明，即这两个函数将在库的下一个迭代版本中合并为一个函数：
```
void functionname(int a);
void functionname(int a, int b); //BCI: merge with int b = 0
```

应该做的：
为了使写的类在将来容易扩展，应该遵循下述规则：
* 使用d-pointer，见后面描述；
* 使用非内联的虚析构函数，即使类中没有成员函数和成员变量；
* 在QObject派生类中重新实现事件（reimplement event in QObject-derived classes），即使函数的主体只是调用基类的实现。这样做是为了避免如下所述添加重新实现的虚函数可能引起的问题；
* 所有的构造函数均为非内联；
* 对于拷贝构造函数和赋值操作符采用非内联的实现方式，除非这个类不能按值拷贝（比如，不能复制从QObject继承的类）。

## 库编程者应熟知的技巧

编写库时最大的问题是，无法安全地添加数据成员，因为这将更改包含该类型对象（包括子类）的每个类、结构或数组的大小或布局。

### Bitflags位域

一个例外是bitflags，位域。如果对枚举或布尔值使用位标志，则至少可以安全地舍入到下一个字节减去1。比如包含下列成员的类，添加新成员后并没有破坏二进制兼容性：
```
uint m1 : 1;
uint m2 : 3;
uint m3 : 1;
uint m4 : 2; // new member
```

请将最大值舍入为7位（如果位字段已大于8，则舍入为15位），因为使用最后一位可能会导致某些编译器出现问题。

### 采用d-pointer

位标志和预定义的保留变量很好，但远远不够。这就是d-pointer技术发挥作用的地方。d-pointer的名字来源于Trolteck的Arnt Gulbrandsen，他首先将该技术引入QT，使它成为维护第一个C++ GUI库版本更迭时保持二进制兼容性的技术之一。这项技术很快被每个看到它的人改编为KDE库的**通用编程模式**。**在不破坏二进制兼容性的情况下向类中添加新的私有数据成员是一个很好的技巧。**

注：在计算机科学史上，d-pointer模式曾多次以各种名称被描述，例如pimpl、handle/body或cheshire cat。谷歌可以帮助找到这些描述的在线文章，只要在搜索词中加上C++即可。

在类Foo的类定义中，定义一个前置声明：

```
class FooPrivate;
```

以及在类的内部private成员区域定义：

```
private:
    FooPrivate* d; 
```

类FooPrivate本身纯粹定义在类的实现文件中（通常是.cpp文件），比如：

```
class FooPrivate {
public:
    FooPrivate()
        : m1(0), m2(0)
    {}
    int m1;
    int m2;
    QString s;
};
```

你现在所要做的就是在类Foo构造函数的init函数中，创建FooPrivate：

```
d = new FooPrivate();
```

在析构函数中再释放掉它。

```
delete d;
```

在大多数情况下，您需要使d-pointer为常量，以应对意外修改或复制d-pointer的情况，否则将会失去对私有对象的所有权并可能引发内存泄漏：

```
private:
    FooPrivate* const d;
```

这种方式允许在初始化之后也能修改指针d所指向的内容，但不能修改指针本身。

不过，你可能并不想将所有数据成员都放在private数据对象中。对于经常要使用的一些成员，将其直接放在类的内部，会加快访问速度。因为对于内联函数，它们不能访问d-pointer指向的数据。同时也要注意，虽然d-pointer指向的数据对象中成员被声明为public，但他们依旧为private的，因为d-pointer本身为private。对于需要public或protected访问的成员，需要提供get/set函数。例如：

```
QString Foo::string() const
{
    return d->s;
}

void Foo::setString( const QString& s )
{
    d->s = s;
}
```

另外，也可以（**但不建议**）将d-pointer的私有类声明为嵌套的私有类（例如Foo::Private）。如果使用这种技术，请记住嵌套的私有类将继承包含导出类的公共符号可见性。这将导致私有类的函数将在动态库的符号表中"榜上有名"。你可以通过在嵌套类的私有实现中使用Q_DEC_HIDDEN符号来手动隐藏符号。（*对于现有的类，这在技术上是一个ABI更改，但不会影响KDE开发人员支持的公共ABI，因此错误公开的私有符号可能会在没有进一步警告的情况下被重新隐藏。*） 嵌套似有类的其它缺点还包括无法与QT以及它的Q_D和Q_Q宏保持一致性，无法在不相关的头文件中添加前置声明（比如对于友元想要使用嵌套似有类时）。基于这些原因，我们采用class FooPrivate的方式。

# 备注

1. 协变返回类型（A Covariant Return Type）
在继承关系中，子类覆盖父类的虚函数的时候，必须连返回值类型也完全相同。
```
class Shape {
  public:
    virtual double area() const = 0;
};
class Circle : public Shape {
  public:
    float area() const; // error! different return type    
};

int main() {
  ...
}
```
所以上面这个程序编译是编不过的：
```
main.cpp:9: error: conflicting return type specified for `virtual float Circle::area() const'
main.cpp:5: error:   overriding `virtual double Shape::area() const'
```
但是这个限制在所谓的“Covariant return type”的情况下可以被放松。如果基类的一个虚函数返回值类型是B*，那么其派生类中覆盖这个函数的时候，返回值类型可以是D*，其中D是任何以public方式继承自B的派生类（也就是说D is-a B）。如果基类的虚函数返回B&，派生类覆盖这个函数的时候也可以返回D&。

```
class Shape {
  public:
//    virtual double area() const = 0;
    virtual Shape* clone() const = 0;
};
class Circle : public Shape {
  public:
//    float area() const; // error! different return type
    virtual Circle* clone() const;
};

int main() {
...
}
```
当程序通过指向基类（接口）的指针操纵不同的派生类（实体类）的对象时，派生类方法返回的D*/D&类型的值，可以被自动转换为B*/B&类型。

# 参考链接
1. [应用二进制接口 - 维基百科；](https://zh.wikipedia.org/wiki/%E5%BA%94%E7%94%A8%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%8E%A5%E5%8F%A3)
2. [Policies/Binary Compatibility Issues With C++ - KDE Community Wiki；](https://community.kde.org/Policies/Binary_Compatibility_Issues_With_C%2B%2B)
3. [Itanium C++ ABI；](https://itanium-cxx-abi.github.io/cxx-abi/abi.html)
4. 