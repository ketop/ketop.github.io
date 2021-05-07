---
title: "C++可调用对象"
date: 2020-12-07T22:30:07+08:00
draft: false
tags: ['c/c++']
---

c++11之前的可调用对象有：函数，函数指针，以及重载operator()运算符的对象。C++11之后，新增了可调用对象lambda表达式和bind函数的返回值，同时引入了新的类型function，用来**统一**可调用对象。声明方式如下

```cpp
function<T>;
```

T表示函数类型，ret-type(paramlist). 例如function<int(int,int)>表示返回值为int，入参为int，int的函数类型。

```cpp
#include <iostream>
#include <functional>
#include <string>
using std::string;
using std::function;

//普通函数
size_t func(const string& a){
    std::cout << "func:" << std::endl;
    return a.size();
}

//函数指针
size_t (*funcp)(const string& a) = func;

//类重载了operator()运算符，即对象的行为跟函数一样，除此之外，也可以包含其他成员
struct FuncA {
    FuncA():name("default instance"){}
    size_t operator()(const string&a){
        std::cout << "FuncA instance " << name << "'s operator()" << std::endl;
        return a.size();
    }
    string name;
};

int main()
{
    //使用funciton来统一调用
    std::function<size_t(const string&)> mofun;
    mofun = func;
    string cs("nice ok");
    std::cout << mofun(cs)<< std::endl;
    mofun = funcp;
    std::cout << mofun(cs)<< std::endl;
    mofun = [](const string& a)->size_t {
        std::cout << "lambda:" << std::endl;
        return a.size();
    };
    string az("nice to see you");
    std::cout << mofun(az) << std::endl;
    mofun = FuncA();//创建了一个FuncA类型的临时对象，等价于
    // FuncA tmp;
    // mofun = tmp;
    std::cout << mofun(az) << std::endl;
    return 0;
}
```



·除此之外，还有bind，它也在functional头文件中。可以将其视作函数适配器，用来对原函数进行**形参位置、个数调整**从而产生新的可调用对象。调用形式如下：

```cpp
auto newCallable=bind(callable, args_list)
```

args_list可能包含形如_n的名字，这些参数都是指占位符，数字n表示 **新的**可调用对象中的参数位置，从1开始。

```cpp
#include <iostream>
#include <functional>
#include <string>
using std::string;
void funN(const string&a, const string&b) {
    swap(a,b);
    std::cout << a << " " << b << std::endl;
}

int main(){
    string az("Love");
    string cs("You");
    funN(az, cs);
    std::function<int(int,int)> func;
    auto newCallable = bind(funN, std::placeholders::_2, std::placeholders::_1);
    newCallable(az, cs);
    //修改入参个数
    auto onlyOne = bind(funN, az, std::placeholders::_1);
    string c("world");
    onlyOne(c);    
    //如果bind中使用的是引用类型变量，必须加上std::ref,否则函数使用的是右值，不改变原有值
    std::cout << "az: " << az << ", c: " << c << std::endl;//az并不变,但是c变了
    //使用std::ref即可
    onlyOne2 = bind(funN, std::ref(az), std::placeholders::_1);
    c="world";
    onlyOne2(c);     
  //如果bind中使用的是引用类型变量，必须加上std::ref,否则函数使用的是右值，不改变原有值
    std::cout << "az: " << az << ", c: " << c << std::endl;//az也改变,c也变了  
    return 0;
}
```



 

