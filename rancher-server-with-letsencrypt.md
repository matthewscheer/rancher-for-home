# Bonus Points

## SSL with LetsEncrypt

If you can create a public DNS entry and your system is accessible from the Internet, you can use an honest-to-goodness-perfectly-trusted LetsEncrypt SSL certificate.

### DNS

Use your DNS provider of choice and create a record that points to your FQDN.

### Ingress
Configure `ingress-nginx` without the SSL Passthrough option.

```
helm install stable/nginx-ingress --name ingress-nginx --namespace ingress-nginx
```

### cert-manager
We need to install `cert-manager` to manage issuance and renewal of our LetsEncrypt certs.

```
helm install stable/cert-manager --name cert-manager --namespace kube-system
```

### Rancher

We need to provide some additional options to the `rancher` catalog to configure LetsEncrypt. Check out the catalog README.md for more details on the options.

```
helm install ./ --name rancher --namespace rancher-system \
--set fqdn=rancher.jgreat.me \
--set ingress.tls=letsEncrypt \
--set letsEncrypt.enabled=true \
--set letsEncrypt.email=test@jgreat.me \
--set letsEncrypt.environment=prod
```
