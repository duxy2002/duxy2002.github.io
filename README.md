# Duke杜的个人空间

## 1. 读书笔记

* [《Web全栈工程师的自我修养》的读书笔记](./Books/fullStack.md)
* [《Web安全深度剖析》的读书笔记](./Books/deepWebSecurity.md)
## 2. 技术调查

### 2.1 Angular2

* [如何引入第三方类库](Tech/Angular2/thirdPackage.md)
* [How to debug AngularJS with Intellij](Tech/Angular2/debug.md)
* [Angular2 入门](./Tech/Angular2/angular2start.md)
* [Angular2 命名规则及最佳实践](./Tech/Angular2/Convertion.md)
* [Angular的依赖包管理 package.json](./Tech/Angular2/package.json.md)
* [Angular父组件和子路由之间传值的方法](./Tech/Angular2/transfer.md)

### 2.2 Typescript
 * [Immutable.js 不可变数据结构](./Tech/TypeScript/Immutable.md)
 * [tsconfig.json的格式](./Tech/TypeScript/tsconfig.json.md)

### 2.3 Material Design
 * [CSS调整](./Tech/Material%20Design/CSS.md)

### 2.4 HTTP基本知识
 * [Expires](./Tech/Http/Expires.md)
 * [Cache](./Tech/Http/Cache.md)
 * [Last-Modified/ETag](./Tech/Http/ETag.md)
 * [HTTP/2](./Tech/Http/Http2.md)
 
### 2.20 JHipster
* [JHipster](./Tech/JHipster/JHipster.md)

### 2.21 JPA
* [JPA](./Tech/JPA/JPA.md)
* [JPA BestPractise](./Tech/JPA/JPABP.md)

### 2.22 SpringBoot
* [AuditEventRepository](./Tech/SpringBoot/AuditEventRepository.md)
* [Spring MVC(+Spring Boot)上での静的リソースへのアクセスを理解する](./Tech/SpringBoot/StaticResource.md)

### 2.23 Java8
* [日期处理](./Tech/Java/8/date.md)

### 2.24 多线程
* [Spring](./Tech/Thread/spring.md)
* [Java](./Tech/Thread/java.md)

### 2.40 Devops
* 代码管理（SCM）
    * GitHub
    * GitLab
    * BitBucket
    * SubVersion
* 构建工具
    * Ant
    * Gradle
    * [maven](Devops/Maven/maven.md)
* 自动部署
    * Capistrano
    * CodeDeploy
* 持续集成（CI）
    * Bamboo 
    * Hudson
    * Jenkins
* 配置管理
    * Ansible
    * Chef
    * Puppet
    * SaltStack
    * ScriptRock GuardRail
* 容器：
    * [Docker](Devops/Container/Docker/DockerStarting.md)
    * LXC、
    * 第三方厂商如AWS
* 编排
    * Kubernetes
    * Core
    * Apache Mesos
    * DC/OS
    * [Docker Swarm](Devops/Container/Docker/DockerSwarm.md)
        * [Docker Compose](./Devops/Container/Docker/DockerCompose.md)
* 服务注册与发现
    * Zookeeper
    * etcd
    * Consul
* 日志管理
    * ELK
    * Logentries
* 系统监控
    * Datadog
    * Graphite
    * Icinga
    * Nagios
* 性能监控
    * AppDynamics
    * New Relic
    * Splunk
* 压力测试
    * JMeter
    * Blaze Meter
    * loader.io
* 预警
    * PagerDuty
    * pingdom
    * 厂商自带如AWS SNS
* HTTP加速器
    * Varnish
* 消息总线
    * ActiveMQ
    * RabbitMQ
    * Kafka
    * SQS
* 应用服务器
    * Tomcat
    * JBoss
* Web服务器
    * Apache
    * Nginx
    * IIS
* 数据库
    * 关系型数据库
        * MySQL
        * Oracle
        * PostgreSQL
        * [H2](./Database/H2/H2.md)
    * NoSQL数据库
        * cassandra
        * mongoDB
        * [redis](./Devops/Redis/redis.md)
* 项目管理（PM）
    * Jira
    * Asana
    * Taiga
    * Trello
    * Basecamp
    * Pivotal Tracker

## 3. Tools

### 3.1 FrontEnd
* [JekyII/Dexy](./Tools/JekyIIAndDexy.md)
* [npm/Bower](./Tools/npmAndBowser.md)
* [Webpack](./Tools/Webpack.md)
* [webpack和TypeScript]()
* 设计工具[Axure/Sketch/<strong>Quartz Composer</strong>]
* [npm](./Tools/npm.md)
* [Node.js](./Tools/Node.md)
* electron

### 3.2 BackEnd

* [gradle](./Tools/gradle.md)
* DB重构工具[LiquiBase](./Tools/LiquiBase.md)
* [Metrics]()
* [dropwizard](http://metrics.dropwizard.io/3.2.2/)

### 3.3 系统工具
[cmder.net](http://cmder.net/)

## 4. 各种遇到的问题
### 4.1 Git
* [SSL certificate problem: self signed certificate](./Problem/Git/SelfSignedCertificate.md)
* [checkout tag](././Problem/Git/CheckoutTag.md)

### 4.2 npm

### 4.3 typings
* [typings ERR! message Unable to connect to "https://api.typings.org/entries/dt/es6-shim/tags/0.31.2%2B20160602141504"](./)

### 4.4 webdriver-manager
* [Error: self signed certificate](./Problem/webdriver/proxy.md)


### 4.5 chrome
* [How to disable 'This type of file can harm your computer' pop up](./Problem/Chrome/keepDiscard.md)

### 4.6 Angular2
* [ng build --prod: Cannot find module css-loader/index.js?sourcemap&minimize](Problem/Angular2/ngbuild.md)

### Docker
* [Docker容器内代理服务器的设定](Problem/Docker/Docker.md)
* [被链接的容器必须运行同一个Docker宿主机上。不同Docker宿主机上运行的容器无法连接](Problem/Docker/Docker.md)
* [Cannot connect to the Docker daemon. Is the docker daemon running on this host?](Problem/Docker/Docker.md)
## 5. 名词解释
* polyfill
Polyfillとは、古いブラウザであってもモダンブラウザと同等の機能を提供する方法のこと。
古いブラウザはHTML5やCSS3などの新しい仕様を実装していませんが、独自実装などを利用して極力モダンブラウザと同じ実装をいくつか実現させています。
olyfillを作成するのはブpラウザなどの実装に知悉してなければなりませんので、多くは有名なライブラリ(Modernizr、html5shiv)などを利用して実現します。

## 6.IDE
* [IntelliJ Idea](./IDE/IntelliJIdea/IntelliJIdea.md)


## Task
### 翻译
1. [Docker swarm mode overlay network security model.](https://docs.docker.com/v17.03/engine/userguide/networking/overlay-security-model/)
2. [Deploy services to a swarm](https://docs.docker.com/v17.03/engine/userguide/networking/overlay-security-model/)
3. node可以有Label？

