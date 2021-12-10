# 5. 智能合约示例

​		本节为星火链智能合约的示例。

​		5.1节描述使用Solidity语言编写的智能合约示例，实现ERC20功能；

​		5.2节描述使用JavaScript语言编写的工业互联网标识合约示例，实现管理标识的功能；

​		5.3节描述使用JavaScript语言编写的SQL智能合约，实现学生管理功能。

## 5.1 ERC20合约
<h2 id="min-content" style="display:none;"> </h2>
​		本节描述通过星火链网实现并部署ERC20智能合约。

​		ERC20可以简单理解成以太坊上的一个代币协议，所有基于以太坊开发的代币合约都遵守这个协议。有关ERC20标准可以参考[官方文档](https://theethereum.wiki/w/index.php/ERC20_Token_Standard)。

### 5.1.1 准备工作

​		用户需准备好docker、docker-compose环境，本节不再详细说明。

### 5.1.2 合约说明

​		本合约为Solidity合约示例，实现ERC20功能 。

+ **合约接口说明**

| 接口                                                | 返回值  | 描述                                                         |
| --------------------------------------------------- | ------- | ------------------------------------------------------------ |
| name()                                              | string  | 代币名称获取                                                 |
| symbol()                                            | string  | 代币符号获取                                                 |
| totalSupply()                                       | uint256 | 发行方总代币金额获取                                         |
| transfer(address to,uint256 value)                  | 无      | 代币转移接口，从自己（创建交易者）账号发送`_value`个代币到 `_to`账号 |
| allowance(address,address)                          | 无      | 账号所有者操作列表                                           |
| transferFrom(address from,address to,uint256 value) | 无      | 账号之间代币交易转移<br />from 发送者地址<br />to 接收者地址<br />value 转移数额 |
| approve(address spender,uint256 value)              | 无      | 设置某个地址（合约）可以创建交易者名义花费的代币数，允许发送者`_spender` 花费不多于 `_value` 个代币 |
| burn(uint256 value)                                 | 无      | 销毁我（创建交易者）账户中指定个代币                         |
| burnFrom(address from, uint256 value)               | 无      | 销毁用户账户中指定个代币                                     |

+ **合约文件**

```solidity
pragma solidity ^0.4.26;

contract TokenERC20 {
    string public name; // ERC20标准
    string public symbol; // ERC20标准
    uint8 public decimals = 2;  // ERC20标准，decimals 可以有的小数点个数，最小的代币单位。18 是建议的默认值
    uint256 public totalSupply; // ERC20标准 总供应量

    // 用mapping保存每个地址对应的余额 ERC20标准
    mapping (address => uint256) public balanceOf;
    // 存储对账号的控制 ERC20标准
    mapping (address => mapping (address => uint256)) public allowance;

    // 事件，用来通知客户端交易发生 ERC20标准
    event Transfer(address indexed from, address indexed to, uint256 value);

    // 事件，用来通知客户端代币被消费 ERC20标准
    event Burn(address indexed from, uint256 value);

    /**
     * 初始化构造
     */
    function TokenERC20(uint256 initialSupply, string tokenName, string tokenSymbol) public {
        totalSupply = initialSupply * 10 ** uint256(decimals);  // 供应的份额，份额跟最小的代币单位有关，份额 = 币数 * 10 ** decimals。
        balanceOf[msg.sender] = totalSupply;                // 创建者拥有所有的代币
        name = tokenName;                                   // 代币名称
        symbol = tokenSymbol;                               // 代币符号
    }

    /**
     * 代币交易转移的内部实现
     */
    function _transfer(address _from, address _to, uint _value) internal {
        // 确保目标地址不为0x0，因为0x0地址代表销毁
        require(_to != 0x0);
        // 检查发送者余额
        require(balanceOf[_from] >= _value);
        // 确保转移为正数个
        require(balanceOf[_to] + _value > balanceOf[_to]);

        // 以下用来检查交易，
        uint previousBalances = balanceOf[_from] + balanceOf[_to];
        // Subtract from the sender
        balanceOf[_from] -= _value;
        // Add the same to the recipient
        balanceOf[_to] += _value;
        Transfer(_from, _to, _value);

        // 用assert来检查代码逻辑。
        assert(balanceOf[_from] + balanceOf[_to] == previousBalances);
    }

    /**
     *  代币交易转移
     *  从自己（创建交易者）账号发送`_value`个代币到 `_to`账号
     * ERC20标准
     * @param _to 接收者地址
     * @param _value 转移数额
     */
    function transfer(address _to, uint256 _value) public {
        _transfer(msg.sender, _to, _value);
    }

    /**
     * 账号之间代币交易转移
     * ERC20标准
     * @param _from 发送者地址
     * @param _to 接收者地址
     * @param _value 转移数额
     */
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(_value <= allowance[_from][msg.sender]);     // Check allowance
        allowance[_from][msg.sender] -= _value;
        _transfer(_from, _to, _value);
        return true;
    }

    /**
     * 设置某个地址（合约）可以创建交易者名义花费的代币数。
     *
     * 允许发送者`_spender` 花费不多于 `_value` 个代币
     * ERC20标准
     * @param _spender The address authorized to spend
     * @param _value the max amount they can spend
     */
    function approve(address _spender, uint256 _value) public
    returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        return true;
    }

    /**
     * 销毁我（创建交易者）账户中指定个代币
     *-非ERC20标准
     */
    function burn(uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value);   // Check if the sender has enough
        balanceOf[msg.sender] -= _value;            // Subtract from the sender
        totalSupply -= _value;                      // Updates totalSupply
        Burn(msg.sender, _value);
        return true;
    }

    /**
     * 销毁用户账户中指定个代币
     *-非ERC20标准
     * Remove `_value` tokens from the system irreversibly on behalf of `_from`.
     *
     * @param _from the address of the sender
     * @param _value the amount of money to burn
     */
    function burnFrom(address _from, uint256 _value) public returns (bool success) {
        require(balanceOf[_from] >= _value);                // Check if the targeted balance is enough
        require(_value <= allowance[_from][msg.sender]);    // Check allowance
        balanceOf[_from] -= _value;                         // Subtract from the targeted balance
        allowance[_from][msg.sender] -= _value;             // Subtract from the sender's allowance
        totalSupply -= _value;                              // Update totalSupply
        Burn(_from, _value);
        return true;
    }
}
```

### 5.1.3 合约部署

* 利用JAVA SDK完成智能合约的部署，部署前需先通过星火链网solidity编译器将合约编译为opCode指令码

```
608060405260028060006101000a81548160ff02191690...
```

* 初始化参数

```
{\"function\":\"TokenERC20(uint256,string,string)\",\"args\":\"1000,'spark token','T'\"}
```

* 使用JAVA SDK连接节点环境，部署合约

```java
public void contractCreate() {
        // 初始化参数
        String senderAddress = "did:bid:efkxG5de56Hg3W5MScC5dfeBXU8zHySb";
        String senderPrivateKey = "priSPKp8W4kZhVvR9k3JgVW5tQeHXazCf54duiUVnyXtBJxSgK";
        String payload = "608060405260028060006101000a81548160ff02191690...";
        Long initBalance = ToBaseUnit.ToUGas("0.01");

        BIFContractCreateRequest request = new BIFContractCreateRequest();
        request.setSenderAddress(senderAddress);
        request.setPrivateKey(senderPrivateKey);
        request.setInitBalance(initBalance);
        request.setPayload(payload);
        request.setMetadata("create evm contract");
        request.setType(1);
        request.setInitInput("{\"function\":\"TokenERC20(uint256,string,string)\",\"args\":\"1000,'spark token','T'\"}");
        // 调用BIFContractCreate接口
        BIFContractCreateResponse response = sdk.getBIFContractService().contractCreate(request);
        if (response.getErrorCode() == 0) {
            System.out.println(JSON.toJSONString(response.getResult(), true));
        } else {
            System.out.println("error:      " + response.getErrorDesc());
        }
    }
```

### 5.1.4 合约调用

```java
public void contractInvokeByGas() {
    // 初始化参数
    String senderAddress = "did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV";
    String contractAddress = "did:bid:efkeicT6SsyJwVkeNKsjKJ4bpW5SKYHY";
    String senderPrivateKey = "priSPKkAwBP7w1ajzwp16hNBHvz5psKsksmgZDapcaebzxCS42";
    Long amount = 1000L;

    BIFContractInvokeRequest request = new BIFContractInvokeRequest();
    request.setSenderAddress(senderAddress);
    request.setPrivateKey(senderPrivateKey);
    request.setContractAddress(contractAddress);
    request.setBIFAmount(amount);
    request.setMetadata("contract invoke");
    request.setInput("{\"function\":\"transfer(address,uint256)\",\"args\":\"did:bid:efdGtrMkGgHuN4qb7Usg1KsARi2JN7nB,10\"}");
    // 调用 BIFContractInvoke 接口
    BIFContractInvokeResponse response = sdk.getBIFContractService().contractInvoke(request);
    if (response.getErrorCode() == 0) {
        System.out.println(JSON.toJSONString(response.getResult(), true));
    } else {
        System.out.println("error:      " + response.getErrorDesc());
    }
}
```

### 5.1.5 合约查询

* 使用JAVA SDK进行合约查询，以查询name接口为例

```java
public void callContract() {
    // Init variable
    // Contract address
    String contractAddress = "did:bid:efkeicT6SsyJwVkeNKsjKJ4bpW5SKYHY";

    // Init request
    BIFContractCallRequest request = new BIFContractCallRequest();
    request.setContractAddress(contractAddress);
    request.setInput("{\"function\":\"name()\",\"return\":\"returns(string)\"}");

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

* 返回结果

```json
{
    "query_rets":[
        {
            "result":{
                "data":"[spark token]",
                "gasused":928
            },
            "error":{
            }
        }
    ]
}
```

## 5.2 工业互联网标识合约

​		本节描述通过星火链网实现并部署工业互联网标识智能合约。

​		工业互联网标识映射的信息资源是具有唯一性的功能，对比传统互联网的DNS功能作用，可以通过标识代表映射具体某资源，而星火链工业互联网标识合约是基于星火链网主链，将各顶级GHR、二级SHR以及企业LHS的数据维护在主链账本中，通过自主身份体系和BID，实现对其所辖数据的自管理。

### 5.2.1 准备工作

​		目前星火链节点主要运行在linux系统服务器上，系统环境在centos7.5及以上，g++ >=4.8.2版本。

​		智能合约只有部署到链上才能运行，因此部署运行之前首先要编译启动链节点。

### 5.2.2 合约说明

​		标识合约是由JavaScript语言开发，在链上由V8虚拟机引擎解释执行。

​		标识合约有GHR（顶级节点），SHR（二级节点），LHS（企业节点）三种类型，在链上部署合约后即生成对应的合约账户，用户就可以根据合约接口分别进行注册，修改，删除以及查询的操作，其中注册，修改以及删除操作需要有白名单权限才可以操作，所以必须在部署发行到链上后，先进行白名单设置在进行后续接口调用操作。

+ 合约接口说明

| 合约函数          | 说明       |
| ----------------- | ---------- |
| createZidData     | 创建标识   |
| modifyZidData     | 修改标识   |
| queryZidRecord    | 查询标识   |
| removeZidData     | 删除标识   |
| setAuditWhitelist | 设置白名单 |

+ **合约示例**

```js
'use strict';


//ZID  data操作
const ZidOp = {
    'INS'                : '0',      //insert
    'DEL'                : '1',      //delete
    'MOD'                : '2',       //modify
	'QUERY'				 : '3'       //query
};

const ZID_RECORD_PRE = 'zidrecord';
const GHR_ZID_INIT = 'ghr_zid_init';
const SHR_ZID_INIT = 'shr_zid_init';
const LHS_ZID_INIT = 'lhs_zid_init';

/*
    ***********************************
                事件定义
    ***********************************
*/
//白名单设置
const TLOG_SET_WT = 'tlog_set_wt';

//创建标识事件
const TLOG_ZID_CREATE_CONTRACT_ADDRESS = 'tlog_zid_create_contract_address';

//修改标识事件
const TLOG_ZID_MODIFY_CONTRACT_ADDRESS = 'tlog_zid_modify_contract_address';

//删除标识事件
const TLOG_ZID_REMOVE_CONTRACT_ADDRESS = 'tlog_zid_remove_contract_address';


/*
    ***********************************
                权限控制
    ***********************************
*/
function isValidator(addr){
    return Chain.isValidator(addr);
}

function getWhiteKey(addr){
    return 'wk-' + addr;
}
  
function checkInAuditWhitelist(addr) {
    const res = Chain.load(getWhiteKey(addr));
    return res !== false;
}

function getKey(first, second, third = ''){
    return (third === '') ? (first + '_' + second) : (first + '_' + second + '_' + third);
}

function loadObj(key){
	let data = Chain.load(getKey(ZID_RECORD_PRE,key));
	return JSON.parse(data);
}

/* 
    函数名称：setAuditWhitelist()
    函数描述：白名单注册接口；该接口需要验证者账户调用
    调用方式：main
    参数：
        caller_address         待操作白名单账户
        addFlag         操作
    返回值：无
*/
function setAuditWhitelist(paramObj) {
    Utils.assert(paramObj.caller_address !== undefined, 'Param obj has no caller_address');
    Utils.assert(paramObj.addFlag !== undefined, 'Param obj has no addFlag');

    Utils.assert(Utils.addressCheck(paramObj.caller_address), `The caller_address(${paramObj.caller_address}) is invalid`);
    Utils.assert(typeof paramObj.addFlag === 'boolean', `The addFlag(${paramObj.addFlag}) is invalid`);

    //isValidator check
    Utils.assert(isValidator(Chain.msg.sender), `The caller_address(${Chain.msg.sender}) is not validator`);
    
    if (paramObj.addFlag === true) {
        Utils.assert(loadObj(getWhiteKey(paramObj.caller_address)) === false, `The account(${paramObj.caller_address}) is already in whitelist`);
        Chain.store(getWhiteKey(paramObj.caller_address), '');
    } else {
        Chain.del(getWhiteKey(paramObj.caller_address));
    }

    Chain.tlog(TLOG_SET_WT, Chain.msg.sender, paramObj.caller_address, `${paramObj.addFlag}`);
}

function createZidValue(key, value){
    Chain.store(getKey(ZID_RECORD_PRE, key), JSON.stringify(value));
}

function deleteZidValue(key){
    Chain.del(getKey(ZID_RECORD_PRE,key));
}

function modifyZidValue(key, value) {
    deleteZidValue(key);
    createZidValue(key, value);
} 

function queryZidRecord(paramObj){
    Utils.assert(paramObj.zid !== undefined, 'Param obj has no zid');
	
    return loadObj(paramObj.zid);
}
/* 
    函数名称：createZidData()
    函数描述：创建标识;

    调用方式：main
    参数：
        zid             标识
        value        	标识值数组
		index			标识值索引
		type			标识类型
		data			标识数据结构
		format			数据格式
		value			标识值
		ttl				ttl值
		ttlType			ttl类型
		timestamp		时间戳
		references		引用
		adminRead		管理员可读
		adminWrite		管理员可写
		publicRead		公共可读
		publicWrite		公共可写
    返回值：
        无
*/
function createZidData(paramObj) {
    //check param
    Utils.assert(paramObj.zid !== undefined, 'Param obj has no zid');
    Utils.assert(paramObj.value[0].index !== undefined, 'Param obj has no index');
    Utils.assert(paramObj.value[0].type !== undefined, 'Param obj has no type');
    Utils.assert(paramObj.value[0].data.format !== undefined, 'Param obj has no format');
	Utils.assert(paramObj.value[0].data.value !== undefined, 'Param obj has no data value');
	Utils.assert(paramObj.value[0].ttl !== undefined, 'Param obj has no ttl');
	Utils.assert(paramObj.value[0].ttlType !== undefined, 'Param obj has no ttl');
	Utils.assert(paramObj.value[0].timestamp !== undefined, 'Param obj has no timestamp');

    //检测操作权限
    Utils.assert(checkInAuditWhitelist(Chain.msg.sender), `The address(${Chain.msg.sender}) checkInAuditWhitelist error`);
	
	//判断对应zid是否已存在
	Utils.assert(loadObj(paramObj.zid) === false, 'createZidData exist of zid:' + paramObj.zid);
	
	let regiRecord = {  
	    'value': [{
		'index': paramObj.value[0].index,
		'type': paramObj.value[0].type,
		'data': {
			'format': paramObj.value[0].data.format,
			'value': paramObj.value[0].data.value
		},
		'ttl': paramObj.value[0].ttl,
		'ttlType': paramObj.value[0].ttlType,
		'timestamp': paramObj.value[0].timestamp,
		'references': paramObj.value[0].references,
		'adminRead': paramObj.value[0].adminRead,
		'adminWrite': paramObj.value[0].adminWrite,
		'publicRead': paramObj.value[0].publicRead,
		'publicWrite': paramObj.value[0].publicWrite
	}]
    };
	
    createZidValue(paramObj.zid, regiRecord);
	Chain.tlog(TLOG_ZID_CREATE_CONTRACT_ADDRESS, 'zid data record', 'create', paramObj.zid);
}

function removeZidData(paramObj) {
	//check param
	Utils.assert(paramObj.zid !== undefined, 'Param obj has no zid');
	if(paramObj.opFlag === ZidOp.DEL)
	{
		 //检测操作权限
		Utils.assert(checkInAuditWhitelist(Chain.msg.sender), `The address(${Chain.msg.sender}) checkInAuditWhitelist error`);
		
		deleteZidValue(paramObj.zid);
		Chain.tlog(TLOG_ZID_REMOVE_CONTRACT_ADDRESS, 'zid data record', 'remove', paramObj.zid);
	}
}

function modifyZidData(paramObj) {
	//check param
    Utils.assert(paramObj.zid !== undefined, 'Param obj has no zid');
	Utils.assert(paramObj.opFlag !== undefined, 'Param obj has no opFlag');
    Utils.assert(paramObj.value[0].index !== undefined, 'Param obj has no index');
    Utils.assert(paramObj.value[0].type !== undefined, 'Param obj has no type');
    Utils.assert(paramObj.value[0].data.format !== undefined, 'Param obj has no format');
	Utils.assert(paramObj.value[0].data.value !== undefined, 'Param obj has no data value');
	Utils.assert(paramObj.value[0].ttl !== undefined, 'Param obj has no ttl');
	Utils.assert(paramObj.value[0].ttlType !== undefined, 'Param obj has no ttl');
	Utils.assert(paramObj.value[0].timestamp !== undefined, 'Param obj has no timestamp');
	
    //检测操作权限
    Utils.assert(checkInAuditWhitelist(Chain.msg.sender), `The address(${Chain.msg.sender}) checkInAuditWhitelist error`);
	
	//判断对应zid是否已存在
	Utils.assert(loadObj(paramObj.zid) === false, 'modifyZidData exist of zid:' + paramObj.zid);
	if(paramObj.opFlag === ZidOp.MOD)
	{
		let modifyRecord = {  
	    'value': [{
		'index': paramObj.value[0].index,
		'type': paramObj.value[0].type,
		'data': {
			'format': paramObj.value[0].data.format,
			'value': paramObj.value[0].data.value
		},
		'ttl': paramObj.value[0].ttl,
		'ttlType': paramObj.value[0].ttlType,
		'timestamp': paramObj.value[0].timestamp,
		'references': paramObj.value[0].references,
		'adminRead': paramObj.value[0].adminRead,
		'adminWrite': paramObj.value[0].adminWrite,
		'publicRead': paramObj.value[0].publicRead,
		'publicWrite': paramObj.value[0].publicWrite
		}]
		};
	
		modifyZidValue(paramObj.zid, modifyRecord);
		Chain.tlog(TLOG_ZID_MODIFY_CONTRACT_ADDRESS, 'zid data record', 'modify', paramObj.zid);
		
	}
}
/*
    ***********************************
                调用入口
    ***********************************
*/

function init(input){
	let inputObj = JSON.parse(input);
    if (inputObj.type === GHR_ZID_INIT){
        Chain.store(GHR_ZID_INIT, 'TRUE');
    }
	else if(inputObj.type === SHR_ZID_INIT)
	{
		Chain.store(SHR_ZID_INIT, 'TRUE');
	}
    else if(inputObj.type === LHS_ZID_INIT)
	{
		Chain.store(LHS_ZID_INIT, 'TRUE');
	}
    return;
}

function main(input){
    let funcList = {
        //验证者账户设置白名单
        'setAuditWhitelist' : setAuditWhitelist,
        //标识功能接口
        'createZidData' : createZidData,
        'modifyZidData' : modifyZidData,
        'removeZidData' : removeZidData
    };
    let inputObj = JSON.parse(input);
    Utils.assert(funcList.hasOwnProperty(inputObj.method) && typeof funcList[inputObj.method] === 'function', 'Cannot find func:' + inputObj.method);
    funcList[inputObj.method](inputObj.params);
}

function query(input){
    let result = {};
    let inputObj = JSON.parse(input);
    if (inputObj.method === 'queryZidRecord'){
        result = queryZidRecord(inputObj.params);
    }
    return JSON.stringify(result);
}
```

### 5.2.3 合约部署

​		运行链节点服务之后就可以部署合约到链上，生成对应的合约账户，合约账户可以后续进行合约管理，部署合约是通过调用SDK接口。

+ **GHR合约部署**

```java
// 初始化参数
String senderAddress = "did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV";
String senderPrivateKey = "priSPKnDue7AJ42gt7acy4AVaobGJtM871r1eukZ2M6eeW5LxG";
//ghr合约压缩后的代码
String payload = "'use strict';const ZidOp=				{'INS':'0','DEL':'1','MOD':'2','QUERY':'3'};const    ZID_RECORD_PRE='zidrecord';const GHR_ZID_INIT='ghr_zid_init';const TLOG_SET_WT='tlog_set_wt';";
Long initBalance = ToBaseUnit.ToUGas("0.01");

BIFContractCreateRequest request = new BIFContractCreateRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setInitBalance(initBalance);
request.setPayload(payload);
request.setInitInput("{\"type\":\"ghr_zid_init\"}");
request.setMetadata("create ghr contract");

// 调用BIFContractCreate接口
BIFContractCreateResponse response = sdk.getBIFContractService().contractCreate(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

​		其中`payload`参数即是对应合约源码压缩后的值，`input`参数里的类型填写对应的类型值即可。

​		合约部署完后返回对应的hash值，可以根据hash查询对应详细交易信息。

+ **SHR合约部署**

​		只需要将`setInitInput`的合约类型换成SHR的即可。

```java
// 初始化参数
String senderAddress = "did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV";
String senderPrivateKey = "priSPKnDue7AJ42gt7acy4AVaobGJtM871r1eukZ2M6eeW5LxG";
//shr合约压缩后的代码
String payload = "'use strict';const ZidOp=				{'INS':'0','DEL':'1','MOD':'2','QUERY':'3'};const    ZID_RECORD_PRE='zidrecord';const GHR_ZID_INIT='shr_zid_init';const TLOG_SET_WT='tlog_set_wt';";
Long initBalance = ToBaseUnit.ToUGas("0.01");

BIFContractCreateRequest request = new BIFContractCreateRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setInitBalance(initBalance);
request.setPayload(payload);
request.setInitInput("{\"type\":\"shr_zid_init\"}");
request.setMetadata("create shr contract");

// 调用BIFContractCreate接口
BIFContractCreateResponse response = sdk.getBIFContractService().contractCreate(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

​		合约部署完后返回对应的hash值，可以根据hash查询对应详细交易信息。

+ **LHS合约部署**

​		只需要修改`setInitInput`的合约类型值即可。

```java
// 初始化参数
String senderAddress = "did:bid:efVmotQW28QDtQyupnKTFvpjKQYs5bxf";
String senderPrivateKey = "priSPKnDue7AJ42gt7acy4AVaobGJtM871r1eukZ2M6eeW5LxG";
//lhs合约压缩后代码
String payload = "'use strict';const ZidOp=				{'INS':'0','DEL':'1','MOD':'2','QUERY':'3'};const    ZID_RECORD_PRE='zidrecord';const GHR_ZID_INIT='shr_zid_init';const TLOG_SET_WT='tlog_set_wt';";
Long initBalance = ToBaseUnit.ToUGas("0.01");

BIFContractCreateRequest request = new BIFContractCreateRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setInitBalance(initBalance);
request.setPayload(payload);
request.setInitInput("{\"type\":\"lhs_zid_init\"}");
request.setMetadata("create lhs contract");

// 调用BIFContractCreate接口
BIFContractCreateResponse response = sdk.getBIFContractService().contractCreate(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

​		合约部署完后返回对应的hash值，可以根据hash查询对应详细交易信息。

### 5.2.4 合约调用

+ **设置白名单**

```java
// 初始化参数
String senderAddress = "did:bid:efVmotQW28QDtQyupnKTFvpjKQYs5bxf";
String contractAddress = "did:bid:ef2gAT82SGdnhj87wQWb9suPKLbnk9NP";
String senderPrivateKey = "priSPKnDue7AJ42gt7acy4AVaobGJtM871r1eukZ2M6eeW5LxG";
Long amount = 0L;

BIFContractInvokeByGasRequest request = new BIFContractInvokeByGasRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setContractAddress(contractAddress);
request.setBIFAmount(amount);
request.setMetadata("contract set whiteList invoke");
request.setInput("{\"method\":\"setAuditWhitelist\",\"params\":{\"caller_address\": \"did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV\",\"addFlag\":true}}");

// 调用 BIFContractInvoke 接口
BIFContractInvokeByGasResponse response = sdk.getBIFContractService().contractInvoke(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

​		白名单设置和其他交易不同在于，源账户是由验证者账户触发的（必须时验证者账户否则报错），合约账户就是对应GHR，SHR以及LHS各自的合约地址，私钥就是验证者账户对应的，然后input合约函数以及参数都是GHR，SHR以及LHS对应里具体的值，然后初始化填充后调用接口发送即可将各个合约地址加入到白名单，以方便后续的管理操作。

+ **标识创建**

```java
// 初始化参数
String senderAddress = "did:bid:efVmotQW28QDtQyupnKTFvpjKQYs5bxf";
String contractAddress = "did:bid:ef2gAT82SGdnhj87wQWb9suPKLbnk9NP";
String senderPrivateKey = 	"priSPKnDue7AJ42gt7acy4AVaobGJtM871r1eukZ2M6eeW5LxG";
Long amount = 0L;

BIFContractInvokeByGasRequest request = new BIFContractInvokeByGasRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setContractAddress(contractAddress);
request.setBIFAmount(amount);
request.setMetadata("contract create zid invoke");
request.setInput("{\"method\":\"createZidData\",\"params\": {\n" +
        "\t\t\"zid\": \"88.1000\",\n" +
        "\t\t\"opFlag\": \"0\",\n" +
        "\t\t\"value\": [{\n" +
        "\t\t\t\"index\": \"1\",\n" +
        "\t\t\t\"type\": \"contract_address\",\n" +
        "\t\t\t\"data\": {\n" +
        "\t\t\t\t\"format\": \"string\",\n" +
        "\t\t\t\t\"value\": 		\"did:bid:efuEAGFPJMsojwPGKzjkkk89jky\"\n" +
        "\t\t\t},\n" +
        "\t\t\t\"ttl\": \"86400\",\n" +
        "\t\t\t\"ttlType\": \"0\",\n" +
        "\t\t\t\"timestamp\": \"0\",\n" +
        "\t\t\t\"references\": [],\n" +
        "\t\t\t\"adminRead\": \"1\",\n" +
        "\t\t\t\"adminWrite\": \"1\",\n" +
        "\t\t\t\"publicRead\": \"1\",\n" +
        "\t\t\t\"publicWrite\": \"0\"\n" +
        "\t\t}]\n" +
        "\t}}");
// 调用 BIFContractInvoke 接口
BIFContractInvokeByGasResponse response = sdk.getBIFContractService().contractInvoke(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

​		zid标识在的value数据结构字段为：

| Value格式   | 参数含义                                                     | 是否必填 | 默认值 |
| ----------- | ------------------------------------------------------------ | -------- | ------ |
| method      | 标识创建的函数名                                             | 必填项   |        |
| index       | 标识值索引(最好不要用默认的0，从1开始递增创建)               | 必填项   |        |
| type        | 标识值类型(GHR和SHR创建的都为contract_address)               | 必填项   |        |
| data        | 包含有format以及value字段如下                                | 必填项   |        |
| format      | string类型                                                   | 必填项   |        |
| value       | 标识值数据(GHR和SHR的为具体合约地址)                         |          |        |
| ttl         | 标识超时时间                                                 |          | 86400s |
| ttlType     | 标识超时类型                                                 |          | 0      |
| timestamp   | 时间戳                                                       |          | 0      |
| References  | 包含zid以及index字段如下                                     |          | 选填   |
| zid         | 引用的标识名称，引用标识和index一一对应，可以是多个，可以没有引用其他标识 |          | 空     |
| opFlag      | 标识操作标志 ，值0是创建，1是删除，2是修改，3是查询          |          | 0      |
| index       | 引用的标识索引值(所引用handle的index不能为0)                 |          | 空     |
| adminRead   | 标识管理员可读,1可读 0不可读                                 |          | 1      |
| adminWrite  | 标识管理员可写，1可写 0不可写                                |          | 1      |
| publicRead  | 标识公共可读，1可读 0不可读                                  |          | 1      |
| publicWrite | 标识公共可写，1可写 0不可写                                  |          | 0      |

​		标识创建调用接口时，每次只需修改contract_address的值为各自合约的账户地址，以及合约函数名,参数等数据。

​		创建接口执行完之后，根据生成的hash调用getTransactionInfo交易记录接口查询对应的交易是否成功情况。

+ **标识修改**

​		标识修改接口和标识创建章节中的接口和参数一样，只需要修改对应的`input`的parames中对应合约修改函数及参数以及标识value的各字段值即可。

+ **标识删除**

```java
// 初始化参数
String senderAddress = "did:bid:efVmotQW28QDtQyupnKTFvpjKQYs5bxf";
String contractAddress = "did:bid:ef2gAT82SGdnhj87wQWb9suPKLbnk9NP";
String senderPrivateKey = 	"priSPKnDue7AJ42gt7acy4AVaobGJtM871r1eukZ2M6eeW5LxG";
Long amount = 0L;

BIFContractInvokeByGasRequest request = new BIFContractInvokeByGasRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setContractAddress(contractAddress);
request.setBIFAmount(amount);
request.setMetadata("contract remove zid invoke");
request.setInput("{\"method\":\"removeZidData\",\"params\":	{\"zid\":\"88.1000\",\"type\":\"contract_address\",\"opFlag\": \"1\"}}");
// 调用 BIFContractInvoke 接口
BIFContractInvokeByGasResponse response = sdk.getBIFContractService().contractInvoke(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

​		标识删除接口与标识创建的核心参数基本相同，差异在input合约调用参数里，method字段为删除操作的合约函数名，params里是要删除的zid标识名，type标识类型（GHR, SHR是contract_address, LHS的是URL等类型）以及opFlag值（此处为1删除标志）。

### 5.2.5 合约查询

+ **标识查询**

​		Input合约参数说明：

​		method：对应合约的查询接口函数名（GHR的查询接口是queryZidRecord，SHR合约的是queryZidRecordShr，LHS的是queryZidRecordLhs）。

​		params：含有type类型参数(GHR以及SHR的是contract_address，LHS可以是URL或其他自定义类型)，以及对应zid名。

```java
// 初始化请求参数
String contractAddress = "did:bid:ef2gAT82SGdnhj87wQWb9suPKLbnk9NP";
BIFContractCallRequest request = new BIFContractCallRequest();
request.setContractAddress(contractAddress);
//标识查询合约接口操作
request.setInput("{\"method\":\"queryZidRecord\",\"params\":{\"type\": \"contract_address\",\"zid\":\"88.1000\"}}");

// 调用contractQuery接口
BIFContractCallResponse response = sdk.getBIFContractService().contractQuery(request);
if (response.getErrorCode() == 0) {
    BIFContractCallResult result = response.getResult();
    System.out.println(JSON.toJSONString(result, true));
} else {
    System.out.println("error: " + response.getErrorDesc());
}
```

​		返回信息：

```json
{
    "query_rets":[
        {
            "result":{
                "type":"string",
                "value":"{\"value\":[{\"index\":\"1\",\"type\":\"contract_address\",\"data\":{\"format\":\"string\",\"value\":\"did:bid:efuEAGFPJMsojwPGKzjkkk89888\"},\"ttl\":\"86400\",\"ttlType\":\"0\",\"timestamp\":\"0\",\"references\":[],\"adminRead\":\"1\",\"adminWrite\":\"1\",\"publicRead\":\"1\",\"publicWrite\":\"1\"}]}"
            }
        }
    ]
}
```

+ **查询账户信息**

​		标识合约操作的过程中可能需要查询账户详细信息，所以需要此接口。

```java
// 待查询的账户地址
String accountAddress = "did:bid:efVmotQW28QDtQyupnKTFvpjKQYs5bxf";
BIFAccountGetInfoRequest request = new BIFAccountGetInfoRequest();
request.setAddress(accountAddress);

// 调用getAccount接口
BIFAccountGetInfoResponse response = sdk.getBIFAccountService().getAccount(request);

if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error: " + response.getErrorDesc());
}
```

+ **查询交易信息**

​		标识合约创建，修改等接口操作后，需要根据生成的hash查询交易是否成功等信息，故需要此接口查询。

```java
// 初始化请求参数
String txHash = "1653f54fbba1134f7e35acee49592a7c29384da10f2f629c9a214f6e54747705";
BIFTransactionGetInfoRequest request = new BIFTransactionGetInfoRequest();
request.setHash(txHash);

// 调用getBIFInfo接口
BIFTransactionGetInfoResponse response = sdk.getBIFTransactionService().getTransactionInfo(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error: " + response.getErrorDesc());
}
```

​		返回信息：

```json
{
	"total_count":1,
	"transactions":[{
		"actual_fee":"477",
		"close_time":1630760729249135,
		"contract_tx_hashes": ["82411f9cd3a1fd820285a426bf43142925273b2e9522dc836ad7185f548500a0"],
		"error_code":0,
		"error_desc":"",
		"hash":"b4495e6c124ea6d4c5c7094a687b8dd73bf85f7ac5f4805577c27fad56af912c",
		"ledger_seq":22656,
		"signatures":[{
     	"public_key":"b0656669660116b0b2ddfbba3f155962b311dc0afb671e2da069563a70fccabd9ff8c4",
            "sign_data":"7bfaa7c625619ddfd9a73c945f3231a21583c75b0d30b0c814dea43debace471edfb2485168429cadae30411b67996331e088ee9ae9636b593693f21c951b902"
        }],
        "transaction":{
			"fee_limit":1000000,
			"gas_price":1,
			"metadata":"contract ",
			"nonce":55,
			"operations":[{
				"pay_coin":{
					"amount":1000,
					"dest_address":"did:bid:efw8T4pMic9goJHe2FCNocaCb8AkDr3P",
					"input":"{\"method\":\"removeZidData\",\"params\":{\"zid\":\"88.1000\",\"type\":\"contract_address\",\"opFlag\": \"1\"}}"
				},
				"type":7
			}],
			"source_address":"did:bid:efuEAGFPJMsojwPGKzjD8vZX1wbaUrVV"
		},
		"tx_size":353
	}]
}
```

## 5.3 SQL智能合约

​		本节描述通过星火链网实现并部署SQL智能合约。

​		SQL智能合约基于JavaScript语言编写。合约支持SQL相关语法，开发者通过在合约中使用SQL语句，达到操作数据库的目的，使合约开发人员可以更高效、更易用地对数据进行管理和使用。

### 5.3.1 准备工作

​		使用星火链支持SQL智能合约功能的版本，需完整部署星火链才可使用。本地或远程服务器上有可用的、拥有root权限的MySQL服务器。

+ **准备环境**

  星火链配置文件中默认不包含SQL相关配置文件，使用前需修改每个节点的配置文件**config/bif.json**，配置MySQL服务器相关字段。

  在配置文件中`"db"`字段添加`"rational_string"`即可。

+ **配置字段说明**

| 字段名          | 描述                                                        |
| --------------- | ----------------------------------------------------------- |
| hostname        | MySQL服务器ip地址                                           |
| user            | MySQL服务器的用户，需具有创建数据库、创建表、创建索引等权限 |
| password_crypto | MySQL服务器的用户对应的密码，加密后的哈希值。               |
| port            | MySQL服务器端口                                             |

+ **配置文件示例**

```json
{
    "db": {
        "ledger_path": "data/ledger.db", 
        "account_path": "data/account.db", 
        "privatetx_path": "data/private.db", 
        "keyvalue_path": "data/keyvalue.db",
        "rational_string": "hostname=127.0.0.1 user=root password_crypto=143e97ddcf387b0b06f68dbaa4a23bed port=3306"
    }
}
```

### 5.3.2 合约说明

​		本合约实现了一个学生数据管理系统，合约支持将学生的基本信息通过SQL语句保存在执行节点的MySQL服务器数据库中。

​		通过合约接口，合约实现了对学生信息的添加、删除、更新和查询，并增加了一个修正学生成绩的应用场景，使SQL语句的使用更加丰富多样。

+ **合约接口说明**

| 接口          | 描述         |
| ------------- | ------------ |
| addStudent    | 添加学生信息 |
| delStudent    | 删除学生信息 |
| updateStudent | 更新学生信息 |
| finalScore    | 修正学生成绩 |
| queryStudent  | 查询学生信息 |

+ **合约文件**

```js
'use strict';

//合约初始化入口
function init(input) {
    //SQL初始化、链接，创建数据库
    let result = Chain.initCreateContractSql();
    Utils.assert(result === true, `init db failed(${result})`);
    //创建数据表
    let createTable = 'create table student (id varchar(128) primary key, teacher_id varchar(128), name varchar(64) DEFAULT \'\', age int DEFAULT 0, score int DEFAULT 0)';
    return Chain.executeDDL(createTable);
}

//功能函数，添加学生信息
function addStudent(params) {
    let studentInfo = params;
    Utils.assert(studentInfo.id !== undefined, 'The id is not existed');
    //执行DML语句
    let insertSql = 'insert into student values (\''
                    + studentInfo.id + '\', \''
                    + studentInfo.teacher_id + '\', \''
                    + studentInfo.name + '\', \''
                    + studentInfo.age + '\', \''
                    + studentInfo.score + '\')';
    let result = Chain.executeDML(insertSql);
    Utils.assert(result === true, 'add student failed');
}

//功能函数，删除学生信息
function delStudent(params) {
    let studentInfo = params;
    Utils.assert(studentInfo.id !== undefined, 'The id is not existed');
    let deleteSql = 'delete from student where id = \'' + studentInfo.id + '\'';
    let result = Chain.executeDML(deleteSql);
    Utils.assert(result === true, 'delete student failed');
}

//功能函数，更新学生信息
function updateStudent(params) {
    let studentInfo = params;
    Utils.assert(studentInfo.id !== undefined, 'The id is not existed');
    Utils.assert(studentInfo.teacher_id !== undefined
              || studentInfo.name !== undefined
              || studentInfo.age !== undefined
              || studentInfo.score !== undefined, 'The update data is not existed');
    let updateSql = 'update student SET';
    if (studentInfo.teacher_id !== undefined) {
        updateSql += ' teacher_id=' + studentInfo.teacher_id;
    }
    if (studentInfo.name !== undefined) {
        updateSql += ' name=' + studentInfo.name;
    }
    if (studentInfo.age !== undefined) {
        updateSql += ' age=' + studentInfo.age;
    }
    if (studentInfo.score !== undefined) {
        updateSql += ' score=' + studentInfo.score;
    }
    updateSql += ' where id=' + studentInfo.id + ';';
    let result = Chain.executeDML(updateSql);
    Utils.assert(result === true, 'update student failed');
}

//功能函数，修正学生成绩
function finalScore(params) {
    let studentInfo = params;
    Utils.assert(studentInfo.targetScore !== undefined, 'The target score is not existed');
    let querySql = 'select * from student where score < ' + studentInfo.targetScore;
    let result = Chain.executeDQL(querySql);
    Utils.assert(result !== false, 'query student score failed');
    Chain.tlog('queryFinalScoreInfo', result);
    let updateSql = 'update student SET score = ' + studentInfo.targetScore + ' where score < ' + studentInfo.targetScore;
    result = Chain.executeDML(updateSql);
    Utils.assert(result === true, 'update student score failed');
}

//合约交易入口
function main(input) {
    //SQL初始化、链接，打开数据库
    let result = Chain.initQueryContractSql();
    Utils.assert(result === true, `init db failed(${result})`);
    let inputObj = JSON.parse(input);
    if (inputObj.method === 'addStudent') {
        addStudent(inputObj.params);
    }
    else if (inputObj.method === 'delStudent') {
        delStudent(inputObj.params);
    }
    else if (inputObj.method === 'updateStudent') {
        updateStudent(inputObj.params);
    }
    else if (inputObj.method === 'finalScore') {
        finalScore(inputObj.params);
    }
    else {
        throw '<Main interface passes an invalid operation type>';
    }
}

//功能函数，查询学生信息
function queryStudent(params) {
    let studentInfo = params;
    let querySql = '';
    if (studentInfo.id !== undefined) {
        querySql = 'select * from student where id = \'' + studentInfo.id + '\'';
    }
    else if (studentInfo.name !== undefined) {
        querySql = 'select * from student where name = \'' + studentInfo.name + '\'';
    }
    else {
        querySql = 'select * from student';
    }
    let result = Chain.executeDQL(querySql);
    Utils.assert(result !== false, 'query student failed');
    Utils.log('queryInfo:' + result);
    return result;
}

//合约查询入口
function query(input) {
    let result = Chain.initQueryContractSql();
    Utils.assert(result === true, `init db failed(${result})`);
    let inputObj = JSON.parse(input);
    if (inputObj.method === 'queryStudent') {
        let queryInfo = queryStudent(inputObj.params);
        let queryJson = JSON.parse(queryInfo);
        return 'test quert sql, student score:' + queryJson.query[0].score;
    }
    else {
        throw '<Query interface passes an invalid operation type>';
    }
}
```

### 5.3.3 合约部署

```java
// 初始化参数
String senderAddress = "did:bid:efCB2YndSDRHrMF7mTbGVkpbatAdybKW";
String senderPrivateKey = "priSPKkPgbe2qsqtZSpkP4jpwxgAtpD4SAdqZzLTya7eBfa5NV";
String payload = "SQL合约转义压缩后的代码";
Long initBalance = ToBaseUnit.ToUGas("0.01");

BIFContractCreateRequest request = new BIFContractCreateRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setInitBalance(initBalance);
request.setPayload(payload);
request.setMetadata("create sql contract");
request.setType(1);
request.setInitInput("");
// 调用BIFContractCreate接口
BIFContractCreateResponse response = sdk.getBIFContractService().contractCreate(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

### 5.3.4 合约调用

```java
// 初始化参数
String senderAddress = "did:bid:efh5RjCeR96pvA9p6bBEx7QMKyzedxEK";
String contractAddress = "did:bid:efMfNGcRct9ao3jUT5huS9K6dNLcU9Tq";
String senderPrivateKey = "priSPKmcKMEknDZ2hr5BEkHMXiXAqrUTqMxXoHLYJzSrkHqGso";
Long amount = 0L;

BIFContractInvokeByGasRequest request = new BIFContractInvokeByGasRequest();
request.setSenderAddress(senderAddress);
request.setPrivateKey(senderPrivateKey);
request.setContractAddress(contractAddress);
request.setBIFAmount(amount);
request.setMetadata("contract addStudent");
request.setInput("{\"method\":\"addStudent\",\"params\":{\"id\": \"1\", \"teacher_id\":\"1\", \"name\":\"xxx\", \"age\": \"18\", \"score\":\"85\"}}");

// 调用 BIFContractInvoke 接口
BIFContractInvokeByGasResponse response = sdk.getBIFContractService().contractInvoke(request);
if (response.getErrorCode() == 0) {
    System.out.println(JSON.toJSONString(response.getResult(), true));
} else {
    System.out.println("error:      " + response.getErrorDesc());
}
```

### 5.3.5 合约查询

```java
String contractAddress = "did:bid:efMfNGcRct9ao3jUT5huS9K6dNLcU9Tq";

// Init request
BIFContractCallRequest request = new BIFContractCallRequest();
request.setContractAddress(contractAddress);
request.setInput("{\"method\":\"queryStudent\",\"params\":{\"id\":\"1\"}}");

// Call call
BIFContractCallResponse response = sdk.getBIFContractService().contractQuery(request);
if (response.getErrorCode() == 0) {
    BIFContractCallResult result = response.getResult();
    System.out.println(JSON.toJSONString(result, true));
} else {
    System.out.println("error: " + response.getErrorDesc());
}
```

​		返回结果：

```json
{
    "query_rets": [
        {
            "result": {
                "type": "string",
                "value": "test quert sql, student score:85"
            }
        }
    ]
}
```
