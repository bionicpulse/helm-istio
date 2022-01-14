# Managing Istio via Helm and ArgoCD

# Prerequisites

* Kubectl and K8s credentials to access kubernetes cluster
* Helm 3
* ArgoCD Instance

# Steps

## Apply the base chart

`k apply -f istio-base-app.yaml -n argocd`

## Apply the istiod chart

`k apply -f istio-istiod-app.yaml -n argocd`

## Apply the ingress chart

`k apply -f istio-ingress-app.yaml -n argocd`

## Apply the istiod canary preview chart

`k apply -f istio-istiod-canary-app.yaml -n argocd`

## Deploy Sample BookInfo chart

```
k label namespace default istio-injection=enabled

k apply -f https://raw.githubusercontent.com/istio/istio/release-1.12/samples/bookinfo/platform/kube/bookinfo.yaml
```

## Overview

While listed as "Alpha" features the Helm Charts successfully deploy Istio (note they currently target 1.12) without needing to use the operator or istioctl (note there are some caveats)

The argocd application successfuly uses the Helm charts to deploy resources overwriting parameters where necessary.  Note that with the exception of 1 or 2 resources (i.e mutating webhook) the argocd application does end up in a synched status.  

The canary deployment is also supported via the argocd application 

## Still Under Investigation

When migrating from a version if istio that was installed via istioctl or helm this will require deletion of the control plane (although any CRDs will be retained).  Need to test if we can use the canary approach here to do a side-by-side (though initial investigation does not indicate this is possible because the canary is related to the istio chart while cluster resources are related to the base chart)

We'll need to further investigate the state of the mutating webhook that is running versus the mutating webhook that is in the default helm chart (this can be addressed probably by overwriting parameter variables or if necessary a fork of the helm chart where we manually set the 1 or two items that are out of sync)

The use of `--defaultRevision` is used alongside the helm upgrade to perform resource validation (need to test /validate across versions )

## Is this a good choice to use in ArgoCD

This appears to be a good choice to use in argocd.  

These caveats will need to be addressed:

* There will be a few manual steps in regards to for example using istioctl to perform prechecks.

* In some cases when applying upgrades there may be the need to "prune" out of sync resources.


Istio Upgrade

* https://istio.io/latest/news/releases/1.10.x/announcing-1.10/upgrade-notes/
* https://istio.io/latest/news/releases/1.11.x/announcing-1.11/upgrade-notes/
* https://cloudnative.to/istio.io/news/releases/1.12.x/announcing-1.12/upgrade-notes/




kind delete cluster --name playground 
kind create cluster --name playground --config multi-config.yaml

./istio-1.9.5/bin/istioctl install --set profile=demo --set hub=gcr.io/istio-release -y | tee -a 1.9.5-instsall.log
./istio-1.9.5/bin/istioctl analyze -A
k delete deploy istio-egressgateway -n istio-system
kubectl label namespace default istio-injection=enabled
k apply -f https://raw.githubusercontent.com/istio/istio/release-1.12/samples/bookinfo/platform/kube/bookinfo.yaml
./istio-1.10.6/bin/istioctl upgrade --set hub=gcr.io/istio-release -y | tee -a 1.10.6-uprade.log
k rollout restart deployment
./istio-1.10.6/bin/istioctl analyze -A
./istio-1.11.5/bin/istioctl upgrade --set hub=gcr.io/istio-release -y | tee -a 1.11.5-uprade.log
k rollout restart deployment
./istio-1.10.6/bin/istioctl analyze -A
./istio-1.12.1/bin/istioctl upgrade --set hub=gcr.io/istio-release -y | tee -a 1.12.1-uprade.log
k rollout restart deployment
./istio-1.10.6/bin/istioctl analyze -A

