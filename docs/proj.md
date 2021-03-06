# 项目介绍

本项目是使用EOS、SmartContract、Node.js、React等技术架构， 采用链上搓合与资产清算的方案实现的去中心化交易所。

在交易所中， 用户可以直接使用自己的钱包进行帐号登录；然后使用自己的用户权限直接创建买/卖订单，而无需进行币种充值；当系统发现订单薄中存在符合搓合价格要求的订单后，则由系统直接进行搓合，并将搓合日志记录至区块链上；最后，由系统将搓合成功的部分或完全成交的TOKEN转帐到对方帐号地址。

另外，当系统调度发现链上存在成交日志时，会自动将成交记录同步至后端服务数据库，并同步更新K线图报表数据以及实时更新币价信息；最后，根据变动的信息数据， 通过socket服务将消息推送到前端展示页。

本系统核心业务逻辑主要是通过智能合约进行实现的， 其中包括搓合逻辑的处理、关键数据的定义、买卖单的创建以及订单薄的维护；而后端服务主要是以node.js技术进行功能实现，一方面用于与区块链的接口交互，比如：查询合约内数据以及链上区块数据；另一方面主要用于对外提供http及socket接口服务，通过整合业务数据及合约数据，以供前端页面的数据展示；除此之外，后端还有配套的调度服务，实时同步链上数据，并生成不同维度的报表数据。

## 功能点介绍
从技术角度讲， 整个项目的技术架构图如下

![](http://assets.processon.com/chart_image/5c6ccd91e4b07fada4ec1a3a.png)

从功能结构上，主要划分为六大模块：

- 基础数据管理；
  主要用于维护币种及交易对数据。 币种管理主要用于定义当前交易所所支持的币种， 比如币种名称， 合约名称及资金精度等；交易对管理主要用于定义基准代币可兑换的币种、交易对最小订单量以及手续费等信息；
- 订单管理
  主要用于维护用户订单数据以及交易所订单薄数据。用户订单功能主要用于记录用户实时创建的买/卖单交易数据， 其中包括交易对、购买价格、订单量、订单状态等数据信息；而订单薄功能主要用于对交易所所有的订单按卖买类型进行分队列排序，从而方便展示当前交易对的交易深度以及供搓合功能处理；
- 搓合管理
  主要用于实时将订单薄中符中搓合条件的订单进行数据处理，并同时更新用户订单数据以及资金清算等业务；
- 系统管理
  主要用于维护交易所配置数据以及运营状态。比如，是否锁定或开启交易所；
- 报表管理
  主要用于实时监控链上搓合成交记录，并实时同步订单数据至数据库，供前端K线图的数据展示；
- 调度管理
  主要用于通过交易市场的交易情况实时展示K线图报表数据以及实时更新所有交易对币价信息， 比如：24小时成交量， 当前币种价格、涨跌幅等信息。

## 流程图

![](http://assets.processon.com/chart_image/5c6e043ee4b056ae2a109843.png)

## 项目进度安排

![](http://assets.processon.com/chart_image/5c6baaf8e4b0fa03ceb804b2.png)

### 项目成果

![](http://cdn.hackdapp.com/2019-03-26-103428.jpg)
