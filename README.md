# Intro to Istio's ingress gateway

This repo contains sample code that introduces you to Istio's ingress gateway.

## It's "ingress", not "Ingress"
Note that the "i" in "ingress" is lower case. This is because we want to make it clear that we're **not** referring to the Kubernetes [`Ingress`](https://kubernetes.io/docs/concepts/services-networking/ingress/) object. In fact, even though the Istio exposes a public IP address to allow ingress into the cluster, it does not use an `Ingress` object — instead it just uses a `Service` object of type, `LoadBalancer` (which your cluster will generate an external IP address for).

## It's "gateway", not "Gateway"

Similarly, we use a lower case "g" in "gateway" to make it clear that we're not referring to the Istio [`Gateway`](https://istio.io/latest/docs/reference/config/networking/gateway/) object or the Kubernetes [`Gateway`](https://gateway-api.sigs.k8s.io/) object.

## Set up Istio and its ingress gateway

**1.** Install and Istio's ingress gateway

```bash
istioctl install
```

Since we haven't specified an Istio configuration profile (for instance, by using `istioctl install --set profile=minimal`), the [`default` Istio profile](https://github.com/istio/istio/blob/1.13.4/manifests/profiles/default.yaml#L25) will be installed in your cluster. This `default` Profile [includes the Istio ingress gateway](https://github.com/istio/istio/blob/1.13.4/manifests/profiles/default.yaml#L25).

This means that there is now a `Deployment`,  a `Pod`, and a `Service` resource in the `istio-system` Namespace for the Istio ingress gateway.

**2.** Verify that the ingress gateway's `Deployment` exists:

```bash
kubectl get deployment istio-ingressgateway -n istio-system
```

Expected output:
```
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
istio-ingressgateway   1/1     1            1           48m
```

**3.** Verify that the ingress gateway's `Pod` exists:

```bash
kubectl get pod -n istio-system
```

Expected output:
```
NAME                                   READY   STATUS    RESTARTS   AGE
istio-ingressgateway-b7ffbd9c6-cb8xc   1/1     Running   0          48m
istiod-7c595445b6-fw467                1/1     Running   0          48m
```

**4.** Verify that the ingress gateway's `Service` exists:

```bash
kubectl get service istio-ingressgateway -n istio-system
```

Expected output:
```
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   10.112.11.198   12.345.678.1    15021:31000/TCP,80:30459/TCP,443:30693/TCP   48m
```

You should also see a value for the `Service`'s `EXTERNAL-IP` if your Kubernetes cluster supports external load balancing.

**5.** If you check the description of the Istio ingress gateway, you will see that the `Service` is of type, `LoadBalancer` — which is why an `EXTERNAL-IP` is generated.

```bash
kubectl describe service istio-ingressgateway -n istio-system
```

```
...
Type:                     LoadBalancer
...
```

## Deploy a "Hello, world!" app

**1.** Deploy the "Hello, world!" app.

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Expected output:
```
deployment.apps/hello-world-deployment created
service/hello-world-service created
```

**2.** Note that we are not generating a public IP address for the "Hello, world!" `Service`. Check that its `EXTERNAL-IP` is `<none>`.

```bash
kubectl get service hello-world-service
```

Expected output:
```
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
hello-world-service   ClusterIP   10.112.6.247   <none>        80/TCP    23m
```
