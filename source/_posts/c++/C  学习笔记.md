---
layout: post
title:  C++学习笔记
date: '2016-05-14 17:02'
categories: C++
tags: [C++]
description:  C++学习笔记
comments: true
---

# C++学习笔记

## 类
```
//
//  main.cpp
//  噼噼啪啪
//
//  Created by wangyonghua on 2021/1/1904431.
//

#include <iostream>
#import "Box.hpp"
#import "Student.hpp"
#import "VLA.hpp"
#import "Shape.hpp"
#include <memory>

void foo(std::shared_ptr<int> i)
{
    (*i)++;
}

int main(int argc, const char * argv[]) {
    // insert code here...
    std::cout << "Hello, World!\n";
    
    //创建对象-创建的对象 stu, 在栈上分配内存，
    Student stu0;
    stu0.name = "小明";
    stu0.age = 15;
    stu0.score = 92.5f;
    stu0.say();
    
    //使用对象指针
    Student *pStu1 = &stu0;
    pStu1 -> name = "小明";
    pStu1 -> age = 16;
    pStu1 -> score = 92.5f;
    pStu1 -> say();
    
    //在堆上创建对象
    Student *pStu = new Student;
    //使用 new 在堆上创建出来的对象是匿名的，没法直接使用，
    //必须要用一个指针指向它，再借助指针来访问它的成员变量或成员函数。
    pStu -> name = "小明";
    pStu -> age = 15;
    pStu -> score = 92.5f;
    pStu -> say();
    delete pStu;  //删除对象
    
    //在栈上创建对象
    Student stu;
    stu.setname("小明");
    stu.setage(15);
    stu.setscore(92.5f);
    stu.show();
    //在堆上创建对象
    Student *pstu = new Student;
    pstu -> setname("李华");
    pstu -> setage(16);
    pstu -> setscore(96);
    pstu -> show();
    
//    //创建一个有n个元素的数组（对象）
//    int n;
//    cout<<"Input array length: ";
//    cin>>n;
//    VLA *parr = new VLA(n);
//    //输入数组元素
//    cout<<"Input "<<n<<" numbers: ";
//    parr -> input();
//
//    //输出数组元素
//    cout<<"Elements: ";
//    parr -> show();
//    //删除数组（对象）
//    delete parr;

//    (new Student("小明", 15, 90.6)) -> show();
//    (new Student("李磊", 16, 80.5)) -> show();
//    (new Student("张华", 16, 99.0)) -> show();
//    (new Student("王康", 14, 60.8)) -> show();
//    int total = Student::getTotal();
//    float points = Student::getPoints();
//    cout<<"当前共有"<<total<<"名学生，总成绩是"<<points<<"，平均分是"<<points/total<<endl;
//
    Shape *shape;
    Rectangle rec(10,7);
    Triangle  tri(10,5);
     
    // 存储矩形的地址
    shape = &rec;
    // 调用矩形的求面积函数 area
    shape->area();
    shape->area1();
     
    // 存储三角形的地址
    shape = &tri;
    // 调用三角形的求面积函数 area
    shape->area();
    shape->area1();
    
    JNI::JDA::Box box;
    box.breadth = 10.0;
    
    auto pointer = std::make_shared<int>(10);
    foo(pointer);
    std::cout << *pointer << std::endl; // 11
    
    /**
     std::shared_ptr 可以通过 get() 方法来获取原始指针，
     通过 reset() 来减少一个引用计数，
     并通过use_count()来查看一个对象的引用计数。例如：
     */
    auto pointer2 = pointer; // 引用计数+1
    auto pointer3 = pointer; // 引用计数+1
    int *p = pointer.get(); // 这样不会增加引用计数
    std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl; // 3
    std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl; // 3
    std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl; // 3

    pointer2.reset();
    std::cout << "reset pointer2:" << std::endl;
    std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl; // 2
    std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl; // 0, pointer2 已 reset
    std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl; // 2
    
    pointer3.reset();
    std::cout << "reset pointer3:" << std::endl;
    std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl; // 1
    std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl; // 0
    std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl; // 0, pointer3 已 reset

    /*
     make_unique 并不复杂，C++11 没有提供 std::make_unique，可以自行实现：
     template<typename T, typename ...Args>
     std::unique_ptr<T> make_unique( Args&& ...args ) {
       return std::unique_ptr<T>( new T( std::forward<Args>(args)... ) );
     }
     至于为什么没有提供，C++ 标准委员会主席 Herb Sutter 在他的博客中提到原因
     是因为『被他们忘记了』。
     */
    //std::unique_ptr 是一种独占的智能指针，它禁止其他智能指针与其共享同一个对象，从而保证代码的安全：
    std::unique_ptr<int> unique_pointer = std::make_unique<int>(10); // make_unique 从 C++14 引入
    //std::unique_ptr<int> unique_pointer2 = unique_pointer; // 非法
    
    return 0;
}


```

Box.h
```
#ifndef Box_hpp
#define Box_hpp

namespace JNI {
namespace JDA {
class Box {
public:
      double length;   // 盒子的长度
      double breadth;  // 盒子的宽度
      double height;   // 盒子的高度
};
}
}

#endif /* Box_hpp */
```
多态
```
#ifndef Shape_hpp
#define Shape_hpp

#include <iostream>
using namespace std;
 
class Shape {
   protected:
      int width, height;
   public:
      Shape( int a=0, int b=0)
      {
         width = a;
         height = b;
      }
      virtual int area()
      {
         cout << "Parent class area :" <<endl;
         return 0;
      }

      virtual int area1() = 0;
};

class Rectangle: public Shape{
   public:
      Rectangle( int a=0, int b=0):Shape(a, b) { }
      int area ()
      {
         cout << "Rectangle class area :" <<endl;
         return (width * height);
      }
    int area1 ()
    {
       cout << "Rectangle class area :" << (width * height)<<endl;
       return (width * height);
    }
};

class Triangle: public Shape{
   public:
      Triangle( int a=0, int b=0):Shape(a, b) { }
      int area ()
      {
         cout << "Triangle class area :" <<endl;
         return (width * height / 2);
      }
    int area1 ()
    {
       cout << "Triangle class area :" << (width * height / 2)<<endl;
       return (width * height / 2);
    }
};

#endif /* Shape_hpp */
```


## 智能指针

C++11 引入了智能指针的概念，使用了引用计数的想法，让程序员不再需要关心手动释放内存。 这些智能指针就包括 
* std::shared_ptr
* std::unique_ptr
* std::weak_ptr

使用它们需要包含头文件 <memory>。

https://changkun.de/modern-cpp/zh-cn/05-pointers/index.html