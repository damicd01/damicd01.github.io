**Deploying the minikube cluster and installing flux**

To deploy the minikuke cluster I first deleted any old traces of minikube installations:

```bash
minikube stop
minikube delete
rm -rf ~/.minikube
```

I then ran 

```bash
minikube start --driver=hyperkit --cpus 4 --memory 8100 --disk-size 100g
```

This will create the following file which will the file that you use to authenticate against the kube-apiserver when running ```kubectl``` commands:

```bash
~/.kube/config
```

Before I started installing any K8s objects I created an alias for ```kubectl``` in my ```~/.zshrc``` like so:

```bash
alias k='kubectl'
```

I then installed ```kubens```

<a href="https://webinstall.dev/kubens/">kubens</a>

