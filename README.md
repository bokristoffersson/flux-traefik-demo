Create a local cluster using k3d
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

```
kubectl describe GatewayClass traefik
```

Traefik dashboard is exposed by the helm install
[http://dashboard.localhost/dashboard/](http://dashboard.localhost/dashboard/)

Apply apps/staging...
```
kubectl apply -k apps/staging/
```

```
curl http://whoami-gatewayapi.localhost
```
