# GitOps multi-tenancy with [Flux v2](https://github.com/fluxcd/flux) and [Capsule](https://github.com/clastix/capsule)

> Forked from [fluxcd/flux2-multi-tenancy](https://github.com/fluxcd/flux2-multi-tenancy) repository.

Introduce the Tenant concept to GitOps multi-tenancy scenario with Flux v2.

## Architetcure

Please see [ARCHITECTURE.md](./docs/ARCHITECTURE.md).

![architecture](./docs/architeture.png)

## Quickstart

### 1. Fork this repository

#### 1.1. Update platform Sync configurations to point to the fork

- `clusters/staging/flux-system/gotk-sync.yaml`
```
  url: ssh://git@github.com/maxgio92/flux2-capsule-multi-tenancy
```
#### 1.2. Update tenants Sync configurations to point to the fork

- `tenants/base/dev-team/sync.yaml`
```
  url: https://github.com/maxgio92/flux2-capsule-multi-tenancy.git
```

### 2. Create a personal access token

This is needed to bootstrap Flux configuration to the forked repository and add a deploy key to read from it.

The access token is then discarded and can be revoked.

### 3. Provision a cluster

```
kind create cluster
```

### 4. Bootstrap Flux

```
export GITHUB_USER=<your-username>
export GITHUB_REPO=<repository-name>

flux bootstrap github \
    --owner=${GITHUB_USER} \
    --repository=${GITHUB_REPO} \
    --branch=main \
    --personal \
    --path=clusters/staging
```

### 5. Overwite Flux configuration with multitenancy lockdown (from local repo)

```
git push origin main -f
```
