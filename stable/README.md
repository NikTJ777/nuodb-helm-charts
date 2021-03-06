### The instructions on this page are in two parts:

1. **[Getting Started with Helm][4]** covers how to install and configure Helm on a client host. It will walk you through deploying a canary application to make sure Helm is properly configured.
2. **[Deploying NuoDB using Helm Charts][5]** covers how to configure hosts to permit running NuoDB, and covers deploying your first NuoDB database using the provided Helm charts.

# Getting Started with Helm 

This section will walk you through getting the Helm client installed in your environment. If using Red Hat OpenShift, this page assumes that you already have the OpenShift `oc` client program installed locally and that you are logged into your OpenShift instance.

There are sub-charts in subdirectories included in this distribution. Instructions provided on this page are for initial configuration of Helm and Tiller, in some cases, required security settings. Sub-charts pages include instructions for deploying each required NuoDB component.

## Software Version Prerequisites

Please visit the **[NuoDB Helm Chart main page][6]** for software version prerequisites.

NuoDB Helm Charts and their privilege requirements:

| Helm Charts | Privilege | Short Explanation |
| ----- | ----------- | ------ |
| transparent-hugepage| allowHostDirVolumePlugin: true | To mount hostPath and disable THP on host|
| transparent-hugepage| volumes.hostPath | To mount hostPath and disable THP on host|
| transparent-hugepage| seLinuxContext.* | To mount hostPath and disable THP on host|
| admin, database| allowedCapabilities.FOWNER | To change directory ownership in PV to the nuodb process|
| admin, database| defaultAddCapabilities.FOWNER | To change directory ownership in PV to the nuodb process|


## Install Helm 3

If you are interested in Helm 2, please follow the [official Helm 2 docs][7].

Every [release][2] of Helm provides binary releases for a variety of OSes. 

1. Download your [desired version][2]
2. Unpack it (`tar -zxvf helm-${helm-version}-linux-amd64.tgz`)

We’ll use Helm version 3.2.4, which can be downloaded via <https://github.com/kubernetes/helm/releases/tag/v3.2.4>.

Run the following commands to install the Helm locally on your Linux client machine:
```bash
$ curl -s https://storage.googleapis.com/kubernetes-helm/helm-3.2.4-linux-amd64.tar.gz | tar xz
$ cd linux-amd64
```

Move the Helm binaries to /usr/local/bin
```
$ mv helm /usr/local/bin
```

If you're running on Mac, we recommend using the Brew Package manager.
```
brew install helm
```

Confirm you can run the Helm client: `helm help`.


## Create the _nuodb_ namespace to install NuoDB

```
kubectl create namespace nuodb
```

## Confirm that the Helm client is installed correctly 

The results should be as follows:

```bash
helm version
version.BuildInfo{Version:"v3.2.4", GitCommit:"0ad800ef43d3b826f31a5ad8dfbb4fe05d143688", GitTreeState:"dirty", GoVersion:"go1.14.3"}
```

# Deploying NuoDB using Helm Charts

The following section outlines the steps in order to deploy a NuoDB database using this Helm Chart repository.

## Configuration Parameters

Each Helm Chart has a default values.yaml parameter file that contains configuration parameters specific to that chart. The configuration is structured where configuration values are implemented following a single-definition rule, that is, values are structured and scoped, and shared across charts; e.g. for admin, its parameters are specified once in a single values file which is used for all the charts, and the database chart can use admin values for configuring connectivity of engines to a specific admin process. The same goes for other values **shared** amongst Helm charts. A few key points here:

- values files have structure, values are scoped
- different values files for different deployments
- values files follow the single definition rule (no repeats)
- global configuration exists under its own scoped section
- each chart has its own scoped section named after it
- cloud information is used to drive availability zones (particularly)

## Deployment Steps

**Note:** You MUST first disable Linux Transparent Huge Pages(THP) on all cluster nodes that will host NuoDB pods. Run the `transparent-hugepage` chart first.

- **transparent-hugepage** ([documentation](transparent-hugepage/README.md))

Optionally, consider configuring storage classes for persistent storage use by installing the NuoDB _Storage Classes_ chart. You can also use persistent storage without using the _Storage Classes_ Chart. See the chart documentation for existing options: 

- **Storage Classes** ([documentation](storage-class/README.md)) 

Deploy the NuoDB Components in this order : 

- **NuoDB Admin** ([documentation](admin/README.md)) 
- **NuoDB Database** ([documentation](database/README.md)) 


## Cleanup

See the instructions for the individual charts for deleting the applications.
An alternative cleanup strategy is to delete the entire project:

`kubectl delete namespace nuodb`

[1]: https://helm.sh/docs/using_helm/
[2]: https://github.com/helm/helm/releases
[4]: #getting-started-with-helm
[5]: #deploying-nuodb-using-helm-charts
[6]: https://github.com/nuodb/nuodb-helm-charts#software-release-requirements
[7]: https://v2.helm.sh/docs/using_helm/
