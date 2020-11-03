# KubeCon 2020
##  Putting Apache Cassandra on Automatic with Kubernetes™

![OK](https://github.com/DataStax-Academy/kubecon-cassandra-workshop/blob/master/3-materials/images/00-screenplay.png?raw=true)

In this repository, you'll find everything for the Cassandra Kubernetes Workshop:
- Materials used during presentations
- Hands-on exercises

Feel free to bookmark this page for future reference!

## Materials and Communications

* [TBD Workshop recording](https://youtu.be/nRf2M4OjGpU)
* [TBD  Presentation](3-materials/presentation.pdf)
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
helm repo add datastax https://datastax.github.io/charts
helm repo update
```

### ✅  Helm 3 Install
```
helm install cass-operator datastax/TODO
```

### ✅  Monitor things as they come up
Navagate to http://localhost:9090 to start seeing metrics

# 2. Working with data

### Deploy Pet Clinic App
https://github.com/DataStax-Academy/kubecon2020#Scaling-up-and-down


# 3. Scaling up and down
### ✅  Get current running config
For many basic config options you can change values in the values.yaml file.  Next we will scale our cluster using this method.

First lets check what our current running values are using the `helm get manifest k8ssandra` command.  This command is used to expose all the current running values in the system. 

In the command line type `helm get manifest k8ssandra` and press enter

Notice how each of the yaml files that make up the deployment is displayed here

### ✅  Scale the cluster up
In the command line type the following
```
helm get manifest k8ssandra | grep size
``` 

Notice the value of `size: 2` in the TODO.yaml .

Open the values.yaml and find the nodeCount paramater.

Change the value from 2 to 3 `nodeCount: 3`.

Next we need to apply the change.  To do this we will use the `helm upgrade` command.

 ```
 helm upgrade
 helm get manifest k8ssandra | grep size
 ``` 

Notice the `size: 3` in the output

### Scale the cluster down
If there is a need to make a config change without needing to edit a file the --set flag can be used from the CLI. Run the following command

✅  
```
helm upgrade --set nodeCount=2
helm get manifest k8ssandra | grep size
```

Notice the `size: 2` in the output again.

# 4. Running repairs

# 5. Backing up and Restoring data
