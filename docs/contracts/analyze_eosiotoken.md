# 剖析eosio.token合约

本章节将带你一起剖析官方智能合约eosio.token，通过整个合约的分拆讲解，让你搞清楚一个完整的合约所必须要实现的各个代码环节。

----
首先，我们先从[eos官方仓库](https://github.com/EOSIO/eosio.contracts)中，找到eosio.token系统合约。

![](http://cdn.hackdapp.com/2019-03-14-111345.jpg) ![](http://cdn.hackdapp.com/2019-03-14-111418.jpg)

从文件结构目录来看， eosio.token合约是由两部分组成，即eosio.token.hpp 、eosio.token.cpp两个文件。 其中.hpp文件主要用于定义合约的接口方法以及数据结构体；而.cpp主要针对接口中的方法进行扩展实现。

这种面向接口实现的设计，一方面是为了与第三方对接时达到隐藏实现的目的；另一方面其实就是用于对外明确接口方法。

然后，打开eosio.token.hpp接口文件，该文件定义了合约的主体结构，主要分为两部分进行数据定义:


```bash
namespace eosio {

   using std::string;

   class [[eosio::contract("eosio.token")]] token : public contract {
		public:
	        using contract::contract;

			//do some stuff

		private:

			//do some stuff			
   }
}
```

`public`部分主要用于定义对外部公开可访问的方法，即可以通过合约进行访问的方法；`private`部分主要用于定义内部方法及私有结构数据，而这些方法只允许本合约内调用。

----
接下来，我们具体看一下`private`部分所要实现的具体内容


```bash
struct [[eosio::table]] account {
  asset    balance;

  uint64_t primary_key()const { return balance.symbol.code().raw(); }
};

struct [[eosio::table]] currency_stats {
  asset    supply;
  asset    max_supply;
  name     issuer;

  uint64_t primary_key()const { return supply.symbol.code().raw(); }
};

typedef eosio::multi_index< "accounts"_n, account > accounts;
typedef eosio::multi_index< "stat"_n, currency_stats > stats;

void sub_balance( name owner, asset value );
void add_balance( name owner, asset value, name ram_payer );
```

以上代码部分中，主要实现了两部分内容：数据结构体定义、索引容器定义。

**数据结构体**，其实可以理解为传统数据库中表结构。在结构体中，需要实现两部分内容：字段定义、索引字段方法定义。

字段数据类型主要分为C++基础数据类型以及EOS平台所封闭的数据类型，比如：asset、symbol、name、capi\_checksum160等。可通过查阅[官方文档](https://eosio.github.io/eosio.cdt/1.5.0/modules.html)，或直接从cdt [源码](https://github.com/EOSIO/eosio.cdt/tree/v1.3.2/libraries/eosiolib)中直接查找。

**索引容器**，其实就是用于对容器所对应的表数据进行数据存储与查询。multi\_index索引容器支持对主键、其他索引字段的多维度数据查询。比于：查询某个字段大于或小于某个数值的数据查询、查询某条数据，遍历所有数据等等。multi\_index的缺省实现是根据主键进行数据查询的，所以需要确保在数据结构体中一定要实现`primary_key() `方法。另外，multi\_index的二级索引定义可以[查询文档](https://eosio.github.io/eosio.cdt/1.5.0/group__multiindex.html)。

示例：多索引的使用方式

```bash
  struct record {
    uint64_t    primary;
    uint64_t    secondary_1;
    uint128_t   secondary_2;
    checksum256 secondary_3;
    double      secondary_4;
    long double secondary_5;
    uint64_t primary_key() const { return primary; }
    uint64_t get_secondary_1() const { return secondary_1; }
    uint128_t get_secondary_2() const { return secondary_2; }
    checksum256 get_secondary_3() const { return secondary_3; }
    double get_secondary_4() const { return secondary_4; }
    long double get_secondary_5() const { return secondary_5; }
  };

 multi_index<"mytable"_n, record,
        indexed_by< "bysecondary1"_n, const_mem_fun<record, uint64_t, &record::get_secondary_1> >,
        indexed_by< "bysecondary2"_n, const_mem_fun<record, uint128_t, &record::get_secondary_2> >,
        indexed_by< "bysecondary3"_n, const_mem_fun<record, checksum256, &record::get_secondary_3> >,
        indexed_by< "bysecondary4"_n, const_mem_fun<record, double, &record::get_secondary_4> >,
        indexed_by< "bysecondary5"_n, const_mem_fun<record, long double, &record::get_secondary_5> >
      > table( code, scope);
```
----
接下来，继续分析`public `部分的接口定义

```bash
[[eosio::action]]
void transfer( name    from, name    to, asset   quantity, string  memo );
```
`
``transfer`接口方法明确了具体的参数及参数类型； `[[eosio::action]] `注解主要用于生成abi文件以及对外开放合约方法。


```bash
static asset get_supply( name token_contract_account, symbol_code sym_code )
{
  stats statstable( token_contract_account, sym_code.raw() );
  const auto& st = statstable.get( sym_code.raw() );
  return st.supply;
}
```
当我们业务中存在重复的业务逻辑时，为了提高复用，所以需要重构代码，封装类似于`get_supply `这样的工具方法。`get_supply `方法是不会对外进行开放的。

----
最后，我们来分析eosio.token.cpp实现部分，该文件其实是整个智能合约的核心。

作为合约接口方法的扩展实现，需要考虑四部分内容：1）对于入参的合法性校验；2）对于权限的校验；3）业务逻辑及数据的存储；4) 选择合适的合约间交互方式。

```bash
void token::create( name   issuer,
                    asset  maximum_supply )
{
    require_auth( _self );

    auto sym = maximum_supply.symbol;
    eosio_assert( sym.is_valid(), "invalid symbol name" );
    eosio_assert( maximum_supply.is_valid(), "invalid supply");
    eosio_assert( maximum_supply.amount > 0, "max-supply must be positive");

    stats statstable( _self, sym.code().raw() );
    auto existing = statstable.find( sym.code().raw() );
    eosio_assert( existing == statstable.end(), "token with symbol already exists" );

    statstable.emplace( _self, [&]( auto& s ) {
       s.supply.symbol = maximum_supply.symbol;
       s.max_supply    = maximum_supply;
       s.issuer        = issuer;
    });
}
```
在上述方法，首先通过`require_auth `验证调用者是否具有合约权限; 然后，便是使用`eosio_assert `对入参进行断言； 最后，便是使用`multi_index `实现对数据的存储

还有最后一个环节，针对合约配置方法路由及访问权限，只有在此配置了对外提供的方法才可能会被合约所执行。

```bash
EOSIO_DISPATCH( eosio::token, (create)(issue)(transfer)(open)(close)(retire) )
```
