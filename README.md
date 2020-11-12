# KubeCon 2020
##  Putting Apache Cassandra on Automatic with Kubernetes™

TODO UPDATE PICTURE
![OK](https://github.com/DataStax-Academy/kubecon-cassandra-workshop/blob/master/3-materials/images/00-screenplay.png?raw=true)

In this repository, you'll find everything for the Cassandra Kubernetes Workshop:
- Materials used during presentations
- Hands-on exercises

Feel free to bookmark this page for future reference!

## Materials and Communications

* [TODO Workshop recording](https://youtu.be/nRf2M4OjGpU)
* [TODO  Presentation](3-materials/presentation.pdf)
* [Discord chat](https://bit.ly/cassandra-workshop)
* [Q&A: community.datastax.com](https://community.datastax.com)

## Exercises

To follow along with the hands-on exercises during the workshop, you need to have a docker-ready machine with at least a 4-core + 8 GB RAM. **KubeCon attendees can request a training cloud instance** using [this link](https://kubecon2020.datastaxtraining.com/). Notice that training cloud instances will be available only during the workshop and to be terminated 24 hours later.

* If you are using **your own computer** or your own cloud node, please check the requirements and install the missing tools as explained [TODO NEED TO ADD here](./0-setup-your-cluster-own).
* If you are using **a cloud training instance** provided by DataStax, relax as we have you covered: prerequisites are installed already. Please do the very last steps as explained [there](./0-setup-your-cluster-datastax)

| Title  | Description
|---|---|
| **1 - Setting up and Monitoring Cassandra** | [Instructions](#1-Setting-up-and-Monitoring-Cassandra)  |
| **2 - Working with data** | [Instructions](#2-Working-with-data)  |
| **3 - Scaling up and down** | [Instructions](#3-Scaling-up-and-down)  |
| **4 - Running repairs** | [Instructions](#4-Running-repairs)  |
| **5 - Backing up and Restoring data** | [Instructions](#5-Backing-up-and-Restoring-data)  |

# 1. Setting up and Monitoring Cassandra

### ✅  Setup the Repo
```
helm repo add k8ssandra https://helm.k8ssandra.io/
helm repo update
```

### ✅  Helm 3 Install
```
helm install k8ssandra-tools k8ssandra/k8ssandra
helm install k8ssandra-cluster-a k8ssandra/k8ssandra-cluster
```

### ✅  Monitor things as they come up
Navigate to http://localhost:9090 to start seeing metrics

# 2. Working with data

### Deploy Pet Clinic App
TODO 
https://github.com/spring-petclinic/spring-petclinic-reactive


# 3. Scaling up and down
### ✅  Get current running config
For many basic config options you can change values in the values.yaml file.  Next we will scale our cluster using this method.

First lets check what our current running values are using the `helm get manifest k8ssandra` command.  This command is used to expose all the current running values in the system. 

In the command line type `helm get manifest k8ssandra` and press enter

Notice how each of the yaml files that make up the deployment is displayed here

### ✅  Scale the cluster up
In the command line type the following
```
helm get manifest k8ssandra-cluster | grep size
``` 

Notice the size is set to one. To grow the cluster, we just need to update the size value and let kubernetes find the new state. 

 ```
 helm upgrade k8ssandra-cluster k8ssandra/k8ssandra-cluster --set size=2
 helm get manifest k8ssandra | grep size
 ``` 

Notice the `size: 2` in the output

### ✅  Scale the cluster down
If there is a need to make a config change without needing to edit a file the --set flag can be used from the CLI. Run the following command

```
helm upgrade k8ssandra-cluster k8ssandra/k8ssandra-cluster --set size=1
helm get manifest k8ssandra | grep size
```

Notice the `size: 2` in the output again.

# 4. Running repairs
TODO
# 5. Backing up and Restoring data
TODO
