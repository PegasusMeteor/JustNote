# CAS Central Authentication Service 

最近在整理单点登陆解决方案，整理一下。


## CAS 介绍

CAS是用于Web的企业多语言单点登录解决方案，它试图成为满足身份验证和授权需求的综合平台。
通俗来说，CAS是可以用来整合多套系统，并且完成统一用户身份认证。
在官方文档的介绍中，CAS包含下面的特点与技术
- Spring Webflow / Spring Boot Java服务器组件
- 插件式地身份验证支持(LDAP, Database, X.509, SPNEGO, JAAS, JWT, RADIUS, MongoDb,等等)
- 支持多种协议(CAS, SAML, WS-Federation, OAuth2, OpenID, OpenID Connect, REST)
- 支持多种服务提供方地多种因素身份验证(Duo Security, FIDO U2F, YubiKey, Google - Authenticator, Authy, Acceptto, 等)
- 支持委托给外部提供商地身份验证，如ADFS, Facebook, Twitter, SAML2 IdPs。
- 内置对密码管理，通知，使用条款和模拟的支持。
- 实时监控，与应用程序行为跟踪，统计信息与日志记录。
- 使用特定地身份验证策略管理和注册客户端应用与服务。
- 跨平台客户端支持Java, .Net, PHP, Perl, Apache。
- 与InCommon，Box，Office365，ServiceNow，Salesforce，Workday，WebAdvisor，Drupal，- Blackboard，Moodle，Google Apps等集成。


## CAS 架构

CAS 的架构图如下所示。

![CAS 架构图](cas/images/cas0.png)


### CAS Server

CAS 服务器是基于Spring 框架构建的 Java servlet 应用程序。主要职责是通过发行ticket来验证用户，并授予用户那些已经启用了CAS的服务的访问权限。当服务器在登录成功后向用户颁发ticket-granting ticket (TGT)时，将会建立一个SSO会话。应用户的请求，使用TGT作为令牌，通过浏览器重定向将service ticket (ST)发送给服务。随后，通过反向通道通信在CAS服务器上验证ST。


## CAS Client

CAS Client 通常包含两种不同的含义。

CAS客户端可以是通过支持的协议与服务器通信的任何启用CAS的应用程序。 

CAS客户端也可以是一个软件包，可以与各种软件平台和应用程序集成在一起，以便通过某种身份验证协议（例如CAS，SAML，OAuth）与CAS服务器进行通信。 

CAS Client 支持下面的平台与应用。
**平台**
- Apache httpd Server (mod_auth_cas module)
- Java (Java CAS Client)
- .NET (.NET CAS Client)
- PHP (phpCAS)
- Perl (PerlCAS)
- Python (pycas)
- Ruby (rubycas-client)
**应用**
- Outlook Web Application (ClearPass + .NET CAS Client)
- Atlassian Confluence
- Atlassian JIRA
- Drupal
- Liferay
- uPortal

## CAS Client

客户端通过几种支持的协议与服务端进行通信。所有支持的协议在概念上是相近的。但是一些协议的特点使得他们适合在特定场景中应用。例如CAS协议支持委托(代理)认证，SAML协议支持属性释放和单点退出。
- CAS (versions 1, 2, and 3)
- SAML 1.1
- OpenID
- OAuth (1.0, 2.0)


## 实验环境架构

- CAS 6.4.0
- CAS Service Management 6.3.0
- Gitlab CE 13.8.2
- OpenLDAP
- openldap-2.4.44-20.el7.x86_64
- tomcat 9.0.
- JDK 11.0


通过将 官方网站的图 简单的修改，我们构建出实验环境架构如下图所示。

![CAS 实验环境架构图](cas/images/cas7.png)