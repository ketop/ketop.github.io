---
title: "设计模式-单例模式"
date: 2021-04-26T00:04:47+08:00
tags: ['设计模式', 'c/c++']
draft: false
---

## 饿汉模式

在程序开始时就完成实例化操作。

```cpp
#include <iostream>
#include <memory>

class Singleton {
    private:
        Singleton(){
            std::cout << "Constructor"  << std::endl;
        }
        ~Singleton() {
            std::cout << "Deconstructor" << std::endl;
        }
        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton&)=delete;
        static Singleton m_instance;
    public:
        static Singleton* getInstance() {

            return &m_instance;
        }
        void echo() {
            std::cout << "Print All Message." << std::endl;
        }
};
Singleton Singleton::m_instance;


int main() {
    Singleton* pst = Singleton::getInstance();
    pst->echo();
    return 0;

}
```



将m_instance声明为Singleton的指针类型

```cpp
#include <iostream>
#include <memory>

class Singleton {
    private:
        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton&)=delete;
        static Singleton* m_instance;
        Singleton(){
            std::cout << "Constructor"  << std::endl;
        }
        ~Singleton() {
            std::cout << "Deconstructor" << std::endl;
        }
    public:
        static Singleton* getInstance() {
    		return m_instance;
		}
        void echo() {
            std::cout << "Print All Message." << std::endl;
        }
};

Singleton* Singleton::m_instance(new Singleton());


int main() {
    Singleton* pst = Singleton::getInstance();
    pst->echo();
    return 0;

}
```

此时单例资源无法释放，使用valgrind来测试下

```bash
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all  --show-reachable=yes ./singleton
==6146== Memcheck, a memory error detector
==6146== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==6146== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==6146== Command: ./singleton4
==6146==
Constructor
Print All Message.
==6146==
==6146== HEAP SUMMARY:
==6146==     in use at exit: 1 bytes in 1 blocks
==6146==   total heap usage: 3 allocs, 2 frees, 73,729 bytes allocated
==6146==
==6146== 1 bytes in 1 blocks are still reachable in loss record 1 of 1
==6146==    at 0x483BE63: operator new(unsigned long) (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
==6146==    by 0x109299: __static_initialization_and_destruction_0(int, int) (singleton4.cpp:22)
==6146==    by 0x1092E7: _GLOBAL__sub_I__ZN9Singleton10m_instanceE (singleton4.cpp:32)
==6146==    by 0x1093BC: __libc_csu_init (in /home/ketop/project/designpattern/singleton4)
==6146==    by 0x4A7903F: (below main) (libc-start.c:264)
==6146==
==6146== LEAK SUMMARY:
==6146==    definitely lost: 0 bytes in 0 blocks
==6146==    indirectly lost: 0 bytes in 0 blocks
==6146==      possibly lost: 0 bytes in 0 blocks
==6146==    still reachable: 1 bytes in 1 blocks
==6146==         suppressed: 0 bytes in 0 blocks
==6146==
==6146== For lists of detected and suppressed errors, rerun with: -s
==6146== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

发现静态资源没有被释放，still reachable依然保留内存。由于析构函数被声明为private，因此只能再声明个static函数来调用析构函数。

```
static void destory(){
	if(m_instance){
	    delete m_instance;
	}
}

//调用destory来释放内存
pst->destory();
```

但是以上方式如果忘记调用，则无法释放资源，还是不够完美。如何能自动释放呢？

1. 内部类

```cpp
#include <iostream>
#include <memory>

class Singleton {
    private:
        Singleton(){
            std::cout << "Constructor"  << std::endl;
        }
        ~Singleton() {
            std::cout << "Deconstructor" << std::endl;
        }
        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton&)=delete;
        static Singleton* m_instance;
        class CGarbo {
            public:
                CGarbo(){}
                ~CGarbo(){
                    if(m_instance){
                        delete m_instance;
                    }
                }
        };
        static CGarbo m_garbo;
    public:
        static Singleton* getInstance() {
            return m_instance;
        }
        void echo() {
            std::cout << "Print All Message." << std::endl;
        }
};

Singleton* Singleton::m_instance=new Singleton();
Singleton::CGarbo Singleton::m_garbo;

int main() {
    Singleton* pst = Singleton::getInstance();
    pst->echo();
    return 0;

}
```







2. shared_ptr智能指针

结合C++的shared_ptr自动释放资源的特性，可以采用以下方式来释放。

1. 定义析构函数为public，但是有可能被用户误用析构函数
2. 定义一个删除器，形参为指针

```cpp
#include <memory>

class Singleton {
    private:
        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton&)=delete;
        static std::shared_ptr<Singleton> m_instance;
        Singleton(){
            std::cout << "Constructor"  << std::endl;
        }
        ~Singleton() {
            std::cout << "Deconstructor" << std::endl;
        }
    public:
        //定义删除器
        static void destory(Singleton* ptr) {
            std::cout << "destroy" << std::endl;
            if(ptr) {
               delete ptr;
            }

        }
        static std::shared_ptr<Singleton> getInstance() {
    		return m_instance;
		}
        void echo() {
            std::cout << "Print All Message." << std::endl;
        }
};

std::shared_ptr<Singleton> Singleton::m_instance(new Singleton(), Singleton::destory);

int main() {
    std::shared_ptr<Singleton> pst = Singleton::getInstance();
    pst->echo();
    return 0;

}
```



## 懒汉模式

在需要的时候实例化，构造时并不实例化。

需要处理的问题：多线程安全

### 静态局部变量 [Best Practice]

> If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for completion of the initialization.   
>
> 如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。
>
> ​                                                                                                        《Effective C++》作者 Meyers

```cpp
#include <iostream>
#include <memory>

class Singleton {
    private:
        Singleton(){
            std::cout << "Constructor"  << std::endl;
        }
        ~Singleton() {
            std::cout << "Deconstructor" << std::endl;
        }
        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton&)=delete;
    public:
    //线程安全：《Effective C++》作者Meyers
    //
        static Singleton* getInstance() {
            static Singleton m_instance;//调用时才构造，与饿汉差异很小
            return &m_instance;
        }
        void echo() {
            std::cout << "Print All Message." << std::endl;
        }
};


int main() {
    Singleton* pst = Singleton::getInstance();
    pst->echo();
    return 0;
}
```

更优雅的做法是返回引用，因为局部静态变量把那个不是分配在堆上，直接返回引用显的更一致。

```cpp
#include <iostream>
#include <memory>

class Singleton {
    private:
        Singleton(){
            std::cout << "Constructor"  << std::endl;
        }
        ~Singleton() {
            std::cout << "Deconstructor" << std::endl;
        }
        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton&)=delete;
    public:
        static Singleton& getInstance() {
            static Singleton m_instance;
            return m_instance;
        }
        void echo() {
            std::cout << "Print All Message." << std::endl;
        }
};


int main() {
    Singleton& pst = Singleton::getInstance();
    pst.echo();
    return 0;

}
```

### 动态方式， double check

另一种是使用内部类的方式自动释放静态类的内存。

```cpp
#include <iostream>
#include <mutex>
using std::cout;
using std::endl;
using std::mutex;
using std::lock_guard;

class Singleton{
    public:
        static Singleton* getInstance(){
            if(_instance == nullptr) {
                lock_guard<mutex> lock(m_mutex);
                if(_instance == nullptr) {
                    _instance = new Singleton();
                }
            }
            return _instance;
        }
        void run() { std::cout << "run" << std::endl; }
    private:
        Singleton(){ cout << "Constructor" << endl; }
        ~Singleton(){ cout << "Deconstructor" << endl; }
        static mutex m_mutex;

        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton& )=delete;
        static Singleton* _instance;
        class CGarbo {
            public:
                CGarbo(){}
                ~CGarbo() {
                    if (_instance) {
                        delete _instance;
                    }
                }
        };
        static CGarbo m_garbo;
};

Singleton* Singleton::_instance=nullptr;
Singleton::CGarbo Singleton::m_garbo;
mutex Singleton::m_mutex;

int main()
{
    Singleton* sp = Singleton::getInstance();
    sp->run();

}
```

使用C++11的shared_ptr方式，自动释放懒汉模式内存。

```cpp
#include <iostream>
#include <memory>
#include <mutex>
using std::cout;
using std::endl;
using std::shared_ptr;

using std::mutex;
using std::lock_guard;

class Singleton{
    public:
        static shared_ptr<Singleton> getInstance(){
            if(_instance == nullptr) {
                lock_guard<mutex> lock(m_mutex);
                if(_instance == nullptr) {
                    _instance.reset(new Singleton(), destroy);
                }
            }
            return _instance;
        }
        void run() { std::cout << "run" << std::endl; }
    private:
        Singleton(){ cout << "Constructor" << endl; }
        ~Singleton(){ cout << "Deconstructor" << endl; }
        Singleton(const Singleton&)=delete;
        Singleton& operator=(const Singleton& )=delete;
        static void destroy(Singleton* p){
            if(p) {
                delete p;
                cout << "delete instance" << endl;
            }
        }
        static shared_ptr<Singleton> _instance;
        static mutex m_mutex;
};

shared_ptr<Singleton> Singleton::_instance(nullptr);
mutex Singleton::m_mutex;

int main()
{
    shared_ptr<Singleton> sp = Singleton::getInstance();
    sp->run();

}
```

