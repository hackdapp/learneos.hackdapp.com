# 同步与异步调用合约方法（Deferred Action、Inline Action）

EOS智能合约中支持两种Action进行合约间方法调用。**Inline actions** 可理解为同步执行操作；**Deferred actions**为异步执行操作。

**Inline actions**，强调在同一事务中的关联方法拥有与调用入口方法相同的权限。它们可以保证当前关联方法中的任何一个方法出现错误，都会回滚整个交易数据。另外，需要说明的是一个交易中的所有方法都是在同一个区块中执行的。

**Deferred actions**，是合约设定为将来某一个时刻进行执行的方法。它与**Inline actions**不同之处在于：它不能保证被执行。因为合约方法内进行**Deferred action**调用时，也就意味着新开启了一个事务。而这个事务最终能否被生产节点所接受其实并不能确定。但即使是失败，**Deferred action** 也并不会对源方法造成数据回滚影响，因为当延迟方法开始执行时，可能源方法所在区块已经被生产出来了。

可能有人会问，既然**Deferred actions**不能确定是否被执行成功，那么它还有存在的意义吗？

其实，**Deferred actions** 在实际开发场景中，确实是有其存在的意义。只不过需要结合不同业务场景使用，不妨想一想，我们一个合约方法中可能业务逻辑稍微复杂一些，而EOS链又要求所有的合约方法执行时间不可超过30ms，那很可能复杂的方法就无法执行了。但如果这时，你将一些非强事务型的方法，进行抽取改为异步执行，这样就优化方法的执行时间确保合约方法的正常执行。

那么，哪些情况是属于非强事务型的方法呢？

比如：日志记录，可重复执行的方法（即异步执行出错时，我们可以通过手动调用来弥补错误）。

接下来，我们通过示例代码的形式，让大家理解整个过程：

----

首先，创建一个新的合约, 定义两个方法`send `、`deferred `。在`send `方法中发起对`deferred `方法的延迟调用

```
	void dexchange::deferred(name from, const std::string &message){
	    require_auth(from);
	    print("Printing deferred ", from, message);
	}

	void dexchange::send(name from, const std::string &message, uint64_t delay){
	    require_auth(from);

	    eosio::transaction t{};
	    t.actions.emplace_back(
	        action(
	            eosio::permission_level(from, "active"_n),
	            _self,
	            "deferred"_n,
	            std::make_tuple(from, message)
	        )
	    );

	    t.delay_sec = delay;
	    t.send(now(), from);

	    print("Scheduled with a delay of ", delay);
	}
```

然后，再定义一个`onError `方法，当EOS链发现延迟方法执行失败时， 会由系统触发合约中的`onError `方法，返回错误信息

```
	void onError(const onerror &error){
		print("Resending Transaction: ", error.sender_id);
		transaction dtrx = error.unpack_sent_trx();
		dtrx.delay_sec = 3;
		dtrx.send(now(), _self);
	}
```

下一步、定义宏定义，根据系统通知来决定如何路由请求。

```
	extern "C" {
	  void apply(uint64_t receiver, uint64_t code, uint64_t action){
	    if (code == "eosio"_n.value && action == "onerror"_n.value){
	        eosio::execute_action(eosio::name(receiver), eosio::name(code), &dexchange::onError);
	    }

	   if (code != receiver)
	      return;

	    switch (action) {
	         EOSIO_DISPATCH_HELPER(dexchange, (send)(deferred));
	    };
	    eosio_exit(0);
	  }
	}

```

上面这个代码示例中，apply方法可以看到有三个参数: receiver、code、action. receiver参数就是我们合约的帐户名； code的是要取决于合约方法调用方，假如是合约调用自己内部的方法，那么code与receiver一致，假如是其他合约调用本合约的方法，则code为其他合约的帐户名；至于action便是调用的方法名称。

在这里之所以使用宏定义，就是因为在实际的开发场景中，我们往往需要判断code来判断是不是合法的合约帐户发送过来的请求。比如：进行交易所EOS资金充值时，往往交易合约是根据收到eosio.token的transfer消息通知时才会进行交易所资金的修改，如果不在宏定义处通过code判断消息来源，那很可能就会造成假充值。

最后，发布智能合约，调用`send `合约方法来进行测试。另外，延迟方法在没有执行之前，其实是可以通过`cancel_deferred `方法进行交易取消的。

----

**小结**

通过本文的学习，我们了解到`deferred `与`inline `方法间的差异性以及如何根据业务场景选择不同的方法进行代码实现。
