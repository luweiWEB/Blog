## 五步理解JSON Web Token(JWT)



> 原文链接[5 Easy Steps to Understanding JSON Web Tokens (JWT)](https://medium.com/vandium-software/5-easy-steps-to-understanding-json-web-tokens-jwt-1164c0adfcec)

​    在这篇文章中我们讲会讲到：  JSON Web Token(JWT)的基本原理是什么， 以及我们为什么要使用它？JWT是确保我们应用程序可信和安全的重要的一环。JWT 允许我们以安全的方式去表示像用户数据之类的声明。



​    为了解释JWT 如何工作，我们从一个抽象的定义开始。



>  JSON Web Token  是一个JSON 对象。在 [REC 7519](https://tools.ietf.org/html/rfc7519) 中这么定义：它表示双方的一组安全的信息方式。它由 头部（header）， 负载(payload) 和签名(signature)组成。

简单来说， JWT 是一个具有下面形式的字符串，

```JSON
header.payload.sinagure
```



我们应该注意的是，带有双引号（“”）的字符串才被认为是一个有效的 JSON 对象

  

为了展示我们如何以及为什么使用它，我将使用三个简单的实体实例（请看下图）

三个实例分别为用户，应用服务器， 以及授权认证服务器。 授权认证服务器将会向用户提供JWT, 有了 JWT 用户可以安全的与应用程序服务器通信。



![diagram](../image/example.png)

在上面这个例子中，用户首先使用认证服务器的的登陆系统登陆认证服务器。（比如： 用户名， 密码，脸书（Facebook）,谷歌（Google）账号登陆）。然后授权认证服务器创建JWT, 并将其发给用户。当用户与应用程序之间有API调用的时候，在此过程中用户会传递JWT。 在这一步中， 应用程序服务器将会验证传过来的JWT 是否为认证服务器创建的。（更详细验证过程将会在后面介绍）。因此， 当用户与应用程序之间使用携带的JWT 进行API 调用的时候，应用程序可以使用JWT 来验证API 调用是否来自经过身份验证的用户。

接下来，我们将更深入的研究 JWT 的构造以及其验证方式。





### 步骤1. 创建 HEADER (头部)
----

JWT 的的头部包含有关如何计算JWT 签名的的信息。 头部是像下面格式的JSON 对象

```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

在这个JSON 对象中。 typ 键 的值表示对象是 `JWT`, `alg` 的值表示用于创建JWT 签名的哈希算法。在示例中， 我们用 HMAC-SHA256 算法，（一种使用密钥的哈希算法）来生成签名.（这个我们会在第三步中更详细的讨论）




### 步骤2. 创建PAYLOAD(负载)
----

负载也是JWT 的一部分，用来存放JWT 里面的数据（该数据也被称为JWT的 '声明' ）。 在我们的例子中， 身份认证服务器会创建一个JWT,并把用户的信息存储在里面， 尤其是用户ID.

```json
{
    "userId": "b08f86af-35da-48f2-8fab-cef3904660bd "
}
```

在我们的例子中， 我们只放了一个字段在负载中。其实你可以根据你都需要添加任何数量的字段。

JWT 有几个标准的官方字段像： “iss”（issure）：签发人，"sub"(subject)：主题 和“exp”（expiration time）:过期时间 .这些字段在创建JWT 的时候很有用， 但是他们是可选的。 更多有关JWT 标准字段列表， 请参考 [维基百科](https://en.wikipedia.org/wiki/JSON_Web_Token#Standard_fields)

我们要记住，数据的大小将会影响JWT 的整体大小，这通常都不是问题， 但是过大的JWT 可能会产生负面影响，并导致延迟。



### 步骤3. 创建SINATURE(签名)
----

签名用下面的伪代码实现：

```js
// 签名算法
data = base64urlEncode（header）+“。”+ base64urlEncode（payload）
hashedData = hash（data，secret）
signature = base64urlEncode（hashedData）
```



该算法所做的是，用在第一步和第二步中创建的header 和payload 进行 base64url 编码。然后将该算法得到的编码字符串在他们之间用'.'链接起来 。 在我们的伪代码中，链接起来的字符串会放在数据中。这个字符串用于JWT 头部中指定哈希算法的密钥。生成的散列数据将会分配给hashData. 然后对 hashData数据进行base64url 编码以产生JWT 签名。

在示例中， 头部和负载都是像下面这样经过base64url 编码的格式：

```json
// header 
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9 
// payload
eyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYZOTA0NjYwYmQifQ
```

 然后，对周期性编码的header和payload 应用带有指定密钥的签名算法。 在我们的例子中，这表示在数据上使用 Header 里面指定的HS256算法， 并将密钥设置为 字符串'secret'以获取hashData字符串。 在hashData 经过base64url 编码后我们得到以下JWT签名:

```js
// signature
-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM
```

### 步骤4. 将JWT 的三个部分组合在一起。
---

至此我们已经生成了JWT 的三个组成部分。 记住JWT 的结构 **header.payload.signature**

我们只需要简单的把这三部分用 `'.'`连接起来就行。我们使用上面的 经过 base64url 编码得到的header， payload 以及第三步得到的signature（签名）组成下面的 JWT

```js
// JWT Token 
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYzOTA0NjYwYmQifQ.-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM
```

你可以尝试在浏览器上通过[jwt.io](http://jwt.io/)来创建自己的JWT

回到我们的例子，现在授权服务器可以把JWT 发送给用户了。



### JWT是如何保证我们的数据安全的。

我们重要 的是要理解我们使用JWT的目的不是为了以任何方式隐藏数据。 使用它的目的是为了验证我们接收到的数据来自真实的数据源。

正如我们前面步骤所示，JWT 里面的数据是经过编码和签名的，而不是加密的。编码数据的目的是为了转化数据结构。签名数据允许数据接收者验证数据远的真实性。因此，编码和签名数据不会保护数据。 另一方面，加密的目的主要是为了保护数据以防止未授权的访问。有关编码的和加密之间的差异的更详细的说明，以及散列的是如何工作的请参阅此[文章](https://danielmiessler.com/study/encoding-encryption-hashing-obfuscation/#encoding)

> **由于 JWT 只是被编码以及签名，并且未加密，因此 JWT并不保证敏感数据的安全性**

### 步骤5. 验证JWT
----

在上面简单的三个实体案例中，我们使用的是由HS256 算法签名的JWT,其中只有认证服务器和应用服务器知道密钥。当应用程序设置认证过程的时候，它接收到来自授权服务器的的密钥。因为应用程序服务器知道密钥，当用户对应用程序进行有JWT 的API调用的时候，应用程序可以使用与步骤3中相同的签名算法。然后应用程序服务器可以验证从其自己的哈希操作中获得的签名是否与JWT本身上的签名匹配。（即，它是否与身份验证服务器创建的JWT签名匹配）。如果签名匹配，则表示收到的JWT 有效，表示API 调用来自可信源。除此之外，如果签名不匹配，则表示收到的JWT 无效，这可能表示应用程序可能受到了潜在的攻击。 因此 ，通过验证JWT, 应用程序在自身和用户之间添加了一层信任。



#### 总结

我们了解了什么是JWT , 以及如何创建和验证他们，如何使用它们来确保应用程序与其用户之前的信任。这是理解JWT 基础知识以及它有用的开始。JWT 只是确保在你的应用程序中的信任和安全性的难题之一。

应该注意的是，本文所描述的JWT 认证设置所使用的是对称密钥算法（HS256）。你也可以以类似的方式设置JWT 身份验证， 除非你使用非对称算法（RS256）,并且应用程序服务器具有公钥。请看这个 在[Stack OverFlow](https://stackoverflow.com/questions/39239051/rs256-vs-hs256-whats-the-difference)中的问题，了解使用对称和非对称算法之间的差异。

还应该注意的是， JWT 应该是HTTPS连接(而不是 HTTP)。HTTPS连接有助于防止未授权的用户通过使用它来窃取发送的JWT,从而别人无法拦截服务器和用户之前的通信。

此外，JWT 中的 payoad(负载)设置过期时间,特别是过期时间短的payload(负载)。这样如果旧的JWT 受到攻击，它们将被是为无效，并且不能再被使用。

如果你喜欢本文并且正在为 AWS Lambda 工作并正在使用JWT,请查看我们的项目：[Vandium](https://github.com/vandium-io/vandium-node)




![fun](https://s2.ax1x.com/2019/08/09/ebhQ1K.png)





【作者简介：】 Mars  芦苇科技web前端开发工程师 喜欢 看电影 ，撸铁 还有学习。擅长 微信小程序开发， 系统管理后台。访问 [www.talkmoney.cn](http://www.talkmoney.cn/) 了解更多。

作者主页：

[github](<https://github.com/Marszht>)

[segmentfault](<https://segmentfault.com/u/mars_5ad9c4d43eed5>)