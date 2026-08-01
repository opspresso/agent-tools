---
name: kubernetes
description: "Read-only Kubernetes cluster inspection: resources, pod logs, events, nodes and namespaces."
url: http://mcp-kubernetes.agent-mcps.svc.cluster.local/mcp
---

# kubernetes

Cluster-internal (`agent-mcps` namespace, no ingress), so registering it
depends on `MCP_INTERNAL_HOST_SUFFIXES` naming that suffix, same as the other
entries here.

No credential on the entry: nothing routes to the Service from outside the
cluster, and the server reads the cluster with its own ServiceAccount rather
than anything from the caller.

Read-only is enforced twice, in the deployment rather than here: `--read-only`
on the pod drops every mutating tool before it is listed, and the
ServiceAccount's RBAC grants get/list/watch with core `secrets` deliberately
excluded. `--toolsets core` follows from that exclusion — helm stores releases
in Secrets, so its toolset would only ever answer 403. Widening what this entry
can see means changing those flags and rules in argocd-env-demo's `agent-mcps`
chart, not this file.

Upstream: https://github.com/containers/kubernetes-mcp-server
