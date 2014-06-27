---
layout: post
title: "Note for The C programming language"
date: 2013-08-02 23:08
comments: true
categories: [C, reading note] 
---

最近读了K&R，也就是传说中的《The C programming language》。作为我第一门学习的语言，对C的喜爱不言而喻。有两年多没接触C，再次回顾，相比其Java和C++等高级语言, 其简洁优美的特性深深吸引了我。以下是阅读过程中在evernote中的一些摘要。 


1.You should note that we are using the words definition and declaration carefully when we refer to external variables in this section. Definition refers to the place where the variable is created or assigned storage; declaration refers to places where the nature of the variable is stated but no storage is allocated.   
强调了声明与定义的区别。声明只是阐述了变量的类型，而没有实际分配内存。定义则为变量分配了内存。  

2.An external variable must be defined, exactly once, outside of any function; this sets aside storage for it. The variable must also be declared in each function that wants to access it; this states the type of the variable. The declaration may be an explicit extern statement or may be implicit from context.   
强调了外部变量的使用方法。必须先在函数外面定义一次，以分配内存空间。而在使用的函数内也要重新声明。声明可以直接使用extern关键字.  

3.The problem is distinguishing the end of input from valid data. The solution is that getchar returns a distinctive value when there is no more input, a value that cannot be confused with any real character. This value is called EOF, for end of file. We must declare c to be a type big enough to hold any value that getchar returns. We can't use char since c must be big enough to hold EOF in addition to any possible char. Therefore we use int.   
说明了getchar的使用方法。为了表明输入结束，getchar会返回EOF。为了能够容纳getchar的返回值，所以必须使用int而不是char  

4.strlen(s) returns the length of its character string argument s, excluding the terminal '\0'.  
strlen(s)返回的长度不包含表示字符串结尾的'\0'.   


5.External and static variables are initialized to zero by default. Automatic variables for which is no explicit initializer have undefined (i.e., garbage) values.  
外部变量与静态变量都会被默认初始化为0。 而未初始化的自动变量的初始值未定义，因此要注意初始化变量。    

6.By definition, the numeric value of a relational or logical expression is 1 if the relation is true, and 0 if the relation is false.  
当关系或逻辑表达式为真时对应的值为1， 假时为0.   

7.atoi, which converts a string of digits into its numeric equivalent.  
atoi将字符串转化为其表示的对应的数值.     

8.Declaring the argument x to be an unsigned ensures that when it is right-shifted, vacated bits will be filled with zeros, not sign bits, regardless of the machine the program is run on.  
将参数x声明为无符号的保证了它是向右移位并且用0来补位.    

9.The moral is that writing code that depends on order of evaluation is a bad programming practice in any language.  
在任何编程语言中， 表达式的结果依赖于其中a,b的估值顺序是一种不好的实践.   


10.double atof(); that too is taken to mean that nothing is to be assumed about the arguments of atof; all parameter checking is turned off. This special meaning of the empty argument list is intended to permit older C programs to compile with new compilers. But it's a bad idea to use it with new C programs. If the function takes arguments, declare them; if it takes no arguments, use void.  
在旧的C语言中，atof()这样会关掉参数检查。空参数列表是为了旧的C程序能够在新编译器下通过编译。新写的程序中，如果有参数，那就声明它。没有参数就使用void.   


11. if an external variable is to be referred to before it is defined, or if it is defined in a different source file from the one where it is being used, then an extern declaration is mandatory.  
必须将变量声明为external的情况有两种，一是在定义它（即为它分配内存前）使用它。二是它在别的文件中定义的。  


12.the variables that push and pop use for stack manipulation can be hidden, by declaring sp and val to be static.  
通过将变量声明为static,可以将其对外隐藏。   


13.A register declaration advises the compiler that the variable in question will be heavily used. The idea is that register variables are to be placed in machine registers, which may result in smaller and faster programs. But compilers are free to ignore the advice.  
将变量声明为register是建议编译器将其放在寄存器中，从而加快读写速度。但是编译器未必采纳该建议。   

14. In the absence of explicit initialization, external and static variables are guaranteed to be initialized to zero; automatic and register variables have undefined (i.e., garbage) initial values. For external and static variables, the initializer must be a constant expression.  
外部变量与静态变量自动初始化为0，而自动变量与寄存器变量则未定义。外部变量与静态变量必须使用常数表达式来初始化。   

15.char\* pattern = "ould";  is a shorthand for the longer but equivalent char pattern[] = { 'o', 'u', 'l', 'd', '\0' };.  

16.#define dprint(expr) printf(#expr " = %g\n", expr)
dprint(x/y) the macro is expanded into printf("x/y" " = &g\n", x/y);  

17.C converts it to \*(a+i) immediately; the two forms are equivalent. Applying the operator & to both parts of this equivalence, it follows that &a[i] and a+i are also identical: a+i is the address of the i-th element beyond a.  
对于数组a[i], 与\*(a+i)是等价的   

A pointer is a variable, so pa=a and pa++ are legal. But an array name is not a variable; constructions like a=pa and a++ are illegal.  
指针是变量，但是数组名不是，因此重新赋值或自增都是非法的   


If a two-dimensional array is to be passed to a function, the parameter declaration in the function must include the number of columns; the number of rows is irrelevant, since what is passed is, as before, a pointer to an array of rows, where each row is an array of 13 ints. In this particular case, it is a pointer to objects that are arrays of 13 ints. Thus if the array daytab is to be passed to a function f, the declaration of f would be:   
	f(int daytab[2][13]) { ... }
It could also be  
	f(int daytab[][13]) { ... }
since the number of rows is irrelevant, or it could be  
	f(int (*daytab)[13]) { ... }
which says that the parameter is a pointer to an array of 13 integers. The parentheses are necessary since brackets [] have higher precedence than \*. Without parentheses
daytab的类型是一个指向有13个元素的数组的指针。将13个元素的数组看成一个对象a,
当我们传递a的数组，我们会用a daytab[], 就像int arr[]一样。这就解释了为什么可以有第二种和第三种表达方式。   


the declaration:  
	int *daytab[13]
is an array of 13 pointers to integers. More generally, only the first dimension (subscript) of an array is free; all the others have to be specified.  
daytab的类型那个是一个指针数组   


argv[0] is the name by which the program was invoked, so argc is at least 1. If argc is 1, there are no command-line arguments after the program name.  
argv[0]是程序的名字，因此argc的值至少为1   


additionally, the standard requires that argv[argc] be a null point  
	char **argv
argv: pointer to char     
	int (*daytab)[13]  
daytab: pointer to array of 13 int elements   
	*daytab[13]   
daytab: array[13] of pointer to int   
	void *comp()
comp: function returning pointer to void   
	void (*comp)()
comp: pointer to function returning void    
	char (*(*x())[])()
x: function returning pointer to array[] of pointer to function returning char   
	char (*(*x[3])())[5]
x: array[3] of pointer to function returning pointer to array[5] of char   


18.Don't assume, however, that the size of a structure is the sum of the sizes of its members. Because of alignment requirements for different objects, there may be unnamed ''holes'' in a structure. Thus, for instance, if a char is one byte and an int four bytes, the structure:  
	struct {
		char c;
		int i;
	};  
might well require eight bytes, not five.  
一个结构的大小不一定是所有成员变量内存大小之和，因为内存需要对齐，所以存在洞。比如例子的结构占用8字节，第一个四字节中只有第一个字节是有效的。  

