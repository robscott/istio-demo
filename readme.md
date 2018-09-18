# Istio Demo

## 1. Download Istio
Download the latest Istio release (1.0.2 when this was written):
```
curl -L https://git.io/getLatestIstio | sh -
```

cd into the istio directory that was just downloaded.
```
cd istio-1.0.2
```

## 2. Installing Istio on Kubernetes
Install CRDs:
```
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```

Setup Helm and Tiller:
```
kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
helm init --service-account tiller
```

Install Istio with Helm, enabling mTLS and Tracing:
```
helm install install/kubernetes/helm/istio --name istio --namespace istio-system --set global.mtls.enabled=true --set tracing.enabled=true --set grafana.enabled=true
```

## 3. Setup BookInfo Sample Apps
Enable Istio Sidecar Injection:
```
kubectl label namespace default istio-injection=enabled
```

Deploy BookInfo sample apps to Kubernetes:
```
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

## 4. Clone this Repository
The rest of this demo will be based on code found in this repository. Clone the repository and cd into it to continue:
```
git clone https://github.com/robscott/istio-demo.git
cd istio-demo
```

## 5. Setup Initial Routing for BookInfo
Deploy Destination Rules:
```
kubectl apply -f bookinfo/destinationrules.yaml
```

Deploy Gateway
```
kubectl apply -f bookinfo/gateway.yaml
```

Deploy Virtual Service
```
kubectl apply -f bookinfo/virtualservice.yaml
```

## 6. Access BookInfo Sample Apps
Get External IP associated with istio ingress LoadBalancer service.
```
kubectl get svc -n istio-system istio-ingressgateway
```

Open a browser and navigate to `{EXTERNAL_IP}/productpage`.

## 7. Lock Down Versions of Reviews Service
Lock down all traffic to the reviews service to go to version 1:
```
kubectl apply -f routing/all-v1.yaml
```

In your browser, refresh `{EXTERNAL_IP}/productpage`. Each request should now show reviews without any associated stars/ratings.

Make an exception for Joe to lock him to version 2 of the reviews service when he's logged in:
```
kubectl apply -f routing/joe-v2.yaml
```

## 8. Rate Limiting
Apply initial rate limiting configuration to limit requests to product page to 5 requests every 5 seconds:
```
kubectl apply -f ratelimiting/0-memquota.yaml
kubectl apply -f ratelimiting/1-quota.yaml
kubectl apply -f ratelimiting/2-quotaspec.yaml
kubectl apply -f ratelimiting/3-quotaspecbinding.yaml
kubectl apply -f ratelimiting/4-rule.yaml
```

Add an exception that removes rate limiting when anyone is logged in:
```
kubectl apply -f ratelimiting/5-rule-joe.yaml
```

