# EOS智能合约开发之CDT套件

[EOSIO.CDT](https://github.com/EOSIO/eosio.cdt/) (Contract Development Toolkit)， 是开发 EOS智能合约所必不可少的一个编译工具。一方面，我们在开发过程中，需要随时检查自己的合约是否存在错误；一方面，发布合约时所需要的ABI和WASM文件，也是能过CDT编译而成的。

本次讲解的CDT版为1.3.2。如果你之前使用的是1.2.x的版本，那么强烈建议阅读[**Differences between Version 1.2.x and Version 1.3.x**](https://github.com/EOSIO/eosio.cdt/tree/v1.3.2#differences-between-version-12x-and-version-13x)，因为在1.3.x版本中移除了一些数据类型，比如: account\_name、permission\_name、symbol\_name等，且而新加入了一些数据类型，来替代之前的数据类型，比如： signature -\> capi\_signature、N(foo) -\> "foo"\_n等。

如果继续使用1.3.x版本之前的数据类型编写合约的化， 是否无法在cdt\_1.3.2中编译成功的。另外需要说明的是：无论是使用哪个CDT版本编译的合约文件，生成的ABI及WASM文件都是可以发布成功的，EOS运行环境节点是向下兼容的。

接下来，我们将逐个介绍以下CDT工具：
![](http://cdn.hackdapp.com/2019-03-14-095444.jpg)

**eosio-abigen**


```
USAGE: eosio-abigen [options] <source0> [... <sourceN>]

OPTIONS:

ABI generator options:

  -context=<string>          - ABI context
  -destination-file=<string> - destination json file
  -extra-arg=<string>        - Additional argument to append to the compiler command line
  -extra-arg-before=<string> - Additional argument to prepend to the compiler command line
  -optimize-sfs              - Optimize single field struct
  -p=<string>                - Build path
  -verbose                   - show debug info
```

该命令主要用于根据合约CPP文件生成对应的ABI文件。比如：

```
eosio-abigen hello.cpp --output=hello.abi
```

## **eosio-cc**

该命令与eosio-cpp作用一样。只不过比eosin-cpp缺少两项可选参数: ` -fcoroutine-ts `、`-std=<string> `， 都可以对智能合约进行合约编译与生成ABI文件。

## **eosio-launcher **

用来协助部署一个多节点区块链网络的应用程序。可通过`eosio-launcher --help`了解其具体参数用法。

## ** eosio-cpp**

作为一个最常用的命令，主要用于生成智能合约接口描述ABI文件以及发布合约所必须的二进制编译WASM文件

```
eosio-cpp -abigen "xx.cpp" -o "xx.wasm" --contract "xx"
```

## **eosio-pp**


```
usage: stripbss [options] filename

  Read a file in the WebAssembly binary format, strip bss or any data segment that is only initialized to zeros.

  $ stripbss test.wasm -o test.stripped.wasm

  # or original replacement
  $ wasm2wat test.wasm
```

## **eosio-wasm2wast** 与 **eosio-wast2wasm**

用于将二进制文件与可阅读的文本格式进行互转。

示例如下：

```
> eosio-wasm2wast eosio.token.wasm --no-debug-names -o eosio.token.wat
> cat eosio.token.wat
  (i32.store
   (tee_local $3
    (i32.add
     (get_local $0)
     (i32.const -4)
    )
   )
   (i32.and
    (i32.load
     (get_local $3)
    )
    (i32.const 2147483647)
   )
  )
 )
 (func $__wasm_nullptr (type $FUNCSIG$v)
  (unreachable)
 )
)
```

[https://webassembly.github.io/wabt/demo/wat2wasm/](https://webassembly.github.io/wabt/demo/wat2wasm/)
[https://webassembly.github.io/wabt/demo/wasm2wat/](https://webassembly.github.io/wabt/demo/wasm2wat/)

## **eosio-blocklog**

用于查询指定区块的详细信息。比如：查询区块高度为1和2的区块信息


```
> eosio-blocklog --blocks-dir /mnt/dev/data/blocks/ --first 1 --last 2
{
  "block_num": 1,
  "id": "00000001bcf2f448225d099685f14da76803028926af04d2607eafcf609c265c",
  "ref_block_prefix": 2517196066,
  "timestamp": "2018-06-01T12:00:00.000",
  "producer": "",
  "confirmed": 1,
  "previous": "0000000000000000000000000000000000000000000000000000000000000000",
  "transaction_mroot": "0000000000000000000000000000000000000000000000000000000000000000",
  "action_mroot": "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
  "schedule_version": 0,
  "new_producers": null,
  "header_extensions": [],
  "producer_signature": "SIG_K1_111111111111111111111111111111111111111111111111111111111111111116uk5ne",
  "transactions": [],
  "block_extensions": []
}
{
  "block_num": 2,
  "id": "000000029ec2518eae47c5b8a5421d6d9114ef839fb09fab9cd4ed7aaedc627f",
  "ref_block_prefix": 3099936686,
  "timestamp": "2019-02-22T03:33:56.500",
  "producer": "eosio",
  "confirmed": 0,
  "previous": "00000001bcf2f448225d099685f14da76803028926af04d2607eafcf609c265c",
  "transaction_mroot": "0000000000000000000000000000000000000000000000000000000000000000",
  "action_mroot": "747d103e24c96deb1beebc13eb31f7c2188126946c8677dfd1691af9f9c03ab1",
  "schedule_version": 0,
  "new_producers": null,
  "header_extensions": [],
  "producer_signature": "SIG_K1_KjL6GwAzkMGfbEYGfH7vDghTaWfWdhSPEgchovxW41ggsMYGTHBvsqKSKmWc8tGG33SkvKxBGoAfseyUU4nPZAMmJ9k8wT",
  "transactions": [],
  "block_extensions": []
}
```

## **eosio-Id**

主要用来链接程序内对库文件的链接依赖。

## **eosio-abidiff**

访命令主要用于比较两个ABI文件的不同.

```
OVERVIEW: eosio-abidiff
USAGE: eosio-abidiff [options] <input file1> ... <input file2> ...

OPTIONS:

Generic Options:

  -help      - Display available options (-help-hidden for more)
  -help-list - Display list of available options (-help-list-hidden for more)
  -version   - Display the version of this program

e.g
  eosio-abidiff hello.abi old_hello.abi
```

----
最后，CDT开发套件的出现，好处就是减少了在开发过程需要对于EOS 仓库的依赖。而且在EOS 1.3版本之后，就移除了EOS仓库里面的eosiocpp程序以及** contracts/eosiolib**、 **contracts/libc++**、**contracts/musl** 等合约依赖库文件，取而代之的便是CDT。另外，CDT是基于[WABT: The WebAssembly Binary Toolkit](https://github.com/WebAssembly/wabt)进行扩展开发的，如果想更深入的了解与学习，也可以阅读其源码。


------------------------------------------------------------------------------------------------------------
**欢迎关注HackDApp博客或公众号**, HackHook将持续为你分享IndieMaker成长路径、DAPP技术知识、高效Mac使用技巧、底层思维认知。
------------------------------------------------------------------------------------------------------------
我的博客:     https://www.hackdapp.com/
我的github:   https://github.com/hackdapp
我的哔哩哔哩:   https://space.bilibili.com/17360859
我的微信公众号: hackdapp
![](http://cdn.hackdapp.com/2019-04-03-mysign.jpg)
IndieMakers:  https://www.indiemakers.cn
------------------------------------------------------------------------------------------------------------
联系邮箱：55269778@qq.com
