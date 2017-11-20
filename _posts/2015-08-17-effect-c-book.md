---
layout: post
title: Effective C++读书笔记
categories: C++ 读书笔记
description: Effective C++读书笔记
keywords: C++, effective C++
---

本文只是简单记录了effective C＋＋的一些重要笔记， 无个人总结。

# 一、让自己习惯C++
 
#### 条款01：视C++为一个语言联邦
1. C、Obeject-Oriented C++、Template C++、STL
 
#### 条款02：尽量以const、enum、inline替换#define
1. 对于纯常量，最好以const对象或enums替换#define。（const有符号表便于输出调试信息、有范围scope可只设定类内常量、可封装带修饰符、类型安全。Enum不会导致非必要的内存分配。#define只用于预处理）
2. 对于形似函数的宏，最好改用inline函数替换#define。
 
#### 条款03：尽可能使用const
1. 将某些东西声明为const可帮助编译器侦查出错误用法。const可被施加于任何作用域的对象、函数参数、函数返回类型、成员函数主题。
2. 编译器强制实施bitwiseconstness，但你编写程序时应该使用“概念上的常量性mutable”
3. 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复。
 
#### 条款04：确定对象被使用前已先被初始化
1. 为内置型对象进行手工初始化，因为C++不保证初始化他们
2. 构造函数最好使用成员初始列（memberinitialization list），而不要在构造函数本体内使用赋值操作（提高效率）。初始列列出的成员变量，其排列次序应该和他们在class中的声明次序相同。
3. 为免除“跨编译单元之初始化次序“问题，请以localstatic对象替换non-local static对象
 

 
# 二、构造、析构、赋值运算

#### 条款05：了解C++默认编写并调用哪些函数
1. 编译器可以暗自为class创建default构造函数、copy构造函数、copyassignment操作符，以及析构函数
 
#### 条款06：若不想使用编译器自动生成的函数，就该明确拒绝
1. 为驳回编译器自动提供的技能，可将相应的成员函数声明为private并且不予实现。使用像Uncopyable这样的base class也是一种做法。
 
#### 条款07：为多态基类声明virtual析构函数
1. polymorphic（带多态性质的）baseclasses应该声明一个virtual析构函数。如果class带有任何virtual函数。它应该拥有一个virtual析构函数。
2. Classse的设计目的如果不是作为baseclasses使用，或不是为了具备多态性，就不该声明virtual析构函数。
 
#### 条款08：别让异常逃离析构函数
1. 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或者结束程序。
2. 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么classs应该提供一个普通函数（而非在析构函数中）执行该操作。
 
#### 条款09：绝不在构造和析构过程中调用virtual函数
1. 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class（在base构造函数中调用virtual函数时调用base class的，而不是derived class的）。而应该将必要的参数从derived的构造函数传递给base的构造函数
![](/images/posts/2015-08-17-effect-c-book/1.png)

#### 条款10：令operator=返回一个refrence to *this
 
#### 条款11：在operator=中处理“自我赋值”
1. 确保当对象自我赋值时operator=有良好行为。其中技术包括比较“来源对象”和“目标对象”的地址、精心周到的语句顺序、及其copy-and-swap。
 
 
#### 条款12：复制对象时勿忘其每一个成分
1. Copying函数应该确保复制“对象内的所有成员”及“所有base class成分”
2. 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个coping函数共同调用。
 
 
 
# 三、资源管理
 
#### 条款13：以对象管理资源
1. 为防止资源泄露，请使用RAII对象，他们在构造函数中获得资源并在析构函数中释放资源。
2. 两个常被使用的RAII classes分别是tr1::shared_ptr和auto_ptr（他们都不适合管理new[]分配的资源。shared_ptr可以知道删除器函数）。前者通常是较佳选择，因为其copy行为比较直观。若选择auto_ptr，复制动作会使它（被复制物）指向null。
 
#### 条款14：在资源管理类中小心coping行为
1. 复制RAII（resourceacquisition is initialzation）对象必现一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
2. 普通而常见的RAII classcopying行为是：抑制copying、施以引用计数法。不过其他行为也都可能被实现。
 
#### 条款15：在资源管理类中提供对原始资源的访问
1. API往往要求访问原始资源（rawresource），所以每一个RAII class应该提供一个“取得其所管理之资源”的办法。
2. 对原始资源的访问可能经由显示转换(XXXhandle.get())或隐式转换（operatorXXX() const）。一般而言显式转换比较安全，但隐式转换对客户的比较方便。
 
#### 条款16：成对使用new和delete时要采用相同的形式
1. 如果你再new表达式中使用[]，必须在相应的delete表达式中也使用[]（尽量使用string、vector代替）。

![](/images/posts/2015-08-17-effect-c-book/2.png)
 
#### 条款17：以独立语句将newed对象置入智能指针
1. 以独立语句（单独放一行写）将newed对象置于智能指针内。如果不这样做，一但中间有异常抛出，有可能导致难以察觉的资源泄露
 
 
 
# 四、设计与声明

#### 条款18：让接口容易被正确使用，不易被误用
1. 好的接口很容易被正确使用，不容易被误用。
2. “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
3. “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，及其消除客户的资源管理责任。
4. tr1::shared_ptr支持定制型删除器（customdeleter）。可防止DLL问题，可被用来自动解除互斥锁（mutex）
![](/images/posts/2015-08-17-effect-c-book/3.png)

#### 条款19：设计class犹如设计type
 
#### 条款20：宁以pass-by-reference-to-const替换pass-by-value
1. 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可避免切割问题（slicing problem）
2. 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对他们而言pass-by-value往往比较适当。
 
#### 条款21：必须返回对象时，别妄想返回其reference
1. 绝不要返回pointer或reference指向一个localstack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而又可能同时需要多个这样的对象。
 
#### 条款22：将成员变量声明为private
1. 切记将成员变量声明为private。这可赋予客户随机访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。
2. protected并不比public更具封装性（涉及derivedclass）。
 
#### 条款23：宁以non-member、non-friend替换member函数
 
#### 条款24：若所有参数皆需类型转换，请为此采用non-member函数
 
#### 条款25：考虑写一个不抛异常的swap函数（多看几次原文）
1. 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常。
2. 如果你提供一个member swap，也该提供一个non-memberswap用来调用前者。对于classse（而非template），也请特化std::swap。
3. 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。
4. 为“用户定义类型”进行stdtemplate全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西。
 
 
 
# 五、实现
 
#### 条款26：尽可能延后变量定义式的出现时间
 
#### 条款27：尽量少做转型动作
1. 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计。
2. 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内。
3. 宁可使用C++-style新型转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。
![](/images/posts/2015-08-17-effect-c-book/4.png)
![](/images/posts/2015-08-17-effect-c-book/5.png)

#### 条款28：避免返回handles指向对象内部成分
1. 避免返回handle（包括reference、指针、迭代器）指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码牌”（dangling handles。对象被销毁时，handle将指向一个无效的地址）的可能性降到最低
 
#### 条款29：为“异常安全”而努力是值得的

1. 异常安全函数即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型。
2. “强烈保证”往往能够以copy-and-swap实现出来，但并非对所有函数都可实现或具备现实意义。
3. 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。
 
#### 条款30：彻底了解inlining的里里外外
1. 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化。使进程的速度提升机会最大化。
2. 不要只因为functiontemplate出现在头文件，就将他们声明为inline
![](/images/posts/2015-08-17-effect-c-book/6.png)
![](/images/posts/2015-08-17-effect-c-book/7.png)

#### 条款31：将文件间的编译依存关系降至最低
1. 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle class和interface class
2. 程序库头文件应该以“完全且仅声明式”的形式存在。这种做法不论是否涉及Template都适用
 
 
 
# 六、继承与面向对象设计
 
#### 条款32：确定你的public继承塑模出is-a关系
 
#### 条款33：避免遮掩继承而来的名称
1. derived classs内的名称会遮掩baseclass内的名称。在public继承下从来没有人希望如此
2. 为了让被遮掩的名称重见天日，可使用using声明式或转交函数（forwardingfunction）
 
 
#### 条款34：区分接口继承和实现继承
1. 接口继承和实现继承不同。在public继承下，derivedclass总是继承base class的接口。
2. pure virtual函数只具体指定接口继承，但没有缺省实现，虽然也可以提供。
3. 简朴的impure virtual函数具体指定接口继承及缺省实现继承。
4. non-virtual函数具体指定接口继承以及强制性实现继承，不能更改（虽然能改，但千万不要）。
 
#### 条款35：考虑virtual函数以外的其他选择
1. virtual函数的替代方案包括NVI手法及Strategy设计模式的多种手段。NV1手法自身是一个特殊形式的TemplateMethod设计模块。
2. 将机能从成员函数移动到class外部函数，带来的缺点是，非成员函数无法访问class的non-public成员。
3. tr1::function对象的行为就像一般函数指针。
![](/images/posts/2015-08-17-effect-c-book/8.png)
![](/images/posts/2015-08-17-effect-c-book/9.png)

#### 条款36：绝不重新定义继承而来的non-virtual函数
 
#### 条款37：绝不重新定义继承而来的缺省参数值
1. 缺省参数值都是静态绑定
 
#### 条款38：通过复合塑模出has-a或“根据某物实现出”
1. 在应用域（applicationdomain），复合意味着has-a（有一个）。在实现域（implement domain），复合意味is-implement-in-terms-of（根据某物实现出）
 
#### 条款39：明智而谨慎地使用private继承
1. private继承意味着is-implemented-in-terms-of（根据某物实现出）。它通常比复合的级别低。但是当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计是合理的。
2. 和复合不同，private继承可以造成emptybase最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。
![](/images/posts/2015-08-17-effect-c-book/10.png)
 
#### 条款40：明智而谨慎地使用多重继承
1. 多重继承比单一继承复杂。它可能导致新的歧义性，以及对virtual继承的需要。
2. virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果virtual base class不带任何数据，将是最具实用价值的情况。
3. 多重继承的确有正当用途。其中一个情节设计“public继承某个interfaceclass”和“private继承某个协助实现的class”的两项组合。
 
 
 
# 七、模板和泛型编程
 
#### 条款41：了解隐式接口和编译期多态
1. class和template都支持接口和多态
2. 对class而言接口是显式的（explicit），以函数签名为中心。多态则是通过virtual函数发型于运行期。
3. 对Template参数而言，接口是隐式的（implicit），基于有效表达式。多态则是通过Template具现化和函数重新解析（function overloading resolution）发生于编译期
 
#### 条款42：了解typename的双重意义
1. 声明Template参数时，前缀关键字class和typename可互换。
2. 请使用typename标识嵌套从属性类型名称；但不得在base clase lists（基类列）或者member initialization list（成员初值列）内以它作为baseclass修饰符。
 
#### 条款43：学习处理模板化基类内的名称
1. 可在derived classTemplate内通过“this->”指涉base class template内的成员名称，藉由一个明白写出的“base class资格修饰符”完成。
 
#### 条款44：将于参数无关的代码抽离Template
1. Template生成多个class和多个函数。所以任何Template代码都不该与某个造成膨胀的Template参数产生相依赖关系。
2. 因非类型模板参数（non-typeTemplate parameters）而造成的代码膨胀。往往可以消除，做法是以函数参数或class成员变量替换Template参数。
3. 因类型参数（typeparameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述的具现类型共享实现码。
 
#### 条款45：运用成员函数模板接受所有兼容类型
1. 请使用member functionTemplate（成员函数模板）生成“可接受所有兼容类型”的函数
2. 如果你声明member Template用户“泛化copy构造”或”泛化assignment操作”，你还是需要声明正常的copy构造函数和copy assignment操作符。
![](/images/posts/2015-08-17-effect-c-book/11.png)
![](/images/posts/2015-08-17-effect-c-book/12.png)
 
#### 条款46：需要类型转换时请为模板定义非成员函数
1. 当我们编写一个classTemplate。而它所提供之“与此Template相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为“class Template内部的friend函数”
![](/images/posts/2015-08-17-effect-c-book/13.png)
![](/images/posts/2015-08-17-effect-c-book/14.png)

#### 条款47：请使用traits class表现类型信息
1. Traits class使得“类型相关信息”在编译期可用。它们以Template和“Template特化”完成实现。
2. 整合重载计数后，traits class有可能在编译期对类型执行ifelse测试
![](/images/posts/2015-08-17-effect-c-book/15.png)
![](/images/posts/2015-08-17-effect-c-book/16.png)
![](/images/posts/2015-08-17-effect-c-book/17.png)
![](/images/posts/2015-08-17-effect-c-book/18.png)
![](/images/posts/2015-08-17-effect-c-book/19.png)
![](/images/posts/2015-08-17-effect-c-book/20.png)
![](/images/posts/2015-08-17-effect-c-book/21.png)
![](/images/posts/2015-08-17-effect-c-book/22.png)

#### 条款48：认识Template元编程
1. Templatemetaprogramming（TMP，模板元编程）可将工作由运行期往编译期，因而得以实现早期错误侦测和更高的执行效率。
2. TMP可被用来生成“基于政策选择组合”（based oncombination of policy choices）的客户定制代码，也可用来避免生成对某些特殊类型并不合适的代码。
![](/images/posts/2015-08-17-effect-c-book/23.png)
![](/images/posts/2015-08-17-effect-c-book/24.png)

 

 
# 八、定制new和delete
 
#### 条款49：了解new-handler的行为
1. set_new_handler允许客户指定一个函数，在内存分配无法满足时被调用。
2. Nothrow new是一个颇为局限的工具，因为它只适用于内存分配；后继的构造函数调用还是可能抛出异常
![](/images/posts/2015-08-17-effect-c-book/25.png)
 
#### 条款50：了解new和delete的合理替换时机
1. 有许多理由需要写一个自定的new和delete。包括改善效能、对heap运用错误进行调试、收集heap使用信息
 
#### 条款51：编写new和delete时需固守常规
1. operator new应该内含一个无限循环，并在其中尝试分配内存，如果它无法满足内存需求，就应该调用new-handler。它应该有能力处理0bytes申请。Class专属版本则还应该处理“比正确大小更大的错误申请”
2. operator delete应该在收到null指针时不做任何事。Class专属版本则还应该处理“比正确大小更大的错误申请”
 
#### 条款52：写了placement new也要写placement delete
1. 当写一个placementoperator new，请确定也写出了对应的placement operator delete。如果没有这样做，你的程序可能会发生隐微而时断时续的内存泄露
2. 当你声明placement new和placementdelete，请确定不要无意识地遮掩了他们的正常版本
![](/images/posts/2015-08-17-effect-c-book/26.png)
![](/images/posts/2015-08-17-effect-c-book/27.png)


 
 
# 九杂项讨论
 
#### 条款53：不要忽视编译期的告警
 
#### 条款54：让自己熟悉包括TR1在内的标准程序库
1. C++标准程序库的主要技能由STL、iostreams、locales组成。并包括C99标准程序库。
2. TR1添加了智能指针、一般函数指针（tr1::function）、hash-base容器、正则表达式以及另外10个组件的支持
3. TR1自身只是一个规范。一个好的实物来源是Boost
 
#### 条款55：让自己熟悉Boost

 
 




