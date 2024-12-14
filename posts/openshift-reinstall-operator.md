---
title: 'How to reinstall an operator on OpenShift'
date: 2024-06-24T09:17:06+02:00
tags: ['openshift', 'olm', 'how-to']
---

I recently ran into a problem where the install plan of an operator failed and I had to figure out how to reinstall an operator.
It was definitely not as easy as I thought, so I decided to document how I managed to do it successfully.

## Preparations

Before proceeding you must create a backup of the operator subscription in your target namespace, otherwise you might have a hard time being able to install the operator again.
Additionally I would propose to also create a backup of the ClusterServiceVersion, which you tried to deploy.

```shell
oc get subscription -n example my-operator-o yaml > subscription.yaml
oc get clusterserviceversion -n example my-operator.v2.3.3 -o yaml > clusterserviceversion.yaml
```

## Recreating the subscription

Before your able to recreate the subscription you should clean up the `subscription.yaml` file by deleting the `status` field.
Now you can proceed to recreate the subscription by deleting the subscription, as well as the ClusterServiceVersion object and any install plans present.

!!! WARNING !!! This should usually not result in the removal of the operator and should not result in data loss, but I can't guarantee this for every operator. You have been warned! !!! WARNING !!!

```shell
oc delete subscription -n example my-operator
oc delete clusterserviceversion -n example my-operator.v2.3.3
oc delete installplan -n example --all
```

Now we can proceed with recreating the subscription.
If you previously had a specific version version of the operator installed your subscription backup will contain `spec.installPlanApproval: Manual` and `spec.startingCSV: my-operator.v2.3.3` and you might have to update the version for `spec.startingCSV` to the one you're trying to install.

```shell
oc apply -f subscription.yaml
```

From now on there are two possible paths, depending on your configuration for `installPlanApproval`.

### Automatic install plan approval

After recreating the subscription it should be picked up by the olm-operator pod in the openshift-operator-lifecycle-manager namespace and a new ClusterServiceVersion object should be created and your operator should be updated to the most current version.
If this is not the case have a look at [recreating the catalog job](#recreating_catalog_job).

### Manual install plan approval

You should now see a new installplan in your operator namespace:

```shell
oc get installplan -n example
```

open the installplan and configure `spec.approved: true`

```shell
oc edit installplan install-abcde
```

after that the ClusterServiceVersion you request should be created and your operator should be installed to the version you requested.
If this is not the case have a look at [recreating the catalog job](#recreating_catalog_job).

### Recreating catalog job

One issue I ran into during the last time I had to perform the above tasks was that for some reason the installplan and ClusterServiceVersion were not created.
After a while debugging I found the error message "already existing job abcdef..." in the output of the catalog-operator pod in the openshift-operator-lifecycle-manager namespace, immediately after I created the subscription.
So all I've had to do was delete the job with the ID mentioned in the error message and recreate the subscription again to resolve this issue:

```shell
oc delete job -n openshift-operator-lifecycle-manager abcedf...
```
