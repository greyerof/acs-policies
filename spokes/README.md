# OpenShift PolicySet for ACS
ACM policies to deploy RedHat's Advanced Cluster Security Operator.

Based on [Openshift Platform Plus policies](https://github.com/open-cluster-management-io/policy-collection/tree/main/policygenerator/policy-sets/stable/openshift-plus)

There's two important things I changed from the OPP policies:
1. Changed namespace "policies" to "policies-acs" instead of "policies" as the namespace for the policies.
2. Changed hub cluster's namespace "stackrox" to "rhacs-operator", as that's the default namespace used by OLM when installing ACS throught the cluster's console GUI.

## Goal

The goal of these policies is to install ACS on each new provisioned/attached spoke cluster via ACM. Also, the new SecuredCluster CR is deployed in the new spoke cluster.

## The hack/trick
For new spoke clusters to be able to communicate with the hub cluster's Central service, they need the [init-bundle](https://docs.openshift.com/acs/4.6/installing/installing_ocp/init-bundle-ocp.html) (expected to be manually retrieved). The trick is performed in the hub cluster [by a job that runs a bash script](input-sensor/policy-acs-central-ca-bundle-v2.yaml#L73) in a [ose-cli container](input-sensor/policy-acs-central-ca-bundle-v2.yaml#L103) to retrieve the init-bundle from the central svc and creates the needed secrets in the "policies-acs" namespace. Then, the SecuredCluster CR reads those secrets using policy templating.

## Prerequisites
 - An install of Red Hat Advanced Cluster Management for Kubernetes version 2.7 or newer.
 - A namespace called "policies-acs" must be created containing two ManagedClusterSetBinding resources must to allow policies to apply to both "global" and "default" cluster sets.
```
kind: ManagedClusterSetBinding
metadata:
    name: global
    namespace: policies-acs
spec:
    clusterSet: global
---
kind: ManagedClusterSetBinding
metadata:
    name: default
    namespace: policies-acs
spec:
    clusterSet: default
```
 - When provisioning or attaching new spoke clusters, they must be assigned to the "default" cluster set.
 - ACS Operator must be already installed in namespace "rhacs-operator". Also, the Central CR should be already deployed in ACM's hub cluster
 - The PolicyGenerator plugin for kustomize must be installed and available locally. See [here](https://github.com/stolostron/policy-generator-plugin/tree/main?tab=readme-ov-file#install-the-binary)

## Installation

Point the KUBECONFIG env var to the ACM Hub cluster (ACS Central), and type:
```
kustomize build --enable-alpha-plugins  | oc apply -f -
```
or
```
oc kustomize --enable-alpha-plugins | oc apply -f -
```

## ToDo: policies explained
