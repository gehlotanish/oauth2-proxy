# External OAUTH Authentication

### Overview

The `auth-url` and `auth-signin` annotations allow you to use an external
authentication provider to protect your Ingress resources.

Sample:

```yaml
...
metadata:
  name: application
  annotations:
    nginx.ingress.kubernetes.io/auth-url: "https://$host/oauth2/auth"
    nginx.ingress.kubernetes.io/auth-signin: "https://$host/oauth2/start?rd=$escaped_request_uri"
...
```

### Example: OAuth2 Proxy + Sample app

This example will show you how to deploy [`oauth2_proxy`](https://github.com/pusher/oauth2_proxy)
into a Kubernetes cluster and use it to protect the Sample App using oidc as oAuth2 provider

#### Prepare
1. Create a rbac role in kubernetes cluster.

```console
kubectl create -f rbac.yaml
```

2. Install nginx ingress controller by helm

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

helm repo add stable https://kubernetes-charts.storage.googleapis.com/

helm repo update

helm install nginx-ingress stable/nginx-ingress     --namespace default     --set controller.replicaCount=2     --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux     --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

3. Install the Sample App

```console
kubectl create -f app.yaml
```

4. Configure oauth2_proxy values in the file [`oidc.yaml`](https://pusher.github.io/oauth2_proxy/auth-configuration) with the values:

- OAUTH2_PROXY_CLIENT_ID with the oauth `<Client ID>`
- OAUTH2_PROXY_CLIENT_SECRET with the oauth `<Client Secret>`
- OAUTH2_PROXY_COOKIE_SECRET with value of `docker run -ti --rm python:3-alpine python -c 'import secrets,base64; print(base64.b64encode(base64.b64encode(secrets.token_bytes(16))));'`
- OAUTH2_PROXY_REDIRECT_URL with oauth `<the OAuth Redirect URL. ie: "https://internalapp.yourcompany.com/oauth2/callback">`
- OAUTH2_PROXY_OIDC_ISSUER_URL with oauth `<the OpenID Connect issuer URL. ie: "https://xyz.okta.com/oauth2/default">`
- OAUTH2_PROXY_UPSTREAM with oauth `<the http url(s) of the upstream endpoint, file:// paths for static files or static://<status_code> for static response. Routing is based on the path>`
- OAUTH2_PROXY_COOKIE_SECURE with oauth `<Default value 'True' for https, For http configuration it should be 'False'>`

5. Customize the contents of the file [`ingress-route.yaml`](https://raw.githubusercontent.com/gehlotanish/oauth2-proxy/ingress-route.yaml):

Create TLS

```console
kubectl create secret tls tls.oidc.example.com  --namespace default --key privkey1.pem --cert fullchain1.pem
```

Replace `tls.oidc.example.com` with a Secret with a valid SSL certificate.

Replace `oidc.example.com` with your valid FDQN.  


6. Deploy the oauth2 proxy and the ingress rules running:

```console
$ kubectl create -f oidc.yaml,ingress-route.yaml
```

Test the oauth integration accessing the configured URL, like `https://oidc.example.com`
