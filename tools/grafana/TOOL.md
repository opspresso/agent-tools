---
name: grafana
description: "Query Grafana: dashboards, datasources, Prometheus and Loki queries, alerting and incidents."
url: http://mcp-grafana.agent-mcps.svc.cluster.local/mcp
---

# grafana

Cluster-internal (`agent-mcps` namespace, no ingress), so registering it at all
depends on `MCP_INTERNAL_HOST_SUFFIXES` naming that suffix. Without it the sync
reports this entry as `invalid-url` and moves on.

No credential: nothing routes to the Service from outside the cluster.
