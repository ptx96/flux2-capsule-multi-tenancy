# Flow

## Platform configuration

**Platform Sync**: This Repository (flux-capsule-multi-tenancy) + main Branch (this)

> God reconciles this stuff.

Clusters:
  - Staging Cluster:
    - Flux System
    - Infrastructure: Capsule infrastructure
    - Tenants: Staging tenants
  
    - Staging tenants:
      - Dev Team tenant

> Here the Tenant and Tenant Owners are created.

        - RBAC
        - Sync

> Here the Tenant configurations are Sync-referenced \*.

                    - Source: GitRepo the-same + Branch dev-team
                    - Apply: Source above + Path /staging

## Tenant configuration

**Dev Team tenant Sync** \*: This Repository (flux-capsule-multi-tenancy) + dev-team Branch

> Tenant Owner reconciles this stuff:

Staging configuration:
    - Namespaces
    - Aps
