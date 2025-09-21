# Teo's Helm Charts# helm-charts

A collection of Helm charts

This is the GitHub Pages branch hosting Helm chart packages.

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/angelnu)](https://artifacthub.io/packages/search?repo=angelnu)

To add this repository:

## Goal

```bash

helm repo add teo https://teo.github.io/helm-chartsThis repo contains Helm charts that I have developed to run applications in my

helm repo update[home Kubernetes cluster](https://github.com/angelnu/k8s-gitops.git). It is

```based in [bjw-s](https://github.com/bjw-s/helm-charts) work.



## Available ChartsThis repo is not intended to be a replacement for any of the large collections

of Helm charts that are out there.

- `games-on-whales` - A Helm chart for Games on Whales streaming solution
## Installation

The Helm repository can be installed as follows:

```console
helm repo add angelnu https://angelnu.github.io/helm-charts
```

You can then run `helm search repo angelnu` to search the charts.

## Documentation

Documentation can be found [here](https://angelnu.github.io/helm-charts/docs/).

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md)

## License

[Apache 2.0 License](./LICENSE)
