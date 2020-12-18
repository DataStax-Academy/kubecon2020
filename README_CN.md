# KubeCon 2020

## 在Kubernetes (K8s) 之上的 Apache Cassandra 自动化管理

以下这个代码/资料库中，有所有关于这次实操课程的所有资料 （敬请加入浏览器书签）：
![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/banner.png?raw=true)

## 开始之前

有两种方式可以运行本次实操课程的所有练习：

```diff
+ 提供的（AWS）云端环境
```

**现场参与活动的可以通过[以下链接](http://cn.hk.uy/3ds)添加DataStax活动小秘书来申请一个云端环境。**

在本次实例演习中我们推荐使用云端环境，因为所需要的软件已经事先预装好了。请注意所提供的云端环境只能在实操课程中使用，而且在**24小时**后被彻底删除。

**⚡ 注意:**
在这个代码/资料库中，如果你看见有`<ADDRESS>`的地方，请用你自己对应的云端环境的DNS hostname替换。 

```diff
+ 本机环境
```

如果你打算使用**自己的环境**（自己的笔记本或自己的云上主机），请确保满足运行此次演示的要求和预装所需软件（需求参考[这里](https://github.com/DataStax-Academy/kubecon2020/blob/main/setup_local.md)）。你的环境必须能运行docker，而且保证至少2核的CPU和8G的内存分配给docker运行环境，土豪上不封顶。


## 目录和资料

* [YouTube录像](https://www.youtube.com/watch?v=DI1bJ1tggmk)
* [演示文件](https://github.com/DataStax-Academy/kubecon2020/blob/main/November%20KubeCon%20Cassandra%20Kubernetes%20Workshop.pdf)
* [Discord 聊天](https://bit.ly/cassandra-workshop)
* [问答: community.datastax.com](https://community.datastax.com)

| 主题 | 说明
|---|---|
| **1 - 搭建并监控Cassandra集群** | [使用说明](#1-Setting-up-and-Monitoring-Cassandra)  |
| **2 - 访问和使用数据** | [使用说明](#2-Working-with-data)  |
| **3 - 集群伸缩** | [使用说明](#3-Scaling-up-and-down)  |
| **4 - 运行Cassandra集群数据一致性修复操作** | [使用说明](#4-Running-repairs)  |
| **5 - 相关资源** | [使用说明](#5-Resources)  |

## 1. 建立和监控Cassandra集群

首先，Helm [官方文档](https://helm.sh/docs/) 相当于是Kubernetes中的“软件包”管理程序。在今天的实操课程中，我们会用相关的Helm资料库（K8ssandra）来进行Cassandra集群的建立，监控和运维工作。有关k8ssandra Helm 资料库的详情，请参考这里：[https://helm.k8ssandra.io/](https://helm.k8ssandra.io/)。

**✅ 步骤 1a: 打开提供给你的云环境网页，选择第一个链接以登陆进提供给你的云环境机器** 

*用户名和密码*

```yaml
Username: ec2-user
password: datastax
```
![images](./Images/home-shell.png)

**✅ 步骤 1b: 确认当前目录是~/kubernetes-workshop-online，并运行git pull** 

```bash
git pull
```

*📃output*
```bash
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> git pull
From https://github.com/DataStax-Academy/kubecon2020
 * branch            main       -> FETCH_HEAD
Updating 72709d8..994a81d
Fast-forward
 README.md        | 24 +++++-------------------
 demo-values.yaml | 13 +++++++++++++
 petclinic.yaml   | 10 ++++++++--
 setup_local.md   |  4 ++--
 4 files changed, 28 insertions(+), 23 deletions(-)
 create mode 100644 demo-values.yaml     
```

**✅ 步骤 1c: 加入 K8ssandra `Helm` 资料库** 

```bash
helm repo add k8ssandra https://helm.k8ssandra.io/
```

*📃output*
```bash
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> helm repo add k8ssandra https://helm.k8ssandra.io/
"k8ssandra" has been added to your repositories      
```

**✅ 步骤 1d: 加入 Traefik `Helm` 资料库** 

在Kubernetes中, 网络端口和服务通常是通过 Ingress 控制器来提供的；在今天的实操课程中，我们用到的 Ingress 控制器是 Traefik。


```bash
helm repo add traefik https://helm.traefik.io/traefik
```

*📃output*
```bash
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> helm repo add traefik https://helm.traefik.io/traefik
"traefik" has been added to your repositories  
```

**✅ 步骤 1e: 更新 `Helm`**

```bash
helm repo update
```

>*📃output*
```bash
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> helm repo update
Hang tight while we grab the latest from your chart repositories...                                                                                              
...Successfully got an update from the "k8ssandra" chart repository                                                                                              
...Successfully got an update from the "traefik" chart repository                                                                                                
Update Complete. ⎈Happy Helming!⎈                                                                                                                                
ec2-user@ip-172-31-5-5:
```

**✅ 步骤 1f: 用 Helm 来安装 `traefik`；具体配置详情请见文件 [traefik.values.yaml](traefik.values.yaml)**

```bash
helm install traefik traefik/traefik --create-namespace -f traefik.values.yaml
```

>*📃output*
```bash
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> helm install traefik traefik/traefik --create-namespace -f traefik.values.yaml
NAME: traefik                                                                                                                                                    
LAST DEPLOYED: Tue Nov 17 15:00:53 2020                                                                                                                          
NAMESPACE: default                                                                                                                                               
STATUS: deployed                                                                                                                                                 
REVISION: 1                                                                                                                                                      
TEST SUITE: None    
```

**✅ 步骤 1g: 用 Helm 来安装 K8ssandra K8s工作员**

运行以下 Helm 命令来安装 K8ssandra，安装过程大约需要30秒。

```bash
helm install k8ssandra-tools k8ssandra/k8ssandra
```

>*📃output*
```bash
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> helm install k8ssandra-tools k8ssandra/k8ssandra
NAME: k8ssandra-tools                                                                                                                                
LAST DEPLOYED: Tue Nov 17 15:01:22 2020                                                                                                              
NAMESPACE: default                                                                                                                                   
STATUS: deployed                                                                                                                                     
REVISION: 1                                                                                                                                          
TEST SUITE: None 
```

**✅ 步骤 1h: 用 Helm 来安装 K8ssandra 集群**

```
helm install k8ssandra-cluster-a k8ssandra/k8ssandra-cluster -f demo-values.yaml
```

>*📃output*
```
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> helm install k8ssandra-cluster-a k8ssandra/k8ssandra-cluster -f demo-values.yaml                                                                                                                                       
NAME: k8ssandra-cluster-a                                                                                                                                    
LAST DEPLOYED: Tue Nov 17 15:04:56 2020                                                                                                                      
NAMESPACE: default                                                                                                                                           
STATUS: deployed                                                                                                                                             
REVISION: 1                                                                                                                                                  
TEST SUITE: None  
```

如果你使用你自己的机器，用 `127.0.0.1` 或 `localhost` 取代 以上命令中的 `${ADDRESS}`

在进行下一步前，确保这个步骤所有的工作都正确运行完成，这个过程大概需要几分钟。

```bash
watch kubectl get pods
```

>*📃output*
```
Every 2.0s: kubectl get pods                                                                     ip-172-31-9-15.eu-central-1.compute.internal: Tue Nov 17 14:20:55 2020
NAME                                                              READY   STATUS      RESTARTS   AGE                                                                   
cass-operator-cd9b57568-2fck2                                     1/1     Running     0          4m27s                                                                 
grafana-deployment-cfc94cf66-j8mw5                                1/1     Running     0          116s                                                                  
k8ssandra-cluster-a-grafana-operator-k8ssandra-6466cf94c9-vv6vl   1/1     Running     0          3m38s                                                                 
k8ssandra-cluster-a-reaper-k8ssandra-59cb88b674-wmb74             1/1     Running     0          75s                                                                   
k8ssandra-cluster-a-reaper-k8ssandra-schema-gsl64                 0/1     Completed   4          3m31s                                                                 
k8ssandra-cluster-a-reaper-operator-k8ssandra-56cc9bf47c-9ghsl    1/1     Running     0          3m38s                                                                 
k8ssandra-dc1-default-sts-0                                       2/2     Running     0          3m36s                                                                 
k8ssandra-tools-kube-prome-operator-6d556b76f8-5h54b              1/1     Running     0          4m27s                                                                 
prometheus-k8ssandra-cluster-a-prometheus-k8ssandra-0             2/2     Running     1          3m38s                                                                 
traefik-7877ff76c9-hpb97                                          1/1     Running     0          5m5s     
```

从上面的命令行输出，我们可以清楚的看到每个 Pod 是否正确上线。

注意：`reaper-k8ssandra-schema` 一开始会显示错误，但之后会自动恢复。不需要担心，这是个已知“问题”。

按 `Ctrl + C` 键退出 watch 命令。


**✅ 步骤 1i: 监控你的系统**

现代的应用程序和系统需要确保你能正确的监控它们，K8ssandra 也不例外。K8ssandra 提供内置的Grafana 和 Prometheus 组件来提供系统监控功能。

如果你使用的是我们提供的云环境，从云环境首页上点击相应的 Grafana 或 Prometheus链接进入相应的监控组件页面。

如果你用的是自己的本地 “kind” 集群环境， 在本地浏览器中输入以下地址 `prometheus.localhost:8080` 或 `grafana.localhost:8080` 来访问相应的监控组件页面。

*Grafana 网页的登陆用户名和密码:*
```yaml
username: admin
password: secret
```

![images](./Images/home-grafana.png)

在左边的面板上选择`Dashboard > Manage`，然后选择相应的监控模版
![images](./Images/grafana-1.png)

Cassandra Overview
![images](./Images/grafana-2.png)

Cassandra Cluster Condenses
![images](./Images/grafana-3.png)

Cassandra Node Metrics
![images](./Images/grafana-4.png)

## 2. 数据操作

**✅ 步骤 2a: 部署 PetClinic 应用程序**

```
kubectl apply -f petclinic.yaml
```

>*📃output*
```
ec2-user@ip-172-31-5-5:~/kubernetes-workshop-online> kubectl apply -f petclinic.yaml
deployment.apps/petclinic-backend created                                                                                                   
service/petclinic-backend created                                                                                                           
deployment.apps/petclinic-frontend created                                                                                                  
service/petclinic-frontend created                                                                                                          
ingress.networking.k8s.io/petclinic-ingress created 
```

就像之前使用的那样，我们可以用 watch 命令来查看应用程序相关 Pod 在K8s集群里的部署情况。同样的，按 `Ctrl + C` 键退出 watch 命令。

```
watch kubectl get pods
```

>*📃output*
```
Every 2.0s: kubectl get pods                                           ip-172-31-5-5.eu-central-1.compute.internal: Tue Nov 17 15:28:10 2020
                                                                                                                                            
NAME                                                              READY   STATUS      RESTARTS   AGE                                        
cass-operator-cd9b57568-hcqc6                                     1/1     Running     0          26m                                        
grafana-deployment-cfc94cf66-n7jk4                                1/1     Running     0          21m                                        
k8ssandra-cluster-a-grafana-operator-k8ssandra-6466cf94c9-skzrs   1/1     Running     0          23m                                        
k8ssandra-cluster-a-reaper-k8ssandra-59cb88b674-lh6cx             1/1     Running     0          20m                                        
k8ssandra-cluster-a-reaper-k8ssandra-schema-2p2tp                 0/1     Completed   4          23m                                        
k8ssandra-cluster-a-reaper-operator-k8ssandra-56cc9bf47c-9nt2l    1/1     Running     0          23m                                        
k8ssandra-dc1-default-sts-0                                       2/2     Running     0          23m                                        
k8ssandra-tools-kube-prome-operator-6d556b76f8-pqbmt              1/1     Running     0          26m                                        
petclinic-backend-7d47bcc6cc-smmv7                                1/1     Running     0          59s                                        
petclinic-frontend-75b98f7f8d-x2zgk                               1/1     Running     0          59s                                        
prometheus-k8ssandra-cluster-a-prometheus-k8ssandra-0             2/2     Running     1          23m                                        
traefik-7877ff76c9-rcm9n                                          1/1     Running     0          27m                                        
```

**✅ 步骤 2b: 使用 PetClinic 应用程序**

在所提供的云环境页面上选择 “Petclinic 演示程序” 链接。如果一切正常，你应该可以看到以下的页面：

![images](./Images/home-petclinic.png)

*如果你是用的是自己的本机环境，在浏览器中输入 localhost:8080 来访问 PetClinic 应用程序的首页*
![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic1.png)

*在页面最上面选择 _pet types_ 菜单。*
![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic2.png?raw=true)

*点击 _add_ 按钮，输入新的宠物类型。*
![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic4.png?raw=true)

*点击 _delete_ 按钮。*
![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic5.png?raw=true)

PetClinic 应用程序的原始代码可以在以下 Github 代码库找到： [这里](https://github.com/spring-petclinic/spring-petclinic-reactive). 在今天的实操演示课程里，我们使用的是自己的分叉版本。 

## 3. Cassandra集群向上扩容和向下缩容

**✅ 步骤 3a: 获取当前运行环境的基本配置**

对一些基本的配置选项，你可以通过改变 `demo-values.yaml` 文件的方式来实现。在下面的步骤中，我们用这种方法来实现集群的扩容和缩容。

运行以下的命令来查看当前运行环境的所有配置设定。

```
helm get manifest k8ssandra-cluster-a
```

请注意上面的命令返回和当前运行环境相关的所有 yaml 文件（有很多）。K8s中包含了许多的 yaml 文件. 

**✅ 步骤 3b: Cassandra集群向上扩容**

我们可以用 Linux 命令 `grep` 来过滤相关 Helm 命令的输出以找到我们感兴趣的一些特定的数据。比如说，我们可以使用下面的命令来得到当前集群中 Cassandra 节点的数目。
```
helm get manifest k8ssandra-cluster-a | grep size -m 1
``` 

注意上面命令行的输出是 `size: 1`。

这就是当前运营环境中 Cassandra 节点的数目。下一步，我们会把集群中Cassandra 节点的数目由 1 扩容到 3。

有好几种方法可以实现这个目标。在这里我们使用以下的方法（一条命令行，不需要编辑任何配置文件）。

```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=3 -f demo-values.yaml
```

检查集群中Cassandra 节点的数目，这次应该显示 `size: 3`。

```
helm get manifest k8ssandra-cluster-a | grep size -m 1
```

**✅ 步骤 3c: Cassandra集群向下缩容**

从 Cassandra 历史上来讲，一个非常困难的课题就是 Cassandra 集群向下缩容。 

现在由于 K8ssandra 的动态弹性特征，这个问题已经大大简化来。Cassandra 集群向下缩容和集群向上扩容变得一样轻松。让我们一起来试一下！

```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=1 -f demo-values.yaml
```

检查集群中Cassandra 节点的数目，这次应该显示 `size: 1`。

```
helm get manifest k8ssandra-cluster-a | grep size -m 1
```

## 4. 运行Cassandra集群数据一致性修复操作

Cassandra 集群数据一致性修复操作是 Cassandra 集群运维管理中一个很重要的操作。过去很长时间中，关于怎样进行这项操作有很多不同的，独立于 Cassandra 安装之外的一些定制化的解决方案。如今在 K8ssandra 解决方案中, 它就自带了一个关于数据一致性修复操作的组件；这样就避免了引入定制化解决方案的必要。这个组件叫做 Reaper，它的存在使 K8ssandra Cassandra 集群数据一致性修复操作变得简单了许多。

**✅ 步骤 4a: 检查集群的健康状态**

在所提供的云环境页面上点击 Reaper 链接。

![images](./Images/home-repear.png)

如果你使用得是自己的本地环境，在浏览器中输入 repair.localhost:8080/webui/ 来打开 Reaper 的网页。

注意在 Reaper 页面上 Cassandra 节点是以集群 datacenter 来显示的。

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper1.png?raw=true)

节点的颜色总体上反映出每个节点当前工作负责的状况。 

**✅ 步骤 4b: 集群数据一致性修复操作的自动调度**

注意页面左面面板上的 Schedule 菜单选项。

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper2.png?raw=true)

点击 _Schedule_

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper3.png?raw=true)

点击 _add schedule_；填入所需详细信息；完成后点击 _add schedule_ 来激活新的自动调度任务。Cassandra 推荐每星期完成一个数据修复循环；这样做可以防止“僵尸数据”在被删除后重新出现。

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper4.png?raw=true)

注意到有一个新的数据一致性修复任务加到了数据列表中。

**✅ 步骤 4c:Run a cluster repair**

在你刚配置的数据修复任务上， 点击 _Run now_ 按钮立即开始任务运行。

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper5.png?raw=true)

注意到数据修复任务已经开始运行。

想要了解更多关于 Reaper 的知识，请参考[以下页面](https://medium.com/rahasak/orchestrate-repairs-with-cassandra-reaper-26094bdb59f6)

## 5. 相关资源

好的，今天就到此为止。由于 Kubernetes 和 Helm技术的帮助，我们在短短的一段时间内完成了以前需要一周左右时间才能完成的工作。

如果想进一步和我们团队学习，敬请关注[datastax.com/dev](datastax.com/dev)页面。我们在这个页面上会发布很多可以帮助你们继续学习和提高的资源。

如果你想成为 Cassandra 认证管理员或开发者, 敬请关注[https://datastax.com/dev/certifications](https://datastax.com/dev/certifications)页面。

如果你想参与到关于这个项目和其他一些项目的讨论，敬请关注[community.datastax.com](community.datastax.com)页面。

如果你想更多的了解 K8ssandra, 敬请关注[k8ssandra.io/preview](k8ssandra.io/preview)页面，还有项目的 Github 资料库：[github.com/k8ssandra/k8ssandra](github.com/k8ssandra/k8ssandra)。
