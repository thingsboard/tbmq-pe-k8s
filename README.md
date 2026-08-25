# TBMQ PE Kubernetes scripts and docs

Here you can find scripts for deployment on different Kubernetes platforms.

Each platform folder is self-contained: the Kubernetes resource configurations plus the shell scripts to install,
deploy, upgrade and delete [TBMQ Professional Edition](https://tbmq.io/) in a `thingsboard-mqtt-broker` namespace.

## Platforms

| Folder      | Target                              | Deployment guide                                                              |
|-------------|-------------------------------------|-------------------------------------------------------------------------------|
| `aws/`      | Amazon EKS                          | [AWS cluster setup](https://tbmq.io/docs/pe/installation/cluster/aws-cluster-setup/)           |
| `gcp/`      | Google GKE                          | [GCP cluster setup](https://tbmq.io/docs/pe/installation/cluster/gcp-cluster-setup/)           |
| `azure/`    | Azure AKS                           | [Azure cluster setup](https://tbmq.io/docs/pe/installation/cluster/azure-cluster-setup/)       |
| `minikube/` | Local single-node cluster           | [Minikube cluster setup](https://tbmq.io/docs/pe/installation/cluster/minikube-cluster-setup/) |
| `helm/`     | Cluster prerequisites for Helm chart installation | —                                                                |

Notes:

* `aws/cluster.yml` is the `eksctl` cluster definition for the manifest-based AWS deployment; `helm/aws/` holds the
  cluster definition with labeled node groups (`kafka`, `redis`, `postgresql`) and a `gp3` default StorageClass used by
  the TBMQ Helm chart installation.
* `gcp/kafka/` and `azure/kafka/` describe two optional ways to run Kafka inside the cluster: plain StatefulSet in KRaft
  mode (`kafka/README.md`) or a Strimzi-managed cluster (`kafka/operator/README.md`). AWS deployments use Amazon MSK by
  default.
* `minikube/` additionally deploys PostgreSQL, Valkey and Kafka in-cluster (`postgres.yml`, `valkey.yml`, `kafka.yml`),
  exposes TBMQ via `routes.yml`, and contains a PostgreSQL [backup and restore](minikube/backup-restore/README.md) guide.

## Resource configurations

| File                        | Description                                                                                                      |
|-----------------------------|------------------------------------------------------------------------------------------------------------------|
| `tbmq-namespace.yml`        | `thingsboard-mqtt-broker` namespace                                                                              |
| `tbmq-configmap.yml`        | Broker `conf` (JVM options) and `logback` configuration                                                          |
| `tbmq-db-configmap.yml`     | PostgreSQL connection settings (cloud platforms only)                                                            |
| `tbmq-cache-configmap.yml`  | Valkey/Redis connection settings, standalone or cluster (cloud platforms only)                                   |
| `tbmq.yml`                  | Broker StatefulSet and Service: `8083` http, `1883`/`8883` MQTT(S), `8084`/`8085` MQTT over WS(S)                 |
| `tbmq-ie-configmap.yml`     | Integration Executor `conf` and `logback` configuration                                                          |
| `tbmq-ie.yml`               | Integration Executor Deployment                                                                                  |
| `database-setup.yml`        | Short-lived `tb-db-setup` pod used to install or upgrade the database schema                                      |
| `receipts/`                 | Load balancer / ingress configurations for HTTP, HTTPS and MQTT(S) traffic (cloud platforms only)                 |

## Scripts

All scripts are meant to be executed from within the platform folder.

| Script                                    | Description                                                                    |
|-------------------------------------------|--------------------------------------------------------------------------------|
| `k8s-install-tbmq.sh`                     | Creates the namespace and config maps, then installs the database schema        |
| `k8s-deploy-tbmq.sh`                      | Deploys the broker and the Integration Executor                                 |
| `k8s-upgrade-tbmq.sh --fromVersion=X.Y.Z` | Re-applies the config maps and runs the database upgrade from the given version |
| `k8s-delete-tbmq.sh`                      | Deletes the broker and the Integration Executor                                 |
| `k8s-delete-all.sh`                       | Deletes all resources in the namespace, including the persistent volume claims  |

## Before you deploy

* Replace the placeholder values in `tbmq-db-configmap.yml` (`SPRING_DATASOURCE_*`) and `tbmq-cache-configmap.yml`
  (`REDIS_HOST` or `REDIS_NODES`) with the endpoints of your database and cache. The `minikube/` setup runs both
  in-cluster and needs no changes.
* Create the license secret referenced by `tbmq.yml`:

  ```bash
  kubectl create secret generic tbmq-license --from-literal=license-key=YOUR_LICENSE_KEY -n thingsboard-mqtt-broker
  ```
* The manifests reference the latest TBMQ PE release images (`thingsboard/tbmq-pe-node`,
  `thingsboard/tbmq-pe-integration-executor`). Change the image tags in `tbmq.yml` and `tbmq-ie.yml` to deploy a
  different version.

## License

The scripts and configurations in this repository are licensed under the Apache License 2.0, see [LICENSE](LICENSE).
TBMQ Professional Edition itself is a commercial product and requires a valid license key.
