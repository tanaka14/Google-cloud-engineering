# BuildKit

![GoDoc](https://godoc.org/github.com/moby/buildkit?status.svg)
![Build Status](https://github.com/moby/buildkit/workflows/build/badge.svg)
![Go Report Card](https://goreportcard.com/badge/github.com/moby/buildkit)
![codecov](https://codecov.io/gh/moby/buildkit/branch/master/graph/badge.svg)




- [Used by](#used-by)
- [Quick start](#quick-start)
  - [Starting the `buildkitd` daemon:](#starting-the-buildkitd-daemon)
  - [Exploring LLB](#exploring-llb)
  - [Exploring Dockerfiles](#exploring-dockerfiles)
    - [Building a Dockerfile with `buildctl`](#building-a-dockerfile-with-buildctl)
    - [Building a Dockerfile using external frontend:](#building-a-dockerfile-using-external-frontend)
    - [Building a Dockerfile with experimental features like `RUN --mount=type=(bind|cache|tmpfs|secret|ssh)`](#building-a-dockerfile-with-experimental-features-like-run---mounttypebindcachetmpfssecretssh)
  - [Output](#output)
    - [Image/Registry](#imageregistry)
    - [Local directory](#local-directory)
    - [Docker tarball](#docker-tarball)
    - [OCI tarball](#oci-tarball)
    - [containerd image store](#containerd-image-store)
- [Cache](#cache)
  - [Garbage collection](#garbage-collection)



 ## Information

:information_source: If you are visiting this repo for the usage of BuildKit-only Dockerfile features like `RUN --mount=type=(bind|cache|tmpfs|secret|ssh)`, please refer to [`frontend/dockerfile/docs/syntax.md`](frontend/dockerfile/docs/syntax.md).

<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

## Sample sTructure 

### Starting the `buildkitd` daemon:

You need to run `buildkitd` as the root user on the host.

```bash
$ sudo buildkitd
```

To run `buildkitd` as a non-root user, see [`docs/rootless.md`](docs/rootless.md).



## Clickable List


-   Dockerfile (See [Exploring Dockerfiles](#exploring-dockerfiles))
-   [Buildpacks](https://github.com/tonistiigi/buildkit-pack)
-   (open a PR to add your own language)


## Grey Text

(`gateway.v0`)


## Code Block


```bash
buildctl build \
    --frontend=dockerfile.v0 \
    --local context=. \
    --local dockerfile=.
# or
buildctl build \
    --frontend=dockerfile.v0 \
    --local context=. \
    --local dockerfile=. \
    --opt target=foo \
    --opt build-arg:foo=bar
```




##  Headings

## Quick start
### Exploring Dockerfiles
#### Building a Dockerfile with `buildctl`

