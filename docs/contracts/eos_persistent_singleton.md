# 数据存储之单例用法（eosio::singleton）


在项目实际开发过程中，我们往往都需要定义一些系统全局参数配置，这类数据有一个存储特性就是它在使用过程中不需要进行多次数据存储与查询

比如：在去中心化交易系统中，可以通过修改全局配置，统一修改所有交易的手续费接收帐户；可以通过修改锁定参数，来开放或暂停交易所买、卖或搓合业务。

那么在EOS合约中，如何对全局配置进行合理化存储与查询呢？ 那便是**eosio::singleton**


----

**eosio::singleton**，是由eos平台所提供数据类型，与multi\_index作用相同，都可以对数据进行存储与查询。但唯一不同之处就是：multi\_index是进行多条数据存储，而**eosio::singleton**仅支持单条数据存储。

也正是因为它只存储单条数据的原因，如同设计模式中的单例模式一样，所以命名为singleton。

在具体使用方面，**eosio::singleton**和multi_index实例化方式相同，但所提供的函数方法有所不同。


|数据类型			|方法名			|备注			|
|:---				|:---				|:---				|
|		|singleton(name code, uint64\_t scope)	|构造函数				|
|bool	|exist()	|是否存在记录				|
|T		|get()		|获取单例数据				|
|T		| get\_or\_default (const T & def = T())	|获取单例数据，如果为空则返回缺少值				|
|T		| get\_or\_create (name bill\_to\_account, const T & def = T())				|获取单例数据。不存在则直接创建。				|
|void	| set (const T & value, name bill\_to\_account)|更新单例数据				|
|void	| remove ()	|清除单例数据				|

注：


## 代码完整示例

1. 定义数据结构体

		struct [[eosio::table]] configstruct {
			bool frozen;
			name feeadmin;
			
			EOSLIB_SERIALIZE(configstruct, (frozen) (feeadmin));
		};

2. 定义单例索引

		typedef singleton< "exchangecfg"_n, configstruct>  CfgHelper;
	在进行单例索引定义时，singleton\<模板名称, 数据结构体\>。 模板名称为name类型，所以长度不可超过13个字符

3. 索引实例与调用

		//1. 实例化单例类
		CfgHelper cfghelper(_self, _self.value);
		
		//2. 初始化或加载实例对象
		configstruct cfg;
		if(cfghelper.exists()){ 
			cfg = cfghelper.get();	//加载实例
		}else{
			cfg = configstruct();	//初始化实例
		}
		//或者直接使用 cfghelper.get_or_default() 代替以上逻辑判断
		cfg.frozen = true;
		
		//3. 合约数据存储
		cfghelper.set(cfg, _self);

## 思考与建议

### 1. 数据隔离
在实际合约开发，有时候可能需要进行数据隔离，防止所有数据都在一起。比如：针对用户的个性化配置，可能就需要按用户维度进行存储。

一般实现方式有两种：基于用户进行隔离、在表属性上定义用户属性。修改用户属性的方式可能大家都比较清楚，那么，我们就讲讲如何基于用户进行数据隔离, 其实现方式就是**基于表索引器构造参数(scope)**进行实现。

	构造函数： singleton (name code, uint64\_t scope)
	
	e.g 
	
	typedef singleton< "exchangecfg"_n, configstruct>  CfgHelper;
	
	//1. 按用户进行数据隔离, 意味着该索引器下所有的数据均为该用户数据。
	CfgHelper cfghelper(_self, <用户名>);
	
	//2. 统一数据存储， 意味着所用用户的数据都存储在一张整体表中。需要通过用户属性进行区分
	CfgHelper cfghelper(_self, _self);

### 2. 资源支付
当调用EOS事件方法时，往往都会伴随一些对合约数据的变动，中间执行过程就需要消耗一定CPU或RAM资源来运行，所以在编写阶段需要事先规划好谁来支付这笔消耗费用，是用户还是合约本身。

示例如下：
	typedef singleton< "exchangecfg"_n, configstruct>  CfgHelper;
	CfgHelper cfghelper(_self, <用户名>);
	
	//1. 由合约自身支付
	cfghelper.set(<对象实例>, _self);
	
	//2. 由用户自己支付
	cfghelper.set(<对象实例>, <用户帐户>);

	
## 参考资料
[https://eosio.github.io/eosio.cdt/1.5.0/classeosio\_1\_1singleton.html](https://eosio.github.io/eosio.cdt/1.5.0/classeosio_1_1singleton.html)
