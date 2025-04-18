# Caddy Ingress Controller

This is the Kubernetes Ingress Controller for Caddy. It includes functionality
for monitoring `Ingress` resources on a Kubernetes cluster and includes support
for providing automatic HTTPS certificates for all hostnames defined in the 
ingress resources that it is managing.

## Prerequisites

- Helm 3+
- Kubernetes 1.19+

## Setup

In the `charts` folder, a Helm Chart is provided to make installing the Caddy
Ingress Controller on a Kubernetes cluster straightforward. To install the
Caddy Ingress Controller adhere to the following steps:

1. Create a new namespace in your cluster to isolate all Caddy resources.

```sh
kubectl create namespace caddy-system
```

2. Install the Helm Chart.

```sh
helm install \
  --namespace=caddy-system \
  --repo https://caddyserver.github.io/ingress/ \
  --atomic \
  mycaddy \
  caddy-ingress-controller
```

Or 

2. Generate kubernetes yaml file.
```sh
git clone https://github.com/caddyserver/ingress.git
cd ingress

# generate the yaml file
helm template mycaddy ./charts/caddy-ingress-controller \
  --namespace=caddy-system \
  > mycaddy.yaml

# apply the file
kubectl apply -f mycaddy.yaml
```

This will create a service of type `LoadBalancer` in the `caddy-system`
namespace on your cluster. You'll want to set any DNS records for accessing this
cluster to the external IP address of this `LoadBalancer` when the external IP
is provisioned by your cloud provider.

You can get the external IP address with `kubectl get svc -n caddy-system`

3. Alternate installation method: Glasskube

To install the Caddy ingress controller using [Glasskube](https://glasskube.dev/), you can select "caddy-ingress-controller" from the "ClusterPackages" tab in the Glasskube GUI then click "install" or you can run the following command: 
```console
glasskube install caddy-ingress-controller
```
Add an email address in the package configuration section in the UI to enable automatic HTTPS, or run: 
```
glasskube install caddy-ingress-controller --value "automaticHTTPS=your@email.com"
```

## Debugging

To view any logs generated by Caddy or the Ingress Controller you can view the
pod logs of the Caddy Ingress Controller.

Get the pod name with:

```sh
kubectl get pods -n caddy-system
```

View the pod logs:

```sh
kubectl logs <pod-name> -n caddy-system
```

## Automatic HTTPS

To have automatic HTTPS (not to be confused with `On-demand TLS`), you simply have
to specify your email in the config map. When using Helm chart, you can add
`--set ingressController.config.email=your@email.com` when installing.

## On-Demand TLS

[On-demand TLS](https://caddyserver.com/docs/automatic-https#on-demand-tls) can generate SSL certs on the fly
and can be enabled in this controller by setting the `onDemandTLS` config to `true`:

```sh
helm install ...\
  --set ingressController.config.onDemandTLS=true
```

> You can also specify options 
> for the on-demand config: `onDemandAsk`


## Bringing Your Own Certificates

If you would like to disable automatic HTTPS for a specific host and use your
own certificates you can create a new TLS secret in Kubernetes and define what
certificates to use when serving your application on the ingress resource.

Example:

Create TLS secret `mycerts`, where `./tls.key` and `./tls.crt` are valid
certificates for `test.com`.

```
kubectl create secret tls mycerts --key ./tls.key --cert ./tls.crt
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example
  annotations:
    kubernetes.io/ingress.class: caddy
spec:
  rules:
  - host: test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 8080
  tls:
    - secretName: mycerts # use mycerts for host test.com
      hosts:
        - test.com
```

### Contribution

Learn how to start contributing on the [Contributing Guidline](CONTRIBUTING.md).

## License

[Apache License 2.0](LICENSE.txt)
