# 存储合约数据

本小节将通过一个简单的示例带大家了解如何实现对数据的CRUD（增、删、改、查）功能。通过本小节学习，你可以学习到：
- 数据结构定义以及如何设置主键索引
- 合约方法定义
- 索引器定义
- 增、删、改、查接口方法使用

----

```
//filename: userlist.hpp
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>

using namespace eosio;

CONTRACT userlist : public contract {
  public:
    using contract::contract;
    userlist(eosio::name receiver, eosio::name code, datastream<const char*> ds):contract(receiver, code, ds) {}

    [[eosio::action]]
    void add(std::string username, uint64_t age);

    [[eosio::action]]
    void modify(uint64_t id, std::string username, uint64_t age);

    [[eosio::action]]
    void del(uint64_t id);


  private:

    struct  [[eosio::table]] tbl_user {
      uint64_t id;
      std::string username;
      uint64_t age;

      uint64_t primary_key() const { return id; }

      uint64_t byage() const { return age; }

    };
    typedef eosio::multi_index<"tbluser"_n, tbl_user,
		indexed_by< "byage"_n, const_mem_fun<tbl_user, uint64_t, &tbl_user::byage>>
	> user_index;
};

EOSIO_DISPATCH(userlist, (add)(modify)(del))
```

```
//filename: userlist.cpp

#include "userlist.hpp"


void userlist::add(std::string username, uint64_t age) {
  userlist::user_index user_stable(_self, _code.value);

  user_stable.emplace(_self, [&]( auto& s ) {
    s.id = user_stable.available_primary_key();
    s.username = username;
    s.age = age;
  });
}

void userlist::modify(uint64_t id, std::string username, uint64_t age) {
  userlist::user_index user_stable(_self, _code.value);
  auto item =  user_stable.find(id);

  eosio_assert(item != user_stable.end(), "the data isn't exist.");

  user_stable.modify(item, _self, [&](auto& s){
    s.username = username;
    s.age = age;
  });
}

void userlist::del(uint64_t id) {
  userlist::user_index user_stable(_self, _code.value);
  auto item =  user_stable.find(id);

  eosio_assert(item != user_stable.end(), "the data isn't exist.");

  user_stable.erase(item);
}
```

以上两个文件userlist.hpp和userlist.cpp向我们展示了一个对用户的增删改查完整示例合约。

而完成以上存储功能，至少需要完成两部分数据定义:
- 数据结构定义
  数据结构体是业务数据的存储载体；也直接决定了业务数据在数据库中的表结构模型；
- 索引定义
  索引定义，主要用于按照不同索引字段执行不同的数据检索或排序；
- 索引器定义
  索引器，主要用于提供对数据进行增删改查的方法调用等

接下来，让我们分步骤拆分其具体实现过程，以及在过程中应该注意的事项：

## 第一步、定义数据结构体

```
struct [[eosio::table]] tbl_user {
    uint64_t id;
    std::string username;
    uint64_t age;

    uint64_t primary_key() const { return id; }
};
```

定义数据结体体时，一定要根据具体业务要求，选择合适长度的字段数据类型，因为在合约中存储数据是需要消耗RAM资源的，合适的数据类型可以帮我们节省不少资源。

另外，在定义数据结构体中的`[[eosio::table]]`, 该注解主要用于在生成ABI文件时选择是否生成具体的表结构。

## 第二步、定义主键字段

在关系型数据库中，往往都需要我们设置某个字段为主键字段。同样，在EOS智能合约中，也是如此。那么如何进行设置呢？

其实超级简单。就是在数据结构体定义中，对要定义的主键字段，实现`primary_key`方法即可，该方法主要用于告诉索引器哪个字段是主键字段。该方法为系统缺省方法，只要你进行数据存储那么就必须定义此方法。

另外，需要注意的是：`primary_key`方法为常量方法，在实现过程中一定注意添加`const`关键字，否则无法通过编译。

那么，可能有人会问，那如何想按照年龄字段进行数据查询，怎么办呢？

同样，也需要在数据结构体中定义具体字段的查询方法，只不过方法名称可以由自己设定。比如：
```
uint64_t byage() const { return age; }
```
这类索引的定义我们称之为二级索引。


## 第三步、索引器定义

数据结构定义完成之后，如何对数据进行访问呢？ 那便是----索引器Multi-Index

```
typedef eosio::multi_index< [TableName], [T],
  	indexed_by<
		    [IndexName],
		    const_mem_fun<[T], [IndexFieldType], [FieldGetter]>
	   >
  > [index_type];
```

**TableName**：数据结构体所对应的表名，其名称长度不可超过12个字符，且名称只允许由小写字母、数字1到5、. 三种字符构成。
**T**： 数据结构体名称
**index\_type**: 索引器类型别名。需要针对不同的结构体进行不同的索引器定义。

**indexed_by**: 主要用于定义多个二级索引
而其中具体参数说明如下：
- **IndexName**: 二级索引名称。其命名长度不可以超过13个字符，其中前12个字符只允许由小写字母、数字1到5、. 三种字符构成，第13个字符只允许从a-p小写字母或.两类字符中选择
- **T**： 数据结构体名称
- **IndexFieldType**：二级索引字段数据类型。目前所支持的数据类型仅支持以下几种
  - uint64\_t
  - uint128\_t
  - eosio\::checksum256
  - double
  - long double。
- **IndexFieldGetter**：数据结构体属性getter方法

举个例子：
```
typedef eosio::multi_index<"tbluser"_n, tbl_user,
		indexed_by< "byage"_n, const_mem_fun<tbl_user, uint64_t, &tbl_user::byage>>
	> user_index;
```
以上示例便是`user`数据结构的索引器定义，其中包括对年龄二级索引字段的定义

## 第四步、索引实例化及增删改查方法调用

在对数据操作之前，我们先需要对索引器`multi_index (name code, uint64_t scope) `进行实例化时，此时需要传入两个参数`code`、`scope`
**code** 参数： 表所归属的合约帐户，即你所要查询或操作的数据表是哪个合约帐户下面的表数据；比如：通过eosio.token查询某个帐户的余额信息，code即为eosio.token
**scope** 参数： 主要用于对表数据的分表处理。即你可以将所有数据都存在某一个scope数据维度下；也可以根据不同scope数据进行分类存储。比如按帐户进行数据隔离，scope参数就传入对应帐户名。如果按照合约帐户进行存储，则所有数据都存在合约帐户下。

比如：该示例便将所有数据都存储在合约帐户数据中。

```
user_index userestable(_self, _self.value); // code, scope
```

接下来，我们讲解如何对数据进行具体的CRUD操作：

### 1）添加

```
void userlist::add(std::string username, uint64_t age) {
  userlist::user_index user_stable(_self, _code.value);

  user_stable.emplace(_self, [&]( auto& s ) {
    s.id = user_stable.available_primary_key();
    s.username = username;
    s.age = age;
  });
}
```

上述代码展示了如何保存一条用户数据到智能合约中。首先，我们需要根据事先定义的索引`userlist::user_index`进行实例化，实例化时需要传入两个参数。
第一个参数即表的归属合约帐户，即这张表属于哪个合约。如果查询的是自己合约的表，就传入自己合约的合约帐户名；如果查询的是其他合约中的表，则需要传入对方合约的合约帐户名；第二参数为数据的存储维度，可以理解为分库或分表逻辑。例如：此参数传用的是用户名称，则代表按用户维度隔离彼此的数据。

`user_stable.emplace(_self, [&]( auto& s )`该行代码为保存数据的索引方法，第一个参数代表谁将为此次数据存储支付相应的资源消耗。如果为`_self`则代表由合约本身支付，如果为用户自己则代表用户需要为此支付资源消耗。

`user_stable.available_primary_key`该行代码展示了如何获取主键字段自增id值。

### 2）修改

```
void userlist::modify(uint64_t id, std::string username, uint64_t age) {
  userlist::user_index user_stable(_self, _code.value);
  auto item =  user_stable.find(id);

  eosio_assert(item != user_stable.end(), "the data isn't exist.");

  user_stable.modify(item, _self, [&](auto& s){
    s.username = username;
    s.age = age;
  });
}

```

修改方法与添加方法的不同之处在于：需要使用（查询find、修改modify）两个方法才能完成数据的更新操作，即先从库中查询到具体数据记录，然后通过数据的指针引用进行修改操作。

`eosio_assert` 方法主要用于对参数或业务逻辑判断的断言处理。

`user_stable.modify(item, _self, [&](auto& s) ` 该行代码第一个参数为需要修改数据的地址引用，第二个参数为谁为此次修改操作花费相应的资源消耗。

### 3）删除

```
void userlist::del(uint64_t id) {
  userlist::user_index user_stable(_self, _code.value);
  auto item =  user_stable.find(id);

  eosio_assert(item != user_stable.end(), "the data isn't exist.");

  user_stable.erase(item);
}
```

删除方法也是需要先定位到修改对象的地址引用，然后通过`erase`进行合约数据删除操作。

需要注意的是，`multi_index`本身并未提供清空整张表的API方法，所以如果要清空数据表只能选择循环迭代的方式一个个进行删除操作。但需要按以下方式进行删除：

```
//清空表数据
userlist::user_index user_stable(_self, _code.value);
auto itr = user_stable.begin();
while(itr != user_stable.end()){
	itr.earse(itr);
	//注意：不可以添加itr++
}
```

### 4）查询

在智能合约实现中，往往我们不需要对外提供数据查询的合约方法，因为EOS节点服务支持我们通过RPC接口的方式直接查询智能合约中的表数据，这样更加适合中心化服务灵活配置自己的查询需求。

**查询单条数据**
```
userlist::user_index user_stable(_self, _code.value);

user_stable.find(id); //根据主键查询数据

```
**查询多条数据**
```
userlist::user_index user_stable(_self, _code.value);
auto ageindex = user_stable.get_index<"byage"_n>();
auto itr = ageindex.find(20);
while(itr != ageindex.end()){
	//print
}
```
以上示例为通过二级索引方式对年龄字段进行数据查询，即查询所有年龄为20的用户列表。

---
通过本小节的学习，我们学会了数据结构的定义、主键及二级索引字段的定义、索引器定义以及索引器具体的CURD操作方法。
