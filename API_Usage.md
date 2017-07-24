##BigBlueButton API

> 官方文档：[http://docs.bigbluebutton.org/dev/api.html#overview](http://docs.bigbluebutton.org/dev/api.html#overview)

###总览
接口是为了让开发者自定义。在自己的BigBlueButton中调用接口，程序发一个HTTP请求到服务器的API(通常服务器的域名后面加上`/bigbluebutton/api`)。所有的API调用必须包含一个BigBlueButton服务和共享秘钥的校验和。
API调用的时候BigBlueButton会返回一个XML响应。

###API数据类型
*API总共三种数据类型*
#####字符串（String）
这个数据类型表示一个UTF-8编码的字符串。当发送一个字符串到BigBlueButton服务的API的时候，要确定使用的URL编码是UTF-8，这样才能不会出现编码错误。字符串中绝对不能包含控制符（就是0x00到0x1F之间的字符）。
一些BigBlueButton的API的参数会添加一些额外的限制条件，或者字符串的长度。这些约束在参数文档中有描述。
#####数字（Number）
这种数据类型应该是一个非负整数，这个参数的值必须是一个0到9之间的有效数字。应该是一个无符号的数字，并且没有逗号或者句号。
#####布尔（Boolean）
值是true或者false，这个参数的值必须是字符串，是true或者false，全部小写，其他的值可能会引发错误。

###API安全模式
BigBlueButton的API支持持有共享秘钥的第三方程序调用，不允许没有共享秘钥的用户调用。
BigBlueButton的API几乎都是服务到服务的调用模式。如果安装了bbb-demo模块，就能得到一份完整的接口示例，使用的是JSP（Java server pages），这份示例演示了如何使用BigBlueButton的API。这个demo运行在tomcat7中。这个服务从BigBlueButton服务中调用接口。
*可以用下面的命令得到API的参数（API终结点和共享秘钥）*
`bbb-conf --secret`
返回如下信息
```
   URL: http://127.0.0.1/bigbluebutton/
Secret: 52870d091c16b723cf683047bdce4b64
```
你可以在网页上或者在浏览器或者js调用接口的时候**不嵌入共享秘钥**。现代浏览器内置的Debug工具可以让任何人都很容易的接近这个秘钥。一旦有人获取到你的BigBlueButton服务器的共享秘钥，他们就自己调用API。这个共享秘钥只允许你的服务器端的程序构建。（因此，对终端用户是不可见的）

###配置
**共享秘钥在下面的文件中设置**
```
/var/lib/tomcat6/webapps/bigbluebutton/WEB-INF/classes/bigbluebutton.properties
```
*其中tomcat6目录因为版本可能是tomcat7或者其他版本号，使用tab补全即可，不要钻牛角尖*
**打开文件修改如下设置项**
```
securitySalt=<your_salt>
```
**当安装号BigBlueButton之后，就会自动生成一个随机的32个字符的字符串，当然也可以使用`bbb-conf`命令随时修改密钥**
```
$ sudo bbb-conf --secret <new_shared_secret>
```
经我测试，下面的命令也可以修改共享密钥
```
$ sudo bbb-conf -setsecret <new_shared_secret>
```
**注意：不要允许终端用户知道你的共享密钥或者你的任何密钥泄露**
###用法
BigBlueButton的安全性模块的实现依赖 `ApiController.groovy` 控制器。每当API接受到请求的时候，这个控制器由HTTP查询字符串和服务的共享字符串计算出一个校验和。然后匹配传入的校验和与计算出来的校验和。如果匹配，这个控制器就允许传入的请求。
要使用安全性模块，必须在你的服务中创建一个调用名称和请求字符串和共享密钥的SHA-1校验和。步骤如下：
- 为API创建一个没有校验和参数的查询字符串
	- 创建会议API调用的例子
	```
    name=Test+Meeting&meetingID=abc123&attendeePW=111222&moderatorPW=333444
    ```
- 预置API调用名称到字符串
	- 上面的字符串的例子
		- API调用名称 `create`
		- 字符串变成了：
		```
        createname=Test+Meeting&meetingID=abc123&attendeePW=111222&moderatorPW=333444
        ```
- 现在，把共享密钥加到字符串中
	- 上面的字符串的例子
		- 共享密钥：639259d4-9dd8-4b25-bf01-95f9567eaf4b
		- 字符串变成了
		```
        createname=Test+Meeting&meetingID=abc123&attendeePW=111222&moderatorPW=333444639259d4-9dd8-4b25-bf01-95f9567eaf4b
        ```
- 现在，求出这个字符串的SHA-1的值（实现方法因语言的不同而不同）
	- 上面的字符串的SHA-1的值是：
	```
    1fcbb0c4fc1f039f73aa6d697d2db9ba7f803f17
    ```
- 现在把checksum参数添加到请求字符串中。
	- 上面的字符串就变成了：
	```
    name=Test+Meeting&meetingID=abc123&attendeePW=111222&moderatorPW=333444&checksum=1fcbb0c4fc1f039f73aa6d697d2db9ba7f803f17
    ```
调用接口的时候必须发送checksum。因为终端用户不知道你的密钥，所以就不能伪造服务，并且也不能修改任何一个接口，因为哪怕参数名或者值有一个字符改变就会让checksum的值发生很大的改变。
实现SHA-1的方法在各种语言中都大同小异，下面是例子
PHP
调用`sha(string + salt)`
###异常处理
当异常发生时，所有的API接口都会尽力返回一个完整的XML格式的信息，其中包含了足够的信息让调用者检查代码中的错误。
错误和`returncode` `FAILED` `message` `messageKey`一起返回，我们尽量让`messageKey`在API的声明周期中不发生改变。然而，`message`的值只是一个普通的文本，它会随着时间改变。
如果你需要，你可以在你自己的系统中使用`messageKey`来查找错误类型查阅国际化文本。举个例子，一个无效的请求可能返回一个这样的错误信息:"No conferece with that meeting ID exists", but the messagesKey is simple "invalidMeetingIndentifier"。
###API方法
#####管理
下面描述了管理调用
| 方法 | 描述 |
|--------|--------|
| create | 创建一个新的会议 |
| getDefaultConfigXML | 获取默认的config.xml(这些设置配置了所有的BigBlueButton客户端)。
| setConfigXML | 添加一个客户自定义的config.xml到现有的会议中 |
| join | 添加一个新的用户到一个现有会议中 |
| end | 结束会议 |
#####监视
下面的信息描述了监视调用
| 方法 | 描述 |
|--------|--------|
| isMeetingRunning | 检查一个正在运行的会议 |
| getMeetings | 获取会议列表 |
| getMeetingInfo | 获取会议详情 |
#####记录
| 方法 | 描述 |
|--------|--------|
| getRecording | 获取记录列表 |
| publishRecordings | 发布记录 |
| deleteRecordings | 删除记录 |
| updateRecordings | 更新记录 |
###API调用
对于任何接口来说，请求参数都是标准化的，它们可能被任何调用返回
#####参数
| 参数名 | 必须/可选 | 类型 | 描述 |
|--------|--------|--------|--------|
| checksum | 可变 | String | 在安全性一节，对这个参数有详细的描述。这是一个基于SHA-1的callName+queryString+securitySalt的哈希字符串。 这个security salt是可以配置的，在开发的时候，调用所有的API都要包含checksum参数。|
#####响应
| 参数名 | 返回时间 | 类型 | 描述 |
| --------- | -------- | --------| -------- |
| returncode | 每次 | String |　显示防范是否达到预期的效果.只有*两个值*:　**FAILED**:这是一些方法的错误——查看`message` 和｀messageKey｀来获取更多的信息，注意，如果returncode是FAILED，这个调用的返回参数标记成”always returned“不会再被返回。它们只返回一一部分成功的响应。**SUCCESS**调用成功，其他的参数通常是和这个调用会返回的东西有关的　｜
| message | 有时 | String | 关于调用的附加信息。一个message参数总是会被返回如果returncode是FAILED。如果附加信息有用的话，message也会返回。|
| messageKey | 有时 | String | 提供一个相似的功能给message，并且遵循相同的规则。然而，一个message key更简短，而且在API的生命周期一样长，而不会像message一样会随着时间变化。如果你的第三方软件要返回国际化的或者其他标准的message，基于messageKey你可以查阅你自定义的message
###create
创建一个BigBlueButton会议
这个create调用是幂等的：你可以使用同样的没有副作用的参数多次调用这个接口。这简化了用户加入会话的逻辑，你的程序可以在返回join URL之前一直调用create。用这种方法，不管用户进入的顺序如何，这个会议会一直存在，但不会为空。BigBlueButton服务将会自动的删除空的会议，但是在`defaultMeetingCreateJoinDuration`在`bigbluebutton.properties`定义几分钟之后就再没有用户。
#####方法URL
**http://yourserver.com/bigbluebutton/api/create?[parameters]&checksum=[checksum]**
参数：
| 参数名 | 必须/可选 | 类型 | 描述 |
| -------- | -------- | -------- | -------- |
| name | 可选 | String | 会议名称 |
| meetingID | 必须 | String |　一个会议的ID用来在第三方app中确定会议。meetingID必须是唯一的：不同的活动的会议不能使用相同搞得meetingID。如果你提供一个不唯一的meetingID（一个正在运行的会议所使用的meetingID，如果另外的接口使用了完全一样的参数，这个create调用也会成功（但是会在响应中返回一个警告信息）。这个create调用是幂等的：调用多次不会产生副作用。这使得第三方app可以避开检查如果一个会议正在运行并且要一直调用create在加入每个用户之前。会议ID应该使用大写或小写的ASCII字符，数字破折号或者下划线。生成一个GUID是一个不错的选择，这会保证不同的会议不会使用相同的会议ID ｜
| attendeePW | 必选 | String |　这个密码在与会者加入会议时用到，这是可选的，如果没有提供，BigBlueButton会分配一个随机的密码。|
| moderatorPW | 必选 | String | 这个密码在主持人进入会议时使用，或者某些管理操作（比如，结束会议）。这是可选的，如果没有提供，BigBlueButton会随机分配一个密码（这个密码在调用create接口时返回）|
| welcome | 可选 | String | 当参与者加入的时候欢迎信息就会显示在聊天窗口上。可以引入一些关键字(%%CONFNAME%%, %%DIALNUM%%, %%CONFNUM%%)这些将会被自动替换。可以在`bigbluebutton.properties`中设置默认欢迎信息。欢迎信息对HTML格式的支持有限，从别的地方粘贴HMTL的时候要小心，比如MS word，因为使用get请求时，URL很容易就会超过长度。 |