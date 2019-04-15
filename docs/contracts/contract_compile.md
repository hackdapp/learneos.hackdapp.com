# EOS合约编译

在前面几个章节曾经介绍过ABI文件结构及CDT开发套件，那么本章节将介绍如何使用CDT套件中的eosio-cpp命令编译并生成具体的abi及wasm文件。

** eosio-cpp**

作为一个最常用的命令，主要用于生成智能合约接口描述ABI文件以及发布合约所必须的二进制编译WASM文件

首先，需要在本地开发环境中安装具体的cdt开发套件


```bash

brew tap eosio/eosio.cdt //增加仓库
brew install eosio.cdt	 //安装工具包
```

在安装完成后，可以尝试在命令容器中执行`eosio-cpp -help `，查看具体的参数使用说明

```
~> eosio-cpp -help
OVERVIEW: eosio-cpp (Eosio C++ -> WebAssembly compiler)
USAGE: eosio-cpp [options] <input file> ...

OPTIONS:

Generic Options:

  -help                    - Display available options (-help-hidden for more)
  -help-list               - Display list of available options (-help-list-hidden for more)
  -version                 - Display the version of this program

compiler options:

  -C                       - Include comments in preprocessed output
  -CC                      - Include comments from within macros in preprocessed output
  -D=<string>              - Define <macro> to <value> (or 1 if <value> omitted)
  -E                       - Only run the preprocessor
  -I=<string>              - Add directory to include search path
  -L=<string>              - Add directory to library search path

```

然后，使用eosio-cpp命令进行合约编译，生成对应abi合约描述文件及wasm合约文件

```
eosio-cpp -abigen "xx.cpp" -o "xx.wasm" --contract "xx"
```

例如:  根据hello.cpp合约文件及合约名hackdappcom1，生成对应的hello.wasm合约二进制文件

```
eosio-cpp -abigen 'contracts/hello.cpp' -o 'contracts/hello.wasm' --contract 'hackdappcom1'
```

编译完成后，会在工程目录生成hello.abi、hello.wasm两个编译文件。hello.abi就好比webservice中的wsdl描述语言一样，主要用于对合约接口及数据结构进行结构性描述， wasm文件为合约编译后的二进制文件。
