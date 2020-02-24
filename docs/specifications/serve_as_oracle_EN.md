# Oracle Service Vendors

The opportunity of becoming an Oracle service vendor is open for any intrested party. Oracles can provide services to fetch public data, or can be limited to a certain Specialized domain that one has understanding of. A fixed amount of processing fee can be charged for each request to the Oracle. Details regarding becoming an Oracle service vendor have been provided below.

## Deploying an Oracle Smart Contract

The first step ivolved in the process of becoming an Oracle service provider is deploying a custom Oracle contract. Please refer to [this](https://github.com/ontio/OEPs/blob/master/OEPS/OEP-34.mediawiki) link for more information on the Oracle protocol.

A sample Oracle contract is available for reference, and can be found by following [this](https://github.com/ontio/ontology-oracle/blob/master/smartcontract/oracle.py) link.

## Downloading the Oracle Client

The following Git command can be used to directly clone Ontology's Oracle repository:

```shell
$ git clone https://github.com/ontio/ontology-oracle.git
```

## Providing Specialized Data Services

The Oracle repository contains services that by default can fetch public data, and the data can be obtained without any extra permissions. The specialized data services from **random.org** can be fetched after applying for the **key**. The sample provided in the repository uses the Basic API version to fetch the development API key.

The data service provider can choose to add any kind of specialized data for the clients to access, or stick with the public data services, as shown in the sample.

The relevant data service files that can be found under the [runners](https://github.com/ontio/ontology-oracle/tree/master/runners) folder are as follows:

1. The `runner.go` file defines the `runner` API and other runner routes.
2. The `http.go` file provides some basic data retrieval services. The data is fetched using the `HTTP` methods.
3. The `json_parse.go` file provides data analysis service.
4. The `random_org.go` file provides specialized data retrieval services to fetch random numbers, including both ordinary random numbers and random numbers used for signature.

Data service vendors can refer to `random_org.go` to get an understanding of how to implement specialized data services.

## Compiling the Oracle Client

```shell
go build main.go
```

## Editing Configuration Files

The contents of the configuration file `config.json` are as follows:

```json
{
  "WalletFile": "./wallet.dat",
  "LogLevel": 0,
  "ONTRPCAddress": "http://127.0.0.1:20336",
  "ScannerInterval": 10,
  "GasPrice": 0,
  "GasLimit": 20000,
  "ContractAddress": "b869eed573863f8efdb3ae39d4963a781e81d4b3",
  "RandomOrgKey": "c7511065-c88d-4f28-af4f-293c91ad20d9"
}
```

|      Field      | Description                                                                                                               |
| :-------------: | ------------------------------------------------------------------------------------------------------------------------- |
|   WalletFile    | Wallet path, this wallet is used to pay the gas fees for data upload requests                                             |
|    LogLevel     | The selected log level                                                                                                    |
|  ONTRPCAddress  | The RPC address and port of the Ontology nodem, the transaction result of data upload request is transmitted to this node |
| ScannerInterval | Time interval for scanning, the Oracle client scans request data from the Oracle contract in fixed time intervals         |
| ContractAddress | Contract address hash of the Oracle contract                                                                              |
|  RandomOrgKey   | The Basic API verion key fetched from **random.org**, permissions for fetching other specialized data can also be added   |

The different log levels supported are:

```go
const (
	DebugLog = iota
	InfoLog
	WarnLog
	ErrorLog
	FatalLog
	MaxLevelLog
)
```

## Starting the Oracle Client

The command used to start the Oracle client node is:

```shell
./main node
```
Please enter the password when prompted by the shell to start the node.

## Making the Oracle Contract Address and Data Services Public

After deploying the Oracle contract and starting the Oracle client, the data service vendor can publicly share the Oracle contract address and fee structure along with the content being provided and the request format.

