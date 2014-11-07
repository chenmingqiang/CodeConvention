# 1. 头文件
一般一个.cpp文件对应一个.h文件。除了单元测试代码和包含main()函数的cpp文件。下面是使用头文件的各种规则
## 1.1 条件预处理器保护
----------------------------------------------------
所有的头文件都应该使用＃define防止头文件被多重包含，命名格式为：<project>_<path>_<file>_H_  

为保证唯一性, 头文件的命名应该依据所在项目源代码树的全路径。例如,项目foo中的头文件foo/src/bar/baz.h 可按如下方式保护:

	#ifndef FOO_BAR_BAZ_H_    
	#define FOO_BAR_BAZ_H_    
	…  
	#endif // FOO_BAR_BAZ_H_  
	
## 1.2 头文件依赖
----------------------------------------------------
能用前置声明的地方尽量不使用#include。因为当一个头文件被#include就相当于引入了依赖，一旦这个头文件被修改，所有包含了此文件的代码都需要重新编译。因此，倾向于减少包含头文件，尤其是头文件中包含头文件。

使用前置声明能够有效减少需要包含的文件数量。但是前置声明只能在不访问类的实现时可以使用，那些操作属于不访问类的实现了。我们已类Foo为例

1. 数据成员类型声明为指针Foo *或者者引用Foo &
2. 将函数的参数或返回值定义为Foo,但不能实现（当然，参数或返回值是Foo *, Foo &也可）
3. 类的静态数据成员类型为Foo，因为静态数据成员的定义在类定义之外。

下列情况则必须包含Foo类定义的头文件。

1. 类是Foo类的子类
2. 包含Foo类型的非静态成员

## 1.3 内联函数
----------------------------------------------------
只有当函数在10行以类才定义为内联函数，而且内联函数应该做很少的事情，对于递归函数，包含复杂的循环或switch的函数都不能定义为内联函数。对于get/set这类函数比较适合于定义为内联函数。


## 1.4 头文件包含方式
----------------------------------------------------
标准库头文件使用尖括号<>， 非标准库头文件使用双引号“”，例子

	#include <iostream>
	#include <string>
	#include "foo.h"
   
## 1.5 头文件包含顺序
----------------------------------------------------

使用标准的头文件包含顺序可增强可读性, 避免隐藏依赖: C库, C++库, 其他库的.h, 本项目内的.h. 在同一类型的头文件可以按字母序排列。

举例来说, google-awesome-project/src/foo/internal/fooserver.cc 的包含次序如下:

	#include "foo/public/fooserver.h" // 优先位置
	#include <sys/types.h> // C 库
	#include <unistd.h>    // C 库
	#include <hash_map>    // C++ 库
	#include <vector>      // C++ 库
	#include "base/basictypes.h" // 其他库
	#include "base/commandlineflags.h" // 其他库
	#include "foo/public/bar.h"        // 本项目


## 1.6 头文件中包含什么
----------------------------------------------------
因为头文件包含在多个文件中，所以不应该含有变量和函数的的定义，否则会出现重复定义的错误。对于头文件有三个例外，可以在头文件中定义类，值在编译时确定的const 变量，以及内联函数。这些可以在多个源文件中定义，只要定义相同就行了。

如果const 定义一个常量，那么可以被包含在头文件中。typedef用于定义类型别名，因此也可以放在头文件中，枚举定义也应该放在头文件中。

# 2. 作用域
## 2.1 名字空间
------------------------------------------------------
鼓励在.cpp 文件中使用匿名名字空间。禁止使用using namespace std; 这种方式来引入名字。 可以使用如下方式；

	using std::string;
	using std::vector;
	
	std::string foo(std::vector<std::string>);
	
## 2.2 嵌套类
------------------------------------------------------

## 2.3 非成员函数、静态成员函数和全局函数
------------------------------------------------------
使用静态成员函数或名字空间内的非成员函数, 尽量不要用裸的全局函数.

如果你必须定义非成员函数, 又只是在.cpp文件中使用它, 可使用匿名名字空间或static 链接关键字 (如 static int Foo() {...})限定其作用域.

## 2.4 局部变量
------------------------------------------------------
将函数含量尽可能放在最小作用域，并在变量声明时进行初始化。
在C＋＋中可以在函数的任意位置声明变量。应当在离使用最近的地方声明变量，最好采用初始化的方式定义一个变量。避免先声明再赋值。
 
	int j = g(); // good
	int i;
	i = f();  // bad
	
注意，不能过度使用这个规则，使用时应当考虑程序性能。比如如下例子

低效的实现

	for (int i = 0; i < 1000000; ++i) {
    	Foo f;    // 构造函数和析构函数分别调用 1000000 次!
    	f.DoSomething(i);
	}
	
高效的实现

	Foo f;        // 构造函数和析构函数只调用 1 次
	for (int i = 0; i < 1000000; ++i) {
    	f.DoSomething(i);
	}
	
	
## 2.5 静态和全局变量
------------------------------------------------------
禁止使用class类型的静态或全局变量: 它们会导致很难发现的bug和不确定的构造和析构函数调用顺序.

静态生存周期的对象, 包括全局变量, 静态变量, 静态类成员变量, 以及函数静态变量, 都必须是原生数据类型 (POD : Plain Old Data): 只能是 int, char, float, 和 void, 以及 POD 类型的数组/结构体/指针. 永远不要使用函数返回值初始化静态变量; 不要在多线程代码中使用非const的静态变量.

不幸的是, 静态变量的构造函数, 析构函数以及初始化操作的调用顺序在 C++ 标准中未明确定义, 甚至每次编译构建都有可能会发生变化, 从而导致难以发现的 bug. 比如, 结束程序时, 某个静态变量已经被析构了, 但代码还在跑 – 其它线程很可能 – 试图访问该变量, 直接导致崩溃.

所以, 我们只允许 POD 类型的静态变量. 本条规则完全禁止 vector (使用 C 数组替代), string (使用 const char*), 及其它以任意方式包含或指向类实例的东东, 成为静态变量. 出于同样的理由, 我们不允许用函数返回值来初始化静态变量.

如果你确实需要一个 class类型的静态或全局变量, 可以考虑在main() 函数或 pthread_once() 内初始化一个你永远不会回收的指针.

# 3. 类
类是C＋＋代码中的基本单元。

## 3.1 构造函数
----------------------------------------------------
构造函数中只进行那些没什么意义的 (trivial, YuleFox 注: 简单初始化对于程序执行没有实际的逻辑意义, 因为成员变量 “有意义” 的值大多不在构造函数中确定) 初始化, 可能的话, 使用 Init() 方法集中初始化有意义的 (non-trivial) 数据.

下面构造函数不能过多的做事情的原因。

1. 构造函数中很难上报错误, 不能使用异常.
2. 操作失败会造成对象初始化失败，进入不确定状态.
3. 如果在构造函数内调用了自身的虚函数, 这类调用是不会重定向到子类的虚函数实现. 即使当前没有子类化实现, 将来仍是隐患.
4. 如果有人创建该类型的全局变量 (虽然违背了上节提到的规则), 构造函数将先 main() 一步被调用, 有可能破坏构造函数中暗含的假设条件. 例如, gflags 尚未初始化.

## 3.2 默认构造函数
----------------------------------------------------
如果一个类定义了若干成员变量又没有其它构造函数, 必须定义一个默认构造函数。 否则编译器将自动生产一个很糟糕的默认构造函数.如果你定义的类继承现有类, 而你又没有增加新的成员变量, 则不需要为新类定义默认构造函数。

## 3.3 显示构造函数
----------------------------------------------------
所有**单参数构造函数**都必须是显式的. 在类定义中, 将关键字*explicit*加到单参数构造函数前: explicit Foo(string name); 添加*explicit*为了避免隐式转化。

例外: 在极少数情况下, 拷贝构造函数可以不声明成explicit. 作为其它类的透明包装器的类也是特例之一. 类似的例外情况应在注释中明确说明.

## 3.4 拷贝构造函数
----------------------------------------------------
仅在代码中需要拷贝一个类对象的时候使用拷贝构造函数; 大部分情况下都不需要, 此时应使用 DISALLOW_COPY_AND_ASSIGN.

这个宏的定义如下

	// 禁止使用拷贝构造函数和 operator= 赋值操作的宏
	// 应该类的 private: 中使用

	#define DISALLOW_COPY_AND_ASSIGN(TypeName) \
            TypeName(const TypeName&); \
            void operator=(const TypeName&)


绝大多数情况下都应使用 DISALLOW_COPY_AND_ASSIGN宏。如果类确实需要可拷贝, 应在该类的头文件中说明原由, 并合理的定义拷贝构造函数和赋值操作. 注意在operator= 中检测自我赋值的情况。  

为了能作为 STL 容器的值, 你可能有使类可拷贝的冲动. 在大多数类似的情况下, 真正该做的是把对象的 指针 放到 STL 容器中. 可以考虑使用 boost::shared_ptr

## 3.5 结构体
---------------------------------------------------
仅当只有数据时使用 struct, 其它一概使用 class.
在C++中struct 和 class 关键字几乎含义一样. 我们为这两个关键字添加我们自己的语义理解, 以便为定义的数据类型选择合适的关键字.

struct 用来定义包含数据的被动式对象, 也可以包含相关的常量, 但除了存取数据成员之外, 没有别的函数功能. 并且存取功能是通过直接访问位域 (field),而非函数调用. 除了构造函数, 析构函数, Initialize(), Reset(), Validate() 外, 不能提供其它功能的函数.

如果需要更多的函数功能, class更适合. 如果拿不准就用class.

为了和STL保持一致, 对于仿函数(functors)和特性(traits)可以不用 class而是使用struct.

注意: 类和结构体的成员变量使用不同的命名规则.

## 3.6 继承
--------------------------------------------------
使用组合常常比使用继承更合理。 如果使用继承的话定义为*public*继承.

所有继承必须是**public**的. 如果你想使用私有继承,你应该替换成把基类的实例作为成员对象的方式.

不要过度使用实现继承. 组合常常更合适一些.尽量做到只在 “是一个” (“is-a”, YuleFox 注: 其他 “has-a” 情况下请使用组合) 的情况下使用继承: 如果 Bar 的确 “是一种” Foo, Bar 才能继承 Foo.

**析构函数声明为virtual. 如果你的类有虚函数,则析构函数也应该为虚函数. 注意数据成员在任何情况下都必须是私有的.
**

当重载一个虚函数, 在子类中把它明确的声明为virtual。理论依据: 如果省略 virtual 关键字, 代码阅读者不得不检查所有父类, 以判断该函数是否是虚函数.
## 3.7 接口
--------------------------------------------------------
接口是指满足特定条件的类,这些类以Interface为后缀(不强制)
满足以下条件才成为纯接口

1. 只有纯虚函数 (“=0”) 和静态函数 (除了下文提到的析构函数).
2. 没有非静态数据成员.
3. 没有定义任何构造函数. 如果有, 也不能带有参数, 并且必须为protected.
4. 如果它是一个子类, 也只能从满足上述条件并以 Interface 为后缀的类继承.

## 3.8 声明顺序
-------------------------------------------------------

在类中使用特定的声明顺序: public: 在 private: 之前, 成员函数在数据成员(变量)前;
类的访问控制区段的声明顺序依次为: public:, protected:, private:. 如果某区段没内容, 可以不声明.

每个区段内的声明通常按以下顺序:

typedefs 和枚举
常量
构造函数
析构函数
成员函数, 含静态成员函数
数据成员, 含静态数据成员
宏 DISALLOW_COPY_AND_ASSIGN 的调用放在 private: 区段的末尾. 它通常是类的最后部分.

.cc 文件中函数的定义应尽可能和声明顺序一致.

不要在类定义中内联大型函数. 通常, 只有那些没有特别意义或性能要求高, 并且是比较短小的函数才能被定义为内联函数.

## 3.9 编写短函数
-------------------------------------------------------
倾向编写简短, 凝练的函数.
我们承认长函数有时是合理的, 因此并不硬性限制函数的长度. 如果函数超过 40 行, 可以思索一下能不能在不影响程序结构的前提下对其进行分割.

即使一个长函数现在工作的非常好, 一旦有人对其修改, 有可能出现新的问题. 甚至导致难以发现的 bug. 使函数尽量简短, 便于他人阅读和修改代码.

在处理代码时, 你可能会发现复杂的长函数. 不要害怕修改现有代码: 如果证实这些代码使用 / 调试困难, 或者你需要使用其中的一小段代码, 考虑将其分割为更加简短并易于管理的若干函数.

# 4. 命名约定

最重要的一致性规则是命名管理。 命名风格快速获知名字代表是什么东东: 类型? 变量? 函数? 常量? 宏 ... ? 甚至不需要去查找类型声明。我们大脑中的模式匹配引擎可以非常可靠的处理这些命名规则.

命名规则具有一定随意性, 但相比按个人喜好命名, 一致性更重, 所以不管你怎么想, 规则总归是规则.

## 4.1. 通用命名规则
--------------------------------------------------------

函数命名, 变量命名, 文件命名应具备描述性; 不要过度缩写。类型和变量应该是名词, 函数名可以用 “命令性” 动词.
如何命名:尽可能给出描述性的名称. 不要节约行空间, 让别人很快理解你的代码更重要。好的命名风格:

	int num_errors;                  // Good.
	int num_completed_connections;   // Good.
	
糟糕的命名使用含糊的缩写或随意的字符:

	int n;                           // Bad - meaningless.
	int nerr;                        // Bad - ambiguous abbreviation.
	int n_comp_conns;                // Bad - ambiguous abbreviation.
类型和变量名一般为名词: 如 FileOpener, num_errors.

函数名通常是指令性的 (确切的说它们应该是命令), 如 OpenFile(), set_num_errors(). 取值函数是个特例 (在 函数命名 处详细阐述), 函数名和它要取值的变量同名.
缩写:除非该缩写在其它地方都非常普遍, 否则不要使用. 例如:

	// Good
	// These show proper names with no abbreviations.
	int num_dns_connections;  // 大部分人都知道 "DNS" 是啥意思.
	int price_count_reader;   // OK, price count. 有意义.

	// Bad!
	// Abbreviations can be confusing or ambiguous outside a small group.
	int wgc_connections;  // Only your group knows what this stands for.
	int pc_reader;        // Lots of things can be abbreviated "pc".
	
永远不要用省略字母的缩写:

	int error_count;  // Good.
	int error_cnt;    // Bad.
	
## 4.2. 文件命名
--------------------------------------------------------

文件名要全部小写, 可以包含下划线 (_) 或连字符 (-). 按项目约定来.
可接受的文件命名:

	my_useful_class.cc
	my-useful-class.cc
	myusefulclass.cc
	
C++ 文件要以.cc结尾, 头文件以.h结尾.

不要使用已经存在于 /usr/include下的文件名.

通常应尽量让文件名更加明确. http_server_logs.h 就比 logs.h 要好. 定义类时文件名一般成对出现, 如 foo_bar.h 和 foo_bar.cc, 对应于类 FooBar.

内联函数必须放在 .h 文件中. 如果内联函数比较短, 就直接放在 .h 中. 如果代码比较长, 可以放到以 -inl.h 结尾的文件中. 对于包含大量内联代码的类, 可以使用三个文件:

	url_table.h      // The class declaration.
	url_table.cc     // The class definition.
	url_table-inl.h  // Inline functions that include lots of code.

## 4.3. 类型命名
--------------------------------------------------------

类型名称的每个单词首字母均大写, 不包含下划线: MyExcitingClass, MyExcitingEnum.
所有类型命名 —— 类, 结构体, 类型定义 (typedef), 枚举 —— 均使用相同约定. 例如:

	// classes and structs
	class UrlTable { ...
	class UrlTableTester { ...
	struct UrlTableProperties { ...
	
	// typedefs
	typedef hash_map<UrlTableProperties *, string> PropertiesMap;
	
	// enums
	enum UrlTableErrors { ...
	
## 4.4. 变量命名
--------------------------------------------------------

变量名一律小写, 单词之间用下划线连接。类的成员变量以下划线结尾, 如:

	my_exciting_local_variable
	my_exciting_member_variable_
	
普通变量命名:

	string table_name;  // OK - uses underscore.
	string tablename;   // OK - all lowercase.
	string tableName;   // Bad - mixed case.
	
结构体变量:结构体的数据成员可以和普通变量一样, 不用像类那样接下划线:

	struct UrlTableProperties {
    	string name;
    	int num_entries;
	}

全局变量:对全局变量没有特别要求, 少用就好, 但如果你要用, 可以用 g_ 或其它标志作为前缀, 以便更好的区分局部变量.

## 4.5 常量命名
--------------------------------------------------------

在名称前加 k: kDaysInAWeek.
所有编译时常量, 无论是局部的, 全局的还是类中的, 和其他变量稍微区别一下. k 后接大写字母开头的单词::

const int kDaysInAWeek = 7;

对于字面值常量，比如定义一个长整形数时，应该用“L”作为后缀，定义无符号可用U作为后缀。

## 4.6. 函数命名
--------------------------------------------------------

常规函数使用大小写混合, 取值和设值函数则要求与变量名匹配: MyExcitingFunction(), MyExcitingMethod(), my_exciting_member_variable(), set_my_exciting_member_variable().
常规函数:
函数名的每个单词首字母大写, 没有下划线:

AddTableEntry()
DeleteUrl()
取值和设值函数:
取值和设值函数要与存取的变量名匹配. 这儿摘录一个类, num_entries_ 是该类的实例变量:

	class MyClass {
	  public:
	    ...
	    int num_entries() const { return num_entries_; }
	    void set_num_entries(int num_entries) { num_entries_ = num_entries; }
	
	  private:
	    int num_entries_;
	};
	
其它非常短小的内联函数名也可以用小写字母, 例如. 如果你在循环中调用这样的函数甚至都不用缓存其返回值, 小写命名就可以接受

## 4.7. 名字空间命名
--------------------------------------------------------

名字空间用小写字母命名, 并基于项目名称和目录结构: google_awesome_project.

## 4.8. 枚举命名
--------------------------------------------------------

枚举的命名应当和常量或宏一致: kEnumName 或是 ENUM_NAME.
单独的枚举值应该优先采用常量的命名方式. 但宏方式的命名也可以接受. 枚举名 UrlTableErrors (以及 AlternateUrlTableErrors) 是类型, 所以要用大小写混合的方式.

	enum UrlTableErrors {
	    kOK = 0,
	    kErrorOutOfMemory,
	    kErrorMalformedInput,
	};
	
	enum AlternateUrlTableErrors {
	    OK = 0,
	    OUT_OF_MEMORY = 1,
	    MALFORMED_INPUT = 2,
	};
	
由于枚举值和宏之间的命名冲突, 直接导致了很多问题. 由此, 这里改为优先选择常量风格的命名方式. 

## 4.9. 宏命名
--------------------------------------------------------

尽量不要使用宏，如果你一定要用, 像这样命名: MY_MACRO_THAT_SCARES_SMALL_CHILDREN.
参考预处理宏 <preprocessor-macros>; 通常不应该使用宏. 如果不得不用, 其命名像枚举命名一样全部大写, 使用下划线:

	#define ROUND(x) ...
	#define PI_ROUNDED 3.0


# 5. 注释
## 5.1 注释风格保持一致
--------------------------------------------------------

C＋＋提供了两种注释，行注释//和块注释/**/; 但 //更常用。因为块注释不能嵌套使用。总之要在如何注释及注释风格上确保统一。

## 5.2 文件注释
--------------------------------------------------------

在每一个文件开头加入版权公告, 然后是文件内容描述。  
法律公告和作者信息:  
每个文件都应该包含以下项, 依次是:

版权声明 (比如, Copyright 2008 Google Inc.)
许可证. 为项目选择合适的许可证版本 (比如, Apache 2.0, BSD, LGPL, GPL)
作者: 标识文件的原始作者。如果你对原始作者的文件做了重大修改, 将你的信息添加到作者信息里. 这样当其他人对该文件有疑问时可以知道该联系谁。
文件内容:紧接着版权许可和作者信息之后, 每个文件都要用注释描述文件内容。

通常, .h 文件要对所声明的类的功能和用法作简单说明. .cc 文件通常包含了更多的实现细节或算法技巧讨论, 如果你感觉这些实现细节或算法技巧讨论对于理解 .h 文件有帮助, 可以该注释挪到 .h, 并在 .cc 中指出文档在 .h.

不要简单的在 .h 和 .cc 间复制注释. 这种偏离了注释的实际意义。

## 5.3 类注释
--------------------------------------------------------
每个类的定义都要附带一份注释, 描述类的功能和用法.

	// Iterates over the contents of a GargantuanTable.  Sample usage:
	//    GargantuanTable_Iterator* iter = table->NewIterator();
	//    for (iter->Seek("foo"); !iter->done(); iter->Next()) {
	//      process(iter->key(), iter->value());
	//    }
	//    delete iter;
	class GargantuanTable_Iterator {
    	...
	};
	
如果你觉得已经在文件顶部详细描述了该类, 想直接简单的来上一句 “完整描述见文件顶部” 也不打紧, 但务必确保有这类注释.如果类有任何同步前提, 文档说明之. 如果该类的实例可被多线程访问, 要特别注意文档说明多线程环境下相关的规则和常量使用.


## 5.4 函数注释
--------------------------------------------------------
函数声明处注释描述函数功能; 定义处描述函数实现.
函数声明:注释位于声明之前, 对函数功能及用法进行描述. 注释使用叙述式 (“Opens the file”) 而非指令式 (“Open the file”); 注释只是为了描述函数, 而不是命令函数做什么. 通常, 注释不会描述函数如何工作. 那是函数定义部分的事情.

函数声明处注释的内容:

* 函数的输入输出.
* 对类成员函数而言: 函数调用期间对象是否需要保持引用参数, 是否会释放这些参数.
* 如果函数分配了空间, 需要由调用者释放.
* 参数是否可以为 NULL.
* 是否存在函数使用上的性能隐患.

如果函数是可重入的, 其同步前提是什么?
举例如下:

	// Returns an iterator for this table.  It is the client's
	// responsibility to delete the iterator when it is done with it,
	// and it must not use the iterator once the GargantuanTable object
	// on which the iterator was created has been deleted.
	//
	// The iterator is initially positioned at the beginning of the table.
	//
	// This method is equivalent to:
	//    Iterator* iter = table->NewIterator();
	//    iter->Seek("");
	//    return iter;
	// If you are going to immediately seek to another place in the
	// returned iterator, it will be faster to use NewIterator()
	// and avoid the extra seek.
	Iterator* GetIterator() const;

不好的风格

	// Returns true if the table cannot hold any more entries.
	bool IsTableFull();
	
注释构造/析构函数时, 切记读代码的人知道构造/析构函数是干啥的, 所以 “destroys this object” 这样的注释是没有意义的. 注明构造函数对参数做了什么 (例如, 是否取得指针所有权) 以及析构函数清理了什么. 如果都是些无关紧要的内容, 直接省掉注释. 析构函数前没有注释是很正常的.
函数定义:每个函数定义时要用注释说明函数功能和实现要点. 比如说说你用的编程技巧, 实现的大致步骤, 或解释如此实现的理由, 为什么前半部分要加锁而后半部分不需要.

不要从.h 文件或其他地方的函数声明处直接复制注释. 简要重述函数功能是可以的, 但注释重点要放在如何实现上.

## 5.5 变量注释
--------------------------------------------------------
通常变量名本身足以很好说明变量用途. 某些情况下, 也需要额外的注释说明.
类数据成员:每个类数据成员 (也叫实例变量或成员变量) 都应该用注释说明用途. 如果变量可以接受 NULL 或 -1 等警戒值, 须加以说明. 比如:

	private:
      // Keeps track of the total number of entries in the table.
      // Used to ensure we do not go over the limit. -1 means
      // that we don't yet know how many entries the table has.
      int num_total_entries_;
      
全局变量:和数据成员一样, 所有全局变量也要注释说明含义及用途. 比如:

	// The total number of tests cases that we run through in this regression test.
	const int kNumTestCases = 6;
	
## 5.6 实现注释
--------------------------------------------------------
对于代码中巧妙的, 晦涩的, 有趣的, 重要的地方加以注释.
代码前注释:巧妙或复杂的代码段前要加注释. 比如:

	// Divide result by two, taking into account that x
	// contains the carry from the add.
	for (int i = 0; i < result->size(); i++) {
    	x = (x << 8) + (*result)[i];
    	(*result)[i] = x >> 1;
    	x &= 1;
	}
	
行注释:比较隐晦的地方要在行尾加入注释. 在行尾空两格进行注释. 比如:

	// If we have enough memory, mmap the data portion too.
	mmap_budget = max<int64>(0, mmap_budget - index_->length());
	if (mmap_budget >= data_size_ && !MmapData(mmap_chunk_bytes, mlock))
    	return;  // Error already logged.
    	
注意, 这里用了两段注释分别描述这段代码的作用, 和提示函数返回时错误已经被记入日志.

如果你需要连续进行多行注释, 可以使之对齐获得更好的可读性:

	DoSomething();                  // Comment here so the comments line up.
	DoSomethingElseThatIsLonger();  // Comment here so there are two spaces between
									 // the code and the comment.
	{ // One space before comment when opening a new scope is allowed,
	  // thus the comment lines up with the following comments and code.
	  DoSomethingElse();  // Two spaces before line comments normally.
	}
	
NULL, true/false, 1, 2, 3...: 向函数传入 NULL, 布尔值或整数时, 要注释说明含义, 或使用常量让代码望文知意. 例如, 对比:

**不好的风格**

	bool success = CalculateSomething(interesting_value,
                                      10,
                                      false,
                                      NULL);  // What are these arguments??
 **好的风格**

	bool success = CalculateSomething(interesting_value,
                                      10,     // Default base value.
                                      false,  // Not the first time we're calling this.
                                      NULL);  // No callback.
或使用常量或描述性变量:

	const int kDefaultBaseValue = 10;
	const bool kFirstTimeCalling = false;
	Callback *null_callback = NULL;
	bool success = CalculateSomething(interesting_value,
                                  	   kDefaultBaseValue,
                                      kFirstTimeCalling,
                                      null_callback);
                                  
注意永远不要用自然语言翻译代码作为注释. 要假设读代码的人 C++ 水平比你高, 即便他/她可能不知道你的用意。好的命名比注释更重要。 因为好的命名能够起到注释的作用。

## 5.7 TODO 注释
-----------------------------------------------
对那些临时的、短期的解决方案，或已经够好但仍不完美的代码使用TODO注释.  
TODO 注释要使用全大写的字符串**TODO**, 在随后的圆括号里写上你的大名, 邮件地址, 或其它身份标识。冒号是可选的。主要目的是让添加注释的人 (也是可以请求提供更多细节的人) 可根据规范的 TODO 格式进行查找。添加 TODO 注释并不意味着你要自己来修正.

	// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
	// TODO(Zeke) change this to use relations.

如果加 TODO 是为了在 “将来某一天做某事”, 可以附上一个非常明确的时间 “Fix by November 2005”), 或者一个明确的事项 (“Remove this code when all clients can handle XML responses.”).

# 6. 格式

## 6.1 行长度
--------------------------------------------------------
每一行代码字符数不超过 80.我们也认识到这条规则是有争议的, 但很多已有代码都已经遵照这一规则, 我们感觉一致性更重要.

特例:如果一行注释包含了超过 80 字符的命令或 URL, 出于复制粘贴的方便允许该行超过 80 字符.
包含长路径的#include 语句可以超出80列. 但应该尽量避免.头文件保护可以无视该原则.

## 6.2 文件编码
--------------------------------------------------------
尽量不使用非 ASCII 字符, 使用时必须使用 UTF-8 编码.
即使是英文, 也不应将用户界面的文本硬编码到源代码中, 因此非 ASCII 字符要少用. 特殊情况下可以适当包含此类字符. 如, 代码分析外部数据文件时, 可以适当硬编码数据文件中作为分隔符的非 ASCII 字符串; 更常见的是 (不需要本地化的) 单元测试代码可能包含非 ASCII 字符串. 此类情况下, 应使用 UTF-8 编码, 因为很多工具都可以理解和处理 UTF-8 编码. 十六进制编码也可以, 能增强可读性的情况下尤其鼓励 —— 比如 "\xEF\xBB\xBF" 在 Unicode 中是 零宽度 无间断 的间隔符号, 如果不用十六进制直接放在 UTF-8 格式的源文件中, 是看不到的. 

## 6.3 缩进
--------------------------------------------------------
只使用空格, 每次缩进4个空格.我们使用空格缩进.不要在代码中使用制符表.你应该设置编辑器将制符表转为空格.


## 6.4 函数声明与定义
--------------------------------------------------------

返回类型和函数名在同一行, 参数也尽量放在同一行.
函数看上去像这样:

	ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
    	DoSomething();
    	...
	}
	
如果同一行文本太多, 放不下所有参数:

	ReturnType ClassName::ReallyLongFunctionName(Type par_name1,
                                             Type par_name2,
                                             Type par_name3) {
    	DoSomething();
    	...
	}
	
甚至连第一个参数都放不下:

	ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
        	Type par_name1,  // 8 space indent
        	Type par_name2,
        	Type par_name3) {
        DoSomething();  // 4 space indent
    	...
	}
	
注意以下几点:

* 返回值总是和函数名在同一行;
* 左圆括号总是和函数名在同一行;
* 函数名和左圆括号间没有空格;
* 圆括号与参数间没有空格;
* 左大括号总在最后一个参数同一行的末尾处;
* 右大括号总是单独位于函数最后一行;
* 右圆括号和左大括号间总是有一个空格;
* 函数声明和实现处的所有形参名称必须保持一致;
* 所有形参应尽可能对齐;
* 缺省缩进为4个空格;
* 换行后的参数保持8个空格的缩进;

如果函数声明成 const, 关键字 const 应与最后一个参数位于同一行

	// Everything in this function signature fits on a single line
	ReturnType FunctionName(Type par) const {
		...
	}

	// This function signature requires multiple lines, but
	// the const keyword is on the line with the last parameter.
	ReturnType ReallyLongFunctionName(Type par1,
                                  Type par2) const {
        ...
	}
	
如果有些参数没有用到, 在函数定义处将参数名注释起来:

	// Always have named parameters in interfaces.
	class Shape {
	  public:
	  	virtual void Rotate(double radians) = 0;
	}

	// Always have named parameters in the declaration.
	class Circle : public Shape {
	  public:
        virtual void Rotate(double radians);
	}

	// Comment out unused named parameters in definitions.
	void Circle::Rotate(double /*radians*/) {}


## 6.5 函数调用
--------------------------------------------------------
尽量放在同一行, 否则, 将实参封装在圆括号中.函数调用遵循如下形式:

	bool retval = DoSomething(argument1, argument2, argument3);
	
如果同一行放不下, 可断为多行, 后面每一行都和第一个实参对齐, 左圆括号后和右圆括号前不要留空格:

	bool retval = DoSomething(averyveryveryverylongargument1,
                      	      argument2, argument3);
                      	      
如果函数参数很多, 出于可读性的考虑可以在每行只放一个参数:

	bool retval = DoSomething(argument1,
    	                      argument2,
        	                  argument3,
            	              argument4);
            	              
如果函数名非常长, 以至于超过 行最大长度, 可以将所有参数独立成行:

	if (...) {
		...
		...
		if (...) {
    		DoSomethingThatRequiresALongFunctionName(
        		very_long_argument1,  // 4 space indent
        		argument2,
        		argument3,
        		argument4);
        }
	 }
        	


## 6.6 条件语句
-------------------------------------------------------
倾向于不在圆括号内使用空格. 关键字else 另起一行.对基本条件语句有两种可以接受的格式. 一种在圆括号和条件之间有空格, 另一种没有.最常见的是没有空格的格式. 哪种都可以, 但保持一致性. 如果你是在修改一个文件, 参考当前已有格式. 如果是写新的代码, 参考目录下或项目中其它文件. 还在徘徊的话, 就不要加空格了.

	if (condition) {  // no spaces inside parentheses
		...  // 4 space indent.
	} else {  // The else goes on the same line as the closing brace.
		...
	}
	
如果你更喜欢在圆括号内部加空格:

	if ( condition ) {  // spaces inside parentheses - rare
		...    // 4 space indent.
	} else {   // The else goes on the same line as the closing brace.
		...
	}
	
注意所有情况下 if 和左圆括号间都有个空格. 右圆括号和左大括号之间也要有个空格:
不好的风格：

	if(condition)     // Bad - space missing after IF.
	if (condition){   // Bad - space missing before {.
	if(condition){    // Doubly bad.
	if (condition) {  // Good - proper space after IF and before {.
	
如果能增强可读性, 简短的条件语句允许写在同一行. 只有当语句简单并且没有使用 else 子句时使用:

	if (x == kFoo) return new Foo();
	if (x == kBar) return new Bar();
	
如果语句有 else 分支则不允许:

**不好的风格**

	// Not allowed - IF statement on one line when there is an ELSE clause
	if (x) DoThis();
	else DoThat();
	
通常, 单行语句不需要使用大括号, 如果你喜欢用也没问题; 复杂的条件或循环语句用大括号可读性会更好. 推荐即使单行语句也用总是使用大括号:

	if (condition)
    	DoSomething();  // 4 space indent.

    // better 
	if (condition) {
    	DoSomething();  // 4 space indent.
	}
	
但如果语句中某个 if-else 分支使用了大括号的话, 其它分支也必须使用:

**不好的风格**

	// Not allowed - curly on IF but not ELSE
	if (condition) {
    	foo;
	} else
    	bar;

	// Not allowed - curly on ELSE but not IF
	if (condition)
    	foo;
	else {
    	bar;
	}
	
**好的风格**

	// Curly braces around both IF and ELSE required because
	// one of the clauses used braces.
	if (condition) {
		foo;
	} else {
		bar;
	}

## 6.7 循环和开关语句

switch 语句可以使用大括号分段. 空循环体应使用 {} 或 continue.
switch 语句中的 case 块可以使用大括号也可以不用, 取决于你的个人喜好,推荐每个case后面加上break. 如果用的话, 要按照下文所述的方法.

如果有不满足 case 条件的枚举值, switch 应该总是包含一个 default 匹配 (如果有输入值没有 case 去处理, 编译器将报警). 如果 default 应该永远执行不到, 简单的加条 assert:

	switch (var) {
	  case 0: {  // 2 space indent
        ...      // 4 space indent
    	break;
      }
      case 1: {
        ...
        break;
      }
      default: {
        assert(false);
      }
    }

空循环体应使用 {} 或 continue, 而不是一个简单的分号.

**好的风格**

	while (condition) {
		// Repeat test until it returns false.
	}
	for (int i = 0; i < kSomeNumber; ++i) {}  // Good - empty body.
	while (condition) continue;  // Good - continue indicates no logic.
	
**不好的风格**

	while (condition);  // Bad - looks like part of do/while loop.
	
无限循环应该使用如下风格
  
     while (1) {
     
     }
     
     for (;;) {
     
     }

 对于计数或者迭代器循环，推荐使用不等号!=来终止循环而不是用<,>, <=, >=。
 
	for (vector<int>::iterator iter = ivec.begin(); iter != ivec.end(); ++iter) {
		...
		doSomething();
		...
	}
	
## 6.8 指针和引用表达式
--------------------------------------------------------
句点或箭头前后不要有空格. 指针/地址操作符 (*, &) 之后不能有空格.
下面是指针和引用表达式的正确使用范例:

	x = *p;    
	p = &x;

在访问成员时, 句点或箭头前后没有空格.

	x = r.y;   
	x = r->y;  
	

在声明指针变量或参数时, 星号与类型或变量名紧挨都可以，但是当定义多个指针时应注意。

**好的风格**

	// These are fine, space preceding.
	char *c;
	const string &str;

	// These are fine, space following.
	char* c;    // but remember to do "char* c, *d, *e, ...;"!
	const string& str;
	
**不好的风格**

	char * c;  // Bad - spaces on both sides of *
	const string & str;  // Bad - spaces on both sides of &
	
在单个文件内要保持风格一致, 所以,如果是修改现有文件, 要遵照该文件的风格.

## 6.9 类格式
--------------------------------------------------------
访问控制块的声明依次序是 public:, protected:, private:, 每次缩进2个空格.
类声明 (对类注释不了解的话, 参考 类注释) 的基本格式如下:

	class MyClass : public OtherClass {
	  public:      // Note the 2 space indent!
	      MyClass();  // Regular 4 space indent.
	      explicit MyClass(int var);
	      ~MyClass() {}
	      
	      void SomeFunction();
	      void SomeFunctionThatDoesNothing() {}
	      
	      void set_some_var(int var) { some_var_ = var; }
	      int some_var() const { return some_var_; }

	  private:
	  	  bool SomeInternalFunction();

		  int some_var_；
		  int some_other_var_;
		  DISALLOW_COPY_AND_ASSIGN(MyClass);
	};
	
注意事项:
所有基类名应在80列限制下尽量与子类名放在同一行.关键词 public:, protected:, private: 要缩进2个空格.
除第一个关键词 (一般是 public) 外, 其他关键词前要空一行. 如果类比较小的话也可以不空.
这些关键词后不要保留空行.public 放在最前面, 然后是 protected, 最后是 private.

## 6.10 初始化列表
--------------------------------------------------------

构造函数初始化列表放在同一行或按四格缩进并排几行.下面两种初始化列表方式都可以接受:

	// When it all fits on one line:
	MyClass::MyClass(int var) : some_var_(var), some_other_var_(var + 1) {
		...
		DoSomething();
		...
	}
	
或

	// When it requires multiple lines, indent 4 spaces, putting the colon on
	// the first initializer line:
	MyClass::MyClass(int var)
    	: some_var_(var),             // 4 space indent
      	some_other_var_(var + 1) {  // lined up
      	...
      	DoSomething();
      	...
	}



## 6.11 名字空间
--------------------------------------------------------

名字空间内容不缩进.名字空间不要增加额外的缩进层次。

**好的风格**

	namespace {

	void foo() {
	...
	}
	
	}  // namespace
	
**不好的风格**

	namespace {
		void foo() {
    	...
    	}
	}  // namespace


## 6.12 预处理指令
--------------------------------------------------------

预处理指令不要缩进, 从行首开始.即使预处理指令位于缩进代码块中, 指令也应从行首开始.

	// Good - directives at beginning of line
		if (lopsided_score) {
	#if DISASTER_PENDING      // Correct -- Starts at beginning of line
    		DropEverything();
	#endif
    		BackToNormal();
    	}
    	
## 6.13 函数返回值
-------------------------------------------------------

return表达式中不要用圆括号包围.函数返回时不要使用圆括号:

	return x;  // not return(x);


## 6.14 初始化
--------------------------------------------------------
用 = 或 () 均可。 在二者中做出选择; 下面的方式都是正确的:

	int x = 3;
	int x(3);
	string name("Some Name");
	string name = "Some Name";

对于基本变量使用＝，对于类类型变量使用( )，使用＝称为赋值初始化，使用（）称为直接初始化对于类类型变量，直接初始化效率更高。

## 6.15 空行
--------------------------------------------------------
垂直留白越少越好. 这不仅仅是规则而是原则问题了: 不在万不得已,不要使用空行.尤其是: 两个函数定义之间的空行不要超过 2行, 函数体首尾不要留空行, 函数体中也不要随意添加空行.

基本原则是: 同一屏可以显示的代码越多, 越容易理解程序的控制流. 当然, 过于密集的代码块和过于疏松的代码块同样难看, 取决于你的判断.但通常是垂直留白越少越好.

## 6.16 布尔表达式 
--------------------------------------------------------

如果一个布尔表达式超过标准行宽, 断行方式要统一一下。
下例中, 逻辑与 (&&) 操作符总位于行尾:

	if (this_one_thing > this_other_thing &&
    	a_third_thing == a_fourth_thing &&
    	yet_another & last_one) {
    	...
	}
	
注意, 上例的逻辑与 (&&) 操作符均位于行尾. 可以考虑额外插入圆括号, 合理使用的话对增强可读性是很有帮助的.

#7 杂项

# 8. 统一风格最重要

运用常识和判断力, 并保持一致.编辑代码时, 花点时间看看项目中的其它代码, 并熟悉其风格.如果其它代码中if语句使用空格, 那么你也要使用. 如果其中的注释用星号 (*) 围成一个盒子状, 你同样要这么做.

风格指南的重点在于提供一个通用的编程规范, 这样大家可以把精力集中在实现内容而不是表现形式上. 我们展示了全局的风格规范, 但局部风格也很重要, 如果你在一个文件中新加的代码和原有代码风格相去甚远, 这就破坏了文件本身的整体美观, 也影响阅读, 所以要尽量避免.


# 9 参考

1. google c++ code style
2. C++ primer
3. 编写可读代码的艺术