# OpenQ CertManager

## NOTE FOR SELF-CHECK ON DIGITAL OCEAN

You will have to add this annotation to the LoadBalancer service on DigitalOcean to enable Pod-Pod communication

This is needed for the HTTPS-01 self-check like:

`curl http://www.development.openq.dev/.well-known/acme-challenge/sP_ooP5vpdDcz53ExuLTaWxysPYCSEZmuXtXh_3l5fI`

to succeed from within the cluster.

This is the annotation:

`service.beta.kubernetes.io/do-loadbalancer-hostname: "openq.dev"`

## 1. Install Custom Resource Definitions (CRDs)

`kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.crds.yaml`

NOTE: Delete crds with:

`kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.crds.yaml`

## 2. Install cert-manager with Helm

```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.3
```

## 3. Create Cluster Issuer

`kubectl apply -f cluster-issuer.yml`

### .spec.acme.server

You can start off with using the ACME v2.0 Staging Environment:
https://letsencrypt.org/docs/staging-environment/

Then graduate to production once you've confirmed everything works as expected:
https://acme-v02.api.letsencrypt.org/directory

Otherwise, all the trial and error may cause you to hit your Let's Encrypt rate limit.

## 4. Annotate Ingress

On all `Ingress` resources, include the following annotation to have SSL certs automatically provisioned:

`cert-manager.io/cluster-issuer: openq-cluster-issuer`

## Additional Commands

### Get All Cert-Manager Resources
`kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces`

### Delete All Resources
`helm uninstall cert-manager -n cert-manager`
`kubectl delete Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces --all`