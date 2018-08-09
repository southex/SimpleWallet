# SimpleWallet 协议文档 V1.0


SimpleWallet是一个EOS钱包和Dapp的通用对接协议。

目前EOS的钱包应用众多、Dapp也在快速发展中，在实际对接过程中，各方标准不统一，对接耗时耗力。

遵照此协议，可以减少各方开发适配工作，低耦合的实现钱包对Dapp进行登录授权和支付。


# 功能列表
- 登录
1. 场景1：钱包App扫二维码进行登录，适用于WEB版Dapp
2. 场景2：Dapp的移动端APP拉起钱包APP，请求登录授权
3. 场景3：钱包APP内嵌Dapp的H5页面，进行登录（暂无）

- 支付
1. 场景1：钱包扫码支付，适用于WEB版Dapp
2. 场景2：Dapp的移动端拉起钱包APP请求支付授权
3. 场景3：钱包APP内嵌Dapp的H5页面，进行支付（暂无）

# 协议内容

### 登录


#### 场景1：使用钱包扫码二维码登录
> 	适合Dapp的网站接入。
> 
> 业务流程图如下：

![image](http://on-img.com/chart_image/5b658d5de4b0be50eacf8f0c.png)

- 钱包扫描dapp web提供的登录二维码，此二维码的数据格式为json，包含以下数据：
```
// 扫码登录的数据格式
{
    version     string     // SimpleWallet的版本信息，如1.0
    appName     string   // Dapp名字
    appIcon     string   // Dapp图标 
    action      string   // 赋值为login
    uuID        string   // Dapp server生成的，用于此次登录验证的唯一标识   
    postUrl     string   // Dapp server上用于登录验证信息的url
}
```
- 钱包对登录相关数据进行签名
```
// 生成sign算法
let data = timestamp + account + uuID
sign = ecc.sign(data, privateKey)
```
- 钱包将签名后的数据POST到Dapp提供的postUrl请求登录验证
```
 // 请求登录验证的数据格式
{
    version    string     // SimpleWallet的版本信息，如1.0
    timestamp  number     // 时间戳
    sign       string     // eos签名
    uuID       string     // Dapp server生成的，用于此次登录验证的唯一标识     
    account    string     // eos账户名
    from       string     // 来源,如钱包名
}
```
- Dapp server收到登录验证数据，验证通过后，返回success == true或false
  
```
// 请求登录返回数据格式
{
    success    bool
}

```
#### 场景2：Dapp的移动端应用拉起钱包App，请求登录授权
> 	适合Dapp的移动端(iOS或安卓端）接入。业务流程图如下：

![image](http://on-img.com/chart_image/5b6591fbe4b0edb750f9a364.png)
- Dapp的移动端拉起钱包要求登录授权，提供给钱包App的数据格式为json,包含以下信息：
```
{
    version     string   // SimpleWallet的版本信息，如1.0
    appName     string   // Dapp名字
    appIcon     string   // Dapp图标Url
    action      string   // 赋值为login
    uuID        string   // 用于Dapp登录验证唯一标识   
    postUrl     string   // Dapp server生成的，用于此次登录验证的唯一标识   
    appKey      string   // 钱包回调拉起Dapp移动端的app标识
}
```
- 之后的流程和上面的扫码登录过程相同

### 支付
#### 场景1：钱包扫描二维码进行支付
> 业务流程图如下:

![image](http://on-img.com/chart_image/5b6594bae4b053a09c24fa9a.png)

```
// 支付通用的数据格式
{
	version     string   // SimpleWallet的版本信息，如1.0
	action      string   // 支付时，赋值为transfer
	subaction   string   // ？？？交易下单为putorder, 此时info中的数据格式如下
	from        string   // 付款人的EOS账号，可选
	to          string   // 收款人的EOS账号，必须
	amount      number     // 转账数量，必须
	symbol      string     // 转账的token名称，必须
	memo        string       // 转账memo，可选
	info {            // 此笔交易的业务附加信息，如下为一个交易所订单的示例
		orderID number      // 订单
		side number         // 0 卖单 1 买单
		limitType number    // 0 限价 1 市价
		price string	    // 价格 1.0001 EOS
		stockAmount string  // 10.0000 BTC
	}
}
```
- 钱包使用上述数据生成一笔EOS的transaction，用户授权此笔转账后，提交转账数据到EOS主网


#### 场景2：Dapp的移动端拉起钱包App，请求支付授权
> 业务流程图如下：

![image](http://on-img.com/chart_image/5b659391e4b0f8477da3138b.png)
```
// 支付通用的数据格式
{
	version     string   // SimpleWallet的版本信息，如1.0
	action      string   // 支付时，赋值为transfer
	subaction   string   // ？？？交易下单为putorder, 此时info中的数据格式如下
	from        string   // 付款人的EOS账号，可选
	to          string   // 收款人的EOS账号，必须
	amount      number     // 转账数量，必须
	symbol      string     // 转账的token名称，必须
	memo        string       // 转账memo，可选
	info {            // 此笔交易的业务附加信息，如下为一个交易所订单的示例
		orderID number      // 订单
		side number         // 0 卖单 1 买单
		limitType number    // 0 限价 1 市价
		price string	    // 价格 1.0001 EOS
		stockAmount string  // 10.0000 BTC
	}
    appKey      string   // 钱包回调拉起Dapp移动端的app标识
}
```
- 钱包使用上述数据生成一笔EOS的transaction，用户授权此笔转账后，提交转账数据到EOS主网


## 错误处理
- code不等于0则请求失败
```
// 错误返回 
{
    code number
    message string
}
```

