# kubelogin [![CircleCI](https://circleci.com/gh/int128/kubelogin.svg?style=shield)](https://circleci.com/gh/int128/kubelogin)

`kubelogin` is a command to get an OpenID Connect (OIDC) token for `kubectl` authentication.


## Getting Started

Download [the latest release](https://github.com/int128/kubelogin/releases) and save it as `/usr/local/bin/kubelogin`.

Run the command.

```
% kubelogin
2018/03/23 18:01:40 Reading config from /home/user/.kube/config
2018/03/23 18:01:40 Using current context: hello.k8s.local
2018/03/23 18:01:40 Using issuer: https://keycloak.example.com/auth/realms/hello
2018/03/23 18:01:40 Using client ID: kubernetes
2018/03/23 18:01:41 Starting OpenID Connect authentication:

## Automatic (recommended)

Open the following URL in the web browser:

http://localhost:8000/

## Manual

If you cannot access to localhost, instead open the following URL:

https://keycloak.example.com/auth/realms/hello/protocol/openid-connect/auth?client_id=kubernetes&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=openid+email&state=********

Enter the code:
```

And then open http://localhost:8000.
If you cannot access to localhost, you can choose manual interaction instead.

`kubelogin` will update your `~/.kube/config` with the ID token and refresh token, as follows:

```yaml
# ~/.kube/config (snip)
current-context: hello.k8s.local
contexts:
- context:
    cluster: hello.k8s.local
    user: hello.k8s.local
  name: hello.k8s.local
users:
- name: hello.k8s.local
  user:
    auth-provider:
      config:
        idp-issuer-url: https://keycloak.example.com/auth/realms/hello
        client-id: kubernetes
        client-secret: YOUR_SECRET
        id-token: ey...       # kubelogin will update ID token
        refresh-token: ey...  # kubelogin will update refresh token
      name: oidc
```

Make sure you can access to the cluster:

```
% kubectl version
Client Version: version.Info{...}
Server Version: version.Info{...}
```


## Prerequisite

You have to setup your OIDC identity provider and Kubernetes cluster.

### 1. Setup OIDC Identity Provider

This tutorial assumes you have created an OIDC client with the following:

- Issuer URL: `https://keycloak.example.com/auth/realms/hello`
- Client ID: `kubernetes`
- Client Secret: `YOUR_CLIENT_SECRET`
- Allowed redirect URLs:
  - `http://localhost:8000/`
  - `urn:ietf:wg:oauth:2.0:oob`
- Groups claim: `groups` (optional for group based access controll)

### 2. Setup Kubernetes API Server

Configure the Kubernetes API server allows your identity provider.

If you are using kops, `kops edit cluster` and append the following settings:

```yaml
spec:
  kubeAPIServer:
    oidcClientID: kubernetes
    oidcGroupsClaim: groups
    oidcIssuerURL: https://keycloak.example.com/auth/realms/hello
```

### 3. Setup kubectl

Run the following script to configure `kubectl` uses your identity provider:

```sh
CLUSTER_NAME=hello.k8s.local

kubectl config set-credentials $CLUSTER_NAME \
  --auth-provider oidc \
  --auth-provider-arg idp-issuer-url=https://keycloak.example.com/auth/realms/hello \
  --auth-provider-arg client-id=kubernetes \
  --auth-provider-arg client-secret=YOUR_CLIENT_SECRET

# Set the context
kubectl config set-context $CLUSTER_NAME --cluster $CLUSTER_NAME --user $CLUSTER_NAME
```

In actual team operation, you can distribute the script to your team members for easy setup.


## Contributions

This is an open source software licensed under Apache License 2.0.
Feel free to open issues and pull requests.

### How to build

```sh
go get github.com/int128/kubelogin
```
