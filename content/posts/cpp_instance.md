---
title: "C++控制类对象实例化仅在栈或者堆上的方法"
date: 2019-04-24T17:29:14+08:00
draft: false
tags: [c/c++]
---

实例化一个类有两种方式

1. 静态实例化。编译器自动在栈上分配内存，然后调用构造函数。实例对象离开所在作用域后，执行析构函数，然后由编译器自动回收栈的内存。*编译器会根据类的构造函数和析构函数的可访问性来判断是否可以在栈上实例化*。
2. 动态实例化。程序员编码使用new操作符来实例化对象。new操作符调用类的operator new来在堆上申请内存，如果没有自定义的operator new，则调用全局的operator new来申请内存，然后使用定位new，调用类的构造函数。动态实例化对象，需要程序员自己调用delete来销毁对象。同理，在调用delete操作符时，会使用类的析构函数，然后使用类的operator delete来释放内存。如果没有定义类的operator delete，则使用全局的operator delete来释放内存。*编译器会根据operator new和operator delete的可访问性来判断是否可以在栈上构造*。

```cpp
//静态实例化
Object o;
//动态实例化
Object * po = new Object;
```





综上所述，如果需要限制类对象仅在栈上实例化，也就是禁止调用new来实例化对象，则可以将类的operator new和operator delete 设为delete.

```cpp
#include <iostream>
class OnlyStack {
    private:
        char* sentence;
    public:
        void* operator new(size_t ) = delete;
        void operator delete(void* ptr) = delete;
        OnlyStack()
        {
            std::cout << "initial ..." << std::endl;
            sentence = new char[10];
            for(int i = 0; i < 9; i++){
                sentence[i]= 'a' + i;
            }
            sentence[9]='\0';
        }
        ~OnlyStack()
        {
            std::cout << "destroy ..." << std::endl;
            delete sentence;
        }
        void printSentence() {
            std::cout << sentence << std::endl;
        }
};

int main(){
    //OnlyStack* osk = new OnlyStack; //无法使用new来实例化对象
    OnlyStack osk;
    osk.printSentence();

}
```





如果需要限制类对象仅在堆上实例化，也就是禁止直接调用构造函数来在栈上实例化，最直接的方式就是将类的构造函数设置为私有的。但是这种方式就需要类提供public的构建函数（例如单例的getInstance()),在函数中调用new操作符来实例化对象。 但是局限性在于，在类外不能使用new来实例化。

```cpp
#include <iostream>
class OnlyHeap {
    private:
        char* sentence;
        OnlyHeap()
        {
            std::cout << "initial ..." << std::endl;
            sentence = new char[10];
            for(int i = 0; i < 9; i++){
                sentence[i]= 'a' + i;
            }
            sentence[9]='\0';
        }
    public:
        static OnlyHeap* create()
        {
            return new OnlyHeap();
        }
        static void destroy(OnlyHeap* ptr){
            delete ptr;
        }
        ~OnlyHeap()
        {
            std::cout << "destroy ..." << std::endl;
            delete sentence;
        }
        void printSentence() {
            std::cout << sentence << std::endl;
        }
};

int main(){
    OnlyHeap* ohp = OnlyHeap::create();//无法直接调用new来类外创建对象。
    ohp->printSentence();
    delete ohp;
    //OnlyHeap::destroy(ohp); //如果析构函数也声明为私有的，则可以使用此函数来释放对象内存。
}
```



 因此可以考虑将类的析构函数设置为私有的。

```cpp
#include <iostream>
class OnlyHeap {
    private:
        char* sentence;
        ~OnlyHeap()
        {
            std::cout << "destroy ..." << std::endl;
            delete sentence;
        }
    public:
        static OnlyHeap* create()
        {
            return new OnlyHeap();
        }
        static void destroy(OnlyHeap* ptr){
            delete ptr;
        }
        void destory_self(){
            delete this;
        }
        OnlyHeap()
        {
            std::cout << "initial ..." << std::endl;
            sentence = new char[10];
            for(int i = 0; i < 9; i++){
                sentence[i]= 'a' + i;
            }
            sentence[9]='\0';
        }
        void printSentence() {
            std::cout << sentence << std::endl;
        }
};

int main(){
    OnlyHeap* ohp = new OnlyHeap();
    ohp->printSentence();
    ohp->destory_self();//无法直接调用delete ohp来释放内存,析构函数定义为私有的了。
}
```

