---
layout: post
title: ［精品］解密函数调用
categories: 操作系统原理 C/C++
description: 解密函数调用
keywords: 函数调用, stack
---



# 栈(Stack)

1)栈在内存中是**从高地址向低地址扩展**。如下图，**上高地址**。因此栈顶地址是不断减小的，越后入栈的数据，所处的地址也就越低。

![](/images/posts/2015-09-07-func-call-stack.md/1.png)


2)在32位系统中，**栈每个数据单元的大小为4字节**。

3)和栈的操作相关的两个寄存器是EBP寄存器和ESP寄存器的。**ESP寄存器总是指向栈的栈顶**，执行PUSH命令向栈压入数据时，ESP减4，然后把数据拷贝到ESP指向的地址；执行POP命令时，首先把ESP指向的数据拷贝到内存地址/寄存器中，然后ESP加4。**EBP寄存器是用于访问栈中的数据的，它指向栈中间的某个位置，函数的参数地址比EBP的值高，而函数的局部变量地址比EBP的值低**，因此参数或局部变量总是通过EBP加减一定的偏移地址来访问的，比如，要访问函数的第一个参数为EBP+8。

4)栈中存储了**函数的参数，函数的局部变量，寄存器的值（用以恢复寄存器），函数的返回地址以及用于结构化异常处理的数据**。这些数据是按照一定的顺序组织在一起的，我们称之为一个**堆栈帧（Stack Frame）**。在函数退出时，整个函数帧将被销毁。
 
```C++
int foo1(int m, int n){
    int p=m*n;
    return p;
}
int foo(int a, intb){
    int c=a+1;
    int d=b+1;
    int e=foo1(c,d);
    return e;
}
int main(){
    int result=foo(3,4);
    return 0;
}
```

# 栈的建立


从main函数执行的第一行代码，即int result=foo(3,4);开始跟踪。这时main以及之前的函数对应的堆栈帧已经存在在栈中了，如下图所示：

![](/images/posts/2015-09-07-func-call-stack.md/2.png)



# 参数入栈


当foo(3,4)函数被调用，首先，caller（此时caller为main函数）把foo函数的两个参数：a=3,b=4压入堆栈。参数入栈的顺序是由函数的调用约定(CallingConvention)决定的。一般来说，参数都是从右往左入栈的，因此，b=4先压入堆栈，a=3后压入，如图：

![](/images/posts/2015-09-07-func-call-stack.md/3.png)


# 返回地址入栈


函数被调用时，会自动把下一条指令的地址压入堆栈，函数结束时，从堆栈读取这个地址，就可以跳转到该指令执行了。

![](/images/posts/2015-09-07-func-call-stack.md/4.png)



# 代码跳转到被调用函数执行


返回地址入栈后，代码跳转到被调用函数foo中执行。到目前为止，堆栈帧的前一部分，是由caller构建的；而在此之后，堆栈帧的其他部分是由callee来构建。


# EBP指针入栈


在foo函数中，首先将EBP寄存器的值压入堆栈。因为此时EBP寄存器的值还是用于main函数的，用来访问main函数的参数和局部变量的，因此需要将它暂存在栈中，在foo函数退出时恢复。同时，给EBP赋于新值。

1）将EBP压入堆栈

2）把ESP的值赋给EBP

![](/images/posts/2015-09-07-func-call-stack.md/5.png)

很容易发现当前EBP寄存器指向的堆栈地址就是EBP先前值的地址，你还会发现发现，EBP+4的地址就是函数返回值的地址，EBP+8就是函数的第一个参数的地址。因此，通过EBP很容易查找函数是被谁调用的或者访问函数的参数（或局部变量）。


# 为局部变量分配地址


接着，foo函数将为局部变量分配地址。程序并不是将局部变量一个个压入堆栈的，而是将ESP减去某个值，直接为所有的局部变量分配空间，比如在foo函数中有ESP=ESP-0x00E4

![](/images/posts/2015-09-07-func-call-stack.md/6.png)

在debug模式下，编译器为局部变量分配的空间远远大于实际所需，而且局部变量之间的地址不是连续的


# 通用寄存器入栈


最后，将函数中使用到的通用寄存器入栈，暂存起来，以便函数结束时恢复。在foo函数中用到的通用寄存器是EBX，ESI，EDI，将它们压入堆栈，如图所示：

![](/images/posts/2015-09-07-func-call-stack.md/7.png)


至此，一个完整的堆栈帧建立起来了。

![](/images/posts/2015-09-07-func-call-stack.md/8.png)

我们发现，EBP寄存器的地址值总是指向先前的EBP，而先前的EBP又指向先前的先前的EBP，这样就在堆栈中形成了一个链表！这个特性有什么用呢，我们知道EBP+4地址存储了函数的返回地址，通过该地址我们可以知道当前函数的上一级函数（通过在符号文件中查找距该函数返回地址最近的函数地址，该函数即当前函数的上一级函数），以此类推，我们就可以知道当前线程整个的函数调用顺序。


# 返回值是如何传递的

1）首先，如果返回值等于4字节，函数将把返回值赋予EAX寄存器，通过EAX寄存器返回。例如返回值是字节、字、双字、布尔型、指针等类型，都通过EAX寄存器返回。

2）如果返回值等于8字节，函数将把返回值赋予EAX和EDX寄存器，通过EAX和EDX寄存器返回，EDX存储高位4字节，EAX存储低位4字节。例如返回值类型为__int64或者8字节的结构体通过EAX和EDX返回。

3)  如果返回值为double或float型，函数将把返回值赋予浮点寄存器，通过浮点寄存器返回。

4）如果返回值是一个大于8字节的数据，将如何传递返回值呢？
我们修改foo函数的定义如下并将它的代码做适当的修改：
```C++
MyStruct foo(int a, int b)
{
    ...
}
//MyStruct定义为：
struct MyStruct
{
    int value1;
    __int64 value2;
    bool value3;
};
```

这时，在调用foo函数时参数的入栈过程会有所不同，如下图所示：

![](/images/posts/2015-09-07-func-call-stack.md/9.png)

caller会在压入最左边的参数后，再压入一个指针，我们姑且叫它ReturnValuePointer，ReturnValuePointer指向caller局部变量区的一块未命名的地址，这块地址将用来存储callee的返回值。函数返回时，callee把返回值拷贝到ReturnValuePointer指向的地址中，然后把ReturnValuePointer的地址赋予EAX寄存器。函数返回后，caller通过EAX寄存器找到ReturnValuePointer，然后通过ReturnValuePointer找到返回值，最后，caller把返回值拷贝到负责接收的局部变量上（如果接收返回值的话）。


# 堆栈帧的销毁


当函数将返回值赋予某些寄存器或者拷贝到堆栈的某个地方后，函数开始清理堆栈帧，准备退出。堆栈帧的清理顺序和堆栈建立的顺序刚好相反：
1. 如果有对象存储在堆栈帧中，对象的析构函数会被函数调用。
2. 从堆栈中弹出先前的通用寄存器的值，恢复通用寄存器。
3. ESP加上某个值，回收局部变量的地址空间（加上的值和堆栈帧建立时分配给局部变量的地址大小相同）。
4. 从堆栈中弹出先前的EBP寄存器的值，恢复EBP寄存器。
5. 从堆栈中弹出函数的返回地址，准备跳转到函数的返回地址处继续执行。
6. ESP加上某个值，回收所有的参数地址。

前面1-5条都是由callee完成的。而第6条，参数地址的回收，是由caller或者callee完成是由函数使用的调用约定（calling convention ）来决定的。


# 函数的调用约定（calingconvention）

函数的调用约定(callingconvention)指的是进入函数时，函数的参数是以什么顺序压入堆栈的，函数退出时，又是由谁（Caller还是Callee）来清理堆栈中的参数。

1. __cdecl。这是VC编译器默认的调用约定。其规则是：参数从右向左压入堆栈，函数退出时由caller清理堆栈中的参数。这种调用约定的特点是支持可变数量的参数，比如printf方法。由于callee不知道caller到底将多少参数压入堆栈，因此callee就没有办法自己清理堆栈，所以只有函数退出之后，由caller清理堆栈，因为caller总是知道自己传入了多少参数。

2. __stdcall。所有的Windows API都使用__stdcall。其规则是：参数从右向左压入堆栈，函数退出时由callee自己清理堆栈中的参数。由于参数是由callee自己清理的，所以__stdcall不支持可变数量的参数。

3.  __thiscall。类成员函数默认使用的调用约定。其规则是：参数从右向左压入堆栈，x86构架下this指针通过ECX寄存器传递，函数退出时由callee清理堆栈中的参数，x86构架下this指针通过ECX寄存器传递。同样不支持可变数量的参数。如果显式地把类成员函数声明为使用__cdecl或者__stdcall，那么，将采用__cdecl或者__stdcall的规则来压栈和出栈，而this指针将作为函数的第一个参数最后压入堆栈，而不是使用ECX寄存器来传递了。


# 反编译代码跟踪

以下代码为和foo函数对应的堆栈帧建立相关的代码的反编译代码，我将逐行给出注释，可对照前文中对堆栈的描述：

main函数中 int result=foo(3,4);的反汇编：
```AMS
008A147E  push       4                     //b=4 压入堆栈  
008A1480  push       3                     //a=3 压入堆栈,到达图2的状态
008A1482  call       foo (8A10F5h)         //函数返回值入栈,转入foo中执行,到达图3的状态
008A1487  add        esp,8                 //foo返回,由于采用__cdecl,由Caller清理参数
008A148A  mov        dword ptr [result],eax //返回值保存在EAX中,把EAX赋予result变量
```

下面是foo函数代码正式执行前和执行后的反汇编代码
```AMS
008A13F0  push       ebp                  //把ebp压入堆栈
008A13F1  mov        ebp,esp              //ebp指向先前的ebp,到达图4的状态
008A13F3  sub        esp,0E4h             //为局部变量分配0E4字节的空间,到达图5的状态
008A13F9  push       ebx                  //压入EBX
008A13FA  push       esi                  //压入ESI
008A13FB  push       edi                  //压入EDI,到达图7的状态
008A13FC  lea        edi,[ebp-0E4h]       //以下4行把局部变量区初始化为每个字节都等于cch
008A1402  mov        ecx,39h
008A1407  mov        eax,0CCCCCCCCh
008A140C  rep stos   dword ptr es:[edi]
......                                      //省略代码执行N行
......
008A1436  pop        edi                   //恢复EDI 
008A1437  pop        esi                   //恢复ESI
008A1438  pop        ebx                   //恢复EBX
008A1439  add        esp,0E4h              //回收局部变量地址空间
008A143F  cmp        ebp,esp               //以下3行为RuntimeChecking,检查ESP和EBP是否一致  
008A1441  call       @ILT+330(__RTC_CheckEsp) (8A114Fh)
008A1446  mov        esp,ebp
008A1448  pop        ebp                   //恢复EBP
008A1449  ret                               //弹出函数返回地址，跳转到函数返回地址执行                                           //(__cdecl调用约定,Callee未清理参数)
```


