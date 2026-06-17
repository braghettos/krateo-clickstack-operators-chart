# krateo-clickstack-operators-chart

Krateo composition wrappers for the **ClickStack observability data-layer operators** — packaged
so the Krateo installer can provision them as **`observability`-gated composition components**
(not bootstrap-only subcharts). Enabling `observability` on the `Installer` CR then installs the
operators **first**, and `krateo-clickstack` (which depends on them) creates its
`ClickHouseCluster` / `KeeperCluster` / `MongoDBCommunity` CRs only once the operators are up and
their CRDs are served.

Each chart is a thin **dependency-wrapper over the UNFORKED upstream operator chart**. The
upstream CRDs ship in `templates/` (not the helm-special `crds/` dir), so core-provider applies
them on render and the composition's helm release deletes them cleanly on teardown.

| Chart | Kind | Wraps | Published to |
|---|---|---|---|
| `charts/clickhouse-operator` | `ClickhouseOperator` | `clickhouse-operator-helm` `0.0.5` (oci://ghcr.io/clickhouse); webhook + cert-manager disabled | `oci://ghcr.io/braghettos/krateo/clickhouse-operator` |
| `charts/mongodb-operator` | `MongodbOperator` | MongoDB `community-operator` `0.13.0` (+ bundled `community-operator-crds`); no sample CR | `oci://ghcr.io/braghettos/krateo/mongodb-operator` |

## Why wrappers (and the alias)

core-provider generates each component's CRD from the chart's `values.schema.json` and names the
Kind after the chart. A values key that pascalizes to the chart's own Kind collides in crdgen, so
each upstream dependency is **aliased to `operator`** — Helm reads the subchart's values from
`operator:` and never from a key that matches the Kind.

## Teardown ordering

`krateo-clickstack` depends on these operators, so the installer's reverse-dependency drain
(`ordered-teardown` pre-delete hook) removes clickstack + its data-layer CRs **before** the
operators — the operators stay alive to clear those CRs' finalizers.

## Release

Tag `N.N.N` (or run the `release-oci` workflow); both charts publish to the `braghettos/krateo`
OCI namespace at their own literal `Chart.yaml` versions.
