# KubeCon 2020
##  Putting Apache Cassandra on Automatic with Kubernetes™

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/banner.png?raw=true)

In this repository, you'll find everything for the Cassandra Kubernetes Workshop:
- Materials used during presentations
- Hands-on exercises

Feel free to bookmark this page for future reference!

## Materials and Communications

* [Workshop On YouTube](https://www.youtube.com/watch?v=DI1bJ1tggmk)
* [TODO  Presentation](3-materials/presentation.pdf)
* [Discord chat](https://bit.ly/cassandra-workshop)
* [Q&A: community.datastax.com](https://community.datastax.com)

## Exercises

To follow along with the hands-on exercises during the workshop, you need to have a docker-ready machine with at least a 4-core + 8 GB RAM. **KubeCon attendees can request a training cloud instance** using [this link](https://kubecon2020.datastaxtraining.com/). Notice that training cloud instances will be available only during the workshop and will be terminated 24 hours later.

* If you are in our workshop we recommend using the provided cloud instance.  If you are doing this on your own using **your own computer** or your own cloud node, please check the requirements and install the missing tools as explained [Here](https://github.com/DataStax-Academy/kubecon2020/blob/main/setup_local.md).
* If you are using **a cloud training instance** provided by DataStax, relax as we have you covered: prerequisites are installed already. Please do the very last steps as explained [there](./0-setup-your-cluster-datastax)

* IMPORTANT NOTE.  Everywhere in this repo you see <YOURADDRESS> replace with the URL for the instance you were given 

| Title  | Description
|---|---|
| **1 - Setting up and Monitoring Cassandra** | [Instructions](#1-Setting-up-and-Monitoring-Cassandra)  |
| **2 - Working with data** | [Instructions](#2-Working-with-data)  |
| **3 - Scaling up and down** | [Instructions](#3-Scaling-up-and-down)  |
| **4 - Running repairs** | [Instructions](#4-Running-repairs)  |
| **5 - Backing up and Restoring data** | [Instructions](#5-Backing-up-and-Restoring-data)  |

# 1. Setting up and Monitoring Cassandra
First things first.  Helm is kind of like a high power package manager for Kubernetes.  In order to use the packages for todays workshop we will need to first add the correct repositories for helm to use.

### ✅  Setup the Repo
```
helm repo add k8ssandra https://helm.k8ssandra.io/
helm repo update
```

In Kubernetes network ports and services are most often handled by an Ingress controller. For todays lab the K8ssandra side of things will be using traefik.  Let's install that now.
### ✅  Install Ingress
```
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik --create-namespace -f traefik.values.yaml
```

Finally the step we have been waiting for lets install our Cassandra by running a helm install of K8ssandra.
### ✅  Helm 3 Install
```
helm install k8ssandra-tools k8ssandra/k8ssandra
helm install k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set ingress.traefik.enabled=true --set ingress.traefik.repair.host=repair.127.0.0.1.xip.io
```

Verify everything is up running
```
watch kubectl get pods
```
From this command we will be able to see the pods as they come on line.  Notice the steps completing. 

### ✅  Monitor your system
It is a requirement of modern applications and systems that you can monitor them.  K8ssandra is no different and to that end provides us with build in Grafana and Prometheus.

To find the UI for Grafana and Prometheus use the links page in your instance and click on the corisponding Grafana and Prometheus. 

# 2. Working with data

### ✅  Deploy Pet Clinic App 

Install the Nginx Ingress controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```


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

Navigate to the petclinic link in your cloud instance page to interact with the pet clinic app.  If you have done everything correctly you should see the following

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic1.png)

Click on the pet types tab at the top of the page

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic2.png?raw=true)

Click the add button and enter a new pet type

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic4.png?raw=true)

Click the delete button next to "bird"

![OK](https://github.com/DataStax-Academy/kubecon2020/blob/main/Images/petclinic5.png?raw=true)

To see the original app the Pet Clinic app Github is [here](https://github.com/spring-petclinic/spring-petclinic-reactive) but we have forked our own version for today. 

# 3. Scaling up and down
### ✅  Get current running config
For many basic config options you can change values in the values.yaml file.  We will not be using this method today but it is important to understand.  If you want to learn more the docs can be found [here](https://helm.sh/docs/chart_best_practices/values/)

First lets check what our current running values are using the `helm get manifest k8ssandra-cluster-a` command.  This command is used to expose all the current running values in the system. 

In the command line type `helm get manifest k8ssandra-cluster-a` and press enter

Notice how each of the yaml files that make up the deployment is displayed here

### ✅  Scale the cluster up
Filtering the output to find specific values can be done with the linux tool grep.  In order to find the current number of nodes we will run the following command.
```
helm get manifest k8ssandra-cluster-a | grep size
``` 
Notice the value of `size: 1`.  This is the current number of cassandra nodes.  Next we are going to scale this up to 3 nodes. While there are a few ways to make this change with Helm we will use a single line command that doesn't require any edits to files. 
```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=3
helm get manifest k8ssandra-cluster-a | grep size
```
Notice the `size: 3` in the output

### ✅  Scale the cluster down
One of the historically most difficult operations with Cassandra was to scale down the cluster.  With K8ssandra's dynamic elasticity it is now just as easy as scaling up.  Let's try it now.

```
helm upgrade k8ssandra-cluster-a k8ssandra/k8ssandra-cluster --set size=1
helm get manifest k8ssandra-cluster-a | grep size
```

Notice the `size: 1` in the output again.

# 4. Running repairs
Repairs are a critical anti-entropy operation in Cassandra. In the past there were many custom solutions to manage them outside of your main Cassandra Installation. In K8ssandra there is a tool called Reaper that eliminates the need for a custom solution. Just like K8ssandra makes Cassandra setup easy, Reaper makes configuration of repairs even easier.

### ✅  Check the cluster’s health
Navigate to the url TODO 

Notice way that the nodes are displayed inside the datacenter inside the cluster.

TODO NEED PICTURE

The color of the nodes indicates the overall load the nodes are experiencing at the current moment. 

### ✅  Schedule a cluster repair
On the left hand side notice the schedule menu option.

TODO NEED PICTURE

Click schedule and then set the time for once a week.  Cassandra best practise is to have one repair complete per week to prevent zombie data from coming back after a deletion. 

TODO VERIFY STEPS

Notice the new repair added to the list.

### ✅  Run a cluster repair
On the repair job you just configured click the run now button.  

TODO NEED PICTURE

Notice the repair job kicking off.


For more reading on reaper visit [this link](https://medium.com/rahasak/orchestrate-repairs-with-cassandra-reaper-26094bdb59f6)

# 5. Backing up and Restoring data
TODO
