# Azure SQL access bootstrap

The Azure SQL access bootstrap is an automated, idempotent platform operation.
It is not a recurring manual `CREATE USER` procedure.

## Ownership

- Terraform creates the Entra access groups and the bootstrap, runtime, and
  migration managed identities.
- The infrastructure repository's protected `Bootstrap Azure SQL access`
  workflow builds and runs the reconciler inside AKS.
- This repository supplies the restricted `platform-system` namespace and its
  network boundary.
- Entra group owners manage human membership after bootstrap. Individual people
  are intentionally not stored in Terraform state.

## Reconciliation flow

1. An approved operator starts the protected workflow and types `bootstrap`.
2. GitHub authenticates to Azure with OIDC; no client secret is used.
3. The workflow reads non-secret identity and database metadata from Terraform
   outputs.
4. It builds and pushes an immutable bootstrap image to ACR.
5. A temporary Job runs in `platform-system` using Azure Workload Identity.
6. The Job connects to both databases through the SQL private endpoint.
7. It creates or verifies Entra group/managed-identity contained users, custom
   roles, grants, and role membership in one transaction per database.
8. The workflow records logs and removes the privileged Job and ServiceAccount.

The Job creates users by explicit Entra GUID/SID and principal type. This avoids
SQL passwords and avoids granting the SQL logical-server identity broad
Microsoft Graph directory-read permission.

## Human access model

| Entra group | Database | Database role |
|---|---|---|
| `azplab-sql-dev-developers` | `pharmacy-dev` | data read/write and execute |
| `azplab-sql-prod-readers` | `pharmacy-prod` | read-only |
| `azplab-sql-prod-admins` | `pharmacy-prod` | database control; JIT/PIM in a licensed production tenant |
| Platform administrators | both | database control and break-glass operations |

Adding or removing engineers is a group-membership operation in Entra. It does
not require rerunning SQL. Membership should use owner approval, access reviews,
and PIM/JIT for production where tenant licensing permits.

## Workload separation

- `pharmacy-backend` receives only data-plane runtime permissions.
- `pharmacy-migration` receives schema-change permissions and is invoked by the
  controlled deployment flow.
- `sql-access-bootstrap` is the Azure SQL Entra administrator and exists in the
  cluster only for the duration of the protected reconciliation workflow.

The current single-node cluster demonstrates the identity and authorization
model but does not provide production availability. A real production platform
would also use separate subscriptions/clusters, private runners or corporate
network access, PIM, and monitored approval boundaries.
