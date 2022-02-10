# IBM Cloud AirGapped Cluster with GitOps Configuration

The IBM Go To Market Assets and Architecture team has developed automation that will allow you to deploy an AirGapped Cluster with Gitops Configuration in [TechZone](https://techzone.ibm.com/collection/production-deployment-guides#tab-6) based on the principles outlined in our [Cloud Pak Production Deployment Guides](https://production-gitops.dev/infrastructure/restricted-networks/).

## Reference Architecture

![Reference Architecture](./media/refarch.svg)

---

Leveraging a pre-existing [IBM Cloud VPC Network](https://cloud.ibm.com/docs/vpc), we deploy a Bastion Host into a subnet with a public gateway, and a Private Image Registry into a subnet with no outbound access.  Only traffic allowed between the two subnets is SSH and HTTPS traffic.

On the Bastion Host, a 1TB disk is mounted in /data for any post-deployment activities you may need to perform.  Although SSH access from the bastion server to the image registry server is allowed, we encourage you not to SSH into the image server directly, as this would not mimic a production environment.

On the Registry Server, we deploy [Harbor](https://goharbor.io/) to provide the private registry, and [Gitea](https://gitea.io/) to provide a private Github repository, prepopulated with the [Multi-Tenancy Gitops Framework](https://github.com/cloud-native-toolkit/multi-tenancy-gitops) from the [Cloud Native Toolkit](https://cloudnativetoolkit.dev/).

Finally, an OpenShift cluster is deployed into another set of subnets with no public gateways, and so the cluster will have no outbound access.

A VPN Server is deployed in each region to provide access to the OpenShift Cluster, as well as the UIs of the OpenShift Cluster, Image Registry and GitHub server.

## Welcome Email

After the environment is deployed from TechZone, you will receive an email from TechZone with your environment details.  The email will include:

- Next Steps:  A link to this repository
- Bastion Host and Registry Host IP Addresses (Private and Public)
- Bastion Host SSH Key
- Image Registry Private IP Address
- Details of your OpenShift Cluster, with links to the IBM Cloud Cluster page
- Harbor URL and Login Credentials
- Gitea URL and Login Credentials
- ArgoCD URL and Login Credentials
- VPN Configuration to access the environment

Some fields in this email will be unformated.  You will be able to download the SSH Key and VPN Configuration by going to the [My Reservations](https://techzone.ibm.com/my/reservations) page in TechZone, clicking your provisioned environment, and clicking the `Download SSH Key` and `Download OpenVPN Config` buttons.

## Environment Overview

### VPN Server

To access the multiple WebUIs deployed in this solution, a VPN Configuration is included in your welcome email. On a Mac, you can use [TunnelBlick](https://tunnelblick.net/) to connect.  On Windows or Linux, you may use [OpenVPN](https://openvpn.net/vpn-client/).

### Bastion Host

The bastion host is the only component in the environment with a public IP address.  It's an Ubuntu 20.04 server with a 1TB /data disk for any components you need to download.  It has docker-ce installed, `oc`, `skopeo` and `ibmcloud` cli utilities. You may SSH into it as the `ubuntu` user with the SSH key you received in the welcome email.

```bash
$ chmod 400 ~/Downloads/pem_airgap_download.pem
$ ssh -i ~/Downloads/pem_airgap_download.pem ubuntu@150.240.66.31
The authenticity of host '150.240.66.31 (150.240.66.31)' can't be established.
ED25519 key fingerprint is SHA256:/KPi86xF4+TLOjVBmSsf+LCm6Vcm6jZHCKQeIL4HgtQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '150.240.66.31' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Feb 10 03:55:17 UTC 2022

  System load:  0.0               Processes:                179
  Usage of /:   4.7% of 96.75GB   Users logged in:          0
  Memory usage: 1%                IPv4 address for docker0: 172.17.0.1
  Swap usage:   0%                IPv4 address for ens3:    10.1.16.11


135 updates can be applied immediately.
64 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Thu Feb 10 03:54:43 2022 from 24.171.197.170
ubuntu@itzroks-1100007b1r-bzpo1222-bastion:~$
```

### Registry Server

The registry server is also an Ubuntu 20.04 server with a 1TB /data disk meant to be used for your private image registry.  The following componets are pre-installed into the cluster

#### Harbor

Harbor is pre-installed into the server on port 443.  You can access the Harbor UI by clicking on the Harbor UI link in the welcome email.  Credentials are in the welcome email as well. ChartMuseum, a helm chart repository, is pre-installed into Harbor, as well as Trivy, a vulnerability scanner for your container images.

The most common registry namespaces are created, with `ocp4` prepopulated with the OpenShift Platform images, and `olm` used to host a limited subset of the OperatorHub Catalog Images (openshift-ocs, openshift-gitops, openshift-pipelines). Prepopulated into the `toolkit-charts` registry namespace are all the Helm charts offered by the [Cloud Native Toolkit](https://charts.cloudnativetoolkit.dev/).

Please consult the [Harbor documentation](https://goharbor.io/docs/2.3.0/) for additional information and instructions.

#### Gitea

Gitea is also pre-installed into the Registry Server on port 3000.  It provides a Private Github Repository for your GitOps deployments.  A fork of the [Cloud Native Toolkit GitOps Framework](https://github.com/cloud-native-toolkit/multi-tenancy-gitops) (and -infra, -services, -apps) is deployed into the Gitea server under the `cntk-gitops` org.
The repos are also cloned on the Bastion Host under `/home/ubuntu/repositories` with the gitea git token already included in the repo configuration.

Please consult the [Gitea documentation](https://docs.gitea.io/) for additional information and instructions.

### OpenShift Cluster

Your welcome email will also include a link to the IBM Cloud ROKS Cluster page.  The default is a 4.8 cluster, but 4.7 is also an option. To access the cluster from command line, you can SSH into the bastion host. The cluster `kubeconfig` file is pre-loaded into `~/.kube/config`, with full cluster-admin access, so you can automatically execute `oc` commands without any futher configuration changes.

```bash
ubuntu@itzroks-1100007b1r-bzpo1222-bastion:~$ oc get nodes
NAME          STATUS   ROLES           AGE    VERSION
10.1.0.31     Ready    master,worker   143m   v1.21.6+bb8d50a
10.1.0.32     Ready    master,worker   139m   v1.21.6+bb8d50a
10.1.128.17   Ready    master,worker   142m   v1.21.6+bb8d50a
10.1.128.18   Ready    master,worker   141m   v1.21.6+bb8d50a
10.1.64.17    Ready    master,worker   143m   v1.21.6+bb8d50a
10.1.64.18    Ready    master,worker   141m   v1.21.6+bb8d50a
```

#### OpenShift Container Storage

If your workloads require RWX storage classes, you can deploy OpenShift Data Foundation when deploying your cluster.  You can choose between 500GB and 2TB of storage.

#### OpenShift GitOps Operator

We also deploy the OpenShift GitOps Operator on your cluster.  The URL and credentials are included in the welcome email.  We automatically deploy the 4 `ArgoCD Applications` for bootstrap,  infra, services and applications, each corresponding to the `cntk-gitops` repositories under `/home/ubuntu/repositories`.

## ClouPak Deployments

Familiarize yourself with  your CloudPak AirGap deployment processes.  A cloudpak agnostic deployment model can be located in our Cloud Pak Production Deployment Guides [Restricted Networks](https://production-gitops.dev/infrastructure/restricted-networks/#mirroring-cloudpak-container-images) section.

Once the images are mirrored, follow the [Update imageContentSourcePolicy](#update-image-content-source-policy) section to update your cluster nodes.

## Pushing External Images to your Private Image Registry

To add any additional images you may need into your registry, you can use the `skopeo` utility.  First logon to both source and private registry with skopeo, then use `skopeo copy` to copy the images

```bash
ubuntu@itzroks-1100007b1r-bzpo1222-bastion:~/$ skopeo login docker.io                                                        
Username: yourusername
Password: 
Login Succeeded!
ubuntu@itzroks-1100007b1r-bzpo1222-bastion:~/$ skopeo login 10.1.32.9
Username: admin
Password:
Login Succeeded!
ubuntu@itzroks-1100007b1r-bzpo1222-bastion:~/$ skopeo copy docker://docker.io/hello-world:latest docker://10.1.32.9/ocp4/hello-world:latest
Getting image source signatures
Copying blob 2db29710123e done
Copying config feb5d9fea6 done
Writing manifest to image destination
Storing signatures
```

## Updating Image Content Source Policies

TODO:  Document DaemonSet and how to update it
