# How to Contribute to the OpenTelemetry Operator

We'd love your help!

This project is [Apache 2.0 licensed](LICENSE) and accepts contributions via GitHub pull requests. This document outlines some of the conventions on development workflow, contact points and other resources to make it easier to get your contribution accepted.

We gratefully welcome improvements to documentation as well as to code.

## Getting Started

### Workflow

It is recommended to follow the ["GitHub Workflow"](https://guides.github.com/introduction/flow/). When using [GitHub's CLI](https://github.com/cli/cli), here's how it typically looks like:

```
$ gh repo fork github.com/open-telemetry/opentelemetry-operator
$ git checkout -b your-feature-branch
# do your changes
$ git commit -sam "Add feature X"
$ gh pr create
```

### Pre-requisites
* Install [Go](https://golang.org/doc/install).
* Have a Kubernetes cluster ready for development. We recommend `minikube` or `kind`.

### Local run

Build the manifests, install the CRD and run the operator as a local process:
```
$ make manifests install run
```

### Deployment with webhooks

When running `make run`, the webhooks aren't effective as it starts the manager in the local machine instead of in-cluster. To test the webhooks, you'll need to:

1. configure a proxy between the Kubernetes API server and your host, so that it can contact the webhook in your local machine
1. create the TLS certificates and place them, by default, on `/tmp/k8s-webhook-server/serving-certs/tls.crt`. The Kubernetes API server has also to be configured to trust the CA used to generate those certs.

In general, it's just easier to deploy the manager in a Kubernetes cluster instead. For that, you'll need the `cert-manager` installed. You can install it by running:

```console
make cert-manager
```

Once it's ready, the following can be used to build and deploy a manager, along with the required webhook configuration:

```
make manifests docker-build docker-push deploy
```

By default, it will generate an image following the format `quay.io/${USER}/opentelemetry-operator:${VERSION}`. You can set the following env vars in front of the `make` command to override parts or the entirety of the image:

* `IMG_PREFIX`, to override the registry, namespace and image name (`quay.io`)
* `USER`, to override the namespace
* `IMG_REPO`, to override the repository (`opentelemetry-operator`)
* `VERSION`, to override only the version part
* `IMG`, to override the entire image specification

Your operator will be available in the `opentelemetry-operator-system` namespace.

## Testing

With an existing cluster (such as `minikube`), run:
```
USE_EXISTING_CLUSTER=true make test
```

Tests can also be run without an existing cluster. For that, install [`kubebuilder`](https://book.kubebuilder.io/quick-start.html#installation). In this case, the tests will bootstrap `etcd` and `kubernetes-api-server` for the tests. Run against an existing cluster whenever possible, though.

## Project Structure

Here's a general overview of the directories from this operator and what to expect in each one of them:

```
.
├── api               # the CRDs that are handled by this operator
│   └── v1alpha1      # the (currently) only CRD version we have
├── build             # where the binaries are placed after `make manager`
├── config            # the Kustomize resources that are used to assemble the operator's deployment units
│   ├── certmanager   # Kustomize options dealing with cert-manager
│   ├── crd           # Kustomize options for our CRDs
│   │   ├── bases     # auto generated based on the code annotations (`make manifests`)
│   │   └── patches   # patches to apply to the generated CRD
│   ├── default       # Kustomize's "entry point", generating the distribution YAML file
│   ├── manager       # the operator's Deployment
│   ├── manifests     # the resulting CSV
│   │   └── bases
│   ├── prometheus    # ServiceMonitor that exposes our operator's metrics
│   ├── rbac          # RBAC rules
│   ├── samples       # Set of examples of how to accomplish specific scenarios. Those are bundled in the final CSV
│   └── webhook       # Webhook configuration and service
├── controllers       # our main controller, where the reconciliation starts
├── hack              # utility scripts
├── internal          # code shared with our internal packages
│   ├── config        # our operator's runtime configuration
│   ├── podinjector   # the webhook that injects sidecars into pods
│   └── version       # our version, as well as versions of underlying components
├── local             # local resources that shouldn't be added to the version control, like CRs used for dev/testing
└── pkg               # packages that are exporter and are part of the public API for this module
    ├── autodetect    # auto detect traits from the environment (platform, APIs, ...)
    ├── collector     # code that handles OpenTelemetry Collector
    │   ├── adapters  # data conversion
    │   ├── parser    # parses the OpenTelemetry Collector configuration
    │   ├── reconcile # reconciliation logic for OpenTelemetry Collector components
    │   └── upgrade   # handles the upgrade routine from one OpenTelemetry Collector to the next
    ├── naming        # determines the names for components (containers, services, ...)
    ├── platform      # target platforms this operator might run
    └── sidecar       # operations related to sidecar manipulation (add, update, remove)
```

## Contributing

Your contribution is welcome! For it to be accepted, we have a few standards that must be followed.

### New features

Before starting the development of a new feature, please create an issue and discuss it with the project maintainers. Features should come with documentation and enough tests (unit and/or end-to-end).

### Bug fixes

Every bug fix should be accompanied with a unit test, so that we can prevent regressions.

### Documentation, typos, ...

They are mostly welcome!
