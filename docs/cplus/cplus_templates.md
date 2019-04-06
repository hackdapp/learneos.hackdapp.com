# 模板定义与使用

模板作为ANSI-C++新引入的功能，其存在意义在于复用代码逻辑，而在实例时动态构建不同类型数据结构或

模板类型分为两种：函数模板、类模板。

## 函数模板
函数模板允许创建一个通用函数，使用任意数据类型作为其参数及返回值，而不必使用重载方式。

我们可以使用以下两种形式进行函数模板定义

```
template <class identifier> function_declaration;
template <typename identifier> function_declaration;
```

虽然两种定义方式中关键字`class`和`typename`有所不同，但其具体作用效果却是一样的。

举个例子, 创建一个模板函数，计算两数之和

```
template <class GenericType>
GenericType GetPlus (GenericType a, GenericType b) {
  GenericType result;
  result = a + b;
  return (result);
}

```

在示例中第一行，我们定义了一个通用数据类型，并在函数中使用通用数据类型来代替具体的参数类型及返回值数据类型。

但需要说明的是`GenericType`并不代替具体的数据类型，而是用于临时代替在函数进行实际调用时所设定的数据类型。比如：
```
function <GenericType> (parameters)
```

那么，计算两个整型之和的模板写法应该如下：
```
double j = 1.12;
double k = 2.32;
double l = GetPlus<double>(j, k);


```
完整示例代码如下:
```
//http://tpcg.io/UsChdl
#include <iostream>

using namespace std;

template <class GenericType>
GenericType GetPlus (GenericType a, GenericType b) {
  GenericType result;
  result = a + b;
  return (result);
}


int main()
{
    int x = 10, y = 12;
    int z = GetPlus<int>(x, y);

    double j = 1.12, k = 2.32;
    double l = GetPlus<double>(j, k);

    cout << z << endl;
    cout << l << endl;

    return 0;
}

```
需要注意的是，当在调用模板函数时，如果没有明确指定数据类型，即没有设置函数 \<\>中数据类型时，编译器会自动确定每次所需的类型。

因为我们的模板函数只允许一种数据类型，所以当你传入两种数据类型时，编译器会提示错误，如下代码调用方式是错误的
```
int x = 19;
long y = 20;
GetPlus(x, y)
```

## 类模板
同样，C++同时也支持使用函数模板的方式进行类模板的定义与创建。我们可以使用GenericType的形式定义类的成员变量类型。

比如:
```
template <class T>
class pair {
    T values [2];
  public:
    pair (T first, T second)
    {
      values[0]=first; values[1]=second;
    }
};
```

我们刚定义的pair类可以存储任何类型的数值。比如:
```
int x = 10, y= 11;
pair<int> mypair1(x, y);

long j = 12, k = 33;
pair<long> mypair2(j, k);
```

完整代码示例如下:
```
//http://tpcg.io/BOA7qu

// class templates
#include <iostream>

template <class T>
class pair {
    T value1, value2;
  public:
    pair (T first, T second)
      {value1=first; value2=second;}
    T getmax ();
};

template <class T>
T pair<T>::getmax ()
{
  T retval;
  retval = value1>value2? value1 : value2;
  return retval;
}

int main () {
  pair <int> myobject (100, 75);
  std::cout << myobject.getmax();
  return 0;
}
```
----

**特殊用法一：特定类型设置**

当我们定义一个模板函数后，可能想针对某一种特定数据类型进行特殊处理
```
#include <iostream>


template <class T>
class pair {
    T value1, value2;
  public:
    pair (T first, T second)
      {value1=first; value2=second;}
    T module () {return 0;}
};

template <>
int pair<int>::module() {
  return value1%value2;
}

int main () {
  pair <int> myints (100,75);
  pair <float> myfloats (100.0,75.0);
  std::cout << myints.module() << '\n';
  std::cout << myfloats.module() << '\n';
  return 0;
}
```
以上代码示例中，除整型调用module方法时是按照取余逻辑进行处理之外，其他数据类型调用module方法时都返回0

**特殊用法二：模板多参数**
在前面的示例中，你会发现在使用模板定义`template <class T>`时，只使用了一个`class T`参数，其实是可以定义多个模板参数的。

```
// array template
#include <iostream>

template <class T, int N>
class array {
    T memblock [N];
  public:
    void setmember (int x, T value);
    T getmember (int x);
};

template <class T, int N>
void array<T,N>::setmember (int x, T value) {
  memblock[x]=value;
}

template <class T, int N>
T array<T,N>::getmember (int x) {
  return memblock[x];
}

int main () {
  array <int,5> myints;
  array <float,5> myfloats;
  myints.setmember (0,100);
  myfloats.setmember (3,3.1416);
  std::cout << myints.getmember(0) << '\n';
  std::cout << myfloats.getmember(3) << '\n';
  return 0;
}
```

同样，模板参数也支持缺省值的设定。 例如，一些可能出现的多参数模板定义如下:

```
template <class T>              // The most usual: one class parameter.
template <class T, class U>     // Two class parameters.
template <class T, int N>       // A class and an integer.
template <class T = char>       // With a default value.
template <int Tfunc (int)>      // A function as parameter.
```
