# 合约发布与升级

在实际的开发过程中，我们往往需要验证合约方法的业务逻辑？而这个过程中需要我们通过多次发布合约来进行验证，对于这个重复性的工作不可能一直使用cleos命令进行重复性操作，因为太影响开发效率，那是否有一种更快的发布方式呢？

那么本文将带你了解**如何通过EOS.js进行智能合约的发布**。

![](http://cdn.hackdapp.com/2019-03-12-083806.jpg)

如上图所示，在进行合约发布时，需要用到eos系统合约中的两个方法：setcode、setabi。而这两个方法分别会用到智能合约编译后的两个文件： \*.wasm、\*.abi。

**注意** ： 本文示例中使用的eosjs的版本为`16.0.9 `。

**首先**，需要初始化EOS-SDK实例。 事先准备好初始化SDK所必需的参数：
- chainid  
	所要发布的目标EOS链chainid. 例如：正式chainid: aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906; jungle测试网chainid: e70aaab8997e1dfce58fbfac80cbbb8fecec7b99cf982a9444273cbc64c41473等等
- httpEndpoint  
	EOS链环境http接口地址。 例如: http://localhost:8888
- keyProvider  
	合约帐户私钥，主要用于交易签名。


```bash
//config.js
const Eos = require('eosjs')

const eos = Eos({
    chainId: "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
    httpEndpoint: "http://localhost:8888",
    keyProvider: "5K7mtrinTFrVTduSxizUc5hjXJEtTjVTsqSHeBHes1Viep86FP5",
    broadcast: true,
    sign: true
})

module.exports = {
    eos,
}

//https://gist.github.com/hackdapp/2522411b98b1acdadc0d842f712ca6e0

```
下一步，需要代码实现对合约文件夹中的.wasm及.abi文件的读取；

```bash
function getDeployableFilesFromDir(dir) {
    const dirCont = fs.readdirSync(dir)
    const wasmFileName = dirCont.find(filePath => filePath.match(/.*\.(wasm)$/gi))
    const abiFileName = dirCont.find(filePath => filePath.match(/.*\.(abi)$/gi))
    if (!wasmFileName) throw new Error(`Cannot find a ".wasm file" in ${dir}`)
    if (!abiFileName) throw new Error(`Cannot find an ".abi file" in ${dir}`)
    return {
        wasmPath: path.join(dir, wasmFileName),
        abiPath: path.join(dir, abiFileName),
    }
}

//https://gist.github.com/69b29103e5cc114f4478390076d8ad39
```

然后，通过调用eos实例，分别执行系统合约的setcode/setabi方法，从而达到智能合约的发布；

```bash
function deployContract({ account, contractDir }) {
  const { wasmPath, abiPath } = getDeployableFilesFromDir(contractDir)

  const wasm = fs.readFileSync(wasmPath)
  const abi = fs.readFileSync(abiPath)

  const codePromise = eos.setcode(account, 0, 0, wasm)
  const abiPromise = eos.setabi(account, JSON.parse(abi))
  return Promise.all([codePromise, abiPromise])
}

//https://gist.github.com/69b29103e5cc114f4478390076d8ad39
```

最后，调用deployContract方法，测试合约发布功能。

```bash
deployContract({ account: "eosio.token", contractDir: "./contract" }).then((result) => {
    console.log(`Deployment successful`, JSON.stringify(result, null , 4))
})
.catch(err => {
    console.error(`Deployment failed`, err)
})

//https://gist.github.com/69b29103e5cc114f4478390076d8ad39
```


通过脚本发布的方式，可以帮助我们实现一键式发布合约，而不需要在命令容器与开发容器间进行频繁切换；甚至你还可以将编译与发布脚本进行整合实现，来进一步提高开发的效率。

----

**小结**

通过本文我们学习了如何通过eos实例的setcode/setabi方法将合约编译文件快速发布到指定链环境。

另外，如果为了提高发布合约效率，我们还可以在package.json中定义发布合约的运行脚本，并配合IDE工具中的快捷键，便可达到开发效率的进一步提升。
