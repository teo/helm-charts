# helm-charts
A collection of Helm charts

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/teo)](https://artifacthub.io/packages/search?repo=teo)

## Goal

This repo contains Helm charts that I have developed to run applications in my
[home Kubernetes cluster](https://github.com/teo/k8s-gitops.git). It is
based in [bjw-s](https://github.com/bjw-s/helm-charts) work.

This repo is not intended to be a replacement for any of the large collections
of Helm charts that are out there.

## Installation

The Helm repository can be installed as follows:

```console
helm repo add teo https://teo.github.io/helm-charts
```

You can then run `helm search repo teo` to search the charts.

## Documentation

Documentation can be found [here](https://teo.github.io/helm-charts/docs/).

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md)

## License

[Apache 2.0 License](./LICENSE)
