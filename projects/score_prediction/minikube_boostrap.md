**Initial pre-reqs**

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


**Flux bootstrap**

The flux documentation is here:

<a href="https://fluxcd.io/flux/get-started/">fluxcd</a>

The reason I wanted to use fluxcd is because it fairly light-weight, documentation is good and I wanted to leverage helm to install my K8s objects instead of creating multiple K8s yaml files and installing them individually.  Also I wanted a tool that would allow me to just make changes to the repo and once pushed and merged it would take care of the deployment for me.

As per the documentation I installed the flux cli:

```brew install fluxcd/tap/flux```