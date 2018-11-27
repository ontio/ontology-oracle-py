# Ontology Oracle
## ontology oracle
区块链预言机（Blockchain Oracles）"概念的提出主要是为了解决区块链协议自身存在的局限性：区块链这种去中心化的网络（包括在其上构建的应用/ 智能合约）不能与外部内容进行交互。但很多时候智能合约又必须依赖外部触发条件，比如合约：

当明天某地的天气在14:00至15:00的平均温度如果高于38摄氏度，则向通讯录中的所有人发放一个50代币的红包。

又或者...

某次航班将于10:00am到达目的地，如果航班延误将触发智能合约，投保人将获得100代币的补偿。

随着区块链应用变得越来越复杂，迫切需要解决“围墙花园”的限制。如航班信息，股票价格，贵金属价格，等等大量的数据需要上链服务。对于这样的数据，智能合约是无法从外部网站获取的。所以就需要预言机来搬运，将外部数据写入到区块链中，使合约得以完成。

Ontology oracle正是这样的一个数据搬运工的角色，它使得在智能合约中获取链外数据成为可能，未来还将会针去中心化oracle的数据正确性问题进行一定的博弈。

## ontology oracle基础架构

![framework](/resources/framework.png)

### 链下部分（oracle node & data source）
在链外，ontology oracle由oracle node组成，node和ontology网络相连，并独立的处理数据请求，未来可以支持更多的区块链网络。
节点的工作由一系列的子任务组成，每个子任务定义了相应的操作，如从数据源获取数据，解析数据，上链等。每个子任务的结果都是下一个子任务的输入。不同节点通过这一系列任务的方式将获取得到的外部数据发送到部署在ontology网络上的oracle contract，由该合约来进行结果聚集。

### 链上部分（oracle contract)
oracle contract主要对node发送的数据进行聚集和存储，供其他合约调用。

### Oracle运作流程
![workflow](/resources/workflow.png)

# 如何成为Ontology oracle

## 部署一本Oracle contract
Oracle contract模板见smartcontract/oracle.cs
### Oracle contract必须实现的标准方法
CreateOracleRequest(string request, byte[] address)

该方法创建oracle请求，参数为请求本身和调用者的地址，该交易需要调用者地址签名该方法可能需要消耗额外的ong，由oracle服务提供方收取。

SetOracleOutcome(byte[] txHash, byte[] result)

该方法只能由Oracle node调用，用于写入用户请求的数据。

GetOracleOutcome(byte[] txHash)

该方法用于用户获取请求的数据。

## 部署并运行Oracle node
### Oracle node部署说明
### 基本配置
```text
{
  "WalletFile": "./wallet.dat",
  "LogLevel": 0,
  "ONTRPCAddress": "http://127.0.0.1:20336",
  "ScannerInterval": 10,
  "GasPrice": 0,
  "GasLimit": 20000,
  "ContractAddress": "f4d61a76b766638061ef73f896637545c6198b99",
}
```
WalletFile配置签名钱包路径，LogLevel配置日志级别，ONTRPCAddress配置监听的ontology网络rpc的地址和端口，MaxLogSize配置单个日志文件的大小，不指定默认20M，ScannerInterval配置node扫描ontology网络中oracle请求的时间间隔，ContractAddress配置对应的oracle合约地址。

### build oracle node
```text
go build main.go
```

### 启动oracle node
```text
go run main.go node
```

# Oracle使用协议标准
## 创建oracle request
用户可以在自己的合约中调用部署在ontology网络上的oracle contract来获取外部数据，目前支持httpGet， httpPost， random.Org获取随机数：
### httpGet
```text
operation = "CreateOracleRequest"
request = `{
		"scheduler":{
			"type": "runAfter",
			"params": "2018-06-15 08:37:18"
		},
		"tasks":[
			{
			  "type": "httpGet",
			  "params": {
				"url": "https://bitstamp.net/api/ticker/"
			  }
			},
			{
				"type": "jsonParse",
				"params":
				{
					"data":
					[
						{
							"type": "String",
							"path": ["timestamp"]
						},
						{
							"type": "String",
							"path": ["last"]
						},
						{
							"type": "Float",
							"decimal": 100,
							"path": ["open"]
						}
					 ]
				}
			}
		]
	}`
args = [request, address]
```
http response:
```text
{
	"high": "5610.00000000",
	"last": "5518.70",
	"timestamp": "1542359479",
	"bid": "5518.12",
	"vwap": "5436.78",
	"volume": "16423.18407040",
	"low": "5199.80000000",
	"ask": "5518.69",
	"open": 5571.12
}
```
params:

url: http get请求的地址url

### JsonParse
该类型的task会以path参数为key逐层遍历，返回得到的value结果。

params:

data: 所要获取的数据结构。

type: 数据类型，支持int，float（乘以精度按照int处理），string ，array，map。

sub_type: array, map的子类型。

decimal: 浮点数所要乘以的精度。

path: 一个数组，每个元素作为下次获取数据的索引, 如果数据为json, 则是该json数据的key, 如果数据为array, 则是该数据的index。

### scheduler定义
```text
{
    "type": "",
    "params": "",
}
```
type目前支持RunAfter，在某时刻后执行。 此项不填为立即执行。
#### RunAfter
该类型的job会在指定的时间之后执行，如赛事结果oracle可以将其设置为赛事结束之后。

params: 代表时间的字符串，格式如"2018-06-15 08:37:18"

### httpPost
```text
operation = "CreateOracleRequest"
request = `{
		"scheduler":{
			"type": "runAfter",
			"params": "2018-06-15 08:37:18"
		},
		"tasks":[
			{
			  "type": "httpPost",
			  "params": {
				"url": "https://api.random.org/json-rpc/1/invoke",
				"contentType": "application/json-rpc",
				"body": "{\"jsonrpc\": \"2.0\",\"method\": \"generateSignedIntegers\",\"params\": {\"apiKey\": \"c7511065-c88d-4f28-af4f-293c91ad20d9\",\"n\": 6,\"min\": 1,\"max\": 10,\"replacement\": false,\"base\": 10},\"id\": 1}"
			  }
			},
			{
				"type": "jsonParse",
				"params":
				{
					"data":
					[
						{
							"type": "Array",
							"sub_type": "Int",
							"path": ["result", "random", "data"]
						}
					 ]
				}
			}
		]
	}`
args = [request, address]
```
http response:
```text
{
    "jsonrpc": "2.0",
    "result": {
        "random": {
            "method": "generateSignedIntegers",
            "hashedApiKey": "oT3AdLMVZKajz0pgW/8Z+t5sGZkqQSOnAi1aB8Li0tXgWf8LolrgdQ1wn9sKx1ehxhUZmhwUIpAtM8QeRbn51Q==",
            "n": 6,
            "min": 1,
            "max": 6,
            "replacement": true,
            "base": 10,
            "data": [
                2,
                4,
                4,
                1,
                5,
                3
            ],
            "completionTime": "2013-09-30 14:58:03Z",
            "serialNumber": 69260
        },
        "signature": "BxHxajeRg7Q+XGjBdFS1c7wkZbJgJlverfZ5TVDyzCKqo2K5A4pD+54EMqmysRYwkL3w2NS2DFLVrsyO1o96bW9BGp5zjjrEegz9mB+04iOTaRwmdQnLJAj/m3WRptA+qzodPCTaqud8YWBifqWCM34q98XwjX+nlahyHVHT9vf5KO0YVkD/yRI1WN5M/qX21chVvSxhWdmIrdCkrovGnysFq8SzCRNhpYx+/1P+YT2IKsH8jth9z82IAz1ANVh918H/UdpuD1dR7TD6nk3ntRgGrIiu2qqVzFi8A7/6viVgRqtffE4KVZY6O9mUJ+sGkF5Ohayms7LHSFy1VC8wMbMgwod+A8nr5yzjAC4SCUkT1bKAyWNF3SdVcLtvWdcf97Ew6RjohzCW4Vs3jUlh6jF/pj3b3++U3lBHCh43IIonw8MQ7afwpqP12yvyDym1isNjhMKYjmzWRerSvnsMyQIH8xFW7IHt2g/0qnzJgABFmUNBRKJPCD9CMgjh60sSwW7EyrGMy7/qisfE0IU74P/F7KCty/g1jIlXX5/O1lQjwY34wnoP0NXL08QteukRZZUfJQnscx1NGE+HX1c9bMBI8LC0ZFYFk+uY6ib/0rCV5OcLLE9PihCdC8WoI1x3bobr8tbtfgnXMTjogxwVXiiSN1TMnTIWlJ+KM5eSWrw=",
        "bitsUsed": 16,
        "bitsLeft": 932400,
        "requestsLeft": 199991,
        "advisoryDelay": 1000
    },
    "id": 1
}
```

### randomOrg
上述httpPost的例子其实是在random.org获取签名随机数，目前的oracle模板专门为random.org的随机数封装了一个更简便的调用方法randomOrg。
#### 签名随机数
```text
operation = "CreateOracleRequest"
request = `{
		"scheduler":{
			"type": "runAfter",
			"params": "2018-06-15 08:37:18"
		},
		"tasks":[
			{
			  "type": "randomOrg",
			  "params": {
				"method": "GenerateSignedIntegers",
				"n": 10,
				"min": 1,
				"max": 10,
				"replacement": false
			  }
			},
			{
				"type": "jsonParse",
				"params":
				{
					"data":
					[
						{
							"type": "Array",
							"sub_type": "Int",
							"path": ["data"]
						},
						{
							"type": "String",
							"path": ["signature"]
						}
					 ]
				}
			}
		]
	}`
args = [request, address]
```
其中n为获取的随机数的个数，min为随机数最小值，max为随机数最大值，replacement为是否允许重复。

response:
```go
type SignedIntegerData struct {
	Raw          json.RawMessage `json:"raw"`
	HashedApiKey string          `json:"hashedApiKey"`
	SerialNumber int             `json:"serialNumber"`
	Data         []int           `json:"data"`
	Signature    string          `json:"signature"`
}
```
#### 非签名随机数
```text
operation = "CreateOracleRequest"
request = `{
		"scheduler":{
			"type": "runAfter",
			"params": "2018-06-15 08:37:18"
		},
		"tasks":[
			{
			  "type": "randomOrg",
			  "params": {
				"method": "GenerateIntegers",
				"n": 10,
				"min": 1,
				"max": 10,
				"replacement": false
			  }
			},
			{
				"type": "jsonParse",
				"params":
				{
					"data":
					[
						{
							"type": "Array",
							"sub_type": "Int",
							"path": [data"]
						},
						{
							"type": "String",
							"path": [completionTime"]
						}
					 ]
				}
			}
		]
	}`
args = [request, address]
```
其中n为获取的随机数的个数，min为随机数最小值，max为随机数最大值，replacement为是否允许重复。

response:
```go
type IntegerData struct {
	Data         []int           `json:"data"`
	Signature    string          `json:"signature"`
}
```


### 获取oracle request的结果
```text
operation = "GetOracleOutcome"
args = txhash
```
txhash为oracle request在链上的交易hash。


