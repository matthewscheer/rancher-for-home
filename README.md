# Rancher 2.0 on your Desktop 

Don't have access to Cloud infrastructure? Maybe you would like to use Rancher for local development just like you do in production? 

No problem, you can install Rancher 2.0 on your desktop.

In this tutorial we will install Docker-for-Desktop Edge release and enable the built in Kubernetes engine to run your own personal instance of Rancher 2.0 on your desktop.

## Prerequisites 

For this guide you will need a couple of tools to manage and deploy to your local Kubernetes instance.

* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - Kubernetes CLI tool.
* [helm](https://docs.helm.sh/using_helm/#installing-helm) - Kubernetes manifest catalog tool.

## Docker-for-Desktop

The Edge install of Docker CE for Windows/Mac includes a basic Kubernetes engine. We can leverage it to install a local Rancher Server. Download and install from the Docker Store.

* [Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
* [Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)

### Docker Configuration

Sign into Docker then right click on the Docker icon in your System Tray and select `Settings`

#### Advanced Settings

In the `Advanced` section increase `Memory` to at least `4096 MB`. You may want to increase the number of `CPUs` assigned and the `Disk image max size` while you're at it.

![advanced](docs/images/advanced.png)

#### Enable Kubernetes

In the `Kubernetes` section, check the box to enable the Kubernetes API. Docker-for-Desktop will automatically create `~/.kube/config` file with credentials for `kubectl` to access your new local "cluster".

![kubernetes](docs/images/kubernetes.png)

Don't see a `Kubernetes` section? Check the `General` section and make sure you are running the Edge version.

#### Testing Your Cluster

Open terminal and test it out. Run `kubectl get nodes`. `kubectl` should return a node named `docker-for-desktop`.

```
> kubectl get nodes

NAME                 STATUS    ROLES     AGE       VERSION
docker-for-desktop   Ready     master    6d        v1.9.6
```

## Preparing Kubernetes

Docker-for-Desktop doesn't come with any extra tools installed.  We could apply some static YAML manifest files with `kubectl`, but rather than reinventing the wheel, we want leverage existing work from the Kubernetes community.  `helm` is the package management tool of choice for Kubernetes.

`helm` `charts` provide templating syntax for Kubernetes YAML manifest documents. With `helm` you can create configurable deployments instead of just using static files. For more information about creating your own catalog of deployments, check out the docs at https://helm.sh/

### Initialize Helm on your Cluster

`helm` installs the `tiller` service on your cluster to manage `chart` deployments. Since `docker-for-desktop` has RBAC enabled by default we will need to use `kubectl` to create a `serviceaccount` and `clusterrolebinding` so `tiller` can deploy to our cluster for us.

Create the `ServiceAccount` in the `kube-system` namespace.

```
kubectl -n kube-system create serviceaccount tiller
```

Create the `ClusterRoleBinding` to give the `tiller` account access to the cluster.

```
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

Finally use `helm` to initialize the `tiller` service

```
helm init --service-account tiller
```

> NOTE: This `tiller` install has full cluster access, and may not be suitable for a production environment. Check out the [helm docs](https://docs.helm.sh/using_helm/#role-based-access-control) for restricting `tiller` access to suit your security requirements.

### Add an Ingress Controller

Ingress controllers are used to provide L7 (hostname or path base) http routing from the outside world to services running in Kubernetes.

We're going to use `helm` to install the Kubernetes stable community `nginx-ingress` `chart`. This will create an ingress controller on our local cluster. 

The default options for the "rancher" `helm` `chart` is to use SSL pass-through back to the self-signed cert on the Rancher server pod. To support this we need to add the `--controller.extraArgs.enable-ssl-passthrough=""` option when we install the chart.

```
helm install stable/nginx-ingress --name ingress-nginx --namespace ingress-nginx --set controller.extraArgs.enable-ssl-passthrough=""
```

### Installing Rancher

We're going to use `helm` install Rancher.

The default install will use Rancher's built in self-signed SSL certificate.  You can check out all the options for this `helm` `chart` here: https://github.com/jgreat/helm-rancher-server

First add the `rancher-server` repository to `helm`

```
helm repo add rancher-server https://jgreat.github.io/helm-rancher-server/charts
```

Now install the `rancher` `chart`.

```
helm install rancher-server/rancher --name rancher --namespace rancher-system
```

### Setting `hosts` file

By default the Rancher server will listen on `rancher.localhost`. To access it we will need to set a `hosts` file entry so our browser can resolve the name.

* Windows - `c:\windows\system32\drivers\etc\hosts`
* Mac - `/etc/hosts`

Edit the appropriate file for your system and add this entry.

```
127.0.0.1 rancher.localhost
```

### Connecting to Rancher

Browse to https://rancher.localhost

Ignore the SSL warning and you should be greeted by the colorful Rancher login asking you to Set the Admin password.

![rancher](docs/images/rancher_login.png)

Congratulations you have your very own local instance of Rancher 2.0. You can add your application `charts` and deploy your apps just like production. - Happy Containering!

