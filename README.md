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


```
export GITHUB_REPO=flux-traefik-demo
```
(also export token and user...)

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
curl http://whoami-staging.localhost
```

## Production
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