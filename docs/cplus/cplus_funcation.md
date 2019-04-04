## 函数定义

函数，其实就是将原始数据通过数据加工后，返回指定要求的数据。

原始数据其实就是函数的入参数据，所谓的数据加工其实就是函数体所要处理的业务逻辑，最后的数据结果类型就是函数所定义的返回类型。

根据返回值类型， 函数形式分为两种：

### 1. 有返回值函数

```
<数据类型> 函数名(参数1， 参数2, ...)
{
    //函数体
    return <返回值>;
}
```

比如： 1 + 2 = 3

```
int add(int a, int b)
{
    return a + b;
}
```

### 2. 无返回值函数

```
void 函数名(参数1， 参数2, ...)
{
    //函数体
    //无返回值，所以不需要返回数据
}
```


**根据传参数参数方式不同， 又可以分为值传递与引用传递**

##### 1) 值传递

值传递，就是我调用某一个函数后，函数内部不论如何修改参数，并不会修改原有参数值内容。


##### 2) 引用传递

引用传递，就是当函数内部重新设置或修改参数值对象时，会连同修改原有参数值内容。


所以大家在设计函数时，需要明确函数内部是否要修改原有对象，如果不允许修改的化，在定义函数时在入参数前面可以添加const常量修改符，防止出现莫名修改数据的问题。


## 多态

---

### 1. 函数重载

函数重载，是指在同一作用域下， 函数名称相同，但参数类型或个数不同的多个函数。

比如，你要实现一个计算不同数据类型和的工具类, 可能实现如下:

 `````````
 int add(int a, int b)
 {
    return a + b;
 }

 int add(int a, int b, int c)
 {
    return a + b + c;
 }

 float add(float a, float b)
 {
    return a + b;
 }
 double add(double a, double b)
 {
    return a + b;
 }
 `````````

调用过程如下:

```
int a = 1, b = 2;
int c = add(a, b);

float a = 1.2, b = 2.2;
float c = add(a, b);
```

在实际的开发场景中，函数重载往往用于处理内部实现逻辑一致，但参数可能不同的情况。而参数在程序过程中会自动根据参数类型或个数选择对应的方法进行调用处理。

### 2. 函数覆盖

函数覆盖, 是指基类与派生类中的两个函数使用相同的函数名、相同的入参与相同的返回类型。可能内部实现不所不同，派生类函数使用新的业务逻辑替代基类中的函数实现。

```
#include <iostream>
#include <stdlib.h>

using namespace std;

class Animal
{
    public:
        virtual void run()
        {
            cout << "the animal run." << endl;
        }
};

class Monkey : public Animal
{
    public:
        void run()
        {
            cout << "the monkey run." << endl;
        }
};

class Bull : public Animal
{
    public:
        void run()
        {
            cout << "the Bull run." << endl;
        }
};

int main()
{

    Monkey monkey;
    Bull bull;

    Animal* inst;

    inst = &monkey;
    inst->run();    //调用派生类实现, 输出结果为: the monkey run.

    inst = &bull;
    inst->run();    //调用派生类实现, 输出结果为： the Bull run.

}
```

### 3. 函数隐藏

函数隐藏，是指派生类与基类存在相同名称的函数。不考虑数列表是否一致，从而隐藏基类中的同名函数。

隐藏分两种情况：
1）派生类与基类中的函数完全相同(函数名称， 入参列表)，只是基类的函数没有使用virtual关键字
2) 派生生类与基类中的函数名相同，但入参参数不同。此种情况，不管理基类的函数是否使用virtual关键字，基类的函数都会被隐藏。另外，这个与之前介绍的函数重载是有区别的，重载发生在同一个类中。

在函数隐藏的情况下，不管指针实际指向的对象是什么类型，完全根据指针类型，执行对应的成员函数。

比如：在上一个函数覆盖的例子中，如果Animal类中的run函数未使用virtual，则执行结果为：

```
int main()
{

    Monkey monkey;
    Bull bull;

    Animal* inst;

    inst = &monkey;
    inst->run();    //调用派生类实现, 输出结果为: the animal run.

    inst = &bull;
    inst->run();    //调用派生类实现, 输出结果为： the animal run.

    //tryit: http://tpcg.io/uHUkzy
}
```

另外，如果系统在调用析构函数时，同样也只会调用基类中的析构函数，不会调用派生类的析构函数。

所以，在开发过程中一定要注意在覆盖函数时，确保添加virtual修饰符，或者同时使用overridew修饰符，然后通过编译错误提醒自己。
