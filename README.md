# Flux traefik demo
Demo of GitOps using [Flux](https://fluxcd.io/)

[Kubernetes Documentation](https://kubernetes.io/docs/home/)

## Prerequisites
- local kubernetes cluster e.g. [k3d](https://k3d.io/)
- kubectl [Install Tools](https://kubernetes.io/docs/tasks/tools/)
- Helm [Installing Helm](https://helm.sh/docs/intro/install/)
- Flux CLI [Install the Flux CLI](https://fluxcd.io/flux/installation/)

If running on Windows it is recommended to use WSL [Install WSL on Windows](https://learn.microsoft.com/en-us/windows/wsl/install)

## Demo 1 - Run image on kubernetes cluster using kubectl
Create a local cluster using k3d
```
k3d cluster create traefik-staging-demo \
  --port 82:80@loadbalancer \
  --port 445:443@loadbalancer \
  --port 8002:8000@loadbalancer \
  --k3s-arg "--disable=traefik@server:0"
```

Install traefik using helm enabling the Gateway API provider
```
helm install traefik traefik/traefik --wait \
  --set ingressRoute.dashboard.enabled=true \
  --set ingressRoute.dashboard.matchRule='Host(`dashboard-staging.localhost`)' \
  --set ingressRoute.dashboard.entryPoints={web} \
  --set providers.kubernetesGateway.enabled=true \
  --set gateway.namespacePolicy=All
```

```
kubectl describe GatewayClass traefik
```

Traefik dashboard is exposed by the helm install
[http://dashboard-staging.localhost:82/dashboard/](http://dashboard-staging.localhost:82/dashboard/)

Test and deploy whoami 
```
kubectl apply -k apps/staging
```

```
curl http://whoami-staging.localhost:82
```

## Demo 2 - GitOps using Flux
For to use fluxcd you need to fork this repo and 
See [Flux bootstrap for GitHub](https://fluxcd.io/flux/installation/bootstrap/github/)
```
export GITHUB_TOKEN=<token>
export GITHUB_USER=<github-user>
export GITHUB_REPO=flux-traefik-demo
```

### Bootstrap Flux staging
```
flux bootstrap github \
    --context=k3d-traefik-staging-demo \
    --owner=${GITHUB_USER} \
    --repository=${GITHUB_REPO} \
    --branch=fluxcd \
    --personal \
    --path=clusters/staging
```


```
curl http://whoami-staging.localhost:82
```

## Demo 3 - Add production cluster
```
k3d cluster create traefik-demo \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --port 8000:8000@loadbalancer \
  --k3s-arg "--disable=traefik@server:0"
```

Install traefik using helm enabling the Gateway API provider
```
helm install traefik traefik/traefik --wait \
  --set ingressRoute.dashboard.enabled=true \
  --set ingressRoute.dashboard.matchRule='Host(`dashboard.localhost`)' \
  --set ingressRoute.dashboard.entryPoints={web} \
  --set providers.kubernetesGateway.enabled=true \
  --set gateway.namespacePolicy=All
```

Traefik dashboard is exposed by the helm install
[http://dashboard.localhost/dashboard/](http://dashboard.localhost/dashboard/)

### Bootstrap Flux production
```
flux bootstrap github \
    --context=k3d-traefik-demo \
    --owner=${GITHUB_USER} \
    --repository=${GITHUB_REPO} \
    --branch=fluxcd \
    --personal \
    --path=clusters/production
```


```
curl http://whoami.localhost
```

## Demo 4 - Upgrade to new version of whoami
change in apps/staging/whoami/whoami-patch.yaml
```
 image: traefik/traefikee-webapp-demo:v2
          args:
            - -ascii
            - -name=FOO
```

## Clean
Delete the created clusters
```
k3d cluster delete traefik-demo traefik-staging-demo
```
