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
	* 改变它的签名，包括：
		* 改变参数列表中参数的类型，包括修改已有参数的const/volatile属性，正确的做法是再添加一个新方法以满足要求。
		* 改变函数的const/volatile修饰符。
		* 改变函数或数据成员的访问权限，比如从private改为public。在一些编译器中，这些信息可能是函数签名的一部分。如果你需要将一个函数的权限从private改为public或者protected，你应该用一个新函数来调用此私有函数。
		* 改变一个成员函数的CV修饰符：只应用于函数本身的const和volatile。
		* 使用另一个参数扩展函数，即使此参数具有默认参数。有关如何避免此问题的建议，请参见下文。
		* 以任何形式改变返回类型。
		* 例外：用外部“C”声明的非成员函数可以更改参数类型（需小心对待）。
	* 对虚函数成员：
		* 在一个没有虚函数或虚基类的类中添加虚函数。
		* 添加新的虚函数到继承体系中的非叶子节点的类中，因为这会破坏它所辖的子类。请注意，在应用程序中被设计为子类的类通常都不是叶子节点的类。请参阅下面的一些解决方法或询问邮件列表。
		* 如果要在Windows上保持二进制兼容，可以出于任何原因添加新的虚拟函数，即使是叶子节点类。**这样做可能会重新排序现有的虚拟函数并破坏二进制兼容性。**
		* 改变类声明中虚函数的顺序。
		* 如果现有虚拟函数不在主基类（第一个非虚拟基类或主基类的主基类及以上）中，重写该函数。
		* 如果重写函数的协变返回类型的指针地址不同于高阶派生类中的类型，重写现有的虚拟函数是不可以的。（通常发生在派生较少的和派生较多的之间，有多重继承或虚拟继承）
		* 移除虚拟函数，即使它是从基类中重新实现的虚拟函数。
	* 对于静态非私有成员或非静态非成员公共数据
		* 移除或者不再导出它；
		* 改变它的类型；
		* 改变它的CV修饰符（const和volatile）；
	* 对于非静态成员
		* 添加新的数据成员到已有的类中；
		* 改变类中非静态数据数据成员的顺序；
		* 更改成员的类型，签名除外；
		* 从现有类中移除非静态数据成员；
		
如果要从现有函数中添加、扩展或修改参数列表，应该采用添加新函数的方式而不是改变参数列表。

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