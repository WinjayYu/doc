
## node方案总结

1. node运维服务
1. node包管理方案
1. log功能

### node服务运维

php方案中有比较成型的apache、Nginx等服务器，保证服务的高性能和运行稳定性。Node原生只提供了cluster，功能和稳定性方面都不能满足需求。
采用PM2作为进程管理和守护工具。

#### 问题

在项目实际的部署中遇到以下几个问题：

1. 服务守护功能无法使用

        服务守护功能利用了linux原生的startup或者systemD，但需要root权限，但线上机器通常没有root权限，无法使用服务守护功能
        如果pm2挂了会导致服务终止，有较大风险

1. node内存泄漏问题

        node web应用程序中内存泄漏是个比较严重的问题，当发生内存泄漏时每个Node子进程会占用大量的内存空间，无法释放。
        pm2本身只有内存监控功能，并没有提供发生内存泄漏后的应急措施。

1. 服务启动或热重启

        pm2提供了start、restart、reload、startOrRestart等多重启服务的功能。在何种情况调用哪种命令给开发人员带来的较大的困惑，
        如果调用不正确可能会影响服务稳定性，难以使用。

1. pm2配置文件比较复杂

        pm2提供了很多的配置选项，通常dev和production模式又需要不同的配置，比较复杂。

针对前三个问题在pm2基础上封装了[strong-pm2](https://github.com/fex-team/strong-pm2)提供了下面三个命令：

* spm2 daemon ： 服务守护功能结合crontab使用
* spm2 memwatch ： 子进程内存检测，发现内存泄漏自动重启该子进程
* spm2 startOrReload ： 提供统一的服务启动命令

#### 后续计划

strong-pm2完善

* 使用文档补充
* 功能完善

        0. pm2 不支持window，需要提供window下的简易解决方案
        1. spm2 daemon需要支持noah crontab服务需求
        2. memwatch发现内存泄漏，增加报警功能
        3. 增加日志记录功能：记录进程守护、子进程内存等情况，方便debug分析问题

提供配置文件脚手架模版

### 包管理最佳实践方案

包管理方案主要是解决两个问题：

* 关于Node应用中依赖的公共资源包如何管理
* 内部团队开发的私有资源包如何管理(下载、更新...)

#### 问题和解决方案

目前文库通过package.json和node_modules管理所有的资源依赖，node_modules提交svn管理，不用每次重新下载安装，这种方案有以下几个问题：

* 通常node_modules文件都非常多加入svn管理，会导致每次代码编译、上线时间都会非常的长
* npm install时由于依赖资源包的更新可能引起产品不稳定、并且给测试、diff codereview都带来很大的麻烦

        例如：yog依赖于express，我们够保证yog依赖express的特定版本，但无法保证express自身依赖的资源包更新的问题，并且这种更新引起的bug非常难以定位

* 如果依赖有环境相关(win、linux、mac)的类库(pm2)，因为node_modules是和特定的环境相关的，没办法做到完全的通用，需要针对不同的环境编译不同的可执行文件
* 私有资源完全依赖svn管理，npm install是无法下载，容易导致遗漏

解决方案参考文档node-module.md

#### 后续计划

* lights封装包管理流程化工具开发

        目前包管理过程涉及步骤较多例如npm install、npm shrinkwrap、pac、pac install、npm rebuild等等，非常复杂，急需提供简单命令完成所有流程化工作

* 私有npm仓库搭建讨论方案讨论

* 包管理实践指导文档

### log模块

日志模块主要提供了node日志记录的功能，日志主要分为：

    webserver日志：access、error
    应用程序日志

实现了日志格式配置、地址配置、级别等常见的功能。

#### 问题

日志模块实现参照了odp以及主流log模块功能，使用中没有太多问题。新需求：

* 能够在线上运行时动态调整log级别，打出debug信息，方便调试

#### 后续计划

* 支持动态调整log级别功能
