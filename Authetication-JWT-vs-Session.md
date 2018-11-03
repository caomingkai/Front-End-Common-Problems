---
title: 'Authetication: JWT vs Session'
comments: true
date: 2018-05-21 21:59:55
type: categories
categories: Full stack
tags: 
- Authetication
- JWT
- Session
---


## I. Why do we need Authetication?
    
**Scenario: bank accout transaction**

> Authetication: an approach to know "Alice" from client side is the real "Alice" in the bank database

The bank web server needs a way to identify its customers' true identity, knowing 'Alice' is the real 'Alice' as she claims.


## II. `session&cookie` authetication

**1. Solution A**   

An approach is checking her account&password. If matches, she can access Alice's bank account; Otherwise, no way to Alice's bank account.


**2. Problem of Solution A** 

HTTP is stateless. After Alice log into the server, next time when Alice make another request to draw ¥100 from her account (*subtract ¥100 from her accout*). Now she still needs to prove she is Alice. But asking her accout&password again and again is rediculous!
    
**3. Solution B:`session` & `cookie`** 

+ First time, the Alice log into her account on bank server; 
+ Bank server maintains a session for her, and issues `sessionID` to her `cookie`. 
+ Later, each time Alice want to make a transaction, she makes HTTP requests with that `sessionID`(all HTTP requests carry `cookie` )
+ Until a certain period of time, the `session` for Alice become invalid if no activities from Alice
+ After a inactive period, if Alice wants to make transaction again, she has to log into system to set up a new `session` in bank server
+ `Server`is the safe guard**, preventing evil from accessing sensitive bank account information. Using `session`and`cookie`, customers finally don't have to input their account&password again and again!

**4. New problem in "distributed" serivce system**   
    
+ **`Distributed`** means: after Alice log into her account on server#1, next time she might be directed by `load balancer` to server#2, if at that time server#1 is already serving extremely large amount of people.
    
+ **New problem is:** `session & cookie` approach doesn't work in this case. Server#1 has Alice's necessary`context` for following transaction after she logs into it. But server#2 doesn't even know who she is! So, Alice is asked for her account and password again! Bad user experience!!!
    
**5. Soutions for "distributed" serivce system**   
>【好文】https://www.cnblogs.com/study-everyday/p/7853145.html

+ **version 1: session dupication (多机同步)**
    + 思路：多个web-server之间相互同步session，这样每个web-server之间都包含全部的session
    + 优点：web-server支持的功能，应用程序不需要修改代码
    + 不足：
        - session的同步需要数据传输，占内网带宽，有时延
        - 所有web-server都包含所有session数据，数据量受内存限制，无法水平扩展
        - 有更多web-server时要歇菜
+ **version 2: Session Storage at brower (客户端存储法)**
    + 思路：服务端存储所有用户的session，内存占用较大，可以将session存储到浏览器cookie中，每个端只要存储一个用户的数据了
    + 优点：服务端不需要存储
    + 缺点：
        - 每次http请求都携带session，占外网带宽
        - 数据存储在端上，并在网络传输，存在泄漏、篡改、窃取等安全隐患
        - session存储的数据大小受cookie限制


+ **version 3: Nginx reverse proxy hash: (反向代理hash一致性)**
    + 思路：反向代理层使用用户ip来做hash，以保证同一个ip的请求落在同一个web-server上
    + 优点：
        - 只需要改nginx配置，不需要修改应用代码
        - 负载均衡，只要hash属性是均匀的，多台web-server的负载是均衡的
        - 可以支持web-server水平扩展（session同步法是不行的，受内存限制）

    + 不足：
        - 如果web-server重启，一部分session会丢失，产生业务影响，例如部分用户重新登录
        - 如果web-server水平扩展，rehash后session重新分布，也会有一部分用户路由不到正确的session
+ **version 4: Database layer for whole session(后端统一集中存储)**
    + 思路：将session存储在web-server后端的存储层，数据库或者缓存
    + 优点：
        - 没有安全隐患
        - 可以水平扩展，数据库/缓存水平切分即可
        - web-server重启或者扩容都不会有session丢失

    + 不足：
        - 增加了一次网络调用，并且需要修改应用代码

    session读取的频率会很高，数据库压力会比较大。如果有session高可用需求，cache可以做高可用，但大部分情况下session可以丢失，一般也不需要考虑高可用。

+ **Summary: for improved `session&cookie` approach**
    
    对于方案3和方案4，推荐后者：
    - web层、service层无状态是大规模分布式系统设计原则之一，session属于状态，不宜放在web层
    - 让专业的软件做专业的事情，web-server存session？还是让cache去做这样的事情吧。

    
## III. JWT authetication

>好文: https://www.ctolib.com/flylib-fly-auth.html

> 1. server不再作为`敏感数据库`的关卡，为client服务；
> 2. brower自己`每次`都携带`authentication token`进行request

**0.【JWT本质：】** 

- **相当于每次browser request，都要求用户输入`account & password`**   
- **即：第一次用户登陆成功后，server根据其“用户名密码”生成唯一“令牌token”，往后该用户只要带上该token进行访问就可以**   
- **相同点：`session`方法类似，`cookie`中的`sessionID`也相当于“令牌token”，每次请求都携带上**
- **不同点：**
    + **`JWT `方法的token生成依赖于`用户名密码`与`SHA256`，不受server环境影响**
    + **`session`方法的`sessionID`生成依赖于`server session context`，每个server生成的`sessionID`不一致，无法适用于distributed system**




**1. Problems with `session` approach** 

+ memcache或者redis的**稳定性**会影响业务系统的可用性
+ **微服务的并发能力**受限于memcache或者redis的并发能力
+ 由此可见，集中的session管理也不是最优的选择。
+ 我们不再关注session本 ，把session的问题抛开，让我们的服务都是无状态的。
+ 于是我们把目光转向WEB TOKEN: JWT（Json web token ）    


**2. Benefits using JWT**
>【好文】https://www.ctolib.com/topics-113075.html

- 解决跨域问题：这种基于Token的访问策略可以克服cookies的跨域问题。
- 服务端无状态可以横向扩展，Token可完成认证，无需存储Session。
- 系统解耦，Token携带所有的用户信息，无需绑定一个特定的认证方案，只需要知道加密的方法和密钥就可以进行加密解密，有利于解耦。
- 防止跨站点脚本攻击，没有cookie技术，无需考虑跨站请求的安全问题。

**3. Principle of JWT 原理简介**

JSON Web Tokens的格式组成，jwt是一段被base64编码过的字符序列，用点号分隔，一共由三部分组成，头部`header`，消息体`playload`和签名`sign`

- 头部`Header`是json格式：

    ```javascript
    {
        "typ":"JWT",            // 该类型是JWT类型
        "alg":"HS256",          // 加密方式声明是HS256
        "exp":1491066992916     // expire 代表过期时间
    }
    ```
- 消息体`Playload`json格式：
    
    消息体的具体字段可根据业务需要自行定义和添加，只需在解密的时候注意拿字段的key值获取value。

    ```javascript
    {
        "userid":"123456",
        "iss":"companyName" 
    }
    ``` 

- 签名`sign`的生成

    - 签名的生成是把header和playload分别使用base64url编码,接着用’.‘把两个编码后的字符串连接起来
    - 再把这拼接起来的字符串配合密钥进行HMAC SHA-256算法加密
    - 然后再次使用base64url编码，这就拿到了签名sign. 
    - 最后把header和playload和sign用’.‘ 连接起来就生成了整个JWT。
    

**4. Principle of JWT Checking 校验简介**

>JWT不能保护数据，只能验证数据是否是原始数据，是否被篡改过

JWT是由`header.playload.sign`连接组成，只有sign是用密钥加密的，而所有的信息都在header和playload中可以直接获取，sign的作用只是校验header和playload的信息是否被篡改过

1. 加密：比如要加密验证的是userid字段：
    - 首先按前面的格式组装json消息头header和消息体playload，按header.playload组成字符串；
    - 再根据密钥和HS256加密header.playload得到sign签名；
    - 最后得到jwtToken为header.playload.sign，在http请求中的url带上参数想后端服务请求认证

2. 解密：后端服务校验jwtToken是否有权访问接口服务，进行解密认证，如校验访问者的userid：
    - 用将字符串按`.`切分三段字符串，分别得到header和playload和sign。
    - 然后将header.playload拼装用密钥和SHA-256算法进行加密然后得到新的字符串和sign进行比对，
    - 如果一样就代表数据没有被篡改，然后从头部取出exp对存活期进行判断
    - 如果超过了存活期就返回空字符串，如果在存活期内返回userid的值
