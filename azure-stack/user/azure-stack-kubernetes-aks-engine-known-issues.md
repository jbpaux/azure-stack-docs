---
title: Known issues with the AKS engine on Azure Stack Hub 
description: Learn Known issues using the AKS engine on Azure Stack Hub. 
author: sethmanheim

ms.topic: article
ms.date: 11/1/2021
ms.author: sethm
ms.reviewer: waltero
ms.lastreviewed: 11/1/2021

# Intent: Not done: As a < type of user >, I want < what? > so that < why? >
# Keyword: Not done: keyword noun phrase

---

# Known issues with the AKS engine on Azure Stack Hub

This topic covers known issues for the AKS engine on Azure Stack Hub.

[!INCLUDE [Expired secret for service principal (SPN) causes cluster to fail](../includes/known-issue-aks-2.md)]

[!INCLUDE [Expired certificates for the front-proxy](../includes/known-issue-aks-3.md)]

## Limit of 50 nodes per subscription

- **Applicable to**: Azure Stack Hub, AKS engine (all)
- **Description**: When creating clusters, you need to ensure that there are not more than 50 Kubernetes nodes (control plane and agent nodes) deployed per subscriptions. The total Kubernetes nodes deployed across all clusters within a single subscription shouldn't exceed 50 nodes.
- **Remediation**: Use less then 51 nodes in your subscription.
- **Occurrence**: When attempting to add more than 50 nodes per subscription.

## Unable to resize cluster VMs with the Compute service

- **Applicable to**: Azure Stack Hub, AKS engine (all)
- **Description**: Resizing cluster VMs through the Compute service doesn't work with AKS engine. The AKS engine maintains the state of the cluster in the API model json file. To ensure that the desired VM size is reflected in any create, upgrade, or scale operation done with AKS engine you must update the API model before executing any of those operations. For example, if you change a VM size  on an already deployed cluster to a different size using the Compute service, the state will be lost when `aks-engine upgrade` is executed.
- **Remediation**: To make this work locate the API model for the cluster, change the size there, and then run `aks-engine upgrade`.
- **Occurrence**: When attempting to resize using the Compute service.

## Disk detach operation fails in AKS engine 0.55.0

- **Applicable to**: Azure Stack Hub (update 2005), AKS engine 0.55.0
- **Description**: When you try to delete a deployment that contains persistence volumes, the delete operation triggers a series of attach/detach errors. This is due to a bug in the AKS engine v0.55.0 Cloud Provider. The Cloud Provider calls the Azure Resource Manager using a newer version of the API than the Azure Resource Manager currently supports in Azure Stack Hub (update 2005).
- **Remediation**: You can find the details and mitigation steps in [the AKS engine GitHub repository (Issue 3817)](https://github.com/Azure/aks-engine/issues/3817#issuecomment-691329443). Upgrade as soon as a new build of AKS engine and corresponding image are available.
- **Occurrence**: When deleting a deployment that contains persistence volumes.



## Upgrade issues in AKS engine 0.51.0

* During upgrade (aks-engine upgrade) of a Kubernetes cluster from version 1.15.x to 1.16.x, upgrade of the following kubernetes components requires extra manual steps: **kube-proxy**, **azure-cni-networkmonitor**, **csi-secrets-store**, **kubernetes-dashboard**. The following describes what you may observe and how to work around the issues.

  * In connected environments, it is not obvious to notice this issue since there are no signs in the cluster that the affected components were not upgraded. Everything appears to work as expected.
  <!-- * In disconnected environments, you can see this problem when you run a query for the system pods status and see that the pods for the components mentioned below are not in "Ready" state: -->

    ```bash  
    kubectl get pods -n kube-system
    ```

  * As a workaround to solve this issue for each of these components, run the command in the Workaround column in the following table.

    |Component Name	|Workaround	|Affected Scenarios|
    |---------------|-----------|------------------|
    |**kube-proxy**	    | `kubectl delete ds kube-proxy -n kube-system`	|Connected, Disconnected |
    |**azure-cni-networkmonitor**	| `kubectl delete ds azure-cni-networkmonitor -n kube-system`	| Connected, Disconnected |
    |**csi-secrets-store**	|`sudo sed -i s/Always/IfNotPresent/g /etc/kubernetes/addons/secrets-store-csi-driver.yaml`<br>`kubectl delete ds csi-secrets-store -n kube-system` | Disconnected |
    |**kubernetes-dashboard** |Run the following command on each control plane node:<br>`sudo sed -i s/Always/IfNotPresent/g /etc/kubernetes/addons/kubernetes-dashboard.yaml` |Disconnected |

* Kubernetes 1.17 is not supported in this release. Although there are GitHub pull requests (PR)s referencing 1.17, it is not supported.

## Next steps

[AKS engine on Azure Stack Hub overview](azure-stack-kubernetes-aks-engine-overview.md)
