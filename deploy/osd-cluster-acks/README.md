# Acknowledgement of Upgrade Gates

Some versions of Openshift may require manual steps as prerequisites for a successful cluster upgrade. These pre-requirements will be formulated in form of gates that block cluster-version-operator of kicking off an upgrade.

Read more about CVO gates [here](https://github.com/openshift/enhancements/blob/master/enhancements/update/upgrades-blocking-on-ack.md#general-implementation).

For AWS STS cluster an additional annotation on the cloud-credentials may be necessary.

This folder contains the SelectorSyncSets to sync the acknowledgment of these upgrade gates for OCP and STS to the clusters. Hive applies them based on the particular annotation of the ClusterDeployment and unblocks an upgrade.

## Layout

- `ocp/`, `sts/`, `gcp/`, `wif/`: `SelectorSyncSet`-based acks applied to classic (Hive-managed) clusters.
- `hcp/`: `Policy`-based acks for Hosted Control Plane (HCP) clusters.

## HCP clusters

The Cloud Credential Operator is enabled on HCP starting with 4.22, so from 4.22 onward HCP clusters also require the STS upgrade ack (the `cloudcredential.openshift.io/upgradeable-to` annotation on `CloudCredential/cluster`) before they can be upgraded.

HCP hosted clusters are managed through ACM policies rather than Hive SelectorSyncSets, so their acks are delivered as `ConfigurationPolicy` manifests (`deploymentMode: Policy`) instead of SelectorSyncSets. Each `hcp/<version>` directory patches the pre-existing `CloudCredential/cluster` resource on the hosted cluster to add the annotation (using `complianceType: musthave`, which merges the annotation without owning or recreating the object), unblocking the upgrade.

Directories under `hcp/` are added to the `directories` list in `scripts/generate-policy-config.py` so the ACM policy is generated into `deploy/acm-policies/`. The `config.yaml` `clusterSelectors` target hosted clusters (`hypershift.open-cluster-management.io/hosted-cluster: "true"`) at the appropriate `openshiftVersion-major-minor`.
