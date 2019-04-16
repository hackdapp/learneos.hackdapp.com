## 一、Multi-Index完整用法及示例讲解---索引器定义篇

本小节将向大家介绍如何对自定义数据结构体定义主键索引、二级字段索引，并通过代码示例的形式方便大家理解。

本文将分两部分进行知识点讲解：
- 普通索引定义
- 二级索引定义

----
### 1. 普通索引定义

```
typedef eosio::multi_index< [TableName], [T] > [index_type];
```

索引定义主要由三个参数进行定义：

**TableName**：数据结构体所对应的表名，其名称长度不可超过12个字符，且名称只允许由小写字母、数字1到5、. 三种字符构成。
**T**： 为数据结构体名称
**index\_type**: 索引器类型别名。需要针对不同的结构体进行不同的索引器定义。

因为缺省索引器是基于主键时查询的，所以在定义结构时，需内部必须实现一个名为`primary_key() `且返回值必须为uint64\_t数据类型的成员常函数。

要实现一个简单的索引器定义，可分为以下四个步骤：
1. 定义结构体
	```
	struct user {
		uint64_t account_name;
		uint64_t age;
	};
	```
2. 定义主键查询方法
	在上述结构体中，实现`primary\_key() `方法
	```
	uint64_t primary_key() const { return account_name; }
	```
3. 定义索引器
	```
	typedef eosio::multi_index< "user"_n, user > user_index;
	```
4. 实例化索引器
	在实例化索引器`multi_index (name code, uint64_t scope) `时，需要传递两个参数： code、scope。**code** 参数指表所归属的合约帐户，即你所要查询或处理的数据是哪个合约帐户下面的数据；**scope** 参数主要用于对表数据的分表处理。比如按帐户进行数据隔离，scope参数就传入对应帐户名。
	```
	user_index userestable(_self, _self.value); // code, scope
	```

完整示例如下:
```
#include <eosiolib/eosio.hpp>
using namespace eosio;
using namespace std;

class booking : contract {
  struct user {
     uint64_t account_name;
     uint64_t age;

     uint64_t primary_key() const { return account_name; }
  };
  public:
    booking (name self):contract(self) {}

    typedef eosio::multi_index< "user"_n, user > user_index;

	void createuser(name user, uint64_t age) {
      user_index userestable(_self, _self.value); // code, scope
    }
}
EOSIO_DISPATCH( booking , (myaction) )
```

----


### 2. 二级索引定义
通常情况下，在实际的业务场景中，很可能会对某一张表数据进行多个字段的数据表统计查询。

那么，这时候就需要进行二级索引定义。
```
typedef eosio::multi_index< [TableName], [T],
  	indexed_by<
		[IndexName],
		const_mem_fun<[T], [IndexFieldType], [FieldGetter]>
	>
  > user_index;
```

与普通索引器的定义方式相比，二级索引多出一个`indexed\_by`的扩展定义。`indexed\_by`需要进行四个参数定义：
```
indexed_by<
		[IndexName],
		const_mem_fun<[T], [IndexFieldType], [IndexFieldGetter]>
	>
```

**IndexName**: 二级索引名称。其命名长度不可以超过13个字符，其中前12个字符只允许由小写字母、数字1到5、. 三种字符构成，第13个字符只允许从a-p小写字母或.两类字符中选择
**T**： 数据结构体名称
**IndexFieldType**：二级索引字段数据类型。目前所支持的数据类型仅支持以下几种
- uint64\_t
- uint128\_t
- eosio::checksum256
- double
- long double。
** IndexFieldGetter**：数据结构体属性getter方法

**注** ：索引器目前最多支持16个二级索引。

现在对普通索引器所提供的代码示例，进行二级索引扩展，新增`age`字段二级索引
1.  在数据结构体中，添加二级索引字段
	```
	struct user {
		uint64_t account_name;
		uint64_t age;	//新增索引字段

		uint64_t primary_key() const { return account_name; }
	}
	```
2. 在数据结构体中，定义对二级索引字段的查询方法
	```
	uint64_t by_age() const { return age; }
	```
	在user数据结构体中，定义对**age**字段的查询方法
3. 在索引器中添加二级索引定义
	```
	typedef eosio::multi_index< "user"_n, user,
		indexed_by<
			"byage"_n,
			const_mem_fun<user, uint64_t, &user::by_age>
		>
	> user_index;
	```
4. 实例化索引器，并获取二级索引实例
	```
	//1. 实例化索引器
	user_index userestable(_self, _self.value); // code, scope

	//2. 实例化二级索引
	auto age_index = userestable.get_index<"byage"_n>();
	```

----
通过本小节的学习，我们熟悉了整个索引器从定义数据结构体、主键与二级索引定义以及索引器、二级索引实例化的完整开发流程。
