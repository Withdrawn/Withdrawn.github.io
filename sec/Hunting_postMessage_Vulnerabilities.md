https://www.exploit-db.com/docs/40287.pdf
捕捉postMessage漏洞



## 1.介绍
Sec-1 Ltd 与 AppCheck.com 合作去承办一个研究项目下一代网页应用的安全挑战.这个项目包含同源通信机制的调查通过HTML，包含postMessage和CORS.

研究中的一个重要发现显示了不安全的postMessage实现漏洞经常被安全扫描器错过并且需要咨询人员进行手动回顾

结果摘要：
	- 跨域通信通过postMessage介绍了一个被感染的数据源是非常难去被现在的工具识别。
	- 跨站脚本和信息泄露漏洞作为不安全的postMassage代码的结果在Alexa前10和许多财富500强企业的网站上出现。本文包含了3个学习案例。
	- 和开发和信息安全社区的成员讨论表明了本文中的知道的非常少。许多情况下postMessage事件不容易被确定为恶意污染数据的潜在来源。
	- 在许多情况下漏洞代码通过第三方库引入，因此可能破坏在其它安全应用程序的安全

	
	本文的目的是去提供最常见的postMessage安全缺陷的概述和介绍一个方法和工具集去快速识别漏洞在黑盒审计的过程中。




1.1 关于SEC-1 有限公司
	Sec-1 有限公司是一个悠久的英国渗透测试公司建立于2001.它的客户合作多种多样，包括为英国金融时报100和财富500强公司的完成测试任务进一步的信息可以在这里找到 http://www.sec-1.com/


1.2.  关于 APPCHECK
	AppCheck 有限公司提供了一个领先的WEB应用和外部基础设施漏洞扫描工具(自动化渗透测试工具)，运行他的使用者自动发现安全漏洞在他们网络中的安全边界内更快，更容易，更准确。进一步的信息可以在这里找到 http://www.appcheck.com

1.3. 跨域通信
	
	现代WEB浏览器采用了一个重要的安全机制叫做同源策略(SOP)作为一个安全边界在加载的不同来源网页中。虽然SOP保留在这是谢天谢地了，允许安全和可控的通信在不同应用中的需求不断在增加。

	1.3.1. 同源策略
		同源策略限制了从一个来源加载的文档或脚本对另一个源的资源如何反应。这是一个关键的机制去隔离潜在的恶意文件(Mozilla, n.d.)
		
		文档(网页)被认为同一个来源如果他们有相同的协议(http或https),端口号，和主机名。
		简单来说，一个给定页面内运行的脚本有下列限制;
		- 脚本只能发起HTTP请求和处理响应在同源的主机中。
		- 脚本只能读取和写入到同源的框架中

		这个机制在保护指定应用用户session是非常关键的；没有同源策略一个恶意的网站能加载一个敏感的应用(比如网上银行)在一个框架或一个窗口并且对已经加载的应用取得完全的控制去读取数据或进行操作代表用户

		下表中列出了一些例子与http://www.appcheck.com/比较不同URL的来源.
		Protocol, Port, Hostname
		http:// 80 www.appcheck.com

		Compared URL 								Outcome					 Reason
		http://www.appcheck.com/admin/users.html 	Success 			Same protocol and host
		http://www.appcheck.com/cms/edit.php 		Success 			Same protocol and host
		http://www.appcheck.com:81/cms/edit.php 	Failure 			Same protocol/host but different port
		https://www.appcheck.com/cms/edit.php 		Failure 			Different protocol
		http://en.appcheck.com/cms/edit.php 		Failure 			Different host
		http://appcheck.com/cms/edit.php 			Failure 			Different host (exact match required)
		http://v2.www.appcheck.com/cms/edit.php 	Failure 			Different host (exact match required)


	1.3.2. 跨源通信的需求
		小的单个来源应用通常记录他们的用户状态信息使用一个session当用户第一次访问网站。cookie的作用域当前应用的域名或父域名，通过浏览器被每个后续请求提交。该应用程序然后能够使用该会话cookie来唯一地标识用户提供的功能，如认证和授权。
			