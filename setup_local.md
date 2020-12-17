# Install Docker
Go to the following address 
https://hub.docker.com/editions/community/docker-ce-desktop-mac/
Click the button that says get docker and run the install.

Once the install is finished launch docker

# Install Kind

## mac
If homebrew is installed simply run 

On Mac
```
brew install kind
brew install kubectl
```

On Linux
```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.9.0/kind-$(uname)-amd64
chmod +x ./kind
mv ./kind /usr/local/bin/kind
```

or to install without brew simply 
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.9.0/kind-darwin-amd64
```
and run the process

# Install helm
To install helm 

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Clone App Repo

```
git clone https://github.com/DataStax-Academy/kubecon2020/
```


## Start Your Kind Cluster
```
cd kubecon2020
kind create cluster --image kindest/node:v1.18.2 --config ./kind-config.yaml
```
