今天是7月1日 目标 快速排序 分治算法

# 1malloc和new的区别

int *a=new int(10);  int *arr=new int [10] ()  //后面那个括号是初始化为零

delete a;                    delete[]  arr;

int * a=(int *)malloc(sizeof(int));

int *a=(int *)malloc(sizeof(int) *10);

1 都在堆区开辟空间 new可以初始化 malloc不初始化

2malloc开辟的是指定字节大小的存储空间 需要类型转换 new开辟相应数据类型空间 不需要强制类型转换

3 malloc是函数 new是运算符

4new底层使用malloc开辟的

# 2define可以定义函数 那么他和inline内联函数的区别

1 define在预处理阶段编译不加分号 内联函数在程序运行时调用

2define是简单的字符串替换 没有类型检查，内联函数是函数调用有类型检查

3类中成员函数默认都是内联函数。类外定义的内联函数必须显式声明 

# 3.define和typedef的区别

1typedef是对类型起别名，define是简单的字符替换

2define是在预处理  typedef是在编译阶段

# 4ifdef和ifndef

#ifndef _A_H

#define _A_H

头文件中的内容

#endif;

这是防止头文件包含，每次包含头文件时检查_A_H是否未定义，没定义就执行下面的代码 #pragma once也是防止头文件重复包含都是写在头文件里的

#ifdef 

代码

#endif

如果宏已经定义 就执行下面代码

# 5类类型所占用的存储空间

空类占用一个字节 为空类对象的地址

类成员注意内存对齐，int和char对齐就是8个字节

有虚函数时实际上是一个虚函数指针指向虚函数表 

## 虚函数指针和虚函数表

只要这个类有虚函数，类就虚函数指针->(新建)虚函数表

如果子类继承基类但自己没有虚函数，那么他也会新建虚函数表，只是将自己的虚函数表中的指针指向父类的虚函数地址

![image-20200704104756434](C:\Users\18302\AppData\Roaming\Typora\typora-user-images\image-20200704104756434.png)

动态绑定就是函数调用在运行时绑定

在多继承中子类自己的虚函数会存在第一个vfptr指向的虚函数表中最后面

https://www.cnblogs.com/zhxmdefj/p/11594459.html  虚函数指针的原理

typedef void(*Fun)(void); 定义一个Fun类型的函数指针

```c++
typedef void(*Fun)(void);//定义一个Fun类型的函数指针

int main()
{
	Base bObj;
	Fun pFun = NULL;	
    //指向void* pf(void)类的函数的指针pFun

	cout << "虚指针的地址：" << (int*)(&bObj) << endl;//虚指针在第一个
	cout << "虚函数表的第一个函数地址：" << (int*) * (int*)(&bObj) << endl;
	//再次取址得到第一个虚函数的地址

	//第一个虚函数
	pFun = (Fun) * ((int*) * (int*)(&bObj));//(Fun)*是强制类型转换
	pFun();
}
```



# 6C++内存分区

C++的内存分区大概分成五个部分：

1. 栈（stack）：是由编译器在需要时自动分配，不需要时自动清除的变量存储区，通常存放局部变量、函数参数等。
2. 堆（heap）：是由`new`分配的内存块，由程序员释放（编译器不管），一般一个`new`与一个`delete`对应，一个`new[]`与一个`delete[]`对应，如果程序员没有释放掉，资源将由操作系统在程序结束后自动回收
3. 自由存储区：是由`malloc`等分配的内存块，和堆十分相似，用`free`来释放
4. 全局/静态存储区：**全局变量**和**静态变量**被分配到同一块内存中
5. 常量存储区：这是一块特殊存储区，里边存放常量，不允许修改

（堆和自由存储区其实不过是同一块区域，new底层实现代码中调用了malloc，new可以看成是malloc智能化的高级版本）

# 7.c++强制类型转换

const int val=5;

int * pval=const_cast<int*>(&val);

指向同一块内存，但是val的值还是5，这是由于编译器一直把5赋值给val；

# 8.new操作符可以不初始化

Base *b=new Base; 后面加括号调用构造函数，

Base *b=new Base（）;默认构造初始化为零

int *a=new int；不加括号不进行初始化，加括号初始化为零

# 9.类的默认构造函数不进行初始化

# 10.单例模式

构造函数私有，就不能通过类名创建其他对象了

只能通过  公有静态函数 才能new类对象的时候 访问类内的 私有构造函数 

```c++
class Singlton 
{
private:
	Singlton() {}
	static Singlton *single;
public:
	static Singlton *GetSinglton();

};
Singlton *Singlton::single = nullptr;
Singlton *Singlton::GetSinglton() {
	if (single == nullptr) {
		single = new Singlton;
	}
	return single;
}

int main(int argc, const char * argv[])
{
	Singlton *s1 = Singlton::GetSinglton();
	Singlton *s2 = Singlton::GetSinglton();
	if (s1 == s2)
	{
		cout << "s1==s2" << endl;
	}
	else
	{
		cout << "s1!=s2" << endl;
	}
	//多个指针指向一个对象内存；
	return 0;
}
```

双重锁

```c++
class singleton
{
protected:
    singleton()
    {
        // 初始化
        pthread_mutex_init(&mutex);
    }
private:
    static singleton* p;
public:
    static pthread_mutex_t mutex;
    static singleton* initance();
};
 
pthread_mutex_t singleton::mutex;
singleton* singleton::p = NULL;
singleton* singleton::initance()
{
    if (p == NULL)
    {
        // 加锁
        pthread_mutex_lock(&mutex);
        if (p == NULL)
            p = new singleton();
        pthread_mutex_unlock(&mutex);
    }
    return p;
}
```
#
