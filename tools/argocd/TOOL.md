---
name: argocd
description: "Work with Argo CD: applications and their resource tree, workload logs, events, clusters and projects — including create, sync and resource actions."
url: http://mcp-argocd.agent-mcps.svc.cluster.local/mcp
---

# argocd

Drives the Argo CD API: what exists (`list_applications`, `get_application`,
`get_appproject`, `list_clusters`), what a deployed app is actually doing
(resource tree, managed resources, workload logs, resource events), and what to
change about it (`create_application`, `update_application`,
`delete_application`, `sync_application`, `run_resource_action`).

Cluster-internal (`agent-mcps` namespace, no ingress), so registering it at all
depends on `MCP_INTERNAL_HOST_SUFFIXES` naming that suffix. Without it the sync
reports this entry as `invalid-url` and moves on.

**The token is the last mile.** The `mcp-argocd` chart in argocd-env-demo lands
the same `mcp-<slug>` Service on port 80 its six siblings use, but it cannot
carry the credential: Argo CD signs its own API tokens. argocd-env-addons'
`install/token.sh` mints one and pins it in parameter store alongside the token
list Argo CD checks it against, so the same token survives a cluster rebuild and
the installer needs no token step. Until that parameter holds a real token the
pod runs and `/healthz` stays green while every tool call answers 401 — a tool
error rather than a registration problem.

No credential on the entry: nothing routes to the Service from outside the
cluster, and the server authenticates to Argo CD with `ARGOCD_API_TOKEN` from
the pod's environment (External Secrets, parameter-store) rather than from the
caller. `ARGOCD_BASE_URL` points in-cluster at
`http://argocd-server.argocd.svc.cluster.local` — plain HTTP, because
argocd-env-addons runs the server with `server.insecure: true` and TLS
terminates at the gateway.

The write tools are listed because `MCP_READ_ONLY` is left unset, but what they
can actually do is Argo CD's own RBAC, not anything this entry decides. The
token belongs to the `mcp` account, which argocd-env-addons grants `role:mcp`:
applications (get, create, update, delete, sync and resource actions), their
logs, and read-only projects and clusters. Nothing else — repositories,
accounts, exec and the RBAC itself stay out of reach, so widening what a run may
do means editing that policy rather than this file.

Upstream: https://github.com/argoproj-labs/mcp-for-argocd
