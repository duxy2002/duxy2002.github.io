# Docker
## 安装
[官方文档](https://docs.docker.com/docker-for-windows/install/#install-docker-for-windows)

## 新的开发流程
* 开发人员开发，提交代码到代码服务器（Github，BitBucket，Gitlab等）
* 代码服务器通过webhook调用CI/CD服务，如Codeship，Shippable，CircleCI或者自建Jenkins等。
* CI服务器下载最新代码，构建Docker镜像，并进行测试。
* 自动集成测试通过之后，就可以将之前构建的镜像推送到私有Registry。
* 运维使用新版的Docker镜像进行部署。

## Docker组件
镜像是构建Docker世界的基石。用户基于镜像来运行自己的容器。镜像可以被创建，启动，关闭，重启以及销毁。
**我们可以架设自己的私有Registry**。
镜像是Docker生命周期中的构建或打包阶段，而容器则是启动或执行阶段。
容器拥有自己的网路，IP地址，以及一个用来和宿主机进行通信的桥接网络接口。

## 我们能用Docker做什么
* 加速本地开发和构建流程，使其更加高效，更加轻量化。本地开发人员可以构建，运行并分享Docker容器。
   容器可以在开发环境中构建，然后轻松地提交到测试环境中，并最终进入生产环境。
* 能够让独立服务或应用程序在不同的环境中，得到相同的运行结果。
* 用Docker创建隔离的环境来进行测试。例如，用JenkinsCI这样的持续集成工具启动一个用于测试的容器。
* Docker可以让开发者先在本机上构建一个复杂的程序或架构来进行测试，而不是一开始就在生产环境部署，测试。
* 构建一个多用户的平台即服务(PaaS)基础设施。
* 为开发，测试提供一个轻量级的独立沙盒环境，或者将独立的沙盒环境用于技术教学，如Unix shell的使用，编程语言教学。
* 提供软件即服务(SaaS)应用程序，如Memcached即服务。
* 高性能，超大规模的宿主机部署。

## Docker容器
* docker info
    该命令会返回所有容器和镜像的数量，Docker使用的执行驱动和存储驱动，以及Docker的基础配置。
* docker run 运行容器
    这个命令会检查本地是否存在ubuntu镜像，如果本地还没有该镜像的话，那么Docker就会链接官方维护的Docker Hub Registry,
    查看Docker Hub中是否有该镜像。Docker一旦找到该镜像，就会下载该镜像并将其保存到本地宿主机中。
    参数 -i: 保证容器中STDIN是开启的。
    参数 -t: 告诉Docker为创建的容器分配一个伪tty终端。
    **若要在命令行下创建一个我们能与之进行交互的容器，而不是一个运行后台服务的容器，则这两个参数已经是最基本的参数了**。
    **下面是创建守护式容器的方法**
    参数-d: Docker会将容器放到后台运行。这样变成守护式容器。守护式容器没有交互式会话，非常适合运行应用程序和服务。大多数时候我们都需要以守护式来运行我们的容器。 
    
    --name <name>:为容器命名。一个合法的容器名称只能包含以下字符：小写字母a-z，大写字母A-Z，数字0-9，下划线，圆点，横线。（正则表达式:[a-zA-Z0-9_.-]）
    --restart: 自动重启容器。--restart=always：无论容器的退出代码是什么，Docker都会自动重启容器。除了always，我们还可以将这个标志设为on-failure，
                   这样，只有当容器的退出代码为非0值的时候，才会自动重启。另外on-failure还接受一个可选的重启次数参数。--restart=on-failure:5,这样但容器退出代码为非0时，Docker会尝试自动重启容器，最多重启5次。
    -p <port>: 用来控制Docker在运行时应该公开哪些网络端口给外部（宿主机）。
             -p \<HostPort:ContainterPort\>可以指定将容器中的端口映射到Docker宿主机的某一特定端口上。   
             -p \<IP:HostPort:ContainterPort\>可以将容器内的80端口绑定到了本地宿主机的IPAddress这个IP的HostPort端口上。
             -p \<IP::ContainterPort\>可以将容器内的80端口绑定到了本地宿主机的IPAddress这个IP的随机端口上。
             -P: 可以用来对外公开在Dockerfile中的EXPOSE指令中设置的所有端口，并且绑定到宿主机的一个随机端口上。
    -g <parameter>: 指定参数
    -w <workDir>: 覆盖Dockerfile文件中定义的WORKDIR目录。
    -e <environmentParameterName environmentParameterValue>: 用于传递环境变量。这些变量将只会在运行时有效。
* docker start <容器名/容器ID>
    重新启动一个已经停止的容器
* docker restart  <容器名/容器ID>
    重新启动一个容器。
* docker attach <容器名/容器ID>  
    附着到该容器的会话上。 
* docker ps  
    默认情况下，当执行docker ps命令时，只能看到正在运行的容器。    
    -a : 查看当前系统中容器（包括正在运行的和已经停止的）的列表。   
    -n x: 会显示最后x个容器，不论这些容器正在运行还是已经停止。   
    -q: 表示只需要返回容器的ID而不会返回容器的其他信息。   
    -l: 列出详细信息？
* docker logs <容器名/容器ID>
    查看容器内部都在干些什么。可以通过Ctrl+C退出日志跟踪。  
    参数-f: 来监控Docker的日志，这与tail -f命令非常相似。  
    参数--tail <number>:可以跟踪容器日志的某一片段。例如docker logs --tail 10 -f daemon_dave命令来跟踪某个容器的最新日志而不必读取整个日志文件。  
    参数-t: 为每条日志项加上时间戳。
* docker top  <容器名/容器ID>   
    查看容器内的进程。  
* docker exec <容器名/容器ID> （Docker1.3以上）  
    在容器内部额外启动新进程。  
* docker stop <容器名/容器ID>  
    停止守护式容器。  
* docker inspect <容器名/容器ID>   
    获取容器更多的信息，返回容器的配置信息，包括名称，命令，网络配置以及很多有用的数据。  
    -f或者--format: 来选定查看结果。  
* docker rm  <容器名/容器ID>  
    删除容器。需要注意的是，运行中的Docker容器是无法删除的！你必须先通过docker stop或docker kill命令停止容器，才能将其删除。  
    可以通过docker rm \`docker ps -a -q\`来一次性删除所有的容器。  
    
    
## Docker镜像
**在构建容器时指定仓库的标签也是一个很好的习惯。
Docker Hub中有两种类型的仓库：用户仓库（user repository）和顶层仓库（top-level repository）。
用户仓库的镜像都是由Docker用户创建的，用户仓库的命名由用户名和仓库名两部分组成，如user/password，
而顶层仓库则是由Docker内部的人来管理的。
* docker images
    列出Docker主机上可用的镜像。

* docker pull <镜像名:标签>  
    从docker registry拉取ubuntu仓库中的所有内容。  
    如果没有指定具体的镜像标签，那么Docker会自动下载latest标签的镜像。

* docker search <镜像名>  
    查找所有Docker Hub上公共的可用镜像。

* docker commit   <Dockerfile>
    创建镜像。

* docker build <Dockerfile>
   
  参数 -t: 为新镜像设置仓库和名称。例子：docker build -t="jamtur01/static_web" .    
               也可以在构建镜像的过程中为镜像设置一个标签，其使用方法为“镜像名：标签”。   
               比如：docker build -t="jamtur01/static_web:v1" .  
               上面的命令中最后的.告诉Docker到本地目录中去找Dockerfile文件。也可以指定一个Git仓库的源地址
               来指定Dockerfile的位置。例：docker build -t="jamtur01/static_web:v1" git@github.com:jamtur01/docker-static_web
               这里假定Git仓库的根目录下存在Dockerfile文件。
   参数 --no-cache: 在构建过程中Docker会将之前构建时创建的镜像当作缓存并作为新的起始点。

* docker history
   深入探求镜像是如何构建出来的。
* docker rmi <镜像名>
   删除镜像
   docker rm \`docker images -a -q\`来删除所有镜像。
   
### Dockerfile的格式
* 第一条指令都应该是FROM。
* 接着指定MAINTAINER，这条指令会告诉Docker该镜像的作者是谁，以及作者的电子邮件地址。
    例子： MAINTAINER duxy "duxy2002@gmail.com"
* ENV指令来设置环境变量。这些环境变量也会被持久保存到从我们的镜像创建的任何容器中。
* 接着是RUN指令。RUN指令会在当前镜像中运行指定的命令。
* 接着设置EXPOSE指令，这条指令告诉Docker该容器内的应用程序将会使用容器的指定端口。
* CMD 指令用于指定一个容器启动时要运行的命令。这有点儿类似于RUN指令。只是RUN指令是指定镜像被构建时要运行的命令，
  而CMD是指定容器被启动时要运行的命令。
  例子：CMD \["/bin/bash", "-l"\] **Docker推荐一只使用以数组语法来设置要执行的命令**。
  **使用Docker run命令可以覆盖CMD指令**。
  **在Dockerfile中只能指定一条CMD指令**。
* ENTRYPOINT 和CMD类似，不过实际上，docker run命令行中制定的任何参数都会被当作参数再次传递给ENTRYPOINT指令中指定的命令。
    也就是说在Dockerfile中ENTRYPOINT和CMD同时存在的话，那么CMD的内容会被当作参数传递给ENTRYPOINT。而docker run -g <parameter>的时候
    CMD指定的内容不起作用，<parameter>会被传递给ENTRYPOINT。
* WORKDIR
   用来从镜像创建一个新容器，在容器内部设置一个工作目录。ENTRYPOINT/RUN/CMD会基于这个工作目录执行。
   **这个定义有可能被docker run -w <workDir>覆盖**。
* USER
    用来指定该镜像会以什么样的用户去运行。我们可以指定用户名或者UID，以及组或GID，甚至是两者的组合。
    **如果不通过USER指令制定用户，默认用户为root**
* VOLUME
    用来向基于镜像创建的容器添加卷。一个卷是可以存在于一个或者多个容器内的特定的目录，这个目录可以绕过联合文件系统，并提供如下共享数据或者对数据进行持久化的功能。
* ADD
    ADD指令用来将构建环境下的文件和目录复制到镜像中。
* COPY
    COPY非常类似于ADD，他们根本的不同是COPY只关心在构建上下文中复制本地文件，而不去做提取（extraction）和解压（decompression）文件。
* ONBUILD
    当一个镜像被用作其他镜像的基础镜像时（比如你的镜像需要某未准备好的位置添加源代码，或者你需要执行特定于构建镜像的环境的构建脚本），该镜像中的触发器将会执行。
    触发器会在构建过程中插入新指令，我们可以认为这些指令是紧跟在FROM之后指定的。ONBUILD触发器会按照在父镜像中指定的顺序执行，并且只能被继承一次
    （也就是说只能在子镜像中执行，而不会在孙子镜像中执行）
    Doc
## Docker入门
* docker login
    登录到Docker Hub。
* docker push <镜像名>
  将镜像推送到Docker Hub.

* 自动构建
除了从命令行构建和推送镜像，Docker Hub还允许我们定义自动构建（Automated Builds)。为了使用自动构建，我们自需要将Gihub或BitBucket中含有Dockerfile文件的
仓库链接到Docker Hub即可。
    1. 打开Docker hub,登陆后单击个人信息链接，之后单击Add Repository -> Automated Build按钮
    2. 单击github logo下面的Select按钮开始账号链接。你将会转到GitHub页面并看到Docker Hub的账号链接授权请求。
    3. 有2个选项：Public and Private(recommended)和Limited.选择Public and Private，并单击Allow Access完成授权操作。
    4. 系统将提示你选择用来进行自动构建的组织和仓库。单击想用来进行自动构建的仓库后面的select按钮，之后开始对自动构建进行配置。（指定想使用的默认的分支名，并确认仓库名）
    5. 为每次自动构建过程创建的镜像指定一个标签，并指定Dockerfile的位置。
    6. 单击Create Repository按钮来将你的自动构建添加到Docker Hub中。
    