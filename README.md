# Grafana example for OpenShift User Workload Monitoring

This repo contains a Kustomize base and example overlay which deploys a
proof-of-concept Grafana instance which is configured to "plug-in" to OpenShift
Container Platform's [user workload monitoring stack][user-workload-monitoring].

This isn't quite production-ready, but if somebody were to go through and fix
the `TODO`s _and_ figure out a graceful way to ensure the manifests and image
tags still work with future OCP versions, it should be pretty solid. You also
need to patch in your own dashboards.

The generated manifests have been tested against OpenShift Container Platform
v4.5.8. Breaking changes in higher versions may also break this Grafana
instance.

## Why?

Because OpenShift's user workload monitoring stack is still a tech-preview,
there is no support for user-provided Grafana dashboards. Even though metrics
scraping and alerting is provided out-of-the-box, most of the
applications/operators I support still need a friendly dashboard to take up
screen real-estate :grin:.

## How does it work?

These manifests assume that Grafana should be installed into a different
namespace from your target application. In other words, Grafana will be deployed
to the `app-monitoring` namespace and will be given permission to query metrics
scraped from the `ns1` namespace.

It's important to understand how RBAC is used to enforce tenancy in OpenShift's
user workload monitoring stack (specifically how `kube-rbac-proxy` and
`prom-label-proxy` enforce namespace permissions):
- [OpenShift user workload monitoring enhancement proposal][uwm-proposal]

Other puzzle pieces which comprise this example:
- [OpenShift OAuth proxy][oauth-proxy], see:
  - `base/grafana/deployment.yaml`
  - `base/grafana/rbac.yaml`
  - `base/grafana/files/grafana.ini`
- [OpenShift Service Serving Signer][service-signer], see:
  - `base/grafana/service.yaml`
  - `base/grafana/tls-bundle-configmap.yaml`

## Install

Edit these `kustomization.yaml` files in the `overlay` directory and change the
target namespaces to match your application and monitoring namespace:
- `overlay/grafana.kustomization.yaml`
  - The namespace which you want Grafana installed to (`app-monitoring`)
  - Your target application's namespace (`ns1`)
  - Grafana's exposed Route hostname (`grafana.apps.my-cluster.company.com`)
- `overlay/rbac/kustomization.yaml`
  - Your application's namespace which Grafana will query metrics for (`ns1`)

**Note,** this POC doesn't include any Grafana dashboards, so you'll have to
patch your own in to a configmap and the Grafana deployment.


[user-workload-monitoring]:https://docs.openshift.com/container-platform/4.5/monitoring/monitoring-your-own-services.html
[oauth-proxy]:https://github.com/openshift/oauth-proxy
[service-signer]:https://docs.openshift.com/container-platform/4.5/security/certificates/service-serving-certificate.html
[uwm-proposal]:https://github.com/openshift/enhancements/blob/master/enhancements/monitoring/user-workload-monitoring.md