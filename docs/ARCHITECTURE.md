# GitOps and Multi-tenancy: Flux locking feature and Capsule

## Flux

### The process

#### 1. Bootstrap

1.1. Install flux controller on the specified cluster (--context arg value)

1.2. Flux controller pushes basic Flux configuration to the specified repository and path (where the Infra (Flux system itself included) and platform (i.e. tenants) desired state is declared):

- Flux GitOps ToolKit operators and related RBAC.
- Sync (`Source` and `Kustomiaztion`) for the GOTK resources themselves (previous point).

	Example commits:
	
	```
	Add Flux sync manifests
	Flux committed 1 hour ago
	
	Add Flux v0.30.2 component manifests
	Flux committed 1 hour ago 
	```

> From now on, it no longer needs to write to the repository. The PAT can be revoked and will be discared.

1.3. Flux controller generate and pushes a deploy key (r/o access) for the gitops Platform admin repository, to reconcile on the cluster the desired state (on Git) with the actual state (on K8s)

1.4. Flux controller reconciles actual state on linked cluster (--context arg value) based on what it reads above ^^^

#### 2. Updates

2.1. Push to repo

2.2. Flux reconciles based on Kustomizations which reference Source (e.g. GitRepository)

### The desired state configuration hierarchy

#### 1. Platform admin repo/branch (Source)

1.1. Cluster configuration with Flux System

1.2. (optional) Cluster configuration with infra (e.g. policies)

1.3. Cluster configuration with Tenants (i.e. dev-team) (the sync source repo/branch + kustomization)

- Tenant Sync's RBAC configuration should configured with least privilege (service account that the kustomization controller should **impersonate**)

- Tenant Sync's Kustomization configuration should make the Kustomize controller impersonate the service account above ^^^ for least privilege principle for soft MULTI-TENANCY 

- Tenant Sync's Source configuration should refer to the tenant-dedicated repo/branch Source

#### 2. Tenant (e.g. dev-team) repo/branch (Source)

2.1. kustomization on the sync source repo/branch specified by the Platform admin (i.e. same repo, dev-team branch)

### Multi-tenancy lockdown Flux feature

> Note: the isolation is per-Namespace, not per-Tenant

With [this configuration](https://fluxcd.io/docs/installation/#multi-tenancy-lockdown), Flux will:

- Deny cross-namespace access to Flux custom resources, thus ensuring that a tenant can’t use another tenant’s sources or subscribe to their events.
- Deny accesses to Kustomize remote bases, thus ensuring all resources refer to local files, meaning only the Flux Sources can affect the cluster-state.
- All Kustomizations and HelmReleases which don’t have spec.serviceAccountName specified, will use the default account from the tenant’s namespace. Tenants have to specify a service account in their Flux resources to be able to deploy workloads in their namespaces as the default account has no permissions.
- The flux-system Kustomization is set to reconcile under a service account with cluster-admin role, allowing platform admins to configure cluster-wide resources and provision the tenant’s namespaces, service accounts and RBAC.


## Enter Capsule

### Multi-tenancy lockdown with Capsule

> Note: the isolation is per Tenant (Capsule CRD)

1. Tenant user can only access resources on a group of Namespaces, which are part of the Tenant.
2. Use Tenant Owner as the identity the Kustomize controller should **impersonate**, in order to reconcile the desired state of all and only its Tenant

### Multi-tenancy lockdown with Flux **and** Capsule

- With Capsule: Tenant Owner identity to deny cross-Tenant access. Capsule will then do the work during requests validation.
- With Flux: Deny accesses to Kustomize remote bases, thus ensuring all resources refer to local files, meaning only the Flux Sources can affect the cluster-state.
- With Flux: All Kustomizations and HelmReleases which don’t have spec.serviceAccountName specified, will use the default account from the tenant’s namespace. Tenants have to specify a service account in their Flux resources to be able to deploy workloads in their namespaces as the default account has no permissions.
- With Flux: The flux-system Kustomization is set to reconcile under a service account with cluster-admin role, allowing platform admins to configure cluster-wide resources and provision the tenant’s namespaces, service accounts and RBAC.
