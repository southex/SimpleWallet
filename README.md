# SimpleWallet 协议文档 （意见征集稿）

版本：beta 1.0.4

最后更新：2018.8.13 18:25


## 简介
SimpleWallet是一个EOS钱包和Dapp的通用对接协议。

目前EOS的钱包应用众多、Dapp也在快速发展中，在实际对接过程中，各方标准不统一，对接耗时耗力。

遵照此协议，可以减少各方开发适配工作，低耦合的实现钱包对Dapp进行登录授权和支付。


## 功能列表
- 登录
1. 场景1：钱包App扫二维码进行登录，适用于WEB版Dapp
2. 场景2：Dapp的移动端APP拉起钱包APP，请求登录授权
3. 场景3：钱包APP内嵌Dapp的H5页面，进行登录（暂无）

- 支付
1. 场景1：钱包扫码支付，适用于WEB版Dapp
2. 场景2：Dapp的移动端拉起钱包APP请求支付授权
3. 场景3：钱包APP内嵌Dapp的H5页面，进行支付（暂无）

## 协议内容

### 1. 钱包APP在系统注册拦截协议

钱包APP应在操作系统内注册拦截协议（URL Scheme、appLink），以便Dapp的APP拉起钱包应用。

拦截协议为：simplewallet://eos.io

Dapp的移动端应用可以调用此协议，传递数据给钱包APP，传递数据的请求格式为：

simplewallet://eos.io?param={json数据}

### 2. 登录
 

#### 场景1：使用钱包扫码二维码登录
> 	适合Dapp的网站接入。
> 
> 业务流程图如下：

![image](http://on-img.com/chart_image/5b658d5de4b0be50eacf8f0c.png)

- 钱包扫描dapp web提供的登录二维码，此二维码的数据格式为json，包含以下数据：
```
// 登录的二维码数据格式
{
    protocol	string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version     string   // 协议版本信息，如1.0
    dappName    string   // Dapp名字
    dappIcon    string   // Dapp图标 
    action      string   // 赋值为login
    uuID        string   // Dapp server生成的，用于此次登录验证的唯一标识   
    loginUrl    string   // Dapp server上用于登录验证信息的url
    expire	number   // 二维码过期时间，unix时间戳
    loginMemo	string   // 登录备注信息，可选
}
```
- 钱包对登录相关数据进行签名
```
// 生成sign算法
let data = timestamp + account + uuID + ref     //ref为钱包名，标示来源
sign = ecc.sign(data, privateKey)
```
- 钱包将签名后的数据POST到Dapp提供的loginUrl，请求登录验证
```
 // 请求登录验证的数据格式
{
    protocol   string     // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version    string     // 协议版本信息，如1.0
    timestamp  number     // 当前UNIX时间戳
    sign       string     // eos签名
    uuID       string     // Dapp server生成的，用于此次登录验证的唯一标识     
    account    string     // eos账户名
    ref        string     // 来源,如钱包名
}
```
- Dapp server收到数据数，验证sign签名数据，返回success == true或false
  
```
// 请求登录返回数据格式
{
    success    bool
}

```
#### 场景2：Dapp的移动端应用拉起钱包App，请求登录授权
> 	适合Dapp的移动端(iOS或安卓端）接入。业务流程图如下：

![image](http://on-img.com/chart_image/5b6591fbe4b0edb750f9a364.png)
- Dapp的移动端拉起钱包APP要求登录授权，提供给钱包App的数据格式为json,包含以下信息：
```
// 传递给钱包APP的数据包结构
{
    protocol	string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version     string   // 协议版本信息，如1.0
    dappName    string   // Dapp名字，用于在钱包APP中展示
    dappIcon    string   // Dapp图标Url，用于在钱包APP中展示
    action      string   // 赋值为login
    uuID        string   // 用于Dapp登录验证唯一标识   
    loginUrl    string   // Dapp server生成的，用于此次登录验证的URL 
    appKey      string   // 钱包回调拉起Dapp移动端的app标识
    loginMemo	string   // 登录备注信息，可选
    callbackUrl string   // 用户完成操作后，钱包回调拉起Dapp移动端APP的回调URL,如appABC://abc.com，可选
    		         // 钱包回调时在此URL后加上操作结果，建议格式：appABC://abc.com?action=login&result=1, 
			 // action的值为login/transfer，result的值为：0为用户取消，1为成功,  2为失败
}
```
- 之后的流程和上面的扫码登录过程相同



### 3. 支付
#### 场景1：钱包扫描二维码进行支付
> 业务流程图如下:

![image](http://on-img.com/chart_image/5b6594bae4b053a09c24fa9a.png)

```
// 扫描的支付二维码数据格式
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    	version     string   // 协议版本信息，如1.0
	dappName    string   // Dapp名字，用于在钱包APP中展示，可选
    	dappIcon    string   // Dapp图标Url，用于在钱包APP中展示，可选
	action      string   // 支付时，赋值为transfer，必须
	from        string   // 付款人的EOS账号，可选
	to          string   // 收款人的EOS账号，必须
	amount      number   // 转账数量，必须
	contract    string   // 转账的token所属的contract账号名，必须
	symbol      string   // 转账的token名称，必须
	precision   number   // 转账的token的精度，小数点后面的位数，必须
	dappData    string   // 由Dapp生成的业务信息，此业务信息需要钱包在转账时附加在memo中发出去
			     // 钱包转账时的memo信息，格式为 dappData=xxxxxxx&ref=walletname
			     // dapp收到转账后,用dappData来关联自己的业务逻辑，用ref标示来区分来源
	expire	    number   // 二维码过期时间，unix时间戳
	info {               // 此笔转账交易的业务附加信息，可选项，仅用于钱包展示，方便用户识别，如下为示例
		orderID number      // 订单
		side number         // 0 卖单 1 买单
		limitType number    // 0 限价 1 市价
		price string	    // 价格 1.0001 EOS
		stockAmount string  // 10.0000 BTC
	}	
	
}
```
- 钱包组装上述数据，生成一笔EOS的transaction，用户授权此笔转账后，提交转账数据到EOS主网
- Dapp需自行监控EOS主网，检查代币是否到账
- 对于流行币种如IQ，如果二维码中给出的contract名和官方的合约名不一致，钱包方要提醒用户，做二次确认


#### 场景2：Dapp的移动端拉起钱包App，请求支付授权
> 业务流程图如下：

![image](http://on-img.com/chart_image/5b659391e4b0f8477da3138b.png)
```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	action      string   // 支付时，赋值为transfer
	dappName    string   // Dapp名字，用于在钱包APP中展示，可选
    	dappIcon    string   // Dapp图标Url，用于在钱包APP中展示，可选	
	from        string   // 付款人的EOS账号，可选
	to          string   // 收款人的EOS账号，必须
	amount      number   // 转账数量，必须
	contract    string   // 转账的token所属的contract账号名	
	symbol      string   // 转账的token名称，必须
	precision   number   // 转账的token的精度，小数点后面的位数，必须	
	dappData    string   // 由Dapp生成的业务信息，此业务信息需要钱包在转账时附加在memo中发出去
			     // 钱包转账时的memo信息，格式为 dappData=xxxxxxx&ref=walletname
			     // dapp收到转账后,用dappData来关联自己的业务逻辑，用ref标示来区分来源
    	callbackUrl string   // 用户完成操作后，钱包回调拉起Dapp移动端APP的回调URL,如appABC://abc.com，可选
    		             // 钱包回调时在此URL后加上操作结果，建议格式：appABC://abc.com?action=login&result=1, 
			     // action的值为login/transfer，result的值为：0为用户取消，1为成功,  2为失败
	info {               // 此笔转账交易的业务附加信息，可选项，仅用于钱包展示，方便用户识别，如下为示例
		orderID number      // 订单
		side number         // 0 卖单 1 买单
		limitType number    // 0 限价 1 市价
		price string	    // 价格 1.0001 EOS
		stockAmount string  // 10.0000 BTC
	}			     
}
```
- 钱包组装上述数据，生成一笔EOS的transaction，用户授权此笔转账后，提交转账数据到EOS主网；如果有callbackUrl，则回调拉起dapp的应用
- Dapp需自行监控EOS主网，检查代币是否到账
- 对于流行币种如IQ，如果二维码中给出的contract名和官方的合约名不一致，钱包方要提醒用户，做二次确认

### 错误处理
- code不等于0则请求失败
```
// 错误返回 
{
    code number     //错误符，等于0是成功，大于0说明请求失败，dapp返回具体的错误码
    message string  //返回的提示信息
}
```

