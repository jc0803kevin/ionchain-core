##  ionchain-core


本项目是ionchain协议golang版本的实现

在这篇博客中[IPOS共识算法设计](http://gcc2ge.github.io/2019/04/02/IPOS共识算法设计/)，详细说明了ionchain的共识算法的设计详情。


## 源码编译

在编译之前，你现需要安装golang（版本大于等于1.10）和`C`编译器

`clone`项目到你指定的目录中：

```
git clone https://github.com/ionchain/ionchain-core
```

使用以下命令编译`ionc`

```
cd ionchain-core

make ionc
```

或者可以通过以下命令编译其他平台的`ionc`版本（`Linux`，`windows`）

```
make all
```

### 在ionchain主网上运行全节点

用户在`ionchain`是最多的使用场景就是：创建账号，转移资产，部署、调用合约。为了满足这个特定的场景，可以使用快速同步方法来启动网络，执行以下命令:

```
$ ionc console
```

上面这个命令将产生以下两个操作:

 * 在快速同步模式下，启动`ionc`节点，在快速同步模式下，节点将会下载所有的状态数据，而不是执行所有`ionchain`网络上的所有交易.
 * 开启一个内置的`javascript console`控制台，在控制台中用户可以与`ionchain`网络进行交互。


#### 使用Docker快速启动节点

启动`ionchain`网络最快速的方式就是在本地启动一个`Docker`：

```
docker run -d --name ionchain-node -v /Users/alice/ionchain:/root \
           -p 8545:8545 -p 30303:30303 \
           ionchain/go-ionchain
```

`docker`会在`/Users/alice/ionchain`本地目录中映射一个持久的`volume`用来存储区块，同时也会映射默认端口。如果你想从其他容器或主机通过`RPC`方式访问运行的节点，需要加上`--rpcaddr 0.0.0.0`参数。默认情况下，`ionc`绑定的本地接口与`RPC`端点是不能从外部访问的。

### 以编程的方式与`IONC`节点交互

作为一个开发人员想通过自己的程序与`ionchain`网络进行交互，而不是通过`JavaScript console`的方式，为了满足这种需求，`ionc`有一个内置`JSON-RPC API`，这些API通过`HTTP`、`WebSockets`和`IPC`方式暴露出去。其中`IPC`端口是默认开启的，`HTTP`和`WS`端口需要手动开启，同时为了安全方面的考虑，这两个端口只会暴露部分API。

基于HTTP的JSON-RPC API 选项：

  * `--rpc` 开启 HTTP-RPC 服务
  * `--rpcaddr` HTTP-RPC 服务监听地址 (默认: "localhost")
  * `--rpcport` HTTP-RPC 服务监听端口 (默认: 8545)
  * `--rpcapi` 通过HTTP暴露出可用的API
  * `--rpccorsdomain` 逗号分隔的一系列域，通过这些域接收跨域请求

基于WebSocket的 JSON-RPC API选项:


  * `--ws` 开启 WS-RPC 服务
  * `--wsaddr` WS-RPC 服务监听地址(默认: "localhost")
  * `--wsport` WS-RPC 服务监听端口 (默认: 8546)
  * `--wsapi` 通过WS-PRC暴露出可用的API

基于IPC的JSON-RPC AP选项


  * `--ipcdisable` 禁用 IPC-RPC 服务
  * `--ipcapi` 通过IPC-PRC暴露出可用的API

**注意：在使用http/ws接口之前，你需要了解相关的安全知识，在公网上，黑客会利用节点在公网上暴露的接口进行破坏式的攻击**

### 创建一个私有链

创建一个自己的私有链会有一点复杂，因为你需要手动修改很多官方创建文件的配置。


#### 定义私有链创世块

首先，为你的私有网络创建一个创始状态，这个创始状态需要你的私有网络中的所有节点都知晓，并达成共识。`genesis.json`以JSON格式组成：

```json
{
		  "config": {
			"chainId":
		  },
		  "alloc": {},
			"0x0000000000000000000000000000000000000100": {
			  "code": "编译后的保证金合约二进制代码",
			  "storage": {
				"0x0000000000000000000000000000000000000000000000000000000000000000": "0x0a",
				"0x33d4e30ad2c3b9f507062560fe978acc29929f1ee5c2c33abe6d050171fd8c93": "0x0de0b6b3a7640000",
				"0xe0811e07d38b83ef44191e63c263ef79eeed21f1260fd00fef00a37495c1accc": "0xd9a7c07f349d4ac7640000"
			  },
			  "balance": ""
			}
		  },
		  "coinbase": "0x0000000000000000000000000000000000000000",
		  "difficulty": "0x01",
		  "extraData": "0x777573686f756865",
		  "gasLimit": "0x47e7c4",
		  "nonce": "0x0000000000000001",
		  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
		  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
		  "timestamp": "0x00",
		  "baseTarget": "0x1bc4fd6588",
		  "blockSignature": "0x00",
		  "generationSignature": "0x00"
		}
```

以上关于保证金合约是如何创建、编译的将在另外一个项目中做详细说明，我们建议你修改`nonce`值为一个随机数，这样可以防止未知的远程节点连接到你的网络中。如果你需要给某些账户预设一些资金，可以使用修改`alloc`值：
```json
"alloc": {
  "0x0000000000000000000000000000000000000001": {"balance": "111111111"},
  "0x0000000000000000000000000000000000000002": {"balance": "222222222"}
}
```
当`genesis.json`文件创建完成时，你需要在所有的`ionc`节点执行初始化操作。

```
$ ionc init path/to/genesis.json
```

#### 创建bootnode节点

当所有的节点都完成初始化时，你需要启动一个`bootstrap`节点，通过`bootstrap`节点可以帮助其他的节点之间进行相互发现，这样他们就可以通过网络连接在一起。

```
$ bootnode --genkey=boot.key
$ bootnode --nodekey=boot.key
```

#### 启动节点

`bootnode`启动后，通过`telnet <ip> <port>`命令测试一下是否可以从外部访问`bootnode`，现在启动所有的`ionc`节点，在启动时加上`--bootnodes`选项。同时为了保存你的私有链上的数据，你需要创建一个`datadir`目录，并通过`--datadir`选项设置。

```
$ ionc --datadir=path/to/custom/data/folder --bootnodes=<bootnode-enode-url-from-above>
```

*注意：从现在开始你的节点已经完全从主链网络上断开，现在你可以配置miner来处理交易，并创建新的区块*

#### 运行私有链miner

在`ionc`网络中miner的算力是通过在保证金合约的保证金数量决定的，启动一个实例用于挖矿：

```
$ ionc <usual-flags> --mine --etherbase=0x0000000000000000000000000000000000000000
```

其中`—etherbase`设置为miner的账号地址，miner可以通过`--targetgaslimit`调整区块的`gas limit`，`--gasprice`设置接受交易的`gas price`
