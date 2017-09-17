# Docker Swarm
 Docker Swarm是Docker的原生集群。它将Docker宿主机池转变成单个虚拟的Docker宿主机。
 The cluster management and orchestration features embedded in the Docker Engine are built using SwarmKit. 
 Docker engines participating in a cluster are running in swarm mode. You enable swarm mode for an engine by either initializing a swarm or joining an existing swarm.
 
 Swarm Mode有两种类型的服务：replicated 和 global。对于replicated服务，你必须指定replica task的数量。对于global服务，调度器会把任务放到每一个有效的节点上。
 你可以通过使用--mode标志来控制这种服务。假如你不制定一个模式，那么服务的缺省为replicated.
 
 # What is a node?
 A node is an instance of the Docker engine participating in the swarm. You can also think of this as a Docker node. You can run one or more nodes on a single physical computer or cloud server, 
 but production swarm deployments typically include Docker nodes distributed across multiple physical and cloud machines.
 
 To deploy your application to a swarm, you submit a service definition to a manager node. The manager node dispatches units of work called tasks to worker nodes.
 
 Manager nodes also perform the orchestration and cluster management functions required to maintain the desired state of the swarm. Manager nodes elect a single leader to conduct orchestration tasks.
 
 Worker nodes receive and execute tasks dispatched from manager nodes. By default manager nodes also run services as worker nodes, 
 **but you can configure them to run manager tasks exclusively and be manager-only nodes**. An agent runs on each worker node and reports on the tasks assigned to it. 
 The worker node notifies the manager node of the current state of its assigned tasks so that the manager can maintain the desired state of each worker.
 
 ## Services 
 
 A service is the definition of the tasks to execute on the worker nodes. It is the central structure of the swarm system and the primary root of user interaction with the swarm.  
 When you create a service, you specify which container image to use and which commands to execute inside running containers.  
 In the **replicated services** model, the swarm manager distributes a specific number of replica tasks among the nodes based upon the scale you set in the desired state.  
 For **global services**, the swarm runs one task for the service on every available node in the cluster.  
  A global service is a service that runs one task on every node. There is no pre-specified number of tasks. Each time you add a node to the swarm,   
  the orchestrator creates a task and the scheduler assigns the task to the new node. Good candidates for global services are monitoring agents,   
  an anti-virus scanners or other types of containers that you want to run on every node in the swarm.  
## Tasks
A task carries a Docker container and the commands to run inside the container. It is the atomic scheduling unit of swarm.   
Manager nodes assign tasks to worker nodes according to the number of replicas set in the service scale.   
Once a task is assigned to a node, it cannot move to another node. It can only run on the assigned node or fail.  

## Load balancing

The swarm manager uses ingress load balancing to expose the services you want to make available externally to the swarm.   
The swarm manager can automatically assign the service a PublishedPort or you can configure a PublishedPort for the service.   
**You can specify any unused port**. If you do not specify a port, the swarm manager assigns the service a port in the 30000-32767 range.  

External components, such as cloud load balancers, can access the service on the PublishedPort of any node in the cluster whether or not the node is currently running the task for the service.   
All nodes in the swarm route ingress connections to a running task instance.  

Swarm mode has an internal DNS component that automatically assigns each service in the swarm a DNS entry.   
The swarm manager uses internal load balancing to distribute requests among services within the cluster based upon the DNS name of the service.  

## 集群工具
集群工具决定在哪里启动job（容器）,如何存储job，什么时候最终重启job等。运维人员仅仅需要配置一些行为，决定集群的拓扑和规模，调优设置，并且启用或者停用高级特性。  
## 容器管理工具  
他们不提供容器托管，但是它们和一个或者多个已有系统交互；这样类型的软件通常都提供了很好的Web接口，监控工具，以及其他可视化或者高层级功能。
Rancher或者Tutum就是这样的容器管理工具。

## 使用Docker Machine创建4个集群节点
根据[官方文档](https://docs.docker.com/machine/drivers/hyper-v/#4-create-the-nodes-with-docker-machine-and-the-microsoft-hyper-v-driver)配置virtual switch manger。  
而后使用下面的命令来生成docker-machine
docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" manager1
附注：Primary Virtual Switch的建立方法请参考Docker官网
而后勇docker-machine ip manager1得到Manager1的IP地址
在有代理服务器的环境下，把Manager1的IP加入NO_PROXY中。
 set NO_PROXY=%NO_PROXY%,192.168.59.103 


## 命令
* swarm init
   初始化Swarm。该命令在后台的当前Docker主机上创建一个管理器，并且生成secret（这是worker传递给API的密码，从而可以通过认证加入集群）
   The Engine sets up the swarm as follows:
   * switches the current node into swarm mode.
   * creates a swarm named default.
   * designates the current node as a leader manager node for the swarm.
   * names the node with the machine hostname.
   * configures the manager to listen on an active network interface on port 2377.
   * sets the current node to Active availability, meaning it can receive tasks from the scheduler.
   * starts an internal distributed data store for Engines participating in the swarm to maintain a consistent view of the swarm and all services running on it.
   * by default, generates a self-signed root CA for the swarm.
   * by default, generates tokens for worker and manager nodes to join the swarm.
   * creates an overlay network named ingress for publishing service ports external to the swarm.

* swarm join
   worker使用该参数加入集群，必须指定secret，以及管理器IP端口值列表。
   The docker swarm join command does the following:
   * switches the Docker Engine on the current node into swarm mode.
   * requests a TLS certificate from the manager.
   * names the node with the machine hostname
   * joins the current node to the swarm at the manager listen address based upon the swarm token.
   * sets the current node to Active availability, meaning it can receive tasks from the scheduler.
   * extends the ingress overlay network to the current node.

* swarm join-token
   用来管理join-token。join-token是一种特别的令牌secret用于加入管理器或者worker（管理器和worker有不同的令牌值）。该命令可以很方便地让Swarm打印出加入管理器或worker的必须命令。
   docker swarm join-token worker
   docker swarm join-token manager
* swarm update
   通过变更一些值来更新集群，比如，可以使用该参数指定证书端点的新URL
* swarm leave
    让当前节点离开集群。如果有什么东西阻止该操作的完成，可以加上--force参数
* node demote/promote
     该命令用来管理节点状态。借助该机制，用户可以将某个节点升级为管理器，或者降级为worker。实际上，Swarm将尝试demote/promote。
* node inspect
     该参数和docker info 一样，只不过是用来得到Swarm节点的信息。它会打印出节点相关的信息。
* node ls
     列出连接到该集群的节点
* node rm
     尝试移除一个worker。如果用户想要移除管理器，必须先将其降级为worker。
* node ps
     显示运行在特定节点上的任务列表
* node update
     让用户可以变更某个节点的一些配置值，也就是标签。
      docker node update --availability active worker1：让worker1重新加入集群
      docker node update --availability drain worker1：让worker1离开集群
* service create
   Create a new service
   Example: Run a nginx web server service on every swarm node
   > $ docker service create \
   >  --mode global \
   >  --publish mode=host,target=80,published=8080 \
   >  --name=nginx \
   >  nginx:latest
   
   Usage:  docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]  
   Options:  
      --config config                                     Specify configurations to expose to the service  
      --constraint list                                     Placement constraints  
      --container-label list                            Container labels  
      --credential-spec credential-spec         Credential spec for managed service account (Windows only)  
  -d, --detach                  Exit immediately instead of waiting for the service to converge (default true)  
      --dns list                           Set custom DNS servers  
      --dns-option list                    Set DNS options  
      --dns-search list                    Set custom DNS search domains  
      --endpoint-mode string               Endpoint mode (vip or dnsrr) (default "vip")  
      --entrypoint command                 Overwrite the default ENTRYPOINT of the image  
  -e, --env list                           Set environment variables  
      --env-file list                      Read in a file of environment variables  
      --group list                         Set one or more supplementary user groups for the container  
      --health-cmd string                  Command to run to check health  
      --health-interval duration           Time between running the check(ms|s|m|h)  
      --health-retries int                 Consecutive failures needed to report unhealthy  
      --health-start-period duration       Start period for the container to initialize before counting retries towards unstable (ms|s|m|h)  
      --health-timeout duration            Maximum time to allow one check to run (ms|s|m|h)  
      --help                               Print usage  
      --host list                          Set one or more custom host-to-IP mappings (host:ip)  
      --hostname string                    Container hostname  
  -l, --label list                         Service labels  
      --limit-cpu decimal                  Limit CPUs  
      --limit-memory bytes                 Limit Memory  
      --log-driver string                  Logging driver for service  
      --log-opt list                       Logging driver options  
      --mode string                        Service mode (replicated or global) (default "replicated")  
      --mount mount                        Attach a filesystem mount to the service  
      --name string                        Service name  
      --network network                    Network attachments  
      --no-healthcheck                     Disable any container-specified HEALTHCHECK  
      --no-resolve-image                   Do not query the registry to resolve image digest and supported platforms  
      --placement-pref pref                Add a placement preference  
  -p, --publish port                       Publish a port as a node port  
  -q, --quiet                              Suppress progress output  
      --read-only                          Mount the container's root filesystem as read only  
      --replicas uint                      Number of tasks  
      --reserve-cpu decimal                Reserve CPUs  
      --reserve-memory bytes               Reserve Memory  
      --restart-condition string           Restart when condition is met  ("none"|"on-failure"|"any") (default "any")  
      --restart-delay duration             Delay between restart attempts  (ns|us|ms|s|m|h) (default 5s)  
      --restart-max-attempts uint          Maximum number of restarts  before giving up  
      --restart-window duration            Window used to evaluate the restart policy (ns|us|ms|s|m|h)  
      --rollback-delay duration            Delay between task rollbacks (ns|us|ms|s|m|h) (default 0s)  
      --rollback-failure-action string     Action on rollback failure ("pause"|"continue") (default "pause")  
      --rollback-max-failure-ratio float   Failure rate to tolerate during a rollback (default 0)  
      --rollback-monitor duration          Duration after each task rollback to monitor for failure (ns|us|ms|s|m|h) (default 5s) 
      --rollback-order string              Rollback order ("start-first"|"stop-first") (default "stop-first")  
      --rollback-parallelism uint          Maximum number of tasks rolled back simultaneously (0 to roll back all at once) (default 1)   
      --secret secret                      Specify secrets to expose to the service  
      --stop-grace-period duration         Time to wait before force killing a container (ns|us|ms|s|m|h) (default 10s)  
      --stop-signal string                 Signal to stop the container 
  -t, --tty                                Allocate a pseudo-TTY  
      --update-delay duration              Delay between updates(ns|us|ms|s|m|h) (default 0s)  
      --update-failure-action string       Action on update failure("pause"|"continue"|"rollback") (default "pause")  
      --update-max-failure-ratio float     Failure rate to tolerate during an update (default 0)  
      --update-monitor duration            Duration after each task update to monitor for failure (ns|us|ms|s|m|h) (default 5s)  
      --update-order string                Update order ("start-first"|"stop-first") (default "stop-first")  
      --update-parallelism uint            Maximum number of tasks updated simultaneously (0 to update all at once) (default 1)  
  -u, --user string                        Username or UID (format: <name|uid>[:<group|gid>])  
      --with-registry-auth                 Send registry authentication details to swarm agents  
  -w, --workdir string                     Working directory inside the container  
* service inspect  
   Display detailed information on one or more services
* service ls  
   List services
* service rm  
   Remove a service
* service scale  
   Scale one or multiple services
* service ps  
   List the tasks of a service.see which nodes are running the service:
* service update  
  Update a service
  The scheduler applies rolling updates as follows by default:
  * Stop the first task.
  * Schedule update for the stopped task.
  * Start the container for the updated task.
  * If the update to a task returns RUNNING, wait for the specified delay period then start the next task.
  * If, at any time during the update, a task returns FAILED, pause the update.

## 高可用
To take advantage of swarm mode’s fault-tolerance features, Docker recommends you implement an odd number of nodes according to your organization’s high-availability requirements.   
When you have multiple managers you can recover from the failure of a manager node without downtime.  
* A three-manager swarm tolerates a maximum loss of one manager.
* A five-manager swarm tolerates a maximum simultaneous loss of two manager nodes.
* An N manager cluster will tolerate the loss of at most (N-1)/2 managers.
* Docker recommends a maximum of seven manager nodes for a swarm.
To prevent the scheduler from placing tasks on a manager node in a multi-node swarm, set the availability for the manager node to Drain. 
The scheduler gracefully stops tasks on nodes in Drain mode and schedules the tasks on an Active node. The scheduler does not assign new tasks to nodes with Drain availability.

## 安全
it is a best practice to implement a regular rotation schedule for any secret including swarm join tokens. We recommend that you rotate your tokens at least every 6 months.
Run swarm join-token --rotate to invalidate the old token and generate a new token. Specify whether you want to rotate the token for worker or manager nodes:

## Docker网络
1. First, create overlay network on a manager node using the docker network create command with the --driver overlay flag.
   
   $ docker network create --driver overlay my-network
   
2. You can create a new service and pass the --network flag to attach the service to the overlay network:

    $ docker service create \
      --replicas 3 \
      --network my-network \
      --name my-web \
      nginx

   The swarm extends my-network to each node running the service.
   
You can also connect an existing service to an overlay network using the --network-add flag.   

$ docker service update --network-add my-network my-web

To disconnect a running service from a network, use the --network-rm flag.

$ docker service update --network-rm my-network my-web

## 最佳实践
1. 通过--advertise-addr 给MangerNode一个静态的IP地址。这样系统重启后，不会因为ManagerNode有了一个新的IPAddress而导致worker加入不了Swarm.
2. 避免Manager节点也做为worker执行Task。方法为在manager node上执行下面的命令
    docker node update --availability  drain <NODE>
    


