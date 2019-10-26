# istio-demo-101
Demo at GDG devfest vn 2019

## Setup

[Codelab link](https://codelabs.developers.google.com/codelabs/cloud-hello-istio)

### Create K8s cluster

* Register GCP account with 300$ free credit.
* Create project GCP
* Enable GKE:

    ```bash
    gcloud services enable container.googleapis.com
    ```

    ```bash
    REGION=us-central1
    ```

* Init k8s cluster

PROJECT_ID=<your_project_id>
REGION=<choose_region> example: REGION=us-central1

   ```bash
    gcloud beta container clusters create hello-istio --project=$PROJECT_ID \
    --cluster-version=latest \
    --machine-type=n1-standard-2 \
    --num-nodes=4 \
    --region=$REGION
   ```

   You may have this error, exceed quota for network resource. You can go to the [console](https://console.cloud.google.com/iam-admin/quotas) to increase.

### Install Istio

```bash
helm-wrapper upgrade istio istio.io/istio --namespace istio-system --install --values istio.yaml
```

Verify

```bash
kubectl get svc -n istio-system
```

```bash
kubectl get pods -n istio-system
```

Get sample

```bash
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.3.3 sh -
```

```enable
kubectl label namespace default istio-injection=enabled
```

Go to Istio folder

```bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

```bash
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

```bash
kubectl get services
kubectl get pods
```

Get host ip

```bash
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
```

Get NodePort

```bash
kubectl get svc istio-ingressgateway -n istio-system
```

```bash
GATEWAY_URL=$INGRESS_HOST:30006
```

->open browser to view

```code
open http://$GATEWAY_URL/productpage
```

```bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
```

Route Review V1: no star

```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

Route Review V2 for user jason

```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

Shifting 50% to V3. -> Refresh multi times to see different star/ no star

```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
```

Clean up

```code
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl delete -f samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml
```
