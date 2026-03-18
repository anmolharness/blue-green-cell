# Blue Green Cell

This repo contains a simple Istio rollout pattern for Harness.

The intended model is:

- stable is applied locally first
- candidate is deployed from Harness
- both run in the `app` namespace
- one Kubernetes `Service` routes to both versions
- one Istio `DestinationRule` defines `stable` and `candidate` subsets
- one Istio `VirtualService` shifts traffic between those subsets
- Prometheus scrapes the app pods for verification

## Files

- `manifests/namespaces.yaml`
- `manifests/service.yaml`
- `manifests/stable.yaml`
- `manifests/candidate.yaml`
- `manifests/smoke-pod.yaml`
- `manifests/istio/gateway.yaml`
- `manifests/istio/destinationrule.yaml`
- `manifests/istio/virtualservice.yaml`
- `harness/pipeline.yaml`

## Local Baseline

Apply these first:

1. `manifests/namespaces.yaml`
2. `manifests/service.yaml`
3. `manifests/stable.yaml`
4. `manifests/istio/gateway.yaml`
5. `manifests/istio/destinationrule.yaml`
6. `manifests/istio/virtualservice.yaml`
7. `manifests/smoke-pod.yaml`

Install Prometheus in `monitoring` and make sure it can scrape pod metrics.

## Endpoints

- `/` returns release text
- `/healthz` returns `ok`
- `/metrics` is exposed on port `9113` by `nginx-prometheus-exporter`

## Harness Flow

Harness should:

1. deploy `manifests/candidate.yaml`
2. shift a small amount of traffic to `candidate`
3. run Continuous Verification using Prometheus
4. shift traffic to `100%` only if verification passes
