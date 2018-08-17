## **消息授权（临时稿）**
---
由dapp发起一个交易，内部包含复数action，在钱包中由用户同意授权后签名并发送到主网。

参数名 | 值类型|是否必须|简介
----- |----|----|----
protocol|   string| YES|    协议名
version|    string| YES|    协议版本
action|     string| YES|    应为 **transactions**
dappName|   string| YES|    用于钱包中展示的dapp名称
dappIcon|   string| YES|    用于钱包中展示的dapp图标
dataType|   number| NO|     data参数所提供的数据类型
data|       string| YES|    dataType=0（JSON模式）   dataType=1（URL模式）
callback|   string| NO|     回调uri，推送结果txId

## **参数详解**
---
## dataType
0 = JSON模式，在data中提供一段json，用于data内容长度不多的情况  
1 = URL模式，在data中提供URL，钱包方get该url获取用于签名的json数据，用于data数据过长时使用

## data
json格式数据或URL  
使用eos标准的json​格式发起请求  
如data格式中任意action包含**authorization**字段：  
钱包方应使用该账户权限签名，如钱包内并没有该账户权限，应中止本次签名
```json
[
  {
    "data": {
      "from": "mastermisaka",
      "to": "zhoujingjing",
      "quantity": "0.5000 ADD",
      "memo": ""
    },
    "code": "eosadddddddd",
    "type": "transfer",
    "authorization": [
      {
        "actor": "mastermisaka",
        "permission": "active"
      }
    ],
    "ricardian": ""
  }
]
```

如data格式中所有action都不包含**authorization**字段：  
钱包用户来决定使用哪个账户权限进行签名，并对data数据支持"{@accountName}"替换符，替换为用户选中的账户名
```json
[
  {
    "data": {
      "from": "{@accountName}",
      "to": "zhoujingjing",
      "quantity": "0.5000 ADD",
      "memo": ""
    },
    "code": "eosadddddddd",
    "type": "transfer",
    "ricardian": ""
  }
]
```

## callback
在将交易签名后发送到主网后，回调的URL地址，DAPP使用POST的方式调用该URL并传递数据  
如协议头为http或https，则执行该方案


>**关于为什么需要callback**  
用于在dapp无需自己搭建节点的情况下，能快速查询指定交易结果，无需搭建节点轮询。  
另外，及时的返回数据利于dapp在得知结果时，在交互层给与更好的用户体验。​​

例子：https://eossw.io/callback

**POST参数：**
参数 | 类型 | 说明
----|----|----|----
result| number  | 0=用户取消，1=成功，>1=失败
txid|   string  | txid 如：ca785f554be18b82007047ec1a7a78b0561ecfc2fbb05429264aae14c8620910
errorMessage| string | 主网返回的错误信息，在result>1时有内容，如：insufficient staked net bandwidth 
