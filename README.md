# KubeCon 2020
##  Putting Apache Cassandra on Automatic with Kubernetes™

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/banner.png?raw=true)

In this repository, you'll find everything for the Cassandra Kubernetes Workshop:
- Materials used during presentations
- Hands-on exercises

Feel free to bookmark this page for future reference!

## Materials and Communications

* [Workshop On YouTube](https://www.youtube.com/watch?v=DI1bJ1tggmk)
* [Presentation](https://github.com/DataStax-Academy/kubecon2020/blob/main/November%20KubeCon%20Cassandra%20Kubernetes%20Workshop.pdf)
* [Discord chat](https://bit.ly/cassandra-workshop)
* [Q&A: community.datastax.com](https://community.datastax.com)

## Exercises

To follow along with the hands-on exercises during the workshop, you need to have a docker-ready machine with at least a 4-core + 8 GB RAM. **KubeCon attendees can request a training cloud instance** using [this link](https://kubecon2020.datastaxtraining.com/). Notice that training cloud instances will be available only during the workshop and will be terminated 24 hours later. Username `ec2-user`, Password `datastax`.

* If you are in our workshop we recommend using the provided cloud instance.  If you are doing this on your own using **your own computer** or your own cloud node, please check the requirements and install the missing tools as explained [Here](https://github.com/DataStax-Academy/kubecon2020/blob/main/setup_local.md).
* If you are using **a cloud training instance** provided by DataStax, relax as we have you covered: prerequisites are installed already.

* IMPORTANT NOTE.  Everywhere in this repo you see <YOURADDRESS> replace with the URL for the instance you were given.

| Title  | Description
|---|---|
| **1 - Setting Up and Monitoring Cassandra** | [Instructions](#1-Setting-up-and-Monitoring-Cassandra)  |
| **2 - Working with Data** | [Instructions](#2-Working-with-data)  |
| **3 - Scaling Up and Down** | [Instructions](#3-Scaling-up-and-down)  |
| **4 - Running Repairs** | [Instructions](#4-Running-repairs)  |
| **5 - Resources** | [Instructions](#5-Resources)  |

# 1. Setting Up and Monitoring Cassandra
First things first.  Helm is kind of like a high power package manager for Kubernetes.  In order to use the packages for todays workshop we will need to first add the correct repositories for helm to use.

### ✅  Setup the Repo
```
helm repo add k8ssandra https://helm.k8ssandra.io/
helm repo update
```

In Kubernetes, network ports and services are most often handled by an Ingress controller. For today's lab the K8ssandra side of things will be using Traefik.  Let's install that now.
### ✅  Install Ingress
```
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik --create-namespace -f traefik.values.yaml
```

Finally, the step we have been waiting for. Lets install our Cassandra by running a helm install of K8ssandra.
### ✅  Use Helm to Install K8ssandra
Note that the long install command will be shortened down post release candidate as it will no longer need the ingress config specified. 

```
helm install k8ssandra-tools k8ssandra/k8ssandra
helm install k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set ingress.traefik.enabled=true --set ingress.traefik.repair.host=repair.${ADDRESS}  --set ingress.traefik.monitoring.grafana.host=grafana.${ADDRESS}  --set ingress.traefik.monitoring.prometheus.host=prometheus.${ADDRESS}
```

If you are using your own instances then replace _${ADDRESS}_ with _127.0.0.1_

Verify everything is up running.  We need to wait till everything has running or completed status before moving on. 
```
watch kubectl get pods
```

Notice that reaper-k8ssandra-schema will error out the first time but will recover.  This is a known issue in the current pre release.

To exit watch use _Ctrl + C_

From this command we will be able to see the pods as they come on line.  Notice the steps as they complete. 

### ✅  Monitor your system
Modern applications and systems require that you can monitor them. K8ssandra is no different and to that end provides us with built-in Grafana and Prometheus.

To find the UI for Grafana and Prometheus use the links page in your instance and click on the corresponding Grafana and Prometheus. 

If running on a local kind cluster navigate to prometheus.localhost:8080 and grafana.localhost:8080 

For Grafana the login is
username: `admin`
password: `secret`

# 2. Working with Data

### ✅  Deploy Pet Clinic App 

Make sure the Cassandra Operator is initialized by trying to extract the password.
```
kubectl get secret k8ssandra-superuser -o yaml | grep password | cut -d " " -f4 | base64 -d
```
If this command returns an error, wait a few seconds and try it again until you see the random password.

Add the Cassandra password to the PetClinic manifest by running the following sed script.
```
sed -i "s/asdfadsfasdfdasdfds/$(kubectl get secret k8ssandra-superuser -o yaml | grep password | cut -d " " -f4 | base64 -d)/g" petclinic.yaml
```

Deploy the PetClinic app by applying the manifest.
```
kubectl apply -f petclinic.yaml
```

We can watch our app come online with the same command we used before. Just like last time use _Ctrl + C_ to exit the watch.
```
watch kubectl get pods
```

Navigate to the petclinic link in your cloud instance page to interact with the pet clinic app.  If you have done everything correctly you should see the following.

If you are using your own infrastructure navigate to localhost:8080 to see the UI

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic1.png)

Click on the _pet types_ tab at the top of the page.

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic2.png?raw=true)

Click the _add_ button and enter a new pet type.

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic4.png?raw=true)

Click the _delete_ button next to "bird".

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic5.png?raw=true)

To see the original app, the Pet Clinic app Github repo is [here](https://github.com/spring-petclinic/spring-petclinic-reactive). But, we have forked our own version for today. 

# 3. Scaling Up and Down
### ✅  Get current running config
For many basic config options, you can change values in the _values.yaml_ file.  Next, we will scale our cluster using this method.

First, lets check our current running values using the `helm get manifest k8ssandra-cluster-a` command.  This command exposes all the current running values in the system. 

In the command line, type `helm get manifest k8ssandra-cluster-a` and press enter.

Notice how each of the yaml files that make up the deployment is displayed here.

### ✅  Scale the cluster up
We can use the linux tool _grep_ to filter the output to find specific values.  For example, to find the current number of Cassandra nodes we run the following command.
```
helm get manifest k8ssandra-cluster-a | grep size -m 1
``` 
Notice the value of `size: 1`.  This is the current number of cassandra nodes.  Next, we are going to scale up to three nodes. While there are a few ways to make this change with Helm, we will use a single line command that doesn't require any edits to files. 
```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=3 --set ingress.traefik.enabled=true --set ingress.traefik.repair.host=repair.${ADDRESS}  --set ingress.traefik.monitoring.grafana.host=grafana.${ADDRESS}  --set ingress.traefik.monitoring.prometheus.host=prometheus.${ADDRESS}
helm get manifest k8ssandra-cluster-a | grep size -m 1
```
Notice the `size: 3` in the output.

### ✅  Scale the cluster down
Historically, one of the most difficult operations with Cassandra has been to scale down a cluster.  With K8ssandra's dynamic elasticity, it is now just as easy as scaling up.  Let's try it!

```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=1 --set ingress.traefik.enabled=true --set ingress.traefik.repair.host=repair.${ADDRESS}  --set ingress.traefik.monitoring.grafana.host=grafana.${ADDRESS}  --set ingress.traefik.monitoring.prometheus.host=prometheus.${ADDRESS}
helm get manifest k8ssandra-cluster-a | grep size -m 1
```

Notice the `size: 1` in the output again.

# 4. Running Repairs
Repairs are a critical anti-entropy operation in Cassandra. In the past, there have been many custom solutions to manage them outside of your main Cassandra Installation. In K8ssandra, there is a tool called Reaper that eliminates the need for a custom solution. Just like K8ssandra makes Cassandra setup easy, Reaper makes configuration of repairs even easier.

### ✅  Check the cluster’s health
Navigate to your cloud instance link list and click on reaper.

If you are running this locally then navigate to repair.localhost:8080/webui/

Notice way that the nodes are displayed inside the datacenter for the cluster.

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper1.png?raw=true)

The color of the nodes indicates the overall load the nodes are experiencing at the current moment. 

### ✅  Schedule a cluster repair
On the left hand side, notice the schedule menu option.


![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper2.png?raw=true)

Click _Schedule_


![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper3.png?raw=true)

Click _add schedule_ and fill out the details when you are done click the final _add schedule_ to apply the new repair job.  A Cassandra best practice is to have one repair complete per week to prevent zombie data from coming back after a deletion. 


![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper4.png?raw=true)

Notice the new repair added to the list.

### ✅  Run a cluster repair
On the repair job you just configured, click the _Run now_ button.  


![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/reaper5.png?raw=true)

Notice the repair job kicking off.

For more reading on Reaper visit [this link](https://medium.com/rahasak/orchestrate-repairs-with-cassandra-reaper-26094bdb59f6)

# 5. Resources
Well that is it for today.  In just a short time we have done what would have taken the better part of a week to do in the past thanks to the power of Kubernetes and Helm.

For further learning from our team please checkout [datastax.com/dev](datastax.com/dev) where we keep many resources and hands on labs to help you improve your skill set. 

If you are looking to get certified on Cassandra please visit [https://datastax.com/dev/certifications](https://datastax.com/dev/certifications).

To get involved in the discussion around this project and others please check out [community.datastax.com](community.datastax.com).

To learn more about K8ssandra please checkout our website at [k8ssandra.io/preview](k8ssandra.io/preview) and our project github at [github.com/k8ssandra/k8ssandra](github.com/k8ssandra/k8ssandra)
