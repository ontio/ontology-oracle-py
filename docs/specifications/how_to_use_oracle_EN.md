# How to Use Oracle Services in Applications

Oracle services can be used in applications by invoking the Oracle contract in the scope of app's smart contract. When there is a requirement for external data a request can be created and sent to the Oracle contract to fetch the results. The process involved is described below: 

### Registring Cross Contract Invocation to an Oracle Contract

```python
from ontology.interop.System.App import RegisterAppCall

oracleContract = RegisterAppCall('e0d635c7eb2c5eaa7d2207756a4c03a89790934a', 'operation', 'args')
```

The address of currently deployed Oracle contract usecase:

Testnet: `e0d635c7eb2c5eaa7d2207756a4c03a89790934a`

Mainnnet: `a6ee997b142b002d49670ab73803403b09a23fa0`

This Oracle contract provides methods that can be used to fetch public data and certain specialized data such as random numbers from **random.org**

### Creating an Oracle Request

```json
req = """{
		"scheduler":{
			"type": "runAfter",
			"params": "2018-06-15 08:37:18"
		},
		"tasks":[
			{
			    "type": "httpGet",
			    "params": {
				    "url": "https://data.nba.net/prod/v2/20181129/scoreboard.json"
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
							"path": ["games"],
							"sub_type":
							[
								{
									"type": "Struct",
									"sub_type":
									[
										{
											"type": "String",
											"path": ["gameId"]
										},
										{
											"type": "String",
											"path": ["vTeam", "teamId"]
										},
										{
											"type": "String",
											"path": ["vTeam", "score"]
										},
										{
											"type": "String",
											"path": ["hTeam", "teamId"]
										},
										{
											"type": "String",
											"path": ["hTeam", "score"]
										}
									]
								}
							]
						}
					]
				}
			}
		]
	}"""

```

For types and methods of Oracle requests, please refer to [this](./oracle_request.md) link.

### Invoking Oracle Contract to Send Data Request

```python
 oracleContract('CreateOracleRequest', [req, Base58ToAddress('AKQj23VWXpdRou1FCUX3g8XBcSxfBLiezp')])
```

The parameters used to call the Oracle contract method are the request content and address. The address here is the address of the invoking party, and this party pays the gas (processing fee) for the request. This fee is fixed by the Oracle service provider. The Oracle contracts deployed on both the testnet and mainnet charge an extra 0.01 ONG for transmitting the result of the request.

### Fetch Result of Oracle Contract Request

```python
res = oracleContract('GetOracleOutcome', [txHash])

a = Deserialize(res)
b = Deserialize(a[0])
```
The response is an array:

```py
res = state(data, status, errMessage)
```

The first element of the array is the result of the particular request. 
The second element is the status of the request. The data type is `string`. **completed** indicates the data request is successful, **errored** indicates the data request failed.
The third element is the error message, data type being `string`.

And **b** is the final result obtained by resolving the result of data request. The format and data type of **b** is defined by the user's Oracle request. Please refer to [this](./oracle_request.md) for more details on Oracle requests.

Sample code for app contract that uses Oracle service is available for reference [here](https://github.com/ontio/ontology-oracle/blob/master/smartcontract/app.py).
