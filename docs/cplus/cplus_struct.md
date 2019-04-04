# 数据结构体

在实际需求设计与开发过程中，我们往往需要业务需求抽象出不同的领域模型，方便对业务数据的结构化存储。而在编程语言中所提供的基础数据类型或者数据结构，是无法满足这样的开发需求的。

所以，就需要我们根据业务需求定义一些复杂的结构体，而结构体本身其实也是由基础数据类型，或者集合类型组合而成的。

比如： 在电商平台业务系统中，如何定义一个订单结构体呢？


```
struct Order {
    uint64_t    id;         //订单编码
    uint64_t    productid;  //产品关联编码
    double      price;      //产品定价
    int         qunitity;   //购买数量
    int         status;     //订单状态
    uint64_t    userid;     //用户编码
}
```

作为新的数据类型，我们同样可以使用在变量定义、常量定义以及函数入参中使用。比如：

```
Order order1, order2;

Vector<Order> mergeOrders(Order order1, Order order2){
    //TODO
    return ...;
}
mergeOrders(order1, order2);

struct Person{
    uint64_t id;
    string name;
} usera, userb, userc;
```

那么如何在程序中初始化新结构体对象呢？

```
Order order;
order.id = xxx;
order.productid = xxx;
......
order.status = xxx;
```
