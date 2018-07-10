# 搭建以太坊私链

| 版本/状态 | 作者   | 参与者  | 日期      | 说明   |
| :---: | :--- | :--- | :------ | :--- |
|  1.0  | 蒋剑   |      | 2018年7月 | 初稿   |

### 一、目的
本文记录搭建以太坊私链、以太币交易、智能合约部署的学习历程。主要参考资料：

https://www.cnblogs.com/sumingk/articles/9030469.html

### 二、私链搭建

#### 2.1、创建创世区块

创世区块内容：

```json
{
  "config": {
        "chainId": 10,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x02000000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}

```

字段意义：

```json
chainId : 以太坊区块链网络Id，ethereum主链是1，私有链只要不和主链冲突即可。
alloc : 预留账户，如下
  "alloc":{
   "0x0000000000000000000000000000000000000001":{"balance":"121312321"},
   "0x0000000000000000000000000000000000000002":{"balance":"121312321"},
  }
Coinbase: 旷工账户
Difficulty: 挖矿难度，0x400，这个是简单。
extraData：相当于备注
gasLimit：最小消耗gas
nonce : 64位随机数，用于挖矿，注意他和mixhash的设置需要满足以太坊黄皮书中的要求
parentHash : 上一个区块的Hash值，因为是创世块，石头里蹦出来的，没有在它前面的，所以是0
Timestamp : 时间戳
```

保存区块内容为一个json文件（genesis.json）。然后初始化区块节点：

```json
geth --datadir  /path/to/datadir  init  /path/to/genesis.json

本机设置：
cd ~/go1.9/src/github.com/ethereum/go-ethereum/build/bin
geth --datadir ~/blockchain/ethereum/data/ init genesis.json
```

#### 2.2 启动geth节点A

启动geth节点，同时开启一个命令行：

```json
geth --identity "TestNode" --datadir "~/blockchain/ethereum/data" --rpc --rpcapi "db,eth,net,web3" --rpcaddr "127.0.0.1" --ipcpath "~/blockchain/ethereum/data1/geth/geth.ipc" --rpcport "8480" --port "30300" --networkid "29382" console
```

字段意义：

```json
--identity : 节点身份标识，起个名字
--datadir : 指定节点存在位置，“data0”
--rpc : 启用http-rpc服务器
--rpcapi : 基于http-rpc提供的api接口。eth,net,web3,db...
--rpcaddr : http-rpc服务器接口地址：默认“127.0.0.1”
--rpcport : http-rpc 端口(多节点时，不要重复)
--port : 节点端口号（多节点时，不要重复）
--networkid : 网络标识符 随便指定一个id（确保多节点是统一网络，保持一致）
```

#### 2.3 挖矿准备

geth常用命令：

```json
#创建账户
personal.newAccount("123456")

#获取账户数组
eth.accounts  

#解锁账户，转账时可使用
personal.unlockAccount(eth.accounts[0], "123456")

#节点主账户
eth.coinbase

#查看账户余额
eth.getBalance(eth.accounts[0])

#开始挖矿
miner.start()，  
#结束挖矿
miner.stop() 

```

#### 2.4 新增节点B

以上步骤可以搭建一个节点，为模拟多节点的情况，再搭建另外一个节点。步骤同上，节点名字、监听端口需要改下,networkid保持相同：

```json
./geth --identity "TestNode1" --datadir "~/blockchain/ethereum/data1" --rpc --rpcapi "db,eth,net,web3" --rpcaddr "127.0.0.1" --ipcpath "~/blockchain/ethereum/data1/geth/geth.ipc" --rpcport "8481" --port "30301" --networkid "29382" console
```

#### 2.5 节点组网

先查看节点B的enode信息：

```json
> admin.nodeInfo.enode
"enode://704cc9fb127c58403b940ff1c96f83aa79ef5a3115fe0fe37a8999eb51e04fc52490893260a5cfca562997076728dcd7d1f1a6b983dffb1a4a93e646d8935970@[::]:30306"
```

再将节点B加入节点A的网络中：

```json
> admin.addpeer("enode://d4f64272de882d2e2ccefc6466c6580ddecd253f5c9d87f977ac3881cbea7b141c07681ea605c53af5815cbfc25b5138b9ddb07be61b757850a55b7197939ba4@127.0.0.1:30306")
```

[::]替换成本机ip地址。

#### 2.6 开始挖矿

两个节点同时开始挖矿（前提是创建了主账号）：


```json
> miner.start()
```

### 三、转账

#### 3.1 启动mist钱包

直接点开程序是不行的，会默认链接到公网。找到可执行文件，这样启动：

```json
cd /Applications/Ethereum Wallet.app/Contents/MacOS
./Ethereum\ Wallet --rpc ~/blockchain/ethereum/data/geth/geth.ipc
```

ipc文件需要找对。如果用 --rpc http://127.0.0.1:8587 这种方式启动，不能进行交易。

#### 3.2 交易

在mist的send页面进行交易，输入目的钱包地址、密码。完成交易。

### 四、智能合约

TODO
