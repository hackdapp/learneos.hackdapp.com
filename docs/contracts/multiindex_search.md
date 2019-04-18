# Multi-Index完整用法及示例讲解---数据检索篇

在本小节，将会为大家讲解五种数据查询方法：find、require\_find、get、lower\_bound、upper\_bound。

### find
**功能**： 基于主键或二级字段索引快速查询某条数据，并返回**const\_iterator**迭代器实例
```bash
const_iterator eosio::multi_index::find (
    uint64_t primary
) const
```
**参数说明**：
- primary 接收主键值
**返回结果**：当对二级字段索引进行查询时，可能会检索到多条数据记录，可通过迭代器进行循环查询, 会在迭代器小节文章中进行介绍。

**代码示例**： 查询主键值为1的用户
```bash
//1. 实例化索引器
user_index userestable(_self, _self.value); // code, scope

//2. 查询用户
auto iter = userestable.find(1); //此处直接返回便是const_iterator对象

//3. 业务逻辑断言
eosio_assert(itr->account_name == 1, "Incorrect user ");
```

注：**userestable.find(1)**方法返回的const\_iterator对象，如果想要访问实例属性，是需要按照指针调用的形式进行获取。 比如: iter-\> account\_name，只有直接通过数据结构体进行实例化时，才会使用user.account\_name的形式获取属性值。
比如: `user newinstance = user(); newinstance.account_name = "xxx";`。

### require\_find
功能： 基于主键字段索引快速查询某条数据，并返回**const\_iterator** 迭代器实例。如若不存在，则抛出事先定义的例外。
```bash
const_iterator eosio::multi_index::require_find (
    uint64_t primary,
    const char * error_msg = "unable to find key"
) const
```
**参数说明**：
- primary   
	所要查询的主键字段值
- error\_msg   
	所要查询的数据不存在时，所要抛出的例外信息。如果不定义则默认使用**unable to find key**提示信息。
**返回结果**：当对二级字段索引进行查询时，可能会检索到多条数据记录，可通过迭代器进行循环查询, 会在迭代器小节文章中进行介绍。

**代码示例**：查询主键值为1的用户
```bash
//1. 实例化索引器
user_index userestable(_self, _self.value); // code, scope

//2. 查询用户
auto iter = userestable.require_find(1, "not found object"); //此处直接返回便是const_iterator对象

//以上代码方式，其实与下面的实现逻辑相同。
auto iter = userestable.find(1);
eosio_assert(itr != userestable.end(), "not found object");
```

注： 在业务开发中，可根据不同场景选择使用find还是require\_find。 对于不允许为空的情况，可以使用require\_find方式使代码更加简洁；但如果是需要根据是否为空处理不同的逻辑，则可以find方式并配合值判空条件来处理不同业务逻辑。

### get
功能：基于主键字段索引快速某条数据，并返回其数据对象。
```bash
const T & eosio::multi_index::get (
    uint64_t primary,
    const char * error_msg = "unable to find key"
) const
```
**参数说明**：
- primary 主键参数
- error\_msg 例外消息。可自行定义消息内容。

**代码示例**：查询主键值为1的数据实例
```bash
//1. 实例化索引器
user_index userestable(_self, _self.value); // code, scope

userestable.emplace(_self, [&](auto& s){
	s.account_name = 1;
	s.age = 18;
});

//2. 查询用户
user user = userestable.get(1, "not found object"); //此处直接返回的便是数据结构体实例本身。

//3. 访问并打印用户名称
print user.account_name

```
### lower\_bound
功能：基于主键或二级字段索引， 查询大于或等于指定参数的**const\_iterator**迭代器对象。

```bash
const_iterator eosio::multi_index::lower_bound (
    uint64_t primary
) const
```

**参数说明**
- primary  
	主键字段值
**返回结果**：当对二级字段索引进行查询时，可能会检索到多条数据记录，可通过迭代器进行循环查询, 会在迭代器小节文章中进行介绍。

**代码示例**：查询年龄小于10的用户列表
```bash
user_index userestable(_self, _self.value); // code, scope

userestable.emplace(_self, [&](auto& s){
	s.account_name = 1;
	s.age = 19;
});
userestable.emplace(_self, [&](auto& s){
	s.account_name = 2;
	s.age = 18;
});
userestable.emplace(_self, [&](auto& s){
	s.account_name = 3;
	s.age = 1;
});

auto agestable = userestable.get_index<"byage"_n>();

//查询年龄大于且等于18的用户列表数据
auto iter = agestable.lower_bound(18);
eosio_assert(iter->account_name == 2, "Incorrect First Lower Bound Record ");
iter++;
eosio_assert(iter->account_name == 1, "Incorrect Second Lower Bound Record");
iter++;
eosio_assert(iter == agestable.end(), "Incorrect End of Iterator");

```
### upper\_bound
**功能**：基于主键或二级字段索引， 查询小于或等于指定参数的**const\_iterator**迭代器对象。
```bash
const_iterator eosio::multi_index::upper_bound (
    uint64_t primary
) const
```
**参数说明**：
- primary  
	主键字段值
**返回结果**：当对二级字段索引进行查询时，可能会检索到多条数据记录，可通过迭代器进行循环查询, 会在迭代器小节文章中进行介绍。

**代码示例**:
```bash
user_index userestable(_self, _self.value); // code, scope

userestable.emplace(_self, [&](auto& s){
	s.account_name = 1;
	s.age = 19;
});
userestable.emplace(_self, [&](auto& s){
	s.account_name = 2;
	s.age = 18;
});
userestable.emplace(_self, [&](auto& s){
	s.account_name = 3;
	s.age = 1;
});

auto agestable = userestable.get_index<"byage"_n>();

//查询年龄小于且等于18的用户列表数据
auto iter = agestable.upper_bound(18);
eosio_assert(iter->account_name == 3, "Incorrect First Upper Bound Record ");
iter++;
eosio_assert(iter->account_name == 2, "Incorrect Second Upper Bound Record");
iter++;
eosio_assert(iter == agestable.end(), "Incorrect End of Iterator");

```

----
通过本小节学习，我们学会了如何基于主键索引、二级索引完成我们对合约数据的单条数据查询或区间数据查询。
