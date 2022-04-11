# OpenQ CertManager

Copied from [the official cert-manage installation instructions](https://cert-manager.io/docs/installation/helm/).

## Preparing Digital Ocean NGINX Ingress Deployment

The cert-manager uses the HTTPS-01 self-check like so:

`curl http://www.development.openq.dev/.well-known/acme-challenge/sP_ooP5vpdDcz53ExuLTaWxysPYCSEZmuXtXh_3l5fI`

"Intracluster calls" are treated differently by Digital Ocean, so some additional annotations must be added to the NGINX Ingress Deployment.

### Change into the `ingress-nginx` namespace

`kubectl config set-context --current --namespace ingress-nginx`

### Edit the `ingress-nginx-controller` deployment:

`kubectl edit deploy ingress-nginx-controller`

### Add necessary annotations

Add the following annotations:

```yaml
service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol: "true"
service.beta.kubernetes.io/do-loadbalancer-hostname: openq.dev
```

In the end, the `ingress-nginx-controller` yaml should look like:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    meta.helm.sh/release-name: ingress-nginx
    meta.helm.sh/release-namespace: ingress-nginx
    service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol: "true"
    service.beta.kubernetes.io/do-loadbalancer-hostname: openq.dev
...
```

### Delete the pods to force a restart of the NGINX Ingress Pods

```bash
kubectl get pods
kubectl delete pod <POD NAME>
```

## 1. Install Cert Manager Custom Resource Definitions (CRDs)

`kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.crds.yaml`

## 2. Install cert-manager with Helm

```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.8.0
```

## 3. Create Cluster Issuer

`kubectl apply -f cluster-issuer.yml`

### 3.1 Note on .spec.acme.server

You can start off with using the ACME v2.0 Staging Environment:
https://letsencrypt.org/docs/staging-environment/

Then graduate to production once you've confirmed everything works as expected:
https://acme-v02.api.letsencrypt.org/directory

## 4. Annotate Ingress

On all `Ingress` resources, include the following annotation to have SSL certs automatically provisioned:

`cert-manager.io/cluster-issuer: openq-cluster-issuer`

## Confirm 

## Helpful Commands

### Get All Cert-Manager Resources
`kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces`

### Delete All Resources and Custom Resource Definitions
`helm uninstall cert-manager -n cert-manager`
`kubectl delete Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces --all`
`kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.crds.yaml`
`kubectl delete ns cert-manager`