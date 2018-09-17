# Istio Demo

## 1. Installing Istio on Kubernetes
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
helm install install/kubernetes/helm/istio --name istio --namespace istio-system --set global.mtls.enabled=true --set tracing.enabled=true
```

## 2. Setup BookInfo Sample Apps
Enable Istio Sidecar Injection:
```
kubectl label namespace default istio-injection=enabled
```

Deploy BookInfo sample apps to Kubernetes:
```
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

## 3. Setup Initial Routing for BookInfo
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

## 4. Access BookInfo Sample Apps
Get External IP associated with istio ingress LoadBalancer service.
```
kubectl get svc -n istio-system istio-ingressgateway
```

Open a browser and navigate to `{EXTERNAL_IP}/productpage`.

