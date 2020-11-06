# Install Docker
## mac
Go to the followign address 
https://hub.docker.com/editions/community/docker-ce-desktop-mac/
Click the button that says get docker and run the install.

Once the install is finished launch docker
## pc
TODO
# Install Kind

## mac
If homebrew is installed simply run 
```
brew install kind
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