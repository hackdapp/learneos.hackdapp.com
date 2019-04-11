# 分级权限授权

本小节将带你了解EOS帐户及合约帐户的整个帐户及权限体系。比如：调整EOS帐户权重、设定合约权限组以及针对不同角色进行不同的权限授权。

----

相信很多朋友对于区块链中的帐户应该不算太陌生，比如大家熟知的ETH帐户是以一串0x开头的字段串，而在EOS中则是以一个12位长度的便于记忆的名称代替。

除此之外，EOS帐户还支持分级权限，默认自带两个权限，即owner、active。当然用户也可以自定义权限。owner作为帐户顶层权限，一般不做常规使用；而是使用active来进行操作，比如转帐、执行合约方法等。

简单来说，owner可以执行所有操作；而active权限则可以执行除了owner更改帐户密钥以外的全部操作。另外，权限是存在父子关系的。

示例：
## 单签名模式
刚创建一个名为hackdappexch的EOS帐号，则其权限为
![](http://image.chaindesk.cn/2019-03-07-121147.jpg)
在此帐户中，每个权限级别为1，而每个公钥所对应的权限为1，所以任意一个公钥匙都可对交易进行签名。

## 多签名模式
比如：我们创建了一个帐号hackdappexch，为了保证资金的安全，我们设定只能hackdapadm1和hackdappadm2同时签名一笔交易时才可进行资金转帐。

|权限组		|帐户			|权重			|权限级别		|
|:---			|:---				|:---				|:---				|
|owner		| EOS7ZitLErPkRS73N5Gh38afrpH6QUuATioqE1HE7Hw8jqAwzonqg				|1				|1				|
|active		|				|				|2				|
|			| hackdapadm1				|1				|				|
|			| hackdapadm2				|1				|				|

由于active权限组的权限级别为2，所以hackdappadm1和hackdappadm2任意一个帐号独自签名都无法操作hackdappexch帐户中的资金，从而保证了资金的安全。

以上介绍了EOS帐户缺省权限的配置与授权，那么EOS合约如何进行权限分组与授权呢？

在智能合约中，往往提供了大量的对外操作方法，其中有牵扯资金操作、有业务数据操作以及数据维护等操作。

同样为了保证整个系统的安全运行，那自然需要分离权限，以便在私钥丢失的情况下，不至于造成大的损失并能够快速修复。所以，我们需要对合约中的方法进行分角色归类建立权限组，并授权给对应的帐号。

### 建立权限组

```
cleos set account permission <合约名称> <权限组> '{"threshold" : 1, "keys": [], "accounts":[] }' active -p <合约名称>@active
```

示例：为合约(hackdappexch)创建一个名为auth.trade的权限组.

```
cleos set account permission hackdappexch auth.trade '{"threshold" : 1, "keys": [], "accounts":[] }' active -p hackdappexch@active
```
在执行命令时，需要注意钱包是否处于解锁状态。

### 权限方法映射

```
cleos set action permission <合约名称> <合约名称> <合约方法> <权限组>  -p <合约名称>@active
```

示例： 将合约(hackdappexch)中的方法(executetrade)映射到auth.trade权限组中。

```
cleos set action permission hackdappexch hackdappexch executetrade auth.trade  -p hackdappexch@active
```

### 授权
将合约中的某个权限组授权给某个帐号或地址。
1）授权给帐号

```
cleos set account permission <合约名称> <权限组> '{"threshold" : 1, "keys": [], "accounts":[{"permission":{"actor":"<EOS帐号>","permission":"active"},"weight":1}] }' active -p <合约名称>@active
```
2）授权给地址

```
cleos set account permission <合约名称> <权限组> '{"threshold" : 1, "keys": [{"key":"<EOS地址>","weight":1}], "accounts":[] }' active -p <合约名称>@active
```

示例：将交易所合约中的搓合权限组授权给EOS地址`EOS7H8xqsUyAwCPDYfQ5RQYSFKzxeX5cuucMLAC6g31GuQEG9hdKz`。

```
cleos set account permission hackdappexch auth.trade  '{"threshold" : 1, "keys": [{"key":"EOS7H8xqsUyAwCPDYfQ5RQYSFKzxeX5cuucMLAC6g31GuQEG9hdKz","weight":1}]}' active -p hackdappexch@active
```

注：任意自定义权限组如果不绑定到对应的方法上，都起不到权限限制的作用。
