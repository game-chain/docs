# API接口



gamed使用REST RPC接口，插件可以在API服务器上注册自己的端点。 本页将解释如何使用一些API来获取有关区块链和发送事务的信息。

在查询gamed之前，您必须先启用必要的API插件。根据您希望启用的API，将以下行添加到您的gamed的config.ini中：另外，对于电子钱包API，您还可以通过单独运行game-walletd将电子钱包功能与gamed分开。

对于以下指南，我们假定我们已经在127.0.0.1:8888（启用了链接API插件，电子钱包API插件已禁用）以及在127.0.0.1:8889上运行的game-walletd运行。

| 版本号 | 制定人 | 制定日期   | 修订日期   |
| :----- | :----- | :--------- | ---------- |
| 1.1.0  | game   | 2020-03-20 | 2020-03-20 |



## 链接口

### get_info

获取与节点相关的最新信息

**调用参数**

无

**响应参数**

- server_version：节点版本

- head_block_num：链头区块序号

- lase_irreversible_block_num：最后不可逆区块号

- head_block_id：链头区块ID

- head_block_time：链头区块生成时间

- head_block_producer：链头区块出块账号

  

**用法示例**

```
curl http://127.0.0.1:8888/v1/chain/get_info
```

**结果示例**

```json
{
	"server_version": "b2eb1667",
	"head_block_num": 259590,
	"last_irreversible_block_num": 259573,
	"head_block_id": "0003f60677f3707f0704f16177bf5f007ebd45eb6efbb749fb1c468747f72046",
	"head_block_time": "2017-12-10T170536",
	"head_block_producer": "initp",
	"recent_slots": "1111111111111111111111111111111111111111111111111111111111111111",
	"participation_rate": "1.00000000000000000"
}
```



### get_block

获取一个块的信息

**调用参数**

- block_num_or_id：字符串，要提取数据的区块序号或ID

**响应参数**

- previous：前一个区块的ID，字符串

- timestamp：区块时间戳，时间字符串

- transaction_mroot

- action_mroot

- block_mroot

- producer：出块账号，字符串

- schedule_version：调度器版本，数值

- new_producers：新出现的出块账号集合，空或数组

- producer_signature：出块账号的签名，字符串

- regions：分区集合，数组

- input_transactions：区块中包含的交易集合，数组

- id：区块ID，字符串

- block_num：区块序号，整数

- ref_block_prefix：参考块前缀，整数

  

**用法示例**

```
$ curl http://127.0.0.1:8888/v1/chain/get_block -X POST -d '{"block_num_or_id":5}' 

$ curl http://127.0.0.1:8888/v1/chain/get_block -X POST -d '{"block_num_or_id":0000000445a9f27898383fd7de32835d5d6a978cc14ce40d9f327b5329de796b}'

```

**结果示例**

```json
{
	"previous": "0000000445a9f27898383fd7de32835d5d6a978cc14ce40d9f327b5329de796b",
	"timestamp": "2017-07-18T20:16:36",
	"transaction_merkle_root": "0000000000000000000000000000000000000000000000000000000000000000",
	"producer": "initf",
	"producer_changes": [],
	"producer_signature": "204cb94b3186c3b4a7f88be4e9db9f8af2ffdb7ef0f27a065c8177a5fcfacf876f684e59c39fb009903c0c59220b147bb07f1144df1c65d26c57b534a76dd29073",
	"cycles": [],
	"id": "000000050c0175cbf218a70131ddc3c3fab8b6e954edef77e0bfe7c36b599b1d",
	"block_num": 5,
	"ref_block_prefix": 27728114
}
```

### get_account

获取账户的信息

**调用参数**

- account_name：账号名称，字符串

**响应参数**

- account_name：账号名称

- permissions: 账号权限集合，权限对象数组，每个权限对象的成员如下：

  - perm_name：权限名称
  - parent：父权限名称
  - required_auth：授权条件，JSON对象，成员如下：
    - threshold：阈值，整数
    - keys：公钥信息集合，数组，每个公钥信息对象成员如下：
      - key：公钥，字符串
      - weight：权重
      - accounts：账号数组

- 

  

**用法示例**

```
$ curl http://127.0.0.1:8888/v1/chain/get_account -X POST -d '{"account_name":"inita"}'
```

**结果示例**

```json
{
	"name": "inita",
	"game_balance": "999998.9574 game",
	"staked_balance": "0.0000 game",
	"unstaking_balance": "0.0000 game",
	"last_unstaking_time": "2106-02-07T06:28:15",
	"permissions": [{
		"name": "active",
		"parent": "owner",
		"required_auth": {
			"threshold": 1,
			"keys": [{
				"key": "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV",
				"weight": 1
			}],
			"accounts": []
		}
	}, {
		"name": "owner",
		"parent": "owner",
		"required_auth": {
			"threshold": 1,
			"keys": [{
				"key": "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV",
				"weight": 1
			}],
			"accounts": []
		}
	}]
}
```

### get_code

获取智能合约代码

**调用参数**

- account_name：账号名称，字符串

**响应参数**

- name：账号名称，字符串

- code_hash：当前部署代码的哈希值，字符串

- wast：当前部署代码的等价文本格式，字符串

- abi：当前部署代码的abi描述对象，其成员如下：

  - types：abi中定义的新类型，数组
  - structs：abi中定义的新结构，数组
  - actions：abi中定义的合约动作，数组
  - tables：abi中定义的数据表，数组

- 

  

**用法示例**

```
$ curl http://127.0.0.1:8888/v1/chain/get_code -X POST -d '{"account_name":"currency"}'
```

**结果示例**

```json
{
	"name": "currency",
	"code_hash": "a1c8c84b4700c09c8edb83522237439e33cf011a4d7ace51075998bd002e04c9",
	"wast": "(module\n (type $0 (func (param i64 i64 i32) (result i32)))\n ...truncated",
	"abi": {
		"types": [{
			"new_type_name": "account_name",
			"type": "name"
		}],
		"structs": [{
			"name": "transfer",
			"base": "",
			"fields": [{
				"name": "from",
				"type": "account_name"
			}, {
				"name": "to",
				"type": "account_name"
			}, {
				"name": "quantity",
				"type": "uint64"
			}]
		}, {
			"name": "account",
			"base": "",
			"fields": [{
				"name": "key",
				"type": "name"
			}, {
				"name": "balance",
				"type": "uint64"
			}]
		}],
		"actions": [{
			"name": "transfer",
			"type": "transfer"
		}],
		"tables": [{
			"name": "account",
			"type": "account",
			"index_type": "i64",
			"key_names": ["key"],
			"key_types": ["name"]
		}]
	}
}
```

### get_table_rows

调用指定多索引数据表的数据行

**调用参数**

- scope：数据表作用域，字符串
- code：合约的托管账号名，字符串
- table：数据表名称，字符串
- table_key：键名称，字符串，可选
- json：是否返回JSON格式的结果，布尔型，默认值：true
- lower_bound：结果数据应当满足的下界值，字符串，可选
- upper_bound：结果数据应当满足的上界值，字符串，可选
- limit：返回结果数量上限，整数，默认值：10
- index_position：要使用的索引序号，例如，主键索引为1，次级索引为2，字符串，默认值：1
- key_type：索引键类型，例如uint64_t或name，字符串
- encode_type：编码类型，字符串，默认值：dec

**响应参数**



**用法示例**

```
$ curl http://127.0.0.1:8888/v1/chain/get_table_rows -X POST -d '{"scope":"inita", "code":"currency", "table":"account", "json": true}' 
$ curl http://127.0.0.1:8888/v1/chain/get_table_rows -X POST -d '{"scope":"inita", "code":"currency", "table":"account", "json": true, "lower_bound":0, "upper_bound":-1, "limit":10}'

```

**结果示例**

```json
{
	"rows": [{
		"account": "account",
		"balance": 1000
	}],
	"more": false
}
```

### abi_json_to_bin

将json序列化为二进制十六进制。得到的二进制十六进制通常用于push_transaction中的数据字段。

**调用参数**

- code：合约托管账号名称，字符串
- action：要调用的合约动作名称，字符串
- args：合约动作参数，JSON对象

**响应参数**

  

**用法示例**

```
$ curl http://127.0.0.1:8888/v1/chain/abi_json_to_bin -X POST -d '{"code":"currency", "action":"transfer", "args":{"from":"initb", "to":"initc", "quantity":1000}}'
```

**结果示例**

```json
{
	"binargs": "000000008093dd74000000000094dd74e803000000000000",
	"required_scope": [],
	"required_auth": []
}
```

### abi_bin_to_json

将二进制十六进制序列化为json。

**调用参数**

JSON对象，用来声明要进行序列化的合约动作调用，其成员如下：

- code：合约托管账号名称，字符串
- action：要调用的合约动作名称，字符串
- args：合约动作参数，16进制码流，字符串

**响应参数**

  

**用法示例**

```
$ curl http://127.0.0.1:8888/v1/chain/abi_bin_to_json -X POST -d '{"code":"currency", "action":"transfer", "binargs":"01002193dd7200000001294dd12103000000120"}'
```

**结果示例**

```json
{
	"args": {
		"from": "initb",
		"to": "initc",
		"quantity": 1000
	},
	"required_scope": [],
	"required_auth": []
}
```

### push_transaction

此方法预期采用JSON格式的事务，并将尝试将其应用于区块链。

**调用参数**

- signatures：签名数组
- compression：是否压缩格式，布尔类型，默认值：false
- packed_context_free_data：上下文无关的数据
- packed_tx：序列化的交易数据

**响应参数**

成功的返回

成功时它将返回HTTP 200和事务ID。

{ "transaction_id": "..." }

仅仅因为交易是在本地进行的，并不意味着交易已经被合并到一个区块中。

错误的返回

如果发生错误，它将返回HTTP 400（无效参数）或500（内部服务器错误）

HTTP/1.1 500 Internal Server Error Content-Length: 1466 ...error message...

**用法示例**

```
$ curl -X POST --url http://127.0.0.1:8888/v1/chain/push_transaction -d '{
  ...
}'
```

**结果示例**

```json
{
    'transaction_id' = "..."
}
```

### push_transactions

该方法一次推送多个事务。

**调用参数**

JSON对象，指定一组要提交的交易，成员如下：

- body

**响应参数**

  事务ID

**用法示例**

```
curl http://localhost:8888/v1/chain/push_transaction -X POST -d '
[{
	"ref_block_num": "101",
	"ref_block_prefix": "4159312339",
	"expiration": "2017-09-25T06:28:49",
	"scope": ["initb", "initc"],
	"actions": [{
		"code": "currency",
		"type": "transfer",
		"recipients": ["initb", "initc"],
		"authorization": [{
			"account": "initb",
			"permission": "active"
		}],
		"data": "000000000041934b000000008041934be803000000000000"
	}],
	"signatures": [],
	"authorizations": []
}, {
	"ref_block_num": "101",
	"ref_block_prefix": "4159312339",
	"expiration": "2017-09-25T06:28:49",
	"scope": ["inita", "initc"],
	"actions": [{
		"code": "currency",
		"type": "transfer",
		"recipients": ["inita", "initc"],
		"authorization": [{
			"account": "inita",
			"permission": "active"
		}],
		"data": "000000008040934b000000008041934be803000000000000"
	}],
	"signatures": [],
	"authorizations": []
}]
'

```

**结果示例**

```json
{
    'transaction_id' = "..."
}
```

### get_required_keys

将json序列化为二进制十六进制。得到的二进制十六进制通常用于push_transaction中的数据字段。

**调用参数**

JSON对象，用来声明要签名的交易和可用的公钥集合，其成员如下：

- transaction：交易数据，JSON对象
- available_keys：可用公钥，数组

**响应参数**

  返回一组用于签名的公钥

**用法示例**

```
curl http://localhost:8888/v1/chain/get_required_keys -X POST -d '
{
	"transaction": {
		"ref_block_num": "100",
		"ref_block_prefix": "137469861",
		"expiration": "2017-09-25T06:28:49",
		"scope": ["initb", "initc"],
		"actions": [{
			"code": "currency",
			"type": "transfer",
			"recipients": ["initb", "initc"],
			"authorization": [{
				"account": "initb",
				"permission": "active"
			}],
			"data": "000000000041934b000000008041934be803000000000000"
		}],
		"signatures": [],
		"authorizations": []
	},
	"available_keys": ["4toFS3YXEQCkuuw1aqDLrtHim86Gz9u3hBdcBw5KNPZcursVHq", "7d9A3uLe6As66jzN8j44TXJUqJSK3bFjjEEqR4oTvNAB3iM9SA", "6MRyAjQq8ud7hV23sdfsPJqcVpscN5So8BhtHuGYqE1T5GDW5CV"]
}
'

```

**结果示例**

```json
{ "required_keys": [ "6MRyAjQq8ud7hV23sdfsPJqcVpscN5So8BhtHuGYqE1T5GDW5CV" ] }
```



## 钱包接口

### wallet_create

用给定的名称创建一个新的钱包.

**调用参数**

无

**响应参数**

  

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/create -X POST -d '"default"'
```

**结果示例**

```json

```

### wallet_create

用给定的名称创建一个新的钱包.

**调用参数**

无

**响应参数**

  

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/create -X POST -d '"default"'
```

**结果示例**

```json

```

### wallet_create

用给定的名称创建一个新的钱包.

**调用参数**

钱包名称

**响应参数**

该命令将返回将来可用于解锁钱包的密码.

PW5KFWYKqvt63d4iNvedfDEPVZL227D3RQ1zpVFzuUwhMAJmRAYyX

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/create -X POST -d '"default"'
```

### wallet_open

打开给定名称的现有钱包

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/open -X POST -d '"default"'
```

**结果示例**

```json
{}
```
### wallet_lock_all

锁定所有钱包

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/lock_all
```

**结果示例**

```json
{}
```

### wallet_unlock

用给定的名称和密码解锁钱包.

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/unlock -X POST -d '["default", "PW5KFWYKqvt63d4iNvedfDEPVZL227D3RQ1zpVFzuUwhMAJmRAYyX"]'
```

**结果示例**

```json
{}
```

### wallet_list

列出所有钱包.

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/list_wallets
```

**结果示例**

```json
["default *"]
```

### wallet_list_keys

列出所有钱包中的所有密钥对.

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/list_keys
```

**结果示例**

```json
[["6MRyAjQq8ud7hVNYcfnVP234N5So8BhtHuGYqET5GDW5CV","5KQwrPb342237FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"]]
```
### wallet_get_public_keys

列出所有钱包中的所有公钥.

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/get_public_keys
```

**结果示例**

```json
["6MRyAjQq8ud7hVNYcfnVP234N5So8BhtHuGYqET5GDW5CV"]
```
### wallet_set_timeout

设置钱包自动锁定超时（以秒为单位）.

**用法示例**

```
$ curl http://localhost:8889/v1/wallet/set_timeout -X POST -d '10'
```

**结果示例**

```json
{}
```
