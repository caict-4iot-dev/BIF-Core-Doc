# 3. Solidity智能合约开发

​		Solidity智能合约使用Spark-Evm引擎，脱胎于原生以太坊EVM架构实现。在星火链合约账户中，Solidity编译后生成的opCode指令码会存储到合约账户中，用于合约的执行。本目录的文档主要介绍在星火链合约平台中支持的 Solidity 合约的特性、语法、功能等。星火链平台支持的solidity语法基本与官方solidity基本一致，目前支持0.4.26版本，可以参考官方文档：

​		https://solidity.readthedocs.io/en/v0.4.26/

## 3.1 Solidity智能合约描述

### 3.1.1 特性说明

* 星火链平台solidity对address关键字进行了重新开发，address表示的合约地址或账户地址，长度为24字节；而官方solidity中address表示的地址是20字节。
* 在星火链链平台上，如果尝试在合约内向一个不存在的地址转账，合约会异常终止；而在以太坊官方 Solidity 合约内向不存在的地址转账时，系统会自动以该地址创建账户。
* 星火链平台中，solidity不支持STATICCALL指令。
* 星火链平台中，solidity不支持CALLCODE指令。
* 星火链平台中，solidity不支持SELFDESTRUCT指令。
* 星火链平台中，合约账户无codehash，solidity不支持EXTCODEHASH指令。
* 星火链平台中，区块中未保存出块人信息，solidity不支持COINBASE指令。
* 星火链平台中，无难度值概念，solidity不支持DIFFICULT指令。
* 星火链平台中，自定义实现了STOI64CHECK指令。
* 星火链EVM引擎处理交易时，使用星火令消耗上限+异常退出（递归深度，栈，内存）+超时。需要指定当前交易接受的最大星火令，执行合约时，按照具体指令扣减星火令。当星火令不足时，合约执行结束。
* 当递归深度超过配置的最大深度时，合约执行结束。此处为兼容星火链框架，合约递归深度未使用以太坊原始最大递归深度1024，合约最大递归深度为4层。

### 3.1.2 指令集比较

| 指令表         | 星火链是否支持 | 以太坊是否支持 |
| -------------- | -------------- | -------------- |
| CREATE2        | 否             | 是             |
| CREATE         | 是             | 是             |
| DELEGATECALL   | 是             | 是             |
| STATICCALL     | 否             | 是             |
| CALL           | 是             | 是             |
| CALLCODE       | 否             | 是             |
| RETURN         | 是             | 是             |
| REVERT         | 是             | 是             |
| SELFDESTRUCT   | 否             | 是             |
| STOP           | 是             | 是             |
| MLOAD          | 是             | 是             |
| MSTORE         | 是             | 是             |
| MSTORE8        | 是             | 是             |
| SHA3           | 是             | 是             |
| LOG0           | 是             | 是             |
| LOG1           | 是             | 是             |
| LOG2           | 是             | 是             |
| LOG3           | 是             | 是             |
| LOG4           | 是             | 是             |
| EXP            | 是             | 是             |
| ADD            | 是             | 是             |
| MUL            | 是             | 是             |
| SUB            | 是             | 是             |
| DIV            | 是             | 是             |
| SDIV           | 是             | 是             |
| MOD            | 是             | 是             |
| SMOD           | 是             | 是             |
| NOT            | 是             | 是             |
| LT             | 是             | 是             |
| GT             | 是             | 是             |
| SLT            | 是             | 是             |
| SGT            | 是             | 是             |
| EQ             | 是             | 是             |
| ISZERO         | 是             | 是             |
| AND            | 是             | 是             |
| OR             | 是             | 是             |
| XOR            | 是             | 是             |
| BYTE           | 是             | 是             |
| SHL            | 是             | 是             |
| SHR            | 是             | 是             |
| SAR            | 是             | 是             |
| ADDMOD         | 是             | 是             |
| MULMOD         | 是             | 是             |
| SIGNEXTEND     | 是             | 是             |
| ADDRESS        | 是             | 是             |
| ORIGIN         | 是             | 是             |
| BALANCE        | 是             | 是             |
| CALLER         | 是             | 是             |
| CALLVALUE      | 是             | 是             |
| CALLDATALOAD   | 是             | 是             |
| CALLDATASIZE   | 是             | 是             |
| RETURNDATASIZE | 是             | 是             |
| CODESIZE       | 是             | 是             |
| EXTCODESIZE    | 是             | 是             |
| CALLDATACOPY   | 是             | 是             |
| RETURNDATACOPY | 是             | 是             |
| EXTCODEHASH    | 否             | 是             |
| CODECOPY       | 是             | 是             |
| EXTCODECOPY    | 是             | 是             |
| GASPRICE       | 是             | 是             |
| BLOCKHASH      | 是             | 是             |
| COINBASE       | 否             | 是             |
| TIMESTAMP      | 是             | 是             |
| NUMBER         | 是             | 是             |
| DIFFICULTY     | 否             | 是             |
| GASLIMIT       | 是             | 是             |
| CHAINID        | 是             | 是             |
| SELFBALANCE    | 是             | 是             |
| POP            | 是             | 是             |
| PUSHC          | 否             | 是             |
| PUSH1/32       | 是             | 是             |
| JUMP           | 是             | 是             |
| JUMPI          | 是             | 是             |
| JUMPC          | 否             | 是             |
| JUMPCI         | 否             | 是             |
| DUP1/16        | 是             | 是             |
| SWAP1/16       | 是             | 是             |
| SLOAD          | 是             | 是             |
| SSTORE         | 是             | 是             |
| PC             | 是             | 是             |
| MSIZE          | 是             | 是             |
| GAS            | 是             | 是             |
| JUMPDEST       | 是             | 是             |
| INVALID        | 是             | 是             |

### 3.1.3 合约数据类型

+ **星火链交易支持的数据类型**

    **基本类型：**

    uint\<M>：M 位的无符号整数，0 < M <= 256、M % 8 == 0。例如：uint32，uint8，uint256。

    int\<M>：以 2 的补码作为符号的 M 位整数，0 < M <= 256、M % 8 == 0。

    uint、int：uint256、int256 各自的同义词。在计算和 函数选择器 中，通常使用 uint256 和 int256。

    bool：等价于 uint8，取值限定为 0 或 1 。在计算和 函数选择器 中，通常使用 bool。

    bytes\<M>：M 字节的二进制类型，0 < M <= 32。

    byte：等价于bytes1。

    bytes：动态大小的字节序列。

    string：动态大小的 unicode 字符串，通常呈现为 UTF-8 编码。

  **一维数组：**

    type[M]\(type=uint/int/address) 有 M 个元素的定长数组，M >= 0，数组元素为给定类型。

  **二维数组：**

    type[][]\(type=uint/int)

+ **平台建议使用数据类型**

| 数据类型             | 参考样例                   | 合约内部是否支持 | 输入参数是否支持 |
| -------------------- | -------------------------- | ---------------- | ---------------- |
| bool                 | bool a = true              | 是               | 是               |
| uint                 | uint a = 1                 | 是               | 是               |
| uint8 ~ uint256      | uint8 a = 1                | 是               | 是               |
| int                  | int a = 1                  | 是               | 是               |
| int8 ~ int256        | int8 a = 1                 | 是               | 是               |
| bytes                | bytes a = “test”           | 是               | 是               |
| bytes1 ~ bytes32     | bytes1 a = “a”             | 是               | 是               |
| string               | string a = “test”          | 是               | 是               |
| int[]                | int256[] a = [1,2,3,4,5]   | 是               | 是               |
| uint[]               | uint256[] a = [1,2,3,4,5]  | 是               | 是               |
| bytes1[] ~ bytes32[] | bytes4[] asd = new bytes4; | 是               | 否               |
| string[]             | string[] a = [“adbc”,”dg”] | 是               | 否               |
| enum                 | enum a {a,b,c}             | 是               | 否               |
| struct               | struct a { string name;}   | 是               | 否               |

## 3.2 Solidity合约工具

​		星火链合约平台支持 Solidity 智能合约，由于Spark-Evm中存在的特性，不能使用以太坊提供的编译器编译处理Solidity合约 ，因此我们提供了星火链Solidity的编译工具docker镜像。

### 3.2.1 镜像下载

```shell
docker pull caictdevelop/bif-solidity:v0.4.26
```

### 3.2.2 使用solc

​		镜像下载之后，需要启动镜像进入容器中，可以使用solc --help 来查看此工具支持的参数说明。

​		常用选项说明：

```bash
--opcodes            Opcodes of the contracts.
--bin                Binary of the contracts in hex.
--abi                ABI specification of the contracts.
```

### 3.2.3 使用实例

+ 启动镜像


```shell
# docker相关操作需要root权限
docker run -it caictdevelop/bif-solidity:v0.4.26 /bin/bash
```

+ 编写合约test.sol

```solidity
pragma solidity ^0.4.26;

contract test{
    function testfun() public returns(string){
        return "hello world";
    }
}
```

+ 编译合约

```bash
#cd /root/solidity/build/solc
#./solc --bin test.sol

======= test.sol:test =======
Binary: 
608060405234801561001057600080fd5b5061013f806100206000396000f300608060405260043610610041576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063031153c214610046575b600080fd5b34801561005257600080fd5b5061005b6100d6565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009b578082015181840152602081019050610080565b50505050905090810190601f1680156100c85780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b60606040805190810160405280600b81526020017f68656c6c6f20776f726c640000000000000000000000000000000000000000008152509050905600a165627a7a723058201a4c9bfcbee5d683f6e46525cf17db2dd46a6ecf5c3f45cbdd148229639263480029
```

## 3.3 合约操作

​		本节内容将以下述合约代码为例，说明合约部署、调用、查询的详细流程。

​		合约实现了在星火链合约中存储和查询数据的接口。合约调用时，将变量设置到合约账户中进行存储。合约查询时，根据输入的参数“key"查询合约账户中存储的变量。

```solidity
pragma solidity ^0.4.26;

contract Test {
	mapping (int256 => string) public keyMap;
	function setKey(int256 key, string val) public{
		keyMap[key] = val;
	}
	function getKey(int256 key) public returns(string){
		return keyMap[key];
	}
}
```

### 3.3.1 合约部署

​		合约部署前，需要先使用solidity编译器生成合约opCode码，具体操作见Solidity合约工具。本节不再详述；

​		生成的opCode指令码如下：

```
608060405234801561001057600080fd5b50610444806...
```

​		Java SDK代码如下：

```java
public void contractCreate() {
    // 初始化参数
    String senderAddress = "did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV";
    String senderPrivateKey = "priSPKkAwBP7w1ajzwp16hNBHvz5psKsksmgZDapcaebzxCS42";
    String payload = "608060405234801561001057600080fd5b50610444806...";

    Long initBalance = ToBaseUnit.ToUGas("0.01");

    BIFContractCreateRequest request = new BIFContractCreateRequest();
    request.setSenderAddress(senderAddress);
    request.setPrivateKey(senderPrivateKey);
    request.setInitBalance(initBalance);
    request.setPayload(payload);
    request.setMetadata("create contract");
    request.setType(1);
    // 调用BIFContractCreate接口
    BIFContractCreateResponse response = sdk.getBIFContractService().contractCreate(request);
    if (response.getErrorCode() == 0) {
        System.out.println(JSON.toJSONString(response.getResult(), true));
    } else {
        System.out.println("error:      " + response.getErrorDesc());
    }
}
```

​		合约部署完成后，查询合约账户返回结果

```json
{
    "total_count":1,
    "transactions":[{
        "actual_fee":"1002548",
        "close_time":1630659261685103,
        "error_code":0,
        "error_desc":"[{\"contract_address\":\"did:bid:efrVoaxEQo9sDqQmv9BCnDstyu1FDTHE\",\"contract_evm_address\":\"6566b65e22eb91b439228dd2cab092cffeccdc2f4f06edd5\",\"operation_index\":0,\"vm_type\":1}]",
        "hash":"1f6f75935ddf164237cd56a5b8a678545ac74c92dece1c5c73873ba5345f0db1",
        "ledger_seq":20961,
        "signatures":[{
            "public_key":"b0656669660116b0b2ddfbba3f155962b311dc0afb671e2da069563a70fccabd9ff8c4",
            "sign_data":"12f528b7ba3c2722e4aad9f46374582c10548e3005ad4cd3d106e54b21e800942d6dd80e88d6f09e0482ce84b1425508ea48a69c6c2ecd4b9a61c9b35fb2390e"
        }],
        "transaction": {
            "fee_limit":2000000,
            "gas_price":1,
            "metadata":"create contract",
            "nonce":50,
            "operations": [{
                "create_account": {
                    "contract": {
                        "payload":"608060405234801561001057600080fd5b50610444806100206000396000",
                        "type":1
                    },
                    "init_balance":1000000,
                    "priv": {
                        "thresholds":{
                            "tx_threshold":1
                        }
                    }
                },
                "type":1
            }],
            "source_address":"did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV"
        },
        "tx_size":2285
    }]
}
```

### 3.3.2 合约调用

​		当调用合约时，合约接口input按照指定格式输入参数：

```json
{
	"function":"xxx",//待调用函数声明 入setKey(int256,string)，仅包含函数名（形参类型）
	"args":{ 
		"xxx":""//待调用合约的参数，多个参数使用“，”分隔
	}
}
```

​		Java SDK代码如下：

```java
public void contractInvokeByGas() {
    // 初始化参数
    String senderAddress = "did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV";
    String contractAddress = "did:bid:efrVoaxEQo9sDqQmv9BCnDstyu1FDTHE";
    String senderPrivateKey = "priSPKkAwBP7w1ajzwp16hNBHvz5psKsksmgZDapcaebzxCS42";
    Long amount = 1000L;

    BIFContractInvokeRequest request = new BIFContractInvokeRequest();
    request.setSenderAddress(senderAddress);
    request.setPrivateKey(senderPrivateKey);
    request.setContractAddress(contractAddress);
    request.setBIFAmount(amount);
    request.setMetadata("contract invoke");
     request.setInput("{\"function\":\"setKey(int256,string)\",\"args\":\"123,'hello world'\"}");
    // 调用 BIFContractInvoke 接口
    BIFContractInvokeResponse response = sdk.getBIFContractService().contractInvoke(request);
    if (response.getErrorCode() == 0) {
        System.out.println(JSON.toJSONString(response.getResult(), true));
    } else {
        System.out.println("error:      " + response.getErrorDesc());
    }
}
```

### 3.3.3 合约查询

​		当调用合约时，合约接口input按照指定格式输入参数：

```json
{
	"function":"xxx",//待调用函数声明 入getKey(int256)，仅包含函数名（形参类型）
	"args":{ 
		"xxx":""//待调用合约的参数，多个参数使用“，”分隔
	},
	"return":"returns(xxx)"//待返回数据类型，returns(形参类型)
}
```

​		Java SDK代码如下:

```java
public void callContract() {
    // Init variable
    // Contract address
    String contractAddress = "did:bid:efrVoaxEQo9sDqQmv9BCnDstyu1FDTHE";

    // Init request
    BIFContractCallRequest request = new BIFContractCallRequest();
    request.setContractAddress(contractAddress);
    request.setInput("{\"function\":\"getKey(int256)\",\"args\":123, \"return\":\"returns(string)\"}");

    // Call call
    BIFContractCallResponse response = sdk.getBIFContractService().contractQuery(request);
    if (response.getErrorCode() == 0) {
        BIFContractCallResult result = response.getResult();
        System.out.println(JSON.toJSONString(result, true));
    } else {
        System.out.println("error: " + response.getErrorDesc());
    }
}
```

## 3.4 Solidity智能合约示例

+  ERC20智能合约示例

  具体合约示例见[5.1 ERC20合约示例](./5-Contract.md)。
